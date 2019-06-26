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
