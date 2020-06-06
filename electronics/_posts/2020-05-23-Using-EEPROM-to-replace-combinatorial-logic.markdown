---
layout: post
title:  "Using EEPROM to replace combinatorial logic"
date:   2020-05-23 16:30:55 +0800
categories: electronics
mathjax: true
---

TODO : Create a React based 'Boolean to ROM conversion application' application

Complete parts list:
- 1x 28C16 EEPROM
- 8x LEDs
- 8x 330Ω resistors
- 1x 7-position DIP switch
- 1x 7-position DIP switch
- 12x 10kΩ resistors
- 1x 100nF capacitor
- 1x 680Ω resistor
- 1x momentary tact switch
- 1x 100Ω resistor

We'll wire up an EEPROM (28C16) so we can read its contents. We'll also take a look at the data sheet to learn how to program it, and try programming some values. Finally, we'll see how the EEPROM can be used to replace any combinational logic circuit such as the 7-segment decoder 