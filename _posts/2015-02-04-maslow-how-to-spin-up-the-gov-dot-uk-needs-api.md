---
layout: post
title: "Maslow - how to spin up the GOV.UK Needs API"
localgov: true
tag:
date: "2015-02-04"
meta: "Set up the GOV.UK needs api on your own machine"
image: "maslow-local.jpg"
---

Caveat: This post should've been written back in October 2014 but life got in the way and I've forgotten most of it, however, like all good developers I wrote myself some great documentation (not) so this should be a breeze right? (possibly not). Either way, I'll try and make it easy to follow!

To bring you up to speed, GDS built an application called [Maslow](https://github.com/alphagov/maslow) to capture and manage the user needs that [GOV.UK](https://www.gov.uk) is designed to meet. I'm sure someone from GDS can correct me on such an underwealming description but regardless, it's a really smart idea and something I hope can be adopted or replicated elsewhere across the public sector.

I appreciate [GDS]() have never intended for their code to be used like an open source project and have far higher priorities than writing documentation for morons like me so here's my brief-ish guide to getting up and running with [Maslow](https://github.com/alphagov/maslow).

## Pre-requisites

This guide assumes:

* You're on a mac running OS X 10.9+
* You're reasonably familiar with ruby, the command line and git
* You're willing to remain calm in the event of having to install a bunch of stuff as we go along

## Step 1: Clone Maslow

We may aswell start simple right?

Depending on your intentions you can either clone [Maslow](https://github.com/alphagov/maslow) directly using the command below head over to the [Maslow repository](https://github.com/alphagov/maslow), fork it yourself. Either way, make sure you get a copy on your machine.

{% highlight bash %}
git clone https://github.com/alphagov/maslow.git
{% endhighlight %}

Once you've done that, give yourself a pat on the back - first job, done.

## Step 2: Clone the Need API

Maslow depends on having a running instance of the [GOV.UK Need API](https://github.com/alphagov/govuk_need_api). Thankfully the Need API is pretty straightforward itself so head on over and grab yourself a copy, you can use the command below or the [GOV.UK Need API repository](https://github.com/alphagov/govuk_need_api)

{% highlight bash %}
git clone https://github.com/alphagov/govuk_need_api
{% endhighlight %}

Sweet job, we now have the [GOV.UK Need API](https://github.com/alphagov/govuk_need_api) on our machine. 

## Step 3: Starting the Need API

Now we have the Need API and Maslow locally, we can start the process of getting them up and running.

The Github page for the Need API says the dependencies are:

* [Elasticsearch](http://www.elasticsearch.org/download)
* [Mongo DB](http://www.mongodb.org/)

You'll need to install both of those. While you're there, what they don't tell you is that [Redis](http://redis.io/topics/quickstart) is also required, go ahead and install that too. (I know right?)

All installed? Move on...

## Step 4: Getting dependencies running

In a new terminal window, head to your Elasticsearch install and start it up:

{% highlight bash %}
bin/elasticsearch
{% endhighlight %}

Oh and we'll also need to have Mongo running, start mongo in a new terminal window using:

{% highlight bash %}
mongod
{% endhighlight %}

Almost there, open another window and get Redis running:

{% highlight bash %}
redis-server
{% endhighlight %}

At this point we should be okay to run the Need API itself, open another terminal window, change directories until you're in your govuk_need_api directory and hit:

{% highlight bash %}
./script/bootstrap
bundle exec unicorn -p 3000
{% endhighlight %}

All being well you should be able to visit [http://localhost:3000](http://localhost:3000) and have a screen that looks a little like the one below.

<figure>
	<img src="/img/content/need-api-local.jpg" alt="GDS's Need API application running locally in devleopment mode">
</figure>

If you get this far, [tweet me](https://twitter.com/intent/tweet?text=@danblundell%20I%20survived%20your%20guide%20to%20Maslow!&amp;related=gdsteam&amp;via=danblundell&amp;hashtags=localgovdigital,localgov,gdsteam,ftw) and I'll work out some sort of prize/badge/hug type deal, points for persistance.

Right, now we have the GOV.UK Need API running, we can get attempt to Maslow running too.

## Step 5: Getting Maslow running

Now, you would hope that you could just follow the instructions on the Github readme page, no such luck unfortunately. What I mean to say is, you can, but it probably won't work.

In short, Maslow has an object reference to the Need API. I don't know nearly enough about ruby or rails to know exactly how it works, I'm still learning but to get up and running, open up the following file:

{% highlight bash %}
~maslow: config\initializers\gds_api.rb
{% endhighlight %}

On about line 5, you'll see the following:

{% highlight ruby %}
Maslow.need_api = GdsApi::NeedApi.new(Plek.current.find('need-api'),
                                      API_CLIENT_CREDENTIALS[:need_api])
{% endhighlight %}

Swap it for this:

{% highlight ruby %}
Maslow.need_api = GdsApi::NeedApi.new('http://localhost:3000/',
                                      API_CLIENT_CREDENTIALS[:need_api])
{% endhighlight %}

All we're doing is swapping out the reference to [Plek](https://github.com/alphagov/plek), GDS's internal dns lookup, for a hardcoded reference to our local instance of the Need API, running on port 3000.

Sidenote: you can set an environment variable and use Plek itself but this is the quick and dirty method that'll get you moving quicker - no one needs hassle right?

## Step 6: The home straight

One final thing before we start Maslow...

We're not using the [GDS authorisation platform](https://github.com/alphagov/gds-sso) so Maslow seeds the database with a user for us. Awesome right?

The annoying thing is, unless we make a change, all you'll get is permission to sign-in to Maslow, you won't get any permissions to create or update needs or do anything of any real use.

So one final thing, updating the user permissions of the default user:

Head to:

{% highlight bash %}
~maslow: db\seeds.rb
{% endhighlight %}

and find the line that defines the users permissions:

{% highlight ruby %}
@user.permissions = ["signin"]
{% endhighlight %}

change that to

{% highlight ruby %}
@user.permissions = ["signin","admin","editor"]
{% endhighlight %}

Excellent, now we can fire it up!

## Fire it up!

Now you should be able to fire up Maslow, open one final new terminal window and head into the directory where you've cloned maslow then run:

{% highlight bash %}
./script/bootstrap
bundle exec unicorn -p 3001
{% endhighlight %}

All being well, you should now have Maslow up and running on your local machine, it'll look something like this:

<figure>
	<img src="/img/content/maslow-local.jpg" alt="GDS's Maslow application running locally in devleopment mode">
</figure>


That's it! If you got this far, bravo. I'm amazed I did. I have pushed this a bit further and got a bunch of local authorities loaded instead of the central government departments from Whitehall but I need to revisit how I ended up doing that to be honest as it was horrendously hacky. As I say, my ruby knowledge is pretty limited, I've just hacked my way through error messages until this point.

GDS have far more important things to do than document how their stuff works so hopefully this goes some way to appeasing that frustration - if anyone else has got GOV.UK code up and running elsewhere, please let me know!