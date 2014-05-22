---
layout: post
title: "Switchboard, what are you?"
description: "The path to building an IMAP -> JMAP client."
tags: [switchboard, imap, jmap, spatch]
mixpanel: switchboard_what
---

Several months ago I was connected with Spatch as they were
getting ready to head to TechStars London. You can check
them out over [here](http://spatch.co/), but real quick:
they're building the next-generation email client by
embedding structured data in emails as they're sent. [They're
pretty great](http://blog.spatch.co/).

Why do we need server-side code at all?

I'd built an application to send iOS push notifications as emails
arrive with my last job at Runway 20, builders of
[weld](http://www.tryweld.com/). As an email client, Spatch needed the
same functionality. I began it as a greenfield project with the
goals of

- manage IMAP connections multiple users across multiple providers
- publish notifications of new emails to subscribed workers


At this point, JMAP seems to me to be a sane protocol
for accessing my email.
