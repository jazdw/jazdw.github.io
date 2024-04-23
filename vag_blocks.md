---
title: VAG Blocks
permalink: /vag-blocks
---

VAG Blocks is open source (GPLv3) beta software for reading VAG group specific measuring blocks from an OBD-II port
using an inexpensive [ELM327](http://en.wikipedia.org/wiki/ELM327) type adapter. Both Bluetooth and USB adapters have
been tested on Windows & Linux. Sample
rates of up to 12 samples per second have been achieved.

Various control modules are supported such as the engine ECU and gearbox. Values such as boost, oil temperature, DPF
soot loading and current gear can be plotted in program and logged to a CSV file.

You will need a label file for each module to determine what the values mean. Without a label file VAG Blocks will only
tell you the units of the parameter eg. You might see a value of 80 °C but without a label file you cannot determine if
it is the oil temperature or the coolant temperature. Plain text VCDS/VAG-COM style label files and redirect files are
supported by VAG Blocks.

Currently the communication protocol is unstable and the software is still in beta phase. The ELM327 is designed for
reading OBD-II PIDs and its raw CAN mode is limited which makes implementing VW TP 2.0 difficult.

## References

* [blafusel.de KW 1281 Protocol (German)](http://www.blafusel.de/obd/obd2_kw1281.html)

## Download

* [Windows binary - Version 1.0.0 beta](https://github.com/jazdw/jazdw.github.io/releases/download/v1.0.0/vagblocks-1.0.0_beta-Windows.zip)
  * Released 15 January 2012
  * ~3.0MB

## Source code

[https://github.com/jazdw/vag-blocks](https://github.com/jazdw/vag-blocks)
