---
path: "/posts/2014/05/rpi-pins-pin-numbering/"
date: "2014-05-17T15:59:51Z"
title: "RPi Pins & Pin Numbering"
categories: electronics
excerpt: "In the first installment of this series we wrote a simple Java program that used the Pi4J library t..."
---

<!--Test sentence to see if this fixes the damn link issue. Another test sentence. Another test sentence.-->
  
In the [first installment](/2014/05/11/raspberry-pi-and-pi4j-tutorial-1/) of this series we wrote a simple Java program that used the Pi4J library to set one of the RPi's GPIO ports low and reads its status. Before we go any further with our tutorials, let us take a quick look at what GPIO is, and what it offers.

Pi4J provides a class `RaspiPin` to give you an abstraction of the GPIO pins on the board. We used the following snippet of code to set the pin low:

```

GpioPinDigitalInput myButton = gpio.provisionDigitalInputPin(RaspiPin.GPIO_00, 
                                            "MyButton",
                                            PinPullResistance.PULL_DOWN);
```

Notice how we used `RaspiPin.GPIO_00` to refer to the pin that we wanted to set as an input pin.

## What is GPIO?

GPIO stands for "General Purpose Input Output" and, as the name suggests, it refers to a set of pins that provide generic input and output capabilities for the RPi. While most of the pins are truly "general purpose" pins, a few pins can be used for specific purposes (such as I2C communication, Serial communication, etc.) Obviously, when those pins are being used for their specific functions, they cannot be used as general purpose pins.

## Pin Numbering

Pin numbering has been the subject of much debate and angst among the Raspberry Pi community for a while now. Since I'm not exactly a fan of angst and/or controversy, let us stay out of the debate and just state that **we will use the WiringPi numbering scheme** from here on out.

Pi4J is based on a project called [WiringPi](http://www.wiringpi.com/), which is a set of native libraries that provide access to the RPi's GPIO ports. The pin numbers and their mapping to the actual pins on the RPi header are given in the picture below.

![RPi Pin Out](https://i2.wp.com/pi4j.com/images/p1header.png?w=1280)

The numbers in bold will be used as our GPIO pin numbers. In Pi4J, the pin numbers are prefixed by `GPIO_` and are zero-padded. So, the pins are referred to as `GPIO_00`, `GPIO_01`, and so on through `GPIO_20`.

In all circuit diagrams going forward, I will be using the image below to represent the RPi header and the pin out. The pin numbers on the outside of the diagram are the _physical_ pin numbers on the header while the textual descriptions within the rectangle are those of the corresponding Pi4J equivalents.

![RPi Header Circuit Schematic](http://calpilot.files.wordpress.com/2014/05/picobbler_wiringpi.png?w=1280)