---
layout: post
title: Play 2.7 Scala REST API from scratch With SBT
comments: true
--- 
 
With the advent of modern web frameworks such as Play, Laravel/PHP, Express/Node, advanced tools have come up to help in scaffolding ready-to-run Web Apps with a single command. In some cases, like with Play, there are even [fully scaffolded examples](https://developer.lightbend.com/start/?group=play) to download and run.

 <!--more-->

Now this is really great as most programmers prefer to just copy and paste already working code, but if you are anything like me, understanding something at a fundamental level and being able build it from scratch can give you much more control over your work.

I downloaded one of the already scaffolded [play examples]((https://developer.lightbend.com/start/?group=play)) and there were so many files and folders most of which I thought were redundant and cause mental clutter and cognitive overload for nothing. But to confirm this, I had to create one from scratch with the bare minimum set of files and directories that could work.

## Requirements
Java 8 must be installed. To confirm this, check the version from your terminal or command prompt:
{% highlight bash %}
MacBook-Pro-9% java -version
java version "1.8.0_172"
Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)
{% endhighlight %}

I am using MacOS but the same should work on Linux and even Windows.
Install `sbt` from the [official website](https://www.scala-sbt.org/). To confirm, run the following command and you should get a similar output:

{% highlight bash %}
MacBook-Pro-9% sbt sbtVersion
[info] Loading global plugins from /Users/benedictayiko/.sbt/1.0/plugins
[info] Loading project definition from /Users/benedictayiko/project
[info] Set current project to benedictayiko (in build file:/Users/benedictayiko/)
[info] 1.2.8
{% endhighlight %}

I am not going to use an IDE for this demo but I would recommend [IntelliJ iDea](https://www.jetbrains.com/idea/download/) which has a community edition. Ensure that you install the [Scala plugin](https://www.jetbrains.com/help/idea/managing-plugins.html).

## Creating The Application

### Overview

Navigate to your preferred location in the file system and create a new directory to hold your project files:
{% highlight bash %}
mkdir PlayFromScratch && cd PlayFromScratch
{% endhighlight %}
As we create the project files and directories, it would help to keep referring to the official [anatomy of a play application](https://www.playframework.com/documentation/2.7.x/Anatomy) to see what they signify if you don't already know.

In summary, the most important files and folders are as below. Of course the controller may vary depending on your needs but the rest of the files are for application config and build system config, so without them, the build won't succeed:
{% highlight bash %}
|_PlayFromScratch
|____app
| |____controllers
| | |____HomeController.scala
|____project
| |____build.properties
| |____plugins.sbt
|____conf
| |____application.conf
| |____routes
|____build.sbt
{% endhighlight %}

### App directory

Let's create the `app` directory for all compilable source code, much as we will only create a controller. There could be a directory for `models` as well as `views`:
{% highlight bash %}
mkdir app && cd app
mkdir contollers && cd controllers
{% endhighlight %}
Let's create the `HomeController` controller:

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

In my opinion, this is the most basic controller we can have in Play with a single action.

### Project definition directory
Next we will add `build.properties` and `plugins.sbt`. Back in the root of the app where `app` directory is located, let's create `project` directory:

{% highlight bash %}
mkdir project && cd project
{% endhighlight %}

Let's create `build.properties` with the following content. This file forms part of your build definition where you specify the version of sbt that your build uses. This allows people with different versions of the sbt launcher to build the same projects with consistent results:

{% highlight bash %}
sbt.version=1.2.8
{% endhighlight %}

The other part of your build definition is `plugins.sbt` which contains plugin definitions that will be used in the build, in a way, plugins extend the build... for our case, we need to add the `play` plugin:

{% highlight bash %}
addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.7.0")
{% endhighlight %}

You can get more details about [plugins](https://www.scala-sbt.org/1.0/docs/Using-Plugins.html) and [build definitions](https://www.scala-sbt.org/1.x/docs/Basic-Def.html).

### Configuration directory

Next we will create `conf` directory which at the minimum contains the application wide configs inside `application.conf` or `application.json` without which the build will fail with an `IO` exception(I know it should be more graceful than that). So by all means create it even if you don't have configs. 

The second file is the `routes` file which helps the `HTTP Server` map routes to controller actions.

Let's navigate back to project root where we now have `app` and `project` directories. Create `conf` directory:

{% highlight bash %}
mkdir conf && cd conf
{% endhighlight %}

Next, we create `application.conf`, currently we don't have any configs, so we will leave it empty

{% highlight bash %}
//application wide configs go here
{% endhighlight %}

Then create a `routes` file, directing base url traffic to our `index` action in the controller

{% highlight bash %}
GET     /                                   controllers.HomeController.index
{% endhighlight %}

### Build Definition
Finally, at the root directory, we will create the most important file for `sbt` to build our project, it is the main build definition. Create `build.sbt` with the following content:

{% highlight scala %}
name := """play-scala-seed"""
organization := "io.egimaben"

version := "1.0-SNAPSHOT"

lazy val root = (project in file(".")).enablePlugins(PlayScala)

scalaVersion := "2.12.8"

libraryDependencies += guice
{% endhighlight %}

The definitive guide to understanding the config items above is the `sbt` docs located [here](https://www.scala-sbt.org/1.x/docs/Basic-Def.html). 

## Running the application
From the root of the project, run `sbt` to launch the sbt console with your build configuration from where you can run any of several commands e.g. `compile`, `clean` and `run`. Or you can run the command straight by appending it right away e.g. `sbt run`.

So let's just go with `sbt run`. If it's your first time to run `sbt`, it's likely to take sometime downloading dependencies. But your console should ultimately look like this:

{% highlight bash %}
MacBook-Pro-9% sbt run
[info] Loading global plugins from /Users/benedictayiko/.sbt/1.0/plugins
[info] Loading settings for project minimalplay-build from plugins.sbt ...
[info] Loading project definition from /Users/benedictayiko/play/MinimalPlay/project
[info] Loading settings for project root from build.sbt ...
....//several downloads
[info] Set current project to play-scala-seed (in build file:/Users/benedictayiko/play/MinimalPlay/)
[info] sbt server started at local:///Users/benedictayiko/.sbt/1.0/server/1210f6650449d35a6be3/sock
[info] Updating ...
[info] Done updating.
[warn] There may be incompatibilities among your library dependencies; run 'evicted' to see detailed eviction warnings.

--- (Running the application, auto-reloading is enabled) ---

[info] p.c.s.AkkaHttpServer - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Enter to stop and go back to the console...)
{% endhighlight %}

When you visit `http://localhost:9000`, then your browser should display `Hello World!` since that is what we return in our only controller action.

## Adding a configuration
I thought it would be interesting to add a configuration and read it from the controller. Inside `application.conf` add a config, the syntax being key value pairs separated by equals sign, like so:
{% highlight bash %}
world = Earth
{% endhighlight %}

In the controller, instead of hardcoding `World`, we want to pick the value from the config file. Play provides a configuration API which loads configs in the background and we can request a dependency to access this API, change the controller declaration so the code is as below:

{% highlight scala %}
package controllers

import javax.inject._
import play.api._
import play.api.mvc._

@Singleton
class HomeController @Inject()(config: Configuration, cc: ControllerComponents) extends AbstractController(cc) {
  def index() = Action { implicit request: Request[AnyContent] =>
    val world = config.get[String]("world")
    Ok(s"Hello $world!")
  }
}
{% endhighlight %}

Notice we have injected `Configuration` API. Reload the browser the you should see `Hello Earth!`

You can read more about the [configuration API](https://www.playframework.com/documentation/2.7.x/ScalaConfig) and also the [syntax](https://www.playframework.com/documentation/2.7.x/ConfigFile) of the `application.conf` config file.

## Continuing developing from intelliJ

Since we have setup a basic structure of a play/scala application hopefully with much understanding, most of which we have done from the console, it is now time to take it to the next level. This will be easier with an IDE such as intelliJ.

Since from an earlier section, we already talked about setting up intelliJ, all you have to do now is to import the project(`File` -> `Open project` then navigate to root directory) into intelliJ. If you create the project from intelliJ, you may run into dependency resolution [issues](https://stackoverflow.com/questions/27063562/cannot-resolve-symbol-routes) and more [issues](https://stackoverflow.com/questions/21577573/intellij-idea-can-not-resolve-symbol-with-play-framework/47632950).

I will write more articles that build on this minimal application introducing new concepts to help us gain more mastery with the Play/Scala stack.
