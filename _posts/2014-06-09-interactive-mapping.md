---
layout: post
title: "Interactive Mapping"
localgov: true
tag:
date: "2014-06-09"
meta: "Open source interactive mapping project for local government using OS Open Space"
image: "interactive-mapping-1.png"
---

Recently I was set the challenge of replacing an interactive mapping application that we ran for Northampton Borough Council. The product I was to replace had an annual support fee and upgrade cost attached to it and we were at a point where an alternative was needed.

With no budget and a limited brief, prototyping something quickly and iterating seemed like a good plan; I've found it's been easier for the product owner to critque and comment on something that they can play with and point at.

Knowing the basic needs was pretty key;

* see a map of the town that can be dragged around and zoomed
* be able to view layers of information over the map
* be able to turn the layers of information on and off
* be able to select a point on the map and see information about that location

On those four basic needs I began pulling together a quick page that did all of the above. I started with [OpenLayers](http://www.openlayers.org) as it's a reasonably lightweight JavaScript framework with built in functions for connecting to all kinds of mapping services. To serve the mapping data, I fired up an instance of [Geoserver](http://www.geoserver.org) as it was a well documented, well used and supported open source mapping server. As an added bonus Geoserver actually uses OpenLayers as a preview library so there's quite a bit of documentation on how to get Geoserver and OpenLayers working together.

Once I'd got a basic map going, we started to revisit some of the functionality available on neighbouring [Daventry District Council's mapping app](http://mapping.daventrydc.gov.uk/). Although there's some great functionality on Daventry's site, we weren't so keen on the loading progress bar or the tight window that the map had been placed in. So we wrote some new 'needs' that weren't neccesarily true needs but they helped inform the design decisions:

* the map must be immersive and the main focus of the application
* the experience must not be interrupted unless a decision is required

To this end, we made the map cover the whole screen ala [Google Maps](http://www.google.com/maps) because it gives a clear indication of the primary function of the application, it also gives more room to view the map itself. I also reworked the loading of the information layers so that they loaded silently in the background on page load. The interface then updates in the background as each layer is retrieved from the server. The idea is that the user has an instant on/off experience with each layer because they've been loaded on page load but they're also never interrupted.

### Go-slow Geoserver 

One of the things we found at this time was that Geoserver was great at serving the vector layers but really struggled with the base map raster layers. Even with compression quite high and pushing the server up a notch or two, the response times just weren't good enough and after about an hour of reasonable use, Geoserver fell over.

I went on the hunt for an alternative and arrived on a revamped offering from Ordnance Survey, [OS Open Space](http://www.ordnancesurvey.co.uk/innovate/innovate-with-open-space.html).

<blockquote><p>OS OpenSpace provides a set of free services and tools to enable developers to create online and mobile applications using Ordnance Survey maps</p></blockquote>

"OS OpenSpace provides a set of free services and tools to enable developers to create online and mobile applications using Ordnance Survey maps." OS Open Space is effectively another JavaScript framework and API. Thankfully OS OpenSpace is also based on OpenLayers which meant that we could not only use OS OpenSpace to serve the base maps directly from OS but we could also remove a seperate dependancy on OpenLayers itself.

I swapped OpenLayers for OS OpenSpace and only had to amend a couple of lines before the whole thing was up and running smoother than ever. With the base map now being served from OS directly, the refresh rate was fast and the quality of the tiles themselves was far higher than we were able to produce ourselves.

<figure>
	<img src="/img/content/interactive-mapping-1.png" alt="Screenshot of the mapping application">
</figure>

The final piece of the puzzle after this was a property search. Being able to find a specific property or place on the map was one of the final key requirements for initial launch. After a bit of internal faffing about I managed to knock up a quick search API against a view of our LLPG and hook that API up to the map.

The new mapping application will continue to be iterated on so who knows what's next. The current version is available to view [here](http://mapping.northampton.gov.uk) and we've made the front-end app available on [Github](https://github.com/lgss/interactive-mapping) too. Feel free to fire any questions at me if you're looking to do a similar thing.

