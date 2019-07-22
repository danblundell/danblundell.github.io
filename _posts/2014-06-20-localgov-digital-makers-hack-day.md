---
layout: post
title: "LocalGov Digital Makers Hack Day"
localgov: true
tag:
date: "2014-06-20"
meta: "What we got up to at the @LGMakers hack day on the 20th June 2014"
---

As a fringe event to [LocalGovCamp](http://localgovdigital.info/localgovcamp/) the [LocalGov Digital Makers](http://localgovdigital.info/localgov-digital-makers/) hack day was set to be a chance for people to "prototype solutions to problems local government currently faces." A great opportunity to see what we could come up with in just 5 hours, whether it be a technical prototype or presentation on the bigger picture to inform and give direction to some existing ideas.

There were three challenges on offer; a combined local government search, a place to share and collaborate on schemas, documents and APIs and a digital inclusion challenge aimed at improving accesiblility to local government services.

I opted for challenge two and quickly got talking to a group in the middle of the room about what we might be able to produce. It seemed a more practical challenge to pull together code-based projects. [Github](http://github.com) already provides a developer friendly platform for sharing code but it doesn't provide some of the data around the edges that would allow us to curate code for a government oriented navigation.

The broader side of this problem is the one where organisations share their customer service strategy or their social media guidelines; how do we curate those in a way that allows for a similar topic and service navigation?  

<blockquote><p>How do we curate things in a way that allows for a government oriented navigation?</p></blockquote>

It seemed that the two sides of the coin could have the same solution if it were done right but it wasn't something we were going to be able to put together in 5 hours. We didn't understand the user needs, we didn't have time to really explore what the wider uses were, we just had a lot of ideas and assumptions. After a short time, the group split to concentrate on the problems in their own right.

### Curating Code

As part of the group pulling together projects from Github, [Paul Mackay](https://twitter.com/pmackay) introduced us to [civic.json](https://github.com/BetaNYC/civic.json), a project designed to attach metadata to civic projects. The principle is simple, you place one extra file in your project and in that file you put some information that describes the project.

The idea is that the [civic.json](https://github.com/BetaNYC/civic.json) file contains the same set of standard data in all projects so that other applications can be built to interpret that data, however, the original civic.json structure isn't yet confirmed and we felt that for the things _we_ wanted to do, it didn't contain the right sort of data. So we [forked the project](https://github.com/danblundell/civic.json) and started adapting it.

While [Paul](https://twitter.com/pmackay) and I got on with refining the data, [Stuart Harrison](https://twitter.com/pezholio) and [Stuart Carruthers](https://twitter.com/codeandstuff) fired up a rails project to build something that could find projects in Github that contained our [new civic.json](https://github.com/danblundell/civic.json) data. This later became the [LG Projects](http://lgprojects.herokuapp.com/) app.

We're still refining the adaptation of civic.json. We're going back and working out what sort of things we'd like to do when curating the projects in terms of filtering, searching and such but changes are all public and if you'd like to contribute or have an idea for its use, [get in touch](http://twitter.com/danblundell)

The [LG Projects](http://lgprojects.herokuapp.com/) app is built to query the Github API, it filters project via the github for government in the UK and currently displays all projects but shows extra information if a project contains a civic.json file. Examples include; [Ratemyplace](http://lgprojects.herokuapp.com/authorities/lichfield-district-council) from [Litchfield District Council](https://www.lichfielddc.gov.uk/) and [Web.Elmah](http://lgprojects.herokuapp.com/authorities/guildfordbc) by [Guildford Borough Council](http://www.guildford.gov.uk/) both of which members of the group had access to and were able to add files to on the day.

We're continuing with this idea and hoping that as we refine the standard a bit more, others will be inspired to tag their project with a civic.json file and we can make some of these great projects more readily available.


### Documenting Documents

The other side of this challenge was about making it easier to curate and find documents. Many organisations publish case studies, guidelines and strategy documents that others could learn from but a majority of these useful things are buried in the organisations own website. Urls are inconsistent and the document is almost always published as a Word Document or PDF because that's what non-developers (regular folk!) use. How do you pull all these documents together so that they can be navigated and searched in a sensible manner? That's exactly what another group led by [Linda O'Halloran](https://twitter.com/LindaSasta) explored. 

There's bound to be more from all these ideas and I'll be sure to try and publish, post and comment on anything I find via [my Twitter account](https://twitter.com/danblundell) so [follow me](https://twitter.com/danblundell) to keep up to date



