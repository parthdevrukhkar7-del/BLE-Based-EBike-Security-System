# ESP32 BLE Smart E-Bike Lock System

An IoT-based smart e-bike locking system developed using ESP32, BLE communication, Embedded C, relay modules, and solenoid lock mechanisms for secure Bluetooth-enabled authentication and locking control.

---

# Project Overview

Traditional bicycle locks can be physically broken or duplicated easily.  
This project introduces a Bluetooth-enabled smart locking system for electric bicycles using ESP32 and BLE authentication.

The user can securely lock or unlock the e-bike using a private secret key through Bluetooth communication from a mobile device.

The system was developed during an internship at Leakamp for practical e-bike security implementation.

---

# Features

- Bluetooth-enabled lock and unlock system
- Secure authentication using secret key validation
- ESP32 BLE communication
- Solenoid-based locking mechanism
- Automatic re-locking after timeout
- Low-voltage hardware control using relay modules
- Real-time BLE notification response system

---

# Technologies Used

- Embedded C
- ESP32 WROOM
- Bluetooth Low Energy (BLE)
- Arduino IDE
- IoT-based Embedded Systems

---

# Components Used

- ESP32 WROOM
- Relay Module
- Solenoid Lock
- DC-to-DC Buck Converter
- Lithium Battery
- Connecting Wires
- E-bike Battery Supply

---

# Working Principle

1. The user purchases the e-bike and receives a private secret key.
2. The ESP32 creates a BLE communication interface.
3. The mobile device connects to the ESP32 via Bluetooth.
4. The secret key is verified through BLE authentication.
5. If authentication succeeds:
   - The solenoid lock unlocks.
   - BLE notification confirms authorization.
6. If authentication fails:
   - The lock remains secured.
   - Unauthorized access message is generated.
7. The system automatically re-locks after 10 seconds for security.

---

# Circuit Functionality

The ESP32 receives Bluetooth signals from the mobile device and validates the secret key.

The DC-to-DC buck converter regulates the battery voltage to the required operating voltage for the ESP32 and relay module.

The relay module controls the solenoid lock mechanism based on the authentication result.

---

# Project Structure

```bash
ESP32-BLE-Smart-EBike-Lock-System/
│
├── Code/
│   └── smart_lock.ino
│
├── Circuit_Diagram/
│   └── circuit_diagram.png
│
├── Images/
│   └── prototype.jpg
│
├── Videos/
│   └── demo.mp4
│
└── README.md
