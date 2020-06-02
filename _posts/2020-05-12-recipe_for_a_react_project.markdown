---
layout: post
title:      "Recipe For a React Project"
date:       2020-05-12 03:04:11 -0400
permalink:  recipe_for_a_react_project
---


Several years ago my wife and I compiled a family cookbook.  We were newly married, short on funds, and were at a loss for what to give folks for the Holidays.  We collected submissions from family members, typed them up,  then had several copies bound.  Tattered and loved copies still live on bookshelves around the family.  We called it the Cooking Tree, it was a fun project, and I always wanted to make a web version.

I think written recipes are too static and too easy to just blindly follow.  They keep you from striving to make something better.  My wife wife argues, "What if the next time is not as good?"   I believe the possibility for "Wow!" is worth the risk of a few "Meh"s.  The Cooking Tree was just snapshot of the moment in time that we made it.  It doesn't reflect the years of cooking and learning since then.  The Cooking Tree also illustrated how different family members have differing recipes for the same thing, often inspired from one seed recipe.  My grandma for example makes a fantanstic pilaf, but with a traumatizing amount of butter, especially for my dietitian wife.  I subsequently use less butter but use stock instead of water.

Last summer breast cancer took my mom.  She had a similar recipe philosophy to mine.  For Christmas my wife and I wanted to make my mom's peppernuts (a hard little anise gingerbread cookie) to remember her by. She had refined the process over the years and would go into mega production during the holidays.  I asked my dad if he could hunt for the recipe.  I know, I know, irony.  We couldn't find one.  Nope she had several.  She had picures of peppernut recipes on her phone.  She had them in text docs on her computer.  She had paper ones.  We picked one.  Either it wasn't 'The One' or we just didn't do it right.  I'm convinced my wiley mother injected misinformation into her tomes like an ancient alchemist or mason.

Well with lessons learned and my wife's urging I ventured to solve the living family recipe problem for my React project.  I call it CookTree.  My wife and I outlined what we wanted to do with CookTree and I wrote some quick user stories.

* Users can sign up, login and logout
* Users can exist without a login
* Users can create Connections with other Users.
* Users can invite Connections to be logged in Users through email.
* Invited Users can follow a link to sign up. 
* Users can create Recipes
* Users can "Make" new snapshots of Recipes
* Users can browse those Recipe Makes
* Users can see who created a Make and when
* Users can add Memories with photos to Recipes

In addition to flexing my newly minted React, Redux and React Router muscles I chose Material UI for styling and had the opportunity to figure out Active Storage for photos and avatars.  For email invitations I dug into Action Mailer as well.

With each of my projects at Flatiron I got in the habit of keeping a setup log for each one.  I did this in case I needed to pivot or something went horribly wrong and I needed to start over.  Setup seems to be finnicky and easy to forget becuase you only really do it once per project.  So as a parting gift here's my cleaned up setup log.  My React Recipe.

## Setup Rails API Project

 > rails new <COOL-PROJECT> --api --skip-test --skip-bundle --database=postgresql

## Gemfile

### group :development, :test
For environment variables like passwords
> gem 'dotenv-rails' 

For debugging
> gem 'pry' 
 
For seeding data
> gem 'faker' 

### Other Gems
For json serializing
> gem 'active_model_serializers'

For firing up multiple servers
> gem 'foreman' 

### Uncomment
Allows access to the API from client
> gem 'rack-cors' 

For has_secure_password
> gem 'bcrypt', '~> 3.1.7' 

## Get PostgreSQL

https://www.postgresql.org/download/

## Setup postgres user

> psql
> 
> create role coolprojectrole with createdb login password '$uper$ecretPassword42';
> 
> \q


## config/database.yml

>default: &default
>  adapter: postgresql
>  encoding: unicode
>  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

>development:
>  <<: *default
>  database: coolproject_development
>  username: coolprojectrole
>  password: <%= ENV['COOLPROJECT_DATABASE_PASSWORD'] %>

>test:
>  <<: *default
>  database: coolproject_test
>  username: coolprojectrole
>  password: <%= ENV['COOLPROJECT_DATABASE_PASSWORD'] %>

>production:
>  <<: *default
>  database: coolproject_production
>  username: coolprojectrole
>  password: <%= ENV['COOLPROJECT_DATABASE_PASSWORD'] %>

## config/application.rb
Add these lines to use cookie based session with the API

> config.middleware.use ActionDispatch::Cookies
> config.middleware.use ActionDispatch::Session::CookieStore, key: '_cookie_name'

## .env

### Add .env to .gitignore 
So you don't commit passwords to your repo

> .env

### Create .env file
> COOLPROJECT_DATABASE_PASSWORD=$uper$ecretPassword42

## setup foreman to run servers
rails servers will run on:
  * Rails api -> http://localhost:3000
  * React client -> http://localhost:3001

### add Procfile
> web: cd client && npm start
> api: bundle exec rails s -p 3001

### add a rake task for simplicity
create /lib/tasks/start.rake

> task :start do
>  exec 'foreman start -p 3000'
> end

## /config/initializers/cors.rb
Uncomment the the middleware block provided by rails and update the origins statement.  Choose authorized methods and resources.

`Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'http://localhost:3000'

    resource '*',
    headers: :any,
    methods: [:get, :post, :put, :patch, :delete, :options, :head]
    credentials: true
  end
end`

## User Model
rails g model User email:string password_digest:string

## setup Active Storage for image files
rails active_storage:install

## setup front end
npx create-react-app client

## node packages
npm install react-router

npm install redux

npm install react-redux

npm install redux-thunk

npm install @material-ui/core

npm install @material-ui/icons





