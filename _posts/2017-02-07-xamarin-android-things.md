---
layout: post
title: Xamarin.Android - Things
---

# All the Android Things - Weather Station

When Android Things was first announced, my inner inventor came out. I wanted to start prototyping all of these ideas that have been spinning around my head. I wanted to be the next *Red Green*.

![](http://indianapublicmedia.org/about/files/2010/08/red_green.jpg)

(If you aren't familiar. He was the main character of *[The Red Green Show](https://www.youtube.com/user/RedGreenTV/videos?flow=grid&view=0&sort=p)*, which was a popular Canadian parody of a typical DIY/home improvement show. In the show, they would have some zany premise of automating things that are often painful like mowing your lawn, rolling down manual car windows, and serving beverages).

## Android Things

Android Things provides the same Android development tools that we know and love to work with embedded devices. You can see a quick overview of this here:

![](https://developer.android.com/things/images/platform-architecture.png)

There are two main components of the `Things Support Library` that we need to know before starting a prototype.

### Peripheral I/O API

The Peripheral I/O APIs let your apps communicate with sensors and actuators using industry standard protocols and interfaces. The following interfaces are supported: GPIO, PWM, I2C, SPI, UART.

### User Driver API

User drivers extend existing Android framework services and allow apps to inject hardware events into the framework that other apps can access using the standard Android APIs.

## What you'll need

- Hardware ([https://developer.android.com/things/hardware/developer-kits.html](https://developer.android.com/things/hardware/developer-kits.html))
- HDMI cable
- HDMI-enabled display
- Micro-USB cable
- Ethernet cable
- 8GB+ SD card
- SD card reader

**Optional:** You can also pick up a [Pimoroni Rainbow HAT](https://shop.pimoroni.com/products/rainbow-hat-for-android-things) for a plug-and-play buffet of inputs, sensors, etc to explore Android Things.

## Getting Started

Once you've gathered your materials, you'll need to flash your device. After you've flashed your device, you will need to setup a network connection. You can follow the various instructions for doing this here:

- [Raspberry Pi 3](https://developer.android.com/things/hardware/raspberrypi.html)
- [Intel Edison](https://developer.android.com/things/hardware/edison.html)
- [NXP Pico i.MX6UL](https://developer.android.com/things/hardware/pico.html)

## Set Up Your Development Environment

After you've flashed your device and have it connected to your network, we want to ensure it's available via `adb`. You can use the following command to ensure this:

`adb devices`

When you bootup the Android Things device, you can either have it connected to an ethernet cord, or to your wireless network. Either way, you will want to use `adb connect <ip address>` to connect to the device.
 
## Android Weather Station Sample

This sample is an integration of multiple peripheral sensors to analyze and display current weather information.

You can find the source code of this sample here:

[https://github.com/JonDouglas/XamarinComponents/tree/master/Android/AndroidThingsContribDrivers/samples/WeatherStation](https://github.com/JonDouglas/XamarinComponents/tree/master/Android/AndroidThingsContribDrivers/samples/WeatherStation)

## Sample

### Monitor

Displays the current weather situation. Sunny, Cloudy, or Rainy.

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/0e8189cb-6a0b-4e92-a0f0-2fd49757566e/02.07.2017-16.32.jpg)

### Raspberry Pi 3

Displays the current realtime temperature in Celsius. Here's a video of it in action:

[https://www.screencast.com/t/i3vaGchmH7](https://www.screencast.com/t/i3vaGchmH7)

## Summary

Android Things is really fun to tinker around with. Let your inner Red Green or Tim "The Tool Man" Taylor shine! 
