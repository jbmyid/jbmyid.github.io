---
layout: post
title: "Paypal Subscription Plan"
quote: "Creating subscription plan and subscribing plan for recursive payments."
image: /media/2014-09-25-paypal-billing-plan-and-subscriptions-rails/Paypal-logo.jpg
video: false
comments: true
---
#Prerequisites
- Create paypal account in developer section of [paypal](https://developer.paypal.com/).
- Login and go to dashboard and then to my apps section.
- Create new app
- After app creation you will get api credentials for production and sandbox.
- Copy `client_id` and `client_secret` and keep in secret which will be needing in next step.

#Setup Gem

How to setup application to use paypal-sdk-rest gem.

- Add gem “paypal-sdk-rest” in gemfile
  - <pre>gem "paypal-sdk-rest"</pre>
- Run `bundle install`
- Run `rails g paypal:sdk:install`.
  It will generate two files in config folder
  1. config/paypal.yml
  2. config/initializers/paypal.rb
- Paste the credenetials which where obtained from application creation in paypal in `config/paypal.yml`.

The `paypal-sdk-rest` gem currently does not include billing plan api which can be added manually.
Create `paypal_ext.rb` and copy following code and place in `config/initializers` folder.

<pre>
<code class="ruby">
  module PayPal::SDK
    module REST
      module DataTypes
        Payer.class_eval do
          object_of :merchant_id, String
        end

        class ChargeModel < Base
          object_of :id, String
          object_of :type, String
          object_of :amount, Currency
        end

        class PaymentDefination < Base
          object_of :name, String
          object_of :type, String #UNLIMITED or FIXED
          object_of :frequency_interval, String
          object_of :frequency, String # DAY, WEEK, MONTH or YEAR.
          object_of :cycles, Integer
          object_of :amount, Currency
          object_of :id, String
          array_of :charge_models, ChargeModel
        end

        class MerchantPreference < Base
          object_of :id, String
          object_of :cancel_url, String
          object_of :return_url, String
          object_of :setup_fee, Currency
          object_of :max_fail_attempts, Integer
          object_of :auto_bill_amount, Currency
          object_of :initial_fail_amount_action, Currency
          object_of :create_time, DateTime
          object_of :update_time, DateTime
        end

        class BillingPlan < Base
          # {
          # name: "Plan Name", type: "INFINITE", description: "Any description",
          # payment_definitions: {name: "Name", type: "REGULAR", frequency_interval: 1, amount: {value: 2.3, currency: "USD"}, frequency: "MONTH"},
          # merchant_preferences: {cancel_url: "http://google.com", return_url: "http://google.com", setup_fee: {value: 0, currency: "USD"}}
          # }
          object_of :id, String
          object_of :name, String
          object_of :description, String
          object_of :type, String
          array_of :payment_definitions, DataTypes::PaymentDefination
          object_of :merchant_preferences, MerchantPreference
          object_of :state, String
          array_of  :links, DataTypes::Links
          object_of :payee, DataTypes::Payer
          object_of :create_time, DateTime
          object_of :update_time, DateTime


          def self.load_members
            object_of :name, String
            object_of :description, String
          end

          include RequestDataType
          def create
            path = "v1/payments/billing-plans"
            response = api.post(path, self.to_hash)
            self.merge!(response)
            success?
          end

          def self.find(resource_id)
            raise ArgumentError.new("id required") if resource_id.to_s.strip.empty?
            path = "v1/payments/billing-plans/#{resource_id}"
            self.new(api.get(path))
          end

          def update(attrs)
            path = "v1/payments/billing-plans/#{self.id}"
            attrs = attrs.blank? ? [{path: "/", value: {state: "ACTIVE"}, op: "replace"}] : attrs
            response = api.patch(path, attrs)
            self.merge!(response)
            success?
          end

          def activate!
            update([{path: "/", value: {state: "ACTIVE"}, op: "replace"}])
          end

        end

        class AgreementDetails < Base
          object_of :outstanding_balance, Currency
          object_of :cycles_remaining, Integer
          object_of :cycles_completed, Integer
          object_of :final_payment_date, DateTime
          object_of :failed_payment_count, Integer
        end

        class AgreeMent < Base
          object_of :id, String
          object_of :name, String
          object_of :description, String
          object_of :start_date, DateTime
          object_of :payer, Payer
          object_of :plan, BillingPlan
          array_of :links, DataTypes::Links
          object_of :override_merchant_preferences, MerchantPreference
          object_of :state, String
          object_of :agreement_details, AgreementDetails

          include RequestDataType

          def create
            path = "v1/payments/billing-agreements"
            response = api.post(path, self.to_hash)
            self.merge!(response)
            success?
          end

          def approval_url
            links.select{|link| link.rel=="approval_url"}[0].try :href
          end

          def execute_url
            links.select{|link| link.rel=="execute"}[0].try :href
          end

          def execute!(token)
            response = api.post("v1/payments/billing-agreements/#{token}/agreement-execute")
            self.merge!(response)
            success?
          end

          def suspend
            path = "v1/payments/billing-agreements/#{id}/suspend"
            response = api.post(path, {note: "Suspending the subscription"})
            self.merge!(response)
            success?
          end

          def reactivate
            path = "v1/payments/billing-agreements/#{id}/re-activate"
            response = api.post(path, {note: "Reactivating the agreement."})
            self.merge!(response)
            success?
          end

          def cancel
            path = "v1/payments/billing-agreements/#{id}/cancel"
            response = api.post(path, {note: "Cencelling the agreement."})
            self.merge!(response)
            success?
          end

          def self.find(resource_id)
            raise ArgumentError.new("id required") if resource_id.to_s.strip.empty?
            path = "v1/payments/billing-agreements/#{resource_id}"
            self.new(api.get(path))
          end

        end


      end
    end
  end
</code>
</pre>

Now you can create billing plans using rest api.

#Creating Billing Plans

Create `BillingPlan` model and add fields required along with paypal_id.
Create `ActAsPaypalPlan` concern in `models/concernes` folder and paste following code.
<pre>
  <code>
    module ActAsPaypalPlan
      extend ActiveSupport::Concern
      include PayPal::SDK::REST::DataTypes

      included do
        after_validation :set_paypal_plan, on: :create
      end

      def paypal_plan
        if paypal_id
          @paypal_plan ||= BillingPlan.find(paypal_id)
        else
          @paypal_plan ||= BillingPlan.new(name: name, type: "INFINITE", description: description,
            payment_definitions: {name: "PaymentDefinition#{name}", type: "REGULAR", frequency_interval: 1, amount: {value: price, currency: "USD"}, frequency: interval},
            merchant_preferences: {cancel_url: "plan_cancel_url", return_url: "return_url", setup_fee: {value: 0, currency: "USD"}})
        end
      end

      def agreement(options={})
        @agreement ||= AgreeMent.new(name: "Subscribe #{name}", description: "Subscribe #{name}", start_date: Date.today+1.days, plan: {id: paypal_plan.id}, payer: {payment_method: "paypal"}, override_merchant_preferences: {cancel_url: options[:cancel_url], return_url: options[:return_url]})
      end

      private
      def set_paypal_plan
        return if errors.any?
        billing_plan = paypal_plan
        if billing_plan.create && billing_plan.activate!
          self.paypal_id = billing_plan.id
        else
          self.errors[:paypal_id] << billing_plan.error[:details].collect{|e| e[:issue]}.join(",")
        end
      end
    end
  </code>
</pre>

- include ActAsPaypalPlan in BillingPlan model

#Creating Subscription

- Create `Subscription` model with required fields and `agreement_id`
- Create `act_as_subscription.rb` concern in `models/concerns` and paste below code.

<pre>
  <code>
    module ActAsSubscription
      STATES = {cancelled: "CANCELLED", active: "ACTIVE", suspended: "SUSPENDED"}
      extend ActiveSupport::Concern
      include PayPal::SDK::REST::DataTypes

      included do
        define_callbacks :cancel
        define_callbacks :reactivate
        define_callbacks :suspend
        after_create :active!
      end

      STATES.each do |k,v|
        define_method "#{k}?" do
          self.state == v
        end
        define_method "#{k}!" do
          update_column(:state, v)
        end
      end

      def agreement
        @agreement ||= AgreeMent.find(agreement_id)
      end

      def cancel
        return unless agreement_id
        run_callbacks :cancel do
          update_column(:state, STATES[:cancelled]) if agreement.cancel
        end
      end

      def suspend
        return unless agreement_id
        run_callbacks :suspend do
          update_column(:state, STATES[:suspended]) if agreement.suspend
        end
      end

      def reactivate
        return unless agreement_id
        run_callbacks :reactivate do
          update_column(:state, STATES[:active]) if agreement.reactivate
        end
      end
    end
  </code>
</pre>
- include ActAsSubscription in subscription model.
- Add following method in user model

<pre>
  <code>
    def create_subscription_with_payment_token!(options)
      agreement = PayPal::SDK::REST::DataTypes::AgreeMent.new
      if agreement.execute!(options[:token])
        subscription = build_subscription(billing_plan_id: options[:billing_plan_id], agreement_id: agreement.id)
        subscription.save
      else
        false
      end
    end
  </code>
</pre>

# Subscription controller
- For creating subscripton agreement which wont be successfull unles and untill you execute the agreement from paypal

<pre>
  <code>
    def approve
      @plan = BillingPlan.find_by_id(params[:plan_id])
      if @plan && @plan.agreement(cancel_url: paypal_cancel_url, return_url: paypal_return_url).create
        session[:plan_id] = @plan.id
        redirect_to @plan.agreement.approval_url
      else
        flash[:alert] = t("flash.error.subscription_approve_failure")
        redirect_to new_subscription_path
      end
    end
  </code>
</pre>
- It will redirect to paypal for authorization and approval.
- After approval paypal will redirect to your return url specified above.
- For execution of agreement

<pre>
  <code>
    def execute
      if session[:plan_id]
        @subscription = @user.create_subscription_with_payment_token!(token: params[:token], billing_plan_id: session[:plan_id])
        session[:plan_id] = nil
        flash[:notice] = t("flash.success.subscription.created")
        redirect_to after_create_path
      else
        redirect_to unauthorized_url
      end
    end
  </code>
</pre>

# After creation of subscription

- You can suspend subscription for sometime and reactivate again.
- But after cancellation, you can not reactivate, you have to subscribe again.

[Gist url](https://gist.github.com/jbmyid/16c70067ea3f5cea7d2b)
