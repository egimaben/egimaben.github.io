---
layout: post
title: A Simple Guide for Java.util.Date 
comments: true
---


<div class="message">
 In this article, we will explore java.util.Date, consume it as a basic cheatsheet. We will see several examples as it's still used in several code bases and later, I mean really later, we will conspire to throw it out of the window and explore better alternatives for date and time handling in Java. 
</div>

Why am I talking about `java.util.Date` in this day and age when the "whole" community has moved on? Because the whole community has not yet moved on and several of us still work with legacy code bases, some going as far back as Java 6 and much as we are moving forth, we can't go into the future without thoroughly coming to terms with our past :)

I bet you often deal with dates and have to frequently google how to parse date from string, convert date to different string formats for SQL storage or user-friendly display, compare dates, add and subtract days, months, years etc from a date. These are pretty common needs for most developers so let's get on with it and move on with our lives. I will do my best to keep it short and sweet!
<!--more-->
## Date Creation
The java.util.Date class has 2 constructors, a default one and one that take a `long` value which represents the the number of milliseconds that have elapsed since 00:00:00 Thursday, 1 January 1970 AKA Unix Epoch time. It's just a way to keep ourselves and our computers sane while dealing with time in different locales and timezones:
To make for easier testing, I captured a `long` time value on the day of writing this article as the default constructor internally invokes `System.currentTimeInMillis()` to get the `now` time:
{% highlight java%}
@Test
public void test_getDateFromTimestamp_basic() {
	Date now = new Date();
//	assertEquals("current time", now.toString());
	Date fixedDate = new Date(1550487961686L);
	assertEquals("Mon Feb 18 14:06:01 EAT 2019", fixedDate.toString());
}
{% endhighlight %}
Take note of the default format when the date is `toString`'d

## Comparing Dates
The Date object has a method called `getTime` which converts the date back to a `long` unix epoch time. It's a reversal of the previously seen creation process. We can just compare the `long` values of 2 dates to know which is an earlier(less) date:
{% highlight java %}
	@Test
	public void test_compareDates() {
		Date older = new Date(1550487961686L);
		Date newer = new Date();
		assertTrue(older.getTime() < newer.getTime());
	}
{% endhighlight %}

We can use the `compareTo` instance method to compare one date to the other which is passed as an argument to `compareTo`. If the date on which `compareTo` is called is earlier(less) than the argument date, the result is -1, if they are the same instance in time, we get a 0 and a 1 if the argument date is less:
{% highlight java %}
	@Test
	public void test_compareDates() {
		Date older = new Date(1550487961686L);
		Date newer = new Date();
		assertTrue(older.compareTo(newer) == -1);
		assertTrue(newer.compareTo(older) == 1);
		assertTrue(newer.compareTo(newer) == 0);
		assertTrue(older.compareTo(older) == 0);
	}
{% endhighlight %}

Finally,we have the instance methods `before` and `after`. They are definitely much more readable and intuitive than the previous 2, especially `compareTo`. We call each of these on a `Date` object and pass the second date to be compared as an argument. The results are boolean:
{% highlight java %}
	@Test
	public void test_compareDates() {
		Date older = new Date(1550487961686L);
		Date newer = new Date();
		assertTrue(older.before(newer));
		assertFalse(newer.before(older));
	}
{% endhighlight %}

## Date to String conversion

Frequently we will want to convert a date to a string either for storage or user-friendly display, this is done by formatting the date. `java.text.DateFormat` class comes in handly, most commonly though its subclass `java.text.SimpleDateFormat`. SimpleDateFormat is very simple to use and the main requirement is to understand the notation used to define formats.

For reference information about the different formatting patterns and codes we will use here(`m` for minute, `M` for month, `s` for seconds etc), head over [here](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html).

Let's first recall the default conversion when we haven't specified any format:
{% highlight java %}
@Test
public void test_dateFormatting() {
// ie Mon Feb 18 14:06:01 EAT 2019
	Date fixedDate = new Date(1550487961686L);
	assertEquals("Mon Feb 18 14:06:01 EAT 2019", fixedDate.toString());
}
{% endhighlight %}
`Mon` stands for monday, `Feb` for february then 18 is day of month, followed by time in hours, minutes and seconds, timezone and finally year. This may be a little verbose for your needs or you would like to have time in 12 hour clock system or remove the timezone or use the full month name etc, let's first try to recreate the above string by specifying our own format(remember to keep [this](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html) reference doc close):
{% highlight java %}
@Test
public void test_dateFormatting() {
    ...
	SimpleDateFormat defaultFormat = new SimpleDateFormat("E M d HH:mm:ss z y");
	assertEquals("Mon 2 18 14:06:01 EAT 2019", defaultFormat.format(fixedDate));
}
{% endhighlight %}
Everything seems to be alright except the month section, it's a `2` because Feb is the second month of the year, we used its code `M` which represents the simplest format for the month, we could choose to make it 2 digits so that single digit months match double digit months(nov, dec) in length, we do this by appending another `M`
{% highlight java %}
SimpleDateFormat defaultFormat2 = new SimpleDateFormat("E MM d HH:mm:ss z y");
assertEquals("Mon 02 18 14:06:01 EAT 2019", defaultFormat2.format(fixedDate));
{% endhighlight %}
To get month in words but abbreviated as our default example gives, we append another `M` to make them 3:
{% highlight java %}
SimpleDateFormat defaultFormat3 = new SimpleDateFormat("E MMM d HH:mm:ss z y");
assertEquals("Mon Feb 18 14:06:01 EAT 2019", defaultFormat3.format(fixedDate));
{% endhighlight %}
If we want full month name, then make them 4:
{% highlight java %}
SimpleDateFormat defaultFormat4 = new SimpleDateFormat("E MMMM d HH:mm:ss z y");
assertEquals("Mon February 18 14:06:01 EAT 2019", defaultFormat4.format(fixedDate));
{% endhighlight %}
This pattern of adding and reducing the number of codes for each part of the date is common in formatting dates. Play around with this while referring to the reference doc for further clarity. For instance, let's get the full day name by using 4 `E`s:
{% highlight java %}
SimpleDateFormat defaultFormat5 = new SimpleDateFormat("EEEE MMMM d HH:mm:ss z y");
assertEquals("Monday February 18 14:06:01 EAT 2019", defaultFormat5.format(fixedDate));
{% endhighlight %}
Watch out though, as this does not work for all codes. Let's try to convert our time to 12 hour clock system(`h` for 12 and `H` for 24, `a` for the AM/PM marker of 12 hour system):
{% highlight java %}
SimpleDateFormat defaultFormat6 = new SimpleDateFormat("EEEE MMMM d hh:mm:ss a z y");
assertEquals("Monday February 18 02:06:01 PM EAT 2019", defaultFormat6.format(fixedDate));
{% endhighlight %}
We can strip the zeros that appear to pad up the time in case it's all single digits. We do this by using single forms of the codes `hh`, `mm` and `ss`:
{% highlight java %}
SimpleDateFormat defaultFormat7 = new SimpleDateFormat("EEEE MMMM d h:m:s a z y");
assertEquals("Monday February 18 2:6:1 PM EAT 2019", defaultFormat7.format(fixedDate));
{% endhighlight %}
But it does not look intuitive right? I have commonly seen the zero stripped from the hour part but not from the minute and second parts, do what works best for your use case. 
Another caveat and common cause of bugs is developers confusing upper case and lower case formatting codes. `m` and `M` are different, the former for minute and the latter for month. Always remember that these codes are case sensitive:
{% highlight java %}
SimpleDateFormat defaultFormat8 = new SimpleDateFormat("M m");
assertEquals("2 6", defaultFormat8.format(fixedDate));
{% endhighlight %}
With the above examples, we can now achieve almost any format we need by a combination of the formatting codes picked from the reference documentation, see SQL date for example:
{% highlight java %}
SimpleDateFormat sqlFormat = new SimpleDateFormat("y-MM-d HH:mm:ss");
assertEquals("2019-02-18 14:06:01", sqlFormat.format(fixedDate));
{% endhighlight %}
One of the most commont `ISO-8601` formats that has given me headache in the past. Some APIs are strict about the date format they accept. Take note of how we are using single quotes here, they are delimiters and enable us to inject any text in the date format:
{% highlight java %}
SimpleDateFormat isoFormat = new SimpleDateFormat("y-MM-d'T'HH:mm:ss'Z'");
assertEquals("2019-02-18T14:06:01Z", isoFormat.format(fixedDate));
{% endhighlight %}
Sometimes the `ISO-8601` format requires fractions of a second to be represented, we use the `SSS` code:
{% highlight java %}
SimpleDateFormat isoFormat2 = new SimpleDateFormat("y-MM-d'T'HH:mm:ss.SSS'Z'");
assertEquals("2019-02-18T14:06:01.686Z", isoFormat2.format(fixedDate));
{% endhighlight %}
Finally, using the powerful single quotes, we can inject any text in the formatted date:
{% highlight java %}
SimpleDateFormat customStringFormat = new SimpleDateFormat("'Payment received on' y-MM-d 'at' HH:mm:ss");
assertEquals("Payment received on 2019-02-18 at 14:06:01", customStringFormat.format(fixedDate));
{% endhighlight %}

## String to Date conversion

`SimpleDateFormat` has a `parse` method that does the opposite of `format` as we have seen above. It takes any string we have received from a file, from an API response or created by ourselves and attempts to parse it into a `java.util.Date` object or throws a `java.text.ParseException` if the pattern we used does not match the date. 

Here it is very important to get the pattern right, that means you need to understand the format of the date string before hand. Remember, date will still give the default value we saw earlier if it is `toString`'d:
{% highlight java %}
String dateString1 = "20190218";
SimpleDateFormat sdf1 = new SimpleDateFormat("yyyyMMdd");
Date date1 = sdf1.parse(dateString1);
assertEquals("Mon Feb 18 00:00:00 EAT 2019", date1.toString());
{% endhighlight %}
Let's see a few other examples to gain more clarity, using the `/` date delimiter which we haven't seen explicitly before:
{% highlight java %}
String dateString2 = "2019/02/18";
SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy/MM/dd");
Date date2 = sdf2.parse(dateString2);
assertEquals("Mon Feb 18 00:00:00 EAT 2019", date2.toString());
{% endhighlight %}
Let's also see a case where we have text in the date e.g. month in words:
{% highlight java %}
String dateString3 = "2019-February-18";
SimpleDateFormat sdf3 = new SimpleDateFormat("yyyy-MMMM-dd");
Date date3 = sdf3.parse(dateString3);
assertEquals("Mon Feb 18 00:00:00 EAT 2019", date3.toString());
{% endhighlight %}
We used `MMMM` month code because we expected a full month name, sometimes, the month is abbreviated, just do what we know already, strip a single `M`: 
{% highlight java %}
String dateString4 = "2019-Feb-18";
SimpleDateFormat sdf4 = new SimpleDateFormat("yyyy-MMM-dd");
Date date4 = sdf4.parse(dateString4);
assertEquals("Mon Feb 18 00:00:00 EAT 2019", date4.toString());
{% endhighlight %}

## Conclusion

In this article, we have gone through the most important ways of using `java.util.Date` class along side the helper classes for formatting. This should act as a quick refresher for those that use this class once in a while or if you are preparing for an interview and just need to remind yourself. 

In the [next article](/2019/02/20/manipulating-dates-with-calendar-class), we will look at how to `add` and `subtract` dates such that you can get a date before or after the available date without apply string manipulation kung-fu. We will also look at how to decompose dates by extracting component parts i.e. year, month, day etc from a Date object as well as applying internationalisation. These are made possible by the `Calendar` class.