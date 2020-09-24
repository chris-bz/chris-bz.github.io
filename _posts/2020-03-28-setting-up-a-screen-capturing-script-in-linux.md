---
layout: post
title:  "Setting up a screen capturing script in Linux"
date:   2020-03-28 12:13:54 +0200
categories: scripts
---
Microsoft Windows came a long way since just storing desktop image in clipboard after pressing print screen key and requiring us to open it in an image editor, pasting and only then saving to disk.

Windows 10 has *Snipping Tool* and *Snip & Sketch*, which together give many interesting methods of selecting parts of the screen we want to capture. It shows unfinished work that there are two tools for the same type of job, but that's just final bit of awkwardness Microsoft has to sort out.

Most Linux distributions are also more equipped these days. But what if we use some niche window manager that doesn't have this functionality built-in? Or if it stopped working for some reason and we can't find a way to make it work again?

Fortunately, so long as we can boot our window manager in Linux, we can make it work with some basic knowledge and just one tool (which might even be installed by default in your distro).

You will need to know how to set up system shortcuts, though. Every window manager on Linux can probably do that. In case you don't know how, make a Google search for 'how to set keyboard shortcuts in [your Linux distribution, or window manager here]'.

This tutorial will explain how to do that in the i3 window manager.

<br>
## What does this script do?

Pressing **print screen** will capture the screen and save it to an image file, in any location you want. You can choose image format as well, all the popular ones are supported.

Pressing **scroll lock** (another ever-busy key on standard keyboards) will also do that, but after saving, it will open an image editor of your choice with that file loaded.

<small>
It is a common task to screen capture something, some video stream we're watching perhaps. But before we post it online, we want to mark some interesting part of our screen with a circle, or point to it with an ugly-looking arrow.
<br>
<br>
Instantly opening image editor is a lot more convenient than saving the image to disk, then manually navigating to the folder in which the image resides and opening it. If that's not enough, we would usually need to use the 'open with' option if plain image viewer and not editor is set as the default program for executing images.
</small>

After pressing the print screen key, the script first checks the folder in which we save capture files if any already exist. If they do, it takes the number of the newest one (the one with biggest number), increments that number by one and saves our new screen capture with that number.

In other words, if you already have print-screen1.jpg and print-screen3.jpg (let's assume 2 has been deleted in the meantime), the script will create print-screen4.jpg instead of substituting print-screen1.jpg, or creating print-screen2.jpg.

If there are no print-screen[num] files in folder, it creates print-screen1.jpg.

This is hands-down the best default behavior, as it retains chronology and prevents sometimes unwanted file loss.

<br>
## Requirements

We're going to use **Bash** as our shell language of choice (default in most popular Linux distributions).

The only program we need for the job is **import**. It is part of Image Magick.

Image Magick is probably the single most powerful tool for command line slash scripting image processing that was ever written. It is worth getting to know it if you deal with images a lot and feel like you could automate some of your tasks.

One example of its use is having **watch** monitor a particular folder where we save memes (the most important images on anyone's hdd). We can order Image Magick tools to check if maximum resolution is not bigger than something and if that's the case, reduce it to some sensible maximum.

Next, the script could check if those meme files have compression level higher than, let's say 90, and if they do, we compress them to 90 to save disk space. Then, we could automatically move those files to our final meme directory.

This is a very basic example, but it already shows how useful this set of tools can be.

You will most likely find it in your system repository. It is spelled **imagemagick**.

<br>
## Code

{% highlight bash %}
#!/bin/bash
ps_numb=$(ls -v /path/to/print-screens/ | grep print-screen | tail -n1 | grep -Po '\d\d?(?=\.png)')
ps_numb=$(($ps_numb + 1))
import -window root -crop 1920x1080+0+0 /path/to/print-screens/print-screen${ps_numb}.png
{% endhighlight %}

<br>
## Code explanation

Thanks to piping, we accomplish many tasks in our first actual line of code:
- **ls -v** - '*ls*' lists files in specified directory, '*-v*' makes sure numbers are sorted properly (with default sorting, for example, 100 is before 2... because 1 is smaller than 2. we do not want that!)
- **grep** - searches directory list for files/directories containing a string 'print-screen'
- **tail -n1** - '*tail*' displays last lines of our result list, '*-n1'* narrows it down to just last line, which we've made sure is the one with the biggest number
- **grep -Po...** - grep extracts single or double digit number from that filename (you can append a second '*\d?*' if for some reason you can end up with 100+ files).
- **(?=.png)** - it is a lookahead, meaning we only look for digits which are instantly followed by '.png'.

That lookahead is a safety measure against directories that contain a string '*print-screen*'. The script may one day encounter such named directory, but it is not likely to encounter a directory named [...]print-screen[num][maybenum].png[...].

Even if such rare thing happens, the script will create file one number higher than the number in this directory, assuming there are no print-screen files with higher numbers. A rare worst case scenario and still nothing hazardous happens.

The second line does arithmetic (which we enable with double parentheses). It adds one to append that number to file name later. If you want a different numbering order, you can change it easily here.

<small>
Bash can take dynamic typing to extreme, as is the case here.
<br>
<br>
If previous line returns nothing (if there are no print-screen files in our folder, or the last sorted one has no digits followed by '*.png*'), the next line takes that nothing and converts it to something arithmetically viable, because it now sits inside double parentheses, which like mentioned before, is used for arithmetic operations.
<br>
<br>
Because there is no data (or false, or an empty string - those are guesses because we can't check data type easily in Bash, someone more knowledgable would have to check ;))... that data is converted to 0, to make it viable in the scary and dangerous world of mathematics. And 0+1 gives 1, saving us from writing our script in a more complicated way to ensure that if there are no print-screen files, proper number for new filename is still generated.
</small>

Notice the file extension in second line and in forth. Change it to any image format you want and test it just to be sure that the import tool supports such format.

The third line is where the magick happens. Import tool is told to capture main display window with '*-window root*' option.

Just like xwininfo, xprop and many other useful X tools, import waits for user click to specify window for capture. But we don't want to capture just one program window, but entire screen, and we want it to happen automatically instead of requiring us to click each time. That's why we have to be specific.

If you only have one monitor, you can safely remove '*-crop 1920x1080+0+0* ' part. I have two monitors, first one set to the 1080p resolution. Import with '*-window root *' captures content from all monitors to one glued together image. I want to get the left-most monitor's content only, so after capturing, I need to crop it to position and dimensions of my first monitor.

That is why the script specifies geometry (you can read more about geometry in [this article]({% post_url 2020-01-14-using-commands-to-resize-programs-in-linux %})). Most people only have one monitor and while the script will work for them as well, they won't need that bit of code.

But because it's there, it will also work for people who have more than one. If yours has different resolution than 1920x1080, change those numbers. If you don't want the left-most one's contents, replace '*+0+0*' with proper starting pixel location.

<br>
## Binding the script to a key

Like mentioned at the start, if your window manager is not i3, search Google for how to add system-wide keyboard shortcuts in your Linux distribution/window manager.

Usually, you can set key to execute a command. Instead of system command, point to a file (with full path, so system knows where to look for it).

In i3, that operation is very simple. We edit this file: '*~/.config/i3/config*', and add these two lines:

> bindsym Print exec /path/to/file.sh  
> bindsym Scroll_Lock exec "import -window root -crop 1920x1080+0+0 /location/to/save/image.jpg; pinta /location/to/save/image.jpg"
 
As far as I know, i3 can only execute single statement, so to execute two, we need to sneak them in as one command inside quotes.

Editing does not require automatic numbering of the following files. After all, we capture, edit and save each time. And so we only use the import command here and not entire script. Be sure it is called with exact same options and parameters as inside your script if you want to capture the same screen area.

*Print* and *Scroll_Lock* keys can be changed to any other ones you want. A list of all available key names which i3 understands can be found [here](http://xahlee.info/linux/linux_show_keycode_keysym.html).

Remember that binding a key, or key combination directly in window manager usually means no application can use it. Otherwise, we'd have one key potentially execute multiple unrelated actions, potentially leaving poor user in the state of confusion ;)

Window managers execute instructions they have for the key and don't send the signal further. Programs don't even know that the key was pressed.