+++
author = "Raul Camacho"
date = 2022-03-05T05:00:00Z
draft = true
keywords = ["arcade buttons", "smart home", "mqtt", "home assistant", "iot"]
summary = "Making a smart button box"
title = "I turned a lunchbox into a nightstand light controller"

+++
![](/uploads/52fa0e5f-b6cd-4f58-aecb-5bcec8dba0f2.jpeg)  
I enjoy overcomplicating things, so this project was a joy. I saw lunchbox at Target a few months ago and immediately put it in my cart. I didn't need it. I've been working from home since the pandemic started. I don't take lunch anywhere. But I like boxes, so I bought it. 

## Part one: wireless charging

The first part of the project that came to mind was to carve out a circle in the lid big enough to fit an iPhone Magsafe charger into, with the hopes of using it as a pretty wireless charger. I decided to take the slow and manual route and carve out this circle with a wood chisel and a whittling knife. After a couple of weeks of carving a little every day, I made the layer of bamboo at the bottom of the hole thin enough for the magnets in the Magsafe charger work.

Admittedly, there is likely some power efficiency loss, but this is used as an overnight charger, so I'm ok with it.

## Part two: iot arcade buttons

I love experimenting with smart home technologies and automations that try to bring Star Trek dashboards into your home, but nothing beats the tactile feel of a button for me. Even better, arcade buttons! So, I bought these [mini arcade buttons](https://www.adafruit.com/product/3429) from Adafruit along with a [Feather HUZZAH with ESP8266](https://www.adafruit.com/product/2821) to be the brains of this whole operation.

## Breakdown

### 1. Home Assistant

I am running an instance of the home assistant docker image on a raspberry pi 4 model B. My docker compose configuration for home assistant is: \`\`\`

### 2. MosQuiTTo

Mosquitto is a message broker for MQTT. It is developed by the Eclipse Foundation and is a great solution for low overhead messaging. The plan here is to use my ESP8266 development board to publish messages to a MQTT topic that home assistant will listen to for controlling my lights.

I decided to run an instance of the official [https://hub.docker.com/_/eclipse-mosquitto](https://hub.docker.com/_/eclipse-mosquitto "Mosquitto docker image") alongside home assistant on my raspberry pi. My docker compose configuration for home assistant is: \`\`\`

On the home assistant side, the [https://www.home-assistant.io/integrations/mqtt/](https://www.home-assistant.io/integrations/mqtt/ "MQTT integration") was easy to set up.

### 3. ESP8266 diagramming

I bought an ESP8266 development chip and arcade buttons from Adafruit, which also provides great resources

### 4. Programming