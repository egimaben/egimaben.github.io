---
layout: post
title: Manipulating Dates with Calendar Class
comments: true
---

 In this article, we will explore `java.util.Calendar` class. In a [previous article](/2019/02/18/a-simple-guide-for-java-util-date), we discussed `java.util.Date` class but its failures around internationalisation concepts like timezone awareness were evident. We were also not able to compute future and previous dates from a `Date` object just like we could not decompose a date into its parts(year, day, month, hour etc) easily.
 
 We will now see how to achieve all this with the `Calendar` class.

<!--more-->

## Calendar Creation
There are 3 ways of constructing a `Calendar` class object. We use the static `getInstance` calendar method instead of the `new` key word. The `Calendar` class is timezone and locale aware and only assumes these settings to a default when we don't specify them:
{% highlight java%}
Calendar defaultCal = Calendar.getInstance();
Calendar tzCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
Calendar tzAndLocaleCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"), Locale.UK);
{% endhighlight %}
As you can see from the snippet above, `defaultCal` will default to the timezone and locale settings in the JVM. A second variation of `getInstance` allows us to specify a timezone and the third, both timezone and locale.

The 2 concepts of timezone and locale are what enables internationalization in the `Calendar` class. When timezone is set, all operations on the class and all internal values reflect the time in that timezone, in the above case, GMT. 

Locale is used to customize the `Calendar` class to a specific language and cultural setting, this is mostly for presentational purposes as is the case where you need to retrieve date and time values for printing. Setting the local to `Locale.France` for instance would return month of January as Janvier. We will see more examples in the internationalization section.

## Conversion to and from Date
We can convert `Date` instances to `Calendar` and vice versae. For easy testing, let's assume we are starting from the fixed time we had in the previous article, the unix epoch time `1550487961686L` corresponds to `Mon Feb 18 14:06:01 EAT 2019` in my timezone.

`Date` is converted to `Calendar` using the `setTime` method: 
{% highlight java %}
Date date = new Date(1550487961686L);
Calendar cal = Calendar.getInstance();
cal.setTime(date);
{% endhighlight %}
We can use `getTime` to convert it back to `Date`:
{% highlight java %}
Date fromCal = cal.getTime();
{% endhighlight %}
It's also possible to convert the unix time directly to `Calendar` and back to unix time:
{% highlight java %}
Calendar cal = Calendar.getInstance();
cal.setTimeInMillis(1550487961686L);
//back
long unixTime = cal.getTimeInMillis();
{% endhighlight %}

## Calendar Time Comparison
Just like `Date`, we can use the `compareTo` instance method of `Calendar` class to compare one date to the other which is passed as an argument to `compareTo`. If the date on which `compareTo` is called is earlier(less) than the argument date, the result is -1, if they are the same instance in time, we get a 0 and a 1 if the argument date is less:
{% highlight java %}
@Test
public void test_compareCalendarDates() {
    Calendar currCal = Calendar.getInstance();
    Calendar fixedCal = Calendar.getInstance();
    fixedCal.setTimeInMillis(1550487961686L);

	assertEquals(currCal.compareTo(fixedCal), 1);
    assertEquals(fixedCal.compareTo(currCal), -1);
	assertEquals(fixedCal.compareTo(fixedCal), 0);
}
{% endhighlight %}

We also have the instance methods `before` and `after`. They are definitely much more readable and intuitive than `compareTo`. We call each of these on a `Calendar` object and pass the second object to be compared as an argument. The results are boolean:
{% highlight java %}
@Test
public void test_compareCalendarDates() {
	Calendar currCal = Calendar.getInstance();
	Calendar fixedCal = Calendar.getInstance();
	fixedCal.setTimeInMillis(1550487961686L);
		
	assertFalse(currCal.before(fixedCal));
	assertTrue(fixedCal.before(currCal));
}
{% endhighlight %}

## Date decomposition With Calendar Class

One of the short comings of `Date` class is that we can't easily retrieve date parts e.g. hour, day, month from the object. `Calendar` class makes this easy with static integer identifiers for each date part e.g. `Calendar.DAY_OF_WEEK`, `Calendar.MONTH` etc in conjunction with the `get` method on a calendar object.

Let's decompose our earlier fixed date:
{% highlight java %}
@Test
public void test_dateTimeDecomposition() {
	//1550487961686L ie  Mon Feb 18 14:06:01 EAT 2019
	Calendar cal = Calendar.getInstance();
	cal.setTimeInMillis(1550487961686L);
	assertEquals(2, cal.get(Calendar.DAY_OF_WEEK));
	assertEquals(1, cal.get(Calendar.MONTH));
	assertEquals(18, cal.get(Calendar.DAY_OF_MONTH));
	assertEquals(14, cal.get(Calendar.HOUR_OF_DAY));
	assertEquals(2, cal.get(Calendar.HOUR));
	assertEquals(6, cal.get(Calendar.MINUTE));
	assertEquals(1, cal.get(Calendar.SECOND));
	assertEquals(2019, cal.get(Calendar.YEAR));
}
{% endhighlight %}
As you can see, the values returned from the `get` calls are the integer representations of each date part. Notice that days of the week start from Sunday by default as opposed to Monday, hence2 for Monday. 

Also notice that months use a zero-based indexing system i.e. January is `0` and February is `1`.
Since we touched a bit on internationalization, we can use the `getTimeZone()` method to see some interesting things:
{% highlight java %}
@Test
public void test_dateTimeDecomposition() {
	//1550487961686L ie  Mon Feb 18 14:06:01 EAT 2019
	Calendar cal = Calendar.getInstance();
	cal.setTimeInMillis(1550487961686L);

	assertEquals("Africa/Kampala",cal.getTimeZone().getID());
	assertEquals("Eastern African Time",cal.getTimeZone().getDisplayName());
	assertEquals("Heure d'Afrique de l'Est", cal.getTimeZone().getDisplayName(Locale.FRANCE) );

	cal.setTimeZone(TimeZone.getTimeZone("Asia/Dubai"));

	assertEquals("Asia/Dubai",cal.getTimeZone().getID());
	assertEquals("Gulf Standard Time",cal.getTimeZone().getDisplayName());
	assertEquals("Heure normale du Golfe", cal.getTimeZone().getDisplayName(Locale.FRANCE) );
}
{% endhighlight %}
We can see that my timezone has defaulted to `Africa/Kampala` which is my current city, this is because we did not specify another. 

The timezone object has important attributes e.g ID which is denotes region/continent and city, it could have been `Europe/Paris`. 

There is also a display name which is more descriptive of the timezone.

Notice how `getDisplayName` takes a `Locale` argument so that it knows how best to describe the timezone. When we specified `Locale.FRANCE` it defaulted to French. Similarly, we played around with Dubai time a little just so the point is clear.

Just as there is a `get` method on `Calendar` class objects, there is also a `set` method in case we want to specify values for the date parts. Let's use the gulf time for the fun of it:

{% highlight java %}
@Test
public void test_dateTimeDecomposition() {
	//1550487961686L ie  Mon Feb 18 14:06:01 EAT 2019
	Calendar cal = Calendar.getInstance();
	cal.setTimeInMillis(1550487961686L);
	cal.setTimeZone(TimeZone.getTimeZone("Asia/Dubai"));

	assertEquals(15, cal.get(Calendar.HOUR_OF_DAY));
	cal.set(Calendar.DAY_OF_WEEK, 3);
	assertEquals("Tue Feb 19 14:06:01 EAT 2019",cal.getTime().toString());

	cal.set(2018, 11, 31);
	assertEquals("Mon Dec 31 14:06:01 EAT 2018",cal.getTime().toString());
}
{% endhighlight %}
Dubai is 1 hour ahead of Kampala, hence 14hrs now appears as 15hrs. Notice how we can use the same static identifiers for date parts(`Calendar.DAY_OF_WEEK` for day of week) to set their values. 

In the example above, we have set day of week to 3 which corresponds to tuesday, remember day 1 is Sunday. 

Best practice is to use the built in static identifiers for named parts e.g. instead of writing `1` for sunday, use `Calendar.SUNDAY` and instead of using `3` for `April`, prefer `Calendar.APRIL`.

There is also another overloaded `set` method that takes different, defined parameters at ago. In the examle above, we are able to set year, month and day which backdates our date to 31st December, 2018.

You can check more options by playing around with `Calendar` class. Also remember `getTime` returns a `Date` object, which we explored in the previous article.

## Adding and Subtracting Dates
With `Calendar` class, we can add days to the current day and get another date in the future or subtract to reset to a date in history. 

The same pattern works for months, years, hours and all other components of the date. There are numerous applications of this.

This is great whenever you need to set, say an expiry date on a product in your application that you need to compute from the manufacture date.

Another example could be when you know someone's birthday and would want to compute their exact date of birth, knowing the age... just subtract that from the current year.

Let's get on with it:
{% highlight java %}
@Test
public void test_dateAddAndSubtract() {
	//1550487961686L ie  Mon Feb 18 14:06:01 EAT 2019

	Calendar cal = Calendar.getInstance();
	cal.setTimeInMillis(1550487961686L);
	
	assertEquals(Calendar.MONDAY, cal.get(Calendar.DAY_OF_WEEK));
	assertEquals(Calendar.FEBRUARY, cal.get(Calendar.MONTH));
	assertEquals(18, cal.get(Calendar.DAY_OF_MONTH));
	assertEquals(14, cal.get(Calendar.HOUR_OF_DAY));
		
	cal.add(Calendar.DAY_OF_MONTH, 1);
	assertEquals(19, cal.get(Calendar.DAY_OF_MONTH));
	cal.add(Calendar.DAY_OF_MONTH, -1);
		
	cal.add(Calendar.MONTH, 1);
	assertEquals(Calendar.MARCH, cal.get(Calendar.MONTH));
	cal.add(Calendar.MONTH, -1);

	cal.add(Calendar.DAY_OF_WEEK, 1);
	assertEquals(Calendar.TUESDAY, cal.get(Calendar.DAY_OF_WEEK));
	cal.add(Calendar.DAY_OF_WEEK, -1);

	cal.add(Calendar.HOUR_OF_DAY, 24);
	assertEquals(14, cal.get(Calendar.HOUR_OF_DAY));
	cal.add(Calendar.HOUR_OF_DAY, -24);
}
{% endhighlight %}

Notice how we are using `add` to advance `18th` to `19th`, `Feb` to `March`, `Monday` to `Tuesday` etc. Notice also there is no `subtract` method but passing a negative offset to `add` makes it a subtraction. We keep using that to reverse the effect of the earlier additions. Which brings us to another caveat.

Whenever you `add` a value to part of a date, the `Calendar` object may recompute everything to reflect this. See the example below:
{% highlight java %}
@Test
public void test_dateAddAndSubtract2() {
//1550487961686L ie  Mon Feb 18 14:06:01 EAT 2019

	Calendar cal = Calendar.getInstance();
	cal.setTimeInMillis(1550487961686L);
	assertEquals(Calendar.MONDAY, cal.get(Calendar.DAY_OF_WEEK));
	assertEquals(Calendar.FEBRUARY, cal.get(Calendar.MONTH));
	assertEquals(18, cal.get(Calendar.DAY_OF_MONTH));
	assertEquals(14, cal.get(Calendar.HOUR_OF_DAY));
		
	cal.set(Calendar.DAY_OF_MONTH, 28);
		
	cal.add(Calendar.HOUR_OF_DAY, 24);
	
	assertEquals(14, cal.get(Calendar.HOUR_OF_DAY));	
	assertEquals(1, cal.get(Calendar.DAY_OF_MONTH));
	assertEquals(Calendar.MARCH, cal.get(Calendar.MONTH));
}
{% endhighlight %}
Notice that our month is `Feb` and we have chosen to set it's day to `28th` before adding 1 day to it. This results in changes in different components. The hour of the day is still 14 hrs since we moved by a whole day, but the day of the months is now `1st`, know why? because `28` is the last day of `Feb`, so we are moved to the next possible day i.e. `1st March`, we have also confirmed by testing the month which is now `March`.

## Internationalization
If you are playing with your application on your machine with no hope of deploying it to different countries, then you can ignore this section. However, any noteworthy developer atleast knows not to ignore this.

It's important to explicitly define the timezone your application is expected to run in and also localize the presentations to suit the main language spoken in that timezone. There are also conventions for display of dates, some use `/` delimiter, others use `-` and others use `.`.

If your application is going to run internally in a French, German or Chinese firm, why would you display the month name in English? Probably the JVM would default to the detected locale and timezone but what if the server is in the US, in a different timezone and locale? So be explicit about this.

One thing some of us may probably not know yet is that `Sunday` is not the first day of the week in all locale's, neither is `Monday`:
{% highlight java %}
@Test
public void test_internationalization() {
	Calendar calBasic = Calendar.getInstance();	
	assertEquals(Calendar.SUNDAY, calBasic.getFirstDayOfWeek());

	Calendar calTimezone = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
	assertEquals(Calendar.SUNDAY, calTimezone.getFirstDayOfWeek());

	Calendar calLocale = Calendar.getInstance(Locale.FRANCE);
	assertEquals(Calendar.MONDAY, calLocale.getFirstDayOfWeek());

	Calendar calLocaleAndTimezone = Calendar.getInstance(TimeZone.getTimeZone("GMT"),Locale.FRANCE);
	assertEquals(Calendar.MONDAY, calLocaleAndTimezone.getFirstDayOfWeek());	
}
{% endhighlight %}
My timezone is `EAT` so `calBasic` defaults to that and `Sunday` is apparently the first day of the week in my side of the world, so if your application made assumptions, it could easily encounter subtle bugs that are hard to detect. Assume your server is in the UK (GMT) and it's processing weekly salaries for clients in France. 
 
Salaries should appear on first day of the week, not before and not after, but your application knows Sunday is the first day and not Monday as is known by your French client, I don't think he would smile and you'd have a hard time figuring this out because, well `it runs well on my end` you say.

Say, you are also printing financial reports for clients in France, Germany and China...after parsing your date, you need to convert it to human readable form, without this awareness, you'd either need to write different logic for each country to hack around the displays or you'd encounter several change requests from your clients after failed user acceptance tests.

Calendar has a `getDisplayName` method to which we can supply the named date component whose name we want e.g. month and day or week, a style specifier and locale:

{% highlight java %}
@Test
public void test_internationalization() {
	Calendar cal = Calendar.getInstance();	
	cal.setTimeInMillis(1550487961686L);
	assertEquals("Monday", cal.getDisplayName(Calendar.DAY_OF_WEEK, Calendar.LONG, Locale.UK));
	assertEquals("lundi", cal.getDisplayName(Calendar.DAY_OF_WEEK, Calendar.LONG, Locale.FRANCE));
	assertEquals("Montag", cal.getDisplayName(Calendar.DAY_OF_WEEK, Calendar.LONG, Locale.GERMAN));
	assertEquals("星期一", cal.getDisplayName(Calendar.DAY_OF_WEEK, Calendar.LONG, Locale.CHINA));	
}
{% endhighlight %}
For different timezones, the end result is customized.

## Conclusion

In this article, we have gone through the most important ways of using `java.util.Calendar` class and how to mix it with `Date` class. This knowledge takes us a long way in knowing how to handle dates in an idiomatic manner, without hacking around. 

In the next article, we will take a step back and explore the major weaknesses of `java.util.Date` class and why it has been deprecated, meaning, why you should prefer not to use it.