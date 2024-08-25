
---

# ESP32 Microcontroller 

Here’s the table of contents for the README:

---

## Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Getting Started](#getting-started)
   - [Requirements](#requirements)
   - [Installation](#installation)
4. [Pinout Diagram](#pinout-diagram)
5. [Writing and Uploading Code](#writing-and-uploading-code)
   - [Open Arduino IDE](#open-arduino-ide)
   - [Basic Blink Example](#basic-blink-example)
   - [Upload the Code](#upload-the-code)
   - [Monitor Output](#monitor-output)
6. [Wi-Fi Example](#wi-fi-example)
   - [Wi-Fi Connection Example](#wi-fi-connection-example)
   - [Replace with your Network Credentials](#replace-with-your-network-credentials)
   - [Upload and Monitor](#upload-and-monitor)
7. [Troubleshooting](#troubleshooting)
8. [Power Management](#power-management)
   - [Deep Sleep Example](#deep-sleep-example)
9. [Conclusion](#conclusion)
10. [More Tutorials](#more-tutorials)

---


## Overview

<table>

  <tr>
    <td>
      The ESP32 is a powerful and versatile microcontroller with built-in Wi-Fi and Bluetooth capabilities. It is ideal for IoT projects, embedded systems, and other applications requiring wireless communication and efficient processing power.
    </td>
     <td>
      <img src="https://jvlobo.com/content/images/2020/05/Esp32DevkitCNewChip1x_1600x-2_optimized.png" alt="ESP 32" width="1200px">
    </td>
  </tr>

</table>

## Key Features

- **Processor**: Dual-core 32-bit Xtensa LX6 CPU
- **Operating Voltage**: 3.3V
- **Flash Memory**: 4MB (expandable)
- **Wi-Fi**: 802.11 b/g/n
- **Bluetooth**: v4.2 BR/EDR and BLE
- **GPIO**: 36 pins (capacitive touch, ADC, DAC, I2C, SPI, PWM, UART)
- **Power Management**: Low-power modes (deep sleep, light sleep)

## Getting Started

### Requirements

- **ESP32 Development Board**
- **Micro-USB Cable**
- **Computer with Arduino IDE (or any other compatible IDE)**
- **ESP32 Board Manager installed in the IDE**
- **Wi-Fi network (for Wi-Fi-based projects)**

### Installation

1. **Install Arduino IDE:**
   - Download and install the latest version of the Arduino IDE from the [official website](https://www.arduino.cc/en/software).

2. **Install ESP32 Board in Arduino IDE:**
   - Open Arduino IDE.
   - Go to `File > Preferences`.
   - In the "Additional Board Manager URLs" field, add the following URL:
     ```
     https://dl.espressif.com/dl/package_esp32_index.json
     ```
   - Go to `Tools > Board > Boards Manager`.
   - Search for "ESP32" and click "Install".

3. **Set Up the ESP32 Board:**
   - Connect your ESP32 board to the computer using the micro-USB cable.
   - Go to `Tools > Board` and select your ESP32 board model (e.g., "ESP32 Dev Module").
   - Select the correct port under `Tools > Port`.

### Pinout Diagram

The diagram below shows the pin out of the ESP32. You use General Purpose Input Output (GPIO) Pins to receive input from sensors. 
 <img src="https://esp32io.com/images/tutorial/ESP-WROOM-32-Dev-Module-pinout.jpg" alt="ESP 32" width="500px">

### Writing and Uploading Code

1. **Open Arduino IDE:**
   - Create a new sketch or open an existing one.
   
2. **Basic Blink Example:**
   - Copy and paste the following code to test your ESP32:

     ```cpp
     void setup() {
       pinMode(2, OUTPUT); // Set GPIO 2 as an output pin
     }

     void loop() {
       digitalWrite(2, HIGH); // Turn on the LED
       delay(1000);           // Wait for a second
       digitalWrite(2, LOW);  // Turn off the LED
       delay(1000);           // Wait for a second
     }
     ```

3. **Upload the Code:**
   - Click the upload button (right arrow) in the Arduino IDE.
   - Wait for the code to compile and upload to the ESP32.

4. **Monitor Output:**
   - Open the Serial Monitor (`Tools > Serial Monitor`) to view any output from the ESP32.

### Wi-Fi Example

To connect your ESP32 to a Wi-Fi network:

1. **Wi-Fi Connection Example:**
   - Use the following code to connect to Wi-Fi:

     ```cpp
     #include <WiFi.h>

     const char* ssid = "your_SSID";
     const char* password = "your_PASSWORD";

     void setup() {
       Serial.begin(115200);
       WiFi.begin(ssid, password);

       while (WiFi.status() != WL_CONNECTED) {
         delay(1000);
         Serial.println("Connecting to WiFi...");
       }

       Serial.println("Connected to WiFi");
     }

     void loop() {
       // Your code here
     }
     ```

2. **Replace with your Network Credentials:**
   - Replace `"your_SSID"` and `"your_PASSWORD"` with your actual Wi-Fi network name and password.

3. **Upload and Monitor:**
   - Upload the code and open the Serial Monitor to see the connection status.

### Troubleshooting

- **ESP32 Not Detected:**
  - Ensure that the correct port is selected.
  - Try using a different USB cable or port.

- **Code Not Uploading:**
  - Press the BOOT button on the ESP32 board when starting the upload.

## Power Management

The ESP32 supports various power-saving modes, including deep sleep and light sleep. These can be useful for battery-powered applications.

### Deep Sleep Example

```cpp
void setup() {
  esp_sleep_enable_timer_wakeup(10 * 1000000); // Wake up after 10 seconds
  esp_deep_sleep_start();                      // Enter deep sleep
}

void loop() {
  // This will not be executed until wake-up
}
```

## Conclusion

The ESP32 is a versatile and powerful microcontroller suitable for various projects. Whether you're building an IoT device or experimenting with embedded systems, the ESP32 provides the capabilities you need.

For more information, visit the [ESP32 Documentation](https://docs.espressif.com/projects/esp-idf/en/latest/).

### More Tutorials

Here's a list of other tutorials that you can try with the pins. Some of these sensors are in the Lab with you. 
Also you can use the same code as arduino, you'll only change the pins most of the time.

- Getting Distance with the HR-SR04 [Tutorial Link](https://randomnerdtutorials.com/esp32-hc-sr04-ultrasonic-arduino/) 

---
