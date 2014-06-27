---
layout: post
title: "Push Notification using AWS SNS on rails"
quote: "Sending Push Notifications from ruby/rails application using Amazon SNS Service"
image: false
logo: /media/2014-06-27-aws-sns-gcm-push-notifications/amazon-logo.png
video: false
comments: true
---
#Push Notification using AWS SNS on rails

Prerequisites.

1. AWS account

2. Note Down the Amazon access_key_id, secret_access_key and region

3. Create an Application in Amazon/SNS Section

4. Note Down the Application Platform ARN

5. Set ENV variables:
  <pre>
    <code class="ruby">
      ENV['SNS_APP_ARN'] = the arn obtained from 4th step
      ENV['AWS_ACCESS_KEY_ID'] = obtained from step 2
      ENV['AWS_SECRET_ACCESS_KEY']= obtained from step 2
    </code>
  </pre>


#Getting Started with 'aws-sdk' gem

- Add gem 'aws-sdk'
- bundle install
- Set configuration for aws
  - Create aws_config.rb in config/initializers directory and add following
    <pre>AWS.config(access_key_id: ENV['AWS_ACCESS_KEY_ID'], secret_access_key: ENV['AWS_SECRET_ACCESS_KEY'], region: "region_code")</pre>


#Using AWS SDK

- Create an sns object
  <pre>
    sns = AWS::SNS.new
  </pre>
- Set sns client
  <pre>client = sns.client</pre>
- Whenever device registers on your site get GCM registration id and store it in your database, and every time the gmc registration id changes subscribe it in SNS
  <pre>response = client.create_platform_endpoint token: gcm_registraion_id, platform_application_arn: ENV['SNS_APP_ARN']</pre>
  - the response will contain the end_point_arn for the device registered, which will be used in sending notifications
  <pre>device_arn = response[:endpoint_arn] # save it</pre>

#Publishing Notification:
<pre>
  gcm_message = {collapse_key: "any_collapse_key", data: {key: "Value"}}
  message = {"GCM"=> gcm_message.to_json}.to_json
</pre>
<pre>
  response = client.publish target_arn: device_arn, message: message , message_structure: 'json'
</pre>
<div class="message">Notification Sent!</div>

Whenever gcm registration id changes remove the existing target_arn and add new one
To remove the existing target arn
<pre>
  client.delete_endpoint(endpoint_arn: device_arn)
</pre>

You can broad cast the notification by creating topics and subscribing the arn to the particular topic and publish specifying topic_arn

e.g.
<pre>
  response = client.publish topic_arn: topic_arn, message: message , message_structure: 'json'
</pre>

#THANK YOU
