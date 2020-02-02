---
layout: post
title:      "Migrating data across database engines with Rails 6"
date:       2020-02-02 00:00:02 -0500
permalink:  migrating_data_across_database_engines_with_rails_6
---

I've recently been trying to create a system to export a Postgres database to SQLite. I was working on a backend for a mobile app that will contain an SQLite database with the plan to release a new version of the SQLite database on a biweekly bases.

I found some solutions that worked on the database level. But they quickly ran into issues with data being marked as corrupt. That seemed odd as I can open the database in a database viewer that showed all the data just fine. A theory I had about the issue was this was caused by the databases using different encodings. I'd probably need an intermediate layer to decode and reencode the data in the correct encoding. After quite a bit of searching and sitting on the issue for a week or two. I wondered if I can use rails 6's multi-database support to connect to both databases at the same time and copy data between databases.

I'll try to keep this simpler then it was for my use case as I was using Heroku which does not allow the SQLite gem (This is actually for a very good reason besides for the general limitations of SQLite, there is a major problem using SQLite on Heroku as it's using an [Ephemeral Filesystem](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem). That gets cleared of any changes daily causing the database to be deleted)

Side note if you want to migrate data between two environments like production and staging you do not need to set up multiple databases in the same environment and won't require the rails 6 features for dealing with multiple databases (jump to step 3). when connecting to databases in step 4 instead of using `:origin` and `:destination` use `:production` and `:development` (or whatever environments you need to migrate between.)

1. you may want to set up a rails environment specifically for this migration (especially if this is not a one-off migration) for more on that see https://nts.strzibny.name/creating-staging-environments-in-rails/ for this article I'll be using an environment called migrator
 I've therefore done the following:

    1. Created a file config/environments/migrator with the following contents  `
require File.expand_path('../development.rb', __FILE__)` as I plan on running this locally on my machine (At least until I can get SQLite running on Heroku or move to another hosting provider.)

    2. Configured my database adaptor gems to be installed in the correct environments.

    3. Set an environment variable on Heroku so it doesn't attempt to install SQLite `BUNDLE_WITHOUT=development:test:migrator` (by default it bundles without development and test)

2. Follow step 1 of [active record multiple-databases guide](https://guides.rubyonrails.org/active_record_multiple_databases.html), to set up the databases you would like to migrate between. Do not follow step 2 of the guide if you'de like to use the same database schema for both databases

  As an example, my database config (config/database.yml) for the migrator environment looks like this.
  ```yaml
  migrator:
    destination:
      adapter: postgresql
      pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
      timeout: 5000
      url: <%= `heroku config:get DATABASE_URL --app redacted-app-name` if Rails.env.migrator? %>
    origin:
      adapter: sqlite3
      pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
      timeout: 5000
      database: db/development.sqlite3
  ```

  The command used above `` `heroku config:get DATABASE_URL --app redacted-app-name` `` is used to get the production Postgres credentials, (reading from the apps environment variables.) `if Rails.env.migrator?` is appended so that this is not run. in production environments where it would fail due to not having the Heroku CLI installed.

3. (3) Make sure to create and migrate both databases.

4.  (4) Run the following code. Changing the models array and the database names `:origin` and `:destination` as needed.

  ```ruby
  # Get all models you'd like to migrate.
  models = [Question, QuestionVersion, Answer, AnswerVersion, Comment, Vote]

  # For each model do the following.
  models.each do |model|

    # Connect to the origin database.
    ActiveRecord::Base.establish_connection(:origin)

    # Load all record from the database.
    # .to_a forces eager loading of records before the connected database is changed
    # by changing the result from an ActiveRecord_Relation  to an array of in records.
    records = model.all.to_a

    # Change over to the destination database.
    ActiveRecord::Base.establish_connection(:destination)

    # Create (and save) a new record in the destination database for each record .
    # This odd way of saving records `.create!(record.attributes)`
    # is used to keep it's id's and therefore it's relationships intact.
    records.each do |record|
      record.class.create!(record.attributes)
    end

  end
  ```

Depending on your model relationships you may need to disable some validations and/or order your models array correctly so that you don't cause validation errors.

A note of warning about running this in a web server (as opposed to a rake task or in the console). The connected database is set for all threads and will affect other web requests happening at the same time. You are better off running this in its own server that isn't accepting web requests.

Some of the information is based on this article <https://alistairisrael.wordpress.com/2007/09/07/using-rails-console-to-copy-records-across-databases/> which is from the rails version 1 days. it is mostly the same until rails 6.

