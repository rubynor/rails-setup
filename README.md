<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Checklist before you start](#checklist-before-you-start)
- [Stepwise Guide for a Good Rails Setup](#stepwise-guide-for-a-good-rails-setup)
  - [1. Generate the app with vue and webpacker](#1-generate-the-app-with-vue-and-webpacker)
  - [2. Initial commit and prep work](#2-initial-commit-and-prep-work)
  - [3. Copy the gemfile and bundle install. Here is our recomendation:](#3-copy-the-gemfile-and-bundle-install-here-is-our-recomendation)
  - [4. Run the following commands](#4-run-the-following-commands)
  - [5. Environment variables](#5-environment-variables)
  - [6. Autotesting using Guard](#6-autotesting-using-guard)
  - [7. Foreman for running webpack server and rails server together](#7-foreman-for-running-webpack-server-and-rails-server-together)
  - [8. Optional. A scaffold that doesn't suck](#8-optional-a-scaffold-that-doesnt-suck)
  - [9. Devise for handling authentication](#9-devise-for-handling-authentication)
  - [10. Background jobs. Sidekiq](#10-background-jobs-sidekiq)
- [ADVICE](#advice)
  - [Configs](#configs)
- [MONTHLY ROUTINE](#monthly-routine)
  - [DATABASE  - lol_dba](#database----lol_dba)
  - [DATABASE - bullet](#database---bullet)
- [PRODUCTION TOOLS](#production-tools)
  - [Error Catcher. Honeybadger](#error-catcher-honeybadger)
  - [Logging. Papertrail](#logging-papertrail)
- [TESTING](#testing)
  - [Testing in parallel:](#testing-in-parallel)
  - [Capybara](#capybara)
- [CODESTYLE and CODE QUALITY](#codestyle-and-code-quality)
  - [Codeclimate](#codeclimate)
- [Performance](#performance)
- [Contribute](#contribute)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Checklist before you start

 - Global gitignore in your home directory `~/.gitignore_global` .ie this:  https://github.com/rubynor/rails-setup/blob/master/.gitignore_global


## Stepwise Guide for a Good Rails Setup

### 1. Generate the app with vue and webpacker

```
rvm use your-version # 2.5.2 or whatever version the latest is. RVM not required ofc, use rbenv or anything of your choice. It will put this ruby version on top of your Gemfile. 

gem install rails # Get latest stable release
rails new the-rubynor-app --webpack=vue --skip-test --database=postgresql
cd the-rubynor-app
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.gitignore-additions >> .gitignore
git init .
git status # check for junk files, edit your .gitignore to exclude
git add .
git commit -am "ran rails new the-rubynor-app --webpack=vue --skip-test --database=postgresql"
```

### 2. Initial commit and prep work

```
cp config/database.yml config/database.yml.sample.generated
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/database.yml.sample > config/database.yml.sample
cp config/database.yml.sample config/database.yml

git diff # double check that database.yml is ignored.
rake db:create db:migrate # don't proceed until this works.

git add .
git commit -m "add database config"

```    


### 3. Copy the gemfile and bundle install. Here is our recomendation:
 https://github.com/rubynor/rails-setup/blob/master/Gemfile

``` 
#Append the recommend Gemfile gems to your gemfile
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/Gemfile >> Gemfile
#check if this makes sense, you might want to move things around, especially to ensure only one group :development, :test 
git diff 
bundle install
git commit -am "install recommended gems"
```    

### 4. Run the following commands

```
rails haml:erb2haml
rm app/views/layouts/application.html.erb # as the above script instructs you to do manually
rails generate rspec:install
git add .
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.rspec > .rspec
git diff
git commit -am "install haml and rspec"
```

### 5. Environment variables

We use `dotenv-rails` to manage environment variables.
```
# The .env.sample will be in source control. Keep it up-to-date when adding new ENV vars to your .env, if you love your fellow developers
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.env.sample > .env.sample
# .env for the ENVs to be loaded locally
cp .env.sample .env

git add . && git commit -am "add dot-env"
```


### 6. Autotesting using Guard

To set up guard you have to generate a Guardfile.

```
bundle exec guard init rspec
```

Edit Guardfile and change to `guard :rspec, cmd: "rspec --tty"`

In next section, you'll see how to autostart Guard with foreman, as guard has been added to `Procfile.dev`. Too cool!



### 7. Foreman for running webpack server and rails server together

```
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/Procfile.dev > Procfile.dev
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.foreman > .foreman

#To run
foreman start

# Ceck it! Browse to 
# localhost:5000

git add .
git commit -m "add foreman"
```

### 8. Optional. A scaffold that doesn't suck

```
rails generate scaffold sausage length:integer name:string price:float --no-assets --no-helper --no-jbuilder --no-view-specs

#To reverse
rails destroy scaffold sausage length:integer name:string price:float --no-assets --no-helper --no-jbuilder --no-view-specs

```

### 9. Devise for handling authentication

We seem to always fall back to devise <3

  1. Install
```
rails generate devise:install
```

  2. Follow the instructions, including step 4 to copy the views.

  3. Then generate your user model

```
rails generate devise User email:string name:string
```

  4. Activate the Authentication regime

```
# A: Global authenticate? Edit your application_controller.rb

class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  before_action :authenticate_user!
end
```

  

```
# B: Public pages? Set skip authenticate

class PagesController < ApplicationController
  skip_before_action :authenticate_user!, only: [:home]
  def home
  end
end
```

5. Test. Restart app


6. Enable `confirmable`, `lockable` and perhaps `trackable`.  This is tedious but important work. See how at https://github.com/plataformatec/devise

7. Test that it works before proceeding, dummy!

Note: You should restart your application after changing Devise's configuration options (this includes stopping spring). Otherwise, you will run into strange errors, for example, users being unable to login and route helpers being undefined.


### 10. Background jobs. Sidekiq

You must have redis installed to use sidekiq. By default, Sidekiq tries to connect to Redis at localhost:6379.

Installing redis is different depending on which OS you are using. Please refer to instructions for your system:

- [--> Mac OS](https://medium.com/@petehouston/install-and-config-redis-on-mac-os-x-via-homebrew-eb8df9a4f298)
- [--> Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04)

To use sidekiq to handle background jobs with Active Job, check out instructions [here](https://github.com/mperham/sidekiq/wiki/Active+Job)

## ADVICE

### Configs
application.rb :

```
    # Raise on unpermitted parameters. Silent failures are bad
    config.action_controller.action_on_unpermitted_parameters = :raise
```

## MONTHLY ROUTINE

### DATABASE  - lol_dba

In the Gemfile, we included this:
```
gem 'lol_dba' # 'rake db:find_indexes' to get hints about missing indexes. https://github.com/plentz/lol_dba
```

### DATABASE - bullet

In the Gemfile, we also added bullet

```
gem 'bullet' # set env BULLET = true to activate. Bullet help to kill N+1 queries and unused eager loading
```

It's not ready to run, you can do both or one of these:

**A: Active for dev env. It'll show warnings when you browse the app locally.***

Follow the instructions. Yep, boring, but you learn more this way: https://github.com/flyerhzm/bullet#configuration

**B: Activate in test.**

```
#test.rb
config.after_initialize do
  Bullet.enable = ENV['BULLET'] == "true"
  Bullet.bullet_logger = true
  Bullet.raise = true # raise an error if n+1 query occurs
end

#spec_helper.rb or rails_helper.rb

  # Bullet helps discover n+1 and unused eager_loading
  if Bullet.enable?
    config.before(:each) do
      Bullet.start_request
    end

    config.after(:each) do
      Bullet.perform_out_of_channel_notifications if Bullet.notification?
      Bullet.end_request
    end
  end

$ BULLET=true rspec spec
```

## PRODUCTION TOOLS

 -1 Error catching tool. Honeybadger is suggested, see below
 -1 Logs. Papertrail, see below
 -1 Performance monitoring. Ie. New Relic. See below


### Error Catcher. Honeybadger
Go to https://app.honeybadger.io/projects/new

bundle exec honeybadger install [secret]

### Logging. Papertrail

Papertrail is not a server log, but a logger to data changes.

```
class MyModel < ApplicationRecord
  has_paper_trail
  ...
end
 
```
You can and should exclude unimportant fields. Read more 

## TESTING

We like RSpec with Fabricator, and RSpec with Capybara for browser integration specs (aka feature specs). Don't make too many and don't make too few, but most importantly, don't make to many feature specs, they come at a high cost (maintanence, feedback delays because of slow tests (MUDA))

### Testing in parallel:
Use parallel_tests when your test suite becomes slow over time. 


Copy `.rspec_parallel` to your repo. 

```
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.rspec_parallel > .rspec_parallel
```

Follow the gem's [documentation](https://github.com/grosser/parallel_tests) on how to set it up. 

You will find an example [database.yml.sample](https://github.com/rubynor/rails-setup/blob/master/database.yml.sample) in this repo.


To run specs in parallel:

```
rails parallel:spec
```

### Capybara
In your test helper file you have to add the following
```
require 'capybara/rails'
```

## CODESTYLE and CODE QUALITY

```
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.codeclimate.yml > .codeclimate.yml
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.rubocop.yml > .rubocop.yml
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.reek.yml > .reek.yml


git add . 
git commit -m "Add rubocop, reek and codeclimate config"

# the gems rubocop and overcommit are already in the Gemfile above.
overcommit --install

# TEST it. my advice is going into user.rb and creating a method with poorly formatted code and see if you are prevented from committing.

git add .
# if you don't have changes, that's ok
git commit -m "rubocop overcommit" 

```

### Codeclimate

Enable codeclimate for your github repository. Have fun challenging yourself :)

## Performance

Use jemalloc to greatly reduce memory usage. https://medium.com/rubyinside/how-we-halved-our-memory-consumption-in-rails-with-jemalloc-86afa4e54aa3

On heroku, add buildpack (https://github.com/gaffneyc/heroku-buildpack-jemalloc):

heroku buildpacks:add --index 1 https://github.com/gaffneyc/heroku-buildpack-jemalloc.git -r staging
heroku config:set JEMALLOC_ENABLED=true -r staging

In Procfile

```
web: jemalloc.sh bundle exec puma -C config/puma.rb
worker: jemalloc.sh bundle exec sidekiq -q sidekiq_queue -c 5 -v
``` 



## Contribute

Externals: Create Issue with proposal.
Internals: Direct edit on Github master is OK.

If made changes to titles, run DocToc to update Table of Content

```
git pull
doctoc .
git commit -am "doctoc"
git push
``
