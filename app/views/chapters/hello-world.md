# Hello World!

Let's build a Hello World app first. We can ensure that we have our development environment setup and get a 10,000 ft view of how things work.

## Create a New Rails App

```
rails new ember-tutorial -d postgresql
cd ember-tutorial
rvm gemset create ember-tutorial
```

Remove username and password from your `config/database.yml` if necessary, then run `rake db:create`.

If you run `rails s` and visit localhost:3000 then you should see the Rails Welcome Aboard page.

## Remove Turbolinks

You'll need to remove Turbolinks because it conflicts with Ember. Make sure to remove it from all of the following places:

* Gemfile
* Application Javascript (app/assets/javascripts/application.js)
* Layout (app/views/layouts/application.html.erb)

[Commit](https://github.com/vicramon/ember-hello-world/commit/2ec275447edd98e2ec004f4f1c281d4fa4418311)

## Add Ember Rails

I am going to use [Ember Rails](https://github.com/emberjs/ember-rails) for this tutorial. It's stable and works great. I think [Ember Appkit Rails](https://github.com/dockyard/ember-appkit-rails) may soon replace Ember Rails as the default Rails/Ember intregration gem, but it's pre 1.0 so I'm going with Ember Rails for this tutorial.

Let's add following gems to our Gemfile.

```ruby
gem 'ember-rails'
gem 'ember-source'
gem 'emblem-rails'
```

```shell
bundle
```

This command will generate a skeleton for our Ember app. It generates quite a few files. I'll go over them soon.

```shell
rails g ember:bootstrap -g --javascript-engine coffee -n App
```

Add the following lines to your environment files. These just tell Ember Rails which version of ember.js to use in each environment.

```ruby
# config/environments/test.rb
config.ember.variant = :development

# config/environments/development.rb
config.ember.variant = :development

# config/environments/production.rb
config.ember.variant = :production
```

Ember Rails generates an `application.js.coffee` for us, so lets use that. Delete `application.js`, and make sure to add jQuery to the top of `application.js.coffee`. Ember needs jQuery.

```coffee
#= require jquery
#= require jquery_ujs
```

[Commit](https://github.com/vicramon/ember-hello-world/commit/8d34669ae4649ca17d80d5a52dccf98535d36786)

## Making Ember Work

We'll need a basic Rails controller and view so that we can output something from Ember. I'm going to make a controller called `HomeController` and make it the root path.

```ruby
# config/routes.rb
root to: 'home#index'

# app/controllers/home_controller.rb
class HomeController < ApplicationController
end
```

Now we need a view. It's going to have a Handlebars template with just one tag: `{{ outlet }}`. If you know Rails, you can think of outlet as yield. It's the tag that says "hey, just output everything here." The double brackets tell Handlebars to evaluate the expression inside.

```html
# app/views/home/index.html.erb
<script type='text/x-handlebars'>
  {{ outlet }}
</script>
```

Last but not least, we need to create a Handlebars template for Ember to put into our outlet. Ember looks for an index template by default, so all we need to do is create it:

```haml
/ app/assets/javascripts/templates/index.js.emblem

h1 Hello World
```

Restart your server then visit http://localhost:3000. You should see 'Hello World' printed on the screen. If you see it, congratulations!

[Commit](https://github.com/vicramon/ember-hello-world/commit/2255b0077f85aeb4d5be6cb8aee041667bc62460)

## Basic Debugging

If you don't see the Hello World output then you must have screwed up big time! Just kidding. I'm going to help you figure it out.

Open your console and you should see output that looks like the following:

```
DEBUG: ------------------------------- ember.js?body=1:3522
DEBUG: Ember      : 1.5.0 ember.js?body=1:3522
DEBUG: Handlebars : 1.3.0 ember.js?body=1:3522
DEBUG: jQuery     : 1.11.0 ember.js?body=1:3522
DEBUG: -------------------------------
```

If you don't have the correct version of Ember, try running `bundle update ember-source`. Also make sure you have removed `app/assets/application.js`. If none of this helps, you should look at the Ember Inspector...

## Ember Inspector

The Ember Inspector is an invaluable tool for debugging Ember. It's a Chrome Extension that helps you see what's going on in your app. You can get it [here](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi).

Once it's installed open your console and refresh your browser. You should see a tab titled "Ember". Inside are all sorts of helpful tools. "View Tree" will show you exactly what's being rendered and where it came from. "Routes" show you all the routes in your app. The route you are currently on will be bold. "Data" will show you all the active records in your app. You can click on a record to view all of its attributes.

I can't say enough good things about the Ember Inspector. It makes inner the workings of your app very visible. It's particularly useful for creating routes. By listing your routes it's a sort of `rake routes` for Ember.

## Conclusion

As you've seen it doesn't take much time to get our Ember App up and running. Most of the work is in preparing the Rails app.

If you've still got issues getting this working then please post your issue in the comments below. This app is on [Github](https://github.com/vicramon/ember-hello-world) also if you want to look at it.
