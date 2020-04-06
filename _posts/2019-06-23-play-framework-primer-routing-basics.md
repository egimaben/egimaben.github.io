---
layout: post
title: Play Framework Primer - Routing
comments: true
tags: [scala, play, playframework, sbt, web, application, routing, routes]
--- 
 
This is part 2 of a series about basics of Play framework. In part 1, we scaffolded a play application using the `playframework/play-scala-seed.g8` template and discussed the components in the resulting directory structure.

In this article, we will explore the basics of routing in Play. 
<!--more-->

## What is routing

With any MVC framework like Play, an application follows a pre-defined structure and any component you create falls in one of 3 categories i.e. models, views or controllers.

Models for your domain objects, views for display and user interaction and controllers for business logic that receives input/requests and renders a view to give a response. 

Each controller is a file that contains different units of logic called actions. Let's look at the controller from our [first article](/2019/04/23/play-scala-rest-from-scratch):
{% highlight scala %}
package controllers

import javax.inject._
import play.api._
import play.api.mvc._

@Singleton
class HomeController @Inject()(cc: ControllerComponents) extends AbstractController(cc) {
  def index() = Action { implicit request: Request[AnyContent] =>
    Ok("Hello world")
  }
}
{% endhighlight %}

The method `def index()` is a controller action. We can have an arbitrary number of actions in a controller, conventionally processing logic around a particular model.

Depending on the http request coming in, Play has to be able to map or translate it to the right controller action. 

How we help Play do this is by creating routes which consist of mainly 3 parts: an http Method, a URI pattern and a call definition respectively on a single line, space delimited.

Below is the content of the routes file in our [first article](/2019/04/23/play-scala-rest-from-scratch):

{% highlight bash %}
GET     /                                   controllers.HomeController.index
{% endhighlight %}

As you can see, we are telling Play framework that whenever a `GET` request comes in at relative URL `/` i.e. the root of the page, invoke the `index` action on the `HomeController`.

Also note that the call definition has to be the fully qualified class name of the controller class.

Each http request is seen as an event to react to by Play framework. The global method `Global.onRouteRequest` is the callback method Play invokes to route the request. 

This method ultimately uses a compiled version of the routes file along with Play's in-built router to map request to action. This is a ripe area for customization(future article alert :)).

## The routes file

In a basic Play application, the `app/conf/routes` file is consulted by the play router to know which controller action to invoke when an http request is received.

This file is compiled when the application is building and any errors in it are forwarded directly to the client e.g. browser, curl, postman etc.

Just like in class files, the routes file can contain comments prepended with a pound/hash symbol:
{% highlight bash %}
#this is a comment
GET     /               controllers.HomeController.index
{% endhighlight %}

## Route types

A route declation defines a path the incoming request should follow. Play framework provides different ways we can declare a route to serve both simple and more complex use cases.

### Static path

A static path matches an exact URI pattern. No further logical processing happens when such a path is matched. This means there can only ever be one URL string in the incoming request that can match this path.

Am assuming your project from the [first article](/2019/04/23/play-scala-rest-from-scratch) is running and when you visit `http://localhost:9000`, the browser renders `Hello world`
 
The path we saw in the previous section is such an example. I think being the root path, it may be a bit confusing as `/`. Let's change this such that our URI pattern is `/home`: 
{% highlight bash %}
GET     /home       controllers.HomeController.index
{% endhighlight %}

Now when we save in auto-reloading mode and visit `http://localhost:9000/home`, we should still see `Hello world` rendered. 

Only typing `http://localhost:9000/home` will ever match this route since it's static. Contrasting with the dynamic path in the next subsection will make it clearer in case it is not.

Further more, notice that if we now visit `http://localhost:9000`, we will see an error in the browser like `Action Not Found` since we changed the route declaration.

### Dynamic path, single path segment

Remember how we loaded a config value from the `app/conf/Application.conf` file in the [first article](/2019/04/23/play-scala-rest-from-scratch)? Well that assumed that our config value would not change from request to request.

Now, if we want Play to process the request depending on a value we provide at request time, we can use a dynamic path. For that let us create another controller action to greet people from different planets. New controller code:
{% highlight scala %}
package controllers

import javax.inject._
import play.api._
import play.api.mvc._

@Singleton
class HomeController @Inject()(cc: ControllerComponents) extends AbstractController(cc) {
  def index() = Action { implicit request: Request[AnyContent] =>
    Ok("Hello world")
  }

  def greet(planet: String) = Action { implicit request: Request[AnyContent] =>
    Ok(s"Hello $planet")
  }
}
{% endhighlight %}

Declare the new route in the routes file:
{% highlight bash %}
GET     /home/:planet         controllers.HomeController.greet(planet)
{% endhighlight %}

When you save and visit `http://localhost:9000/home/earth` in the browser, you should now see `Hello earth`. If you replace `earth` in the address bar with `moon`, you should see `Hello moon`.

You can have more than 1 dynamic part in the URI. Let's update the controller with a new method:
{% highlight scala %}
package controllers

import javax.inject._
import play.api._
import play.api.mvc._

@Singleton
class HomeController @Inject()(cc: ControllerComponents) extends AbstractController(cc) {
  def index() = Action { implicit request: Request[AnyContent] =>
    Ok("Hello world")
  }

  def greet(planet: String) = Action { implicit request: Request[AnyContent] =>
    Ok(s"Hello $planet")
  }

  def greetI18n(planet: String, language: String) = Action { implicit request: Request[AnyContent] =>
    Ok(s"Hello $planet in $language")
  }
}
{% endhighlight %}

And update the routes file with a new dynamic route:
{% highlight bash %}
GET     /home/:planet/:language     controllers.HomeController.greetI18n(planet,language)
{% endhighlight %}
Now visit `http://localhost:9000/home/mars/french`, the browser should render `Hello mars in french`.

Likewise you can mix both dynamic and static parts in the URI in any order, e.g. `/home/:planet/english`. 

Just make sure the number of parameters in the action match those coming in or provide a default value for parameters not provided in the URI or which are optional.

### Dynamic path, arbitrary path segments

When you we wanted to add language in the path above, we had to declare a parameter in both action and URI declaration. 

What if, for some arbitrary reason, we now want to allow the user to specify whether American or British in case the language is English but we don't want other users that are used to the API to be affected.

We can define the dynamic part using the `*id` syntax instead of the `:id` syntax. The former accepts any number of path segments. 

For instance, with the following route:
{% highlight bash %}
GET     /home/:planet/:language      controllers.HomeController.greetI18n(planet,language)
{% endhighlight %}
If we visited `http://localhost:9000/home/earth/english/american`, then we would get the `Action Not Found` error rendered in the browser. 

But if we changed the route to:
{% highlight bash %}
GET     /home/:planet/*language      controllers.HomeController.greetI18n(planet,language)
{% endhighlight %}
Then the browser would render `Hello earth in english/american`. This way, we can choose how to process the `english/american` in the action while `Hello mars in french` greetings remain unaffected.

Note that the [Play Documentation](https://www.playframework.com/documentation/2.8.x/ScalaRouting) clearly states that this kind of parameter is not decoded by the router so it's your responsibility to validate and parse it. For example, it should not contain multiple leading slashes or non-ASCII characters.

### Dynamic path, regex

Regex is used to match text. Play provides a way for us to restrict what would be a valid path segment using regex. 

Let's say we only want to greet in 2 languages i.e. english and french, and we want to make sure the user provides the whole word i.e. english and not en, french and not fr as `Hello earth in en` would not make much sense in this context.

We can use the `/$id<regex>` syntax in this case. Let's update our internationalized greeting route to the following:
{% highlight bash %}
GET  /home/:planet/$language<\b(english|french)\b>  controllers.HomeController.greetI18n(planet,language)
{% endhighlight %}

Now if we visit the site with a different language e.g. `http://localhost:9000/home/earth/german` we will get `Action Not Found for request /home/earth/german`, similarly `http://localhost:9000/home/earth/en` will fail.

Only `http://localhost:9000/home/earth/english` and `http://localhost:9000/home/earth/french` will pass.

### Dynamic path, assets

Play, like any useful web framework, gives us an idiomatic way to declare file download paths. This will enable us to download physical files from the server in the public directory.

Remember from the [previous article](/2019/04/23/play-framework-primer-scaffolding), we discussed the public folder and its content. 

As a first example, assume we want to download files from the `app/public/images/` directory, we would declare a route using the `/assets/*id` syntax. 

Remember this syntax enables us to match an arbitrary number of path segments since there could be subdirectories in the `app/public/images/` directory e.g. for `app/public/images/avatars`. You can organize the `app/public` folder the way you prefer:
{% highlight bash %}
GET     /assets/*file    controllers.Assets.versioned(path="/public", file: Asset)
{% endhighlight %}
Remember the syntax for the call definition? yes, Play ships with an inbuilt controller called Assets, which has an action called `versioned` that takes 2 parameters as seen above: a root path which defaults to `/public` and a file of param of type `Asset`.

Now you can download any file in any subdirectory in the `/public` directory e.g. `http://localhost:9000/assets/images/favicon.png`.

## Route Parameters

As we saw in the previous section, you can provide parameters in your URL that are searched for by the router to supply the controller action. We only saw path parameters e.g. `http://localhost:9000/home/earth/english`. 

Let's look at another type of URI parameters in the form of query parameters.

### Query Parameters

Just like path parameters, query parameters are used to send dynamic data from the client to the server. However, query parameters are named in the browser URL but path params are named in the URI path in the route definition.

Query parameters take the form of a query string appended to the path using a `?` sign, each query param is then added in `name=value` form.

Say we want to add language parameter as a query parameter, our URL would be `http://localhost:9000/home/earth?language=english`. To add more query params to the query string, we append the name-value pair with an ampersand (&) e.g. `http://localhost:9000/home/earth?language=english&variant=american`.

Path parameters directly affect the mapping of request to a route. What I mean is that in the previous example, if I loaded `http://localhost:9000/home/earth/english` yet my route is as follows:
{% highlight bash %}
GET     /home/:planet/   controllers.HomeController.greetI18n(planet,language)
{% endhighlight %}
then I would get an `Action Not Found` error because there is an additional path seqment in my request which is not catered for in the route definition.

However, if I the path parameters are valid e.g. `http://localhost:9000/home/earth` and I add a query string e.g. `http://localhost:9000/home/earth?language=english`, it will not block the request from succeeding, even without any change to our code.

There is nothing special to do so as to access query parameters. Take a look at the following route:
{% highlight bash %}
GET     /home    controllers.HomeController.greetI18n(planet,language)
{% endhighlight %}
The call definition has a controller action that takes 2 parameters i.e. `planet` and `language`, so the router will search for these in both the path and query params. Before invoking the action.

So when we load `http://localhost:9000/home?planet=mars&language=english` the browser will render `Hello mars in english`. One important fact is that if a param is provided in both path and query params, then the router will take the one in path.

To demonstrate, strip down the routes file to only have one route:
{% highlight bash %}
GET     /home/:planet   controllers.HomeController.greetI18n(planet,language)
{% endhighlight %}
So `planet` is provided in the URI pattern. Now let's load `http://localhost:9000/home/earth?planet=mars&language=english`. As you can notice, planet is available in both in path and query, but when it loads, the browser renders `Hello earth in english`.

The final thing is that if a parameter is not captured in URI pattern and also does not appear in the query string, then Play will render a `Bad Request` error.

### Typing Parameters

You may have noticed that our parameters in the call definition are not typed:
{% highlight bash %}
GET     /home     controllers.HomeController.greetI18n(planet,language)
{% endhighlight %}
Both `planet` and `language` don't have type information. The play router will default to `String` type. In case you want a specific type, then you may have to declare that in the call definition.

For example, if you added a new action called `sum` which adds the 2 parameters it receives, you'd have to add the types as well like below: 
{% highlight bash %}
GET     /home   controllers.HomeController.greetI18n(planet,language)
GET     /sum    controllers.HomeController.greetI18n(x: Int,y: Int)
{% endhighlight %}

And invoke them as `http://localhost:9000/sum?x=3&y=8`

### Parameters with default values

Sometimes you want to provide a fallback value incase a parameter is not provided, take our internationalized greeting route for example:
{% highlight bash %}
GET     /home   controllers.HomeController.greetI18n(planet,language)
{% endhighlight %}
Here, we have to provide both `planet` and `language` in the query string `http://localhost:9000/home/earth?planet=mars&language=english`.

If we eliminate the language like so `http://localhost:9000/home/earth?planet=mars`, we will get an error `Bad Request For request 'GET /home?planet=mars' [Missing parameter: language]`.

However, in the real world, we may want to greet in english in case the user does not provide the `language` param, in that case, we provide this value in the call definition using the `?=` operator:
{% highlight bash %}
GET     /home   controllers.HomeController.greetI18n(planet,language ?= "english")
{% endhighlight %}

### Parameters with fixed values

In case you want to fix a parameter value such that it disregards any user provided value, then it is possible, similar to providing a default value.

Someone would wonder why we'd do that since if we don't want a user to take a user's input, we could just not pick the value from a parameter but rather from the config.

One scenario I can think of when this could come in handy is if our internationalized greeting was a public API and several clients were already consuming it. 

Assume we wanted to temporarily restrict it only to English perhaps because we are fixing the French language translations. We would not have the luxury to remove it from the param list as this would break all existing clients.
{% highlight bash %}
GET     /home   controllers.HomeController.greetI18n(planet)
{% endhighlight %}
We also don't wan't to just ignore it in code like so:
{% highlight scala %}
def greetI18n(planet: String, language: String) = Action { implicit request: Request[AnyContent] =>
    Ok(s"Hello $planet in english")
  }
{% endhighlight %}
This would work technically but it would cause a lot of confusion especially if there are many developers going to work with the codebase, the confusion is because the error is in routing but we are fixing it in the controller by ignoring a variable.

Secondly unused variables are a code smell.

Why not make the change closer to the error, perhaps with a comment for clarity:
{% highlight bash %}
GET     /home   controllers.HomeController.greetI18n(planet, language = "english")
{% endhighlight %}

Notice that we use `=` operator for fixed parameters

### Optional parameters
We can also declare a parameter as optional by using Scala's `Option` monadic type such that if a user does not provide a value, the clients don't break:
{% highlight bash %}
GET     /home   controllers.HomeController.greetI18n(planet, language: Option[String])
{% endhighlight %}
We also make a similar change in the controller action:
{% highlight scala %}
def greetI18n(planet: String, language: Option[String]) = Action { implicit request: Request[AnyContent] =>
    language match {
      case Some(lang) => Ok(s"Hello $planet in $lang")
      case _ => Ok(s"Hello $planet")   
    }
{% endhighlight %}
This way, we can handle a case where the value is not provided in a smart way.

## Conclusion
In this article, we have explored basics of routing in Play framework. You should be able to leverage routing at this stage to handle most common scenarios in a web application. 

