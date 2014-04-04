---
layout: post
title: "Generate PDF(e-ticket) from html in Rails"
quote: "In most of the projects we get requirements in which we need to send an attachment or download PDFs for order’s e-ticket or any certificate kind of things. So here I have written a post  for how to create PDFs from html using “pdfkit” gem."
image: /media/2014-02-27-hello-cosette/cover.jpg
video: false
comments: true
---
#PDF(e-ticket) from html in Rails

Following are the steps for creating pdf from html views.

- Add gem “pdfkit” in gemfile
- Download following 2 files
  - wkhtmltopdf-amd64 from [here](http://code.google.com/p/wkhtmltopdf/downloads/detail?name=wkhtmltopdf-0.9.9-static-amd64.tar.bz2&can=2&q=)
  - wkhtmltopdf-i386 from [here]( http://code.google.com/p/wkhtmltopdf/downloads/detail?name=wkhtmltopdf-0.9.9-static-i386.tar.bz2&can=2&q=)
- Create “bin” folder in your rails application folder and extract above files in this “bin” folder. These are the binaries which are needed for creating  pdf from html views.
- Create “pdfkit.rb” in “config/initializers” folder of the application. And add following code
<pre>
<code class="ruby">
  #pdfkit.rb
  PDFKit.configure do |config|
    if ["development"].include?(Rails.env)
      #only if your are working on 32bit machine
      config.wkhtmltopdf = Rails.root.join('bin', 'wkhtmltopdf-i386').to_s
    else
      #if your site is hosted on heroku or any other hosting server which is 64bit
      config.wkhtmltopdf = Rails.root.join('bin', 'wkhtmltopdf-amd64').to_s
    end
    config.default_options = {
      :encoding=>"UTF-8",
      :page_size=>"A4",
      :margin_top=>"0.25in",
      :margin_right=>"0.1in",
      :margin_bottom=>"0.25in",
      :margin_left=>"0.1in",
      :disable_smart_shrinking=> false
    }
  end
  </code>
</pre>
- Create a file e_ticket.rb in models folder as following
<pre>
  <code class="ruby">
    # e-ticket
    class ETicket < AbstractController::Base
      include AbstractController::Rendering
      include AbstractController::Helpers
      include AbstractController::Translation
      include AbstractController::AssetPaths
      include Rails.application.routes.url_helpers
      helper ApplicationHelper
      self.view_paths = "app/views"
      attr_reader :html
      def initialize(order)
        @order = order
        @html = render_to_string(partial: "shared/e_ticket", :layout => false,
             :disposition => 'inline')
      end
      def pdf
        kit = PDFKit.new(@html)
        kit.stylesheets << "#{Rails.root}/app/assets/stylesheets/pdf.css"
        kit.to_pdf
      end
    end
  </code>
</pre>

In above code, we have added style sheet “pdf.css” in which we have to write the pdf related css rules. Keep this file as separate for pdf only .
Now you have your e-ticket ready to download and attach to any mail. You have to just create e-ticket by
<pre>
  <code class="ruby">
    eticket = ETicket.new(order)
    pdf = eticket.pdf
  </code>
</pre>

Yehhhhhh……………..its ready to download and attach.

Now I am writing brief for downloading and attaching the pdf to mail.

### How to download eticket pdf.
In your orders controller you can create an separate action for downloading or you can do this in your orders show action in pdf response. Any way I am creating separate action for downloading order’s e-ticket pdf.
<pre>
  <code class="ruby">
    class OrdersController < ApplicationController
      def download
        @order = Order.find(params[:id])
        eticket = ETicket.new(@order)
        send_data(eticket.pdf, :filename => "e-Ticket #{@order.number}",
           :type => 'application/pdf')
      end
    end
  </code>
</pre>

###Sending e-ticket as attachment in mail
<pre>
  <code class="ruby">
    class OrderMailer < ActionMailer::Base
      def send_e_ticket(order)
        @order = order
        subject = " E-Ticket"
        eticket = ETicket.new(@order)
        attachments["E-Ticket #{@order.number}.pdf"] = eticket.pdf
        mail(:to =>”abc@gmail.com”, :subject =>”E-ticket”)
      end
    end
  </code>
</pre>
Now send an mail using following code
<pre>
  <code class="ruby">
    OrderMailer.send_e_ticket(order).deliver
  </code>
</pre>

yippeeeeee…..Mail sent with attached e-ticket pdf.

#THANK YOU
