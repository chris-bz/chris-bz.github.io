---
layout: post
title:  "Using commands to resize programs in Linux"
date:   2020-01-14 14:45:20 +0200
categories: guides
---
Neither Linux nor Windows remember window size and position for applications.

Minority of programs remember dimensions and position on exit. For the rest of them, a troll is sitting inside PC case and rolls his dice. "Where should we put it this time?".

It is sad that something so basic and important is still largely ignored by people behind both operating systems and we have to look for separate tools to try and fix that while much less important issues take precedence.

In this article, we will explore options to modify window sizes and positions through commands and scripts.

<br>
## Geometry

Ocassionaly, Linux applications support setting geometry, which more than solves this problem, but only for those apps.

Geometry is a special string containing four numbers:
- window width
- window height
- starting position in pixels from the left screen border
- starting position in pixels from the top screen border

We can append this special geometry string to some programs when we start them from command line. This functionality has to be written by the developer into the app.

There are also programs which can set any graphical window's parameters through special commands. The below article explains both in detail.

First two parameters decide how big the window is vertically and horizontally, the other two decide where the top-left corner of the application is on our monitor. Here's an example:  

> 1200x600+20+0

Assuming we have a 1080p monitor, it's exact resolution is 1920x1080. Here, the program window is being set to 1200px wide and 600px high. It will occupy a considerable amount of space on our monitor.

The third number indicates that we want our program to be 20px from the left border of our monitor. The last one means it will start at pixel 0 from the top. In other words, there will be no space between our program's top border and monitor's top border.

Notice two pluses at the end. We can either put '+' or '-' in front of each of the last two numbers. Plus means 'pixels from the left edge of the screen', minus means 'pixels from the right edge of the screen'.

Sometimes it is more convenient to align our windows from the right side, so it is great that we have this flexibility.

Of course, pixels then align towards the left side, just like when we get an order to move our piece of furniture 20cm from the right wall, we move it to the left instead of trying to demolish the wall and put it inside ;)

Without the option to align from the right side with minuses, if we'd want to set it 20 pixels from the right, we'd have to calculate each time: current monitor resolution - (window width + 20px). It would be very obnoxious to do it.

Not all window managers on Linux support geometry, so make sure that the one in your distribution does before starting to tinker with it. Fortunately, all the popular ones (and many less popular ones) do, so it is likely that yours does to.

Another thing to consider is that some window managers have different window 'states'. For example, in i3 windows can be either floating or tiling. Geometry only works for floating windows. But if you're using a tiling window manager, you most likely do that on purpose, and so already know about that distinction.

Like mentioned at the beginning of this article, some programs support setting it on program startup. In those cases, the magic formula almost always looks like in this shell command:

{% highlight bash %}
mpv --geometry=1280x1024+1920+0 
{% endhighlight %}

It is super useful for multi-monitor setups. Sometimes, we want to start the player on monitor one. Tutorials, we might want to watch on monitor 2, a TV show on our TV, which is set up as monitor 3.

We can have different commands to start the same player in these different contexts, which is much more powerful than system-wide memory of last known window state (which nevertheless needs to be implemented).

<br>
## Getting window geometry

Finding out exact position and dimensions of our window does not require print-screening entire desktop and then probing it in an image editor.

It's very simple. We just resize the window to our liking, position it where we want on our screen and run a command to find out its dimensions.

For that, a program called **xwininfo** is needed. It has many more uses than just geometry, but since right now it is all we need, we enter this command into our console:

{% highlight bash %}
xwininfo | grep geometry
{% endhighlight %}

Right after executing it, terminal window will freeze and our cursor will most likely change to a plus sign. All you have to do now is click on the window whose parameters you want.

The second half of this command means that its output we will pipe through a grep search. That command will print many lines, but we are only interested in one which contains geometry, so we order grep to only return us lines which contain the word '*geometry*'.

<br>
## Introduction to wmctrl

Many people who work on their computers have a small set of programs that they run each time they start working. For entertainment, often there is another set of programs etc.

Some applications need to have different window properties during work than during slacking off. For example, a terminal emulator window could be smaller during leisure time to give more screen estate to the internet browser, but larger during work hours when Vim or Kakoune needs to be run inside it.

Because with Linux sky is the limit, we can create extremely convenient shortcuts to instantly get the desktop design we want.

The tool that makes it all possible is **wmctrl**. It is a simple, lightning fast application that looks for specified window on our desktop and gives it various behavior commands, like maximize it, change its geometry, close it and even change its title.

First, we need to install it with our Linux distro's package manager of choice. In case of Ubuntu/Mint etc., the command is:

{% highlight bash %}
sudo apt install wmctrl
{% endhighlight %}

Let's start by changing geometry of a single window. We're gonna need to give wmctrl some identification, so it knows which window we want to tinker with. Previously used *xwininfo* command can be used here as well.

Write this in your console window:

{% highlight bash %}
xwininfo | grep "Window id" | grep -Po '(?<=").+(?=")'
{% endhighlight%}

...and then click on the window you want. You should get the window title in your console.

The command may look complicated if you don't understand regex and piping, but in reality it's simple. It filters the output of xwininfo first to just get us the line where the title resides and then extracts it from inside brackets so full title is the only thing we get.

It is easier and makes things faster to go these extra two piping steps and get us just what we want, plus if need be, we can point a bash variable to it, or save it to a file for future use.

The full command may not work in some edge cases (like browser with focused tab containing brackets in page title). If it doesn't for you, just write '*xwininfo*' in your console and the title will be inside brackets next to Window id number.

Write down that title. Now, let's look at wmctrl syntax for changing window geometry:

{% highlight bash %}
wmctrl -r TITLE -e GEOMETRY
{% endhighlight %}

One last obstacle. Wmctrl does not accept geometry in its standard format. You have to take those four numbers and re-arrange them to fit wmctrl's standard, which is:

> GRAVITY,PX_FROM_LEFT,PX_FROM_TOP,WIDTH,HEIGHT

As you can see, wmctrl takes another item, here right at the beginning.

That first number is *gravity*, which specifies the alignment, source of reference of the moved window. You most likely don't need to understand what it is and can always set it to 0. Just remember to put it there. You can safely skip the next section describing how it works.

<br>
## Gravity details

As you can see, we don't have pluses and minuses to specify if the window should be moved "to the right from left side of the screen", or "to the left from the right side of the screen".

Instead, we specify that reference point as if the screen would be a compass. We have four cardinal directions (north, south, east, west) and four intercardinal (northeast, northwest...) between them.

Giving 0 refers to system default, which most likely is north-west. That means we are aligning from the top-left corner of our screen. If we'd give it a gravity of 3 (north-east), we'd be adjusting from the top-right corner, just like when we give minuses in front of numbers when setting geometry.

Below are codes for all 8 gravity directions. Keep in mind that many window managers do not support this and instead will always secretly adjust to system default, no matter which number we feed it.

> NorthWest (1), North (2), NorthEast (3), West (4), Center (5), East (6), SouthWest (7), South (8), SouthEast (9) and Static (10) 

<br>
## Wmctrl traps

Read this part if your wmctrl command does nothing, or if you intend to use it often and want to know some caveats in advance.

Let's look at an example of a wmctrl command:

{% highlight bash %}
wmctrl -r "Sublime Text" -e 0,30,20,800,1080
{% endhighlight %}

The program first generates a list of currently existing windows, finds first title that *contains* a string specified within brackets and then sets it to gravity 0, position from left 30px, position from top 20px, window width 800px and window height 1080px.

Because only the first found item gets adjusted, if we have few windows which are really one program, only one of them will be done and each next command given will still reposition just that one window.

Another potential issue is that windows can have surprising titles sometimes. Browsers troll the most if we seek inside window title.

A website we are visiting might have a "Sublime Text registered version cost" title, which could in result make your Firefox have a "Sublime Text registered version cost - Mozilla Firefox" title whenever that particular tab is focused.

If those 3 conditions were met:
- the title we order wmctrl to search for is 'Sublime Text'
- the browser is higher in wmctrl's item search order than Sublime Text (and so would be checked first)
- the title of the currently focused Firefox tab includes a string 'Sublime Text'

...then wmctrl would adjust Firefox's parameters instead of Sublime Text's.

There are multiple ways to solve that problem:
1. use wmctrl with '-x' to search WM_CLASS instead of title, and that you can find with 'xprop \| grep WM_CLASS' command (followed by clicking a desired application window, just like with xwininfo)
2. use it with the '-F' option to match exact string to the letter, instead of looking for string inside app title ('Sublime Text' only matches if the application title is 'Sublime Text', it will fail if it is 'How much does Sublime Text cost - Mozilla Firefox'
3. change conflicting application titles right before searching (for example: 'wmctrl -r " - Mozilla Firefox" -N Firefox' and Sublime Text resizing wmctrl command right after)

If you are playing title guessing games in your scripts, it is generally a good idea to set application titles to some unique strings that are not likely to get matched (':.:BROWSER:.:', ':.:EDITOR:.:', ':.:MP3PLAYER:.:'...) in all applications that give an option to change their titles. Many do, many don't ;)

If you can't, it is good practice to order your resizing commands starting from applications most likely to cause conflicts (ie. those whose names dynamically change based on file opened, website visited etc.), through less likely to do so, with the ones which never will (due to constant unique name) at the bottom. For the last ones, the order doesn't matter.

For each that can cause problems, follow the line in which you resize with the line that changes that same program's name (the mentioned '-N', you can see it below in an example file). This will ensure that the application won't cause any conflicts.

Don't worry if you have to do a lot of title renaming - both repositioning and renaming are trivial operations from your processor's point of view. You will never see any performance hit, no matter how old your computer is and how many windows you shuffle around.

Again, many apps offer an option to set a permanent app title and your fixing attempts should start there. Some even allow having different titles when starting them with different profiles. Fantastic terminal emulator called [Tilix](https://gnunn1.github.io/tilix-web/) is a good example.

<br>
## Creating a script file to resize multiple windows

Each wmctrl command only changes one window, so we need to invoke as many commands as windows we need changed.

To contain all these instructions, we will use a bash script. Traditionally, in its first line we need to point to the interpreter. Most often, the line you need is:

{% highlight bash %}
#!/bin/bash
{% endhighlight %}

Then, in each line a separate wmctrl command. No need for colon, semicolon or any other sign by line end, just one command in each line.

The quickest way to write such script is to set up exactly the desktop layout we want. Align everything so it looks perfect. Then, use the mentioned xwininfo commands to get each window's geometry and title. Write everything down.

Next, write wmctrl commands. Remember to convert geometry strings (WIDTHxHEIGHT+FROMLEFT+FROMTOP) to format that wmctrl accepts (0,FROMLEFT,FROMTOP,WIDTH,HEIGHT) in each command.

A complete example winresize.sh file looks like this:

{% highlight bash %}
#!/bin/bash
wmctrl -r Firefox -e 0,942,18,961,1042
wmctrl -r Firefox -N Firefox
wmctrl -r 'Sublime Text' -e 0,942,18,961,1042
wmctrl -r qBittorrent -e 0,942,18,961,1042
wmctrl -r 'Google Chrome' -e 0,942,18,961,1042
wmctrl -r 'Tor Browser' -e 0,942,18,961,1042
wmctrl -r Kid3 -e 0,942,18,961,1042
wmctrl -r Pinta -e 0,394,17,1510,1044
wmctrl -r LibreOffice -e 0,394,17,1510,1044
wmctrl -r QuiteRSS -e 0,942,18,961,1042
wmctrl -r easystroke -e 0,942,18,961,1042
wmctrl -r Telegram -e 0,16,95,359,605
wmctrl -r deadbeef -e 0,16,720,359,340
{% endhighlight %}

Don't forget to order them from most conflicting to least conflicting and follow each problematic application with another wmctrl command that renames it. Above, you see me doing it with Firefox.

<br>
## Making convenient shortcuts for resizing

The final step is to set up some way to execute our new script at will. Below are main 3 categories of options we have.

1. create a bash alias for execution of that file (in file *~/.bash_aliases* add line: *resizer='/path/to/script.sh'*), source that file (or restart the system if you don't know how) and then execute it from console by writing *'resizer'*
2. set up a system-wide keyboard shortcut for that script (different for every window manager, in i3 it's as simple as adding this line: bindsym F9 exec /path/to/script.sh)
3. create a mouse gesture to execute a script (easily achieved with easystroke, choosing 'command' option and then in command just pasting /full/path/to/our/script.sh

And voila! We now can reshape all our desktop with something as simple as a single command, pressing of a key, or a mouse gesture. We can create as many of those files and shortcuts as we want.