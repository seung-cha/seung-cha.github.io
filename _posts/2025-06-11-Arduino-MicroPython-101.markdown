---
layout: post
title: "Arduino MicroPython 101"
date: 2025-06-07 15:00 +1100
categories: Arduino & MicroPython
---

COMP6733 looks like a course for 2nd year engineering students yet I'm doing it in my 4th year as an 'advanced' course. 
Nice. This whole 5 'advanced' courses requirement for my degree is stupid but for this course I won't complain.
From the first impression, this course looks genuinely fun and is worth spending tuition fees for.

# Arduino
We are given [Nano 33 BLE Sense Rev2](https://docs.arduino.cc/hardware/nano-33-ble-sense-rev2/#features) to work with throughout the term. Nice.
This thing isn't that expensive but having mentors supervising us is always nice especially when the device is fragile.


# Pins
Pins are the I/O of Arduino devices. They receive and send voltages.

Input pins with nothing connected to the other end are `floating` (or in high-impedance state). This means their reading is undefined due to noise. 
Hence, input pins need to be `pulled up` or `pulled down`. It just means setting their idle value as high or low by adding a resistor.

Pins can be digital or analogue, and on my Arduino device they are represented as `D2 ~ D12` and `A0 ~ A7`.

There are other unique pins. For example:

* Serial pins (TX1, RX0): Used to transmit and receive data, one byte at a time. The first letter essentially tells if it's for `transmit` or `receive`, but on the board you can distinguish them by the arrow too.
* Reset pin (RST): self-explanatory. Pull it to low to trigger reset.
* Power pin (VIN): Receive power from external device to power the arduino or send power to power other devices.
* Ground pin (GND)

In [MicroPython](https://docs.micropython.org/en/latest/library/machine.Pin.html), the `Pin` class can be imported from `machine`.

```python
from machine import Pin
```

The class constructor:
```python
machine.Pin(id, mode=-1, pull=-1, *, value=None, drive=0, alt=-1)
```

where `id` is either `int` (internal Pin number), `str` (Pin name) or tuple of [port, pin].
All other arguments, if not provided, will use information from the previous state.

* `mode=` `Pin.IN`, `Pin.OUT`, `Pin.ANALOG` (for analogue pins)
* `pull=` `Pin.PULL_UP`, `Pin.PULL_DOWN`

Read or write to a pin using .value:
`Pin.value(x?)` where if not specified, reads 0 or 1 and if specified and evaluates to bool, outputs 1 on True and 0 otherwise.

`Pin.on()`, `Pin.off()` = `Pin.high()`, `Pin.low()` = `Pin.value(True)`, `Pin.value(False)`.
* `Pin.toggle()` self-explanatory.

Functions to set pin properties:
* `Pin.mode(mode?)`
* `Pin.pull(pull?)`  
... etc


