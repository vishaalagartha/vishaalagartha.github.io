---
title: "`tmux` notes"
date: 2018-05-20
permalink: /notes/2018/05/20/tmux-notes
--- 

### What is `tmux`? 

tmux is my favorite screen multiplexer. AKA a way of creating, dividing, and navigating window panes in your command line environment. It eases switching between multiple command line interfaces, increasing productivity and code speed!

Note: In this tutorial, I have changed my key bind from `ctrl-b` to `ctrl-a`. You can do this by adding this to your `~/.tmux.conf` file:
``` bash
# remap prefix to Control + a
set -g prefix C-a
# bind 'C-a C-a' to type 'C-a'
bind C-a send-prefix
unbind C-b
```

### Basics
Creating sessions:
``` bash
$ tmux new #creates new session
$ tmux new -s [session name] #creates named session
```
Detaching:
``` bash
C-a d 
```
Attaching:
``` bash
$ tmux a #attach to latest session
$ tmux a -s [session name] #attach to named session
```
Killing:
``` bash
$ tmux kill-session #kill latest session
$ tmux kill-session -t [session name] #kill named session
$ tmux kill-server #kill all sessions
```
Listing:
``` bash
$ tmux ls #list all sessions
```

### Navigation and Splitting
We can split window panes horizontally or vertically.
``` bash
C-a " #split pane horizontally
C-a % #split pane vertically
```

Now, we're going to want to navigate between panes.
``` bash
C-a [arrow key]
```
