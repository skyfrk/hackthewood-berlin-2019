### Building the connector

As there were two routes for events we focused on one at a time. First we took a look at the path events take when they're coming from IFTTT:

1. An IFTTT-Applet was triggered which sent a HTTP request through the Webhook action to our connector.
2. The connector forwards the event via the tapio commanding API to the corresponding CloudConnector instance.
3. CloudConnector received the commanding request and forwarded it as a call to the preconfigured OPC UA server.
4. The OPC UA server processed the event.

The tapio commanding API is normally used to alter items or call methods on a OPC UA server associated with the CloudConnector but we figured we can use a item write request to transmit an event. On OPC UA server side we then just had to wait for item state changes and interpret them as events. In detail we used an `DataVariableState` of type `String` and used the value to transmit a serialized JSON object, which contained metadata about the event.
When we noticed that our connector only had to listen for a http request and then make another http request we opted for a [serverless](https://martinfowler.com/articles/serverless.html) approach using an [Azure Function](https://docs.microsoft.com/en-us/azure/azure-functions/) to save time (A Azure Function basically is a piece of code which gets executed when a certain condition arises).

For testing our function the command-line program [ngrok](https://ngrok.com/) came in handy. Through ngrok you can expose a local development server to the internet. This way you can debug webhooks directly on your local machine without exposing ports or renting a webserver.

