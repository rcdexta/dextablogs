---
layout: post
title: "3 ways to keep a Linux process alive"
date: 2012-12-25 10:00
comments: true
categories: [devops, linux]
---

Assume that you have a ruby script `data_cruncher.rb` that will run for a long time. You would like to log into a VM, run the script and keep it running in the background when you log out. Now, I end up writing long running scripts like these all the time and I found the following ways to make them run uninterrupted:

## 1. Job Control

[Job control](http://mywiki.wooledge.org/BashGuide/JobControl) allows you to interact with background jobs, suspend foreground jobs and manage multiple jobs in a single session. Also, when you later realize that the currently running command must be pushed to the background, job control makes it easy to do that.

* Start running the process using the command `ruby data_cruncher.rb`
* Press <kbd>ctrl</kbd>+<kbd>z</kbd> to suspend the process
* type `bg` to resume it in the background

That's it. The job will run in the background. Note that using `bg` command later is the same as running the parent process with & at the end like this: `ruby data_cruncher.rb &`.

<!-- more -->

You can verify active jobs by using the `jobs` command. The `-l` option will include the process ID.

{% codeblock bash.sh %}
$ jobs -l
1]  + 13701 running    ruby data_cruncher.rb
{% endcodeblock %}

Note that the number 1 at the beginning is the jobspec which is a number allocated to each job. 13701 is the process ID. The jobspec is useful when there are multiple active jobs. The significance of jobspec is illustrated below.

$ jobs -l
1]  + 13701 running    ruby data_cruncher.rb
2]  + 14312 running    tail -f data_cruncher.log
$ fg %1
[1]  + 13701 continued  ruby data_cruncher.rb

fg is used to pop out a process to the foreground. fg and bg commands can identify specific jobs with `%<jobspec>`. If you are using bash shell, before you logout, you will have to fire another command `disown -h` to make sure the process is not killed when the terminal session ends. `-h` is used to ignore HUP signal. For more information on HUP, see next section.

{% codeblock bash.sh %}
$ jobs -l
1]  + 13701 running    ruby data_cruncher.rb
2]  + 14312 running    tail -f data_cruncher.log
$ disown %1
$ jobs -l
1]  + 14312 running    tail -f data_cruncher.log
{% endcodeblock %}

What disown actually does is mark the job so that `SIGHUP` signal is not sent to the job when the parent receives it. To explain SIGHUP, let's dive into the next technique.

## 2. nohup to the rescue

To give some context, inter-process communication happens in linux using signals. When a signal is sent, the OS interrupts the normal flow of execution of the process. The most important signals relevant to this blog post are described below. For an exhaustive list, see [here](http://en.wikipedia.org/wiki/Unix_signal)

**SIGINT** (<kbd>CTRL</kbd>+<kbd>C</kbd>) - Sent to a process from controlling terminal when user wishes to interrupt the process

**SIGTSTP** (<kbd>CTRL</kbd>+<kbd>Z</kbd>) - Sent to a process from controlling terminal to request it to stop temporarily

**SIGHUP** - Sent to a process when controlling terminal is closed

When a user logs out of a terminal session, the HUP signal is used to warn all dependent processes of the logout action. nohup command is used to ignore the HUP signal for a specific process.

{% codeblock bash.sh %}
$ nohup ruby data_cruncher.rb &
{% endcodeblock %}

This will cause the script to run in the background and will pay no heed to HUP signal, if received. By default, STDOUT and STDERR will be redirected to nohup.out. To explicitly mention the output log file,

{% codeblock bash.sh %}
$ nohup ruby data_cruncher.rb > data_cruncher.log 2>&1 &
{% endcodeblock %}

What does `2>&1` mean? 1 and 2 are the standard file descriptors for STDOUT and STDERR. We redirect STDERR to STDOUT. Why the ampersand? `1` instead of `&1` would indicate a regular file by the name 1 and not the STDOUT file descriptor.

## 3. Screen your commands

Screen is the most sophisticated way to run background jobs when you want to continously  monitor their activity. According to wikipedia,

> GNU Screen is a software application that can be used to multiplex several virtual consoles, allowing a user to access multiple separate terminal sessions inside a single terminal window or remote terminal session.

What follows is a smash-course of Screen:

* Start screen by typing `screen`. You will be greeted with a welcome message.
* Every program under screen runs in a virtual window. The windows are numbered from 0.
* You are currently in window 0. To start another window, press <kbd>ctrl</kbd>+<kbd>A</kbd>+<kbd>C</kbd>
* Now you have two windows 0 and 1. There are several ways to switch between them. We will use <kbd>ctrl</kbd>+<kbd>A</kbd>+<kbd>"</kbd>
* You should see all the windows that are active. Choose the one you want.

Now, this is where things get interesting. Now press <kbd>ctrl</kbd>+<kbd>A</kbd>+<kbd>D</kbd> You will be detached from current screen session and back to the bash shell. *Note that the session has not been killed and all the programs that you started running with screen will be still active.*

To reattach, type `screen -r` Press <kbd>ctrl</kbd>+<kbd>A</kbd>+<kbd>"</kbd> and you see that nothing has been lost. One obvious perk of screen that job control or nohup does not offer is that you always have easy and immediate access to the process that you were running in the background.

Screen is an awesome utility and this [page](http://kb.iu.edu/data/acuy.html) has more valuable info.
