/*
  LED

  This example creates a Bluetooth® Low Energy peripheral with service that contains a
  characteristic to control an LED.

  The circuit:
  - Arduino MKR WiFi 1010, Arduino Uno WiFi Rev2 board, Arduino Nano 33 IoT,
    Arduino Nano 33 BLE, or Arduino Nano 33 BLE Sense board.

  You can use a generic Bluetooth® Low Energy central app, like LightBlue (iOS and Android) or
  nRF Connect (Android), to interact with the services and characteristics
  created in this sketch.

  This example code is in the public domain.
*/

#include <ArduinoBLE.h>

BLEService accelService("2eabb710-354b-4066-95af-d8c1d4723e48"); // Bluetooth® Low Energy LED Service

// Bluetooth® Low Energy LED Switch Characteristic - custom 128-bit UUID, read and writable by central
BLEByteCharacteristic acceleration("61811ad4-035a-4d1f-a60d-39828b887606", BLERead | BLEWrite | BLENotify);
BLEByteCharacteristic count("f681bdcf-33af-4278-acc5-46770b2a331d", BLERead | BLEWrite | BLENotify);
float threshold = 1.15; //g's
float timeThreshold = 500; //miliseconds
unsigned long timeOfCrossThreshold = 0;
float steps = 0.0;
float sampleSize = 20;
float values[2];

#include <Arduino_LSM9DS1.h>

void setup() {
  Serial.begin(9600);
  // begin initialization
  if (!BLE.begin()) {
    Serial.println("starting Bluetooth® Low Energy module failed!");
    while (1);
  }
  
  if (!IMU.begin()) {
    Serial.println("Failed to initialize IMU!");
    while (1);
  }

  // set advertised local name and service UUID:
  BLE.setLocalName("MovementTracker");
  BLE.setAdvertisedService(accelService);

  // add the characteristic to the service
  accelService.addCharacteristic(count);
  accelService.addCharacteristic(acceleration);
 
  // add service
  BLE.addService(accelService);

  // start advertising
  BLE.advertise();
}


void loop() {
  int stepPrint = 0;
  float magnitudePrint = 0;
  float currTime = 0;
  bool flag = false;
  // listen for Bluetooth® Low Energy peripherals to connect:
  BLEDevice central = BLE.central();

  // if a central is connected to peripheral:
  if (central && count.subscribed()) {
    Serial.print("Connected to central: ");
    // print the central's MAC address:
    Serial.println(central.address());
    
    while (central.connected()) {
      float x, y, z;
      if (IMU.accelerationAvailable()) {
        float reading = 0;
        for (int i=0; i < sampleSize; i++) {
          IMU.readAcceleration(x, y, z);
          reading += sqrt(x*x+y*y+z*z);
        }
      float magnitude = reading / sampleSize;
      if (magnitude > threshold && !flag) {
        if (millis()-timeOfCrossThreshold > timeThreshold) {
          
          Serial.println("Movement Detected!");
          timeOfCrossThreshold = millis();
          flag = true;
          steps = steps + 1;
        }     
      }
      else if (flag) {
          flag=false;
      }
      values[0] = magnitude;
      values[1] = steps;
      if(count.subscribed() && acceleration.subscribed()) {
        count.writeValue(steps);
        acceleration.writeValue((int)(magnitude * 100)); //Going to be multiplying by 100 so can send as an int
      }
    }
   }
    // when the central disconnects, print it out:
    Serial.print(F("Disconnected from central: "));
    Serial.println(central.address());

  }
}
