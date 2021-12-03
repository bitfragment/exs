---
title: Pushbutton device
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


## Arduino board pins and electronic components connected to them

Assign board pins input/output roles.

```cpp
int inputPins[] = {2};
int outputPins[] = {LED_BUILTIN, 3, 4, 5};
```

After connecting breadboard pushbutton and LEDs to Arduino board pins, 
identify the pins here.

```cpp
int pushButton = 2;
int greenLED = 3;
int redLEDs[] = {4, 5};
```

Put breadboard LED pin states in an array for convenience.

```cpp
int ledStates[] = {HIGH, LOW};
```

Set starting state of breadboard pushbutton.

```cpp
int pushButtonState = LOW;
```


## Helper functions

```cpp
void pulseLED(int pin, int delayTime) {
    for (int state : ledStates) {
        digitalWrite(pin, state);
        delay(delayTime);
    }
}
```

```cpp
void multiPulseLED(int pin, int pulses, int delayTime) {
    for (int i = 0; i < pulses; i++) {
        pulseLED(pin, delayTime);
        delay(delayTime);
    }
}
```


On button down, green LED off and alternately pulse red LEDS.
On button up, green LED on and red LEDs off.

```cpp
void handlepushButtonState() {
    pushButtonState = digitalRead(pushButton);
    if (pushButtonState == HIGH) {     // button down
        digitalWrite(greenLED, LOW);
        for (int led : redLEDs) {
            multiPulseLED(led, 2, 25); // fast double pulse
        }
    } else {                           // button up
        digitalWrite(greenLED, HIGH);
        for (int led : redLEDs) {
            digitalWrite(led, LOW);
        }
    }
}
```



## `setup()` and `loop()`

```cpp
void setup() {
    for (int pin : inputPins) pinMode(pin, INPUT);
    for (int pin : outputPins) pinMode(pin, OUTPUT);
    digitalWrite(greenLED, HIGH); // green LED on
}

void loop() {
    handlepushButtonState();
}
```
