---
layout: post
title: Play Framework Primer - Scaffolding
comments: true
tags: [scala, play, playframework, sbt, web, application]
--- 
 
In the [previous article](/2019/04/23/play-scala-rest-from-scratch), we created a very stripped down REST application with only one controller end point using Play framework and scala. We assembled it piece by piece while going a little deep into what purpose each component in the final directory structure does. 

<!--more-->

We also learned how to add custom configurations and finally how to import the project into intellij for easier development.
With the bare basics clear, now we will proceed to learn more about play framework. I thought it best to build from the official Play framework template which has more components in the directory structure. 

The example we used in the previous article was assembled from scratch to enable us to use only the critical files/directories without which it would not run. 

By contrast, in the template we will use in this article, there will be more files/directories that may be important but not critical and we will keep diving deeper and swimming wider to learn more about how to build applications with play framework.

This article will form part 1 of a series am working on, all revolving around the basic template [offered](https://www.playframework.com/getting-started) by the wonderful folks building the framework. I intend to cover scaffolding, routing, templating, logging, testing, database connection and more.

 

## Create a play application with sbt

Most web frameworks heavily use build tools to make developers' lives easier, favoring the concept of convention over configuration. 

With a build tool, you get access to commands to enable you do usually tedious tasks like compiling, fetching dependencies, building and testing your application.

Examples include maven, gradle, npm, yarn etc. The most popular build tool used by Play developers is `sbt`. Before that, we used one called `activator` and even before that, `Play`. 

To create a new web application from a template, we use `sbt new <template-name>`. We will use the `playframework/play-scala-seed.g8` template as below:
{% highlight scala %}
sbt new playframework/play-scala-seed.g8
{% endhighlight %}
After this command, you will be prompted for a project name which will default to `play-scala-seed` if you by pass and just press enter. For this example, input `play-scaffold-basic`. 

You will also be asked for organization name which should be your package root by convention. I used `io.egimaben`.

To bypass the prompts and create your application in one command, you can provide param names for them as below:
{% highlight scala %}
sbt new playframework/play-scala-seed.g8 --name=play-scaffold-basic --organization=io.egimaben
{% endhighlight %}
I should mention that the template name should be a valid git repo as `sbt` will have to clone it and seed off that. That means that you can also create your own template/seed/scaffold to suit your needs and probably share with the world

## Run Application

The template is now applied and the project is ready to run. I will not spend time on commands we already covered in the [previous article](/2019/04/23/play-scala-rest-from-scratch).
{% highlight bash %}
cd play-scaffold-basic && sbt run
{% endhighlight %}
When the above commands finish executing, you can visit the page at `http://localhost:9000` or replace `9000` with whichever port the server has provided.

## Directory structure

Now let us take a look at the resulting directory structure:
{% highlight bash %}
|play-scaffold-basic
|____app
| |____controllers
| | |____HomeController.scala
| |____views
| | |____main.scala.html
| | |____index.scala.html
|____test
| |____controllers
| | |____HomeControllerSpec.scala
|____project
| |____build.properties
| |____plugins.sbt
|____public
| |____images
| | |____favicon.png
| |____javascripts
| | |____main.js
| |____stylesheets
| | |____main.css
|____logs
| |____application.log
|____.gitignore
|____conf
| |____logback.xml
| |____messages
| |____application.conf
| |____routes
|____build.sbt
{% endhighlight %}

There are several files and directories that were not present in the stripped down example we used in the [previous article](/2019/04/23/play-scala-rest-from-scratch), we will focus on these.

We can immediately see that this is a more complete web application than the previous with tests, a log file, more configuration options plus more. 
Let's explore some of these in more depth.

### Views
The template we used provides a near complete MVC (model-view-controller) style example. In our example there is no `models` directory but there is now both `controllers` and `views`. 

`models` should be in the same hierarchy under `app` but the simplicity of this template does not warrant its presence.
The `views` directory is where `html` style templates live and they power the front end of our application. 

They interact seamlessly with the controllers using scala as we will see with more detail in a later part of this series.

### Test
The `test` directory is where all automated tests you write will reside. Conventionally, the package and file names of the files you are testing should be reflected in the `tests` directory. 

In our project, we only have one controller called `HomeController` which is in the default package i.e. no package name. So its test file is called `HomeControllerSpec`. 

I personally prefer to name it `HomeControllerTest`.
If for example, our controller was in a package like `io.egimaben.HomeController`, my test file would be `io.egimaben.HomeControllerTest`. This kind of organization brings more clarity to a project.

### Public
This directory contains your front end resources i.e. images, javascript and css. These are used by the views.

### Logs
Application log files reside in the `logs` directory. We will see later how to do more customizations fit for a production grade application.

### Conf
The configuration directory was covered in the [previous article](/2019/04/23/play-scala-rest-from-scratch) but since there are more files in this case than before, we will discuss abit about them.

Play will use `logback.xml` as the single source of truth for logging configuration. We will go into further details in a later part of the series. 

By default, the file looks like below and this defines the default state of config used by Play:
{% highlight xml %}
<configuration>

  <conversionRule conversionWord="coloredLevel" converterClass="play.api.libs.logback.ColoredLevel" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>${application.home:-.}/logs/application.log</file>
    <encoder>
      <charset>UTF-8</charset>
      <pattern>
        %d{yyyy-MM-dd HH:mm:ss} %highlight(%-5level) %cyan(%logger{36}) %magenta(%X{akkaSource}) %msg%n
      </pattern>
    </encoder>
  </appender>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <withJansi>true</withJansi>
    <encoder>
      <charset>UTF-8</charset>
      <pattern>
        %d{yyyy-MM-dd HH:mm:ss} %highlight(%-5level) %cyan(%logger{36}) %magenta(%X{akkaSource}) %msg%n
      </pattern>
    </encoder>
  </appender>

  <appender name="ASYNCFILE" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="FILE" />
  </appender>

  <appender name="ASYNCSTDOUT" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="STDOUT" />
  </appender>

  <logger name="play" level="INFO" />
  <logger name="application" level="DEBUG" />

  <root level="WARN">
    <!--<appender-ref ref="ASYNCFILE" />-->
    <appender-ref ref="ASYNCSTDOUT" />
  </root>

</configuration>
{% endhighlight %}

The `messages` file is used for internationalization and multi-language support as we will see in another article.

## Conclusion
In this article, we scaffolded a working play application using an existing template. It extends the [previous article](/2019/04/23/play-scala-rest-from-scratch) where we built an application from scratch. 

Here, we looked at a more fleshy web app with more of features one would typically need in a real production application.
As I said at the beginning, this is part 1 of a series, next, we will cover routing.
