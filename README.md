# Throttle Body Model in a Hardware-in-the-Loop System

This repository contains the Simulink models and technical documentation for the development and integration of a real-time virtual Electronic Throttle Body (ETB) within a Propulsion Hardware-in-the-Loop (HiL) environment. The project was developed for the **"Control Lab"** course.

## 📝 Project Overview

During intensive automotive validation cycles, physical throttle bodies are subjected to high-frequency actuation commands from the Engine Control Module (ECM), often leading to mechanical failure of gears and return springs. The objective of this project is to replace the physical component with a robust, real-time simulated model that guarantees test repeatability and eliminates hardware downtime, while strictly satisfying the ECM's electrical and diagnostic requirements.

## 🛠️ Hardware & Software Environment

The simulated model interacts directly with a real production ECU.
* **Simulator:** dSPACE SCALEXIO.
    * **DS2680 I/O Unit:** Used for high-speed PWM measurement (commands) and analog signal generation (sensor feedback).
    * **DS5380 Electronic Load Module:** A programmable current sink used to emulate the motor's power consumption.
* **ECU:** Bosch Engine Control Module.
* **Toolchain:** dSPACE ConfigurationDesk & ControlDesk, ETAS INCA (for calibration and measurement), Vector Tools.

## ⚙️ Modeling Architecture

Simulating an actuator for a real ECU requires more than just kinematic equations; it requires tricking the ECU's diagnostics (DTCs).

### 1. Dynamic Plant Modeling
Instead of a complex second-order system, the model utilizes a **Modified First-Order Filter**. Because the ECU's internal closed-loop controller is highly robust, this simplification is effective while reducing the parameters needed for calibration. The model incorporates a specific feedback loop to simulate the physical restoration forces (return springs) and friction.

### 2. ECM Diagnostic Handling (The Edge Cases)
* **Learning Phase:** At every Key-On, the ECU sweeps the valve to learn the mechanical end-stops. The model detects this phase and dynamically switches its time constant ($\tau_{learn}$) to mimic the specific calibration response, preventing "Self-Learning" diagnostic errors.
* **Rest Position:** When uncommanded, physical springs hold the valve slightly open (~6.4°). The model detects a $0\%$ PWM command and forces the integrator to this specific default position to avoid "Limp-Home Position" faults.

### 3. Electrical Load Emulation
A simple resistor cannot simulate the dynamic impedance of a DC motor; an open circuit triggers an "Open Load" fault, causing the ECU to cut power. 
To solve this, the model calculates the theoretical motor current using an **RL Circuit Model**. This computed current is sent to the physical **dSPACE DS5380 Load Board**, which physically draws the exact current from the ECU, perfectly emulating the real motor coils.

## 📊 Testing & Validation

The model was validated on a full Jeep Wrangler HiL simulator using ETAS INCA and ControlDesk. 
* **Transient & Dynamic Tracking:** The model successfully matched the physical component's rise time ($20ms$) and settling time in step responses and train pulse tracking.
* **Diagnostic Success:** The model successfully passed the ECU's strict Learning Phase. Diagnostics checks (via CDA tool) confirmed **Zero Active DTCs** after integration.

## 📂 Repository Structure

* **`ETC_Model.slx`:** The Simulink model containing the input PWM processing, dynamic First-Order Filter, learning phase logic, and the RL load emulation subsystem.
* **`Control_Lab_Report.pdf`:** The full technical report detailing the mathematical equations, hardware pinouts, dSPACE configuration, and test analyses.
* **`Control_Lab_Presentation.pdf`:** Summary slide deck of the project architecture and validation results.
