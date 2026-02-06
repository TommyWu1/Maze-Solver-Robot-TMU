# Maze Solver Robot (EEbot)

![IMG_3420](https://github.com/user-attachments/assets/aef956cc-7cb9-4ed3-aa7d-8614a7a46ae7)


This project contains the embedded Assembly software for an autonomous maze-solving robot, designed to run on the **eebot** platform powered by a **Freescale HCS12 microcontroller** (9S12C32). The system uses real-time sensor data to navigate lines, detect intersections, and handle physical obstacles.

## 🤖 Hardware Platform: The "eebot"

The software is engineered for the "eebot" (v2), a mobile robot platform used at Toronto Metropolitan University (formerly Ryerson University).

* **Microcontroller:** Freescale HCS12 (9S12C32).
* **Sensors:** A custom "Guider" board featuring 6 Cadmium Sulphide (CdS) photoresistors.
    * **Absolute Sensors (A, B, C, D):** Arranged in a "T" pattern to detect line junctions and sharp turns.
    * **Differential Sensors (E, F):** A voltage divider pair used for precision edge tracking of the black tape.
* **Actuators:** Dual DC gearmotors (203:1 ratio) controlled via HCS12 Output Compare timers (PWM for speed) and digital ports (Direction).
* **Feedback:** Two bumper switches (Bow/Stern) for collision detection and Wheel Rotation Detectors for odometry.

## 📂 Project Structure

### 1. `Maze Navigation (MAIN)`
The core application logic for the maze run.
* **File:** `Sources/main.asm`
* **Function:** Implements a state-machine-based control system. It initializes the Hardware (ADC, PWM, LCD), polls sensors, and executes movement logic based on the current state (e.g., `FWD`, `LEFT_TRN`, `ALL_STOP`).

### 2. `Sensor Calibration`
A diagnostic utility to visualize raw sensor data.
![IMG_2833](https://github.com/user-attachments/assets/f072fca9-e1d8-46de-a05d-ca5f0e1bd34a)

* **File:** `Sources/main.asm` (in Calibration directory)
* **Function:** Continuously reads analog values from all 5 optical sensors and displays them in Hexadecimal on the onboard 20x2 LCD.
* **Usage:** Used to determine the `BASE_LINE` and `VARIANCE` constants required for the main navigation logic.

## ⚙️ Software Architecture

The system is built on a **State Design Pattern** implemented in Motorola 68HC12 Assembly.

### State Machine
The robot transitions between the following states based on sensor inputs:
* **START:** Idle state, waiting for activation.
* **FWD:** Normal line following. The robot checks the "Bow" (Front) and "Mid" sensors against calibrated thresholds.
* **LEFT_TRN / RIGHT_TRN:** Triggered when "Port" or "Starboard" sensors detect a line, indicating an intersection or sharp turn.
* **LEFT_ALIGN / RIGHT_ALIGN:** Fine-tuning states to re-center the robot on the line after a turn.
* **REV_TRN:** Triggered by the **Bow Bumper**; the robot reverses and attempts to realign.
* **ALL_STOP:** Triggered by the **Stern Bumper** or specific sensor conditions.

### Key Logic Features
* **Interrupt Driven Timing:** Uses Timer Overflow Interrupts (`TOF_ISR`) to handle non-blocking delays for turn duration (e.g., `T_LEFT`, `T_RIGHT`).
* **Threshold Comparison:** Logic compares real-time ADC readings against hardcoded `BASE` values plus a `VARIANCE` margin to filter noise.
* **LCD Feedback:** Real-time display of the current state (e.g., "LeftTurn") and battery voltage monitoring.

## 🚀 How to Run

1.  **Hardware Setup:** Ensure the eebot is powered (9.6V NiCad battery) and the sensor board is connected to Port A.
2.  **Calibration:**
    * Flash the **Sensor Calibration** program.
    * Place the robot on the maze surface (white background) and black tape.
    * Note the hex values for "Light" and "Dark" readings.
    * Update the `BASE_` and `_VARIANCE` constants in the `Maze Navigation` source code.
3.  **Deployment:**
    * Compile and flash the **Maze Navigation** program using CodeWarrior.
    * Place the robot at the start line.
    * The robot will initialize in the `ALL_STOP` state and begin navigation upon detecting the start condition.

## 🛠 Technology Stack

* **Language:** Assembly (HCS12/9S12)
* **IDE:** CodeWarrior for HCS12
* **Peripherals Used:** ATD (Analog-to-Digital), GPIO (Ports A, B, T, J), TIM (Timer Module).

## Citations

eebot Technical Description by: Peter Hiscocks
