## Setup

Before running, you should have the following installed on your machine:

* Ruby
* Rails
* Postgres
* MongoDB

## Walkthrough
* Create the rails application: `rails new rails-postgres-mongo`

## Setup Postgres connection for Rails application
* Change `Gemfile` from `sqlite3` to `pg` so we will use our Postgres locally
* Change `config/database.yml` to:

```yaml
default: &default
  adapter: postgresql
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: rails_postgres_mongo_development
```

## Create the model mapping for ActiveRecord
* Generate a new ActiveRecord model for User: `rails g model User`
* Change `db/migrate/*_create_users.rb` to:

```ruby
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
      t.string :email
      t.timestamps null: false
    end
  end
end
```

## Run migrations for Postgresql database
* Run `rake db:create`
* Run `rake db:migrate`

## Test Postgres connection
* Run `rails console`:

```
> User.create!(name: “Chris”, email: “chris@compose.io”)
```

## Create Mongo connection for Rails application
* Append to `Gemfile`: `gem ‘mongoid’`
* Run a Mongoid configuration installation script: `rails g mongoid:config`
* Replace `config/application.rb` line 7 with `Bundler.require(:default, Rails.env)`

## Create Mongoid model files
* Run `rails g model Status`
* Change `app/model/status.rb` to:

```
class Status
  include Mongoid::Document
  include Mongoid::Timestamps

  field :user_id, type: Integer
  field :status, type: String
end
```

## Create the relationships in the data model

* Define how `user.rb` will use status:
```ruby
class User < ActiveRecord::Base

  def status
    Status.where(user_id: self.id).order_by({created_at: -1}).one
  end

  def status=(status)
    Status.create!(user_id: self.id, status: status)
  end

  def statuses
    Status.where(user_id: self.id).order_by({created_at: -1})
  end

end
```

* Define how `status.rb` will use user:
```ruby
class Status
  include Mongoid::Document
  include Mongoid::Timestamps

  field :user_id, type: Integer
  field :status, type: String

  def user
    User.find(self.user_id)
  end
end
```

# Start your application

```
rackup
```

open http://localhost:9292/

# Setup a default route

Change `config/routes.rb` to:

```
Rails.application.routes.draw do
  root “users#index”
end
```

# Create a users controller

```
rails g controller Users
```

Then, set the UsersController to:

```
class UsersController < ApplicationController
  def index
    @users = User
  end
end
```

# Create view

```
<h1>Users</h1>

<ul>
  <%- @users.all.each do |user| %>
    <li><%= user.name %> - <%= user.status.status %></li>
  <%- end %>
</ul>
```

# Add bootstrap stylesheets to the body

```
<!— Latest compiled and minified CSS —>
<link rel=“stylesheet” href=“https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css” integrity=“sha512-dTfge/zgoMYpP7QbHy4gWMEGsbsdZeCXz7irItjcC3sPUFtf0kuFbDz/ixG7ArTxmDjLXDmezHubeNikyKGVyQ==“ crossorigin=“anonymous”>

<!— Optional theme —>
<link rel=“stylesheet” href=“https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap-theme.min.css” integrity=“sha384-aUGj/X2zp5rLCbBxumKTCw2Z50WgIr1vs/PFN4praOTvYXWlVyh2UtNUU0KAUhAX” crossorigin=“anonymous”>

<!— Latest compiled and minified JavaScript —>
<script src=“https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js” integrity=“sha512-K1qjQ+NcF2TYO/eI3M6v8EiNYZfA95pQumfvcVrTHtwQVDG+aHRqLi/ETn2uB+1JqwYqVG3LIvdm9lj6imS/pQ==“ crossorigin=“anonymous”></script>
```

# Let’s add a form to automatically add new statuses

Append this to your users#index form:

```
<%- form_for [:user, Status.new] do ||f| %>
  <div class=‘form-input’>
    <input type=‘text’ name=‘user’>
  </div>
  <div class=‘form-input’>
    <input type=‘text’ name=‘status’>
  </div>
<%- end %>
```

Add this your `config/routes.rb` to this:

```
  resources :users do
    resources :statuses
  end
```
