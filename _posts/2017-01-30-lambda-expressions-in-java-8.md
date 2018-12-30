---
layout: post
title: Lambda Expressions in Java 8
---


<div class="message">
  In this post, we will learn what, exactly, lambda expressions are and how they fit into the whole Java ecosystem.
</div>

Lambda expressions are the most popular feature of Java 8. They introduce functional programming concepts to Java, which is a completely object-oriented and imperative programming language. 

How functional programming languages work is beyond the scope of this article, but we will extract a feature that will make the difference obvious for us who work with OOP.

How functional programming languages work is beyond the scope of this article, but we will extract a feature that will make the difference obvious to us who work with OOP.

In this post, we will learn what, exactly, lambda expressions are and how they fit into the whole Java ecosystem. We will also look at example code that works without using lambda expressions and then refactor this code to use lambdas.

## Understanding a Lambda Expression

A Lambda expression is a block of code that we can pass around to execute. Passing a block of code to a function is something that we as Java programmers are not used to. 

All our behavior defining code is encapsulated inside method bodies and executed through object references as it is with this code:

{% highlight java %}
public class LambdaDemo {
    public void printSomething(String something) {
        System.out.println(something);
    }
    public static void main(String[] args) {
        LambdaDemo demo = new LambdaDemo();
        String something = "I am learning Lambda";
        demo.printSomething(something);
    }
}
{% endhighlight %}

This is classic OOP style of hiding method implementations from the caller. The caller simply passes a variable to the method which then does something with the value of the variable and returns another value or produces a side effect as it is in our case.

We are now going to see an equivalent implementation that uses behavior passing other than variable passing. To achieve this, we have to create a functional interface that defines that abstracts the behavior instead of a method. 

A functional interface is an interface that has only one method:

{% highlight java %}
public class LambdaDemo {
    interface Printer {
        void print(String val);
    }
    public void printSomething(String something, Printer printer) {
        printer.print(something);
    }
}
{% endhighlight %}

In the above implementation, the `Printer` interface is responsible for all printing. The method `printSomething` no longer defines the behavior but rather executes one defined by `Printer`:

{% highlight java %}
public static void main(String[] args) {
    LambdaDemo demo = new LambdaDemo();
    String something = "I am using a Functional interface";
    Printer printer = new Printer() {
        @Override
        public void print(String val) {
            System.out.println(val);
        }
    };
    demo.printSomething(something, printer);
}
{% endhighlight %}

We may have noticed we have not done something new here. This is true because we have not applied a lambda expression yet. We simply created a concrete implementation of the Printer interface and passed it to the printSomething method.

The above demo was meant to bring us to a key objective of introducing Lambda expressions in Java:
>Lambda expressions are used primarily to define inline implementation of a functional interface.
Before we refactor the above example using lambda expressions, let’s learn the necessary syntax:
{% highlight java %}
(param1,param2,param3...,paramN) - > {//block of code;}
{% endhighlight %}

A lambda constitutes a comma separated list of formal parameters enclosed in parenthesis as we would define in a method declaration, followed by an arrow token pointing to the code to execute. Now let us refactor the above code to use a lambda:

{% highlight java %}
public static void main(String[] args) {
    LambdaDemo demo = new LambdaDemo();
    String something = "I am learning Lambda";
    /**/
    Printer printer = (String toPrint)->{System.out.println(toPrint);};
    /**/
    demo.printSomething(something, printer);
}
{% endhighlight %}

I find the above snippet very compact and beautiful to look at. Since the functional interface declares only a single method, the parameters passed in the first part of the lambda are automatically mapped to the method’s parameter list and the code on the right of the arrow is treated as the method’s concrete implementation

## Why Use Lambda Expressions?

As pertains the demo in the previous section, lambda expressions enable us to have more compact code, easier to read and follow. There are other benefits in line with performance and multi-core processing, but they can only make sense after understanding the Streams API, therefore beyond the scope of this article.

Comparing the `main` method implementations with and without lambdas sure shows us the power of lambda expressions when it comes to shortening code:

{% highlight java %}
public static void main(String[] args) {
    LambdaDemo demo = new LambdaDemo();
    String something = "I am learning Lambda";
    /**/
    Printer printer = (String toPrint)->{System.out.println(toPrint);};
    /**/
    demo.printSomething(something, printer);
}
{% endhighlight %}

We can make our code more concise than this. It so happens that even without specifying the type of the parameter on the left part of the arrow, its type will be inferred by the compiler from the formal params of the interface method:

{% highlight java %}
Printer printer = (toPrint)->{System.out.println(toPrint);};
{% endhighlight %}

We can still do better. Another characteristic of lambdas is that: if there is only one parameter, we can eliminate the parenthesis altogether. Similarly, if there is only one statement on the right of the arrow, we can also eliminate the braces:
{% highlight java %}
Printer printer = toPrint -> System.out.println(toPrint);
{% endhighlight %}

Now our code is really starting to look cute. We are just getting started. If our interface method did not take any parameters, we could replace the declaration with empty parenthesis:

{% highlight java %}
() -> System.out.println("anything");
{% endhighlight%}

How about we just inline the lambda without first creating an object and then passing it to the saySomething method:

{% highlight java %}
public static void main(String[] args) {
    LambdaDemo demo = new LambdaDemo();
    String something="I am Lambda";
    /**/
    demo.printSomething(something, toPrint -> System.out.println(toPrint));
}
{% endhighlight %}

Now we are really starting to talk functional programming. Our initial main body of nine lines is now down to only 3 lines. This compactness of code is what makes lambda expressions very appealing to Java programmers.

## Conclusion

In this post, we have gone through a gentle introduction to Lambda expressions in Java and learned how they can be used to improve the code quality of functional interface implementations. 

Watch out for more coverage of Lambdas on this site as I will be touching the Stream API and discussing how it gives us more advantages of Lambdas when used together with the Collections framework.