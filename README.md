## Setup

Before running, you shoudl have the following installed on your machine:

* Ruby
* Rails
* Postgres
* MongoDB

## Walkthrough

1. Create the rails application: `rails new rails-postgres-mongo`

## Setup Postgres connection for Rails application
2. Change `Gemfile` from `sqlite3` to `pg` so we will use our Postgres locally
3. Change `config/database.yml` to:

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
4. Generate a new ActiveRecord model for User: `rails g model User`
5. Change `db/migrate/*_create_users.rb` to:

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
6. Run `rake db:create`
7. Run `rake db:migrate`

## Test Postgres connection
8. Run `rails console`:

```
> User.create!(name: "Chris", email: "chris@compose.io")
```

## Create Mongo connection for Rails application
9. Append to `Gemfile`: `gem 'mongoid'`
10. Run a Mongoid configuration installation script: `rails g mongoid:config`
11. Replace `config/application.rb` line 7 with `Bundler.require(:default, Rails.env)`

## Create Mongoid model files
12. Run `rails g model Status`
13. Change `app/model/status.rb` to:

```
class Status
  include Mongoid::Document
  include Mongoid::Timestamps

  field :user_id, type: Integer
  field :status, type: String
end
```

## Create the relationships in the data model

14. Define how `user.rb` will use status:
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

15. Define how `status.rb` will use user:
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
