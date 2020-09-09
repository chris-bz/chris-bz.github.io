---
layout: post
title:  "Converting date to ISO 8601 in Python"
date:   2020-09-07 11:53:01 +0200
categories: python
---
## Why use ISO 8601?

No matter which area a programmer works in, he most likely has to deal with dates in some way. Even writing a basic application log benefits from stamping information with dates, so we know when a potential problem ocurred etc.

Unfortunately, they are a tricky subject. It's often an extra that just needs to work, but doesn't. And it can fail for many reasons.

One of the main ones is incompatibility between different date formats. Sometimes, the month is a word, sometimes it's a number. The standard for order of year, month and day is different in one country than it is in another.

When month is a number and it's 12 or lower, we can confuse it with day number and not be able to verify which is which. One month has 30 days, another has 31. April has 28 days... but only 3 years out of 4. All that needs to be taken into consideration when dealing with dates.

To alleviate the pain from some of those problems, ISO 8601 was introduced. It is an international standard, an exact way of writing date and time so that there is no confusion.

Having all dates in one format is important for one's own projects. When they are always written the same, there is no need to convert anything, they are compatible with each other for future use by default.

It is also important externally. When someone else needs to process them, there is no need to verify what is what, to format string to make it compatible with another company's own internal formatting standard.

Choosing ISO 8601 is also very convenient for the programmer. Most frameworks, modules etc. that deal with dates use it as the default standard and force the user to first convert the input data string to ISO 8601 before further processing. Such is the case with SQLite, Pandas, Django and many more.

It is always the best practice to try to stick to one data formatting standard whenever possible. And since ISO 8601 is the most popular standard for date formatting in the world, it is most convenient to stick to it. It makes our dates most compatible with other dates "out there", saving us from many potential headaches in the future. 

<br>
## What needs does this converter address?

Unfortunately, there is no one correct way to write a converter. It all depends on which context it needs to be used, and based on that, the optimal behavior can be different.

For example, in some cases it is crucial that failure to convert produces an error and instantly stops the program from running to not pollute the data with pseudo-dates, or empty strings which have to be addressed quickly not to break something else.

Another time, it is mandatory that the service continues working and error is handled silently and gracefully. There is no right or wrong way to write it, it's all context-dependent.

The converter function below behaves like so:
- deals just with date, not date and time
- does not require hint parameters to know the format, it tries to guess which is which
- it returns None on failure, does not break whatever's running it
- when month and day are both numbers, it assumes that month is always the digit on the left side, and day on the right
- the year is always a four-digit number
- the month can be either a number, or a word
- a day can be either a one-digit number or a two-digit number
- there can be no day, but month and year both have to be present

Make sure that a converter you need meets all the criteria above, or that you have fixes for when it doesn't.

Below I break apart the code. If you just want the entire thing to copy-paste, click [here](#full_code).

<br>
## Code breakdown

The converter is best written as a function, convenient to keep in a separate python helper file and imported as needed. It takes a date, ie. a string hopefully containing all the ingredients we need.
> def toiso8601(date):

First, we write a sub-function that converts a month, written as a word, to a number (still written as string). This sequence can be potentially executed twice in our code, so there is no point repeating the whole thing. Instead, we make a function of it and call it when needed.

{% highlight python %}
	def month_to_digit(month_as_word):
		months_full = {'january':'01', 'february':'02', 'march':'03', 'april':'04', 'may':'05', 'june':'06', 'july':'07', 'august':'08', 'september':'09', 'october':'10', 'november':'11', 'december':'12'}
		months = {'jan':'01', 'feb':'02', 'mar':'03', 'apr':'04', 'may':'05', 'jun':'06', 'jul':'07', 'aug':'08', 'sep':'09', 'oct':'10', 'nov':'11', 'dec':'12'}
		month_as_word = month_as_word.lower()
		if month_as_word in months_full.keys():
			month_final = months_full[month_as_word]
		elif month_as_word in months.keys():
			month_final = months[month_as_word]
		else:
			print('ERROR: not a valid date. month word unknown')
			month_final = None
		return month_final
{% endhighlight %}

There are two types of month words popular in writing dates in English. The first one is full words, the second the first 3 letters of each word. In ISO 8601, month is a two-digit number. And so we have to convert.

For that, we have two dictionaries, one for each type. The keys are the month-words, the values are numbers to which they convert. Alternative is to create one dictionary and test for full name against full key name, or for first three letters of the parsed string against the first three letters of each dictionary key, saving us from the need to create the second dictionary.

Notice that the numbers are in quotes. They are strings, first because '03' is not a number understandable by Python, second because the entire process is taking string input and outputting another string. There is no reason to switch to any other data type at any point, as sooner or later it would have to be converted back into a string.

If conversion fails (ie. the word doesn't match a month name in either dictionary), we set month_final to None. At the end of our function, we make a condition that year, month and day all have to be set and only then do we generate and return the final date. Otherwise, we'd let some gibberish word posing as month pass through to the final string.

There is no ISO 8601 date without a year, and setting it to some default is a risky business. You could, for example, decide to go with '0000'. If such year has no reasons to exist in our data (let's say we have a movie database), setting it to such would conform to the ISO 8601 standard while giving us a valuable information that that particular piece of data is missing. In some datasets, the data has to exist, even if our database tool of choice allows setting datetime fields to be empty. This would force us to use a solution like that.

Since for my usecase it's not a problem, if a four-digit number indicating year is not found, the function informs about the problem, returns None and that's the end of it.

{% highlight python %}
	year = re.search('\d\d\d\d', date)
	if not year:
		print('ERROR: not a valid date. year missing')
		return None
	year = year[0]
{% endhighlight %}

The last line of that chunk of code extracts the string from a regex object. If we had it done earlier (not knowing if regex found anything), pointing to a non-existing item would throw an error. That behavior we do not want, as it would force us to use it inside a 'try' clause.

After setting up a year variable, we now need to delete this number from the date string and only seek in that new variable. Otherwise, it would extract sub-numbers from it as potential month and day numbers.
	date_no_year = re.sub(year, '', date)

Then, we proceed to look inside that newly cut string for a word indicating month and as many sets of digits as we can find. The combination of these two findings are the base for our final assessments on what data we have and how to process it.

{% highlight python %}
	month_day = re.findall('\d\d?', date_no_year)
	month_word = re.search('[A-Za-z]{3,}', date_no_year)
{% endhighlight %}

For that, we use regex module's 'findall' and 'search' functions. Both month and day can be a single, or a double-width string (hence '?' by the second one, meaning "0 or 1 of it"). Month words only contains letters and all the words in both dictionaries are of 3+ length. '{3,}' means "minimum 3, maximum unspecified, so equal or higher than 3".

Being specific here is important. If someone would surprise our converter with a peculiar date, like 1969x05x20, our function would still work and omit 'x' separators, as they are only of length 1. Not demanding three or more letters in a word, those x-es would be added needlessly as separate 'month or day' items and cause the script to fail to fullfill its duty and return None.

Now we get into different scenarios.

{% highlight python %}
	if month_word and len(month_day) == 1:
		month = month_to_digit(month_word[0])
		day = month_day[0]
		if len(day) == 1:
			day = '0' + day
{% endhighlight %}

First is the case of month being a word (as in containing letters, not in regex sense, where digits also qualify as words) and month_day search returning only one object. If we have a year, some word and some digit, it is safe to assume that the word is a month and the digit is a day. So we set up both variables, giving us a first potential complete set of year, month and day variables.

The last two lines are important. ISO 8601 does not accept single-digit numbers, neither for date nor for time. If the input format is 'Sep 1, 1995', the day needs '0' in front of 1.

Because strings are iterable in Python, it is very easy to check for its length and if it's 1 (which means the number is most likely 1-9 with no 0 in front), we add it and the day variable is complete.

{% highlight python %}
	elif not month_word and len(month_day) == 1:
		month = month_day[0]
		day = '01'
{% endhighlight %}

The first alternative check is for no word indicating month and just a single digit found. In most cases, when we only have a year and a single number, that single number is a month. Nobody writes year and day of a month that is not specified.

In rare cases, this extra number means day of year. This converter is not equipped to deal with such scenario and if it is indeed what you have to deal with, you'd have to use Python's datetime module to convert it to a final date. It is easily achievable once you know this primary tool for dealing with dates in Python.

In many scenarios, when the day is not found, we might want the whole date to be invalid. For my use, I still need the date, which is only used for getting approximate guesses to another object with another date (the ones with smallest timedelta between them being paired), so few days off are much less hurtful than not having a date at all.

If you want to not return any date if the day is missing, it is easy to change in this function. Because the final date generation will fail if any of the date elements is not set, all you have to do is change "day = '01'" to "day = None" in two places.

{% highlight python %}
	elif month_word and not month_day:
		month = month_to_digit(month_word[0])
		day = '01'
{% endhighlight %}

Next is a straightforward case, where we have a word for month, but no extra number that would represent a day. Here, again we blindly set the day to first of the month.

{% highlight python %}
	elif not month_word and len(month_day) == 2:
		month = month_day[0]
		day = month_day[1]
		if len(day) == 1:
			day = '0' + day
{% endhighlight %}

Lack of a word representing month and two sets of digits indicates both month and day being numbers. Like mentioned in the function summary, this part assumes month number being to the left of day number. You can reverse this order by setting month to month_day[1] and day to month_day[0].
 
{% highlight python %}
	else:
		print('ERROR: not a valid date')
		year = None
{% endhighlight %}

Finally, we write what to do if all our previous conditions failed. If we got to it, this means that either of these is true:
- word representing month was not found and regex was not able to fetch any numbers from string, leaving us with no data to work with
- besides year, there were more than two other numbers

Three digits make it impossible to assess which one is month and which one is day. Usually, it means that the string we got is not date at all, or date with some unfortunate extra luggage.

One way or the other, under this circumstance it is best to not play wild guesses and instead call it quits. Print will inform us about the problem in a terminal window. Setting year to None will make the final condition fail, as it tests for the presence of year, month and day. 'None' is faulty in Python, it indicates failure, and so the condition is not met.

The final piece of our code checks for the presence of all objects and constructs a string from it.

The last return is silent and applies only when that condition is not met. Every function in Python returns None, if not specified elsewhere. Here, we want to return date, but only if we have all the ingredients. If not, there are no final return instructions, which means the function will return 'None'.
 
{% highlight python %}
	if year and month and day:
		final_date = f'{year}-{month}-{day}'
		return final_date
{% endhighlight %}

In other words, the final lines could be more redundantly written this way:

{% highlight python %}
	if year and month and day:
		final_date = f'{year}-{month}-{day}'
		return final_date
	return None
{% endhighlight %}

First return reached instantly ends the function, and so it reaches 'return None' only if year, word and month are not set to anything truthy.

<br>
And here is the entire code:<a id="full_code" />

{% highlight python %}
	import re
	def toiso8601(date):
		def month_to_digit(month_as_word):
			months_full = {'january':'01', 'february':'02', 'march':'03', 'april':'04', 'may':'05', 'june':'06', 'july':'07', 'august':'08', 'september':'09', 'october':'10', 'november':'11', 'december':'12'}
			months = {'jan':'01', 'feb':'02', 'mar':'03', 'apr':'04', 'may':'05', 'jun':'06', 'jul':'07', 'aug':'08', 'sep':'09', 'oct':'10', 'nov':'11', 'dec':'12'}
			month_as_word = month_as_word.lower()
			if month_as_word in months_full.keys():
				month_final = months_full[month_as_word]
			elif month_as_word in months.keys():
				month_final = months[month_as_word]
			else:
				print('ERROR: not a valid date. month word unknown')
				month_final = None
			return month_final

		year = re.search('\d\d\d\d', date)
		if not year:
			print('ERROR: not a valid date. year missing')
			return None
		year = year[0]

		date_no_year = re.sub(year, '', date)

		month_day = re.findall('\d\d?', date_no_year)
		month_word = re.search('[A-Za-z]{3,}', date_no_year)

		if month_word and len(month_day) == 1:
			month = month_to_digit(month_word[0])
			day = month_day[0]
			if len(day) == 1:
				day = '0' + day
		elif not month_word and len(month_day) == 1:
			month = month_day[0]
			day = '01'
		elif month_word and not month_day:
			month = month_to_digit(month_word[0])
			day = '01'
		elif not month_word and len(month_day) == 2:
			month = month_day[0]
			day = month_day[1]
		else:
			print('ERROR: not a valid date')
			year = None

		if year and month and day:
			final_date = f'{year}-{month}-{day}'
			return final_date
{% endhighlight %}
