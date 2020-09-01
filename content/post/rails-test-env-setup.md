+++
title = "Setting up testing environment in rails"
description = ""
draft = false
tags = [
  "Rails",
]
date = "2020-03-15"
+++

Rails is a popular web application framework. You have this amazing idea and want to turn it into a web application? What framework do you choose? Of course Rails. We tend to quickly scaffold the application and write some code and bring the idea to life, But when it comes to testing the application, there is no scaffold command or anything that does the job within seconds. So we then spend hours trying to set up a testing environment and then later write tests.

The intention of my article is to show how to quickly set up a testing environment to avoid future hassle. Sound good? Let’s begin.

<b>NOTE</b>: This article assumes you are familiar with Rails, so we won’t dive deep into the topic just read the documentation for each and every dependency used and you will be good to go. This guide won’t be showing actual tests in code.

### Generate a new rails app

```shell
$ rails new test_app --tests
```

---tests flag tells Rails to exclude the default Minitest framework because we will use RSpec instead.

### The tools we will need

- rspec-rails  — Testing framework.
- factory_bot_rails  — A fixtures replacement with a more straightforward syntax. You’ll see.
- shoulda_matchers  — Provides RSpec with additional matchers.
- database_cleaner  — You guessed it! It literally cleans our test database to ensure a clean state in each test suite.
- faker — A library for generating fake data. We’ll use this to generate test data.
- simplecov — A code coverage analysis tool for Ruby.

### The work begins

Let’s set them up. In your Gemfile:

```ruby
group :development, :test do
  gem 'database_cleaner'
  gem 'factory_bot_rails', '~> 4.0'
  gem 'faker', '~> 2.4.0'
  gem 'pry-rails'
  gem 'rspec-rails'
  gem 'shoulda-matchers', '~> 3.1'
  gem 'simplecov', require: false
end
```

Do a bundle install

Initialize the spec directory (This is where the tests file will reside)

```shell
$ rails generate rspec:install
```

This quickly adds the following files which are used for configuration:

- .rspec
- spec/spec_helper.rb
- spec/rails_helper.rb

Create a factories directory in the spec folder (factory bot uses this as the default directory). This is where you define the test data.

```shell
$ mkdir spec/factories
```

### Configuration

In spec/rails_helper.rb

```ruby
# add below require 'spec_helper'

require 'rspec/rails'
require 'support/factory_bot'
require 'simplecov'
require 'database_cleaner'
SimpleCov.profiles.define 'no_coverage' do
  load_profile 'rails'
  add_group 'Routes', 'config/routes'
  add_filter 'vendor'
  add_filter 'app/channels'
  add_filter 'app/mailers'
  add_filter 'app/helpers'
end
SimpleCov.start 'no_coverage'

# Add below the do block

RSpec.configure do |config|
  config.use_transactional_fixtures = false
  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
    DatabaseCleaner.strategy = :transaction
end

  config.around(:each) do |example|
    DatabaseCleaner.cleaning do
      example.run
    end
  end
end

# Add after the do block

Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
```

Create a support folder inside the spec directory

```shell
$ mkdir spec/support
```

Create a file factory_bot.rb inside the support directory and add the following to the file

```ruby
# spec/support/factory_bot.rb

RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
```

And that’s a wrap. We have successfully added the testing stack to our Rails app.
