## Checklist before you start

 - Global gitignore in your home directory `~/.gitignore_global` .ie this:  


### Our prefered rails setup

1. Generate the app with vue and webpacker
`rails new rubynor-app --webpack=vue --skip-test --database=postgresql`

2. Add the gems from Gemfile that you want to use
`bundle install`

3. Run the following commands


`rails generate haml:application_layout convert`

`rails generate rspec:install`

`bundle exec guard init rspec`

### Honeybadger
Go to https://app.honeybadger.io/projects/new

bundle exec honeybadger install [secret]
