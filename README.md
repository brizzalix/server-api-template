# Server API Template

This is our CodePath template for giving people an easy way to create RESTful JSON APIs
to power their applications.

## Outline

 - [Installation](#installation)
 - [Building APIs](#building-apis)
 - [Deploying to Heroku](#deploying)

## Libraries

This project is using several libraries and frameworks:

 - Rails 3.2 (web framework)
 - Grape  (API endpoints)
 - Devise (authentication)

## Installation

### Dependencies

Make sure you have Ruby 1.9.3 and Git installed. Type into the terminal:

```bash
$ ruby -v
```

Verify you see "ruby 1.9.3pXXX" (where the XXX can be any number). If not then download and run [RailsInstaller](http://railsinstaller.org/) in order to get Ruby.

Then type into the terminal:

```bash
$ git --version
```

If you get an error then download and run [RailsInstaller](http://railsinstaller.org/) in order to get Ruby.

### Fork and Clone

The first step is to register a [github account](https://github.com/) which allows you
to store your code for free on their servers.

You probably need to [upload your ssh key](https://help.github.com/articles/generating-ssh-keys) to Github
in order to get or push repositories:

```bash
$ pbcopy < ~/.ssh/id_rsa.pub
```

If that command fails with a file not found, run `$ ssh-keygen -t rsa -C "your_email@example.com"` to generate your SSH key.

Next, go to your [ssh keys](https://github.com/settings/ssh) and paste the contents of your clipboard.

Now we need to **fork this repository** to your own account at <https://github.com/thecodepath/server-api-template>.
You can do that by clicking "Fork" on the top right.

Next, you want to clone your version of this repository locally:

```bash
$ git clone git@github.com:myusername/server-api-template.git
```

**Note:** Be sure to replace `myusername` with your own bitbucket username above.

### Setup

Run the task to install dependencies:

```bash
$ bundle
```

When this is finished, let's start th :

```bash
$ rm -rf .git
$ git init
$ git commit -am "initial commit of my app"
$ git remote add origin git@bitbucket.org:myusername/server-api-template.git
$ git push origin master --force
```

**Note:** Be sure to replace `myusername` with your own bitbucket username above for remote.

Now setup your database:

```bash
$ rake db:migrate db:test:prepare
```

### Running

Once you are setup, be sure to start your Rails application:

```bash
$ rails server
```

## Building APIs

### Design API Endpoints

 - Based on use cases, flesh out API calls needed by mobile client
 - Specify the "type", "url" and "parameters" (i.e "GET /tweets?user_id=5")

### Define Schema

 - Identify resources in your application (i.e user, tweets, favorites)
 - Identify attributes of each resource (i.e a tweet has a body, timestamp)
 - Identify the "associations" for each resource (i.e a tweet has a user_id)

### Create Models

Models are the way that a Rails application stores data for your application resources.
On a high level:

  - Use `rails g model <NAME>` to generate models for each resource
    - `rails g model Tweet`
  - Fill out the "db/migrate/xxxxx" file for each model
  - Fill out associations for each model (i.e `belongs_to :user`)
  - Fill out any validations for each model (i.e `validates :body, :presence => true`)

For example, imagine we want to create a "Tweet" resource that has a status and is created by a user. First,
we would generate the tweet resource:

```bash
$ rails g model tweet body:string user_id:integer
```

This will generate a file in `db/migrate/xxxxxx_create_tweets.rb` that defines the fields
for the tweet (right now just a body and a number representing the user).

Now we can run the migrations with `rake db:migrate` and then check out our
model file at `app/models/tweet.rb`. Models are blank by default and often don't need any
additional code. The fields (body and user_id) will work automatically.

We can now create, update or destroy tweets from within our APIs:

```ruby
# create
tweet = Tweet.create(:body => "foo", :user_id => 1)
# update
tweet.update_attribute(:body, "bar")
# find
my_tweet = Tweet.where(:body => "bar").first
# delete
my_tweet.delete
```

Once we have our models, we can build the related API endpoints so our client mobile applications
can modify the resources.

### Build Grape Resources

In Grape, APIs are defined in terms of "resources" which are different nouns within your application.
An 'endpoint' is a URL that creates, updates, returns or deletes resource data. For example, creating a new
tweet, returning a list of tweets, deleting a tweet, et al. On a high level, APIs are defined through:

  - Editing `app/api/endpoints` for various resource endpoint files
  - Add resource declarations into grape endpoint files
  - Write the endpoint code for each resource API

An API endpoint lives inside of `app/api/endpoints/someresource.rb` where "someresource" is
the noun being affected by the API. For instance, the API endpoint for registering
a new user lives in `app/api/endpoints/users.rb` and is described by the following:

```ruby
resource :users do
  desc "Register a new user"
  params do
    requires :email, type: String, desc: "email for user"
    requires :password, type: String, desc: "password for user"
  end
  post do
    @user = User.new(params.slice(:email, :password))
    if @user.save
      status 201
      @user.as_json
    else # user didn't save
      error!({ :error => "user could not be registered", :details => @user.errors }, 400)
    end
  end
end
```

Notice that there are three main parts: description (`desc`) for describing the purpose, params for specifying
required parameters for the API request and then the API code which starts with an HTTP request method such
as `get`, `post`, `put`, or `delete`.

As a rule of thumb, the request method to pick is as follows:

|Method|Description|Example|
|get|For returning resources from read-only endpoint|Get user tweets|
|post|For creating new resources|Create new tweet|
|put|For updating an existing resource|Editing a user's password|
|delete|For deleting a resource|Trashing a tweet|

API endpoints are defined in terms of other resources (tweets, trips, appointments, etc)
based on the APIs and models in your application.

## Other Tasks

Here's a list of a few other todos:

  - In "config/initializers/configure_api.rb" configure the username / password for your authenticated APIs
  - In "app/api/api_router.rb" uncomment lines to create basic authenticated endpoints.
  - In "config/environments/production.rb" fill in the real domain for your application
  - In "config/initializers/airbrake.rb" fill in the token for your free airbrake account (for error reporting)

## Usage

Key files to edit:

  - "app/api/endpoints/*" - Adding endpoints and APIs
  - "db/migrate" - Defining the model attributes in the database
  - "app/models" - Defining any additional model information
  - "test/api"   - Defining tests for your APIs (if needed)

Few URLs to note (once rails server is running):

  - "/api/sessions" - Simple endpoint that returns text
  - "/rails/routes" - See a list of common rails routes
  - "/users/sign_in" - Login (or register) a user
  - "/admin" - Admin panel for viewing database content

## Deploying

The easiest way to deploy your APIs is to use [Heroku](http://heroku.com).

### Register for an account

First, register yourself a (free) Heroku account at <https://api.heroku.com/signup>. This is
your developer account that can contain any number of free applications.

### Create app

Run the following command in the terminal to create your app:

```bash
$ gem install heroku
$ heroku login
$ heroku create myappname
```

**Note:** Be sure to replace `myappname` with your own application name above.

Be sure to enter your username and password as defined when you created your Heroku account earlier.

### Deploy App

Next it is time to deploy your application:

```bash
$ git push heroku master
```

You may need to type 'yes' when it asks if you want to continue. At this point you should see
Heroku deploying your application to the internet:

```
Warning: Permanently added the RSA host key for IP address '50.19.85.156' to the list of known hosts.
Counting objects: 206, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (184/184), done.
Writing objects: 100% (206/206), 53.24 KiB, done.
Total 206 (delta 70), reused 0 (delta 0)

-----> Ruby/Rails app detected
-----> Installing dependencies using Bundler version 1.3.2
...
```

Wait while this command sets up your application on their servers. Once this is finished, it is time to setup our application on their servers:

### Verify App

Now you can open the url to your app with:

```bash
$ heroku open
```

and now you can visit `/api/vi/sessions` in your browser to confirm this app is running if you see:

```
"This is a sign that the API endpoints are configured"
```

### Wrapping Up ###

At this point you have a deployed API application. If you make changes to your app, simply run:

```bash
$ git add .
$ git commit -am "describe my changes here"
$ git push heroku master
```

and the updated code will be pushed to Heroku accordingly.