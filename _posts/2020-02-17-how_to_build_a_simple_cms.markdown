---
layout: post
title:      "How to build a simple CMS "
date:       2020-02-17 05:07:42 +0000
permalink:  how_to_build_a_simple_cms
---

### with Sinatra and Active Record

## Project Requirements: 

1. Build an MVC Sinatra application.
2. Use ActiveRecord with Sinatra.
3. Use multiple models.
4. Use at least one has_many relationship on a User model and one belongs_to relationship on another model.
5. Must have user accounts - users must be able to sign up, sign in, and sign out.
6. Validate uniqueness of user login attribute (username or email).
7. Once logged in, a user must have the ability to create, read, update and destroy the resource that belongs_to user.
8. Ensure that users can edit and delete only their own resources - not resources created by other users.
9. Validate user input so bad data cannot be persisted to the database.

## Planning

What will your app be used for? How will your users be interacting with their data? How will your Ruby objects be associated with one another? 

Think through your database. I found [this blog post](https://alicebrunel.github.io/sinatra_portfolio_project_-_database_design) by fellow Flatiron student Alice Brunel helpful when thinking this through.

This process takes time. Initially I had grand ideas for what my app would do, and when I got to designing the database I decided to pair back to meet the requirements of my project without going overboard. If you aren't super comfortable with has_many_through associations, you can stick to has_many and belongs_to relationships.

Once you have a pretty good idea of how the models, views and controller will interact, it's time to...

## Get Started

At this point can build out your directory structure, add config and environment files, connect your app to a database and build out the database migration files to set up your database tables. You'll want to add some gems in your Gemfile to help get things working (Sinatra, ActiveRecord, Rake, Require all, Sqlite3, Pry, and Shotgun to name a few essentials). BUT! You can skip this part of the process by installing a gem called [Corneal](https://github.com/thebrianemory/corneal) that will populate your app with all the files and gems it needs to get started. 

## Here's how:

Install corneal with `gem install corneal`
Make sure you are in the appropriate directory (one level up from where your app will live) and type `corneal APP-NAME new` into the command line. This will create a directery named whatever you replace APP-NAME with and populate it with all the files you need for a simple Model-View-Controller (MVC) Sinatra application. The entire directory structure will look something like this ![]http://)

CD into the new app directory and run `bundle install` to install your gems.
Then run `shotgun` to test that everything is working. Debug as needed.

Add your models. Corneal can get this started for you too! 
Type `corneal scaffold MODEL` (replacing MODEL with the name of your model, singular). You can even add attributes to your model that will be added to the database migration files at the same time by typing `corneal scaffold MODEL ATTR-NAME:DATATYPE ATTR-NAME:DATATYPE`
For example, `corneal scaffold user username:string password_digest:string` will add a new user.rb file in the models directory, a user directory under "views" with index.erb, show.erb, new.erb and edit.erb, and a database migration file that looks like the following:

`class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :username
      t.string :password_digest

      t.timestamps null: false
    end
  end
end`

Once the database tables are created (`rake db:migrate`) you'll want to add assocations to the models. 

Take time to think this through again. If you have to, you can create and run more migrations if you need to add columns to your tables.

**NOTE:** it is possible to adjust associations later on, but I definitely don't recommend it! I made the poor decision to change an attribute to another model the day before my project was due, and a lot of debugging ensued. All instances of the attribute in the code needed to be update to become an object, and it's easy to miss something.

Here's an example from my project of some associations in the list model (list.rb):

`class List < ActiveRecord::Base
    belongs_to :user
    has_many :list_items   
end`

## Security

If you are building a CMS allows users to log in, log out, and manipulate data, you'll need to be mindful of security. *Never* store passwords as plaintext in the database. An easy way to avoid this is to use the built in ActiveRecord method `has_secure_password`. Put this in your user model.

If your user will be entering info via forms, you will need to validate some of their input. 

You will also need to limit the routes so user1 can't edit (or maybe worse, delete) user2's content.  Logic can be placed in the routes to avoid this, and you can add an additional layer of security by limiting what loads in the view based on who the current user is, and who is the owner of the content. A simple helper method in the application controller to determine the current user might look like:

`    def current_user
      @current_user ||= User.find_by(:username => session[:username]) if session[:username]
    end`
