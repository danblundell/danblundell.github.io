---
layout: post
title: "AWS Application Load Balancers and WordPress"
tag:
date: "2019-01-02"
meta: ""
---

You had a problem recently where a WordPress site that used an AWS Application Load Balancer didn't load over HTTPS. We kept seeing alot of `mixed-content:blocked` errors when the site loaded and even though all the usual settings within WordPress were set to use HTTPS, things like JavaScript files and stylesheets were still being requested over http.

## Context
We'd set up our application load balancer to serve HTTPS (port 443) traffic to the client but have the connection between the load balancer and the web servers over HTTP (port 80). This means less management and configuration on the servers but it seems to cause a problem if the application you're serving uses things like port number to determine if the request is HTTPS or not. 

We found this out first hand using with WordPress behind an application load balancer, and it got a little confusing. 

The problem stems from the way the [is_SSL()](https://core.trac.wordpress.org/browser/trunk/src/wp-includes/load.php#L1060) WordPress function works. 

The WordPress `is_SSL()` function checks two things;

1. the HTTPS server variable, it's expecting the HTTPS variable to be either `1` or `on`
2. the port number, it's expecting it to be `443`

This makes complete sense as for HTTPS requests those are the expected values. That's not quite the case if you're behind a load balancer and only encrypting traffic between the load balancer and the client (like we were, eek).

Instead, the traffic comes from a client to the load balancer over port 443 (HTTPS) but once it hits the load balancer, the load balancer sends the request to a server over port 80 (straight up HTTP). This means to the servers knowledge, the request is HTTP so it gets handled like and HTTP request.

We needed WordPress to understand that the traffic was actually from an HTTPS request so that when it worked out what to send as a responses it generated https links not http links.

## Solutions
Thankfully one of the headers available once a request has been forwarded from a load balancer to a server is `X-Forwarded-Proto`. 

In short, `X-Forwarded-Proto` is the protocol of the request that the load balancer recieved rather than the one the server is being sent. In our case, it's `https`. Bonus.

This means we could use the `X-Forwarded-Proto` header to determine if the request from the client was HTTPS or not, even though the request from the load balancer to the server was over HTTP.

So how do we use this I hear you cry, well a few options depending on where you prefer to handle your changes and what orchestration you have in place.

### Web server (Apache)
Add the following to your web server config, something like `vhost.conf` would do just nicely.

{% highlight aconf %}
<VirtualHost *:80>
SetEnvIf X-Forwarded-Proto "https" HTTPS=on
</VirtualHost>
{% endhighlight %}

This checks if the request that terminated on the load balancer was an HTTPS request and if so, sets the local `HTTPS` environment variable to `on` (true).

### WordPress directly (wp-config.php)
Alternatively you could add something inside your `wp-config.php` file which will do a similar job;

{% highlight aconf %}

if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
{
  $_SERVER['HTTPS'] = 'on';
}
{% endhighlight %}


This checks if the request that terminated on the load balancer was an HTTPS request and if so, sets the local `HTTPS` environment variable to `on` (true).

## Summary
Although ideally we'd end-to-end encrypt, this does make some orchestration and scaling a little easier as we don't have to worry about enabling or managing SSL on the individual servers. In this case we can just let the load balancer handle that traffic and let the servers serve content.