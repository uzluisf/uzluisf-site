---
title: "Syncing Your Tasks with fruux and Tasks"
author: "Luis Uceta"
date: "2019-03-30"
tags: ['tasks', 'fruxx', CalDav]
draft: false
---


Recently I came across [Tasks](https://tasks.org/), an Android app that lets you
manage tasks (*ta-da!*). While I've been using the app to write down simple
tasks, I've always wished that the tasks could be synced. Thus I was quite
surprised when I learned that the app is compatible with CalDAV servers and
thus allows you to sync your tasks with any service that uses CalDav, an open
standard that provides client access to schedule information on a remote
server. Although CalDav is mainly used to synchronize calendars, the protocol
also includes a specification for to-do lists entries.

For its part, Tasks allows you to synchronize your to-do lists entries with a
variety of proprietary or open source calendar servers. Among these third-party
services is [fruux](https://fruux.com/) which provides synchronization of your
contacts, calendars and tasks across devices. As of now, I'm using fruux's 
Basic plan, which is free and offers unlimited contacts, calendars and tasks
and synchronization with up to 2 devices. fruux has its own Android app, which
albeit simple, I've found it to be quite cumbersome which is the reason I decided 
to use Tasks instead. The process to set up a fruux account with Tasks is
quite simple. This is how I did it:

1) [Sign in](https://fruux.com/account/login/) to your fruux account. Now
   you must go to the [Sync](https://fruux.com/sync/) section and add a new
   device. By default there's a `no name` device that I think it's created when
   you first use any of the fruux apps. However, this device might not be useful
   to sync anything since it has no custom password.

2) After adding the device, a username and a password will be automatically
   generated which we'll use to add our account in Tasks. It's worth mentioning
   that the password is soon forgotten by fruux servers so keep it at hand until
   you've set up the account.

3) In the Tasks app, go to `Settings » Synchronization » Add account`. Once
   there you must enter a name, a url and the username-password pair which was
   generated for us when a new device was added to the fruux account. The url 
   part was quite tricky for me since I thought entering `https://fruux.com/`
   would do the job. Instead you must use `https://dav.fruux.com` as the base 
   url.

4) If you entered everything currently, your fruux account's tasks lists
   should now be showing in the Task app.
 
Should you need additional information, the following links might be of help:

* https://tasks.org/docs/caldav_intro.html

* http://support.fruux.com/faq/how-to-set-up-sync-with-my-device-or-app

* https://www.davx5.com/tested-with/fruux

*Alright... see ya!*
