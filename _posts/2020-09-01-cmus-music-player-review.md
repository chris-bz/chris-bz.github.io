---
layout: post
title:  "CMUS music player review"
date:   2020-09-01 11:03:28 +0200
categories: reviews
---
![CMUS library view](/img/cmus-music-player-review/cmus1.png)
version reviewed: 2.7.0<br>
installed on: Linux Mint 19

CMUS is probably the most popular terminal-based music player for Unix operating systems. The competition is almost non-existent, and even from that small pool, some applications do not look to meet the same needs CMUS addresses (some basically play a single specified track and that's it).

Despite being very satisfied with my previous player, DeadBeef, I was eager to look for something that could be fully controlled from within by keyboard, and thus free up various system-wide (and Ranger-wide) playback shortcuts with one application shortcut, and give all the commands from inside the program inside the terminal window. Which, of course, has its advantages over graphical interface.

DeadBeef already comes chock-full of keyboard shortcuts, including specialized ones that you'll struggle to find in most music players, like remove currently played file from disk, or look it up on last.fm. Spoiled for choice, I was coming with high expectations.

<br>
## Views

By pressing numbers 1-7, we switch between different tabs, which in CMUS are called 'views'. First one is tree view (aka. library), second is sorted library, third is playlist, fourth is queue. We get three options for three different scenarios, depending on how long do we plan on keeping the file on the list:
- library is for long-term  additions that is supposed to reflect all the music we have on disk (or at least the part of it which we want to have quick access to)
- playlist is for 'current' use
- queue is a list that takes priority - whatever is added there is going to be played immediately after the currently played track and only after 

Majority of users will spend majority of time either in library view, or playlist view, depending of course on which they find more convenient.

The librarians will find separate playback queue extremely useful. Since in library, an order of tracks is predetermined on loading, it is very convenient to press whatever key we have bound to the win-add-q command to add it to music queue. This way, we use library to play entire albums and in between we add to queue whatever music we scavenge outside the current album's directory.

On the other hand, using queue for playlistians is not as useful. In playlist, we set whatever order of tracks played ourselves. And so queue view doesn't address the natural limitations of library view. We end up splitting our playlist into two for no reason.

A better solution for people who use playlist as their primary queuing tool would be to have an extra 'insert into playlist after currently played track" command for cmus-remote. Unfortunately, you'd struggle to find it in almost all music players out there and it's a very handy feature that would completely remove any need to use queue view for most playlist users.

The version 2.7.0, which I installed from official repository, is missing an ability to set the default view on program startup. As a playlist user, I have to manually focus the player and press '3' on each system startup to switch from an empty library window to playlist view.

![CMUS playlist view](/img/cmus-music-player-review/cmus2.png)

An auto-detect feature would be nice as well - if playlist is populated, but library is empty, start with playlist view. If both are empty, but queue contains tracks, focus queue view on startup.

An auto-switch to view when adding to it would also solve this problem, but is missing too. (edit: in version 2.8.0, there is a new cmus command 'start_view', which probably solves the problem)

<br>
## Speed

One of the reasons why someone might want to switch to a terminal-based application is speed. Perhaps the user has a very old laptop and wants to make it usable, but the modern bulky applications slow it down too much.

With music players for people with large music libraries on their hard drives, this is very relevant. Many programs perform well with single playlists and up-to-medium-size libraries, but once the going gets tough, they can start forever and crush often, searching is extremely slow etc.

A program that is displayed in a terminal emulator window does not necessarily have to be fast (Kakoune being a good example), but most of the time switching to a text interface helps to increase system performance on old computers.

Here, CMUS delivers. It can handle ridiculously large playlists with ease. Searching works like a breeze, too. If performance is your primary concern, switching to CMUS is definitely worth considering.

<br>
## Key bindings

Just like so many other terminal applications, CMUS is somewhat inspired by VIM when it comes to navigation. You can move up and downwith home row keys, quit the program with ':q'.

It is possible to move selected track up or down, switch views, increase/decrease volume, pause/unpause, control playback, mark tracks for operations, execute shell commands and more.

Together with options, keybindings reside in a config file. Conveniently, this file is editable from inside the program as one of views. You can reach it by pressing '7'. Lines are highlighted same as tracks are in other views. We roll down or up to reach the line we want to edit and press enter. A colon followed by the command to change set to current value is displayed at the bottom, enabling us to edit.

The operation needs to be finished by pressing enter. That means if we screw up, we have to correct, because whatever we accept with enter will stay in config.

The program checks for validity of input only for options. If it gets the value that doesn't exist, it reverts to last correct setting. If we edit a boolean field and we insert non-bolean value, whatever that is, it will display an error and then set the option's value to false, since we just fed it something that is not allowed, which boolean translates to false. It doesn't matter if it was true or false before.

When binding keys to commands, there is no check. Any gibberish will be accepted and the error will show only after pressing the bound key.

<br>
## String formatting

CMUS offers surprising flexibility in what is displayed in each field in each view (and in program title, which is very important for some applications).

It gives access to many tag variables and lets the user format them in any way he sees fit. Here, the program is as good as they come. You'd struggle to find a music player that gives you more freedom to shape the way tag data is displayed.

<br>
## Tag viewing/editing

And yet, it doesn't have an option to display full tag, even just for mp3 files.

Tag-editing duties are usually split between two programs: tag editor for bulk operations and music player for single corrections. I see a misspelled artist name in player window, I edit this file to make a quick correction in-place.

No need to start another terminal session, find the directory and write full command pointing to a file just to remove one 'd' from the end of an artist string.

You could argue that perhaps the user should leave tag editing duties to his tag editor, but that would be sacrificing convenience for ideology. And it's a poor argument, the file tag is read and displayed by the program already. Whatever tags are read by the player are then potentially scrobbled. It's not a separate case.

But there is no argument for not letting the user view the tag in full. This is really a basic, elemental feature that, just like equaliser, everybody takes for granted in a music player. I doubt many people, when researching a potential change of their music software, checks for the presence of the two, instead taking it for granted.

Notice that there is a a plugin for editing tags, just like there is for playing a random track. How good it is, I don't know. Even if they are great, this functionality needs to be integrated into the base software.

<br>
## File browsing

Ranger offers too much convenience with file management to delegate some of it to CMUS' file browser. For me, it's most convenient to select either single track, multiple tracks, or a folder and then order CMUS to either play or enqueue it (append it to the bottom of current playlist).

People who operate this way will find file browser useless, and in CMUS that browser is a first-class citizen, by default available by pressing '5' on keyboard. Built browser will always have its limitations compared to a piece of software that is dedicated to just this task. Same for people who navigate their hard drives directly in command line, spoiled for endless customization possibilities.

Of course, it's not a problem in any way, as it doesn't take any unnecessary space and the access to it can be unbound. Whoever likes using it has the option to use it. And some internal file pointer had to be developed anyway.

<br>
## Issuing commands from outside CMUS

Here's where things get complicated. Every time CMUS starts, a tool named cmus-remote starts as well. It enables us to control playback, volume control and few other things through a socket, from outside the program. This way, we can, for example, configure custom wireless remote to jump between tracks on a playlist without the need to do it on keyboard.

List of commands is short, and they are stripped in functionality compared to how other players react to particular commands.

For example, the convention is that using the equivalent of 'cmus-remote filename.mp3' (directly giving the program a file to work with) does all these:
- stops current playback
- clears current playlist
- adds the pointed file
- focuses it
- starts playback

In CMUS, this command just adds it to the bottom of whatever place we point to. (library, playlist or queue). It does not automatically switch, nor start playback. Whether switching 5-step convenient shortcut for a simple enqueue as default is a good choice, it is of course a matter of preferences. But the above is default in most applications for a reason. Most people prefer it that way.

Most CMUS commands refuse to offer anything extra. '--next' doesn't switch to next file and begin playback, it just switches to next file. This enables, for example, hopping 5 positions down without hearing the beginning of each track. But it is a very mild inconvenience to which people are used to and comes at a cost in other contexts.

Good design choice was to make '--next' loop playlist. If we are at the last position in the playlist and give that command, it will start from the beginning. It saves us from traversing the entire playlist, or having a separate command to 'go to top'.

I may be scanning an album for a good track I remember from youth. And so I have to issue --next command followed by --play command. Not this track. Another --next, another --play.

Most 'nexts' will most likely be the cases of "don't want this, play me the next one" and here again we're taxed with either double keystrokes for that common use case, or writing a separate shortcut/programming a separate button for it, One way, or the other, not good.

And things get more complicated.

Currently played track remains being one no matter what. If I pause playback and then clear whatever list I'm using for playback, it is somewhat logical that the paused track remains present and ready to continue playback, despite removing it from a list in which it was played. After all, we just paused it, the pointer sits somewhere between its start and end.

![CMUS options view](/img/cmus-music-player-review/cmus3.png)

But when the track is stopped and then the playlist is cleared, I am giving two commands to get rid of it, and yet when I add some new music and order a '--play' command, it doesn't start playing the first track from the freshly added ones. Instead, it starts the last played one from the beginning!

We are past it twice, already have a new batch ready for playback, but the player doesn't want to let go of the past. Hence, instead of 'cmus-remote --clear --play sometrack.mp3', the command to produce behavior surely desired by majority of users has a mandatory '--next'. We have to use next to point to a first object from the list.

And since 'next' does not imply "and start playback", if we want it, we have to ensure it. This litters large part of commands we'll be issuing with '--play', because if entire playlist had finished playing before we added new objects to it, the playback got stopped.

And in CMUS' commands logic, if it was playing, switching tracks also continues playing. If it wasn't, switching will not play. Switching is just switching. It doesn't concern itself with playback status. Sounds good on paper, in practice it is a big annoyance.

Look at this command from DeadBeef:

> --play-pause       Start playback if stopped, toggle pause otherwise

Simple conditional logic and very convenient to use. In CMUS, if you say play, it plays if it isn't already. You say stop, it stops if it's not stopped already. You can toggle play with pause, but not with stop.

Another nice DeadBeef command absent in CMUS, particularly interesting for people who like to gamble, is:

> --random           Random song in playlist

'--seek' is a single exception from the trend here. 'seek +10' will move the playback 10 seconds forward. We can seek backwards, for example with 'seek -20' to go twice that length in the opposite direction. A cherry on top is 'seek 30' (ie. without an operator in front) that enables us to jump to the 30th second of that track. A rare need for sure, but it's always nice that we have this possibility.

Armed with so much flexibility, we can program a remote so that, for example, we have 3 seek options: +3s/-3s, +10s/-10s and +60s/-60s, giving us ingredients for a beefy remote.

<br>
## Bugs

I did not have the opportunity to test CMUS on another machine, but on mine cmus-remote is extremely unreliable. I remotely control behavior of many programs and scripts and no software ever misbehaves, but CMUS does that all the time.

For some time, I thought there is something in the way program acts that I don't understand, and so I tested various command combinations either as a single command, or in chain, but in the end everything failed. There was no reliable way to set cmus-remote for player control.

Ranger's rifle.conf file contains default applications, so the file manager knows what to run when it encounters a file with a particular extension. We can set it up to do some action in CMUS, by executing 'cmus-remote --clear --next --play -- "$1"'. It clears the playlist, adds the track, switches to it and begins playback... or at least is does so some of the time.

Other times it adds the file, but does not switch to it and begins playback of the last-played track.

Because every '--next' has a chance to miss bigger than even more adventurous RPG players might now want to gamble on, we can work around it by issuing the same command 4 times. I did test it 50 times and not once has it failed.

The problem arises when we are adding more than one file. With one, it's not a problem since it either switches to that one file or not and ultimately it does. But with more than one, if each next succeeds, the selection travels further and further! Or at least it would, as even one '--next' can move the cursor down by a random number.

Outside of this behavior, I haven't encountered any other bugs, but this one alone makes the program completely unusable for me. If you decide to install it, right away go to a folder that contains multiple mp3 files, add them to playlist with 'cmus-remote *' and then start ordering 'cmus-remote --next' and see if it switches one position down each time.

Because if it doesn't, well... good luck using the program ;) 

<br>
## Errors

Errors are being displayed at the penultimate line in terminal window, by default in a dark red color (customizable, as colors of all other types of objects). Unfortunately, they don't timeout, but stay there until we do anything inside the player window.

For playlist users, it is an annoyance, since they will be issuing most commands through cmus-remote and not touch terminal window directly.

You can play 10 albums, clear playlist 10 times, pause and unpause 100 times, and that ancient error will still be there for no reason.

<br>
## Equalizer

Again, for some reason, it's missing! For music player, it is an absolute must. My headphones, for example, have a very bad default sound, but when tweaked in an equalizer, they can sound pretty good. Without the ability to play with frequencies, I am forced to use system-wide solutions, which I do not mind, but many people will.

Some of those solutions are obscure, some are hard to wrap head around, some produce artifacts and choke on sound when processor is running some taxing script.

PulseEffects is a very nice system set of tools (assuming someone uses PulseAudio) that contains an equalizer. Combining it with CMUS puts the user in a potentially weird space - a terminal-based music player backed by a graphical interface equalizer. Some choose terminal applications for the sake of them being terminal applications, in which case an extra graphical dependency make the switch a no-go.

For people willing to go this route, there may be a nasty surprise awaiting: PulseEffects fails to start as service on some distributions, requiring always having a graphical interface present to work.

<br>
## Summary

I had high hopes regarding CMUS, to the point of blindly switching to it from a program which, like already mentioned, I was very satisfied with. Unfortunately, its many problems and shortcomings made it impossible to use and after a week I gave up looking for solutions and went back to DeadBeef.

If on your machine cmus-remote commands work as intended and lack of equalizer and tag viewer/editor are irrelevant to you, then you may still want to try it out. Because outside of those issues, everything works like a charm.

Unfortunately, for me each one of those is a dealbreaker, and all together make CMUS unarmed for daily use.
