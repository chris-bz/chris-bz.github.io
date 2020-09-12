---
layout: post
title:  "Setting up single navigation pane in Ranger and why it's a good idea"
date:   2020-09-01 11:03:28 +0200
categories: programs
---
By default, Ranger starts in 3-pane view, which makes it unique, but not very practical. Fortunately, this can be easily changed.

*If you want to skip past gibberish and straight to solution, click [here](#solution).*

For many years, I was using Total Commander (best file manager on Windows, or used to be one at least). Maximized, I had classical two-pane design. One day, after buying my first full HD monitor, I opened the program and realized how insane amount of space I am losing by having that second pane. Half of my screen space, to be precise.

The truth is, in that classical Norton Commander design, half screen is a glorified directory link. Yes, it can be used to compare folders, to get a broader multi-directory view which can help to make sense of things.

But those are situational. Most of the time, one pane is important, the other just points where to copy, move, from where to copy or move. It is a waste of space.

If reference is indeed its main purpose, the second pane might as well have a vertical split option and just hold stackable directory paths, with pre-selected modifier+randomly chosen key generated for each one to allow operations to/from these.

This would be a very good use of space and a real lifesaver when working on projects with complex directory structures. But one link taking half of screen estate?

This isn't a problem only if someone is used to have maximized applications and alt+tabs between them. Plenty of that space will be wasted in most programs, but that won't matter anyway since we decide to focus one thing at the time. For many people, having multiple things at once makes them bleed attention. That's why distraction-free modes in editors are popular.

I often do multiple things at once and so I'd rather have them next to each other to be able to compare, rewrite code snippets etc.. Often, I have to run different scripts at once and need to see how they perform. And so every bit of space is important.

That's why after switching to Linux few years back, I decided to look for file manager that would just have one pane... end ended up using the one that has three!

In Ranger's default view, parent directory, which is always one keypress away, takes one third of space. Another shows sub-directory. How often is that actually useful?

Some of the most advanced 'navigators' I've met just use command line for their daily routines. They get zero information even about the current directory when navigating, and yet not only they can function, but they are absurdly fast with their operations.

Even when we actually need to scan subfolder for presence of file or two, doing a recursive search will be much faster and much less eye-straining. A view of multiple directories at once is a gimmick, something that is there probably because it makes the program stand out from the crowd.

Since I had absolutely loved Ranger from the get-go, I wasn't going to let this spoil things. Even if there would be no option to change it, I would probably stick to it still. And fortunately, this is not the case.

# Editing config file

Ranger stores configuration in two files: rc.conf (program settings, key bindings) and rifle.conf (file associations). We will be editing the first one.

You can find it in ~/.config/ranger directory. If it's not there, paste this command in console and run it:

> ranger --copy-config=all

Now both files should be there.

First setting command that interests us is:
> set viewmode [mode]

This command takes one of two mode arguments:
- miller: the one I mentioned, showing folder's parent and children folder on both sides
- multipane: multiple unrelated directory views next to each other

Multipane view puts an equation mark between tabs and panes. If there are no tabs, there is one pane. Every new tab creates a new pane, splitting the available space evenly between all displayed ones.

Another command compliments it:

> set column_ratios 1

It takes 0 to 3 arguments. Giving it 4 or more is not considered properly in code. Setting it up in config file and then starting Ranger runs it in a broken state.

For example, feeding it '1,2,1' will set parent and child directory pane to take 50% of total window width, while the main window in which we navigate will occupy the other 50%. Keep in mind though that as of now, it only works for miller mode.

It would be convenient to set it to something like this:

> set column_ratios 5,1

If multipane was set, this command could only accept two argements: first one pointing to relative size of the currently focused tab, the other to all other unfocused ones.

In miller mode, if we only give it one value, it will just give us one column. And since the one in which we traverse directories is the only one we can't live without, it is set to be the only one remaining in such case.

Notice that both setting column_ratios to 0 and not writing any number at the end will also give the same result.

So, if someone does not ever intend to use tabs, he might just set viewmode to multipane and be done with it, but there's no need to do that. Today we don't use it, maybe tomorrow we will ;) Let's set it to miller route and set column_ratios to 1.

That's not all, though. The problem arises when we hover over a file in Ranger. It's gonna get previewed. Now, half the screen will be occupied by a file each time we encounter a previewable one. Sometimes, that's useful, bot mostly it's distracting.

The solution is to disable file previews:

> set preview_files false

<a id="solution" />
## Summary

1. Check if file rc.conf exists in path: ~/.config/ranger/. If not, paste and execute the following command: ranger --copy-config=all. Now the file should be there.
2. Open it in a text editor
3. Add the following lines inside it:
- set viewmode miller
- set column_ratios 1
- set preview_files false

## But why cripple myself with one pane when I can have more?

If removing two panes will leave the user with empty space that he does not intend to populate in any way, it might be better to leave default 3 panes on (unless the extra ones are distracting).

Even for people that don't get much use from them but are used to having them always on, it can feel like the navigation is now crippled.

Don't fret, though. First of all, we have tabs. Switching between them requires single keystroke.

For copying files/directories, popular approach is to:
- change path to the destination directory
- open a new tab
- go to some directory
- select some objects
- yank/copy them
- close tab
- paste

Or reverse it if the file source directory and not destination directory is where we want to do some next thing.

It takes the same amount of keystrokes to complete as doing that operation while jumping between panes. We just jump between tabs instead of panes. We lose visibility of a reference folder entire time, but require half the program space to complete it.

For diff purposes, tabbing is great too. We can, for example, check some potentially modified project folder against backup of the same content and switch between them to quickly see if there are some differences in files. As they take turns in occupying same space, it will be easier to notice differences.

There are better solutions for scenarios like that, but for a quick check, it often is convenient to do it this way.

Second, we can simplify things further. We move always legally downloaded movies to 'movie' folder, legally downloaded mp3 tracks to 'mp3' folder etc. Some folders receive high amount of traffic. Almost everyone has a routine.

Many people who despise file managers and people that use them have bash aliases to folder paths set to allow easy moving, in Ranger it is also very simple to do.

Ranger adopted one of Vim's greatest tools - leader keys. No need to press a modifier key (of which there are few) and control its press and release in combination with another key to trigger some operation bound to that particular shortcut. Almost any key can lead and it doesn't have to be kept pressed.

And so, we can do this:

> map mp shell mv %s /mnt/ssd/books/programming  
> map mn shell mv %s /mnt/files/img/nakedpersons  
> map ma shell mv %s /mnt/files/music/ambient

We select all books we want to move, then press 'mp' and we're done. No need to go to any folder, open any tabs. A simple two-step operation.

If having two directories next to each other is a necessity at some point, nothing stops us from using terminal multiplexer. So long as we remember that two separate Ranger instances can't pass things between them, as Ranger doesn't start a system deamon that could move that information. Another fantastic file manager, [nnn](https://github.com/jarun/nnn), has that functionality.

Same can be done with copying, or any other shell command that takes a list of files and does something to them. Such is an amazing flexibility of command line, and in result, of programs that allow sending some input to it for further processing, like Ranger.

Another fantastic tool that supercharges single-pane navigation is history. In rc.conf, we can set:

> map h history_go -1  
> map H history_go 1

By default, the 'h' key is used for navigation in Ranger, as it is in Vim. I am using Colemak keyboard layout, and so for me this key is free, which allows for a nice mnemonic binding. You can set it to any key you have available.

In a default config Ranger use, coming back in history will mostly go to the directory from which we came, which will be one level higher, or lower in directory structure. Moving forward will do the opposite and will, of course, only work if we already went back in history (like redo in text editors).

But when we have all sorts of 'map [key(s)] cd /some/path' shortcuts and jump between them, those two bindings become way more powerful. And we can bookmark folders on the fly as well ('m' saves current folder to bookmarks, tilde and ' keys open a list of bookmarked folders).
