# Connecting the digital worlds

Hello world! As a dual student at HOMAG I took my chance to represent the HOMAG Digital Factory at [#hackthewood2019](http://www.hackthewood.com) in Berlin. In this article I want to share how we built an idea into a working prototype in less than four days.

## The idea

The challenge I was responsible for - connecting the digital worlds - consisted of the implementation of a rather simple idea: Connecting the tapio ecosystem with [IFTTT](https://ifttt.com/).

IFTTT or "**IF T**his **T**hen **T**hat" is an Internet of Things and automation platform which can be used by anyone even without technical knowledge. As the name suggests it's all about very simple automations which you can click together in their website or mobile app like:

* **IF** I like a music video on YouTube **THEN** add it to my Spotify library.
* **IF** my vacuum cleaner robot is stuck somewhere **THEN** send me a push notification to my smartphone.

If the tapio ecosystem was connected with IFTTT a carpenter could setup individual automations on his own like:

* **IF** my CNC machine set off an alarm **THEN** turn on pulsing Philips Hue lights so that a nearby employee is informed and can investigate the issue.
* **IF** I tell Amazon Alexa to turn on my edge banding machine **THEN** my edge banding machine starts up.

### How does IFTTT work?

At IFTTT things are simple, which is great because it enables everyone to use the platform.

An IFTTT-Service can be any kind of digital platform like YouTube, iRobot, Wordpress or tapio. However after a successful integration into IFTTT every service only consists of a brand, triggers and actions anymore.

An IFTTT-Trigger can be used to listen for an event to occur in its origin service. For example the BMW Labs service provides among others following trigger:

![trigger-example](assets/trigger-example.png)

As you can see a trigger has a name, a description and an amount of properties.

On the contrary an IFTTT-Action can be used to invoke something in its origin service. Actions are built with the same pattern as triggers like another example from the BMW Labs service:

![action-example](assets/action-example.png)

Finally one can combine a trigger with an action from any service, which is then called an IFTTT-Applet. Applets can be shared and discovered on [IFTTT.com](http://www.ifttt.com/discover) in the discover section. An excerpt:

![applet-example-2.png](assets/applet-example-2.png)

## The implementation

On the first day of the hackathon we used the available time to create an implementation plan broken down to small tasks. The remaining time until the presentation session on the fourth day, where all teams showcased their prototypes, consisted of coding, reading documentation and scratching our heads.

As integrating tapio into IFTTT [the official way](https://platform.ifttt.com/docs) would've been a project on its own and required access to tapio internal structures we used an IFTTT-Service called [Webhooks](https://ifttt.com/maker_webhooks) instead. There is no official specification on what exactly a Webhook is but it's generally accepted to think of a Webhook as an endpoint for a HTTP call which when called triggers something. The IFTTT Webhook service provides a trigger which can receive HTTP calls and an action which can send HTTP calls. Therefore it can be used as an interface to inject or receive any kinds of events.

![ifttt-webhook-service](assets/ifttt-webhook-service.png)

So there were two event flows we had to implement: From tapio-ready machines to IFTTT and back: 

![Sequence diagram](assets/tapio-ifttt-sequence_v1.svg)

### Building a demo machine

In order to test the tapio-IFTTT-Connector as we were developing it and also to showcase our accomplishments later on we needed a test machine which

* could run the tapio CloudConnector,
* could run a OPC UA server,
* has an input so that we could trigger events
* and has an output so that it could process actions.

The tapio CloudConnector is the piece of software from tapio which has to be installed on a machine so that it can connect to the tapio ecosystem. CloudConnector can speak a machine to machine communication protocol called [OPC UA](https://opcfoundation.org/about/opc-technologies/opc-ua/) which we can use to hook up input and output components with CloudConnector.

The first idea that came to mind was to use a [Raspberry Pi](https://www.raspberrypi.org/). It's cheap, reliable and easy to set up. It also provides a GPIO-interface which can be used to connect any input or output component like a motion sensor as input and a RGB LED as output.

So we went ahead, got ourself a Pi, installed [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/) and set up SSH access. Then we took a look at the [tapio developer portal](https://developer.tapio.one) to figure out how to install the tapio CloudConnector linux. When the CloudConnector was running on the Pi we then continued with setting up [remote debugging with Visual Studio Code](https://www.hanselman.com/blog/RemoteDebuggingWithVSCodeOnWindowsToARaspberryPiUsingNETCoreOnARM.aspx). This way the implementation of the sensor-connecting OPC UA server was a lot easier. To access the GPIO pins we used the NuGet packages [Unosquare.RaspberryIO](https://github.com/unosquare/raspberryio) and [System.Device.Gpio](https://www.nuget.org/packages/System.Device.Gpio). The latter even offered the possibility to provide a custom event handler for GPIO pin status changes which simplified listening for events by a bit.



