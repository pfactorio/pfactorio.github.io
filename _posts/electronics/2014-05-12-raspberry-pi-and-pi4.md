---
path: "/posts/2014/05/raspberry-pi-and-pi4j-tutorial-1/"
date: "2014-05-12T05:18:00Z"
title: "Raspberry Pi and Pi4J Tutorial 1"
categories: electronics
excerpt: "This is part 1 of a multi-part series on programming the Raspberry Pi with..."
---

This is part 1 of a multi-part series on programming the (abbreviated to "RPi") with Java using the Pi4J programming library. One of the things I love about the RPi is the fact that it can be a full-fledged Linux-based computer with the ability to function as a development system as well. If you add a JDK to the mix, you can actually use the RPi as a Java development environment.


## GOALS

What I wanted to achieve with my RPi was the following:

* Set up Java SE.
* Install Maven
* Use the Pi4J libraries to interact with "low-level" features such as GPIO, I2C, etc.

Let's look into how I set up each of these.

## SET UP JAVA SE

Oracle has been working very closely with the RPi community to make builds of Java SE available on the RPi. The end result is that the JDK is pre-installed with the stock Raspbian distribution. Updating to newer versions of Java SE is as simple as using `apt-get` on the RPi to update to the newer version.

```

$ sudo apt-get update
$ sudo apt-get install oracle-java8-jdk
```

### Make Java 8 the default

Run the `update-alternatives` command to bring up a list of installed Java versions, and to select the default.

<pre>$ sudo update-alternatives --config java
There are 3 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                                 Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-6-openjdk-armhf/jre/bin/java        1057      auto mode
  1            /usr/lib/jvm/java-6-openjdk-armhf/jre/bin/java        1057      manual mode
  2            /usr/lib/jvm/jdk-7-oracle-armhf/jre/bin/java          317       manual mode
* 3            /usr/lib/jvm/jdk-8-oracle-arm-vfp-hflt/jre/bin/java   318       manual mode

Press enter to keep the current choice[*], or type selection number:
</pre>

Type "3" and hit enter to make Java 8 the default.

## INSTALL MAVEN

For a long time, I resisted the move to Maven. I was uncomfortable with setting up the `pom.xml` file. I could never understand how things worked, etc.

To make a long story short, I recently solicited the help of a friend and actually set up a simple development environment using Maven. He helped me create my first `pom.xml`. Once that was done, I was astounded by the simplicity of the tool and its abilities. The dependency management, by itself, blew my mind away. Let's just say I will not be using Ant any more.

Installing Maven on the RPi is as simple as running `apt-get` again.

```

$ sudo apt-get install maven
```

## USE PI4J

The [Pi4J](http://www.pi4j.com/) library is a Java wrapper around a native library for the RPi. It provides a set of Java classes to abstract access to the RPi's ports, specifically the GPIO, SPI and I2C ports. The easiest way to use Pi4J in an application is to include the repository into the project's `pom.xml` file.

```

<dependencies>
    <dependency>
        <groupId>com.pi4j</groupId>
        <artifactId>pi4j-core</artifactId>
        <version>0.0.5</version>
    </dependency>
</dependencies>
```

## PUTTING IT ALL TOGETHER

Here are the steps we will take to create our first Java / Pi4J application on the RPi.

* Set up the project layout.
* Create the `pom.xml`.
* Create the Java source file(s).
* Build and run the application.

Let us go through each of these steps in detail.

### Set up the project layout

All Maven projects follow a standard directory layout as summarized below. More details can be obtained from the Maven [documentation](http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html).

**Remember to replace `PROJECT_HOME` with the name of your project.**

```

PROJECT_HOME
|
+-- src
      |
      +-- main
      |      |
      |      +-- java
      |      |
      |      +-- resources
      |
      +-- test
             |
             +-- java
             |
             +-- resources
```

So, set up this standard layout in your project's home.

```

$ mkdir ~/PROJECT_HOME
$ mkdir -p ~/PROJECT_HOME/src/main/java
$ mkdir -p ~/PROJECT_HOME/src/resources
$ mkdir -p ~/PROJECT_HOME/test/java
$ mkdir -p ~/PROJECT_HOME/test/resources
```

Obviously, you will create the appropriate "Java-specific" directory structure withing the `src/main/java` directory. For example, if your code were to be a part of the `org.calpilot` package, you would create an `org/calpilot` directory under `src/main/java`.

### Create the POM

I, personally, like to use the Maven assembly plugin in my build to build what is called a `jar-with-dependencies`, i.e. a jar file with all the dependencies baked into it. This way, you just copy the jar file where you want to and execute the `main` class with a `java -jar JAR_FILE` command.

Notice how we mark the Pi4J library as a dependency. We exclude the `pi4j-native` POM because it includes the `.so` files for the native library and that causes issues with the build targets.

Also, note how we specify the class name of the "`main`" class in the `pom.xml`.

{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.calpilot</groupId>
    <artifactId>Pi4jTest</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>Testing Pi4J and Maven</name>
    <url>http://maven.apache.org</url>

    <properties>
        <pi4j.version>0.0.5</pi4j.version>
        <compiler-plugin.version>3.1</compiler-plugin.version>
        <assembly-plugin.version>2.4</assembly-plugin.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.pi4j</groupId>
            <artifactId>pi4j-core</artifactId>
            <version>${pi4j.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>com.pi4j</groupId>
                    <artifactId>pi4j-native</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${compiler-plugin.version}</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>${assembly-plugin.version}</version>
                <executions>
                    <execution>
                        <id>jar-with-dependencies</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>
                            <descriptorRefs>
                                <descriptorRef>
                                    jar-with-dependencies
                                </descriptorRef>
                            </descriptorRefs>
                            <archive>
                                <manifest>
                                    <mainClass>
                                        org.calpilot.Pi4jTest
                                    </mainClass>
                                </manifest>
                            </archive>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
{% endhighlight %}

### Create the Java Source File(s)

For our first example, we will

* Initialize the Pi4J library.
* Set one of the GPIO pins to a "low" state using its internal pull-down resistor.
* Read, and print, the status of that pin.

It's very simple, but all we want to do at this stage is to make sure that we can build our project and that we can load the Pi4J library in our code.

Here's the Java source file.

{% highlight java linenos %}
package org.calpilot;

/* Pi4J imports */
import com.pi4j.io.gpio.GpioController;
import com.pi4j.io.gpio.GpioFactory;
import com.pi4j.io.gpio.GpioPinDigitalInput;
import com.pi4j.io.gpio.PinPullResistance;
import com.pi4j.io.gpio.PinState;
import com.pi4j.io.gpio.RaspiPin;

public class Pi4jTest {

    public static void main(String[] args) {
        System.out.println("Hello, Raspberry Pi!");

        /* Initialize pi4j */
        final GpioController gpio = GpioFactory.getInstance();

        /* Initialize GPIO 0 as an input pin called "MyButton" and set
           it low using the pull-down resistor.
        */
        GpioPinDigitalInput myButton =
            gpio.provisionDigitalInputPin(RaspiPin.GPIO_00,
                                           "MyButton",
                                            PinPullResistance.PULL_DOWN);

        /* Read the state (high or low) of the pin. Remember, it should be &quot;low&quot; */
        PinState pinState = myButton.getState();
        System.out.println("GPIO 0 is set to " + pinState.getName());

        /* Close all open connections. */
        gpio.shutdown();
    }
}
{% endhighlight %}

### Build and Run the Application

```

$ mvn compile package
```

This compiles the project and creates the jar with dependencies in a `PROJECT_HOME/target` directory. The jar file is named `Pi4jTest-1.0-SNAPSHOT-jar-with-dependencies.jar`.

To run the application, just type

```

$ sudo java -jar Pi4jTest-1.0-SNAPSHOT-jar-with-dependencies.jar
```

at the command prompt. _**Remember that you need to run the code as `root` (or a superuser) since access to GPIO ports is a superuser privilege.**_

You should see the following output:

<pre>Hello, Raspberry Pi!
GPIO 0 is set to LOW
</pre>

## THAT'S ALL, FOLKS!

This brings us to the end of this first tutorial. I hope you have as much fun as I did in setting up my first RPi/Java/Pi4J application. In subsequent tutorials, we will start looking at interfacing real hardware to the RPi, and controlling that hardware. Keep your soldering irons handy!