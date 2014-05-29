---
layout: post
title: meetmicah
title: "My First Javascript Adventure"
description: "Sometimes going the extra mile means gaining an extra headache. Usually worth it though, particularly if you actually accomplish something."
tags: [javascript, jQuery, front-end]
---

I'm not a programmer. I'm a designer and a front-end guy and I love making things look and work beautifully, but I am _not_ a programmer. But even I am well aware that front-end guys like myself are being asked to assume more than your traditional design duties these days. A search for a front-end developer on a job forum will find you with the expected job requirements (html, css) but others are becoming more commonplace, particularly javascript and it's many libraries. And I, the non-programmer, knew I needed to get more familiar with this.

The opportunity came as what I thought was an easy assignment: "Give this text box a character countdown." Well that doesn't appear to be too tough. Pretty sure I can figure out how to get the maxlength that's being defined in the markup and display that underneath the textarea.

"Oh, and we want the countdown to turn red if you hit zero and keep going."

Hm. Well, I suppose I just tell it to add a class when it hits zero.

Yeah, this is a 2-point ticket, tops. I'll have this done in a few hours.

It took three days.

I'll admit, the reason it took much longer is because of this nasty habit I have called "going the extra mile". It's typically a good thing, and can really take you places in life. Like a dimly-lit parking garage surrounded by programmers with bats and chains. But I digress.

The first problem cropped up when I discovered that having a maxlength set in a textarea means you can't go below zero. Once you hit that character limit, you were done, bubba.

Psh, that's easy, we'll just remove it.

```javascript
textbox.removeAttr('maxlength');
```
Oh, wait. We need the maxlength value though, so we can use it in the text. Okay, let's assign it to a variable and THEN we'll remove it.

```javascript
var text_max = textbox.attr('maxlength');
textbox.removeAttr('maxlength');
```
So long, sucka.

With a few more lines, I had the maxlength value being displayed in a div beneath the textarea with the words "characters remaining." I then added more so that when the countdown hit zero, it would turn red. That was easiest for me because CSS is my town, y'all.

The turning point was when I said, "What if we gave you a warning when you were getting close to the maxlength?" Great idea, right? Turns out, when you're developing, saying "what if" is akin to opening Pandora's Box.

Saying "What if" is a rabbit hole that real programmers know all too well. Every "what if" usually brings with it more "what if's" and before you know it, they're multiplying and moving into your basement. Some of the what if's that began to crop up here were:

* What if we want to warn them when they're getting close to the max?
* What if we need to have multiple textarea's with character countdowns?
* What if the maxlength is set to a really low number, like 50? Is the warning really necessary?

As the barrage of "what if's" started to settle, I was tempted to say, "just forget it," and send the ticket to QA at this point, but something held me back. It was a strong desire to learn how to make this little easter egg happen, coupled with the realization that the product team may come back and ask for it eventually, anyway. Why not make it now and give them a nice surprise?

Over the next several hours, I recruited almost everyone in the office to help me solve this problem. Most of them offered sound advice on various different scales. Piece by piece, this little plugin started to come together, and then get massaged and molded to do something slightly more, a tiny bit better.

When I was nearly done (or so I thought I was), one of our other programmers dropped an A-bomb: "What if this changes in the future?"

See, everything up to that point had been based on the assumption that when the character count hit 50, it would turn yellow. With his question, he had brought up what I felt like was a shocking revelation. What if they wanted that number to be different down the road? We might use this more later on; what if they wanted it to be different every time? What if they didn't want it at all?

The answer - much as I hated it at the time - was to essentially, make it an optional thing. I _hated_ that. I'd spent two or three days on this neat li'l feature that was certainly going to wow everyone. But nobody would be wowed unless they saw it. Which they would not, by default. Dang.

But as they explained to me the possible scenarios of the future, I began to realize it was the right decision. We needed to make this something that _could_ be implemented, but didn't _have_ to be there. We'd already met the requirements of the ticket - my little yellow bit was just icing on the cake. And some people don't like icing. Or cake.

I'm sitting now waiting on the final version of this to be merged into dev, where it will eventually make it's way to the live site. It's a little dejecting to know that it may never see the light of day (though I'm quite certain it will), but I have to remember the fact that I did learn a lot, and I got to see a glimpse of what real programmers have to deal with a lot. It was a fun challenge and I'm sure it won't be my last. At some point, I'm sure we'll get to have our cake and eat it, too. Or something.