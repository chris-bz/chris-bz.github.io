---
layout: post
title:  "JPG compression guide for Linux"
date:   2020-09-16 08:01:21 +0200
categories: linux
---
## Basics

Almost all jpg files out there are compressed to some extent. The compression is on a scale of 1 to 100, 1 being most compressed (do not ever go that low!), 100 meaning no compression.

Recompression reduces image quality. Exactly how much, it varies. With some images, recompressing 100 times with the same compression level will barely decrease image quality, but other times even single recompression can introduce ugly artifacts.

That's why we should recompress only when changing compression level by a considerable amount, and do it as rarely as possible.

Another important thing is to recompress only to lower levels. Recompressing to higher level means the image will take more hard disk space, but at best it will still only look as good as it did before the last compression.

Image quality can't be magically improved by changing to higher compression level, just like details of a completely blurred image can't be made sharp just by increasing image sharpness - the sharp contours and exact object colors just aren't there behind an image, waiting to be exposed.

Recompression to the same quality is always a bad idea. If the previous compressor did a terrible job, the damage is done already. We can only go lower on the scale and test if it's possible to save space while doing minimum of extra damage.

Usually the only good reason to recompress is to reduce image quality so it can take less space on hard disk, or so it can be sent faster on the internet. If you don't have these concerns, don't compress. We can always compress later, but once we lose that image quality, we can never get it back.

<br>
## Finding out compression level of an image

You can check the level of jpg file compression by installing Image Magick (more about it soon) and then running this shell command:

{% highlight bash %}
identify -format '%Q' filename.jpg
{% endhighlight %}

Replace *filename* with the name you want to inspect.

It is extremely useful for scripting purposes to be able to check for image compression level.

Let's say we run a forum website with many users who upload many images. We don't want those images to be just left there on the server.

Going down from 100 compression (no compression) to just 95 rarely produces visible image problems, but it can drastically reduce size!

So, in whichever programming language we're running on a server, we can write a script to get each incoming image's compression level and if it's above a desired number, recompress it with jpegoptim to whichever we want.

You can also check compression level in graphical programs. Look for 'image properties', or something similar, in your program's drop-down menus. Some graphic editors ask for compression level each time we save new image, or even re-save an already existing one.

By default, the slider in some of those programs shows program default for the file extensions, but in others it shows current image compression. You have to find out for yourself how your program behaves.

<br>
## Saving uncompressed image, compressing with another tool

We've established that we want to compress as few times as possible and select the best tool for the job. Let's go through an example to illustrate that approach.

In [another post]({% post_url 2020-03-28-setting-up-a-screen-capturing-script-in-linux %}), I presented a script that (in short) captures screen content to a file. It uses the **import** tool, which is part of Image Magick.

If you don't understand part of its command, check the linked article where it is explained in detail. Here, we will use it in concert with another tool.

There are other command line programs whose only purpose is to compress images. For the jpg format, probably the most popular one right now is **jpegoptim** - and that's what we're going with.

If we really want best image quality for the size, we need to use a specialized tool like that, instead of assuming that the program that produced the image has good algorithm. It might, it might not. We will use it in this short tutorial.

Jpegoptim probably doesn't ship with any Linux distro, so you have to install it manually. First, try from your distro's official repository.

Now, if we order import to save screen capture to a .jpg file, it will compress it to 85, which is the program default.

But since jpegoptim will do a better job optimizing its quality, we don't want import to compress it at all. The first compression might seriously worsen image quality.

We can change import's compression level to any other percentage we want by appending the import command with '*-quality 95*' (or any other number). We need to order it not to compress. We do that by adding '-quality 100' to the command.

Jpegoptim generally has two modes. The first, default one, optimizes the image, leaving its compression level unchanged. It does not recompress (so no quality loss), but only improves compression to squeeze more out of it.

The second we invoke with the '*-m*' option. Executing this command:

{% highlight bash %}
jpegoptim -m70 filename.jpg
{% endhighlight %}

will recompress filename.jpg with 70% compression, using its own optimized algorithm.

Let's take a real life use case. We have a monitor which displays some sort of statistics. Every hour, we need to capture content of that monitor to a file for reference.

We only want to have the latest reference image, so we want it to always write to the same file, overwriting its content every hour.

Depending on some conditions, we might want to rename the file by hand and move it to other folder with images. And since in that folder files will stay forever, we need to take precautions so one day this folder's size does not become a problem. We need to compress.

Our Bash script file only needs to contain two lines, plus path to interpreter on line one:

{% highlight bash %}
#!/usr/bin/bash
import -window root -crop 2560x1440+0+0 -quality 100 /path/to/filename.jpg
jpegoptim -m90 /path/to/filename.jpg
{% endhighlight %}

Import captures our 1440p left-most monitor's screen and saves it to an uncompressed jpg file. Jpegoptim then finishes the work by compressing it.

To generate that image every hour, we save that file and end it's name with '*.sh*'. We then go to a command line window, start editing of current user's crontab file with '*crontab -e*' and paste that line into it:

{% highlight bash %}
0 * * * *	/path/to/our/script.sh
{% endhighlight %}

<br>
## PNG instead of JPG

What if we want to save to PNG file instead? In some cases, it's a better choice. If the image contains transparent elements, it's mandatory since png supports them and jpg does not.

Since it's a popular format, there are many compression tools written for it as well.

**pngquant** is one such tool. It is easy to install on many platforms and is even present in Ubuntu's official repository. To save it to a png file, we need to do two things:

1. Change file extension from '*.jpg*' to '*.png*' in the line with import command.

2. Replace the jpegoptim command line with this one (remember to change image paths):

{% highlight bash %}
pngquant --quality 90 --speed 1 --force --output /path/to/filename.png /path/to/filename.png
{% endhighlight %}

Quality setting is self-explanatory. We set speed to 1 (slowest, but gives best results).

'*Force*" option is needed here, as the default pngquant behavior is that it doesn't overwrite images, and we order the program to save to the same file that it reads from. If we don't specify the output file, it appends its ending to a filename, and for this example we need to overwrite the source file.