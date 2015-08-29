---
layout: post
title:  "Android and Captive Portals"
date:   2015-08-29 08:04:43
categories: troubleshooting
---

Our family stayed at a hotel that offered free AT&T WiFi with an access code 
was distributed at the lobby. 

"Just connect to the network `attwifi` and enter the code," the brochure said.

The network had a (captive portal)[https://en.wikipedia.org/wiki/Captive_portal],
a page you had to open to enter your code before your device would be allowed
to use the Internet. 

Ordinarily, my Android phone (running Lollipop) detects this and shows a
notification asking me to sign into the network. This time, though, it wasn't
working. On most captive portals, attempting to load *any* URL causes a
redirect to the captive portal, but on this particular network, the HTTP
requests were just failing. 

After a bit of searching I unearthed two helpful bits of information:

- People mentioned that their Apple devices "just work" when connecting, but
  have had problems with Android.

- Apple devices [attempt to load a well-known URL](http://blog.erratasec.com/2010/09/apples-secret-wispr-request.html) 
  on Apple's web site to detect a captive portal: 
  [http://www.apple.com/library/test/success.html](http://www.apple.com/library/test/success.html).

The solution? I brought up Chrome on the Android phone and manually entered the
Apple URL. This time the portal detected the request, redirected me to the
log-in page, and all went swimmingly.

So, the moral of the story: if your Android device won't detect captive
portals, it may be that the problem is that the captive portal isn't
redirecting all URL requests--only those that seem to be sniffing for a portal.
Try Apple's URL. 
