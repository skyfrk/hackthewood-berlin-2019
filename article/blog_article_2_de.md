### Der Connector

Von den zwei Routen für Ereignisse haben wir uns zuerst auf die Route von IFTTT zur Maschine fokussiert:

1. Ein IFTTT-Applet wird ausgelöst, welches dann über den Webhook IFTTT-Service den Connector aufruft.
2. Der Connector leitet das Ereignis über die tapio Commanding-API an die entsprechende CloudConnector Instanz weiter.
3. Der CloudConnector leitet das Ereignis an den vorkonfigurierten OPC UA Server weiter.
4. Der speziell angepasste OPC UA Server verarbeitet das Ereignis.


Die tapio Commanding-API wird normalerweise dazu verwendet, Werte von Knoten auf einem OPC UA Server zu ändern oder OPC UA Methoden aufzurufen. Wir nutzen die Eigenschaft der Commanding-API Werte von Knoten zu ändern um immer dann einen Wert zu ändern, wenn ein Ereignis übertragen werden sollte. Serverseitig mussten wir dann lediglich den Zustand eines Knoten überwachen und bei Änderung die übertragenen Daten als Ereignisdaten interpretieren. Im Detail benutzen wir einen `DataVariableState` vom Typ `String` und benutzen den Wert des Knoten um ein serialisiertes JSON Object zu übertragen, welches Metadaten zu dem Ereignis mit sich führte. Als wir feststellten, dass der Connector lediglich auf eine HTTP-Anfrage wartet und selbst eine HTTP-Anfrage stellt, entschieden wir uns für einen [Server-losen](https://martinfowler.com/articles/serverless.html) Ansatz mit einer [Azure Function](https://docs.microsoft.com/en-us/azure/azure-functions/), um Zeit zu sparen (Eine Azure Function ist im Grunde genommen ein Stück Code, welcher ausgeführt wird, wenn eine bestimmte Bedingung eintritt).

Zum Testen unserer Azure Function kam uns das Befehlszeilenprogramm [ngrok](https://ngrok.com/) gelegen. Mit ngrok kann ein lokaler Entwicklungsserver dem Internet zugänglich gemacht werden. Auf diese Weise können Webhooks debuggt werden, ohne manuell Ports freizuschalten oder einen extra Webserver zu mieten.
