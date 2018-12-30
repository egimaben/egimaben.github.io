---
layout: post
title: Getting The Most Out of HashMaps
---
<div class="message">
HashMaps are a great tool, but you need to know the internals to get the best performance from them. This guide covers collisions, iterations, load factors, and more.</div>

`HashMap` is perhaps the most popular implementation of the `Map` interface in **Java Collections Framework (JCF)**. If you want to fully understand the internal workings of `HashMap`, the best tutorial I have found online is [here](http://www.baeldung.com/java-hashmap).

For this article, we are only going to focus on the performance specific concepts. `HashMap` offers us a very fast, well-optimized data structure solution for many problem scenarios.
<!--more-->
## Good News

`HashMap` is a great advantage for us because it is so versatile and fits in a lot of use cases:

- Want to load objects from an SQL result set and marshal them over the network? use a `HashMap`.
- Want to group some log file content according to a certain field? `HashMap` is available.
- Want to move some arbitrary data around without the restriction of creating a POJO or DTO? Oh! Hail! `HashMap`.
What can’t you do with `HashMap`, plus the API is extremely easy to use.

## Bad News

The caveat is that, it is easy to use `HashMap` in Java but not exactly easy to use it optimally or efficiently if you may. A few years back, when I was new to Java and programming in general, we were building a school information system.

We would query some SQL database somewhere and then need to marshal the objects over the network using RPC. I can still recall the excitement I experienced seeing `HashMap` at work:

{% highlight java %}
HashMap<String,Student> records = new HashMap<>(); 
ResultSet rsWithHundredsOfRecords= //... 
  while(rs.next()){ 
    Student student = new Student(); 
    student.setName(rs.getString("name")); 
    student.setAge(rs.getString("age")); 
    // plus several other attributes 
    records.put(student.getId(), student); 
  }
{% endhighlight %}

Test data would average around a thousand records and we would not realize any performance dip. Awesome feat right? **WRONG**.

Even without any test, a seasoned developer can look at the above code and immediately become uncomfortable — the **rat smell** is too strong. The **happy go lucky** developer, like myself a few years ago, would not lose their smile though.

If you are among the less comfortable with the above code, feel free to stop reading at this point. However, if you are like me a few years ago and are very comfortable with the above code, endeavor to read the post up to the end and you will surely gain the competence to refactor my code for me.

## Initial Capacity
`HashMap` uses an array as its primary storage. Every `put` operation inserts a Key-Value pair at an index in this array, and every get operation retrieves a Key-Value pair from an index of this array.

Capacity refers to the size of this underlying array. Initial capacity is, therefore, the size of this array when the HashMap is created:
{% highlight java %}
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
{% endhighlight %}

The above code resides inside `java.util.HashMap` class. The above value(`1<<4 // 16`) is used as the initial capacity of the hashmap if we initialize it the way I did:
{% highlight java %}
HashMap<String, Student> records = new HashMap<>();
{% endhighlight %}

Now that the hashmap has an initial capacity of 16 and my records run in the hundreds:

{% highlight java %}
 ResultSet rsWithHundredsOfRecords = ...;
{% endhighlight %}

What happens when the number of elements being added to the underlying array are exhausting the initial capacity?

It so happens that **HashMap is a self-regulating data structure**. It has an internal mechanism for detecting when its current capacity is being exhausted and resizing its internal array.

What happens is that when the number of entries is found to have reached a certain threshold, a new array is created with twice the size of the previous one and all the entries are transferred from the old array to the new array. A process called **rehashing**.

**Rehashing is quite an expensive process and should be mercilessly minimized** during design, or even totally avoided if possible.

The `HashMap` class contains another constructor that enable us to pass in a custom value for **initial capacity**. In the bad example that we saw in the Bad News section, we could simply have anticipated the size of the ResultSet and used that to create a **well-dimensioned hashmap**, say we expect about 1,000 records:
{% highlight java %}
 HashMap<String, Student> hashMapWithCapacity = new HashMap<>(1024);
{% endhighlight %}

With the above refactored code, we would have saved ourselves at least six rehashing operations. Wondering how? Just picture that every time the number of entries reaches the current capacity, a rehashing operation doubles the capacity, starting from 16, then 32, 64, up until 1024.

The above explanation suffices, for now, however, we will realize later that the decision to rehash is a little more complicated than this.

The take-home point here is: **Choose an initial capacity high enough to minimize the frequency of rehashing**.

## Collision
As a simple reminder to ourselves. `HashMap` works on the principle of hashing. For reasons beyond the scope of this article, hash code collisions are a common occurrence, **where different keys end up at the same array index**. More about this can be found in the articles linked at the beginning of this post.

It sounds like an anomaly, but it is quite normal, specifically because of the `equals` and `hashCode` contract of `java.lang.Object`, the mother of all objects in Java.

It suffices to only know that **where there are collisions, HashMap performance is degraded**.

When a collision occurs at a given array index, instead of a K-V pair being stored there, a linked list is created in place and the colliding entries are chained together in this list.

This way, if another entry hashes to the same location, each element of the linked list is compared with the new entry. If a match is found, the match is replaced, but if none is found, the new entry is placed at the head of the list.

Something similar happens during retrieval. When the key in `hashMap.get(key);` hashes to an index that has collisions, the key is compared to all other keys in the linked list, if a match is found, the matched key’s value is returned, but if no match is found, null is returned.

One of the strengths of a hashmap is **lightning speed storage (put) and retrieval (get) operations**. In more technical terms, they occur in constant time, `O(1)`. This means that regardless of the size of the hashmap or its capacity, a put or a get will return instantly.

However, in the event of collisions, we can no longer guarantee `O(1)` performance. Because the performance will now be affected by the length of the linked list at a location where there is collision.

Specifically, **the time to retrieve an entry will be the constant time to hash to the array index, then the linear time O(n) to iterate over the linked list**, where `n` is the size of the linked list.

We can not do much about this problem, apart from ensuring that the hashmap has a **big enough capacity to minimize collisions** and if possible, the **hash function is good enough to disperse entries with equal distribution across the array**.

## Load Factor

Load Factor, abbreviated as LF, is the ratio of the hashmap’s size to its capacity. It is a float between 0 and 1 and the default value is `0.75f`:

{% highlight java %}
 /** * The load factor used when none specified in constructor. */ 
static final float DEFAULT_LOAD_FACTOR = 0.75f;
{% endhighlight %}

The LF is the ratio of the hashmap size (number of entries stored so far) to capacity (total array length regardless of entries) at which a rehash operation occurs. The default LF of 0.75 implies that if our hashmap is initialized with 100 initial capacity, then as soon as we make 75 entries in

The default LF of 0.75 implies that if our hashmap is initialized with 100 initial capacity, then we can make 75 entries in it comfortably. The 76th entry will trigger a rehash. We can use a custom LF by creating hashmap with another constructor which takes LF along side initial capacity:

{% highlight java %}
 HashMap<String, Student> records = new HashMap<>(100, 0.5f);
{% endhighlight %}

Through some tests and calculations, it can be proved that in the case of collisions, **the average number size of the linked lists in the array is about the same as the LF**. We have already seen that collision degrades the performance of hashmap in most operation.

This can only mean that **when LF which is too high, it will degrade performance** due to its relationship with the linked list size. **When LF is too low, it will also degrade performance** due to frequency of rehashes. However, 0.75f offers a good optimum value.

## Iteration Performance

**Iterating over a hashmap is affected by its size and capacity**. Read the preceding sentence again and again until you think you have understood and then realize it’s very confusing and then accept that you need more explanation!

The time complexity of hashmap iteration is O(n) where n= capacity + size. This implies that if the hashmap’s initial capacity is 100 and it only has a single entry so far, iteration will still levy a worst case performance time of O(n) where n=101.

To picture this, we need to remember that **iteration starts from the beginning of the array and proceeds sequentially to its end**. There is **no search efficiency in hashmap** as there is in binary search trees.

It is possible that the value being searched is close to index 0 in the array, in which case the time will be a fraction of worst case run time. However, as computer scientists, we are better off planning for the worst case scenario.

We also need to remember how hashing works. The first entry in the map may hash to index 95 in the array and the second to index 34, etc. There is absolutely no order.

It is now obvious that **a high initial capacity will result in poor iteration performance**. This means that we should **choose a high initial capacity where we know there will be a lot of entries but little iteration needed**.

As we saw in the previous section, LF is approximately the average length of linked lists in the hashmap, so a low LF is a good thing for iteration performance.

## Conclusion

In this article, we have explored HashMap class and its internal implementation details that are critical to performance. One should now be able to refactor the initial Bad News snippet to pass as optimized code. Afterall initial capacity is just one of the several implementation details that we have seen in this post.

It is very important to understand these intricacies so that we can become better designers and decision makers.

