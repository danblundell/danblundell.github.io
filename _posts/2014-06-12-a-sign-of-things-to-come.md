---
layout: post
title: "A sign of things to come?"
localgov: true
tag:
date: "2014-06-12"
meta: "At the beginning of this week, Individual Electoral Registration went live on GOV.UK and I was quite surprised how underplayed this was in many Local Government circles."
---

At the beginning of this week, [Individual Electoral Registration](https://www.gov.uk/government/collections/individual-electoral-registration) went live on [GOV.UK](https://www.gov.uk) and I was quite surprised how underplayed this was in many Local Government circles.

After seeing the launch promoted on twitter by [Ben Terret](https://twitter.com/benterret) I asked if the register to vote process was informing local registers. I hadn't been involved in the implementation personally and I was curious how it all worked. In true GDS style, Ben got back to me in minutes and brought in [Pete Herlihy](https://twitter.com/yahoo_pete) on the conversation. Pete was part of the team directly repsonsible for the register to vote process and responded with;

<blockquote class="twitter-tweet" data-conversation="none" lang="en"><p><a href="https://twitter.com/benterrett">@benterrett</a> <a href="https://twitter.com/danblundell">@danblundell</a> <a href="https://twitter.com/gdsteam">@gdsteam</a> Hi - yes, this is integrated with all of the local electoral registers.</p>&mdash; Pete Herlihy (@yahoo_pete) <a href="https://twitter.com/yahoo_pete/statuses/476334865198383104">June 10, 2014</a></blockquote>

Wow. This intrigued me more and I continued the conversation with Pete about how it's all working. After a few quick conversations internally and a message from [Phil Rumens](https://twitter.com/philrumens) on the [LocalGov Digital](https://plus.google.com/communities/114124478761452023264) boards, it came to light that the central [GOV.UK](https://www.gov.uk) process validates with the DWP and then calls an api at the relative local authority over the [Public Services Network](https://www.gov.uk/public-services-network) (PSN). Quite a complex set of technologies all working together to produce a seamlessly efficient transaction for the user.

After a bit of thought, it struck me that this is actually more significant that it first appears.

To my knowledge, this is the first end-to-end transactional process that uses PSN. As PSN is used more and more, I suspect we'll see a rise in the creative use of the network as a tool. A key thing about PSN is that it's described as a "logical network based on industry standards" which lends itself quite neatly to a Local GDS type idea. Although a Local GDS is a totally seperate topic altogether, the use of PSN to deliver integrated services across the country is incredibly exciting. As a supporting idea to this, [Ben Cheetham](https://twitter.com/_BforBen) and I have been discussing some of the underlying systems thinking that could support this kind of cross-authority integration. Ben has also written [a great post](https://ben.cheetham.me.uk/blog/2014/06/a-shared-web-platform-for-local-government/) that explains some of our ideas and what we're doing about them.

The register to vote process as part of IER showcases how central policy and process can directly integrated into local authorities, it's not a huge leap to think that this could work the other way around. It stands to reason that if there were other api's at [GOV.UK](https://www.gov.uk) to receive local information it could be 'standardised' into a website all about _your_ local services, no matter where you are. If that were to happen, it's arguable that Local GDS wouldn't even happen. Just think how many local authorities have recently had to update their own elections web pages to explain IER. With a central api at [GOV.UK](https://www.gov.uk), local information could be fed into a local.gov.uk website and the website could adapt it's content based on your location.

I urge you to go and have a read of [Ben's post](https://ben.cheetham.me.uk/blog/2014/06/a-shared-web-platform-for-local-government/) and please come and talk to us at the [LocalGov Digital Makers Hackday](http://localgovdigital.info/news/makers-hack-day/) or [LocalGov Camp](http://localgovdigital.info/news/localgovcamp-2014/).




