---
layout: post
title: "Reusing GDS Assets"
localgov: true
tag:
date: "2014-09-30"
meta: "Reusing assets from GDS in local government"
image: ""
---

This week we've been working on ways to improve the data we capture about users of our online services. We have reports coming out of our ears from phone systems, CRM systems, the CMS and Google analytics but one thing we haven't been measuring until recently is how users flow through individual transactions online.

A few months back I attended a [Local Digital Campaign](http://www.localdirect.gov.uk/about-us/local-digital-campaign/) day about the [GOV.UK Performance Platform](http://gov.uk/performance) and the exciting things that are being put together to better understand the performance of services. On the day we talked with Matt Harrington about the data being collected for dashboards like [Practical driving test bookings](https://www.gov.uk/performance/book-practical-driving-test) and [Carers Allowance applications](https://www.gov.uk/performance/carers-allowance). Matt gave us a great insight into how data was being collected but also the value behind using the dashboards to ask better questions about how services are delivered.

As a result of the day 

We've included a tiny part of the GOV.UK performance platform setup, [stageprompt.js](https://github.com/alphagov/stageprompt). Stageprompt abstracts your analytics platform from your website. In our case, it allows us to include triggers within our web pages to track user journeys at each stage of a transaction and push that information into Google analytics. You can do this quite easily purely using Google's analytics api, however, the advantage of something like stageprompt is that if Google decide (as they often do) to change their analytics api, we only have to update stageprompt, or even hope that GDS have kept their copy up to date and download the latest version.

<figure class="pushed">
	<img src="/img/content/reusing-gds-assets-1.png" alt="Screenshot of the mapping application">
</figure>

The idea is simple and we have the new event tracking running in our test system. We're already seeing the potential that tracking user flow gives us within individual pages and stageprompt.js has meant that integration of analytics events into forms and pages is seamless with very little overhead.

We can reuse this idea across all our transactions and getting into more depth with how services are working for users, we can start to better improve them as time goes on. The real bonus is, we can choose what we track. Want to know how many people fail to enter a post code correctly? We can track that. Want to know how many people request a receipt via email? We can track that. Any 'event' that a user causes or encounters can be pushed into an analytics platform so that it can support continuous improvement. 

This is only one of the things we've reused thanks to GDS and although it may only be a small addition, it's incredibly valuable.