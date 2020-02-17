---
layout: post
title:      "How to build a simple CMS "
date:       2020-02-17 00:07:43 -0500
permalink:  how_to_build_a_simple_cms
---

The simple answer? With Corneal, Sinatra and Active Record!

## Planning

Like with all projects, it helps to sit down and do some planning before digging in. Some questions that might help get you started:

* What will your app be used for?
* Who will be using it? 
* What can you do to help ensure it's accessible to the greatest number of people?
* How and where will your users be interacting with their content? 
* Which pieces of data are related to one another, and how? That is, how will your Ruby objects and their attributes be associated with one another, and how will that inform your database structure?
* Will you need a controller and views for every model or will your users be manipulating different pieces of data on the same page? 

For my project, I wanted to keep track of data stored in lists. I didn't want my site visitors to have to edit each list item separately, and thought it made more sense for them to be able to add list items via the same form in they were adding or editing a list, so I opted out of using a ListItems Controller and kept that logic in the parent Lists Controller, with the forms to add or edit the content nested within the List views. 

Think through your database, before building it out. I found [this blog post](https://alicebrunel.github.io/sinatra_portfolio_project_-_database_design) by fellow Flatiron student Alice Brunel helpful when thinking this through.

The planning process takes time--don't rush it! Initially I had grand ideas for what my app would do, and when I got to designing the database I decided to pair back to meet the requirements of my project without going overboard. If you aren't super comfortable with has_many_through associations, you can start with has_many and belongs_to relationships.

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

```
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :username
      t.string :password_digest

      t.timestamps null: false
    end
  end
end
```

Once the database tables are created (`rake db:migrate`) you'll want to add assocations to the models. 

Take time to think this through again. If you have to, you can create and run more migrations if you need to add columns to your tables.

**NOTE:** it is possible to adjust associations later on, but I definitely don't recommend it! I made the poor decision to change an attribute to another model the day before my project was due, and a lot of debugging ensued. All instances of the attribute in the code needed to be update to become an object, and it's easy to miss something.

Here's an example from my project of some associations in the list model (list.rb):

```
class List < ActiveRecord::Base
    belongs_to :user
    has_many :list_items   
end
```

Once your models are setup you can move on to Views and Controllers.

## Controllers

Controllers are where you'll be setting the routes for your CRUD actions, among other things. CRUD stands for Create, Read, Update, Delete, all ways of manipulating data. In your controller, you will determine what view is rendered based on the URL the user is visiting. To create a new piece of content, a user might be directed to an html form in a view file (conventionally called new.erb). The controller will direct them there when, for instance they go to www.example.com/list/new.  Once the user submits the form, its POST action will send the inputted information back to the controller, which will extract the inputted data and store it to the database using the params hash. If a user enters information into an input with the name "content", the inputted data can be retrieved in the POST route within the controller viea the content key of the params hash, eg. params[:content].  Edit forms use a PATCH action and delete forms use a DELETE action, but otherwise work in pretty much the same way.

## Views

As mentioned above, views are the way in which we display content to a site visitor.  They are what's often referred to as the "front-end" of a website, and, in a Sinatra app, mostly consist of HTML intersperced with Ruby code wrapped in ERB tags. The HTML is styled with CSS and sometimes JavaScript. Forms to add, edit, or delete content are displayed in a view, as well as anything else displayed on a web application. An example of a "show" view for a simple list might look something like:

```
<p class="details">
		authored by: <a href="/users/<%= list_owner.id %>"><%= list_owner.username %></a>
		<% if @list.category %>
				<br>
				category: <a href="/categories/<%= @list.category.id %>"><%= @list.category.name %></a> 
		<% end %>
</p>

<ol class="show-list">
		<% @list.list_items.each do |item| %>
				<li><%= item.content  %></li>
		<% end %>
</ol>
```

You'll notice this view has ruby embedded within ERB tags. It might be better for me to take a bit more logic out of the view and put it into the lists controller.

## Security

If you are building a CMS that allows users to log in, log out, and manipulate data, you'll need to be mindful of security. A good first step is to *Never* store passwords as plaintext in the database. The gem 'bcrypt' paired with ActiveRecord tightens security by transforming the user's password into a seemingly random string of letters and numbers. Bcrypt allows you to authenticate the user's password against this encrypted version stored in the database (for example when they are logging in). Name the password column of your user's table (in your database) `password_digest` (you'll use the normal "string" datatype) and put this line of code in your user model: `has_secure_password`.

You will also need to limit GET, POST, PATCH and DELETE routes so user1 can't edit (or maybe worse, delete) user2's content. You can add an additional layer of security by limiting what loads in the view based on who the current user is, and who is the owner of the content. A simple helper method in the application controller to determine the current user might look like:

```
def current_user 
    @current_user ||= User.find_by(:username => session[:username]) if session[:username]
end
```

Another way to harden security is with a session secret. 
This is a key used for signing and encrypting cookies set by your application to allow user to remain logged in without  having to re-login with every page load. A malicious party can use the secret to decrypt these cookies and pretend to be logged in to someone else's account.

The session secret:
* should *not* manually entered (a randomly generated key of at least 64 bytes is best)
* should *not* be easy to guess (please don't use "secret" in development because you may forget to switch it over when deploying to a production environment) and 
* should *not* be submitted to your github repo (use the DotEnv ruby gem)

[This blog post](https://mackenzie-km.github.io/secure_session_secrets_for_sinatra) by Mackenzie Moore dives a bit deeper into session secrets and how to install and implement DotEnv.


