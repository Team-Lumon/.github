# LumonBot — 8-Cable Driven Parallel Robot

**EN2160 - Electronic Design Realization | University of Moratuwa**
**Department of Electronic and Telecommunication Engineering**

LumonBot is a **Cable-Driven Parallel Robot (CDPR)** designed to automate the manual crate-loading bottleneck found in garment manufacturing pipelines. Instead of a bulky six-axis robotic arm, LumonBot uses a lightweight frame of 8 tensioned cables to control a 3-DOF actuator — enabling a lower-cost, scalable, and reconfigurable alternative for constrained factory floor spaces.

---

## 🛠️ System Architecture

```
Control UI (Python Web App)
        │  Wi-Fi (UDP)
        ▼
   ESP32 (Comms Bridge)
        │  UART
        ▼
  STM32H7 (Main Controller)
   — Inverse Kinematics
   — Cable-length → Motor step conversion
   — Trajectory segmentation (~100ms intervals)
        │  CAN Bus (Sync + Commands)
        ▼
 8× Motor Controller Nodes (STM32G0 + TMC2209)
        │
        ▼
 8× Cable-Spool Assemblies (NEMA23 Steppers)
        │
        ▼
   Moving 3-DOF Actuator/Grabber
        (ESP32 + TMC2209, NEMA17, Wi-Fi controlled)
```

The main controller performs inverse kinematics on incoming target coordinates, discretizes motion into ~100ms trajectory segments, and broadcasts synchronized cable-length commands to 8 distributed motor nodes over CAN — ensuring coordinated, jitter-free platform motion.

---

## 🔌 PCB Subsystems

| PCB | Qty | Layers | Core Components | Function |
|---|---|---|---|---|
| **Main Controller** | 1 | 4-layer | ESP32 + STM32H7 | Parses UI commands, computes inverse kinematics, generates synchronized CAN commands for all 8 motor nodes, bridges Wi-Fi ↔ CAN |
| **Motor Controller** | 8 | 2-layer | STM32G0 + TMC2209 | One per cable-drive actuator; converts CAN length-deltas into step/direction signals; reports ACK & target-reached status |
| **Power Distribution** | 2 | 2-layer | 24V→5V Buck Converter | Distributes main 24V rail, regulates 5V logic supply, wide copper pours for high-current paths |
| **Actuator Board** | 1 | 2-layer | ESP32 + TMC2209 | Drives the NEMA17 gripper motor; receives open/close commands wirelessly via Wi-Fi |

**Stack-up rationale:** The main controller uses a 4-layer board (dedicated GND/PWR planes) to support the mixed digital, wireless, and CAN circuitry of the ESP32 + STM32H7 pairing with minimal EMI. The three peripheral boards use standard 2-layer, 1.6mm FR-4 stack-ups to keep manufacturing cost low across 8 identical motor-node units.

---

## 🧮 Inverse Kinematics

- 8 cables control 6 pose variables (3-DOF translation implemented in the prototype; orientation fixed).
- Cable-length model accounts for **pulley wrap geometry** (not just point-source anchors), giving accurate free-length + wrap-length segments per cable.
- A **Quadratic Programming (QP) solver** resolves the redundant cable tension distribution, keeping all 8 cables within safe tension bounds (2N–50N) while tracking the required dynamic force/torque.
- The theoretical workspace assuming a point-mass actuator was ~80% of the frame volume; once real actuator dimensions (13×11×6 cm) were included, the tension-feasible, collision-free workspace dropped to **34.6%** of the frame volume.

---

## 📋 Final Product Specifications

| Parameter | Target | Achieved |
|---|---|---|
| Controlled DOF | 6 | 3 (design intent) |
| Actuated cables | 8 | 8 |
| Workspace | 1 m³ | 0.8 m³ enclosed / 34.6% usable |
| Payload | 2 kg | 1.5 kg |
| Position accuracy | 5 mm | 1 cm |
| Repeatability | — | 0.8 mm |
| Max cable speed | 0.6 m/s | 0.2 m/s (7 cm/s translational) |
| Frame dimensions | — | 1m × 1m × 1m |
| Actuator mass | — | 500 g |
| Cable type | — | Nylon paracord |
| Motor | — | NEMA23 (motors), NEMA17 (gripper) |
| Driver | — | TMC2209, 2.8A/phase |
| Supply voltages | — | 24V motor / 3.3V logic |
| Communication | — | Wi-Fi/UDP + CAN + local UART |
| E-Stop | — | Physical E-Stop button |

---

## 🚀 Future Improvements

- Closed-loop cable tension sensing via load cells
- Grooved/level-wind spool or direct cable-length encoders
- Automatic workspace/pose validation (reject infeasible commands)
- Self-calibration of anchor coordinates
- Redundant E-stop, cable-break detection, formal fault-tree analysis
- Production-grade integrated PCBs (replace dev boards with bare ESP32/STM32 modules)
- Digital twin simulation for workspace/tension/power validation
- Scale from current 2–3 kg validated payload to a 25–30 kg target payload (real garment crate weight) using higher-torque motors, Dyneema cables, and a reinforced frame

---

## 📚 References

Key datasheets and papers used in this project include the TMC2209 stepper driver, ESP32-S3-WROOM-1 module, TCAN341x CAN transceiver, STM32 documentation, ISO 11898-2 (CAN), and academic literature on CDPR wrench-feasible workspace analysis (Pott, Bosscher et al., Gouttefarde et al.). See the full final report for the complete bibliography.

---

*Developed as part of EN2160 - Electronic Design Realization, Department of Electronic and Telecommunication Engineering, University of Moratuwa.*
