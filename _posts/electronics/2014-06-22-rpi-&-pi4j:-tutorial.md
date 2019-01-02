---
path: "/posts/2014/06/rpi-pi4j-tutorial-2-gpio-output/"
date: "2014-06-22T06:01:46Z"
title: "RPi & Pi4J: Tutorial 2: GPIO Output"
categories: electronics
excerpt: "In the first installment of this series, we saw how we could quickly set up a Java development envi..."
---

In the [first installment](/2014/05/11/raspberry-pi-and-pi4j-tutorial-1/ "Raspberry Pi and Pi4J: Tutorial 1") of this series, we saw how we could quickly set up a Java development environment on the RPi, and wrote a simple program to demonstrate how easy it is to configure GPIO using Pi4J.

Let us now dive a little deeper into Pi4J, and understand how we can use the GPIO pins as output pins.

In order to demonstrate the Pi4J APIs that control pin output, we will use the circuit in the figure below.

[![](22-1.png)](http://pfactor.io/wp-content/uploads/2014/05/tutorial22-e1501568030268.png)

The circuit is simple - it connects three GPIO pins (`GPIO_01`, `GPIO_04` and `GPIO_06`) to an LED each (LED1 through LED3) through a current limiting resistor (R1 through R3). In our next article, we will discuss the function of the current limiting resistor, and figure out how we can calculate a reasonable value for it.

As you probably know by now, LEDs are diodes that emit light when a current flows through them (hence the term "**L**ight **E**mitting **D**iode"). We drive a current through them by turning the corresponding GPIO pin high.

## The Code

Source code for all the tutorials on this site is available at [BitBucket](https://bitbucket.org/calpilot/rpi-tutorials/overview).

We initialize each of the GPIO pins by making a call such as the one below:

```java

final GpioPinDigitalOutput led1 = gpio.provisionDigitalOutputPin (
RaspiPin.GPIO_01,
"LED1",
PinState.LOW);
```

Notice how the last argument for the call sets the pin state to `LOW`. This ensures that the pins are set to a known state when they are initialized.

### Blocking vs. Non-Blocking Calls

Some of the methods in Pi4J have the ability to be either blocking or non-blocking. What is the difference?

When a method is specified as a "blocking" method, execution of that thread is halted until the method returns. If the method is specified as "non-blocking" (which is the default), the thread continues past that method call even if that method has not returned.

Another way to think about this is that "blocking" methods are synchronous while "non-blocking" methods are asynchronous.

### Pulse

The `pulse` method does what it says - it sends out a pulse on the GPIO pin once. The simplest form of the `pulse` method takes one argument - a time in milliseconds - for which the pulse is active. So, specifying `pulse(1000);` causes the pin to be pulsed for 1 second (1000 ms = 1 s).

```java

led1.pulse(1000);
```

A slightly more complicated form of this method is one which allows you to specify whether the method should be blocking or non-blocking. This is specified by providing a second argument to the method - a boolean value where `true` implies a blocking method whereas a `false` implies a non-blocking method.

```java

led2.pulse(1000, true); // Blocking call
led3.pulse(1000, false); // Non-blocking call
```

### Blink

The `blink` method sends out a periodic stream of pulses to a GPIO pin, raising it high and then low again.

```java

led1.blink(500, 10000); // blinks LED1 once a second
// (500ms on, 500ms off) for a
// total of 10 seconds.
```

### Sample Program

A sample program to demonstrate the API methods mentioned in this tutorial is given below. You can download the source code from [BitBucket](https://bitbucket.org/calpilot/rpi-tutorials/overview).

[java]
package org.calpilot;

/* Required imports for Pi4J */
import com.pi4j.io.gpio.GpioController;
import com.pi4j.io.gpio.GpioFactory;
import com.pi4j.io.gpio.GpioPinDigitalOutput;
import com.pi4j.io.gpio.PinState;
import com.pi4j.io.gpio.RaspiPin;

/**
* Created by calpilot on 5/14/2014.
*/
public class Tutorial2 {
public static void main(String[] args) {

boolean BLOCKING = true;
boolean NONBLOCKING = false;

System.out.println("===== Starting Tutorial 2 =====");

final GpioController gpio = GpioFactory.getInstance();

/*
* Initialize all the pins as outputs, and pull them to a LOW on
* startup.
*/
final GpioPinDigitalOutput led1 = gpio.provisionDigitalOutputPin(
RaspiPin.GPIO_01,
"LED1",
PinState.LOW);

final GpioPinDigitalOutput led2 = gpio.provisionDigitalOutputPin(
RaspiPin.GPIO_04,
"LED2",
PinState.LOW);

final GpioPinDigitalOutput led3 = gpio.provisionDigitalOutputPin(
RaspiPin.GPIO_06,
"LED3",
PinState.LOW);

/*
* Add a shutdown hook so that the application can trap a Ctrl-C and
* handle it gracefully by ensuring that all the LEDs are turned off
* prior to exiting.
*/
Runtime.getRuntime().addShutdownHook(new Thread() {
@Override
public void run() {
System.out.println("nnPROGRAM WAS INTERRUPTED. SHUTTING " +
"DOWN!");
led1.low();
led2.low();
led3.low();
gpio.shutdown();
}
});

System.out.println("Pins initialized. All LEDs should be off.");

/*
* Starting the main loop of our program. We put it in a try-catch
* since the various sleep's throw an exception and we need to catch
* those exceptions.
*/
try {
System.out.println("Blocking Call: Each LED will be on for 1 sec" +
" one after another.");
Thread.sleep(1000);

/*
* Blocking call: Execution proceeds to next line only after current
* line is finished. LEDs will blink one after the other.
*/
led1.pulse(1000, BLOCKING);
led2.pulse(1000, BLOCKING);
led3.pulse(1000, BLOCKING);

System.out.println("Non-Blocking Call: Each LED will be on for 1 " +
"sec, but simultaneously.");
Thread.sleep(1000);

/*
* Non-blocking call: Each LED should turn on and off
* simultaneously. Execution does not wait for a statement to finish
* before moving to next statement.
*/
led1.pulse(1000, NONBLOCKING);
led2.pulse(1000, NONBLOCKING);
led3.pulse(1000, NONBLOCKING);

System.out.println("Blinking LED2 for 10 seconds, " +
"every second.");
Thread.sleep(1000);

/*
* Blink LED2 for a total of 10 seconds, with each cycle being 1 sec
* long. LED2 will be on for 500 ms and off for 500 ms.
*/
led2.blink(500, 10000);

System.out.println("Blinking LED3 forever, every 2 seconds. Press" +
" Ctrl-C to abort the program.");

/*
* Another way to blink a GPIO pin is to set it high and low
* alternately, with a sleep.
*/
for (; ; ) {
led3.high();
System.out.print(".");
Thread.sleep(1000);
led3.low();
System.out.print(" ");
Thread.sleep(1000);
}
} catch (InterruptedException ie) {
System.out.println("Execution Interrupted.");
led1.low();
led2.low();
led3.low();
gpio.shutdown();
}
}
}

[/java]