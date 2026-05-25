---
layout: post
title: "Episode 04 - The Engineering Process (I)"
author: dodger-one
tags:
  - rp2040
  - dmg
  - hardware
---

## The Engineering Process (I)

This episode follows the first serious hand-crafted modifications — both hardware-wise and in terms of the challenge of fitting everything inside the original shell.

At this stage I thought it would be great to have an externally accessible USB-A female connector to reprogram the Pico.  
But the connectors were too bulky, and the cable itself was not flexible enough to fit inside.  
So I remove almost all the external shell from a USB-otg cable (USB-A female to USB-C male):  

<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260319103417.webp" width="400" alt="Episode 04">
</div>

<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260319171459.webp" width="400" alt="Episode 04">
</div>

<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260322101840.webp" width="400" alt="Episode 04">
</div>

It was also time to begin fitting the 2.8-inch screen into the shell. First I took some screenshots of the cabling (completely useless in later stages :rofl:)  
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260319214137.webp" width="400" alt="Episode 04">
</div>

<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260319214145.webp" width="400" alt="Episode 04">
</div>

I also removed the pin headers, since the screen would not fit inside with them attached...
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260319220243.webp" width="400" alt="Episode 04">
</div>

Visual check
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260319221015.webp" width="400" alt="Episode 04">
</div>

DMG-LCD-06 sitting where it belongs.
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260319221026.webp" width="400" alt="Episode 04">
</div>

More fitting and placement tests:
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260322094429.webp" width="400" alt="Episode 04">
</div>
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260322094603.webp" width="400" alt="Episode 04">
</div>
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260322094611.webp" width="400" alt="Episode 04">
</div>

Cabling the screen:
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260322214825.webp" width="400" alt="Episode 04">
</div>
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260322214859.webp" width="400" alt="Episode 04">
</div>

Removed the original ribbon cable from the DMG-LCD-06:
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260323104151.webp" width="400" alt="Episode 04">
</div>
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260323104153.webp" width="400" alt="Episode 04">
</div>

And soldered the new ribbon with only the needed pins attached:
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260323120658.webp" width="400" alt="Episode 04">
</div>
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260323120701.webp" width="400" alt="Episode 04">
</div>

Looks good, right?
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260323121737.webp" width="400" alt="Episode 04">
</div>
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260323121745.webp" width="400" alt="Episode 04">
</div>
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260323122627.webp" width="400" alt="Episode 04">
</div>

Here we go, fixing the USB-A to the DIY board. You'll notice I removed a piece of the board to fit the USB-A :rofl:
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260323122718.webp" width="400" alt="Episode 04">
</div>

MicroSD reader soldered in place:
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260402101414.webp" width="400" alt="Episode 04">
</div>

Test-fitting the internals:
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260402101420.webp" width="400" alt="Episode 04">
</div>
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260402130613.webp" width="400" alt="Episode 04">
</div>

My original dream layout... a true "World of Illusion", because there was absolutely no way this arrangement would fit in the end :rofl:
<div align="center">
  <img src="{{ site.baseurl }}/media/episode04/IMG20260402130624.webp" width="400" alt="Episode 04">
</div>

That's all for now — lots of hand-crafted work, not much technical documentation, but plenty of photos.

I was really enjoying the process during these stages :-)

