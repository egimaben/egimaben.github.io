---
layout: post
title: Java Garbage Collector and Reference Objects
---


<div class="message">
We take a look at garbage collector concepts in Java and that various reference object types that can be handled by it, demystifying the Java GC!</div>

In this article, we will discuss a few memory management concepts in Java with a heavy focus on the interaction between the Garbage Collector and the different reference objects available.

This is no introduction, so let us mutually agree that you have Java Heap and GC basics down. Many articles cover this topic quite well, and you might actually wonder why am covering something that is already well discussed over the www.
<!--more-->
- I found that most articles introduce Java memory very well, but start choking later on stuff like reference objects. It is either a variant of <em> Who doesn’t know that stuff?</em> attitude or author fatigue. I would like to give my shot and, hopefully, not fall into the same pool.

- Memory is a gold mine of interview questions for Senior Dev positions. <em> Java manages its own memory, I really don’t have to know how it does it.</em> Nice story for your neighbor or close friend. Good luck convincing that mean-looking interviewer. Moral: You must understand memory in the JVM.

## Analogy

Analogies make understanding computing concepts short and sweet, full of oooh! and uh huh! moments. I hope you experience nothing less. 

Picture a school cafeteria. Plates are scarce, but the manager is smart. He derives a strategy together with his staff with one aim: To feed all the hungry students in time without anyone missing food due to a shortage of plates.

**Strategy0**: They plan to collect all used plates for washing whenever a student finishes eating and leaves the cafeteria, such that the same plates are reused by those that have not yet eaten.

Therefore, whenever the serving team reports that plates are running out, a dedicated waitress is sent out to collect all used plates. A plate will be collected as long as the student that was using it has left the cafeteria. These plates are then washed and added to the pile to serve more students.

This strategy works very well, and Mr. Manager is pleased with himself and his staff. Soon, he realizes that some students finish eating but stick around chatting and laughing with friends. He had told his collector waitress to only collect a plate when a student has left the table. The result, a lot of dirty plates stuck on tables simply because satisfied students are still seated there while the cafeteria is frequently running into plate crisis.

**Strategy1**: Mr. Manager has another trick up his sleeve. His instruction to the collector waitress changes: Collect a plate as long as a student has finished eating, regardless of whether he/she is still seated on the table or not. Word reaches students of the new and “hostile” changes, a lot of special groups come with complaints seeking a compromise with Mr. Manager. This strategy is marked a failure.

**Strategy2**: Being the smart man he is, he comes up with yet more brilliant ideas and draws up an instruction set for any collector waitress:

- If a student is a prefect or guild rep, allow them to retain their plate for as long as they like until they explicitly call you to collect it or they leave the table.
- If a student is a girlfriend or boyfriend to a prefect or guild rep, allow them privileges of their partner… meaning, even if they have finished eating but are still on the table, don’t rush to collect their plates unless there is absolutely no alternative and the cafeteria has literally reached its limit in terms of available plates.
- If the student is not a prefect and not in a relationship with any prefect, as is the case with most first-years, be very vigilant about collecting his/her plate — no delays, no compromise. Whether they are still seated on the table or not, as long as they have finished eating… even if they beg and cry. They are not allowed any kind of privilege whatsoever.
- Finally, the doctor has sent a list of diabetic students and needs to know the exact time they had their last meal every day in order to determine when to do routine blood sugar level checks. Collect their plates as you do for the students in the previous category, but always note down the exact time when you pick the plate and notify me.
This turned out to be an excellent strategy. Talk about iterative problem solving .

References to objects in Java follow a certain hierarchy of strength and privilege, as do the students in the cafeteria. Let us dive into the technical rundown.

## Strong Reference

In every Java program, objects are the way to hold and manipulate data:

{% highlight java %}
StringBuilder sb = new StringBuilder();
{% endhighlight %}

In the above snippet, the new key word creates a `StringBuilder` object and stores a **strong** reference to it in variable `sb`. This is the default level of strength for all objects we create, so we don’t use any special label to identify them as we do for the rest.

To create the rest of the references in the hierarchy, we need special wrapper objects that reside in the `java.lang.ref` package.

Any object with at least one **strong reference** is not eligible for garbage collection. In our analogy, **prefects** hold **strong references** to their plates. In technical terms, we say the object is **strongly reachable**.

It **only becomes eligible for collection when we nullify its reference**, akin to a prefect leaving the table:

{% highlight java %}
sb = null;
{% endhighlight %}

## Soft Reference
A soft reference can be created to an object by wrapping its instance in a `java.lang.ref.SoftReference` object:
{% highlight java %}
StringBuilder sb = new StringBuilder();
SoftReference<StringBuilder> sbSoftRef = new SoftReference<>(sb);
sb = null;
{% endhighlight %}

In the above snippet, the first line creates a string builder object with a **strong reference** stored in `sb`. The second line creates a soft reference to it inside `sbSoftRef` so that now the string builder object has two references.

At this stage, the string builder is not eligible for collection whatsoever. However, the third line nullifies the strong reference, and now the object only boasts of a soft reference.

The object is now similar to a used plate sitting in front of the boyfriend/girlfriend of a prefect — it can be collected only as a last resort when the cafeteria staff is absolutely sure there are no more plates available. Technically, we say the object is **softly reachable**.

In this phase, we can still retrieve a **strong reference** to the object by calling the `get` method of the **SoftReference object**, which returns null if the object is already collected:

{% highlight java %}
sb = sbSoftRef.get();
{% endhighlight%}

Objects with only soft references to it are **collected if and only if the JVM has concluded that there is no more memory to allocate to new objects and is on the brink of throwing an OutOfMemory error**.

**Soft references are intended for use in memory-sensitive caches**. As the cache grows, available memory for new objects reduces yet you need the cache, so the JVM compromises with “you” until it can’t anymore.

## Weak Reference
A weak reference can be created to an object by wrapping its instance in a `java.lang.ref.WeakReference` object:

{% highlight java %}
StringBuilder sb = new StringBuilder();
WeakReference<StringBuilder> sbWeakRef = new WeakReference<>(sb);
sb = null;
{% endhighlight %}

In the above snippet, after nullifying the strong reference in the third line, the object immediately becomes eligible for GC.

The object is now similar to a used plate sitting in front of a first-year student with no privileges. It can be collected right away without any regard for whether the student is still seated at the table. Technically, we say the object is **weakly reachable**.

Though we can still retrieve a strong reference to the object, the window of opportunity is way smaller than that of soft reference. We will get a null much more often than in the other cases:

{% highlight java%}
sb = sbWeakRef.get();
{% endhighlight %}

Objects with only weak references are collected eagerly by the GC whether memory availability is tight or not.

Weak references are intended for use in **Canonicalized mapping**. An understanding of this use case is beyond the scope of this article, so we will not dwell on that…


## Phantom Reference

A phantom reference can be created to an object by wrapping its instance in a `java.lang.ref.PhantomReference` object:
{% highlight java %}
StringBuilder sb = new StringBuilder();
ReferenceQueue<StringBuilder> refQ = new ReferenceQueue<>();
PhantomReference<StringBuilder> sbPhantomRef = new PhantomReference<>(sb, refQ);
sb = null;
{% endhighlight %}

In the above snippet, after nullifying the strong reference in the fourth line, the object immediately becomes eligible for GC. Ignore the `ReferenceQueue object`, we'll get back to it. For now, just remember that **PhantomReference, unlike soft and weak references, is useless without a ReferenceQueue**.

The object is now similar to a used plate sitting in front of a first-year student who is diabetic. It can be collected right away without any regard for whether the student is still seated at the table, albeit with one condition: the exact time of collection is noted and the doc notified.

For as long as the piece of paper on which the time has been noted has not yet been used and discarded by the doctor, we refer to the object as phantom reachable. (Major point of confusion, let's keep going :)).

The `get` method of a `PhantomReference` object is useless because it always returns `null`. This further strengthens the uniqueness of this reference object vis-a-vis soft and weak refs. The next sections will make this uniqueness even more crystal clear.

Phantom references are intended for use as a more flexible alternative to the `Object.finalize()` method.

## Reference Queue

True to its name, a reference queue is a data structure that enqueues reference objects namely: `WeakReference`, `SoftReference`, and `PhantomReference`.

Whether a reference object is enqueued at any time or not depends on whether we provide a `ReferenceQueue` object upon creation of the reference object. Apart from the case of `PhantomReference`, it is not mandatory or even useful to provide one.

Depending on the type of reference object, the exact point at which it is enqueued varies. I don’t intend to dwell much on this subject apart from furthering the coverage of phantom references. **Let's be more keen on the next paragraph**.

A **phantom reference is enqueued as soon as the object it is referring to has been finalized** by the garbage collector. **Finalized** means its `finalize()` method has been called. The GC calls the `finalize()` method of objects just before collecting them for the benefit of the application that created them. The benefit is the opportunity to release **non-“GC’eable”** resources that the object must have created or used during its earthly life (read Java heap life). **Non-“GC’eable”** resources are inaccessible to the GC. One obvious example is a file handle provided by the OS. To demonstrate, take a look at the finalize method of `FileInputStream` class in all its glory:

{% highlight java %}
protected void finalize() throws IOException {
  if ((fd != null) &&  (fd != FileDescriptor.in)) {
    /* if fd is shared, the references in FileDescriptor
     * will ensure that finalizer is only called when
     * safe to do so. All references using the fd have
     * become unreachable. We can call close()
     */
    close();
  }
}
{% endhighlight %}

Notice the `close()` method call at the end, look familiar? Yes, it's that `fis.close()` call our Java teachers taught us to always place in the `finally` clause of `try/catch/finally` blocks. Wondering whether it is a duplicate call, since we always `close()`‘d our file resources? Great, read on!

Now notice the `if` check at the top of the method body. If we already called `close()` inside our code, then `fd` in the above code would be `null` and the rest of the method body would not be executed. Phewww!!!! (Did that come too late???). Read on!

It so happens that, these days, the `finalize()` method, especially in library/platform classes such as `FileInputStream` above, acts as a safety net for cases where we as developers forget to release non-heap resources.(I may already by whispering to you that you may never have to use phantom refs, but read on).

So what the heck does all this have to do with phantom references and reference queues. Am I losing focus? Well, you asked, so I'll answer:

The `finalize()` method is plagued with numerous issues that counter almost all the advantages it sought to offer. There is actually a detailed coverage in **Joshua Bloch’s Effective Java book**, and several blogs talk about its problems extensively. Don’t think that they are just staining the image of an otherwise brilliant API. Unless you are an upcoming James Gosling, I would highly recommend that you just take their word and make peace with other alternatives for cleaning up after your resources. Now, keep reading — we're almost done.

We can now `finalize()` this section with why phantom refs and ref queues come in. `PhantomReference` object was created to **provide a more flexible alternative to finalize()**. Whether it has succeeded or not is quite a huge topic, but my bet is **NO**.

How we as programmers do this is by supplying a `ReferenceQueue` instance while creating a `PhantomReference` instance as we saw earlier. As soon as the `finalize()` method of the object is called, its phantom ref is enqueued. It is up to us to keep polling the reference queue to keep track of when this event happens. Here is where we manually release resources, including a final call to `phantomRef.clear()` method to nullify the underlying reference.

## Differences Put Together

I know the title of this section is kinda lame, but bear with me. I will try to put together the differences of the three reference objects just to clear any doubts:

- **Enqueuing**: Soft and weak reference objects are enqueued as soon as the GC, from its algorithms, has “marked” their memory objects for collection, but has not yet finalized them. On the other hand, phantom references are enqueued as soon as the memory objects have been finalized.
- **Reference.get()**: The get method of both soft and weak references returns a strong reference to the underlying object or null if that object has already been marked for collection. On the other hand, the `get` method of phantom reference always returns null (whether you are Josh Bloch or Jon Skeet calling it. Kidding!). This difference strikes a key point about their use cases, for weak and soft refs to be of any use, one should be able to recreate a strong ref to their objects within the “window of opportunity.” But phantom refs don’t need this quality as we explored earlier.
- **Reference.clear()**: This method is available in all the three ref objects. It sets the underlying reference to `null` hence explicitly availing the memory object for collection before its time. This is a useless facility for weak and soft refs as it would counter their benefit. But it is key for phantom refs as you have to call it on Mr. phantom at the end of your clean up steps.
- **ReferenceQueue**: it is useless for soft and weak refs but without it, phantom refs are useless.
- **Use cases**: Soft refs: memory-sensitive cache, weak-refs: canonicalized mapping, phantom-refs: more flexible substitute for `finalize()`.

## Conclusion

In this, albeit very long, article, I have tried to discuss my findings on the relationship between the GC and reference objects. I hope it has, at least, given us a better understanding of the reference objects specifically and the GC generally.

It goes without saying that you may never have to deal with these reference types directly:

- Need memory-sensitive caching? (A very complex and meticulous endeavor to get right, by the way?) Use solid and battle-hardened libs like ehcache.
- Need canonicalized mapping? Simply use `java.util.WeakHashMap` and save yourself hair loss.
- Need to do pre-mortem clean up? Don’t be deceived by Mr. Phantom’s offering as opposed to `finalize()`, stick to age-old `try/catch/finally`.

