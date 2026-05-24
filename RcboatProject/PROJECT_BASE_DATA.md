# RC Boat Electrical Leak Detector - Project Base Data

## 1. Project Overview
A dual-core ESP32-based RC Boat designed to detect AC electrical leaks (via ZMPT101B) in water. It features GPS tracking, Telegram alerts, a robotic arm (MG996R servos), remote control via Flysky iA6B, and an autonomous Return-To-Home (RTH) failsafe mechanism.

## 2. Hardware Components & Specifications

### 2.1 Microcontroller
*   **Module:** ESP32-WROOM-32 (TYPE-C module with CP2102)
*   **Operating Voltage:** 3.3V Logic
*   **Power Input:** 5V via USB or 5-12V via VIN pin
*   **Role:** Main processing (Core 0: Network/GPS, Core 1: Control/Telemetry/Sensors)

### 2.2 Power Architecture
*   **Main Battery:** 18650 x 4 (Configured as 2S2P)
    *   Nominal Voltage: 7.4V
    *   Full Charge: 8.4V
    *   Capacity: 5200 mAh
*   **Motor Power:** Direct 7.4V from Battery to L298N V_in.
    *   *Capacitor:* 1000uF / 16V across L298N V_in for surge protection.
*   **Logic Power (Step-Down 1):** LM2596S Buck Converter
    *   Input: 7.4V
    *   Output: Tuned to **5.0V**
    *   Powers: ESP32 (VIN), GPS NEO-7M, Flysky iA6B Receiver, ZMPT101B.
    *   *Capacitor:* 100uF across output for smoothing.
*   **Servo Power (Step-Down 2):** UBEC 5A
    *   Input: 7.4V
    *   Output: Set to **6.0V** (via jumper/switch)
    *   Powers: 3 x MG996R Servos ONLY.
    *   *Capacitor:* 100uF across output for smoothing.

### 2.3 Motor Control
*   **Driver:** L298N Dual H-Bridge
*   **Logic Voltage:** 5V (Internal 5V regulator MUST be enabled if input is 7-12V, but Logic IN from ESP32 is 3.3V which is compatible).

### 2.4 Servos (Robotic Arm)
*   **Model:** MG996R (All Metal Gears) x 3
*   **Operating Voltage:** 4.8V - 7.2V (Powered at 6.0V via UBEC)
*   **Stall Current:** High (Requires the dedicated UBEC).

### 2.5 Sensors & Modules
*   **Leak Detector:** ZMPT101B AC Voltage Mutual Inductance Module.
    *   Output: Analog Sine Wave (0-VCC range, VCC=5V, but scaled via POT to 0-3.3V for ESP32 ADC).
*   **GPS:** U-blox NEO-7M
    *   Baud Rate: 9600
    *   Communication: UART (TX from GPS only).
*   **Receiver:** Flysky FS-iA6B
    *   Communication: IBUS (UART2 on ESP32)
    *   Features: Telemetry (Sending Leak RMS voltage and potentially Battery Voltage back to remote).

## 3. Pin Mapping (ESP32)

| Component | Pin Function | ESP32 GPIO | Notes |
| :--- | :--- | :--- | :--- |
| **GPS NEO-7M** | TX | 32 (RX1) | UART1 RX |
| **Flysky iA6B** | IBUS Servo | 16 (RX2) | Control Input |
| **Flysky iA6B** | IBUS Sens | 17 (TX2) | Telemetry Output |
| **ZMPT101B** | OUT | 34 (ADC1_CH6) | Analog Input |
| **L298N (Left)** | ENA | 26 | PWM (1kHz) |
| **L298N (Left)** | IN1 | 27 | Direction |
| **L298N (Left)** | IN2 | 14 | Direction |
| **L298N (Right)**| ENB | 23 | PWM (1kHz) |
| **L298N (Right)**| IN3 | 18 | Direction |
| **L298N (Right)**| IN4 | 19 | Direction |
| **MG996R 1** | Base | 13 | PWM (50Hz) |
| **MG996R 2** | Pitch | 21 | PWM (50Hz) |
| **MG996R 3** | Gripper | 25 | PWM (50Hz) |
| **Battery Monitor**| V_Div OUT | 33 (ADC1_CH5)| *Planned: 22k/10k Divider* |

## 4. Software Features
*   **Core 0:** WiFi STA, Telegram Alerting (Blocking queue based), GPS NMEA parsing.
*   **Core 1:** Fast loop for IBUS control, Motor mixing, Servo PWM, ZMPT101B sampling (EMA filtered).
*   **RTH Failsafe:** Engages if IBUS signal is lost for >1000ms. Navigates back to the first locked GPS coordinate.