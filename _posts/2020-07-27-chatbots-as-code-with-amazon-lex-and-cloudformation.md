---
layout: post
title: "Chatbots as Code: automatically deploying Amazon Lex bots using Cloudformation"
date: "2020-07-27"
meta: ""
image: ""
categories: ["cloudformation", "aws", "technology", "lex", "aws sam", "infrastructure as code"]
published: true
youtubeId: RdRnJPIbYmc
---

<p class="lede" markdown="1">Creating a bot as a one-off is [pretty well documented][lex-tutorial] but replicating that bot consistently and safely to create multiple environments is a little bit more tricky. How do you safely and consistently create, update and throw away bots in a similar way that we can with virtual servers, databases and other infrastructure services?</p>

{% include youtubePlayer.html id=page.youtubeId %}

Developing bots of one type or other is something we’ve been asked to work on more and more. More recently we've created a bot that answers phone calls and redirects them like a switchboard operator depending on which service you ask for. Another is a bot that will run automated surveys at the end of a transaction or phone call and capture customer feedback.

![chatbot-conversation-gif]{:class="fr"}

But the challenge isn't creating a bot itself, the challenge is how do we create a bot with multiple environments in a similar way that we're used to on more typical software projects? What happens when the bot starts being used by people and we need to work on the next version, make changes or test out new ideas?

We’d normally be able to easily create multiple environments for development, testing, quality assurance (QA) and production but with a bot, this process involves a lot of manual work. It's made worse still because there's so much configuration with bots, questions, response types, workflow and logic that all needs to sit neatly together but often using separate services. How do we safely handle the configuration of multiple environments in a consistent and predictable way?

I wanted to find a way to use what we know about Infrastructure as Code, with bots, so that we get the same level of safety and consistency with our bot applications as we do with other software.

Before I get into how we’re currently doing this, there's a bit of a pre-requisite. Typically we'd use [Cloudformation][aws-cloudformation] or the [Serverless Application Model (SAM)][aws-sam] to code up AWS infrastructure. Both these services take a tempalte file and make the various required API calls to Amazon services to create infrastructure. 

Surprise! Out the box, Cloudformation doesn't support Amazon Lex 🎉. Equally, because [AWS SAM][aws-sam] is a wrapper for Cloudformation, it doesn't support Lex either, so yeah, that's a problem.

Lucky for this post, we'll get over that hurdle pretty quickly (yay). Onward.

## Amazon Lex as a Custom Resource
Before we can create our bot using Cloudformation, we need to create custom resources. These will allow us to create custom slots, intents and bots using Cloudformation.

Cloudformation allows for ‘custom’ resources as an extension of the native Cloudformation functionality. Once created, Cloudformation templates can reference resources that wouldn’t otherwise be available.

In this case, Cloudformation needs three custom resources to exist; 
* [cfn-lex-bot][cfn-lex-bot] which wraps the PutBot method from the Lex API
* [cfn-lex-intent][cfn-lex-intent] wraps the PutIntent method from the Lex API
* [cfn-lex-slot-type][cfn-lex-slot-type] wraps the PutSlotType from the Lex API

For this project, I forked [cfn-lex-bot][cfn-lex-bot], [cfn-lex-intent][cfn-lex-intent] and [cfn-lex-slot-type][cfn-lex-slot-type] and made a couple of changes. 

Amazon Lex is now available in the London region so I changed the [deployment region for Lex resources to eu-west-2][github-repo-region] and whilst I was there, [fixed a small bug][github-repo-commit] where the `create` process worked fine but subsequent `update` operations failed because the checksum couldn’t be found. But y'know, that's probably too much info.

The readme files on each repo should be relatively easy to follow and get deployed into your own AWS account if that helps you and equally, there's a one-click deploy option in the readme that should get you up and running fairly easily.

## Deploying a bot using the Serverless Application Model or Cloudformation

With custom resources deployed and available in our AWS account, we’re now able to use the `bot`, `intent` and `slot-type` Cloudformation resources in a template.

For this use case, we’ve actually used [AWS SAM][aws-sam], a wrapped for Cloudformation that gives a few extra options for our CI/CD pipeline but you could use native Cloudformation if you choose to. Or don't, whatever.

Each time we deploy our template, it’ll create a new ‘stack’. This creates a set of linked, resources that can be managed as a whole application rather than disparate, separate services or bits of infrastructure. This is great in our use case because a bot-driven application typically uses quite a few different services so finding a way to link them together as one 'application' helps make sense of things long term creating a much more sustainable way of working.

The full code of this example is on [Github][github-repo] but the next few sections walk through the template to give a little bit of insight beyond the comments in the code.

### Template overview

The first part of our template if for our parameters. We only really need to specify what kind of workload or environment we want us to run as an easy way to distinguish between stacks.


{% highlight yaml %}
Parameters:
  Workload:
    Type: String
    Default: "dev"
    Description: "The type of workload (environment) that this stack will run"
{% endhighlight %}

Next we define our resources. Resources, in this case, are our custom slot-type, intent and the bot itself. We’re also deploying a validation function using [AWS Lambda][aws-lambda]. This function validates user input with some custom logic to work out what question the user should be asked next.

#### Custom Slot Type
Custom slot types are defined according to the [Lex PutSlotType API documentation][lex-documentation-putslottype]. The key part on being able to include this in a SAM or Cloduformation template is the `Service Token` which specifies where CloudFormation sends requests to. In our case, this value should be the Exported value from our `cfn-lex-slot-type` custom resource stack.

{% highlight yaml %}
likertScaleSlot:
Type: Custom::LexSlotType
Properties:
    ServiceToken:
    Fn::ImportValue: cfn-lex-slot-type-1-0-3-ServiceToken
    name: !Sub
    - '${AWS::StackName}_agree_disagree_slot_${Workload}'
    - Workload: !Ref Workload
    description: 1-5 scale for strongly agree through to strongly disagree
    valueSelectionStrategy: TOP_RESOLUTION
    enumerationValues:
    - value: 1
    synonyms:
        - strongly agree
        - yeah loads
    - value: 2
    synonyms:
        - agree
    - value: 3
    synonyms:
        - neither
        - don't care
        - not bothered
    - value: 4
    synonyms:
        - disagree
    - value: 5
    synonyms:
        - strongly disagree
{% endhighlight %}

This creates a custom slot type akin to a [likert scale][likert-scale]. The slot values include synonyms as examples and by setting the `valueSelectionStrategy` to `TOP_RESOLUTION`, the values are treated as synonyms and the values are fixed to those provided in the template.

#### Intent
[The intent][github-repo-intent] template snippet isn't included here as it's massive and does a majority of the work. It brings together utterances (the things users might say to suggest what they need) and slots (the various bits of information that the bot needs to fulfill what the user wants). We’re once again using `ServiceToken` to point Cloudformation at our custom `cfn-lex-intent` resource stack.

The first thing to notice is that our intent `DependsOn` our custom slots, Lambda function and permission resources existing already. Whilst I don’t think this is required, it seems like good practice to make it explicit.

All the other settings map to those in the [Lex PutIntent API documentation][lex-documentation-putintent] so should be referenced there.

#### Bot
Adding the bot to the template is actually fairly lightweight in comparison to the intent and it contains properties documented in the [PutBot][lex-documentation-putbot] method of the Lex API.

{% highlight yaml %}
  conciergeBot:
    Type: Custom::LexBot
    DependsOn:
      - conciergeBotIntent
    Properties: 
      ServiceToken:
        Fn::ImportValue: cfn-lex-bot-1-0-4-ServiceToken
      name: !Sub
        - '${AWS::StackName}_bot_${Workload}'
        - Workload: !Ref Workload
      abortStatement:
        messages:
          - content: "I don't understand. Can you try again?"
            contentType: "PlainText"
          - content: "I'm sorry, I don't understand."
            contentType: "PlainText"
      childDirected: false
      clarificationPrompt:
        maxAttempts: 1
        messages:
          - content: "I'm sorry, I didn't hear that. Can you repeate what you just said?"
            contentType: "PlainText"
          - content: "Can you say that again please?"
            contentType: "PlainText"
      description: 'a bot'
      voiceId: Joanna # have to use a en-US voice because of locale error
      idleSessionTTLInSeconds: 300
      intents:
        - intentName:
            Ref: conciergeBotIntent
          intentVersion: "$LATEST" # always use the latest intent
      locale: en-US # for some reason it won't build as en-GB which is odd, throws a 400 error
      processBehavior: "BUILD" # build automatically, although this doesn't trigger if the bot itself isn't changed
{% endhighlight %}

Worth noting is the `processBehavior` which is set to `BUILD` in our instance so that when the bot resource is created by Cloudformation, it automatically builds and is ready to use straight away.

#### Other bits

Our Lambda function, policies, permissions and parameters are all as described in the AWS SAM and Cloudformation documentation so nothing really that notable there.

#### Deployment
The [full AWS SAM template][github-repo-template] can be deployed with AWS SAM as-is using `sam deploy --guided` and will create a ready-to-use bot that captures survey feedback from the user. 

### Things to note

* For some reason bots won’t deploy with the locale of ‘en-GB’. The API documentation says this should be possible but when using en-GB the returned error code suggests a permission issue. Haven’t spent any time on it to work it out but it does work fine with ‘en-US’. 
* The permission to invoke the Lambda function should really restrict access down further, currently it lets anything Lex invoke the function, this really needs to be down to the bot or intent level to tighten this up.
* Any Lex API options can be included as properties in the template file. This means, if there’s an option in the Lex PutBot, PutIntent or PutSlotType API that you need to use but it’s not set in the example Cloudformation template, including that option in your template file should ‘just work.’
* Editing the configuration of the bot in the console once deployed is really tempting, changing a question phrasing or adding another synonym. Don’t do this. Once part of a stack, much better to manage everything in code
In another project, we’ve moved slot values to a database table so that they can be managed as code more easily and outside the ‘infrastructure’ of the bot.
* Using `sam deploy` if you change something in a slot-type or intent, the bot doesn't automatically rebuild because the `bot` resource hasn't changed. To get the bot to rebuild every time, we'll be switching to a CI/CD pipeline and scripting another API call post-deployment.

### Round up

Now that it’s working, the process for deploying an AWS Lex bot using AWS SAM/Cloudformation is pretty stable. It’s now as easy to create one new bot as it is to create 100 bots with exactly the same configuration which means we can confidently create, throw-away and recreate bot environments with the same confidence we would virtual servers or databases.

[github-repo]: https://github.com/danblundell/aws-sam-feedback-bot
[github-repo-template]: https://github.com/danblundell/aws-sam-feedback-bot/blob/master/template.yml
[github-repo-commit]: https://github.com/danblundell/cfn-lex-bot/commit/65b754932045dbac222553444c0969ac0d874a5b
[github-repo-region]: https://github.com/danblundell/cfn-lex-bot/commit/9a669a3001ab65c33ed335ec35719e1a1aa529b0
[github-repo-intent]: https://github.com/danblundell/aws-sam-feedback-bot/blob/c5ad20b19e14c5ffa72d031aca5457a93dd48a00/template.yml#L153

[cfn-lex-bot]: https://github.com/danblundell/cfn-lex-bot
[cfn-lex-intent]: https://github.com/danblundell/cfn-lex-intent
[cfn-lex-slot-type]: https://github.com/danblundell/cfn-lex-slot-type

[aws-sam]: https://aws.amazon.com/serverless/sam/
[aws-cloudformation]: https://aws.amazon.com/cloudformation/
[aws-lex]: https://aws.amazon.com/lex/
[aws-lambda]: https://aws.amazon.com/lambda/

[lex-documentation-putintent]: https://docs.aws.amazon.com/lex/latest/dg/API_PutIntent.html
[lex-documentation-putslottype]: https://docs.aws.amazon.com/lex/latest/dg/API_PutSlotType.html
[lex-documentation-putbot]: https://docs.aws.amazon.com/lex/latest/dg/API_PutBot.html
[lex-tutorial]: https://docs.aws.amazon.com/lex/latest/dg/getting-started.html

[likert-scale]: https://en.wikipedia.org/wiki/Likert_scale

[chatbot-conversation-gif]: /img/content/chatbots-as-code-chat.gif
