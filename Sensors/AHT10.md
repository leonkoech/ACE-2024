
---

# Connecting AHT10 Sensor to ESP32

## Overview

<table>

  <tr>
    <td>
      The AHT10 is a digital temperature and humidity sensor that communicates with microcontrollers over the I2C interface. It provides accurate readings of temperature and humidity with minimal power consumption, making it ideal for environmental monitoring in IoT projects.
    </td>
     <td>
      <img src="https://ezmation.com/101-large_default/aht10-temperature-and-humidity-sensor-i2c.jpg" alt="AHT10" width="1200px">
    </td>
  </tr>

</table>

### Components Required

- **ESP32 Development Board**
- **AHT10 Temperature and Humidity Sensor**
- **Jumper Wires**
- **Breadboard (optional)**

### Pinout for AHT10

The AHT10 sensor typically has four pins:

1. **VCC**: Power supply (3.3V)
2. **GND**: Ground
3. **SDA**: I2C data line
4. **SCL**: I2C clock line

### Connections

To connect the AHT10 sensor to the ESP32, follow these steps:

1. **VCC to 3.3V**: Connect the VCC pin of the AHT10 sensor to the 3.3V pin on the ESP32.
2. **GND to GND**: Connect the GND pin of the AHT10 sensor to a GND pin on the ESP32.
3. **SDA to GPIO 21**: Connect the SDA pin of the AHT10 sensor to GPIO 21 on the ESP32 (this is the default I2C data pin).
4. **SCL to GPIO 22**: Connect the SCL pin of the AHT10 sensor to GPIO 22 on the ESP32 (this is the default I2C clock pin).

### Wiring Diagram

Here is a simple wiring diagram for connecting the AHT10 to the ESP32:

```
   AHT10      ESP32
   ------    ------
   VCC   ->  3.3V
   GND   ->  GND
   SDA   ->  GPIO 21 (I2C Data)
   SCL   ->  GPIO 22 (I2C Clock)
```

### Installing the AHT10 Library

Before using the AHT10 with the ESP32, you need to install the AHT10 library in the Arduino IDE:

1. Open Arduino IDE.
2. Go to **Sketch > Include Library > Manage Libraries**.
3. In the Library Manager, search for "AHT10".
4. Install the "AHT10" library by **Environ**.

### Example Code

Here’s an example code to read temperature and humidity from the AHT10 sensor and display the values on the Serial Monitor:

```cpp
#include <Wire.h>
#include <AHT10.h>

AHT10 aht10;

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22);  // Initialize I2C with SDA on GPIO 21 and SCL on GPIO 22

  if (!aht10.begin()) {
    Serial.println("Could not find AHT10 sensor!");
    while (1);
  }

  Serial.println("AHT10 sensor found and initialized.");
}

void loop() {
  float temperature = aht10.readTemperature();
  float humidity = aht10.readHumidity();

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" °C");

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");

  delay(2000);  // Delay between readings
}
```

### Uploading and Testing

1. Connect your ESP32 to the computer using a USB cable.
2. Upload the code to the ESP32 using the Arduino IDE.
3. Open the Serial Monitor (`Tools > Serial Monitor`) to view the temperature and humidity readings from the AHT10 sensor.

### Troubleshooting

- **No I2C Communication**: Ensure the SDA and SCL lines are connected correctly to GPIO 21 and GPIO 22, respectively.
- **No Sensor Found**: Verify the wiring and make sure the AHT10 library is installed correctly.

