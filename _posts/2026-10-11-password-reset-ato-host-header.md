---
title: "Password Reset Account Takeover by Changing the Host Header"
date: 2026-10-11
categories: [Bug Bounty, Web]
tags: [host-header-injection, account-takeover, password-reset]
description: How changing a single header let me redirect password reset links to my own website and steal reset tokens for any account.
---

![lets go](/assets/host-header-demo.gif)

How I found a simple bug that could let me take over anyone's account just by getting them to click a link.

## Where it started

I was testing the password reset feature of a website. The request looked like this:

```http
POST /forgot-password HTTP/2
Host: www.redacted.com
Cookie: cookie_test=please_accept_for_session; osCsid=<REDACTED>
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://www.redacted.com/redacted_catalog/gv_send.php
Content-Type: application/x-www-form-urlencoded
X-Requested-With: XMLHttpRequest
Origin: https://www.redacted.com

email=
```

## First attempts

I first tested the `email` parameter by putting in an email that didn't end with `.com` or `.it`. The client just rejected it.

I then tried turning the `email` parameter into an array, hoping the server would send the reset request to both addresses. The server rejected that too.

## Testing the Host header

So I moved on to testing the `Host` header. Changing it to an arbitrary value just returned something like "proxy not found" (I'm not sure of the exact status code).

After a while, I discovered that the server was expecting the domain name of the website inside the `Host` header. For example, when I put:

```
Host: mywebsite.com.redacted.com
```

the website accepted it, and the reset email contained a link that looked like this:

```
mywebsite.com.redacted.com/users/recoverypwd/update/C3-PxH73fgmgbVJ
```

The problem was that clicking this link just returned "URL not valid", because `mywebsite.com.redacted.com` isn't a real website I control; it's just a subdomain-looking string.

## Finding a way around it

I needed a way to make the link point to my own site, `mywebsite.com`, while still keeping `redacted.com` somewhere in the `Host` header so the server would accept the request.

I found a neat trick: using a colon (`:`). When you write:

```
Host: mywebsite.com:redacted.com
```

the server treats everything after the colon as a **port number**, rather than as part of the hostname. So from the server's point of view, the host was still valid (`redacted.com`-related), but the actual hostname was just `mywebsite.com`.

I sent the request again, and this time the reset link looked like this:

```
mywebsite.com/users/recoverypwd/update/<token>
```

Now the link pointed straight to a server I control.

## Putting it together

All I had to do from there was:

1. Submit the forgot-password request with a victim's email address in the body.
2. Send the request with the modified `Host` header (`victim-controlled-domain:redacted.com`).
3. Wait for the victim to receive the password reset email and click the link.
4. Since the link now pointed to my own server, the password reset token would land in my server logs.
5. Use that stolen token to reset the victim's password myself.

No interaction was needed from the victim beyond clicking a password reset link that looked completely legitimate, since it came from the real company's email system.
