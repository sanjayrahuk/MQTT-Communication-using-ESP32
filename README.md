# EXP 10 CLOUD-BASED DEVICE CONTROL USING MQTT AND WI-FI COMMUNICATION

## Aim

To control an electrical device remotely through a cloud platform using MQTT communication and a Wi-Fi module.

# Hardware / Software Tools Required

- Arduino UNO / ESP32 / ESP8266 Wi-Fi Module
- USB Cable
- PC/Laptop
- Arduino IDE
- Wi-Fi Network
- Relay Module
- LED / DC Load
- Breadboard
- Jumper Wires
- Cloud Platform such as Blynk or ThingSpeak
- MQTT Broker / MQTT Service

# Circuit Diagram

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/707cbbff-a867-45bf-92d3-786a3244d2ab" />


# Procedure

## Step 1: Assemble the Circuit

1. Place the microcontroller board, Wi-Fi module, relay module, LED/load, and breadboard on the workbench.
2. Connect the required power supply and GND connections.
3. Ensure that the Wi-Fi module and controller operate at their specified voltage levels.

## Step 2: Connect the Relay Module

1. Connect the **VCC** of the relay module to the appropriate power supply.
2. Connect the **GND** of the relay module to **GND**.
3. Connect the relay input pin to a suitable digital output pin of the controller.
4. Connect the LED or other low-voltage load through the relay contacts.
5. Do not connect mains voltage directly during laboratory testing unless the setup is specifically designed and supervised for it.

## Step 3: Configure Wi-Fi Communication

1. Connect the Wi-Fi-enabled controller to the required Wi-Fi network.
2. Enter the Wi-Fi SSID and password in the program.
3. Verify that the device obtains an IP address.
4. Confirm that the controller can establish an Internet connection.

## Step 4: Configure the Cloud / MQTT Platform

1. Create an account on the selected cloud platform such as **Blynk** or **ThingSpeak**, as applicable.
2. Configure the required device, virtual control, channel, or dashboard.
3. Configure the MQTT broker/server details.
4. Set the MQTT topic used for ON/OFF control.
5. Define the payload values, for example:
   - `ON` – Switch device ON
   - `OFF` – Switch device OFF

## Step 5: Write and Upload the Program

1. Open the Arduino IDE.
2. Include the required Wi-Fi and MQTT libraries.
3. Enter the Wi-Fi credentials and MQTT broker details.
4. Configure the relay output pin.
5. Establish a connection with the Wi-Fi network.
6. Establish a connection with the MQTT broker.
7. Subscribe to the required MQTT topic.
8. Write the callback function to process ON/OFF commands.
9. Verify the program using the **Verify** button.
10. Upload the program to the controller.

## Step 6: Execute the Program

1. Power ON the controller and Wi-Fi module.
2. Open the configured cloud dashboard or MQTT client.
3. Send an **ON** command through the configured MQTT topic.
4. Observe that the relay activates and the connected device turns ON.
5. Send an **OFF** command.
6. Observe that the relay deactivates and the connected device turns OFF.
7. Monitor the Serial Monitor to verify MQTT connection and received commands.

## Step 7: Verify the Output

1. Check whether the controller successfully connects to Wi-Fi.
2. Verify the MQTT broker connection.
3. Send an ON command from the cloud platform.
4. Observe the device switching ON.
5. Send an OFF command from the cloud platform.
6. Observe the device switching OFF.
7. Record the commands and corresponding device states.

# Program
```py
#define BLYNK_TEMPLATE_ID "TMPL3G6VfRBpc"
#define BLYNK_TEMPLATE_NAME "Ultra Sonic Sensor"
#define BLYNK_AUTH_TOKEN "RhQ77Nuz1lSRGpPvrKx86JLEsyzrn__4"

#include <WiFiS3.h>
#include <BlynkSimpleWifi.h>

char ssid[] = "Baranikumar's Galaxy S24+";
char pass[] = "baranikumar5116";

// HC-SR04
#define TRIG_PIN 9
#define ECHO_PIN 8

// Distance at which object is detected
#define DISTANCE_LIMIT 40

BlynkTimer timer;

// Switch state from Blynk
int ledSwitch = 0;

// --------------------------------------------------
// BLYNK SWITCH - V2
// --------------------------------------------------
BLYNK_WRITE(V2)
{
  ledSwitch = param.asInt();

  if (ledSwitch == 1)
  {
    digitalWrite(LED_BUILTIN, HIGH);
    Serial.println("Blynk Switch ON - LED ON");
  }
  else
  {
    digitalWrite(LED_BUILTIN, LOW);
    Serial.println("Blynk Switch OFF - LED OFF");
  }
}

// --------------------------------------------------
// ULTRASONIC SENSOR
// --------------------------------------------------
void checkDistance()
{
  // Trigger ultrasonic sensor
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);

  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);

  digitalWrite(TRIG_PIN, LOW);

  // Read echo
  long duration = pulseIn(ECHO_PIN, HIGH, 15000);

  // No echo
  if (duration == 0)
  {
    Serial.println("No Echo Received");

    Blynk.virtualWrite(V1, 0);
    Blynk.virtualWrite(V3, 0);

    return;
  }

  // Calculate distance in cm
  float distance_cm = (duration * 0.0343) / 2.0;

  // Display distance
  Serial.print("Distance = ");
  Serial.print(distance_cm, 1);
  Serial.println(" cm");

  // Send distance to Gauge V1
  Blynk.virtualWrite(V1, distance_cm);

  // ------------------------------------------------
  // OBJECT DETECTION
  // ------------------------------------------------

  if (distance_cm <= DISTANCE_LIMIT)
  {
    // Object is close
    Blynk.virtualWrite(V3, 1);

    Serial.println("Object Detected");

    // Automatic LED ON
    digitalWrite(LED_BUILTIN, HIGH);

    // Update Blynk switch
    Blynk.virtualWrite(V2, 1);

    ledSwitch = 1;
  }
  else
  {
    // No object close
    Blynk.virtualWrite(V3, 0);

    Serial.println("No Object Nearby");

    // LED OFF
    digitalWrite(LED_BUILTIN, LOW);

    // Update Blynk switch
    Blynk.virtualWrite(V2, 0);

    ledSwitch = 0;
  }
}

// --------------------------------------------------
// SETUP
// --------------------------------------------------
void setup()
{
  Serial.begin(115200);

  // HC-SR04
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  // LED
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LED_BUILTIN, LOW);

  // Connect Blynk
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  // Check ultrasonic every 200 ms
  timer.setInterval(200L, checkDistance);
}

// --------------------------------------------------
// LOOP
// --------------------------------------------------
void loop()
{
  Blynk.run();
  timer.run();
}
```




# Observation
<img width="1918" height="1140" alt="Screenshot 2026-08-21 111824" src="https://github.com/user-attachments/assets/3211168e-3283-4adf-b421-024c70fcc94d" />

<img width="1919" height="1138" alt="Screenshot 2026-08-21 111842" src="https://github.com/user-attachments/assets/5413e6a3-f5fa-4156-9a45-a2ea69323872" />

<img width="1920" height="1200" alt="Screenshot 2026-08-21 111751" src="https://github.com/user-attachments/assets/83ebce57-1f3d-45c1-afd4-85329ee2fb7f" />

<img width="1920" height="1200" alt="Screenshot 2026-08-21 111731" src="https://github.com/user-attachments/assets/52323118-f0a6-429d-a401-1f2600207af4" />

# Result

The **cloud-based device control system was successfully implemented using MQTT and Wi-Fi communication**. The device was remotely controlled by sending ON/OFF commands through the MQTT communication channel. The experiment demonstrated the use of **IoT cloud connectivity, MQTT messaging, Wi-Fi communication, and remote device control**.
