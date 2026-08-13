---
layout: page
title: STM32 Closed-Loop Temperature Controller
description: Custom heater-control board with dual temperature sensing, PWM power control, and gain-scheduled PID.
img: assets/img/projects/temperature-control/system-overview.png
importance: 2
category: work
_styles: |
  article {
    font-size: 1.1rem;
    line-height: 1.75;
  }

  article blockquote {
    font-size: inherit;
  }
---

**June 2025 · Two-person course project · Electronic Circuit Systems Laboratory, Zhejiang University**

I built a closed-loop temperature-control system for an aluminum plate heated by a high-power resistor. An **STM32F103C8T6** reads a contact NTC thermistor and a GY906 infrared sensor, drives a 12 V heater through PWM, and provides both local and Bluetooth control.

**My role:** system schematic design, PCB soldering and electrical bring-up, PID implementation and tuning, hardware–software integration, and project documentation.

**Source code:** [STM32 firmware and CubeMX project](https://github.com/ch-yu02/My-ISEE-Course-Reference/tree/main/%E7%9F%AD%E5%AD%A6%E6%9C%9F-%E7%94%B5%E5%AD%90%E7%94%B5%E8%B7%AF%E7%B3%BB%E7%BB%9F%E7%BB%BC%E5%90%88%E5%AE%9E%E9%AA%8C/%E6%9C%80%E7%BB%88%E7%89%88/20250627)

<div style="width: 54%; margin-inline: auto;">
  {% include figure.liquid loading="eager" path="assets/img/projects/temperature-control/system-working.jpg" title="Working temperature-control system" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Final heating test.
</div>

## System architecture

<div style="width: 72%; margin-inline: auto;">
  {% include figure.liquid loading="eager" path="assets/img/projects/temperature-control/system-overview.png" title="Temperature-control system overview" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Closed-loop system architecture.
</div>

The system is organized around four functional paths:

- **Sensing:** a 10 kΩ NTC thermistor is sampled through the STM32 ADC, while a GY906 infrared sensor provides a second temperature reading over I²C.
- **Control:** the STM32 computes the heater command from the measured temperature and target setpoint.
- **Actuation:** a MOSFET power stage switches the 12 V resistive heater using PWM.
- **Interface and safety:** an OLED and four buttons provide local control, a UART Bluetooth module accepts remote commands, and a buzzer provides an over-temperature alarm.

> **Control loop:** Sense → PID → PWM → MOSFET → Heater → Temperature

<div style="width: 72%; margin-inline: auto;">
  {% include figure.liquid path="assets/img/projects/temperature-control/circuit-diagram.jpg" title="Temperature-control schematic" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  System schematic.
</div>

## Hardware implementation

I designed the system-level schematic, soldered the assembled board, and brought it up electrically by checking connectivity, supply rails, sensor interfaces, and PWM output. The PCB layout itself was completed by my teammate.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/temperature-control/pcb-design-3d.jpg" title="PCB design preview" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/temperature-control/hardware_connections.jpg" title="Assembled controller" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="caption">
  PCB design and assembled hardware.
</div>

## Control implementation

I implemented a gain-scheduled PID controller for the heater loop. The gains vary with the magnitude of the temperature error, integration is enabled only within a limited error window, the derivative term is smoothed with a moving average, and the final control command is clamped to the valid PWM range.

During tuning, I also compared PID control with staged heating and hysteresis-based control. Hardware tests exposed temperature-sampling jitter and response delay, which led me to revise the filtering and control timing.

## Results

Bench measurements showed a regulated 5 V rail of approximately **5.04 V**, peak heater power of approximately **13 W**, and idle system power of approximately **0.7 W**. System tests covered temperature acquisition, PWM heater control, button input, OLED updates, over-temperature alarms, and Bluetooth commands.

One hardware issue remained in the final prototype: the buzzer's series resistance was too large, which reduced the alarm volume.

_This was a two-person project. My teammate designed the PCB layout and implemented the sensor, OLED, button, alarm, and Bluetooth functions; the work described in “My role” above is my individual contribution._
