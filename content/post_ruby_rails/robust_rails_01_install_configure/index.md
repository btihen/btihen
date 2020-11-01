---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Install and Configure Rails"
subtitle: ""
summary: ""
authors: ["btihen"]
tags: ["rails", "configure", "install", "durable", "testing"]
categories: ["Rails"]
date: 2020-09-10T01:46:07+02:00
lastmod: 2020-10-31T01:46:07+02:00
featured: true
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
# Intro

To document is mostly for me -- at least until I automate my setup defaults. However, I am glad to share and get ideas from others too.  I will build a little calendar app I use with friends (it's focused on being mobile friendly and easy to use -- not a full featured calendar).

# Rails Setup

Taken from:
* https://gist.github.com/alxndr/7569551
* https://www.codewithjason.com/rails-integration-tests-rspec-capybara/
* https://hackernoon.com/how-to-build-awesome-integration-tests-with-capybara-j9333y68

## create the project:
```
# -T - skips tests;              I like rspec
# -d postgresql;                 I like postgresql best for the db
# --skip-spring --skip-listen;   Spring caches and doesn't notice all changes (even after rails restart)
#                                I have lost several hours not realizing Spring wasn't seeing my changes

rails new calendar -T -d postgresql --webpack=stimulus --skip-spring --skip-listen

cd calendar

# in some cases you may have serveral bundlers or need to create binstubs
# gem install bundler:2.1.4
# rails app:update:bin
```

## update the README and initialize Git
```
git add .
git commit -m "intial commit"
git remote add origin git@gitlab.com:btihen/calendar.git
git push -u origin master
```

## Add extra Gems for this project
add rspec, devise, factory_bot and stimulus_reflex

Execute the following command (or add to the Gemfile)
```
cat <<EOF >> Gemfile
# Project Gems
##############

# FRONT END
###########
gem "stimulus_reflex", "~> 3.3"

# BACK END
##########
gem 'devise'

# DEV / TESTS
#############
group :development, :test do
  gem 'awesome_print'        # formats pry (& irb outputs into readable formats)

  gem 'pry-rails'
  gem 'pry-byebug'           # Adds byebug's step debugging and stack navigation
  # gem 'pry-debugger'       # adds step, continue, etc (alternative to pry-byebug)
  gem 'pry-stack_explorer'   # easy stack traces when debugging
  # more pry gems if needed at: https://spin.atomicobject.com/2012/08/06/live-and-let-pry/

  gem 'factory_bot_rails'
  gem 'faker'

  # gem 'rspec-rails'
  gem 'capybara'
  gem 'rspec-rails', '~> 4.0.0'

  # lets spring work with rspec
  gem 'spring-commands-rspec'
end

group :test do
  # easier tests (inside rspec)
  gem 'shoulda-matchers'

  # cucumber can test emails (rspec too?)
  # gem 'email_spec'

  # code coverage
  gem 'simplecov'
  gem 'simplecov-console'
end
EOF
```

## Now uncomment a few Gems in the Original Gemfile

Uncomment the following to ensure ActionText and Stimulus Refelx (work properly).

`gem 'image_processing', '~> 1.2'`

is needed by Active Storage (ActionText needs Active Storage)

and 

`gem 'redis', '~> 4.0'`

is needed by Stimulus Reflex (which uses Action Channels) to manage WebSockets

## Install and configure base gems

now run:

`bundle install`

to install all the new gems and create a `Gemfile.lock`

## JavaScipt packages to support new Gems.

`yarn add cable_ready stimulus_reflex`

## Now Install/Configure Stimulus Reflex:

```
bin/rails stimulus_reflex:install
# bin/rails dev:cache
```


## Install ActiveStorage and ActionText

run the following commands:
```
# bundle exec rails webpacker:install
# bundle exec rails webpacker:install:stimulus
bundle exec rails active_storage:install
bundle exec rails action_text:install
```

No configuration is needed for development (but is needed for production environments)

## RSPEC

### Install with:

`bin/rails g rspec:install`

## Configure:

### Create needed folders for our config

```
mkdir spec/features

# a place to put test helper code
mkdir spec/support
mkdir spec/support/features
```

### Rspec Config file `spec/rails_helper.rb`

1. To enable integration tests with rspec add: `require 'capybara/rspec'` below `require 'rspec/rails'` 
2. To load Test helper code add: `Dir[Rails.root.join("spec/support/**/*.rb")].each { |file| require file }` below `require 'capybara/rspec'`
3. just after the ActiveRecord config and before RSpec.configure block add:
```
Capybara.register_driver :selenium_chrome do |app|
  Capybara::Selenium::Driver.new(app, browser: :chrome)
end
Capybara.javascript_driver = :selenium_chrome
```
4. Add the FactoryBot config in the section with:
```
RSpec.configure do |config|
  # ...

  # support for Factory Bot
  config.include FactoryBot::Syntax::Methods

  # setup devise login helpers in Rspec
  config.include Devise::Test::IntegrationHelpers, type: :request

  # allows us for force session logouts (im feature tests)
  config.include Warden::Test::Helpers
end
```
5. finally at the end of the file add support for shoulda matchers with:
```
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
```

NOW `spec/rails_helper.rb` should look like (its long, sometimes the full context is clearer):
```
# This file is copied to spec/ when you run 'rails generate rspec:install'
require 'spec_helper'
ENV['RAILS_ENV'] ||= 'test'
require File.expand_path('../config/environment', __dir__)
# Prevent database truncation if the environment is production
abort("The Rails environment is running in production mode!") if Rails.env.production?
require 'rspec/rails'
# Add additional requires below this line. Rails is not loaded until this point!

# enables integration/feature tests using rspec
require 'capybara/rspec'

# loads custom helper test code
Dir[Rails.root.join("spec/support/**/*.rb")].each { |file| require file }
# or you could use:
# Dir[Rails.root.join('spec', 'support', '**', '*.rb')].sort.each { |f| require f }

# Checks for pending migrations and applies them before tests are run.
# If you are not using ActiveRecord, you can remove these lines.
begin
  ActiveRecord::Migration.maintain_test_schema!
rescue ActiveRecord::PendingMigrationError => e
  puts e.to_s.strip
  exit 1
end

# configure capybara integration tests
Capybara.register_driver :selenium_chrome do |app|
  Capybara::Selenium::Driver.new(app, browser: :chrome)
end
Capybara.javascript_driver = :selenium_chrome

RSpec.configure do |config|
  # Remove this line if you're not using ActiveRecord or ActiveRecord fixtures
  config.fixture_path = "#{::Rails.root}/spec/fixtures"

  # If you're not using ActiveRecord, or you'd prefer not to run each of your
  # examples within a transaction, remove the following line or assign false
  # instead of true.
  config.use_transactional_fixtures = true

  # You can uncomment this line to turn off ActiveRecord support entirely.
  # config.use_active_record = false

  # RSpec Rails can automatically mix in different behaviours to your tests
  # based on their file location, for example enabling you to call `get` and
  # `post` in specs under `spec/controllers`.
  #
  # You can disable this behaviour by removing the line below, and instead
  # explicitly tag your specs with their type, e.g.:
  #
  #     RSpec.describe UsersController, type: :controller do
  #       # ...
  #     end
  #
  # The different available types are documented in the features, such as in
  # https://relishapp.com/rspec/rspec-rails/docs
  config.infer_spec_type_from_file_location!

  # Filter lines from Rails gems in backtraces.
  config.filter_rails_from_backtrace!
  # arbitrary gems may also be filtered via:
  # config.filter_gems_from_backtrace("gem name")

  # support for Factory Bot
  config.include FactoryBot::Syntax::Methods

  # setup devise login helpers in Rspec (login helpers)
  config.include Devise::Test::IntegrationHelpers, type: :request

  # allows us for force session logouts (im feature tests)
  config.include Warden::Test::Helpers
end

Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
```

# Create / Test a landing page

A simple config test before we setup devise (authentication).

1. **Generate a page** -- I don't (generally) use helpers nor contoller or view specs - so I'll create the landing page using the following generator:
```
rails g controller Landing index --no-helper --no-assets --no-controller-specs --no-view-specs
```
2. **Update Routes** `config/routes.rb` with:
```
  get 'landing/index'
  root to: "landing#index"
```
3. **Add Hidden Test Content** to simplify testing add:
```
<p hidden id='landing_index'>Landing Index</p>
```
4. Request test:
```
# spec/requests/landing_request_spec.rb
require 'rails_helper'

RSpec.describe "Landings", type: :request do

  describe "GET /index" do
    it "returns http success" do
      get "/landing/index"
      expect(response).to have_http_status(:success)

      expect(response.body).to include("<p hidden id='landing_index'>Landing Index</p>")
    end
  end
end
```
5. Feature Test (to be sure they are working too)
```
# spec/features/landing_page_spec.rb
require 'rails_helper'

RSpec.describe 'Landing Page Works without a login', type: :feature do
  scenario 'Visit landing Page' do
    visit root_path

    page_tag = find('p#landing_index', text: 'Landing Index', visible: false)
    expect(page_tag).to be_truthy
  end
end
```
6. Create DB and Test
```
bin/rails db:create
bin/rails db:migrate
bundle exec rspec
```
7. Assuming test run and are green - we can commit a functioning setup:
```
git add .
git commit -m "rspec configured and working"
git push
```

If you plan to user database_cleaner -- then also see this article to finish your config:

https://medium.com/@amliving/my-rails-rspec-set-up-6451269847f9

a basic login feature test might look like:
```
require 'rails_helper'

RSpec.describe 'Users Login', type: :feature do
  let(:user)  { FactoryBot.create :user }
  after :each do
    Warden.test_reset!
  end
  describe 'user logs in successfully' do
    scenario 'and is redirected to user home page' do
      user_log_in(user)
      expect(current_path).to eql(auth_user_root_path)
    end
  end
end
```
