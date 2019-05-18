---
title:
- tapio goes IFTTT
author:
- Simon Eßlinger
date:
- 20. May 2019
theme:
- Copenhagen
classoption: "aspectratio=169"
---

## The Idea

The idea is simple. We want to connect the tapio ecosystem with [IFTTT](https://ifttt.com/).

### IFTTT

IFTTT or "**IF T**his **T**hen **T**hat" is an IoT (Internet of Things) automation platform. As the name suggests it's all about very simple connections like these:

* **IF** I like a music video on YouTube **THEN** add it to my Spotify library.
* **IF** my vacuum cleaner is stuck somewhere **THEN** send me a push notification.
* **IF** my CNC machine set off an alarm **THEN** safe the last 5 minutes of the video feed of my webcam which was monitoring the machine.

> You're always limited to combining **one** trigger with **one** action.

#### IFTTT Services

#### IFTTT Trigger

#### IFTTT Actions

## The Implementation Plan

### Building a demo machine

* must run a opc ua server
* cc should run
* must have input we can trigger
* must have action we can trigger

The first idea that comes to mind is to use a [Raspberry Pi](https://www.raspberrypi.org/). It's cheap, reliable and setup fast.

* install raspbian
* setup ssh
* install dotnet
* setup remote debugging
* setup cc
* setup motion sensor
* setup rgb led

### Building the connector

* uses webhooks
* must listen for events
  * from IFTTT
  * from tapio eventhub
  * or simple stateAPI polling
* must be able to send events to IFTTT
* must be able to send "events" (stateAPI calls) to tapio
* ngrok as tool
* auth?