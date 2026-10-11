---
title: "IDOR Vulnerability Let Me Access 100k+ Accounts' Information"
date: 2026-10-11
categories: [Bug Bounty, Web]
tags: [idor, information-disclosure, authorization]
description: How changing a single numeric parameter let me access the account information of over 100,000 users across multiple country-specific websites.
---

How changing a parameter allowed me to access accounts' information.

As you can see by the title, I will show you how I found a simple IDOR vulnerability that let me access accounts I'm not authorized to access.

## The target

This company has a different website for each country, but all of them use the same underlying technology. For example:

```
redactedmusic.com.ar
redactedmusicusa.com
redactedmusicbrasil.com
redactedmusicchile.com
redactedmusicperu.com
redactedmusiccolombia.com.co
redactedmusicecuador.com
```

Because every country's site shares the same codebase, a bug found on one of them is very likely to exist on all of them.

## Why I tested for this

I was testing for an IDOR because I was curious whether simply changing a numeric parameter would let me access information I'm not supposed to see. With AI tools everywhere nowadays, I honestly find it hard to believe that this kind of bug still exists.

And to my surprise, it still does.

![lets go](/assets/cant-belive.gif)

## Finding it

When testing for IDOR, a user's own profile page is usually a good place to start, so that's what I did. I went to my user profile, and the request looked like this:

```
GET /listas_precio/area_publico/contenido_form_alta_publico.php?abm=11385&clienteid_publico=XA4402
```

I immediately began testing both parameters and found that I only needed to change `abm`, since the `clienteid_publico` parameter didn't actually do anything.

So, as you can probably guess by now, simply changing the `abm` parameter gave me access to any user's information associated with that ID, without the server ever verifying that I was authorized to access it.

![lets go](/assets/not-good.gif)

I reported this bug immediately.
