## Checklist before you start

 - Global gitignore in your home directory `~/.gitignore_global` .ie this:  https://github.com/rubynor/rails-setup/blob/master/.gitignore_global


### Our prefered rails setup

1. **Generate the app with vue and webpacker**

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

2. **Initial commit and prep work**

```
cp config/database.yml config/database.yml.sample.generated
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/database.yml.sample > config/database.yml.sample
cp config/database.yml.sample config/database.yml

git diff # double check that database.yml is ignored.
rake db:create db:migrate # don't proceed until this works.

git add .
git commit -m "add database config"

```    


3. **Copy the gemfile and bundle install. Here is our recomendation:** 
 https://github.com/rubynor/rails-setup/blob/master/Gemfile

``` 
#Append the recommend Gemfile gems to your gemfile
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/Gemfile >> Gemfile
#check if this makes sense, you might want to move things around, especially to ensure only one group :development, :test 
git diff 
bundle install
git commit -am "install recommended gems"
```    

4. **Run the following commands**

```
rails generate haml:application_layout convert
rm app/views/layouts/application.html.erb # as the above script instructs you to do manually
rails generate rspec:install
git add .
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.rspec > .rspec
git diff
git commit -am "install haml and rspec"
```

5. **Environment variables**

We use `dotenv-rails` to manage environment variables.
```
# The .env.sample will be in source control. Keep it up-to-date when adding new ENV vars to your .env, if you love your fellow developers
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.env.sample > .env.sample
# .env for the ENVs to be loaded locally
cp .env.sample .env

git add . && git commit -am "add dot-env"
```



6. **Foreman for running webpack server and rails server together**

```
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/Procfile.dev > Procfile.dev
curl -sS https://raw.githubusercontent.com/rubynor/rails-setup/master/.foreman > .foreman

#To run
foreman start

git add .
git commit -m "add foreman"
```

7. **Devise for handling authentication**

```
rails generate devise:install
```

At this point, a number of instructions will appear in the console. Among these instructions, you'll need to set up the default URL options for the Devise mailer in each environment. Here is a possible configuration for `config/environments/development.rb`:

```
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

Generate your user model or whatever you want to call it

```
rails generate devise User email:string name:string
```

Override devise views, so they can be customized

```
rails generate devise:views
```

You should restart your application after changing Devise's configuration options (this includes stopping spring). Otherwise, you will run into strange errors, for example, users being unable to login and route helpers being undefined.

8. **Autotesting using Guard**

To set up guard you have to generate a Guardfile.

```
bundle exec guard init rspec
```

Guard has been added to `Procfile.dev`, and can be started, with 
```
foreman start
```

If you prefer to run guard in a seperate window, remove it from `Procfile.dev` and run one of the following command:
```
guard -c
``` 

9. **Background jobs. Sidekiq**


## BEST PRACTICES Config

application.rb :

```
    # Raise on unpermitted parameters. Silent failures are bad
    config.action_controller.action_on_unpermitted_parameters = :raise
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





### Capybara
In your test helper file you have to add the following
```
require 'capybara/rails'
```
