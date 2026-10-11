---
title: "How I Found a Simple Information Disclosure and an Unlimited Gift Card in the Same Company"
date: 2026-10-11
categories: [Bug Bounty, Web]
tags: [idor, information-disclosure, password-reset, business-logic, gift-card]
description: How I found two different bugs in the same e-commerce company, starting with a password reset link and ending with a gift card that added money instead of taking it away.
---

As you can read in the title, in this blog I will show how I found two different bugs in the same company. Let's start by calling this company `redacted.com`. It's an e-commerce website, and a shout-out goes to this company for their fast response, which doesn't happen often.

As usual, when I'm doing my testing, I separate it into two parts. The first part is where I test functionality while I'm not logged in, and the second part is where I test the areas that are only available once I'm logged in.

## Bug 1: Information Disclosure in Password Reset

I usually start my testing with the password reset. On this company's website, you need to enter a valid email address that is registered on the site so that the system can send you the reset email. The link in that email looks like this:

```
https://www.redacted.com/redacted_catalog/login.php?action=reg&pass=<REDACTED>&id=162786
```

As you can see, the first thing a good hunter thinks of is to change the `id` parameter. When I changed the `id` to arbitrary numbers, I got a `302` redirect to the forgot password page. To my shock, the email field was also populated with the email address of the user associated with the `id` I had entered.

I reported this bug immediately and got a response 24 hours later.

![The company's reply to my first report](/assets/bug1-email.png)
_The company's reply (company name and personal details redacted)_

**Translation of the email:**

> Thank you for your report. The bug you pointed out has been verified and fixed.
>
> We would like to thank you for the report and for the responsible approach with which you contacted us. As a sign of appreciation, we would like to give you a gift card worth €50, which can be used on our shop with no expiry date.
>
> To link it correctly to your account, we need to know the email address you used to register on our site.
>
> We look forward to hearing from you, and we thank you again for your cooperation.

## Bug 2: Unlimited Gift Card

After the company gave me a gift card for reporting the previous bug, I immediately wanted to test whether I could find bugs in the purchasing process. While testing this, I noticed that the website has functionality that lets me gift someone a gift card by entering their email address or username. So I created a new account and began testing.

The request looks like this when gifting a gift card to someone:

```http
POST /redacted_catalog/gv_send.php?action=gvprocess HTTP/2
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

action=gvprocess&to_name=charles&email=charles%40example.com&amount=1&message=
```

I noticed the `amount` parameter immediately and began testing. At first, I entered an amount that my account didn't have, and the response just returned an "insufficient funds" message telling me to check my balance.

Then I entered a negative number, `-1`:

```
action=gvprocess&to_name=charles&email=charles%40example.com&amount=-1&message=
```

To my surprise, the server accepted it, and instead of deducting from my funds, it **added** to them. Basically, you could enter any arbitrary negative number and your balance would just keep increasing.

I reported this and got a response within 24 hours.

![The company's reply to my second report](/assets/bug2-email.png)
_The company's reply (company name and personal details redacted)_

**Translation of the email:**

> Thank you for your notice, the bug has been fixed. I doubled your credit (after normalizing it).
>
> Thank you very much.
