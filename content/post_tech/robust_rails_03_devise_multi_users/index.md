---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Durable_rails_users"
subtitle: ""
summary: ""
authors: ["btihen"]
tags: []
categories: []
date: 2020-07-10T20:45:51+02:00
lastmod: 2020-07-10T20:45:51+02:00
featured: false
draft: true

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

## Configure devise (for multiple types of accounts)

install the devise engine:
```
bin/rails generate devise:install
```

now follow the basic setup config -- add to `config/environments/development.rb`
```
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

add notifications to the layout for devise in `app/views/layouts/application.html.erb` just above `<%= yeild %>`

```
<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

now create one or more models for devise:
```
rails generate devise user
rails generate devise admin
```

update the routes to put the login in separate routes in `config/routes.rb` - make the routes look like:
```
  devise_for :users,  path: 'users'  # http://localhost:3000/users/sign_in
  devise_for :admins, path: 'admins' # http://localhost:3000/admins/sign_in
```

turn on scoped views (since login forms can be different) in `config/initializers/devise.rb`
```
config.scoped_views = true
```
Create the scoped views: (instead of: rails g devise:views) do:
```
rails g devise:views users/devise
rails g devise:views admins/devise
```

now we should open these migrations and uncomment any added fields we use - I generally like to use most of the fields:
```
# frozen_string_literal: true

class DeviseCreateAdmins < ActiveRecord::Migration[6.0]
  def change
    create_table :admins do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      t.integer  :sign_in_count, default: 0, null: false
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.inet     :current_sign_in_ip
      t.inet     :last_sign_in_ip

      ## Confirmable
      t.string   :confirmation_token
      t.datetime :confirmed_at
      t.datetime :confirmation_sent_at
      t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      t.string   :unlock_token # Only if unlock strategy is :email or :both
      t.datetime :locked_at

      t.timestamps null: false
    end

    add_index :admins, :email,                unique: true
    add_index :admins, :reset_password_token, unique: true
    add_index :admins, :confirmation_token,   unique: true
    add_index :admins, :unlock_token,         unique: true
  end
end
```

and adjust the `user` and `admin` models too and turn on the features we want or need. We will go into detail later, for now I will just add trackable to the models:

```
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable, :trackable,
         :recoverable, :rememberable, :validatable
end
```


and of course migrate too.
```
bin/rails db:migrate
```

Create custome controllers for each sessions - this also allows the users to have different fields and features:
```
rails generate devise:controllers users/devise
rails generate devise:controllers admins/devise
```

configure the routes to point to these new controllers:
```
  # http://localhost:3000/users/sign_in
  devise_for :users,  path: 'users',
                      controllers: {
                        sessions:      'users/devise/sessions',
                        passwords:     'users/devise/passwords',
                        registrations: 'users/devise/registrations'
                      }
  # http://localhost:3000/admins/sign_in
  devise_for :admins, path: 'admins',
                      controllers: {
                        sessions:      'admins/devise/sessions',
                        passwords:     'admins/devise/passwords',
                        registrations: 'admins/devise/registrations'
                      }
```

now the routes should look like:
```
$ bin/rails routes
                     Prefix Verb   URI Pattern                        Controller#Action
           new_user_session GET    /users/sign_in(.:format)           users/sessions#new
               user_session POST   /users/sign_in(.:format)           users/sessions#create
       destroy_user_session DELETE /users/sign_out(.:format)          users/sessions#destroy
          new_user_password GET    /users/password/new(.:format)      users/passwords#new
         edit_user_password GET    /users/password/edit(.:format)     users/passwords#edit
              user_password PATCH  /users/password(.:format)          users/passwords#update
                            PUT    /users/password(.:format)          users/passwords#update
                            POST   /users/password(.:format)          users/passwords#create
   cancel_user_registration GET    /users/cancel(.:format)            user/registrations#cancel
      new_user_registration GET    /users/sign_up(.:format)           user/registrations#new
     edit_user_registration GET    /users/edit(.:format)              user/registrations#edit
          user_registration PATCH  /users(.:format)                   user/registrations#update
                            PUT    /users(.:format)                   user/registrations#update
                            DELETE /users(.:format)                   user/registrations#destroy
                            POST   /users(.:format)                   user/registrations#create
          new_admin_session GET    /admins/sign_in(.:format)          admin/sessions#new
              admin_session POST   /admins/sign_in(.:format)          admin/sessions#create
      destroy_admin_session DELETE /admins/sign_out(.:format)         admin/sessions#destroy
         new_admin_password GET    /admins/password/new(.:format)     admin/passwords#new
        edit_admin_password GET    /admins/password/edit(.:format)    admin/passwords#edit
             admin_password PATCH  /admins/password(.:format)         admin/passwords#update
                            PUT    /admins/password(.:format)         admin/passwords#update
                            POST   /admins/password(.:format)         admin/passwords#create
  cancel_admin_registration GET    /admins/cancel(.:format)           admin/registrations#cancel
     new_admin_registration GET    /admins/sign_up(.:format)          admin/registrations#new
    edit_admin_registration GET    /admins/edit(.:format)             admin/registrations#edit
         admin_registration PATCH  /admins(.:format)                  admin/registrations#update
                            PUT    /admins(.:format)                  admin/registrations#update
                            DELETE /admins(.:format)                  admin/registrations#destroy
                            POST   /admins(.:format)                  admin/registrations#create
```

lets make logged in home pages (for the user and admin)
```
rails g controller users/home index --no-helper --no-assets --no-controller-specs --no-view-specs
rails g controller admins/dashboard index --no-helper --no-assets --no-controller-specs --no-view-specs
```

now lets update our routes to ponit to these pages if the user is logged in add the following belos the deivse_for commands
```
Rails.application.routes.draw do

  # keeping for default routes for request specs
  namespace :admins do
    get 'dashboard/index'
  end
  namespace :users do
    get 'home/index'
  end
  get 'landing/index'

  # http://localhost:3000/users/sign_in
  devise_for :users,  path: 'users',
                      controllers: {
                        sessions:      'users/sessions',
                        passwords:     'users/passwords',
                        registrations: 'users/registrations'
                      }
  # http://localhost:3000/admins/sign_in
  devise_for :admins, path: 'admins',
                      controllers: {
                        sessions:      'admins/sessions',
                        passwords:     'admins/passwords',
                        registrations: 'admins/registrations'
                      }

  get '/landing', to: 'landing#index',          as: :landing
  get '/users',   to: 'users/home#index',       as: :users_home
  get '/admins',  to: 'admins/dashboard#index', as: :admins_dashboard

  authenticated :user do
    root 'users/home#index',        as: :auth_user_root
  end
  authenticated :admin do
    root 'admins/dashboard#index',  as: :auth_admin_root
  end
  root to: 'landing#index'
end
```

now lets require these pages to have authenticated the correct user type:
```
# app/controllers/users/home_controller.rb
class Users::HomeController < ApplicationController
  before_action :authenticate_user!
  def index
  end
end

# app/controllers/admins/dashboard_controller.rb
class Admins::DashboardController < ApplicationController
  before_action :authenticate_admin!
  def index
  end
end
```

Now prevent student and admin accounts from cross visits (during testing, or whatever) -- create this new file:
```
# ../controllers/concerns/accessible.rb
module Accessible
  extend ActiveSupport::Concern
  included do
    before_action :check_user
  end

  protected
  def check_user
    if current_admin
      flash.clear
      # The authenticated admin root path can be defined in your routes.rb in: devise_scope :admin do...
      redirect_to(auth_admin_root_path) and return
    elsif current_user
      flash.clear
      # The authenticated user root path can be defined in your routes.rb in: devise_scope :user do...
      redirect_to(auth_user_root_path) and return
    end
  end
end
```

Now add `include Accessible` in the appropriate controllers:

Note:
You must skip_before_action for the destroy action in each SessionsController to prevent the redirect to happen before the sign out occurs.
 ```
# eg. ../controllers/admins/sessions_controller.rb
class Admins::SessionsController < Devise::SessionsController
  include Accessible
  skip_before_action :check_user, only: :destroy
  # ...
end

# eg. ../controllers/admins/registrations_controller.rb
You must also skip_before_action for the edit, update, destroy, and cancel actions in each RegistrationsController to allow current users to edit and cancel their own accounts. Otherwise they will be redirected before they can reach these pages.

class Admins::RegistrationsController < Devise::RegistrationsController
  include Accessible
  skip_before_action :check_user, except: [:new, :create]
  # ...
end

# eg. ../controllers/users/sessions_controller.rb
class Users::SessionsController < Devise::SessionsController

  include Accessible
  skip_before_action :check_user, only: :destroy
  # ...
end

# eg. ../controllers/users/registrations_controller.rb
class Users::RegistrationsController < Devise::RegistrationsController

  include Accessible
  skip_before_action :check_user, except: [:new, :create]
  # ...
end
```

lets create some common feature test code:

https://forum.upcase.com/t/rspec-support-vs-helpers/4986
https://thoughtbot.com/blog/rspec-integration-tests-with-capybara
```
# spec/support/features/session_helpers.rb
module Features
  module SessionHelpers
    def user_sign_up(username:, password:)
      visit new_user_registration_path
      expect(page).to have_button('Sign up')
      fill_in 'Email', with: email
      fill_in 'Password', with: password
      click_button 'Sign up'
    end
    def user_log_in(user = nil)
      user = FactoryBot.create :user if user.nil?
      visit new_user_session_path
      expect(page).to have_button('Log in')
      fill_in 'Email', with: user.email
      fill_in 'Password', with: user.password
      click_on 'Log in'
    end

    def admin_sign_up(email:, password:)
      visit new_admin_registration_path
      expect(page).to have_button('Sign up')
      fill_in 'Email', with: email
      fill_in 'Password', with: password
      click_button 'Sign up'
    end
    def admin_log_in(admin = nil)
      admin = FactoryBot.create :admin if admin.nil?
      visit new_admin_session_path
      expect(page).to have_button('Log in')
      fill_in 'Email', with: admin.email
      fill_in 'Password', with: admin.password
      click_on 'Log in'
    end
  end
end
```

Lets tell rspec how to access this code in feature tests:
```
# spec/support/features.rb
RSpec.configure do |config|
  config.include Features::SessionHelpers, type: :feature
end
```


Lets create test for our devise model factories:
```
# spec/models/user_spec.rb
require 'rails_helper'

RSpec.describe User, type: :model do
  describe "factory functions" do
    it "generates a valid user" do
      model = FactoryBot.build :user
      expect(model.valid?).to be true
    end
    it "saves a valid user" do
      model = FactoryBot.build :user
      expect(model.save).to be_truthy
    end
  end

  describe "DB settings" do
    it { have_db_index(:email) }
    it { is_expected.to have_db_column(:encrypted_password) }
  end
end

# spec/models/admin_spec.rb
require 'rails_helper'

RSpec.describe Admin, type: :model do
  describe "factory functions" do
    it "generates a valid admin" do
      model = FactoryBot.build :admin
      expect(model.valid?).to be true
    end
    it "saves a valid admin" do
      model = FactoryBot.build :admin
      expect(model.save).to be_truthy
    end
  end

  describe "DB settings" do
    it { have_db_index(:email) }
    it { is_expected.to have_db_column(:encrypted_password) }
  end
end
```

be sure these fail - run:
```
rspec spec/models/
```


Now we need to configure the factories so all is working:
```
# spec/factories/users.rb
FactoryBot.define do
  factory :user do
    sequence(:email)      { |n| "#{Faker::Internet.email}".split('@').join("#{n}@") }
    password              { 'LetM3-InNow' }
    password_confirmation { 'LetM3-InNow' }
    # enable this if using confirmable
    # confirmed_at { Date.today }
  end
end

# spec/factories/admins.rb
FactoryBot.define do
  factory :admin do
    sequence(:email)      { |n| "#{Faker::Internet.email}".split('@').join("#{n}@") }
    password              { 'LetM3-InNow' }
    password_confirmation { 'LetM3-InNow' }
    # enable this if using confirmable
    # confirmed_at { Date.today }
  end
end
```

be sure these pass now - run:
```
rspec spec/models/
```

Now we are ready to test devise and our restricted access to the user home page and admins dashboard.:

https://www.madetech.com/blog/feature-testing-with-rspec
https://thoughtbot.com/blog/rspec-integration-tests-with-capybara
https://github.com/heartcombo/devise/wiki/How-To:-Test-with-Capybara
https://radavis.github.io/sign-in-out-test-helpers-for-and-devise-and-capybara/
https://www.vanderpol.net/2014/10/07/rspec-integration-tests-devise-user-registration/
```
# spec/features/users/user_signup_spec.rb
require 'rails_helper'

RSpec.describe 'Users Home Page', type: :feature do
  # note user is NOT created in DB!
  let(:user)  { FactoryBot.build :user }
  after :each do
    Warden.test_reset!
  end
  describe 'user is not signed-up' do
    scenario 'user signs-up on registration page' do
      user_sign_up(email: user.email, password: user.password)
      expect(current_path).to eql(users_home_path)
    end
  end
end


# spec/features/users/user_login_spec.rb
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


# spec/features/users_home_page_spec.rb
require 'rails_helper'

RSpec.describe 'Users Home Page', type: :feature do
  let(:user)    { FactoryBot.create :user }
  after :each do
    # force a logout (clear warden info) after each test
    Warden.test_reset!
  end
  describe 'user is not authenticated' do
    scenario 'user is redirected to user login before access to user home' do
      visit users_home_path
      expect(current_path).to eql(new_user_session_path)
    end
  end
  describe 'user is already authenticated' do
    before    { user_log_in(user) }
    scenario 'user gets direct access to the user homepage' do
      visit users_home_path
      expect(page).to have_current_path(users_home_path)
    end
  end
end
```

and test to be sure admin can log in too:
```
# spec/features/admins/admin_login_spec.rb
require 'rails_helper'

RSpec.describe 'Users Login', type: :feature do
  after :each do
    Warden.test_reset!
  end
  scenario 'logs in successfully and is redirected to user home page' do
    admin_log_in
    expect(current_path).to eql(auth_admin_root_path)
  end
end


# spec/features/admins/admin_signup_spec.rb
require 'rails_helper'

RSpec.describe 'Admin Signup', type: :feature do
  # IMPORTANT is NOT created in DB!
  let(:admin)  { FactoryBot.build :admin }
  after :each do
    Warden.test_reset!
  end
  describe 'admin is not signed-up' do
    scenario 'admin registers' do
      admin_sign_up(email: admin.email, password: admin.password)
      expect(page).to have_current_path(admins_dashboard_path)
    end
  end
end


# spec/features/admins/admins_dashboard_spec.rb
require 'rails_helper'

RSpec.describe 'Admins Dashboard Page', type: :feature do
  let(:admin)  { FactoryBot.create :admin }
  after :each do
    Warden.test_reset!
  end
  describe 'un-authenticated' do
    scenario 'attempts to access admins dashboard page is redirected to user login' do
      visit admins_dashboard_path
      expect(current_path).to eql(new_admin_session_path)
    end
  end
  describe 'already authenticated' do
    before    { admin_log_in(admin) }
    scenario 'gets access to the user homepage' do
      visit admins_dashboard_path
      expect(current_path).to eql(admins_dashboard_path)
    end
  end
end
```

before we wrap up - we need to fix our request specs - now that we added login restrictions:
```
# spec/requests/users/home_request_spec.rb
require 'rails_helper'

RSpec.describe "Users::Homes", type: :request do
  describe "GET /index" do
    let(:user)              { FactoryBot.create :user }
    before do
      sign_in user
    end
    after do
      sign_out user
    end
    it "returns http success" do
      get "/users/home/index"
      expect(response).to have_http_status(:success)
    end
  end
end

# spec/requests/admins/dashboard_request_spec.rb
require 'rails_helper'

RSpec.describe "Admins::Dashboards", type: :request do
  describe "GET /index" do
    let(:admin)              { FactoryBot.create :admin }
    before do
      sign_in admin
    end
    after do
      sign_out admin
    end
    it "returns http success" do
      get "/admins/dashboard/index"
      expect(response).to have_http_status(:success)
    end
  end
end
```

run the tests and be sure all is green - if so, now is a good time to make a commit!
```
git add .
git commit -m "rspec and devise configured and tests green"
```
