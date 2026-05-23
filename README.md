ESP32 BLE Smart E-Bike Lock System

An IoT-based smart e-bike locking system developed using ESP32, BLE communication, Embedded C, relay modules, and solenoid lock mechanisms for secure Bluetooth-enabled authentication and locking control.

---

Project Overview

Traditional bicycle locks can be physically broken or duplicated easily.  
This project introduces a Bluetooth-enabled smart locking system for electric bicycles using ESP32 and BLE authentication.

The user can securely lock or unlock the e-bike using a private secret key through Bluetooth communication from a mobile device.

The system was developed during an internship at Leakamp for practical e-bike security implementation.

---

Features

- Bluetooth-enabled lock and unlock system
- Secure authentication using secret key validation
- ESP32 BLE communication
- Solenoid-based locking mechanism
- Automatic re-locking after timeout
- Low-voltage hardware control using relay modules
- Real-time BLE notification response system

---

 Technologies Used

- Embedded C
- ESP32 WROOM
- Bluetooth Low Energy (BLE)
- Arduino IDE
- IoT-based Embedded Systems

---

 Components Used

- ESP32 WROOM
- Relay Module
- Solenoid Lock
- DC-to-DC Buck Converter
- Lithium Battery
- Connecting Wires
- E-bike Battery Supply

---

Working Principle

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

Circuit Functionality

The ESP32 receives Bluetooth signals from the mobile device and validates the secret key.

The DC-to-DC buck converter regulates the battery voltage to the required operating voltage for the ESP32 and relay module.

The relay module controls the solenoid lock mechanism based on the authentication result.

---
Code
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLE2902.h>

#define SERVICE_UUID "6E400001-B5A3-F393-E0A9-E50E24DCCA9E"
#define CHARACTERISTIC_UUID_RX "6E400002-B5A3-F393-E0A9-E50E24DCCA9E"
#define CHARACTERISTIC_UUID_TX "6E400003-B5A3-F393-E0A9-E50E24DCCA9E"
#define AUTHORIZED_SECRET_KEY "4362265f-750c-475d-87b5-d6170ea990ee"
#define SOLENOID_PIN 25

bool deviceConnected = false;
bool isAuthorized = false;
bool isUnlocked = false;
String receivedData = "";
unsigned long unlockTime = 0;
BLECharacteristic *pTxCharacteristic;
BLEServer *pServer;

class MyServerCallbacks : public BLEServerCallbacks {
  void onConnect(BLEServer* pServer) {
    deviceConnected = true;
    isAuthorized = false;
    receivedData = "";
    Serial.println("Device connected");
  }
  
  void onDisconnect(BLEServer* pServer) {
    deviceConnected = false;
    isAuthorized = false;
    isUnlocked = false;
    digitalWrite(SOLENOID_PIN, LOW);
    pServer->getAdvertising()->start();
    Serial.println("Device disconnected - Solenoid secured");
  }
};

class MyCallbacks: public BLECharacteristicCallbacks {
  void onWrite(BLECharacteristic *pCharacteristic) {
    uint8_t* data = pCharacteristic->getData();
    size_t length = pCharacteristic->getLength();

    if (length > 0) {
      String chunk = "";
      for (int i = 0; i < length; i++) {
        chunk += (char)data[i];
      }
      receivedData += chunk;

      if (receivedData.indexOf("{") >= 0 && receivedData.indexOf("}") > receivedData.indexOf("{")) {
        int secretKeyPos = receivedData.indexOf("\"secretKey\":\"");
        if (secretKeyPos >= 0) {
          secretKeyPos += 13;
          int secretKeyEndPos = receivedData.indexOf("\"", secretKeyPos);
          if (secretKeyEndPos > secretKeyPos) {
            String secretKey = receivedData.substring(secretKeyPos, secretKeyEndPos);

            if (secretKey == AUTHORIZED_SECRET_KEY) {
              isAuthorized = true;
              isUnlocked = true;
              unlockTime = millis();
              digitalWrite(SOLENOID_PIN, HIGH);
              Serial.println("Authorized - Solenoid Powered ON");

              String response = "{\"status\":\"authorized\",\"secretKey\":\"" + secretKey + "\",\"solenoidStatus\":\"unlocked\"}";
              pTxCharacteristic->setValue(response.c_str());
              pTxCharacteristic->notify();
            } else {
              isAuthorized = false;
              isUnlocked = false;
              digitalWrite(SOLENOID_PIN, LOW);
              Serial.println("Unauthorized - Solenoid remains OFF");

              String response = "{\"status\":\"unauthorized\",\"message\":\"Invalid secret key\",\"solenoidStatus\":\"locked\"}";
              pTxCharacteristic->setValue(response.c_str());
              pTxCharacteristic->notify();
            }
          }
        }
        receivedData = "";
      }
    }
  }
};

void setup() {
  Serial.begin(115200);
  Serial.println("Starting BLE Smart Lock...");

  pinMode(SOLENOID_PIN, OUTPUT);
  digitalWrite(SOLENOID_PIN, LOW);

  BLEDevice::init("smartlock_1");
  pServer = BLEDevice::createServer();
  pServer->setCallbacks(new MyServerCallbacks());

  BLEService *pService = pServer->createService(SERVICE_UUID);
  pTxCharacteristic = pService->createCharacteristic(CHARACTERISTIC_UUID_TX, BLECharacteristic::PROPERTY_NOTIFY);
  pTxCharacteristic->addDescriptor(new BLE2902());

  BLECharacteristic *pRxCharacteristic = pService->createCharacteristic(CHARACTERISTIC_UUID_RX, BLECharacteristic::PROPERTY_WRITE);
  pRxCharacteristic->setCallbacks(new MyCallbacks());

  pService->start();
  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->addServiceUUID(SERVICE_UUID);
  pServer->getAdvertising()->start();

  Serial.println("BLE Smart Lock ready - Solenoid secured (OFF)");
}

void loop() {
  unsigned long currentTime = millis();

  if (isUnlocked && (currentTime - unlockTime > 10000)) {
    isUnlocked = false;
    digitalWrite(SOLENOID_PIN, LOW);
    Serial.println("Auto-locked - Solenoid power OFF");

    if (deviceConnected) {
      String response = "{\"status\":\"locked\",\"message\":\"Auto-locked after 10 seconds\",\"solenoidStatus\":\"locked\"}";
      pTxCharacteristic->setValue(response.c_str());
      pTxCharacteristic->notify();
    }
  }

  if (!isUnlocked) {
    digitalWrite(SOLENOID_PIN, LOW);
  }

  delay(100);
}

Project Structure

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


