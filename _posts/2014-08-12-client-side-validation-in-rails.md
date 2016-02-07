---
layout: post
title: "Client Side Validation in Rails"
quote: "Client side validation in rails using gem rails_jq_validations."
image: /media/2014-08-12-client-side-validation-in-rails/ruby.jpg
video: false
comments: true
---
#Setup Gem

How to setup application to use rails_jq_validations gem.

- Add gem “rails_jq_validations” in gemfile
  - <pre>gem "rails_jq_validations", git: "git@github.com:jbmyid/rails_jq_validations.git"</pre>
- Run `bundle install`
- Add js to your application.js
  - <pre> //= require rails_jq_validations/form_validator </pre>

#Adding Client side validation to form
- Considering you are having `Product` model and you have added `validates :name, presence: true` to your model.

- Add  `data: {validate: true}` to your `form_for` heleper like following

<pre>
<code class="ruby">
  #_form.html.erb
  <%= form_for @product, data: {validate: true} do |f| %>
    <%= f.text_field :name %>

    <%= f.submit %>
  <% end %>
</code>
</pre>
Will give you output as follow(message which is set in en.yml for your models)

{% include image.html url="/media/2014-08-12-client-side-validation-in-rails/validation.png" width="auto" %}


It uses jquery [validation plugin](http://jqueryvalidation.org/), so you can add any additional specific rule to particular field. Like following.
<pre>
<%= f.text_field :name, data:{"jq-rules"=> {required: true, number: true, messages: {required: "Please enter name."}}.to_json}%>
</pre>

<div class="message">Note: styling error messages are upto you. You can set default error message placement with validation plugin.</div>

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- First Ad -->
<ins class="adsbygoogle"
     style="display:inline-block;width:728px;height:90px"
     data-ad-client="ca-pub-8459710473289765"
     data-ad-slot="8694625139"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
