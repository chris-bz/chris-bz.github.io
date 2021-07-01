---
layout: post
title:  "Ranger configuration extras"
date:   2020-04-01 22:43:18 +0200
categories: guides
---
Ranger is probably the single most powerful file manager application on any operating system.

It can be customized to the point of practically being a system wide remote, one tool that can do anything. In this regard, it's a very neat extension of command line in Linux.

Because of that, almost every Ranger config file I've stumbled upon contains at least an interesting line or two. I've used the application for few years already, but feel like there's a lot that still needs attention. Like, for example, a convenient way of dealing with archives of all sorts, or different file information views, depending on a particular need.

Below are some of my favorite additions to the rc.conf file. You can usually find it in ~/.config/ranger/ folder. If it's not there, run this command in command line: 'ranger --copy-config=all', which should create it.

Just paste the quoted lines wherever you see fit inside that file, but keep in mind that some of those commands invoke programs you will have to install.

<br>
## Keyboard layout change

Using alternative keyboard layouts can be very painful for that person's friends and family. If they sometimes need to operate on this person's machine, after each session they might hate that person a little bit more.

Dvorak and Colemak are the unsung heroes of filesystem encryption, making all attempts to search Youtube, finding something out on Google etc. fruitless, unless someone's brain knows the layout. Even with keycaps being changed accordingly.

In order to help those poor people, we can set up easy keyboard layout change to and from basic qwerty US layout to any we want. For that, we need to install setxkbmap with our package manager. For Ubuntu users, the command to paste in command line is:

> sudo apt-get install setxkbmap

Then, insert these two lines to rc.conf:

> map Ku shell setxkbmap -variant ,us  
> map Kc shell setxkbmap -variant colemak

'Ku' and "Kc' are my key combinations. You can set them to whatever you have unmapped. Then you need to restart Ranger and whenever you want to change the layout, you just use those key combinations.

<br>
## MP3 file tag preview

Sometimes, it is convenient to check the content of an mp3 tag. For example, a track we have on hard drive might just have artist and title in its filename, but we want to quickly check from which album it is, or which year it was released.

Opening a browser for a Rate Your Music/Discogs/etc. check might take longer than getting to our music directory, quickly locating that file and... pressing a key to see its tag contents.

For that, we're going to need a CLI tag viewer. I highly recommend eyeD3, available both as a CLI app and importable Python library. So, first in command line:

> sudo apt install eyed3

And in Ranger config file:

> map T shell clear; eyeD3 %s; read -p "----------press ENTER to continue-----------"

We are making a 3-step operation here:
1. clear terminal window, so that tag information can be the only thing displayed, so we don't have to strain our eyes looking for it
2. pass selected file(s) to eyeD3 for tag preview
3. using read to display some information

We need step 3, read with -p option (prompt). It waits for user input, enabling us to view the tag before coming back to Ranger.

Otherwise, we'd have to quit Ranger, display tag and have to start it again manually. Without the '-p' option, Ranger would display tag information and then instantly come back, requiring photographic memory coupled with Spiderman instincts to pick up the information we want. It would only be visible for fracture of a second.

Notice that you can select multiple desired files, just like you normally mark them for operations in Ranger, and for each of them, tag information will be displayed in vertical order. Very handy.

<br>
## MP3 player playback control

In my opinion, it is much better to have a system-wide playback control shortcuts, than narrowing it down to a specific application. This way, we can change tracks while the monitor is off and never have to worry about what's focused. Even if we spend large majority of time in our file manager.

If you don't have any free system-wide available key combinations for that porpuse, you can let Ranger do the ordering. So long as your music player accepts playback commands from terminal session (if it doesn't, you need to change your music player).

Ordering player to start playback of hovered/selected file(s) is as simple as setting up your default music application in Ranger's rifle.conf file. But maybe we want to enqueue instead of play?

In this case, we have to check what playback control commands particular player accepts. Most applications will give such information when starting it up from terminal window with ' -h' (short for 'help') added at the end.

In DeadBeef, enqueing is done with '--queue' parameter. Entire command looks like this:

> map de chain shell deadbeef --queue %s; mark_files all=True val=False

You need to replace it with your player of choice and its equivalent command (which might very well be exactly the same).

The second part is also important. Ranger has a nasty habit of leaving selection after operation and no way to change that behavior, as far as I know. There is an option to drop all selections when leaving folder, but it doesn't work on my computer.

It is not a problem, because we can slap the above 'mark_files' command whenever we want to unselect after an operation and skip it whenever we don't, giving us full control.

<br>
## Starting Twitch streams

Youtube-dl is a great tool to download Youtube videos, but it can also be used in concert with MPV to play its videos, or playback Twitch streams.

For that, we of course first need to both have MPV and youtube-dl installed. Then, it's as simple as adding one line for each stream to rc.conf:

> map qan shell killall mpv; mpv https://www.twitch.tv/boxerpete </dev/null &>/dev/null &

First part is optional and it kills all mpv instances. Since I want to play a stream now, first I want to end playback if something is on already, as I'm not gonna watch two things at once.

Then we start MPV with Twitch streamer url address. The player automatically invokes youtube-dl and begins stream playback.

In order not to freeze Ranger until stream playback is finished, which could be in 23 hours and 59 minutes in case of a 24-hour stream, we need to order it to start as separate process, hence line ending.

Or are you like me and sometimes watch beautiful places on webcams while working, to cheat yourself that you're not sitting inside dark cave until the work is done?

> map qge shell mpv https://www.youtube.com/watch?v=yMSc-qqW3To --no-audio --no-resume-playback </dev/null &>/dev/null &

<br>
## Open with...

Default applications often don't meet all our requirements for file handling. We might be using one application for default action, but also need another option for marginal cases.

Take images, for example. For casual users, default action could be to open them in image viewer. Sometimes, we might need to edit them, carve something out and send it to someone? It's good to prepare for such cases:

> map wp shell pinta %s</dev/null &>/dev/null &

Pinta is a simple image editor for Linux. It is an equivalent of the old Paint Shop Pro on Windows, back before Corel bought them.

For simple modifications like slicing particular areas out of an image, there is no point to start any graphical powerhouse, as we'd waste a lot of time looking at loading screen while the program computes stuff we will not need.

Maybe I need to open file in an editor I'm not using daily:

> map us shell subl %s

Sublime Text spawns separate process by default, so it doesn't require command ending to make it to.

Another use is to start program with a specified geometry. I want to start MPV player, but want it on my second 4:3 monitor:

> map uv shell mpv --geometry=1280x1024+1920+0 %s</dev/null &>/dev/null &

We could add '--fs' before sending output to /dev/null to also start the program in full screen, but my experience is that it's bugged and sometimes fullscreens on first monitor, sometimes on second.

Omitting it and specifying geometry is successful 100% of the time and there is no practical difference. It can be annoying if someone often click-drags the player window by accident (fullscreen prevents moving window around). But in this case it's just better to disable, with the exception of someone using mouse and click-dragging MPV window by accident. If that is your problem, just add this line to your mpv.conf:

> no-window-dragging