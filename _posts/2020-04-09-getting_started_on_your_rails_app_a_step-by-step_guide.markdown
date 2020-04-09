---
layout: post
title:      "Getting Started on Your Rails App: A Step-by-Step Guide"
date:       2020-04-09 05:43:46 +0000
permalink:  getting_started_on_your_rails_app_a_step-by-step_guide
---

This guide assumes you are using Rails 5+ with ActiveRecord.

### Planning
As always everything starts with planning. Think through what you want your app to do, what you'd like its user experience to be and start writing out the requirements of the app, noting features that are mandatory and ones that would be nice to have (add these after the minimum requirements are met). 

#### User Stories
Writing some user stories can help you determine what your requirements will be. These can be formatted something like this:
* As a <type of user>
* I want <some goal>
* So that <some reason>

#### Mapping out your models
Mapping out what models you think it'll need and how they will associate with one another. It helps to write this down in a notes doc so you can take the information and fill in the migration files and model associations later. 

Post

`title:string`
`content:text`

belongs_to :user
has_many :post_categories
has_many :categories, through: :post_categories
has_many :comments

#### Draw out your schema
You may be ok just intuiting your schema based on the model maps above, but it can also help to draw them out. You can use software, like [draw.io](https://www.draw.io/) or just a piece of paper.

####  Wireframing
To determine what views you'll need, it helps to keep in mind how the app will work from a user's perspective. Think through the user flow, how are they going to acheive the user stories you wrote out? Wireframes are one way help you picture this, and they can guide your layouts and views. Sketch and [Figma](https://www.figma.com/) are two apps that can help you achieve this but you can still use the trusty piece of paper.

### Rails New
```
rails new AppName
```

Now's a good time to link your app to a remote git repository to track changes. Git has already been initialized so you just have to commit your changes (`git commit -m "initial commit"`, create a remote repository (on Github or wherever you prefer),  add the remote location ( eg. for GitHub: `git remote add origin git@github.com:USERNAME/NAME-OF-REPO.git`) and push your commit back up to origin (`git push -u origin master`).  The more often you commit during the build process the easier it will be to track down the source of errors--plus, it can be a reminder to take a breathe and/or break, so commit early and often!

### Generate Models

```
rails g model ModelName
```
or, if you include the attributes and their datatypes, this will also generate a migration for create_modelnames
Eg.
```
rails g model User name:string email:string password_digest:string height:integer something:boolean
```

The model generator will also add routes in *config/routes.rb*,  controllers, eg. *app/controllers/users_controller.rb* and helpers eg. *app/helpers/users_helper.rb*.

**Note:** If you aren't sure how to use Rails generators, running `rails generate GENERATOR --help` will return all the options that can be passed to the generator.

### Generate Migrations
as mentioned before this might not be necessary depending on how you generated your models but the format is:
```
rails g migration migration_name attr_name:datatype attr2_name:datatype
```
datatype defaults to string, so it is not necessary to declare datatypes for strings, eg.
```
rails g migration create_users name password_digest
```

### Run migrations
```
rails db:migrate
```

### Set model associations

Decide how your models will associate with one another. I find this to be one of the most difficult parts of building a Rails app. The following resources helped me wrap my head around this:

* [Learn curriculum](https://learn.co/tracks/online-software-engineering-uci-structured/rails/associations-and-rails/activerecord-associations-review) for more info on ActiveRecord associations
* [Ruby on Rails API](https://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html) on ActiveRecord associations and class methods
* [Tutorial on model relationships](http://tutorials.jumpstartlab.com/topics/models/relationships.html)

After thinking this through a bit more you may need to alter the tables you just migrated by adding association columns (eg. user_id if a model belongs to a user and a user has_one of that particular model), or join tables for many_to_many relationships that can't use one of the already existing tables. Consider keeping detailed information that doesn't need to be accessed often in its own table to help improve performance.

Open up your console by running `rails c` or `rails c -s` (the -s flag is to open up a sandbox so anything you add here while testing won't persist to your database).

Test out your relationships by creating new instances of each model in the console with associations and making sure your results are as expected. Debug as needed.

### Add authentication
If your app will be allowing users to log in with a password, you'll need a way to authenticate them. Rails makes this easy if you:
* add has_secure_password to User model
* add gem 'brycrpt' to your Gemfile and run ```bundle```

These will add an authenticate method to validate a user's password and/or password confirmation.

### Add more gems
I include 'pry' for testing. 

There are [so many](https://rubygems.org/) other [gems](https://www.ruby-toolbox.com/) that can [help you accomplish](https://dwayne.fm/rails-gems-to-consider/) your app's goals, but before installing them you'll want to consider if the problem is big enough to require a gem or if you'd be better off coding it out yourself. 
If the problem is a common one, chances are someone else has come up with a solution that may fit your needs. Gems with clean code that are well maintained and documented and relatively popular are genereally the sefest bet when deciding which gems to go with.

### Routes
If you used the model generator, it will have add all RESTful routes for each model by adding a line for each model in *config/routes.rb*, eg.

```
  resources :users
  resources :posts
	resources :categories
  resources :comments
```

You will probably want to edit these to only include what you need, extra routes will slow down your app. If you are adding any custom routes, I suggest putting them at the top, since the order here matters and you'll end up with errors if you declare custom routes after the built in dynamic ones. I might edit the above to look like:

```
  root 'posts#index'
  get '/signup' => 'users#new'

  resources :categories, only: [:index, :show]
  resources :users, except: [:delete]
  resources :posts
```


### Add Controllers and Actions

As mentioned above the model generated a controller in *app/controllers/*. The controller should inherit from ApplicationController. At this point I will start up the server with `rails s` to help me see how things are working and let th e errors help guide me. I'll typically add controller actions, then views then go between the two while clicking through on the front-end until the app starts to take shape.

### Add Views
Forms for adding new resources, indexes (indeces?), and show pages.

### Validations and Error messaging

### Refactoring
#### Helper Methods
#### Class Methods
#### Partials





