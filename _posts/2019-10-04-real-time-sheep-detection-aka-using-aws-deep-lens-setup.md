---
layout: post
title: "Real time 'is that a sheep' detection aka using AWS DeepLens"
date: "2019-10-04"
meta: ""
image: ""
categories: ["tech"]
published: true
yt: hJ7HBvQPSJM
---

A few weeks ago I was kindly loaned a DeepLens video camera by AWS. The camera has a video feed and an AI engine capable of processing that video feed in real time. This means you can use the camera to do things like detecting specific objects or faces and when the camera 'spots' something or someone, trigger some kind of action. 

Its a fascinating bit of kit so here's a bit of a run through of how I got on with it straight out the box.

### Registering the camera

First step was to unbox the camera, get it plugged in to some power and a laptop.

Once it's powered on and connected to a laptop, you register the camera into an AWS account. This binds the camera, to your account and therefore, your control. Yay security.

Registration is pretty simple, you just go to the [Deep Lens][deeplens] service in your AWS account and choose 'Register'. From there, a wizard 'just keep clicking next' type process guides you through until some whizzy confirmation screens tell you everything's cool. If something breaks there, a) I have no idea how to help you b) Good luck.

There were a few other things to do at this point but it's all really well documented. Ultimately the camera went from box to ready to go in under 30 minutes which is quite impressive.

### Real time sheep detection
Once the camera was up and running, the simplest starting point I found was to deploy the pre-made ['object detection'][object-detection] project, it's one of [eight template projects][template-projects] that AWS supply to get you started quickly.

Included in the eight templates there are projects for; Object detection which detects 20 'popular' objects, 'Artistic style transfer' which makes the camera feed look like a Van Gogh painting, face detection which guess what, detects a face and also the infamous hot-dog-or-not or [Silicon Valley][silicon-valley] fame. I went with object detection, mostly because I wanted to see how many objects I had nearby were considered 'popular'.

Deploying the templated project is _really_ easy (like, Noddy simple). It was a case of hitting 'Create project' and choosing a template.

![Create a project in AWS DeepLens][create-a-project-image]

Once a project's created, it needs to be deployed to a camera, otherwise it just sits there, doing nothing, sad and alone.

Thankfully deploying a project to the camera has been made really simple too, just select the project and click 'Deploy to device'.

![Option for deploying a project to a camera][deploy-to-camera]

It takes a bit of time to deploy to the device but once it's done, you can immediately view the camera video stream and watch the AI engine do its thing.

Here's what it looks like directly from the camera's video stream:

{% include youtubePlayer.html id=page.yt %}

As you may be able to see from the video, the objects detected were 'dining table', 'person', 'bottle' and 'chair'. Sadly I didn't have a sheep, aeroplane or cow to hand to give those a try, however, I have been sufficiently assured by the [documentation][object-detection] that it can, indeed, detect sheep.

### Sheep detected, then what?

As well as the video stream, AWS DeepLens is automagically integrated with the [AWS IoT console][iot-console] which means you can see a 'message stream' showing the camera's interpretations, in other words, the code equivalent of the video stream.

In other words, the message stream means that you can have other services _subscribe_ to messages from your AWS DeepLens camera. You can then decide whether or not to react or respond to what the camera sees, depending on the message. 

For example, I'm currently working on creating a service to send a welcome message to each person in our team whenever the camera sees them enter the office. That said, I might just set it to recognise me because I'm a crazed narcassist (not really).

Here's an example of what the message stream looked like for the earlier video.

![The message stream from the AWS IoT console showing a feed from the AWS DeepLens camera][message-stream-image]

Each message shows name of any objects the camera detected alongside a probability score (how confident the AI model is that the object is actually that object).

### Future's crazy y'all
The first time I deployed a project to the camera and saw the video stream detecting objects in the room around me, I danced in my chair like a complete weirdo. I was so happy, mostly because it worked but also because it was totally astounding. This kind of technology is £235 and available for almost anyone to use. 

You can literally buy the the camera on [Amazon.co.uk][deeplens-buy] which is a _consumer_ store. It's incredible to think that a high-definition video camera with customisable AI technologies is available online for almost anyone to buy with next day-delivery.

This puts some incredibly powerful technology in the hands of a lot more people, likely to produce far more diverse and interesting services and opportunities in the future, especially as the price of technology continues to drop and the quality, reach and scale increases exponentially.

### What's next?

As far as use cases go, there are plenty but my teams first project will be to use DeepLens camera to replace a mobile app we built not too long ago to create a safer working environment for workers on-site at recycling centres.

We built a mobile app for a team at Cambridgeshire County Council. The app enables them to photograph a vehicle to check if that vehicle has a valid permit. The app uses image recognition to grab the number plate from the photo and look up that number plate against permits. 

We'll be running a pilot project to see if that whole process of 'take a photo, check the permit, deduct a visit' can be automated and deployed to the camera. If we can, the app wouldn't really needed and we'd free up the team to concentrate on their main job of keeping people safe from giant waste-crushing machinery at recycling centres.

🚀

[deeplens-buy]: https://www.amazon.co.uk/AWS-DeepLens-2nd-Generation-learning-enabled/dp/B07KYLSRZM/
[deeplens]:https://aws.amazon.com/deeplens 
[object-detection]: https://docs.aws.amazon.com/deeplens/latest/dg/deeplens-templated-projects-overview.html#object-recognition
[template-projects]: https://docs.aws.amazon.com/deeplens/latest/dg/deeplens-templated-projects-overview.html
[iot-console]: https://console.aws.amazon.com/iotv2/home?region=us-east-1#/test

[create-a-project-image]: /img/content/deeplens-create-a-project.png
[message-stream-image]: /img/content/deeplens-video-object-detection-text.gif
[deploy-to-camera]: /img/content/deeplens-deploy-to-camera.png
[silicon-valley]: https://www.hbo.com/silicon-valley