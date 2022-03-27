+++
author = "Raul Camacho"
date = 2022-03-05T05:00:00Z
draft = true
keywords = ["arcade buttons", "smart home", "mqtt", "home assistant", "iot"]
summary = "Making a smart button box"
title = "I turned a lunchbox into a nightstand light controller"

+++
![](/uploads/52fa0e5f-b6cd-4f58-aecb-5bcec8dba0f2.jpeg)  
I enjoy overcomplicating things, so this project was a joy. I saw a lunchbox for sale at Target a few months ago and immediately put it in my cart. I didn't need it. I've been working from home since the pandemic started. I don't take lunch anywhere. But I like boxes, so I bought it.

## Materials

I'm not going to list all of the materials I used, just the main parts of the box itself.

* [1 x bento box from Target](https://www.target.com/p/bento-box-with-bamboo-lid-wise-green-threshold-8482/-/A-83084112#lnk=sametab)
* 1 x Magsafe iPhone charge
* 1 x Development board with Wifi [(I used this ESP8266-based board)](https://www.adafruit.com/product/2821)
  * 1 x micro USB cable
* 2 x Buttons [(I used these mini arcade buttons, cute and satisfying!)](https://www.adafruit.com/product/3429)
  * 2 x [Arcade button connectors](https://www.adafruit.com/product/1152)
* Breadboard for prototyping, perfboard for more permanent mounting

## Part one: wireless charging

The first part of the project that came to mind was to carve out a circle in the lid big enough to set an iPhone Magsafe charger into, with the hopes of using it as a pretty wireless charger. I decided to take the slow and manual route and carve out this circle with a wood chisel and a whittling knife. After a couple of weeks of carving a little every day (picture me in a rocking chair, watching tv after dinner, whittling away), I made the layer of bamboo at the bottom of the hole thin enough for the magnets in the Magsafe charger work.

Admittedly, there is likely some power efficiency loss, but this is used as an overnight charger, so I'm ok with it.

## Part two: iot arcade buttons

I love experimenting with smart home technologies and automations that try to bring Star Trek dashboards into your home, but nothing beats the tactile feel of a button for me. Even better, arcade buttons! So, I decided that I wanted that button feel on my nightstand, to control the lamps of my partner and I. Is this an overcomplicated way to turn off lamps that have switches on them anyway? Absolutely. Here's a diagram of what I wanted.

![](/uploads/lunchbox-iot.png)

## Breakdown

### 1. Home Assistant (docker)

I am running an instance of the home assistant docker image on a raspberry pi 4 model B. My docker compose configuration for home assistant is:

    version: "2.1"
    services:
      homeassistant:
        container_name: homeassistant
        image: "ghcr.io/home-assistant/home-assistant:stable"
        volumes:
          - /home/pi/hass:/config
          - /etc/letsencrypt:/etc/letsencrypt
          - /home/pi/media:/media
        restart: unless-stopped
        privileged: true
        network_mode: host

With this running, I am able to integrate these [TP-Link smart plugs](https://www.kasasmart.com/us/products/smart-plugs/kasa-smart-wifi-mini-plug-hs103) that each nightstand lamp is plugged into and control them from the Home Assistant app, web app, or lots of other mechanisms.

### 2. Mosquitto (docker)

Mosquitto is a message broker for MQTT. It is developed by the Eclipse Foundation and is a great solution for low overhead messaging. The plan here is to use my ESP8266 development board to publish messages to a MQTT topic that home assistant will listen to for controlling my lights.

An analogy for a message broker I like is that it acts like a TV channel that can be broadcast to, and anyone who happens to be watching will see whatever is broadcast. So we want our smart button to broadcast a message that our Home Assistant will be listening for and can perform actions whenever it is received (toggling our lamp!).

I decided to run an instance of the official [Mosquitto docker image](https://hub.docker.com/_/eclipse-mosquitto) alongside home assistant on my raspberry pi. My docker compose configuration for home assistant is:

    version: "2.1"
    services:
      mosquitto:
        container_name: mosquitto
        image: eclipse-mosquitto:2.0.14
        volumes:
          - /home/pi/mosquitto/mosquitto.conf:/mosquitto/config/mosquitto.conf
        ports:
          - 1883:1883
          - 9001:9001
        restart: unless-stopped
        privileged: true

Given how low overhead mosquitto is, a raspberry pi 4 is easily able to handle both it and Home Assistant at the same time.

In order for Home Assistant to be able to listen for messages in mosquitto, we need the [mqtt Home Assistant integration](https://www.home-assistant.io/integrations/mqtt/). Our use-case is pretty basic, so it was easy to set up and is running without any configuration changes.

### 3. ESP8266 with arcade buttons

The development board I chose works fantastically and Adafruit's documentation is thorough enough to get most people with some software/hardware knowledge on their way to programming and flashing it. In fact, the board is most definitely overkill for my purposes (I only needed 2 IO pins for example, the ESP8266 has lots more).

To bring my smart lunchbox to life, I needed a program that would:

1. Connect to my wifi
2. Connect to my mosquitto docker instance
3. Publish messages to a predefined MQTT topic when a button is pressed

Tutorials for [Adafruit's MQTT client library](https://github.com/adafruit/Adafruit_MQTT_Library) got me #1 and #2, and most of the way to #3! My resulting code is repetitive, straightforward, and by no means clever, but it gets the job done. I'll hopefully refine it in the future with a focus on reusability and DRY-ness.

I picked two digital IO pins on the board and wired them to two arcade buttons. I didn't connect the cool LEDs on the arcade buttons because this is meant for a nightstand, next to where people sleep, and we don't want lights on at night, duh. But the LEDs are very useful if you plan on emulating this project in somewhere like a living room or office.

    <script src="https://gist.github.com/raulcodes/23fcc05275c7161dbe1b0fb531fc3545.js"></script>

With the code and wiring complete, I plugged in the board to power and voila! I had a functioning prototype to work with.