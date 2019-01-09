---
layout: post
title: Java Method References
comments: true
---


<div class="message">
  A double colon might appear so innocently, but there's a lot of things to know about it. In this post, we try to put together the most important of them.
</div>

In this post, we are going to discuss yet another feature of Java 8: Method References. In a previous post, we explored lambda expressions and learned how to use them to write better and more compact Java code, especially with the advent of functional interfaces.

Unless you are very well-versed with lambda expressions, I would highly recommend that you first go through my lambda expressions tutorial before continuing with this post.

To even encourage you further, when you understand lambda expressions, method references are a walk over.
<!--more-->
## Method References vs. Lambda Rxpressions

From the lambda expressions tutorial, we can remember that a lambda expression simply gives us a more compact way of writing an anonymous function or implementing a functional interface.

In summary, we discussed that, if we have an interface such as below for printing something and a method that takes the interface as a parameter:

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

Pre-Java 8, we can call the printSomething method and provide an implementation of Printer interface as an anonymous function:

{% highlight java %}
public static void main(String[] args) {
    LambdaDemo demo = new LambdaDemo();
    String something = "I am Lambda";
    demo.printSomething(something, new Printer() {
        @Override
        public void print(String val) {
            System.out.println(val);
        }
    });
}
{% endhighlight %}

The above code, when run, produces the desirable result i.e. printing something to the console:
{% highlight java %}
I am Lambda
{% endhighlight %}

However, as Java developers, aware of new features in Java 8, we found this too verbose and decided to apply a lambda expression:

{% highlight java %}
public static void main(String[] args) {
    LambdaDemo demo = new LambdaDemo();
    String something = "I am Lambda";
    demo.printSomething(something, (toPrint) -> System.out.println(toPrint));
}
{% endhighlight %}

We were able to reduce the number of lines significantly. Notice that, we are simply using a single line of code in the lambda expression i.e. invoking only a single method. But we are free to add more lines and wrap them within braces:

{% highlight java %}
public static void main(String[] args) {
    LambdaDemo demo = new LambdaDemo();
    String something = "I am Lambda";
    demo.printSomething(something, (toPrint) -> {
            System.out.println("We are testing lambda expressions");
            //more lines of code
            System.out.println(toPrint);
    });
}
{% endhighlight %}

Which prints:

{% highlight java %}
We are testing lambda expressions
I am Lambda
{% endhighlight %}

To use a lambda expression like we have done in the last example is awesome as it saves us from the verbosity of anonymous functions. However, the first case where we were only calling a single method is not the most compact code we can get…ENTER METHOD REFERENCES:

{% highlight java %}
public static void main(String[] args) {
    LambdaDemo demo = new LambdaDemo();
    String something = "I am Lambda";
    demo.printSomething(something, System.out::println);
}
{% endhighlight %}

A method reference can replace a lambda expression where we only have a single method call. Instead of the method call, we simply supply the method name attached to its class/object separated by a double colon(::) operator.

The real invocation of is done under the hood and we don’t even need to directly supply parameters (e.g. toPrint) as we did for lambdas, the compiler infers this information automatically and more such as return type.

More formally, we can say that:
>A method reference is a more compact and easy-to-read construct to replace a lambda expression that does nothing more than invoking an existing method.

## Static Method Reference

One of the more obvious and easiest ways to use a method reference is where static methods are involved. Assuming we have a class that only defines a single static method for adding two integers:

{% highlight java %}
public class MRefDemo {
    public static int add(int a, int b) {
        return a + b;
    }
}
{% endhighlight %}

We can write a unit test for this class using a lambda expression if we don’t know about method references:

{% highlight java %}
@Test
public void givenStaticFn_createsFunctionalInterfaceWithLambda_pass() {
    BiFunction<Integer, Integer, Integer> adder = (a, b) -> add(a, b);
    int addResult = adder.apply(5, 6);
    assertEquals(11, addResult);
}
{% endhighlight %}

We have to explicitly reference the parameters in the lambda expression wrapped by the BiFunction. But with method refs, we don’t need any of that:

{% highlight java %}
@Test
public void givenStaticFn_createsFunctionalInterfaceWithMethRef_pass() {
    BiFunction<Integer, Integer, Integer> adder = MRefDemo::add;
    int addResult = adder.apply(5, 6);
    assertEquals(11, addResult);
}
{% endhighlight %}

Notice the syntax for static method reference: the class name, MRefDemo and the method name, add separated by the double colon(::) operator.

## Constructor Reference

Constructor reference is very similar to static method reference. It is a way to supply the name of the constructor of a class without invoking it. The invocation takes place under the hood.

Assume that we have a class called LogText.java, whose soul purpose in life is to wrap some raw text and avail it at log time in a beatiful way such that instead of our log appearing like this:

{% highlight js %}
server0
{% endhighlight %}

It would appear like:

{% highlight js %}
==server0==
{% endhighlight %}

Simple, but serves our demo purpose.

Now here is the content of LogText.java:

{% highlight java %}
class LogText {
    private String s;
    public LogText(String s) {
        this.s = "==" + s + "==";
    }
    @Override
    public String toString() {
        return s;
    }
}
{% endhighlight %}

The constructor only beatifies the given string and stores it ready for retrieval at log time. Now we want to wrap this class in a functional interface such that a call to its functional method constructs an instance of LogText with the given string.

The lambda approach:

{% highlight java %}
@Test
public void givenConstructor_createsFunInterfaceWithLambda_pass() {
    Function<String, LogText> beautifier = (s) -> new LogText(s);
    LogText log = beautifier.apply("server0");
    assertEquals("==server0==", log.toString());
}
{% endhighlight %}

And the constructor reference option:

{% highlight java %}
@Test
public void givenConstructor_createsFunInterfaceWithMethRef_pass() {
    Function<String, LogText> beautifier = LogText::new;
    LogText log = beautifier.apply("server0");
    assertEquals("==server0==", log.toString());
}
{% endhighlight %}

## Bound Instance Method Reference
We have only looked at uses of method reference where we are not instantiating the object ourselves. We can also apply a method reference to name a method attached to an already constructed object.

Assuming that we want to convert the above constructor reference example to use bound instance method reference. We would have to construct our LogText instance in advance.

For this example to make sense, let us also assume that we now want the functional interface to supply us directly with the beautified string rather than an instance of LogText. The lambda example:

{% highlight java %}
@Test
public void givenObj_createsFunInterfaceWithLambda_pass() {
    LogText beautifulText = new LogText("server0");
    Supplier<String> beautifier = () -> beautifulText.toString();
    String log = beautifier.get();
    assertEquals("==server0==", log);
}
{% endhighlight %}

And the method reference variant:

{% highlight java %}
@Test
public void givenObj_createsFunInterfaceWithMethRef_pass() {
    LogText beautifulText = new LogText("server0");
    Supplier<String> beautifier = beautifulText::toString;
    String log = beautifier.get();
    assertEquals("==server0==", log);
}
{% endhighlight %}

## Unbound Instance Method Reference

Unbound instance method reference or unbound non-static method reference is very close in both syntax and even closer in semantics to bound instance method reference.

In the section on bound instance method reference above, we were telling the compiler:

`Here is our object, it is the only thing we have to offer you, it is ready and contains everything we need, we only want you to invoke a certain method called toString on this object and return to us whatever that call returns.`

That is why we had to use a Supplier, a functional interface whose functional method takes no arguments and returns a single value. We did not have anything inform of an argument as we only had a fully built LogText object.

Now take a look at an example of unbound instance method reference:

{% highlight java %}
@Test
public void givenObj_createsFunInterfaceWithLambda_pass2() {
    Function<LogText, String> beautifier = (logText) -> logText.toString();
    LogText beautifulText = new LogText("server0");
    String log = beautifier.apply(beautifulText);
    assertEquals("==server0==", log);
}
{% endhighlight %}

The concept is not clear with a lambda, let’s look at the method reference approach:

{% highlight java %}
@Test
public void givenObj_createsFunInterfaceWithMethRef_pass2() {
    Function<LogText, String> beautifier = LogText::toString;
    LogText beautifulText = new LogText("server0");
    String log = beautifier.apply(beautifulText);
    assertEquals("==server0==", log);
}
{% endhighlight%}

Now this is exactly what we are telling the compiler:

`Hey dude, see we don’t really have a ready object at this point, we will have it ready very soon. But in the meantime we want you to take note of a certain method called toString. When we eventually get around to creating this object, please remember to call the said method on this object and return to us whatever that call returns.`

This is why we are using a Function. A functional interface whose functional method takes one argument and produces a result. We will have to pass the LogText object as an argument to the functional method.

The syntax `Class::instanceMethod` is strange and this is where part of the confusion comes from. That syntax is common with static methods. It is like calling String.toString yet we all know it should be called on some string s, like s.toString.

Therefore, understanding this difference is key to correct and informed use of method references.

## Conclusion

In this tutorial, we have explored one of the features of Java 8. Method references can make your code more compact and easier to read.

Just remember that whenever you are going use a lambda expression, first ask yourself if the lambda expression is evaluating a single known function. If so, it is high time to replace that with a method reference.

Finally, endeavor to clearly understand the difference between bounded and unbounded instance/non-static method references. I have tried to clear that in this tutorial as well.
