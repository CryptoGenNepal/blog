---
title: "XSS on Samy Pl"
date: 2019-06-19T00:00:00+05:45
# post image
image: "images/blog/XSS_on_samy_pl/7th.png"
# author
author: "Nirmal Dahal"
# post type (regular/featured)
type: "regular"
# meta description
description: "I was explaining the work of Samy Kamkar to one of my friends. Samys site has so many easter-egg like challenges. We are not allowed to view the source code using normal methods like ‘right-click’ , ‘Inspect element’ , ‘Ctrl + shift + I’ etc. Then I tried the following snippet."
# post draft
draft: false
---


In this article, I am going to explain a security issue that I found on a web site which is famous within the information security researchers. Samy Kamkar is an American privacy and security researcher, computer hacker, entrepreneur and for me a very big influencer. Samy Kamkar is the person who created the first JavaScript-based worm known as Samy Worm which went viral within a few hours ultimately compelling myspace to shut down temporarily.

I was explaining the work of [Samy Kamkar](https://samy.pl/) to one of my friends. Samys site has so many easter-egg like challenges. We are not allowed to view the source code using normal methods like ‘right-click’ , ‘Inspect element’ , ‘Ctrl + shift + I’ etc. Then I tried the following snippet.

```html
  javascript:https://samy.pl/?%0aalert(document.body.innerText=document.body.innerHTML)
```

I was able to see the source code that was extracted from the DOM for the Samy’s website.

You guys can go to [https://samy.pl](https://samy.pl) and try to right-click, view his source code using ctrl + u or even go to another tab and type:

```html
  view-source:https://samy.pl
```

Lastly, try to view the source using inspect element and I think you will be surprised. Do comment if you find any other way to view the source code.

Then I started going through everything that was on the website. I saw that *he was using JS code to detect proxies like Burp, Charles & Fiddler*. I dug deeper to see everything he was using to make the awesome website that he has. Sometime later I found a suspicious “jsonp endpoint” that was used to reflect the public IP address of whoever is requesting his website.

{{< image src="/images/blog/XSS_on_samy_pl/1st.png" caption="Verifying proxy tools" alt="Verifying proxy tools" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Verifying proxy tools" >}}

{{< image src="/images/blog/XSS_on_samy_pl/2nd.png" caption="This code was verifying if the user was logged in to disqus or not" alt="This code was verifying if the user was logged in to disqus or not" height="" width="" position="center" command="fit" option="" class="img-fluid" title="This code was verifying if the user was logged in to disqus or not" >}}

{{< image src="/images/blog/XSS_on_samy_pl/3rd.png" caption="Vulnerable EndPoint" alt="Vulnerable EndPoint" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Vulnerable EndPoint" >}}

The endpoint was something like [https://samy.pl/ipp.php?jsonp=st](https://samy.pl/ipp.php?jsonp=st) then it would reflect something like this

{{< image src="/images/blog/XSS_on_samy_pl/4th.png" caption="Vulnerable EndPoint 1.0" alt="Vulnerable EndPoint 1.0" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Vulnerable EndPoint 1.0" >}}

After finding the endpoint I started altering the endpoint. First I removed ‘st’ from the endpoint and I saw that it was also removed on the reflected output.

{{< image src="/images/blog/XSS_on_samy_pl/5th.png" caption="Vulnerable EndPoint 1.1" alt="Vulnerable EndPoint 1.1" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Vulnerable EndPoint 1.1" >}}

I then altered furthermore by adding ‘<>’ which was successfully reflected on the site.

{{< image src="/images/blog/XSS_on_samy_pl/6th.png" caption="Vulnerable EndPoint 1.2" alt="Vulnerable EndPoint 1.2" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Vulnerable EndPoint 1.2" >}}

When I saw that the greater than and smaller than symbols were reflected I straight jumped into using my custom script to reflect a cross-site scripting vulnerability that was successfully executed and that was how I was able to perform XSS on Samy.pl.

{{< image src="/images/blog/XSS_on_samy_pl/7th.png" caption="R-XSS DOM Overwrite" alt="R-XSS DOM Overwrite" height="" width="" position="center" command="fit" option="" class="img-fluid" title="R-XSS DOM Overwrite" >}}

{{< image src="/images/blog/XSS_on_samy_pl/8th.png" caption="R-XSS DOM Overwrite 1.0" alt="R-XSS DOM Overwrite 1.0" height="" width="" position="center" command="fit" option="" class="img-fluid" title="R-XSS DOM Overwrite 1.0" >}}

{{< image src="/images/blog/XSS_on_samy_pl/9th.png" caption="The Popup" alt="The Popup" height="" width="" position="center" command="fit" option="" class="img-fluid" title="The Popup" >}}

## **Video PoC:**

{{< youtube AlG0-x5QG3A >}}

## **Timeline**
  - June 6, 2019 — Report Sent
  - June 7, 2019 — Fixed & allowed to publish by Samy Kamkar

## **२ शब्द:**

I have found in my past experience that even some top security researchers from *HackerOne* have had their site hacked or other security researchers have discovered some type of vulnerability. No matter how much people claim to have their site/system it is 100% secure or no matter how much a person is experienced it is only a matter of time and the amount of knowledge to discover a new or even existing flaw.

***Note**: I did not think I would find an XSS issue in Samy Kamkar's site. The person who created the first-ever javascript-based worm. I have written this blog with the full permission of [Samy Kamkar](https://twitter.com/samykamkar).*

{{< image src="/images/blog/XSS_on_samy_pl/10th.png" caption="Thanks Received from Samy Kamkar" alt="Thanks Received from Samy Kamkar" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Thanks Received from Samy Kamkar" >}}


## **धन्यवाद!!!**