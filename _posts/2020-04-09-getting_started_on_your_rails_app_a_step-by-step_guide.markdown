---
layout: post
title:      "Getting Started on Your Rails App: A Step-by-Step Guide"
date:       2020-04-09 01:43:47 -0400
permalink:  getting_started_on_your_rails_app_a_step-by-step_guide
---

This guide assumes you are using Rails 5+ with ActiveRecord.

## Planning
As always everything starts with planning. Think through what you want your app to do, what you'd like its user experience to be and start writing out the requirements of the app, noting features that are mandatory and ones that would be nice to have (add these after the minimum requirements are met). 

### User Stories
Writing some user stories can help you determine what your requirements will be. These can be formatted something like this:
* As a *type of user*
* I want *a goal*
* So that *a reason*

### Mapping out your models
Mapping out what models you think it'll need and how they will associate with one another. It helps to write this down in a notes doc so you can take the information and fill in the migration files and model associations later. [This article on the Odin Project](https://www.theodinproject.com/courses/ruby-on-rails/lessons/active-record-associations) was really helpful for me when thinking through some of the more complex associations.

#### Shop

`name:string`
`description:text`

* belongs_to :user
* has_many :items

### Draw out your schema
You may be ok just intuiting your schema based on the model maps above, but it can also help to draw them out. You can use software, like [draw.io](https://www.draw.io/) or just a piece of paper.

###  Wireframing
To determine what views you'll need, it helps to keep in mind how the app will work from a user's perspective. Think through the user flow, how are they going to acheive the user stories you wrote out? Wireframes are one way help you picture this, and they can guide your layouts and views. Sketch and [Figma](https://www.figma.com/) are two apps that can help you achieve this but you can still use the trusty piece of paper.

## Create your app
```
rails new AppName
```

Now's a good time to link your app to a remote git repository to track changes. Git has already been initialized so you just have to commit your changes (`git commit -m "initial commit"`, create a remote repository (on Github or wherever you prefer),  add the remote location ( eg. for GitHub: `git remote add origin git@github.com:USERNAME/NAME-OF-REPO.git`) and push your commit back up to origin (`git push -u origin master`).  The more often you commit during the build process the easier it will be to track down the source of errors--plus, it can be a reminder to take a breathe and/or break, so commit early and often!

## Generate Models

```
rails g model ModelName
```
or, if you include the attributes and their datatypes, this will also generate a migration for create_modelnames
Eg.
```
rails g model Shop Shop name:string status:string user:references
```

The model generator will also add routes in *config/routes.rb*,  controllers, eg. *app/controllers/shops_controller.rb* and helpers eg. *app/helpers/shops_helper.rb*.

This generation would also create a belongs_to association between shops and users.

**Note:** If you aren't sure how to use Rails generators, running `rails generate GENERATOR --help` will return all the options that can be passed to the generator.

## Generate Migrations
As mentioned before this might not be necessary depending on how you generated your models but the format is:

`rails g migration migration_name attr_name:datatype attr2_name:datatype`

datatype defaults to string, so it is not necessary to declare datatypes for strings, eg.

`rails g migration create_users name password_digest`


## Run migrations
```
rails db:migrate
```

## Set model associations

Decide how your models will associate with one another. I find this to be one of the most difficult parts of building a Rails app. The following resources helped me wrap my head around this:

* [Learn curriculum](https://learn.co/tracks/online-software-engineering-uci-structured/rails/associations-and-rails/activerecord-associations-review) for more info on ActiveRecord associations
* [Ruby on Rails API](https://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html) on ActiveRecord associations and class methods
* [Tutorial on model relationships](http://tutorials.jumpstartlab.com/topics/models/relationships.html)

After thinking this through a bit more you may need to alter the tables you just migrated by adding association columns (eg. user_id if a model belongs to a user and a user has_one of that particular model), or join tables for many_to_many relationships that can't use one of the already existing tables. Consider keeping detailed information that doesn't need to be accessed often in its own table to help improve performance.

Open up your console by running `rails c` or `rails c -s` (the -s flag is to open up a sandbox so anything you add here while testing won't persist to your database).

Test out your relationships by creating new instances of each model in the console with associations and making sure your results are as expected. Debug as needed.

## Add authentication and validations
If your app will be allowing users to log in with a password, you'll need a way to authenticate them. Rails makes this easy if you:
* add has_secure_password to User model
* add gem 'bcrypt' to your Gemfile and run ```bundle```

These will add an authenticate method to validate a user's password and/or password confirmation.

Another option is to use [Devise](https://github.com/heartcombo/devise) to generate your User model. 
1. Install Devise: `rails g devise:install`
2. Configure devise in *config/initializers/devise.rb*
3. Generate your user model: `rails g devise model User `
4. Setup authentication in your user model. This is how I did it in my Rails project, you will have to determine the best setup for your needs:
```  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable,
         :omniauthable, omniauth_providers: [:facebook]```
				 
With Devise you can use built-in methods like `authenticate_user!` (this makes sure the user is logged in) in your controllers. It can also validate user data for you, set up a password recovery flow, determine password requirements for stronger passwords and use omniauth to log in users from 3rd parties like Facebook and Google.

## Add more gems
I like to inclue 'pry' in development and test environments.

There are [so many](https://rubygems.org/) other [gems](https://www.ruby-toolbox.com/) that can [help you accomplish](https://dwayne.fm/rails-gems-to-consider/) your app's goals, but before installing them you'll want to consider if the problem is big enough to require a gem or if you'd be better off coding it out yourself. 
If the problem is a common one, chances are someone else has come up with a solution that may fit your needs. Gems with clean code that are well maintained and documented and relatively popular are genereally the sefest bet when deciding which gems to go with.

In this project I also included:
* [Bootstrap](https://getbootstrap.com/)
* [Omniauth](https://github.com/omniauth/omniauth)
* [Kramdown](https://github.com/gettalong/kramdown)

## Routes
If you used the model generator, it may have added all RESTful routes for each model by adding a line for each model in *config/routes.rb*, eg.

```
  resources :users
  resources :shops
	resources :categories
  resources :items
```

You will probably want to edit these to only include what you need, extra routes will slow down your app. If you are adding any custom routes, I suggest putting them at the top, since the order here matters and you'll end up with errors if you declare custom routes after the built in dynamic ones. I edited my routes look like:

```
root to: 'items#index'
  get '/items', to: 'items#index'
  
  resources :shops do
    resources :items
  end
  resources :categories, only: [:show, :index]

  resources :conversations do
    resources :messages
  end
  
  devise_for :users,
    :controllers => { registrations: 'users/registrations', omniauth_callbacks: "users/omniauth_callbacks" },
    :path_names => {
      :sign_in => 'login',
      :sign_out => 'logout',
      :registration => 'register',
      :sign_up => 'signup' 
    }
```

## Add Controllers and Actions

As mentioned above, the model generated a controller in *app/controllers/*. The controller should inherit from ApplicationController. At this point, if I haven't already, I will start up the server with `rails s` to help me see how things are working and let the errors help guide me. I'll typically add controller actions, then views then go between the two while clicking through on the front-end until the app starts to take shape.

## Add Views
Views include forms for adding new resources, indexes (indeces?), and show pages. Logic should be kept out of views as much as possible. Views are for displaying information to those using the site. They contain HTML and ERB code.

## Error messaging

Error messaging will help you build your app as well as indicate what is going on to the end users.
I added this code to my site's header in the *application.html.erb* layout:

```
<% flash.each do |type, message| %>
        <div class="notices-and-alerts">
          <% if type == "alert" %>
            <div class="bg-red">
              <div class="container"><%= message %></div>
            </div>
          <% end %>
          <% if type == "notice" %>
            <div class="bg-green">
              <div class="container"><%= message %></div>
            </div>
          <% end %>
        </div>
      <% end %>
```


That also differentiates between notices (when things go as expected) and alerts (when they don't). I ran out of time to style these so the background changes color depending on the type of alert, but I like the idea anyway.
			

			
		





