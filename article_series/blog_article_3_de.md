Nachdem der erste Ereignisablauf einmal erfolgreich durchlief, begannen wir mit der Arbeit am zweiten:

1. Ein Sensor einer Maschine hat seinen Zustand geändert.
2. Die Zustandsänderung wird an den OPC UA Server der Maschine weitergeleitet.
3. Der OPC UA Server löst ein OPC UA Ereignis aus.
4. Der CloudConnector leitet das Ereignis an den tapio Core weiter.
5. Der tapio Core leitete das Ereignis an einen benutzerdefinierten EventHub weiter (vorkonfiguriert für die tapio-Maschinen-ID).
6. Der Connector bekommt eine neue Nachricht vom EventHub.
7. Der Connector interpretiert die Nachricht als Ereignis und leitet dieses an IFTTT über den Webhook-Service weiter.
8. In IFTTT empfängt ein IFTTT-Applet das Ereignis und leitet es entsprechend seiner Konfiguration an einen beliebigen Dienst in IFTTT weiter.

Ein [Azure EventHub](https://azure.microsoft.com/en-us/services/event-hubs/) kann viele Ereignisse gleichzeitig verarbeiten und stellt eine Warteschlange bereit, über welche die einzelnen Ereignisse von verschiedenen Konsumenten nacheinander verarbeitet werden können.
Wir haben eine EventHub-Instanz in Azure eingerichtet und diese über [my tapio](https://admin.tapio.one/) mit unserer Testmaschine verbunden. Dann haben wir eine weitere Azure Function eingerichtet, welche Nachrichten aus dem EventHub ausliest, diese als Ereignisse interpretiert und über den Webhook-Service an IFTTT weiterleitet.

## Fazit

Das war 's! Zwei Azure Functions, ein EventHub, ein Raspberry Pi, ein paar Sensoren und drei Werktage später konnten wir einen funktionsfähigen Prototyp präsentieren. Für unsere Demo am vierten Tag haben wir die Daten eines Bewegungssensors an unserer Testmaschine über den tapio-IFTTT-Connector in eine Google Drive Tabelle protokolliert, eine RGB-LED mit einem Widget auf einem Smartphone eingeschaltet und ein neues IFTTT-Applet live konfiguriert. Wir haben kein fertiges Produkt entwickelt, aber einen funktionierenden Machbarkeitsnachweis geliefert, welcher in ein richtiges Produkt weiterentwickelt werden kann. Eine Form der Authentifizierung, Autorisierung und eine Weboberfläche zur Konfiguration von maschinenbezogenen Ereignissen stünden beispielsweise noch aus.

Neben der Bearbeitung der Herausforderungen war der Hackathon aber vor allem eine lustige Veranstaltung mit motivierten Teilnehmern, welche sich jederzeit gegenseitig ausgeholfen haben und gemeinsam eine spannende Woche in Berlin erlebt haben! :)
# Connecting the digital worlds (3/3)

In the [previous article][article_2] we implemented the flow of events from IFTTT to tapio-ready machines. Now we want to implement forwarding events from tapio-ready machines to IFTTT.

* [Connecting the digital worlds (1/3)][article_1]
* [Connecting the digital worlds (2/3)][article_2]
* [Connecting the digital worlds (3/3)][article_3]

![Sequence diagram](assets/tapio-ifttt-sequence-from-machine.png)

First off we need a way to trigger an event on our demo machine. We decided to go with a [AM312](https://www.sunrom.com/p/micro-pir-motion-detection-sensor-am312) [PIR](https://en.wikipedia.org/wiki/Passive_infrared_sensor) motion sensor which can be directly connected to the demo machine through its [GPIO interface](https://www.raspberrypi.org/documentation/usage/gpio/). When properly connected to a ground pin, a voltage pin and a generic GPIO data pin it'll output a current on the GPIO pin when there is motion in front of it.

Now that our sensor is ready we have to monitor it for status changes in order to react to motion events. Microsofts GPIO library [System.Device.Gpio](https://github.com/dotnet/iot) provides a method `RegisterCallbackForPinValueChangedEvent` through which one can subscribe to rising or falling currents. Using this method we wrote a little wrapper to simplify things a bit:

```csharp
public class MotionSensorMonitor : IMotionSensorMonitor
{
    private int _Pin;

    public event EventHandler OnMotion;

    public MotionSensorMonitor(int pinBoardNumber)
    {
        _Pin = pinBoardNumber;
        var controller = new GpioController(PinNumberingScheme.Board);
        controller.OpenPin(_Pin);
        controller.SetPinMode(_Pin, PinMode.Input);
        controller.RegisterCallbackForPinValueChangedEvent(_Pin, PinEventTypes.Rising, (sender, args) =>
        {
            OnMotion?.Invoke(this, new EventArgs());
        });
    }
}
```

Now that we're able to react to status changes we want them reflected in the OPC UA server of our demo machine so that these changes can get forwarded to tapio through the CloudConnector also running on our demo machine. So we have to a add a node to our OPC UA servers address space:

```csharp
protected override void CreateAddressSpace()
{
    base.CreateAddressSpace();

    _MotionSensorState = new DataVariableState<string>(false, "MotionSensorState", RootFolder, SystemContextObject);

    AddNode(_MotionSensorState);
}
```

Then we have to add an event handler method to our node manager which updates our `MotionSensorState` node when called:

```csharp
public void OnMotionDetected()
{
    if(_MotionSensorState == null)
    {
        throw new InvalidOperationException("Please create the base event state first");
    }
    _MotionSensorState.StatusChanged("motiondetected", StatusCodes.Good);
}
```

Finally we have to hook up the motion sensor monitor with our event handler method:

```csharp
static void Main(string[] args)
{
    var led = new Led(16, 20, 21);
    var ledController = new LedController(led);
    var motionSensorMonitor = new MotionSensorMonitor(10);
    var nodeManager = new NodeManager(ledController, motionSensorMonitor);
    var server = new Server(nodeManager);
    motionSensorMonitor.OnMotion += nodeManager.OnMotionDetected;
}
```

Now we're able to reflect motions in front of our PIR to OPC UA node status changes. CloudConnector can forward these status changes to tapio if we configure the data module of the CloudConnector [correctly](https://developer.tapio.one/docs/CloudConnector/DataModule.html#sourcedataitem). Keep the configured `SrcKey` in mind.

```xml
...
<SourceItem xsi:type="SourceDataItem">
  <NodeId>ns=2;s=PiSensorServer.MotionSensorState</NodeId>
  <SrcKey>MotionSensorState</SrcKey>
</SourceItem>
...
```

As tapio [supports](https://developer.tapio.one/docs/TapioDataCategories.html#streaming-data) streaming this data into an [Azure Event Hub](https://azure.microsoft.com/en-in/services/event-hubs/) we deployed us one through the [Azure Portal](http://portal.azure.com/) and configured an app in [my tapio](https://my.tapio.one/) to forward the data to our event hub.

Data ingested into an Azure Event Hub can easily be processed, stored or served for third party apps. As we only want to process and forward events as they come in we opted for a serverless approach [again][article_2] with another Azure Function hooked up to our event hub.

Below you can see how an event message ingested into our event hub looks like:

```json
{
  "tmid": "741ab3a2-040a-44bf-b8ce-4333d567a99a", // machine id
  "msgid": "bf8610fa-a5e9-4ede-ad37-51082e7eb372", // message id
  "msgt": "itd", // message type
  "msgts": "2017-06-29T10:39:03.7651013+01:00", // ISO8601 timestamp 
  "msg":  {
    "p": "pi", // provider name
    "k": "MotionSensorState", // key, usually the node id
    "vt": "s", // data value type (string)
    "v": "motiondetected", // payload
    "q": "g", // quality of the value (OPC UA thing)
    "sts": "2017-06-29T10:38:43.7606016+01:00", // ISO8601 timestamp by OPC UA server
    "rts": "2017-06-29T10:38:53.7606016+01:00" // ISO8601 timestamp by CloudConnector
  }
}
```

[Streaming data](https://developer.tapio.one/docs/TapioDataCategories.html#streaming-data) messages from tapio received by our event hub can have different structures. We're only interested into [item data](https://developer.tapio.one/docs/TapioDataCategories.html#item-data) messages because we just want to monitor status changes of our PIR motion sensor monitor node. You can recognize an item data message if you take a look at the `msgt` (message type) key. Every item data message has the type `itd`.

Inside the value of the `msg` key we can find the actual item data message as JSON object. Here we're looking for messages with their `k` (key) being the previously configured `SrcKey`. The value we're forwarding hides behind the `v` key. Once again to keep things simple this is only a string (`motiondetected`) but it could be any complex event object when serialized as JSON object for example.

Below you can see a simplified version of our event hub processor Azure Function:

```csharp
[FunctionName("EventHubProcessorFunction")]
public static async void Run([IoTHubTrigger("ifttt", Connection = "EventHubConnection")]EventData message, Microsoft.Azure.WebJobs.ExecutionContext context,
CancellationToken cancellationToken)
{
    var streamingDataMessage = JsonConvert.DeserializeObject<StreamingDataMessage>(Encoding.UTF8.GetString(message.Body.Array));
    if(streamingDataMessage.MessageType != "itd")
    {
        return;
    }

    var itemDataMessage = JsonConvert.DeserializeObject<ItemDataMessage>(streamingDataMessage.Message);
    if (itemDataMessage.SrcKey != "MotionSensorState")
    {
        return;
    }

    await SendEventToIFTTT(itemDataMessage.Value);
}
```

In similar fashion to our [other Azure Function][article_2] we're simply consuming messages from our event hub and then forward them to IFTTT. The `SendEventToIFTTT` method wraps a simple HTTP request towards the IFTTT Webhook service:

POST `https://maker.ifttt.com/trigger/<name of the event>/with/key/<your accounts webhook service key>`

The Webhook service key can be obtained [here](https://ifttt.com/maker_webhooks) (if you're logged in). The Webhook service is also able to interpret an `application/json` body if it has the structure below:

```json
{
    "value1": "some string",
    "value2": "#MWIGA",
    "value3": "hello world"
}
```

Taking advantage of the body structure one could transmit any kind of event data to IFTTT. Finally we have to configure an applet in IFTTT to test our implementation. We combined [Google Sheets](https://ifttt.com/services/google_sheets) with the Webhook service for example:

![Receiving applet config](assets/receiving-applet-config.png)

## Conclusion

That's it! Two Azure Functions, an EventHub, a Raspberry Pi and three workdays later we were able to present a functioning prototype. For our demo on day four we logged motion sensor data through our tapio-IFTTT-Connector into a Google Drive sheet, turned on a RGB LED with the press of a widget button on a smartphone and configured a new IFTTT-Applet live. We didn't develop a shippable product but built a working proof of concept which can be transformed into a proper solution. Authentication, authorization and a web interface for configuring events to be be forwarded on a per machine basis are mandatory for a feature complete solution. Our code base also lacks a huge refactoring... :D

Aside from resolving the actual challenges [#hackthewood2019](https://www.tapio.one/en/blog/hack-the-wood-2019) was most notably a fun event with awesome attendees who helped each other out at any time and had a great time together in Berlin!

[article_1]: https://www.tapio.one/de/blog/connecting-the-digital-worlds-1-3
[article_2]: https://www.tapio.one/de/blog/connecting-the-digital-worlds-2-3
[article_3]: https://www.tapio.one/de/blog/connecting-the-digital-worlds-3-3
