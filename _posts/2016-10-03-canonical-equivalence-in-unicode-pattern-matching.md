---
layout: post
title: Canonical Equivalence In Unicode Pattern Matching
comments: true
---


<div class="message">
 In this article, we will define and explain the term Canonical Equivalence as applied to pattern matching according to the Unicode character specification.
</div>

In this article, we will define and explain the term Canonical Equivalence as applied to pattern matching according to the Unicode character specification.

Pattern matching is one of the most common concepts in computer science. Take a look at the Wikipedia definition:

<!--more-->

>In computer science, pattern matching is the act of checking a given sequence of tokens for the presence of the constituents of some pattern. In contrast to pattern recognition, the match usually has to be exact. The patterns generally have the form of either sequences or tree structures.

Notice how it differs from pattern recognition, the word EXACT is key here. Canonical Equivalence is the term used in the [Unicode Equivalence specification](https://en.wikipedia.org/wiki/Unicode_equivalence) of the Unicode character encoding system to emphasize the EXACTing nature of its matching system.

## <em>Canonical</em> Explained

Wikipedia defines Canonicalization as:
>A process for converting data that has more than one possible representation into a “standard” canonical representation.

In this context, the term Canonical means to have many forms or representations which must all boil down to a single standard value which is hereby known as the canonical value.

Another common example in programming to help us understand canonical representation is to look at the XML schema datatype definition of “boolean”:

The “lexical representation” of boolean can be one of: true, false, 1, 0.

On the other hand, the “canonical representation” can only be one of: true, false.

This means that at the end of it all, every accepted boolean value must be able to map to its canonical equivalent:

1 – > true

0 – > false.

Another useful way to look at the word canonical is in theological terms where it refers to the real truth that has been and will always be the real truth regardless of any layers of information added to it.

In Java, when we compare two objects:
{% highlight java %}
a.equals(b)
{% endhighlight %}

The JVM will compare them based on different aspects such as hashCode and attribute values to establish their sameness with each other. If there is even one aspect that differs, then the test will fail. If you are sure of their sameness, however, and want a quick test, then you are better off using the == operator:

{% highlight java %}
a==b
{% endhighlight %}

This comparison is faster because it only bases on the one **canonical truth** that is used to judge the sameness of the two objects and that is the object id.

## Canonical Equivalence In Unicode

Having looked at the term Canonical in depth, it is now easy to conclude that two different characters are canonically equivalent if, and only if what is considered their canonical value is the same.

In file systems, this can be the absolute path to a file which will work regardless of the present working directory.

Finally, in Unicode terms, canonical equivalence is a fundamental equivalence between individual Unicode characters and sequences of Unicode characters.

Let’s take a look at an example:

`In Unicode, there is a character e, whose code point is u0065. However, Unicode also has a composite character é which is the same character e with an acute accent superimposed on top of it. The code point for the accented e is u00E9. However, the accent character on top of the composite character also has a Unicode code point, u0301. This means that the composite character u00E9 is canonically equivalent to the character sequence u0065 u0301.`

## Canonical Equivalence In Java

The concept of Canonical equivalence is encountered in a number of areas in Java programming language. Knowledge of this will save you hair loss. We looked at one example earlier with Object comparison. We can look at some more.

### Java Reflection

When we are inspecting the different aspects of a Java class using reflection, we can retrieve the class name of an object in different ways. Consider an empty class called Person.java:

{% highlight java %}
package com.egima;
public class Person {
    private String name;
    private int age;
    public Person() {
        super();
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
{% endhighlight %}

Let’s run some inspection on this class:

{% highlight java %}
public static void main(String[] args) {
    Person ob = new Person();
    Class<?> clazz = ob.getClass();
    String name = clazz.getSimpleName();
    String canonicalName = clazz.getCanonicalName();
    System.out.println("Simple Class Name: " + name);
    System.out.println("Canonical Class name= " + canonicalName);
}
{% endhighlight %}

Console output:

{% highlight java %}
Simple Class Name: Person
Canonical Class name= com.egima.Person
{% endhighlight %}

Notice that if we don’t really care much about the detailed truth about the class we call the `getSimpleName` method. We can easily dispute this by having another `Person.java` class in a different package.

The JVM is so wise that when you instantiate a class object reflectively, you will only get success when your class name can be traced through an explicitely defined class hierarchy. And this is only possible with the canonical name.

The following code will fail and throw a `ClassNotFoundException`:

The lambda approach:

{% highlight java %}
public static void main(String[] args) {
    Class<?> clazz = Class.forName("Person");
    Person person = (Person) clazz.newInstance();
}
{% endhighlight %}

While this one will pass:

{% highlight java %}
public static void main(String[] args) {
    Class<?> clazz = Class.forName("com.egima.Person");
    Person person = (Person) clazz.newInstance();
}
{% endhighlight %}

## Java Regex
When searching Unicode characters with Java Regular Expression API, we must be very informed about the notion of Canonical Equivalence or else we will waste a lot of time and effort with little success.

Let’s consider our earlier example of accented character `é` . We concluded that it’s unicode code points can be represented as a composite: `u00E9` or a sequence:  `u0065 u0301`.

Now let us see this in action:

{% highlight java %}
public static void main(String[] args) {
    Pattern pattern = Pattern.compile("\u00E9");
    Matcher matcher = pattern.matcher("Text with unicode accented character \u0065\u0301");
    boolean found = matcher.find();
    System.out.println("Was the character found: " + found);
}
{% endhighlight %}

And the output is:

{% highlight java %}
Was the character found: false
{% endhighlight %}

The character was not found and yet we know the composite character used as the regex pattern and the sequence of characters in the input text are canonically equivalent.

What this means is that the Java regex engine by default does not take into consideration canonical equivalence, so it compares the Unicode character `\u00E9` against `\u0065 \u0301` as is without regard to their underlying final values.

If we want the engine to take into account the canonical equivalence of Unicode characters while searching, we must specify the `Pattern.CANON_EQ` flag of the `java.util.regex.Pattern`  class during regex compilation:

{% highlight java %}
public static void main(String[] args) {
    Pattern pattern = Pattern.compile("\u00E9", Pattern.CANON_EQ);
    Matcher matcher = pattern.matcher("Text with unicode accented character \u0065\u0301");
    boolean found = matcher.find();
    System.out.println("Was the character found: " + found);
}
{% endhighlight %}

Now the output is:
{% highlight java %}
Was the character found: true
{% endhighlight %}

## Conclusion

In this article, we discussed the notion of Canonical Equivalence which is common in Unicode character encoding schemes which automatically makes it common in all programming languages.