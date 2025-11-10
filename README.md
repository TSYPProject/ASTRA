#  ASTRA Artificial Supervision & Time-Critical Recovery Automation
**IEEE IES & AESS Challenge – Phase 1 Submission**

---

##  Project Overview

This project introduces a **fragmented AI agent architecture** distributed across **two linked STM32H7 boards** to provide **autonomous fault detection, diagnosis, and preventive action** in CubeSat systems.  
The boards operate in a **mutual supervision loop**, ensuring redundancy and continuity:  
if one board fails, the other automatically activates a backup process to maintain operation stability.

The system is designed to:
- Detect anomalies in real-time from sensor inputs,  
- Diagnose the **root cause** of faults (thermal, power, or sensor-related), and  
- Execute **preventive or corrective actions** via an **FPGA-based control unit**.

---

##  System Concept

The architecture forms a **distributed AI network** of two microcontrollers and one FPGA controller:

STM32H7_1 – AI Layer 1: Anomaly Classifier
- Reads temperature, voltage, current data
- 1D CNN classifies inputs: Normal/Anomaly
- Outputs confidence matrix (e.g.,
[Normal=0.91, Anomaly=0.09])
- Sends matrix → STM32H7_2

STM32H7_2 – AI Layer 2: Root Cause Diagnosis
- Receives matrix + live telemetry data
- AI Model (Tiny RF / XGBoost)
- Predicts root cause:
(Voltage_Sag, Solar_Instability,
Sensor_Drift, Overheating, etc.)
- Sends preventive action vector → FPGA


FPGA – Preventive & Recovery Controller
- Executes power/thermal corrective actions
- Enables safe mode or redundancy
- Reacts within < 1 ms

---

##  Mutual Supervision & Continuity Mechanism

Both STM32 boards are interlinked through a **bidirectional supervision channel**:

| Function | Description |
|-----------|-------------|
| **Feedback Loop** | Each board monitors the heartbeat of the other via GPIO interrupts or serial watchdog signals. |
| **Failover Logic** | If one board becomes unresponsive, the other activates its backup AI model and restores system operation automatically. |
| **Synchronization** | Both boards exchange operational states and inference confidence to maintain consistent decision-making. |

This ensures that **no single-point failure** can halt anomaly detection or power management.

---

## ⚙️ Functional Breakdown

| Module | Processor | AI Role | Output | Action |
|---------|------------|----------|---------|--------|
| **STM32H7_1** | MCU #1 | CNN classifier | Confidence matrix `[normal, anomaly]` | Sends matrix to STM32H7_2 |
| **STM32H7_2** | MCU #2 | Random Forest / XGBoost | Root cause label + preventive action | Sends action vector to FPGA |
| **FPGA** | Xilinx/Intel | Logic executor | Hardware control signals | Cuts/restores power, safe mode, redundancy activation |

---

##  AI Model Flow

### **1️⃣ AI Layer 1 – CNN Classification**
- **Input:** Temperature, voltage, current (OBC + PDU)
- **Output:** Confidence matrix
  ```text
  [Normal: 0.92, Anomaly: 0.08]

Logic: Lightweight 1D-CNN trained on sensor data to recognize deviations from nominal CubeSat behavior.

2️⃣ AI Layer 2 – Root Cause Diagnosis

Input: Confidence matrix + OBC/PDU telemetry

Model: Tiny Random Forest or XGBoost

Output Example:
Cause: Voltage_Sag
Risk Level: Medium
Recommended Action: Shed non-critical loads & restart converter

Response: Sends structured preventive command to FPGA.

3️⃣ FPGA Logic Execution

Receives control vector and applies hardware actions:

Cut or restore power rails

Enable redundant modules

Trigger Safe Mode or alert logic
Example Actions Database
| Root Cause              | Component | Risk Level | Action                                     |
| ----------------------- | --------- | ---------- | ------------------------------------------ |
| **Voltage_Sag**         | PDU       | MEDIUM     | Shed non-critical loads, restart converter |
| **Overheating**         | OBC       | CRITICAL   | Switch to backup OBC, enable Safe Mode     |
| **Sensor_Drift**        | OBC       | LOW        | Increase sampling, recalibrate sensors     |
| **Solar_Instability**   | PDU       | MEDIUM     | Reduce bus voltage, alert ground           |
| **Thermal_Instability** | OBC       | HIGH       | Throttle CPU, engage redundant cooling     |

 Key Features

 -Fragmented dual-AI agent with bidirectional supervision 

 -1D-CNN classification + Random Forest/XGBoost diagnosis 

 -Mutual failover mechanism for continuous operation

 -FPGA-based corrective actions and redundancy management

 -Real-time inference with <1 ms hardware response latency

<img width="611" height="866" alt="image" src="https://github.com/user-attachments/assets/58b3f1eb-f69f-424d-a9c8-b3571b92101a" />
