---
layout: post
title: "Switchboard, what are you?"
description: "Building some server-side IMAP tools."
tags: [switchboard, imap, jmap, spatch]
mixpanel: switchboard_what
---

## Intro

Several months ago I was lucky enough to be connected with Spatch as
they were getting ready to join the TechStars London spring '14
class. You can check them out over [here](http://spatch.co/), but real
quick: they're building the next-generation email client by embedding
structured data in emails as they're sent. [They're building an incredible
project](http://blog.spatch.co/).

I'd had some experience working on the backend of an email client
application with Runway 20, builders of
[weld](http://www.tryweld.com/). As it turned out, one of the features
I'd built was useful to Spatch: sending out push notifications to the
iOS client as new emails arrive.

## Remote vs Local Push Notifications

First off, why send push remote push notifications? With most
approaches, the client has to manage an IMAP connection already, e.g.
Spatch uses [`mailcore2`](https://github.com/MailCore/mailcore2). In
addition, iOS7 introduced [Background
Fetch](http://www.objc.io/issue-5/multitasking.html). The app could
connect to the IMAP server, check for new emails, and send local push
notifications as necessary. However,

- Your app still can't maintain a persistent network connection
  outside of it's 30s backgroudned execution window, limiting it to
  using polling methods.
- You aren't in complete control of when your app enters the
  background state. `minimumBackgroundFetchInterval` only does what it
  says on the tin, only specifying a lower bound for the background
  fetch intervals.

This does a fine job of protecting you from overly aggressive
applications, but it also prevents apps from being able to monitor
for real-time updates client-side. That's fine, that's why remote
push notifications were introduced.


## Form meets Function: A worker system

The mile high view of the serverside push notifier is simple: an
application monitors for new emails for an account, and sends push
notifications to the account owner's devices as they arrive.  Zooming
in on the first half, we open one IMAP connection per mailbox we want
to monitor.  These processes auth, select the mailbox, and use the
`IDLE` command if present to be notified of emails as they
arrive. Typically only `INBOX`, or the mailbox with the `\INBOX` flag,
will be monitored. An additional authorized IMAP connection is opened,
and used by the rest of the application to make requests. As emails
arrive, the application is notified by `IDLE` messages. All of this is
greatly muddied by varying implementation of IMAP specs and
extensions.

In the second half, these messages are used to fetch the data
necessary to create a push notification, form a push notification, and
send it down to the client.

My not-so-subtle halving of the task is because while that first-half
is wonderfully generic, the second half can be swapped in with
whatever processing you'd like. It's a perfect fit for a worker
system. I have this set up for my own email accounts running some
goofy tallying and NLP scripts. Email hacking!


## Worker Interface

I decided to put together a test worker interface, and made some quick
decisions under the assumption that I'd end up replacing this part
soon. I went with with workers connecting to the server over some
network protocol. As new emails arrive, messages are sent from the
server to notify the worker. The worker can make requests of the
server, for example a fetch. I went tickled an itch of mine and went
with websockets. It helped that my webserver,
[Cowboy](https://github.com/extend/cowboy), mixes great with
websockets.

In the continued interest of just building something, I went
with JSON as the data structure. I reimpemented enough IMAP
commands in JSON over websockets to allow for the push notifier
to be built as a worker. However, my 1:1 implementation provided
no real benefit to just passing raw IMAP commands, and came
at the cost of additional code.

A late and surprising google search of `imap websocket json` led me to
[JMAP](`http://jmap.io/`). "JMAP is a new JSON-based API for
synchronising a mail client with a mail server." "The JSON may be
exchanged between client and server over either HTTP or the WebSocket
protocol." JMAP, written by Neil Jenkins of
[FastMail](http://fastmail.fm), 

Given that the application

- manage IMAP connections multiple users across multiple providers
- publish notifications of new emails to subscribed workers


At this point, JMAP seems to me to be a sane protocol
for accessing my email.
