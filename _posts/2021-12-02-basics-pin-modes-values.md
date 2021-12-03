---
title: 'Basics: pin modes and values'
date: 2021-12-02
tags: arduino
src-author: Scott Fitzgerald and Michael Shiloh
src-title: 'The Arduino Projects Book Chapter 2: Spaceship Interface'
src-url: 
---

## Contents
{:.no_toc}

* TOC
{:toc}



## Install `arduino-cli`, run script from command line

<https://arduino.github.io/arduino-cli/0.20/getting-started/>

```sh
$ brew install arduino-cli
$ arduino-cli core update-index
$ arduino-cli board list
Port                            Protocol Type              Board Name  FQBN            Core
/dev/cu.usbmodem143301          serial   Serial Port (USB) Arduino Uno arduino:avr:uno arduino:avr
$ arduino-cli core install arduino:avr
$ arduino-cli core list
ID          Installed Latest Name
arduino:avr 1.8.4     1.8.4  Arduino AVR Boards
$ arduino-cli sketch new project_first
$ arduino-cli compile --fqbn arduino:avr:uno project_first
$ arduino-cli upload -p /dev/cu.usbmodem143301 --fqbn arduino:avr:uno project_first
```

Good to know how to do this, but henceforth I'll use Microsoft's
Arduino extension for Visual Studio Code.



## Arduino board pins and electronic components connected to them

Assign board pins input/output roles. You will set each pin to
`INPUT` or `OUTPUT` in `setup()`, below.

Pin for pushbutton:

```cpp
int inputPins[] = {2};
```

Pins for LEDs:

```cpp
int outputPins[] = {LED_BUILTIN, 3, 4, 5};
```

After connecting breadboard pushbutton and LEDs to Arduino board pins, 
identify the pins here.

```cpp
int pushButton = 2;
int greenLED = 3;
int redLED1 = 4;
int redLED2 = 5;
```

Possible states of an output pin.

```cpp
int ledStates[] = {HIGH, LOW};
```

Set starting state of breadboard pushbutton.

```cpp
int pushButtonState = LOW;
```



## `setup()` and `loop()`

Are run when the program loads.

```cpp
void setup() {

    // Set pin modes.
    for (int pin : inputPins) pinMode(pin, INPUT);
    for (int pin : outputPins) pinMode(pin, OUTPUT);
    
    // You may want to set the initial state of a component. Here, turn
    // the green breadboard LED on.
    digitalWrite(greenLED, HIGH);
}


void loop() {

    // Get pushbutton state.
    pushButtonState = digitalRead(pushButton);

    // Respond to changes in a pin value.
    // When button is down...
    if (pushButtonState == HIGH) {

        // ...turn green LED off...
        digitalWrite(greenLED, LOW);

        // ... and turn on red LEDs.
        digitalWrite(redLED1, HIGH);
        digitalWrite(redLED2, HIGH);

    } else {

        // When button is up, turn green LED on...
        digitalWrite(greenLED, HIGH);

        // ... and turn off red LEDs.
        digitalWrite(redLED1, LOW);
        digitalWrite(redLED2, LOW);
        }
}
```
