requestanimationframe-fix.js
============================

The issue this fixes: You've got a page with many canvases each independently
using requestAnimationFrame (rAF) to handle animation. Most browsers as of March 2015
don't check if the canvas is off the screen. In fact in a stunningly bad API design
the default use case of rAF doesn't give the browser enough information to know
if your canvas is off the screen since by default rAF doesn't require you to tell
it which element you care about.

The signature for rAF is

   requestAnimationFrame(callback, optionalElement)

Ideally everyone would pass in an element but almost no one does. No browser that
I know of actually looks at the second parameter. [Oh, I see, double checking they
stupidly removed the second parameter](http://www.w3.org/TR/animation-timing/) :(

Why is this a problem? Imagine you're making a webpage about physics. Every 2 or 3 paragraphs
you have an animated diagram using rAF. In total you have 15 diagrams as you
go down the page. Because of this poor API design every diagram is always running even
though at any one time only 1 to 3 of them are visible. You find that your diagrams
are running janky at 10fps instead of smooth at 60fps.

Well, this fix implements the second parameter so if passed it will check that
element is on screen. It will also check iframes so if your various examples
happen to be in iframes and those iframes are off the screen you're rAF callback
will get delayed until are on screen.

##Examples:

[Here's an example of 12 iframes, all running using standard rAF](examples/lots-of-iframes.html).
It will likely run slow.

[Here's an example using the fix](examples/lots-of-iframes-polyfill.html). It should run much faster.

##Usage:

include it on every page that uses rAF

    <script src="requestanimationframe-fix.js"></script>

It wraps the built in rAF automatically.

##Caveats:

Unfortunately this is not full solution. It requires every piece of code using rAF to
use this fix. In other words, if you have a page with 10 animated diagrams and each
diagram is in an iframe each of those diagrams have to include this fix. If you control
all the content that might be fine. If on the other hand you're trying to host 3rd party
content or ads you're S.O.L.

On top of that it has to do work for every rAF. If you're in the most common situation
which is a single rAF on a full or close to full page app there's almost no overhead.
If on the other hand you had 100 rAFs there are actually 100 rAF callbacks running
each checking if their element is on the screen. If this was implemented at a browser
level it would be much easier for it to do the minimal work and not have to check 100
elements every frame.

