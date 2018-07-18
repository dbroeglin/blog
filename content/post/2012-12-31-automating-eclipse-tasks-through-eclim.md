---
title: "Automating Eclipse tasks through Eclim"
date: 2012-12-31
categories:
  - QuickTips
---
Lately my day to day work included developing applications with the [Play! framework](http://playframework.org). Play!,
in both its 1.x and 2.x versions, is a great framework but its integration with Eclipse is not always easy.
One of the most frequent irritant I had, was having to use of dreaded _play eclipsify_ command which helpfully states:

{{< highlight none >}}
$ play eclipsify
~        _            _ 
~  _ __ | | __ _ _  _| |
~ | '_ \| |/ _' | || |_|
~ |  __/|_|\____|\__ (_)
~ |_|            |__/   
~
~ play! 1.2.5, http://www.playframework.org
~
~ OK, the application is ready for eclipse
~ Use File/Import/General/Existing project to import /Users/dom/Sources/play-ground into eclipse
~
~ Use eclipsify again when you want to update eclipse configuration files.
~ However, it's often better to delete and re-import the project into your workspace since eclipse keeps dirty caches...
~
{{< / highlight >}}

The key phrase being «...it's often better to delete and re-import the project
into your workspace since eclipse keeps dirty caches...».  Unfortunately, it is
better to heed this piece of advise. However, if you work with several external
libraries whose version numbers change on a frequent basis, it can quickly
become quite annoying to do all this mouse work. Especially, if you trained
yourself for years to do most of your tasks by using the keyboard or some sort
of automation.

This is where [Eclim](http://eclim.org) can help! Eclim is an integration layer between Eclipse and VIM.
You can both use vim inside Eclipse or use some cherry-picked features of Eclipse from VIM. I'm not yet
at the point where I would use VIM to develop Java applications, if ever, and I'm not suggesting you to
switch from Eclipse to VIM either. However, the API used by the VIM plugin to communicate with it's
Eclipse counterpart is a handy entry point into Eclipse automation.

After some digging around I found out that Eclim comes with an _eclim_ command which, in turn, accepts commands 
to be sent to Eclipse running an _eclimd_ server. I will not enter  here into the details of how Eclim
works but it was quite easy to put together this simple script:

{{< highlight bash >}}
play eclipsify && \
  ${ECLIPSE_HOME}/eclim -command project_delete -p "$(basename $(pwd))" && \
  ${ECLIPSE_HOME}/eclim -command project_import -f "$(pwd)"
{{< / highlight >}}

The script basically executes _eclipsify_ then deletes the project whose name is the name of the
current directory and then reimports the project from the current directory. The `ECLIPSE_HOME`
variable should contain the location of your Eclipse enhanced with Eclim.

Installing _Eclim_ is a breeze if you [follow those steps](http://eclim.org/install.html).

Have fun automating your Eclipse work!
