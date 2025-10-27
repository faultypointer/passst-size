+++
title = "Whats Going On"
date = 2025-07-15
[taxonomies]
tags = ["rust", "go", "rant", "life"]
+++

It's been more than three months since my [last post](https://faulty.carboxi.de/goingallin/) where I waffeled about
focusing on rust. I can't say I have kept my word nor can I say that I went back on what I said. Instead of explaining
what I mean, I though maybe I should write about all the things I did during the last three months. I am also curious to
know how those three months went by cause it's all hazy up here.

Instead of writing about the things I did in chronological order, I though its better to write about them topic wise. _Since I can't
remember exactly the order of events_

Let's start with

## Debian
I switched. To arch BTW.
I'm still on arch. I did a quick jump to debian trixie and back because of major project (more on that later).
I was using Gnome in debian and stayed with Gnome after switching to arch. That changed after that little hop to debian
and back. I am using [niri](https://github.com/YaLTeR/niri) window manager now.

### Gnome, Niri and Me
A little history about my experience with niri. I was introduced to niri by [scientiac](https://flux.carboxi.de/). Niri
is a scrollable window manager.
I used niri on nixos before I moved to debian. Since niri wasn't (and still isn't I think) available as a package on debian, I
tried to build it myself. It depended (maybe not directly) on [gtk4-layer-shell](https://github.com/wmww/gtk4-layer-shell) which
was also not available on debian. Again, I tried to build it myself. I couldn't. I don't remember the exact problem but it was
something about wayland protocols. At the time, freedesktop.org was mitrating its server or something so it was not working. So
I gave up on building niri and stayed with gnome.

After using gnome for a while, I began to like it and niri was completely out of my mind. I was happy with gnome and debian until I
wasn't. This was more of a problem with debian than gnome. So I switched to arch _btw_. I was so in love with gnome that I stayed with
gnome. I had totally forgotten about niri.

It was maybe two weeks ago that I decided to switch to debian trixie due to something related to major project. One thing lead to another
and I decided to build [ghostty](ghostty.org). I ran into some problem and after looking for some time, it said gtk4-layer-shell missing.
I decided to give it another try and build gtk4-layer-shell and to my surprize it build and so did ghostty after that. This got me thinking
that maybe I can build niri too and I could. I switched back to arch the next day because the major project problem was still a problem in
debian too.

I had forgotten all about niri. I know I had. But just building it on debian (I couldn't even use it there) brought back so many memory that
I had to go back and I did. I spent the next day configuring niri and am using it as a daily driver. I also have gnome but to be honest I don't
think I'll go back anytime soon.


## Emacs
I am still using emacs. I have helix (for when I have to do something quickly from terminal) and zed (for looking through linux and redox source code)
but emacs is still the editor of choice (and probably will be for a long time) when it comes to development. I could have included
this in the previous topic if not for ...

### The Organ
The emacs [org-mode](https://orgmode.org/) is an excelent tool for keeping personal notes, literate programming, planning projects among other things.
I had a pretty neat system in place for organizing notes, journals, todos and everything else. I called it *Organ*. _kinda cringe now that I think about it_
It was going great. I kept track of things I needed to do each day. I kept the links to articles and videos I found interesting and tried to read or watch one
each day. I kept track of the things I was learning. I don't know whether it happened gradually or spontaneously, but I stopped.

I had recently taken up [Logseq](https://logseq.com/) for note taking but it has also been dry for past few days and I have to intention or motivation to continue it.


## Rust
I don't know where to begin with this. The books that I mentioned in the previous blogs, I have read two out of four which isn't a lot. Actually thats kinda low given the
three months of time frame since I also haven't done any project in that time. I do have these to show for:
- Prime number generation using different techniques in rust. [repo](https://github.com/faultypointer/largestprime)
- One Billion Rows Challenge in rust. [repo](https://github.com/faultypointer/OneBRC)
I learnt some things about performance and performance testing in rust.

I books that I did complete ([Learn Rust Entirely with Too Many LinkedList](https://rust-unofficial.github.io/too-many-lists/index.html), [Rust Atomics and Locks](https://marabos.nl/atomics/))
taught me quite a lot about unsafe rust, atomics and memory ordering, rust types thread safe programming etc.

### Go
I have also been learn go for a week or two. I wanted to learn backend from the ground up and thought go would be perfect for that. Another reason for learning a language other than rust
is to be able to find a job after engineering finishes. _9 months to go_ I probably should have gone with something like java or c++ or python but I like go and after understanding the
the ins and outs of backend development, it shouldn't be that hard to learn something like Django, or Laravel or dotnet.

I am currently reading "The Go Programming Language" book. I'm halfway through it even though its almost been two weeks (or maybe more). But thats just how I
decided to go with this. Rust is still the main focus and with college and major project, I don't really have time to "Go all in" on go. Thus goal for this year
on go is to finish this book and follow through two other books called "Lets Go" and "Lets Go Further". They are tutorial books on web development which is perfect
for this because major project, college etc.
 
## Major Project
I'm in my final year of bachelors and we have to do a project this year. With AI becoming more and more relevant these days, we _project is done in a group_, like everyone else, decided
to do something in the field of AI is what I would have said if we were not complete idiots. Instead we decided to implement OverlayFS for RedoxOS. You can read the proposal [here](https://gitlab.com/carboxide/proposal).

Remember how I said that I haven't done any project in rust. Well, a (pseudo) file fystem implementation for a modern micro-kernel based operating system seems like a good first project. _also incredebly hard but thats beside the point_

I don't know what else to say. The mid-term defence for the project is in less than a month and we still have little clue on where to start. The good thing is that the build system for redox os
seems to be working fine now.

## Honourable Mentions
I think I've spent significant amount of time on this post so I'm just going to go through with the rest of the topics fast.
### Haskell
I tried to learn haskell. Why? Function programming and I first tried to learn erlang then switched to haskell for asthetics. Had to leave it when the title defence for major project came about and haven't
picked it up since.

## Where will it go?
Now that I have talked about what I have been doing, it only seems fair to talk about what I plan on doing. As for rust and go, the way they are going feels fine. I am doing major project in
rust so the learn won't stop and the three books for go is fine. Maybe if I finish early with those books, I will so a project but that depends on the status of major project at that time and
studies too.

### An Organized Workflow
I through about ranting anout this here but as I said I have spent enough time for this post so maybe I'll write about it in a future post. The gist of it is that I can't seem to find a consistent
way to organize notes, todos and all other stuff. Nothing has worked be it obsidian, org-mode, logseq, neovim and markdown. I want to have a workflow that works. 


And that is it. I have said all that I wanted to say. _maybe not but there are future posts for that_

Bye.


.

.

.


<br>







Did I mention that I use Arch BTW.
