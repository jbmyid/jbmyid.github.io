---
layout: post
title: "Quick Rails Deployment with Capistrano"
quote: "Deployment of Rails 4 app with Capistrano 3"
image: false
video: false
comments: true
---

#Deploying Rails 4 App with Capistrano 3

Assuming we have a rails app as my_blog
add following in your Gemfile

<pre>
group :development do
  gem 'capistrano'
end
</pre>
now bundle install

`$ bundle install`

Capify: make sure there's no "Capfile" or "capfile" present

`$ cap install`

This creates the following files:
<pre>
├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb
│   │   └── staging.rb
│   └── deploy.rb
└── lib
    └── capistrano
            └── tasks
</pre>
Open deploy.rb and set
<pre>
<code class='ruby'>
set :application, 'application-name'
set :repo_url, 'git@github.com:abc/xyz.git' # your git repo
set :deploy_to, '/var/www/my_blog' # where you have to keep your application on server

set :linked_files, %w{config/database.yml config/any_config.yml tmp/restart.txt}
set :linked_dirs, %w{log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

set :default_env, { path: "/usr/local/bin:$PATH" }

namespace :deploy do

  desc 'Restart application'
  task :restart do
    on roles(:app), in: :sequence, wait: 5 do
      # Your restart mechanism here, for example:
      execute :touch, release_path.join('tmp/restart.txt')
    end
  end

  after :publishing, :restart

  after :restart, :clear_cache do
    on roles(:web), in: :groups, limit: 3, wait: 10 do
      # Here we can do anything such as:
      # within release_path do
      #   execute :rake, 'cache:clear'
      # end
    end
  end

end
</code></pre>

Now setup stage for deploy
eg. staging.rb #in config/deploy/staging.rb
<pre>
<code class='ruby'>
role :app, %w{deploy@ip-addressORserver-address}
role :web, %w{deploy@ip-addressORserver-address}

server 'ip-addressORserver-address', user: 'deploy', roles: %w{web app}

set :branch, 'staging' # staging branch of git repo(You can set whatever you want)

# in my case i had to set port no for ssh you can omit if you are using default ssh port no.
set :ssh_options, {
    port: 900
 }
</code>
</pre>

Now you can deploy your app with.

`$ cap staging deploy`

Wait.. its not done yet you will have to add some task for each deployment you have to do like migration, asset precompile etc.

For that we have capistrano-rails
add `gem 'capistrano-rails'` inside development group of Gemfile
like:
<pre>
<code class='ruby'>
group :development do
  gem 'capistrano'
  gem 'capistrano-rails
end
</code>
</pre>

add following line in `Capfile`
<pre>
require 'capistrano/rails'
</pre>
It will run basic task required while rails application deployment like `rake db:migrate` and `rake assets:precompile`

<strong>Yeh you are done with basic setup of deplyment.</strong>


