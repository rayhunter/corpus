# Corpus

Corpus.js is a Javascript framework written entirely in CoffeeScript that makes it easy to develop large client side web applications without too much overhead.

## Features

* Light-weight URL Routing. (Does not support hashbang urls.. yet)
* Modular views
* Conventions for maintaining views
* Models for persisting state
* Model bindings
* Written entirely in coffeescript
* Currently developed with a RESTful ruby on rails backend in mind, but can be expanded for others.

## Why not backbone, sproutcore, Sammy, etc?

* Lighter weight than sproutcore 
* More modular than backbone (you can just take the model pieces if you want and ignore the controller/routes)
* Doesn't have a view object like backbone, but merely enforces a sane and simple convention
* Models have association support within a request. (backbone lacks this)
* Models can bind to the changes events of form fields (backbone lacks this)
* Written entirely in coffeescript. Most of the big client side microframeworks are written in javascript

## Dependencies

* JQuery 1.4+ (http://docs.jquery.com/Downloading_jQuery)
* Underscore (http://documentcloud.github.com/underscore/)
* JSON2 (https://github.com/douglascrockford/JSON-js)

## Routing/Controllers

You can choose to use corpus routing or not. Presently it's only purpose is to load specific views/models when a specific page is loaded. This is helpful for consolidating behavior per page rather than having all of your JQuery selectors running on every page. 

### Example (posts.coffee)

```coffeescript
$ -> 
  # Only runs the PostView.Form view when the /posts/new or /posts/:id/edit urls are requested
  _.each ["/posts/new", "/posts/:id/edit"], (route) ->
    $.R route, (id) ->
      post = new Post()
      new PostView.Form(post)

  # Only runs the PostView.Index view when the /posts url is requested
  $.R "/posts", -> new PostView.Index(post)
```

## Models

To maintain state in the browser, Corpus provides a light-weight model layer that is similar to the popular server-side framework Ruby on Rails. Some of the features it provides are:

* Key, value attributes management (managed attribute changes too)
* Record UID's as well as database ID management
* Associations
* Persistence with a RESTful backend
* Event system
* Collections

### Example 

#### post.coffee

```coffeescript
this.Post = Model "post"
Post.hasMany "comments"
Post.hasOne "user"

_.extend Post.prototype,
  initialize: (attributes) ->
    this.bind "save:before", this.onBeforeSave

  onBeforeSave: ->
    # do something

  createdAt: ->
    this.get("created_at")

  authorName: ->
    this.user.name()
```

#### comment.coffee

```coffeescript
this.Comment = Model "comment"
```

This allows you to do the following:

```coffeescript
post = new Post
post.set("body", "Lorem ipsum...")
comment = new Comment(body: "Some Comment")
post.comments.add comment
post.save
  success:
    # Do whatever
  error:
    # Validation errors
```

The json will look like:

```javascript
{ 
  body: "Lorem Ipsum",
  comments: [
    body: "Lorem Ipsum"
  ]
}
```

### Binding to events

```coffeescript
post.comments.bind "add", ->
  # Will run code after a comment is added to the collection

post.comments.bind "refresh", ->
  # Will run code after the collection is changed

post.bind "change", ->
  # Will run when any of the attributes on a post have changed
```

### Binding to forms

```coffeescript

# Will automatically bind change events from fields in this form and update the post record. 
# Works with associations too!
post.bindTo "#post_form"
```


## Views:

There is no special constant/class to extend for views. We merely use a convention at this point. In the future we may offer a constant/class you can extend for additional assistance but for now, the convention we've adopted has worked pretty well.

### Example:

```coffeescript
this.PostView.Form = (post) ->
  view = this
  $("#post_form").submit ->
    self = this
    post.save
      success: (respPost) ->
        if post.isNew
          post = respPost
        if $(self).data("redirect")
          document.location.href = $(self).data("redirect")
        $(self).trigger "saved"
      error: (post) ->
        $("#error_explanation ul").html("")
        messages = _.each post.errors(), (message, value) ->
          $("#error_explanation ul").append("<li>" + value + " " + message + "</li>");
        $("#error_explanation").show()
```

## Recommended project directory structure using Barista + Ruby on Rails:

If you're using jammit with barista, you should use something like this:

```
app/coffeescripts/controllers/     # all of your controllers go here
app/coffeescripts/models/          # all of your models go here
app/coffeescripts/views/*/**       # all of your views go here
app/coffeescripts/vendor/          # all of your vendor files go in here. If you use only model.coffee for example, you can stick that file here.
app/coffeescripts/lib/             # all if your library files like underscore.string.coffee go here
```

If you're using barista, this should generate each of the above directories inside of public/javascripts/.

## Recommended project directory structure using Rails 3:

```
app/assets/javascripts/controllers/     # all of your controllers go here
app/assets/javascripts/models/          # all of your models go here
app/assets/javascripts/views/*/**       # all of your views go here
app/assets/javascripts/vendor/          # all of your vendor files like model.coffee and route.coffee go here
app/assets/javascripts/lib/             # all if your library files like underscore.inflection.coffee go here
```