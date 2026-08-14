---
layout: page
title: STM32 Closed-Loop Temperature Controller
description: Custom heater-control board with dual temperature sensing, PWM power control, and gain-scheduled PID.
img: assets/img/projects/temperature-control/system-overview.png
importance: 2
category: work
---

**June 2025 · Two-person course project · Electronic Circuit Systems Laboratory, Zhejiang University**

I built a closed-loop temperature-control system for an aluminum plate heated by a high-power resistor. An **STM32F103C8T6** reads a contact **10 kΩ B3950 NTC thermistor** and a GY906 infrared sensor, drives a 12 V heater through PWM, and provides both local and Bluetooth control.

**My role:** system schematic design, PCB soldering and electrical bring-up, PID implementation and tuning, hardware-software integration, and project documentation.

**Source code:** [STM32 firmware and CubeMX project](https://github.com/ch-yu02/My-ISEE-Course-Reference/tree/main/%E7%9F%AD%E5%AD%A6%E6%9C%9F-%E7%94%B5%E5%AD%90%E7%94%B5%E8%B7%AF%E7%B3%BB%E7%BB%9F%E7%BB%BC%E5%90%88%E5%AE%9E%E9%AA%8C/%E6%9C%80%E7%BB%88%E7%89%88/20250627)

<div class="row mt-3">
  <div class="col-sm-3 mt-2">
    <div class="border rounded p-3 h-100 text-center">
      <strong>±2 °C</strong><br>
      <span>steady-state fluctuation</span>
    </div>
  </div>
  <div class="col-sm-3 mt-2">
    <div class="border rounded p-3 h-100 text-center">
      <strong>&lt; 2 °C</strong><br>
      <span>startup overshoot</span>
    </div>
  </div>
  <div class="col-sm-3 mt-2">
    <div class="border rounded p-3 h-100 text-center">
      <strong>&lt; 10 s</strong><br>
      <span>settling time</span>
    </div>
  </div>
  <div class="col-sm-3 mt-2">
    <div class="border rounded p-3 h-100 text-center">
      <strong>~13 W</strong><br>
      <span>peak heating power</span>
    </div>
  </div>
</div>

<div style="width: 54%; margin-inline: auto;" class="mt-4">
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

- **Sensing:** a 10 kΩ, B3950, 1% NTC thermistor is sampled through the STM32 ADC, while a GY906 infrared sensor provides an independent temperature reading over I²C.
- **Control:** the STM32 computes heater demand from the measured temperature and target setpoint.
- **Actuation:** a MOSFET power stage switches a 12 V resistive heater using PWM.
- **Interface and safety:** an OLED and four buttons provide local control, a UART Bluetooth module accepts remote commands, and a buzzer provides an over-temperature alarm.

> **Control loop:** Sense → Gain-scheduled PID → PWM → MOSFET → Heater → Aluminum Plate

<div style="width: 72%; margin-inline: auto;">
  {% include figure.liquid path="assets/img/projects/temperature-control/circuit-diagram.jpg" title="Temperature-control schematic" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  System schematic.
</div>

The hardware was designed around a **12 V input**. An LM7805 stage generates the 5 V rail for the control electronics, while the heater remains on the 12 V path. The heating element used in the lab was a **10 Ω / 10 W high-power resistor** thermally coupled to the aluminum plate.

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

Electrical bring-up produced a regulated output of approximately **5.04 V**. At maximum heating command, measured heater power was approximately **13 W**; with heating disabled, total system power was approximately **0.7 W**. PWM waveforms were also checked on an oscilloscope to confirm that duty-cycle commands reached the power stage correctly.

## Control implementation

I implemented and tuned a **gain-scheduled PID controller** rather than using one fixed set of gains across the entire heating range. The final implementation divides control into several error regions:

- for a positive error above **8 °C**, the controller commands full heating;
- for errors above **3 °C**, it uses `Kp = 18`, `Ki = 0`, `Kd = 0`;
- for errors above **1.5 °C**, it uses `Kp = 14`, `Ki = 0`, `Kd = 100`;
- near the setpoint, it uses `Kp = 10`, `Ki = 4`, `Kd = 500`.

Integration is only accumulated when the absolute error is below **1 °C**, limiting integral wind-up away from the target. The derivative term is smoothed with a moving-average buffer, and the final controller output is saturated to the valid **0-100% PWM** range.

During tuning, I also experimented with staged heating and hysteresis-based control. Those alternatives were ultimately left out of the active control path after hardware tests showed that the gain-scheduled PID gave the better balance between heating time and temperature fluctuation for this prototype.

<!--The final prototype **passed acceptance against all specified control requirements**, achieving a steady-state temperature fluctuation range within **±2 °C**, startup overshoot below **2 °C**, and settling within **10 s**.-->

## Interaction and safety

The local interface uses four interrupt-driven buttons for set mode, +1 °C, -1 °C, and start/stop. The OLED displays the target temperature, both GY906 and NTC readings, and controller state.

Bluetooth control is handled over UART and supports commands to:

- set the target temperature;
- set the maximum / alarm temperature;
- query both sensor readings and the current target;
- start or stop heating.

In the implementation, target updates are accepted from **30 °C up to the configured maximum**, while the configurable maximum-temperature limit is constrained to **35-60 °C**.

The over-temperature path drives a buzzer and also sends a UART warning. One hardware issue remained in the prototype: the buzzer's series resistance was too large, so the alarm was functionally correct but quieter than intended.

## Results

System-level testing covered:

- NTC calibration against the GY906 reference trend;
- PWM generation and MOSFET heater control;
- multiple heating and cooling conditions around the setpoint;
- button debouncing and OLED state display;
- over-temperature alarm behavior;
- Bluetooth commands for setpoint, status, and start/stop control.

The completed prototype passed the final course acceptance and satisfied all basic and extended design requirements. It maintained stable closed-loop operation across the tested setpoints, while meeting the required **±2 °C steady-state fluctuation range, <2 °C startup overshoot, and <10 s settling time**.

_This was a two-person project. My teammate designed the PCB layout and implemented the sensor, OLED, button, alarm, and Bluetooth functions; the work described in “My role” above is my individual contribution._
