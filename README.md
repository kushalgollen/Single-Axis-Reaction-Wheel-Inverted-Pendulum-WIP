<div align="center">

# ⚙️ 1-DoF Reaction Wheel Inverted Pendulum
### Dynamic Balancing via High-Precision Field-Oriented Control (FOC)

[![Status](https://img.shields.io/badge/Status-Work_In_Progress-amber.svg?style=for-the-badge)](#roadmap)
[![Hardware](https://img.shields.io/badge/MCU-ESP32--WROOM--32E-blue.svg?style=for-the-badge)](#hardware-architecture)
[![Control](https://img.shields.io/badge/Algorithm-FOC_%2B_Cascaded_PID-green.svg?style=for-the-badge)](#control-theory--architecture)
[![License](https://img.shields.io/badge/License-MIT-lightgrey.svg?style=for-the-badge)](LICENSE)

<p align="center">
  <b>A mechatronic testbed designed to explore real-time nonlinear control, high-torque BLDC actuation, and inertial momentum transfer.</b>
</p>

</div>

---

## 📌 Executive Summary

This repository hosts the hardware design, embedded firmware, and control architecture for a single-axis self-balancing reaction wheel inverted pendulum. 

Rather than relying on low-bandwidth stepper motors or standard drone ESCs (which lack fine zero-velocity modulation), this project implements true **Field-Oriented Control (FOC)** on an ultra-low KV gimbal motor. By coupling rotor angular acceleration with an attitude estimation loop, the system exchanges angular momentum in real time to stabilize an inherently unstable equilibrium state.

> ⚠️ **Project Status: Work in Progress (Active R&D Phase)**  
> Hardware integration, telemetry acquisition, and physical characterization are actively being finalized. Current commits reflect prototype firmware and initial bench validation.

---

## 🔬 Core Engineering Highlights

- **Direct-Drive Gimbal Actuation**: Powered by an **iPower GM4108H-120T** high-pole-count BLDC motor running without mechanical reduction to eliminate backlash and deadband.
- **True Field-Oriented Control (SimpleFOC)**: Sinusoidal current generation via space-vector modulation (SVPWM) enabling seamless bidirectional torque reversal across 0 RPM.
- **Dual Sensing on a Shared I2C Bus**: High-rate synchronization between an **AS5600** magnetic rotary encoder (rotor commutation feedback) and an **MPU6050** 6-DoF IMU (tilt/attitude tracking).
- **Dual-Core Asynchronous Pipeline**: ESP32 RTOS architecture allocating motor commutation to an ultra-fast inner loop while delegating filtering and attitude PID loops to the secondary core.

---

## 🏗️ Hardware Architecture

### System Block Diagram

```text
       +-------------------------------------------------------------+
       |                     24V DC / LiPo Power                     |
       +------------------------------+------------------------------+
                                      |
                                      v
+---------------------------------------------------------------------------------+
| ESP32-WROOM-32E Integrated FOC Controller                                       |
|                                                                                 |
|  [Core 0]  Attitude Loop (Cascaded PID) <------- MPU6050 (6-DoF IMU)            |
|       |                                                                         |
|       v                                                                         |
|  [Core 1]  FOC Commutation Loop (Voltage/Torque) <-- AS5600 Magnetic Encoder    |
|       |                                                                         |
|       +--> 3x IR2104 Half-Bridge Drivers --> 6x Discrete N-MOSFETs (Up to 20A) |
+-------------------------------------+-------------------------------------------+
                                      |
                                  3-Phase
                                  (MA/MB/MC)
                                      |
                                      v
                     +----------------------------------+
                     | iPower GM4108H-120T BLDC Motor   |
                     | (Hollow Shaft, 0.6+ Nm peak)     |
                     +-----------------+----------------+
                                       |
                                       v
                     +----------------------------------+
                     | High-Inertia Perimeter Flywheel  |
                     | (Optimized Radius-to-Mass Ratio) |
                     +----------------------------------+
