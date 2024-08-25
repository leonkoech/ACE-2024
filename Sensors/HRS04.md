
---

# Connecting HC-SR04 Ultrasonic Sensor to ESP32

## Overview



<table>

  <tr>
    <td>
      The HC-SR04 is an ultrasonic sensor that measures distance by sending out a pulse of ultrasound and measuring the time it takes to reflect back. It is widely used in robotics, obstacle detection, and distance measurement projects.
    </td>
     <td>
      <img src="https://robu.in/wp-content/uploads/2014/08/hc-sr04_ultrasonic_sensor_distance_measuring_module_1_.jpg" alt="HS04" width="1200px">
    </td>
  </tr>

</table>

### Components Required

- **ESP32 Development Board**
- **HC-SR04 Ultrasonic Sensor**
- **Jumper Wires**
- **Breadboard (optional)**

### Pinout for HC-SR04

The HC-SR04 sensor has four pins:

1. **VCC**: Power supply (5V)
2. **GND**: Ground
3. **TRIG**: Trigger pin (initiates the ultrasound pulse)
4. **ECHO**: Echo pin (receives the reflected pulse)

### Connections

To connect the HC-SR04 sensor to the ESP32, follow these steps:

1. **VCC to 5V**: Connect the VCC pin of the HC-SR04 sensor to the 5V pin on the ESP32.
2. **GND to GND**: Connect the GND pin of the HC-SR04 sensor to a GND pin on the ESP32.
3. **TRIG to GPIO 5**: Connect the TRIG pin of the HC-SR04 sensor to GPIO 5 on the ESP32.
4. **ECHO to GPIO 18**: Connect the ECHO pin of the HC-SR04 sensor to GPIO 18 on the ESP32.

### Wiring Diagram

Here is a simple wiring diagram for connecting the HC-SR04 to the ESP32:

```
   HC-SR04   ESP32
   -------   ------
   VCC   ->  5V
   GND   ->  GND
   TRIG  ->  GPIO 5
   ECHO  ->  GPIO 18
```

### Note

The HC-SR04 sensor operates at 5V, while the ESP32's GPIO pins operate at 3.3V. The ECHO pin on the HC-SR04 outputs a 5V signal, so you should use a voltage divider or level shifter to reduce this to 3.3V before connecting it to the ESP32.

### Example Code

Here’s an example code to read distance measurements from the HC-SR04 sensor and display the values on the Serial Monitor:

```cpp
#define TRIG_PIN 5
#define ECHO_PIN 18

void setup() {
  Serial.begin(115200);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
}

void loop() {
  // Send a 10us pulse to trigger the measurement
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Measure the duration of the pulse
  long duration = pulseIn(ECHO_PIN, HIGH);

  // Calculate the distance in centimeters
  float distance = (duration / 2.0) * 0.0343;

  // Display the distance on the Serial Monitor
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  delay(500);  // Delay between measurements
}
```

### Uploading and Testing

1. Connect your ESP32 to the computer using a USB cable.
2. Upload the code to the ESP32 using the Arduino IDE.
3. Open the Serial Monitor (`Tools > Serial Monitor`) to view the distance readings from the HC-SR04 sensor.

### Troubleshooting

- **No Distance Measurement**: Ensure the TRIG and ECHO pins are connected correctly. Check the wiring and make sure the sensor is receiving power.
- **Incorrect Readings**: Verify that the ECHO pin voltage is correctly stepped down to 3.3V before connecting to the ESP32.

---
