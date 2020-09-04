---
layout: post
title:      "Adding Stripe to a Rails Marketplace app"
date:       2020-09-04 05:28:51 +0000
permalink:  adding_stripe_to_a_rails_marketplace_app
---

This guide is adding Stripe configuration to a marketplace app built on Ruby on Rails. It assumes you are using Devise for your user model and setups up an Omniauth login as well as a way for the sellers on your app to associate their user accounts with their Stripe account so they can each accept payments seperately.

### Setup Stripe accounts

1. Create two Stripe accounts for testing, one for the app owner and one for an example buyer. 
2. Collect API keys on the Strpe website and client_id under settings > connect settings from the app owner's account and set callback uri to http://localhost:3000/users/auth/stripe_connect/callback (settings we set with stripe omniauth controller sets this route)
3. Install Stripe, omniauth, omniauth-stripe-connect: 
```gem 'stripe'
gem 'omniauth', '~> 1.9'
gem 'omniauth-stripe-connect'
```
4. `rails g migration add_stripe_fields_to_users uid:string provider:string access_code:string publishable_key:string `
5. Migate the database with `rails db:migrate`

### Omniauth Flow

1. Update routes by adding a user controller 
`devise_for :users, controllers: { omniauth_callbacks: "omniauth_callbacks" }`
2. Add omniauth configuration to devise config file config/initializers/devise.db (you can name the key whatever you want, all the providers might have slightly different names but you can stick to private_key if you want:`config.omniauth :stripe_connect, Rails.application.credentials.dig(:stripe)[:connect_client_id], Rails.application.credentials.dig(:stripe)[:private_key]`
3. Rails command to decrypt/generate credential YAML files
My editor is VS Code so I typed this into my terminal to create it in VS Code:
`EDITOR=" code --wait " rails credentials:edit`
This is where you put production keys.

You can also specify an environment if you want to have separate keys for dev, testing and production:
`EDITOR="code --wait" rails credentials:edit --environment=development`. You will add your keys here in this credentials file
4. Find your connect client_id key, currently located at: dashboard.stripe.com/account/applications/settings
publishable and private keys can be found under Developers > Api keys
```
stripe: 
connect_client_id: 
publishable_key: 
private_key:
 ```

### Configure Stripe

This is how to add omniauth for stripe to your user model if you are using devise:

1. To update your user model under Devise, first add `:omniauthable, omniauth_providers: [:stripe_connect]`
2. Create omniauth_callbacks_controller.rb and set the flow for successful and unsuccessful logins
`touch app/controllers/omniauth_callbacks_controller.rb`

```
class OmniauthCallbacksController < Devise::OmniauthCallbacksController

  def stripe_connect
    auth_data = request.env["omniauth.auth"]
    @user = current_user
    if @user.persisted?
      @user.provider = auth_data.provider
      @user.uid = auth_data.uid
      @user.access_code = auth_data.credentials.token
      @user.publishable_key = auth_data.info.stripe_publishable_key
      @user.save

      sign_in_and_redirect @user, event: :authentication
      flash[:notice] = 'Stripe Account Created And Connected' if is_navigational_format?
    else
      session["devise.stripe_connect_data"] = request.env["omniauth.auth"]
      redirect_to root_path
    end
  end

  def failure
    redirect_to root_path
  end
end
```
3. Add Stripe config under config/initializers/stripe.rb
`touch config/initializers/stripe.rb`

```
Rails.configuration.stripe = {
  :publishable_key => Rails.application.credentials.dig(:stripe)[:public_key],
  :secret_key => Rails.application.credentials.dig(:stripe)[:private_key]
}
Stripe.api_key = Rails.application.credentials.dig(:stripe)[:private_key]
```
4. Verify it's working by running `Rails.application.credentials.dig(:stripe)[:publishable_key]` and the other lines in that file in rails console.
5. Update helper methods to add `stripe_url` (in app/helpers/application_helper.rb or appliation_controller.rb wherever those helpers are going), eg:
```
module ApplicationHelper
  def stripe_url
    "https://connect.stripe.com/oauth/authorize?response_type=code&client_id=#{Rails.application.credentials.dig(:stripe)[:connect_client_id]}&scope=read_write"
  end
end
```

a note on hash dig: https://apidock.com/ruby/Hash/dig
6. add this logic to the user model:
```
def can_receive_payments?
    uid? && provider? && publishable_key? && access_code?
end
```
		

### Views
Update your registration form (app/views/devise/registration/edit.html.erb)

* let users know they need to connect to stripe in order to complete their registration to become a seller 
* add "Sign in with Stripe" button for your Omniauth login
* test: verify in rails console that User.last has stripe fields filled out.




		
		
	
		








