## Checklist before you start

 - Global gitignore in your home directory `~/.gitignore_global` .ie this:  https://github.com/rubynor/rails-setup/blob/master/.gitignore_global


### Our prefered rails setup

1. Generate the app with vue and webpacker

```
rails new the-rubynor-app --webpack=vue --skip-test --database=postgresql
```

2. do some initial clean up

```
cd the-rubynor-app
cp config/database.yml config/database.yml.sample
git init .
git diff # double check that database.yml is ignored. Did you put the .gitignore_global in your home dir?
git commit -am "ran rails new"
```    


2. Copy the gemfile and bundle install. Here is our recomendation: 
 https://github.com/rubynor/rails-setup/blob/master/Gemfile

``` 
cp https://github.com/rubynor/rails-setup/blob/master/Gemfile .
#check if this makes sense. Your ruby or rails versions might be different etc, that shouldn't be overridden
git diff 
bundle install
git commit -am "install recommended gems"
```    

3. Run the following commands

```
rails generate haml:application_layout convert
rails generate rspec:install
bundle exec guard init rspec
```

### Honeybadger
Go to https://app.honeybadger.io/projects/new

bundle exec honeybadger install [secret]

### Environment variables
We use `dotenv-rails` to manage environment variables.

Create a file named `.env` and put them there. Remember to add a .env.sample that is added to the repo
