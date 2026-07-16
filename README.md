# EV ADAS Dashboard

**Real-time Electric Vehicle telemetry + ADAS warning system**
STM32 Blue Pill · PicsimLab · Python (Matplotlib)

A simulated EV control unit built on an STM32 Blue Pill (STM32F103C8T6) that models core electric-vehicle dynamics and a collision/blind-spot warning system, streaming live telemetry over UART to a real-time Python dashboard.

---

## Overview

Modern EVs generate a constant stream of sensor data — speed, battery SOC, motor temperature, torque, and range — alongside safety-critical ADAS signals like collision distance and blind-spot occupancy. This project builds a compact, hardware-simulated version of that pipeline end to end:

```
Sensors (simulated) → STM32 Blue Pill → UART (115200 bps) → Python Dashboard
```

The firmware models EV dynamics and an ultrasonic-based ADAS engine, escalates faults through a tiered alarm system, and transmits structured telemetry frames that a Python/Matplotlib dashboard parses and visualizes in real time.

---

## Features

- **Real-time EV dynamics** — speed, motor torque, battery SOC, estimated range, regenerative braking, and ECO/NORMAL/SPORT drive modes
- **ADAS safety stack**
  - Forward Collision Warning (FCW) with hysteresis filtering
  - Time-To-Collision (TTC) estimation
  - Blind Spot Detection (BSD), speed-gated
  - Parking Assist scoring
  - Overspeed advisory
- **Fault detection & alarm escalation** — bit-field fault register with a 4-level alarm priority system (`NONE → ADVISORY → WARNING → CRITICAL`)
- **UART telemetry protocol** — two structured ASCII frames transmitted every 100 ms
- **UART shell** — inject speed/SOC/temperature/fault values live for deterministic testing, without touching hardware
- **Live Python dashboard** — speedometer, SOC bar, ADAS bird's-eye view, speed history chart, and system metrics panel, rendered at 10 fps via `matplotlib.animation.FuncAnimation`

---

## Architecture

| Stage | Description |
|---|---|
| **1. Sensors (simulated)** | Potentiometers for accelerator, brake, and motor temperature; HC-SR04 ultrasonic sensors (front/left/right) |
| **2. STM32 Blue Pill** | EV control loop, ADAS engine, fault handler — all running on timer interrupts |
| **3. UART (115200 bps)** | Two telemetry frames per cycle: EV state + ADAS state |
| **4. Python Dashboard** | Parses incoming frames and renders live visualizations |

### Real industry ↔ project mapping

| Real Industry Component | This Project |
|---|---|
| Pressure / wheel-speed / proximity sensors | Potentiometers (ADC) + PicsimLab simulation |
| ECU / Vehicle Control Unit (VCU) | STM32 Blue Pill (F103C8T6) |
| CAN Bus / LIN Bus | UART 115200 bps (USART1) |
| Industrial cluster / HMI display | Python Matplotlib dashboard |
| OBD-II diagnostic port / DTC reader | UART shell + fault register (bit-field) |
| Multi-tone buzzer alarm system | Tiered alarm (NONE / ADVISORY / WARNING / CRITICAL) |

---

## Tech stack

- **Firmware:** C (STM32CubeIDE, HAL), timer interrupts (TIM1–TIM4), ADC, PWM, GPIO, USART1
- **Simulation:** PicsimLab (STM32 Blue Pill virtual hardware)
- **Dashboard:** Python 3, `matplotlib`, `numpy`, `pyserial`

---

## Project phases

1. **Dev environment** — STM32CubeIDE + PicsimLab + Python setup
2. **GPIO & peripherals** — LEDs, PWM buzzer, timer interrupts
3. **ADC sensor inputs** — accelerator, brake, motor temperature
4. **EV dynamics model** — speed, torque, SOC, range, regen braking, drive modes
5. **Ultrasonic ADAS engine** — FCW, BSD, TTC, parking assist
6. **Fault & alarm system** — bit-field faults, 4-level alarm priority
7. **UART telemetry** — structured frame protocol
8. **UART shell** — live parameter injection for testing
9. **Python dashboard** — real-time Matplotlib visualization

---



| Field | Meaning |
|---|---|
| `SPD` | Vehicle speed (km/h) |
| `SOC` | Battery state of charge (%) |
| `TRQ` | Motor torque (Nm) |
| `TMP` | Motor temperature (°C) |
| `RNG` | Estimated range (km) |
| `F / L / R` | Ultrasonic distance, front/left/right (cm) |
| `TTC` | Time-to-collision (s) |
| `COL` | Collision level (0–2) |
| `BSD` | Blind spot L/R bits |
| `ALM` | Alarm priority (0–3) |
| `FLT` | Fault byte (hex) |

---

## UART shell commands

For deterministic testing without moving physical potentiometers:

| Command | Description |
|---|---|
| `speed <km/h>` | Set vehicle speed directly |
| `soc <0-100>` | Set battery state of charge |
| `temp <°C>` | Set motor temperature |
| `mode <0-2>` | ECO=0 / NORMAL=1 / SPORT=2 |
| `fault <hex>` | Inject fault bits (e.g. `fault 01`) |
| `reset` | Clear all faults, restore normal state |
| `status` | Print current EV + ADAS state summary |

---

## Getting started

### Requirements
- [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)
- [PicsimLab](https://sourceforge.net/projects/picsimlab/) (STM32 Blue Pill simulation)
- Python 3.8+

### Setup

```bash
# Clone the repo
git clone https://github.com/<your-username>/ev-adas-dashboard.git
cd ev-adas-dashboard

# Install Python dependencies
pip install matplotlib numpy pyserial
```

1. Open the firmware project in STM32CubeIDE and build it.
2. Load the built binary in PicsimLab (or flash to a real STM32 Blue Pill).
3. Run the dashboard:

```bash
# With real/simulated hardware on a serial port
python dashboard.py --port COM3

# Demo mode, no hardware required
python dashboard.py --demo
```

---

## Results

- Speedometer updates at 10 fps; SOC drains correctly under load and recovers under regenerative braking
- Forward collision alerts escalate reliably to CRITICAL below 20 cm, with a dashboard red-flash effect
- Blind spot detection triggers above 20 km/h with 3-sample hysteresis — no false positives at highway speed
- Motor over-temperature fault (`FAULT_OT`) correctly cuts torque and latches the fault LED
- All 5 dashboard panels render live from the UART stream; demo mode runs without hardware
- UART shell commands (`speed`, `soc`, `fault`, `reset`) verified via PicsimLab's serial monitor

---



---

## Author

Dheeraj L
Built as part of an embedded systems intership — **EMERTXE**
