---
layout: post
title: "Setup Twitter Bootstrap on Rails"
quote: "Setting up bootstrap on rails with assets precompiling fix."
image: /media/2014-04-15-bootstrap-on-rails/twitter_bootstrap-wallpaper-1024x768.jpg
video: false
comments: true
---

#"Setup Twitter Bootstrap on Rails"

####Here are the steps which will be needed to make bootstrap working on rails app.

1. Download bootstap from [here](http://getbootstrap.com/getting-started/#download)
2. Extract the zip file and copy assets folder content to `yourApp/vendor/assets/`
3. Now you will have following folder in `vendor/assets/` directory
  * fonts
  * stylesheets
  * javascripts
4. Create `bootstrap_overrides.css.scss` in `yourApp/app/assets/stylesheets/` directory and add following line at the top.
  * `@import "bootstrap";`
5. Rename your `application.css` file to `application.css.scss`
6. Add `*= require bootstrap_overrides` to your `application.css.scss`
7. Add following code to your `bootstrap_overrides.css.scss` file
  *
    <pre>
      <code class='ruby'>
        @font-face {
          font-family: 'Glyphicons Halflings';
          src: url(asset_path('glyphicons-halflings-regular.eot'));
          src: url(asset_path('glyphicons-halflings-regular.eot?#iefix')) format('embedded-opentype'), url(asset_path('glyphicons-halflings-regular.woff')) format('woff'), url(asset_path('glyphicons-halflings-regular.ttf')) format('truetype'), url(asset_path('glyphicons-halflings-regular.svg#glyphicons_halflingsregular')) format('svg');
        }
      </code>
    </pre>
    <div class="message">
      Above code is to fix the glyphicons issue in asset pipelining (in production).
    </div>
8. Add `config.assets.paths << "#{Rails}/vendor/assets/fonts"` in your `application.rb`
9. Add `//= require bootstrap.min` in your application.js if you want to include bootstrap js.

####If even after doing all above steps the glyphicons doesnt work then

* move the fonts to `yourApp/app/assets/fonts/`. And change fonts path in `application.rb`
<br>i.e.  `config.assets.paths << "#{Rails}/assets/fonts"`

  <div class="message">
  You are welcome :P
  </div>
