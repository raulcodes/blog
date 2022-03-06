+++
author = "Raul Camacho"
date = 2022-03-05T05:00:00Z
draft = true
keywords = ["arcade buttons", "smart home", "mqtt", "home assistant", "iot"]
summary = "Making a smart button box"
title = "The iot Lunchbox"

+++
My experience with Home Assistant has been frustrating and satisfying. Frustrating because under its surface there is a steep learning curve. Getting things like SSL working with the docker image installation involved lots of scrolling through forums, hoping I find a solution. On the other hand, Home Assistant is a fantastically huge repository of learning potential. Its focus on being composable and flexible has created a community that develops integrations and features for almost anything I've imagined so far. 

## Breakdown

### 1. Home Assistant

I am running an instance of the home assistant docker image on a raspberry pi 4 model B. My docker compose configuration for home assistant is: \`\`\`

### 2. MosQuiTTo

Mosquitto is a message broker for MQTT. It is developed by the Eclipse Foundation and is a great solution for low overhead messaging. The plan here is to use my ESP8266 development board to publish messages to a MQTT topic that home assistant will listen to for controlling my lights. 

I decided to run an instance of the official [https://hub.docker.com/_/eclipse-mosquitto](https://hub.docker.com/_/eclipse-mosquitto "Mosquitto docker image") alongside home assistant on my raspberry pi. My docker compose configuration for home assistant is: \`\`\`

On the home assistant side, the [https://www.home-assistant.io/integrations/mqtt/](https://www.home-assistant.io/integrations/mqtt/ "MQTT integration") was easy to set up.

### 3. ESP8266 diagramming

### 4. Programming