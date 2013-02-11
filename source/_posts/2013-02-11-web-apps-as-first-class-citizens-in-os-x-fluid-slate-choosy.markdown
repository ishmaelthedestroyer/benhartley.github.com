---
layout: post
title: "Web apps as first-class citizens in OS X: Fluid + Slate + Choosy"
date: 2013-02-11 06:47
comments: true
categories: [productivity, os x]
---

Promote the web apps you regularly use to first-class citizens in OS X using 3 simple utilities.<!-- more -->

## Fluid
[Fluid](http://fluidapp.com/) is a free app that lets you create a "native" version of a webapp - it's essentially a stand-alone version of webkit for each web app which can then have its own Dock icon etc. If you aren't already using it, I'll leave you to experiment with it here.

## Slate
[Slate](https://github.com/jigish/slate) is an insanely fully-featured window manager, and I encourage you to dive into it since it has way more features than anyone could ever need. The functionality we're interested in for this post is assigning hotkeys to apps that are running, so your Fluid apps are only ever a couple of keystrokes away. No more alt-tabbing between them! Here's a snippet from my `~/.slate`:

``` 
alias hotkey ctrl;cmd
bind b:${hotkey} focus 'Google Chrome'
bind i:${hotkey} focus 'iTerm'
bind m:${hotkey} focus 'Mail'
bind c:${hotkey} focus 'gCal'
bind f:${hotkey} focus 'Finder'
bind r:${hotkey} focus 'RTM'
bind t:${hotkey} focus 'iTunes'
bind d:${hotkey} focus 'Google Drive'
bind w:${hotkey} focus 'Workflowy'
```

*You can have a look through the rest of the file in my [.dotfiles repo](https://github.com/benhartley/dotfiles)*

These config options specify a hotkey combination of `cmd` + `ctrl` and then assign different keys to different apps, e.g. `cmd` + `crtl` + `c` switches to Google Calendar.

## Choosy
The final piece in this puzzle is [Choosy](http://www.choosyosx.com/) - self-described as "a smarter default browser for OS X". Choosy acts as a router for any links you click, anywhere in the O. Instead of them opening automatically in your default browser, Choosy lets you set rules, and open different "browsers" (or Fluid apps) depending on where the link points to. So if someone emails me a link to a story in Pivotal and I click that link, Choosy matches the URL to my Pivotal Fluid app and launches that instead of Chrome.


