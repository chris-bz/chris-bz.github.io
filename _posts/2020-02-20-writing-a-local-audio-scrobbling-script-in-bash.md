---
layout: post
title:  "Writing a local audio scrobbling script in Bash"
date:   2020-02-20 22:43:18 +0200
categories: scripts
---
<br>
## Last.fm vs local scrobbling
Once merged with Audioscrobbler, Last.fm became the leading tool for tracking music history and probably remains so to this day, despite being considerably less popular now than it used to be.

One of the reasons why it became so popular was because it added a social dimension to music tracking. We can start a conversation about an interesting track we've both discovered recently, or concert which, as it turns out, we both attended.

But what if someone doesn't care about all that and just wants to track down history and store it? In that case, there is probably no reason to keep something as precious as lifetime of our musical adventures on a remote server.

That applies even if the company is trustworthy. And people behind last.fm, both before and after [the CBS takeover](https://techcrunch.com/2007/05/30/cbs-acquires-lastfm-for-280m/?guccounter=1), have not given us any reasons doubt them.

Anything can happen to a website though. A disgruntled employee on his way out might [erase large portion of its data](http://www.digitaljournal.com/article/249311).

Just to be safe, we might want to download our entire listening history. In case of Last.fm, this isn't always easy. The service itself used to provide downloading all user data, but for years that option is gone.

There are other websites which can extract it. But if our history contains hundreds of thousands of items, there is big chance downloading those large files will fail at some point. Fetching data through those kinds of websites also takes a lot of time.

When our history is out there with no backups, then even if that company has many backups on different servers, we can't really be 100% sure that our data is safe. It needs to be backed up somehow.

If downloading last.fm user history fails, the only option left is to write a script that will extract that data from their website, page by page. That can be tens of thousands of pages... and now we have to know web scraping.

But it can't just be a script that we run once and are done with it. It needs to check for new scrobbles periodically, so we lose as little data as possible.

In other words, if we want to continue using a remote scrobbling service, but also want regular and reliable backup plan for our data, we're in a world of trouble.

Why not writing our own script and store musical history on our very own disk, right under our noses? Backing it up further from local file for 100% safety is so much simpler and faster, too.

We won't need any application that might spy on us, to whom we might one day lose password by accident, or which might stop working at one point due to some peculiar bug.

And we need it to be able to scrobble, or our listening history comes to a halt until the problem is solved. Do we not listen to anything until we manage to, just to keep historical accuracy? That would be a questionable solution to a problem that we could do without.

Local script means every time the system is on, scrobbling is on. 100% reliability. Let's explore other arguments for it.

<br>
## Advantages of local scrobbling

- the script is just few lines long, possible to fully understand quickly even for someone who has just started learning Bash
- once everything is set up, it always works. if problematic strings are properly escaped/removed, nothing will break our script
- data can be stored in any format we want. in case we change our mind at some point, converting it to another format using regex is very easy
- with minor modification of this script, we can set up a local SQL server and directly feed it scrobbled data. keep in mind that if you don't plan to do advanced stuff with that data, choosing database over plain CSV file for storage is overkill
- we decide which tag fields should be stored
- deleting unwanted items is as simple and powerful as our editor allows, as opposed to a click-fest if we decide to do it through a website
- even decade-long logs still won't take more than dozen megabytes, which by today storage standards is zero
- we can extract whatever statistical information from it, imagination is the limit (but we have to do the work)

<br>
## Requirements

1. This tutorial uses Bash inside Linux as the environment of choice. We will be issuing system commands, which is always more natural from Unix shell languages than from other programming languages. The latter usually require importing system libraries first and might produce code that is harder to read.

2. For local scrobbling to be an easy ride, you have to use a music player that supports editing its title. Popular standard is for the player to display currently played track's tag information as the application title. We need to be able to change from which tags we get the information.

	If you intend to use graphical application and don't care about library, [DeadBeef](https://deadbeef.sourceforge.io/) comes highly recommended. For console interface, [CMUS](https://cmus.github.io/) is a popular choice. This tutorial will work with DeadBeef.

3. You will need to download **wmctrl**. Your system package manager most likely has it. It is a small, very fast and very useful tool. I recommend getting to know it as it makes many interesting system scripts possible.

<br>
## Configuring DeadBeef
Go to **Edit** -> **Preferences** -> **GUI/MISC** -> **Player**

In the '*Titlebar text while playing*' field, replace the current string with:

> deadbeef: %artist%;%album%;%title%;%year%;%length%

...or any tags you want, in any order. The beginning part is important, we will be identifying program title by a string 'deadbeef: ', so it has to be there.

You might want to change it to something more unique to never match that title elsewhere (in case you one day visit a website whose title contains 'deadbeef: ' or a similar situation). If you decide to do so, remember to change it to something identical both in DeadBeef options and in line 4 of the script.

In the '*Titlebar text while stopped*' field, replace the current string with:

> deadbeef - not playing

It is important that there is no '*deadbeef: ' string here, as different string is a signal that deadbeef is not playing. If we'd add colon here, the script would assume that deadbeef is playing and that '- not playing' is another currently playing track that needs to be added to a scrobbling history file.

<br>
## CSV as our storage format of choice

Semicolons between tags decrease readability, but we are gonna make our log a CSV file, with semicolons instead of commas.

And so to make our life easier, DeadBeef will be displaying tag information already formatted for CSV file write. CSV file is like a table, with each line being a row and cell items are divided by a comma, semicolon or any other symbol you choose. Tab is another good option.

We no longer have COMMA separated values, so it's not a CSV file anymore, strictly speaking. But the idea is the same, so we keep calling it that anyway.

Even though it doesn't look pretty (compare it to YAML, for example), we keep using it for practical reasons. If we ever intend to analyze that data in any way (most popular artist this year, most listened to track ever...), parsing a CSV file is kid's play.

Yaml is the best option for readability, but files can be considerably larger because of it and it can be a nightmare to parse. CSV is simple and reliable.

Because the format is very popular, many frameworks, libraries and other tools have CSV importing/exporting built-in. We won't have to do any additional work to prepare the data ahead of our tasks. With YAML, we might as well need to hire an assistant parser ;)

Remember that the character you chose as field separator can't be present inside your music tags, or it will mess up your data. It's good when all music tracks get pre-cleaned before being played for the first time. Check article list on this blog, I will write such a script and post it here sooner or later.

<br>
## Breaking apart the script

{% highlight bash %}
#!/bin/bash
last_title_path='/path/to/scrobble-last.log'
{% endhighlight %}

We start by pointing to Bash interpreter, which usually is in the main bin folder. Then, we create a variable which points to a string representing the path to a file which stores the last scrobbled track.

We could store the last scrobbled track in memory to avoid disk writes. An average of one disk write every 3 or 4 minutes is not exactly the case of machine cruelty and it won't have any impact on our hard disk lifespan. And if we write to disk, we don't have to be afraid of losing data on reboot etc.

{% highlight bash %}
while true; do
{% endhighlight %}

This is a common programming use for an infinite loop. The idea is to start the script on system startup and repeat some sequence of events forever. For that, we need a loop that under no condition ever breaks.

'While' checks for something being true (or truthy, depending on the language). True is always  true, it's not gonna change at iteration 131. It guarantees the loop will run until terminated from outside (like *kill* command, or system reboot).


Entire sequence of code will be executed line after line, after which next loop iteration will start, ie. it will begin to execute entire sequence again.

{% highlight bash %}
	curr_title=$(wmctrl -l | grep -Po "(?<=deadbeef: ).+")
	prev_title=$(head -n1 $last_title_path)
{% endhighlight %}

The logic of the script is that every 5 seconds, we will check current track against previously played one and only if it's different, we will put it in log.

Wmctrl has a useful '-l' argument, which lists all graphical windows currently open on all our monitors. It also states on which monitor each window currently resides and, what interests us here, program title.

We wrap the system command in '*$()*' and make variable point to its output. The command first lists all open windows, and then pipes it through grep. For the latter, we specify two options:
- '*-P*', which changes default regex engine to Perl's (best Regex version). it needs to be specified because the default grep regex engine does not support lookbehind
- '*-o*', which only returns what regex matches, and not a whole line in which regex match was found

A variable representing previously played track is set by printing it through **head* with the '-n1' option. On default, head prints the first ten lines of a specified file. We just want one line, the first one, which the 'n' option secures.

We can't just **cat** the whole file, because then we'd also get the newline. We don't want it, so we have to remove it one way or the other.

{% highlight bash %}
	if [[ "$curr_title" ]] && [ "$curr_title" != "$prev_title" ]; then
{% endhighlight %}

After defining all the necessary variables, we get to the heart of our script. First, we check if the music player is playing anything, or the second test would produce an error and break our script if it doesn't.

Then, we check if the currently played track is different from the last played one.

{% highlight bash %}
		echo "$curr_title" > "$last_title_path"
{% endhighlight %}

The next step is to replace the last played one with the currently played one inside our scrobble-last.log file. We need to compare future playback against this track from now on.

{% highlight bash %}
		echo -e "$(date +'%Y.%m.%d;%H:%M');$curr_title" >> '/path/to/scrobbles.csv'
{% endhighlight %}

Here, we're constructing the final string and appending it to our main scrobbling file. The desired format features date and time in front, either together in ISO 8601 format, or split by another separator *(in our case, a semicolon)*.

It is convenient to keep it separate from date, as we might want to also extract hourly statistics. Having them separate saves us from additional step of extracting it from another field when we need it.

Our file entries are added chronologically, so it's good to have time of addition in front. And since date and time are always same width, readability does not suffer.

{% highlight bash %}
	sleep 5
{% endhighlight %}

Before closing the if statement content and the loop, we order the script to sleep for 5 seconds. You can do so every any other time interval. Linux's sleep program is very flexible (unlike Python's, for example), not only allowing seconds as a unit. The smallest unit (in theory) and the program default is seconds, though.

We can even use floats, so you can order sleeping for .1 to get ten checks every second, giving a 50 times greater resolution than the default, and producing near-instant additions to the log. It is a trivial task for the processor, so you don't have to worry about performance.

For me, 5 seconds is precise enough. It's gonna check everything anyway (unless all you listen to is grindcore), and few second delays are completely irrelevant for statistical operations I am doing on the scrobbled data.

One case when setting lower time interval is important is when we need statistics of unfinished tracks. Then, we'd have to calculate timedelta between beginning of playback of one file and beginning of playback of the following one to see if it's shorter than track length and by how far.

If we need that 'how far', few seconds will seriously screw up the data and so sleep needs a very low value.

<br>
## Final script

{% highlight bash %}
#!/bin/bash
last_title_path='/path/to/scrobble-last.log'
while true; do
	curr_title=$(wmctrl -l | grep -Po "(?<=deadbeef: ).+")
	prev_title=$(head -n1 $last_title_path)
	if [[ "$curr_title" ]] && [ "$curr_title" != "$prev_title" ]; then
		echo "$curr_title" > "$last_title_path"
		echo -e "$(date +'%Y.%m.%d;%H:%M');$curr_title" >> '/path/to/scrobbles.csv'
	fi
	sleep 5
done
{% endhighlight%}

<br>
## Making the script run on system startup

We want the script to always run whenever the system is up.

The alternative is to either remember to manually start it before playback (stupid), or have the script monitor music player for beginning of playback and turn itself on when the playback is on and turn off when the playback is off (pointless).

While technically the second solution is cleaner, it has no practical meaning, while introducing a mechanism that perhaps could get broken one day if we don't prepare for some rare exotic scenario bound to happen sooner or later.

In any modern operating system, every second many things are happening in the background. Addition of another tool performing such trivial task is irrelevant.

After we have our script in a file, we need to make it run on system startup. There are many options for that. Below are two popular ones.

### Put it in crontab

Crontab file contains instructions on running applications. Every Linux user profile has its crontab file. We can put precise time when we want to run something, specify days of the week when we need it run, run it every x minutes and many more.

It is a very flexible tool and it needs to be, since Linux is mainly used for servers, and servers often need complex task schedules.

One special use in crontab is to replace all the normal specifications of when it needs to be run with *@reboot*, so the command is executed on system startup and only then. In the case of our scrobbler.sh file, the entry looks like so:

{% highlight bash %}
@reboot	/path/to/scrobbler.sh
{% endhighlight %}

That line needs to be added to the user crontab file, which can be edited by writing this command in console:

{% highlight bash %}
crontab -e
{% endhighlight %}

'*e*' is short for '*edit*'. It doesn't matter which line you put it in.

### Order window manager to run it on startup

Window managers often have configuration files which are loaded on system startup. In those files, we can order starting up of applications, so they run on startup as well. This way, we ensure that every time we load window manager, the script gets loaded as well, which is what we want.

I am using **i3**, in which I need to edit ~/.config/i3/config file and add this line to it:

> exec /path/to/scrobbler.sh

<br>
## Limitations

One feature this script is lacking is an ability to detect same track being played few times in a row.

The way the script is designed, if we listen to the same track for half a day, the script does not see track change, and so it cannot see that we got a bit carried away with it.

This is one area in which scrobbler applications perform better. Remember that we're comparing it to 11 lines of code ;)

Since I wanted to write a basic scrobbler, I did not implement checking for track repeats. I'm sure there are many ways to solve this problem, but I can't find one that's good enough and short enough to implement into the script while still keeping it short and simple.

One is to convert track length to seconds (so we get single integer for comparison) and then monitor if the playback of that track spans for more than its length. The refresh in which it does for the first time, we append the log with another instance of that track.

That solution is not perfect though. If we pause the track mid-way for 2 hours and then resume it, it will add second line to log even though we are listening to it once.

Another thing is that services like last.fm have many algorithms for filling out the missing data. If tags contain misspelt information, they check against their databases to try and find a guaranteed match to correct it. We don't have to do this, it gets done automatically.

With our solution, everything that is inside tags is logged directly, and so it is our responsibility to make sure the data is correct. It's more work, but it gives better results than an automated guessing game.

Yet another problem is logging playback from services like Spotify. Popular services of this kind support scrobbling to last.fm. It will get logged the same as if we'd be listening through the music player.

You can try to go around it, but it will most likely be a messy road. You can set up a script to monitor music streaming service applications and extract data from their titles instead, but maybe the customization options for title display are not built into those applications and we'll miss some of the data that we want, like track length.
