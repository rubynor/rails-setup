gem 'haml-rails'     # Use haml instead of erb
gem 'pry-rails'      # Pretty print in rails console and set debug breakpoints in server console
gem 'honeybadger'    # honeybadger error reporting tool
gem 'devise'         # Devise is a flexible authentication solution for Rails based on Warden
gem 'sidekiq'        # Background jobs

group :development, :test do
  gem 'rspec-rails'  # Testing framework
  gem 'fabrication'  # Test subject builder for the specs
  gem 'faker'        # Create dummy data for the test objects
  gem 'guard-rspec', require: false # automatically run tests on changes
  gem 'dotenv-rails' # Handle env vars
  gem 'capybara'     # Write tests that simulate how your users will use your application
  gem 'foreman'      # To run app and webpack together
  gem 'rubocop'      # code style rules
  gem 'overcommit'   # Run Rubucop rules when committing
  gem 'parallel_tests' # rails parallel:spec to run specs in parallel
  gem 'bullet'       # Set env BULLET = true to activate. Bullet help to kill N+1 queries and unused eager loading
end

group :development do
  gem 'lol_dba' # 'rake db:find_indexes' to get hints about missing indexes. https://github.com/plentz/lol_dba
end
