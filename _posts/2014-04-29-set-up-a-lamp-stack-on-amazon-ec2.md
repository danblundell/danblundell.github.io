---
layout: post
title: "LAMP stack on Amazon EC2: Part 1"
localgov: false
tag:
date: "2014-04-29"
meta: "Set up a LAMP Stack on Amazon EC2's Free Tier: Part 1"
---

Amazon Web Services offer a free tier of most of their services allowing you to host a full blown server in the cloud within their usage limits which can be pretty useful for trying things out and getting used to the options available without it costing you anything.

The next few steps is the process I went through to set up an empty server on Amazon EC2.

First, sign up to Amazon Web Services and head over to the EC2 pages from your services dashboard.

Amazon splits it's EC2 service into a few different sections so I want to clarify a few bits of terminology in a way that makes sense to me, they may not be the best technical description but ti's the easiest way I managed to get my ahead around this stuff so:

* <strong>Instances:</strong> AWS EC2 'instances' are the server itself, the processor, the memory and so forth. This is the 'machine' that's  server. They can be started, restarted, stopped, terminated and upgraded like a physical box can.

* <strong>Volumes:</strong> AWS EC2 can be tied to an Elastic Block Storage 'volume' which is essentially a separate storage drive for the server. I say 'can be tied to' because a volume is not always tied to an instance but as you become familiar with the structure, it'll become clear how and why this is the case. I recommend running volumes because they allow you to separate the data storage part of your server from the moving parts of the instance. Volumes can be attached and detached from instances in the same way you'd plug in and unplug a drive from a regular computer. Volumes can also have snapshots taken of them and be used to launch new instances.

* <strong>Security Groups:</strong> All amazon instances are locked down by default and security groups allow you to manage firewall permissions separately from your actual server which means they're instantly reusable on multiple instances. This gives you both reassurance and scalability. Adding the same set of firewall permissions to instances is ridiculously simple because you only need set them up the once. Second, if you want to update firewall permissions for all your instances, you only have to change the security group settings and the new settings will be applied to all instances associated with that group.

* <strong>Key-Pairs:</strong> EC2 instances don't have a typical username / password log in, instead you use a generated key-pair. You generate a key-pair within your EC2 dashboard and save the generated key-file somewhere safe. The important thing to note about key-pairs is that you cannot re-download the generated key file, it's a one time thing. If you lose your key-file you have to generate a new one. Equally, when you create an instance you'll be asked to specify a key-pair, the key-pair you select cannot be changed once the instance is launched. The key pair is the only way you can log into your instance so don't lose it! You can generate as many key pairs as you like so how you manage the key-files and instances is up to you.

* <strong>Elastic IP:</strong> A static IP that you can point your domain DNS towards and then from within EC2 you can reassign your Elastic IP to a server in about 3 seconds flat. This is great if you have a problem with your current instance and need to launch a new one quickly, no DNS propagation to wait for just launch the new instance and attach your Elastic IP to the new instance and your site is live. Other scenarios include zero risk, zero downtime CMS upgrades. You can create a duplicate of an existing server, perform the upgrade and test it and simply repoint the elastic IP when you're happy.

The first thing we're going to want to actually do is some prep work for make the process of launching servers and their  configurations a bit easier.

## Security Groups

If we actually want to use our server once it's created, we need some firewall rules that are going to let us in. For this guide we only want http (port 80) and ssh (port 22) as anything more just means more to think about.

Fire up your EC2 dashboard in the browser and head over to the 'Security Groups' section from the left hand side under 'Network and Security.'

Once you've got the Security Groups tab open, select 'Create Security Group.' The dashboard will open an overlay where you'll need to enter a name and description. I'm using 'Web Server' as a name and description.

To enable inbound traffic to the server, we'll need to add inbound traffic rules. In the lower 'Security group rules' section select the 'Add Rule' button and a new row will be added. Under 'Type' Amazon now gives you a helpful dropdown list of port options to select from which is a nice touch, we want to add 'HTTP' so that people can view our web pages. We'll then need to hit 'Add Rule' again but this time select 'SSH' so that we can log into the instance over SSH.

The security group does allow you to specify a few other options like 'Source' which lets you restrict where traffic is coming from but for now, HTTP and SSH should do us fine.

We can reuse this security group across servers so it's good to keep things generic at this point.

## Key-Pair

Amazon actually has two types of key/pair but for this process we only need be concerned with the more obvious one, server key pairs.

We need to remember that AWS will only let you log into a server via SSH with the correct key-pair for an instance. This is pretty important as we can't regenerate or redownload key-pairs once we've got them and we can't change the key-pair for an instance without relaunching it as a new instance, so in short; we don't want to it the generated key-file at the end of this process.

To generate a key-pair, head to the 'Key Pairs' section of the EC2 dashboard under 'Network and Security' and hit the 'Create Key Pair' button.

Name the key-pair something sensible, I'm using 'development' as my key-pair name. Follow the process, download the key-file and save it somewhere safe on your machine, you'll need it later.

Again it's a good idea to keep things generic at first to save going through this process for each server you launch, especially while you're getting used to how EC2 works, it just saves a lot of messing about.

Now we have a security group and a key pair generated, we can start the process of creating our server.

Head over to the 'Instances' tab and we'll get going.

## Launch your first Instance

Lets start with the big 'Launch Instance' button, it'll start the EC2 wizard for creating a new server. We're only interested in the free tier so head on and click select next to the 'Micro Linux 64bit AMI.' The 'AMI' part stands for 'Amazon Machine Image' and is a group of preconfigured settings for the instance, which processor / how much memory etc.

Next we get the opportunity to configure the instance, select 'Next: Configure Instance Details.' Here you get to choose things like, how many instances we're launching (1 is fine) and the availability zone of the instance, since I'm in Europe, I'm selecting EU-West-1a. You can just choose a zone nearest you.

Ignore IAM roles, that's a whole other topic and isn't relevant at this point.

The final options for this section are around shutdown behaviour and termination. I tend to always leave shutdown behaviour on 'stop' as we then have the option to start and stop and instance as we like. If we selected 'terminate' the instance would disappear and we wouldn't be able to bring it back up. Also check the 'protect against accidental termination' box which will save us accidental killing off the server. Don't worry about cloud watch monitoring for now as it takes us outside the free tier.

Once you're done, select 'Add Storage.' We're adding a separate storage volume because storage on the instance itself is only temporary and is lost when the instance is stopped, whereas storage on a volume allows us to handle our data independent of the instance. Think of a volume as a disk drive, we want to be able to keep our data even if the server is shut down.

Free tier storage is 30gb but just set an amount that you're actually going to need, I generally choose around 8/10gb and uncheck the 'delete on termination' option. This allows us to terminate our instance but keep hold of our storage volume, great for replicating our data across multiple availability zones or simply launching a test server using the current volume as a template.

The final steps to getting the server launched are tagging the instance and applying our security settings so go ahead and click 'Tag Instance.'

Here we get to pick a name for our server, something you're going to recognise the server as. It could be an application name, a domain name or simply 'dev server' - whatever you like. Once you've added a name, click 'Next: Configure Security Group'

We've already set up a security group so we can simply choose 'Select and existing security group' and check the box next to our 'Web Server' security group.

Finally we click 'Review and Launch' which offers us the last option of choosing our key-pair or generating a new key-pair. We've already got our key-pair to hand to select 'Choose an existing key-pair' and select 'development' or whatever you called your key-pair and hit that launch button.

Amazon will whirl away for a while spinning up an instance and a  volume for us and once it's done the main 'Instances' page will say 'running'

## So my instance is running, how do I get onto it?

This is something that was quite baffling to me at first, I had a server, I had all these settings but how the heck did I use the server?! Thankfully it's actually quite simple, one SSH command gets you in and then you're free to do as you wish.

So what is it? Well, fire up terminal and change to the directory you put your key pair file in so for me that would be something like:

{% highlight bash %}
cd ~/Documents/keys/amazon
{% endhighlight %}
Then we want to SSH to the server using a the key file as our authentication so for me that would be:
{% highlight bash %}
ssh -i development.pem ec2-user@amazon-dns-path-goes-here
{% endhighlight %}

The ec2-user is the default user for all Amazon Machine Images (AMIs) so you can use that, however, the amazon public DNS path is specific to your instance. As a helpful addition it's actually listed on your 'Instances' page in the browser, just tick the box on the left of your newly launched instance and scroll through the lower pane of the page until you see 'Public DNS.' Copy and paste it the value into the terminal.

Once you hit enter, you'll be connected to your instance and there'll be a nice little message from Amazon to tell you you're in.

As a final point, I tend to run 'sudo yum update -y' immediately, just to update the server to the latest version of any installed packages.

That's it, you're done! An AWS EC2 instance launched with a EB Volume for storage. The server's all yours to do whatever you like with, if you're at a loose end and want to set the server up as a LAMP stack, you can follow the next tutorial where I'll take you through installing apache, mysql and php as well as making the most of the micro instance by introducing a bit of additional memory.
