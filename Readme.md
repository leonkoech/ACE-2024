
---

## Documentation

This documentation provides an overview of the sensors and microcontrollers available. Here's a detailed overview as how you can locate specific documentation. 
This documentation also details how you would connect the ESP32 (a microcontroller) to a VR headset

### Microcontrollers
- [Arduino Nano](./MicroControllers/ArduinoNano.md)
- [ESP32](./MicroControllers/esp32.md)
- [ESP32-CAM](./MicroControllers/Esp32Cam.md)

### Sensors
- [AHT10 Temperature and Humidity Sensor](./Sensors/AHT10.md)
- [HRS04 Ultrasonic Sensor](./Sensors/HRS04.md)
- [MAX30102 Pulse Oximeter and Heart Rate Sensor](./Sensors/MAX30102.md)
- [MPU6050 Accelerometer and Gyroscope](./Sensors/MPU6050.md)

---


## Connecting MCUs to a VR headset

There are two ways of connecting an MCU to a VR headset;
- Through WiFi (Easy)
- Through USB (Advanced)

### Through WiFi
The two devices should be connected to the same WiFi network. 
In your ESP32 C code you need to have the connection. 

```cpp
const char *ssid = "YOUR_SSID";
const char *password = "YOUR_PASSWORD";
const int port = 1234; // this is your port, you'll need it 

const int input_length = 3;

WiFiServer server(port);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED ) {
    delay(1000);
    Serial.print("Connecting to WiFi... ");
    Serial.println(WiFi.status());
  }

  Serial.println("Connected to WiFi");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  server.begin();
}

void loop(){
  WiFiClient client = server.available();

  if (client) {
    Serial.println("Client connected");

    while (client.connected()) {

      String data_to_send = Get_sensor_data(); // make sure to implement this function, for simplicity your data can be a string 
      client.println(data_to_send);  
      Serial.println(data_to_send);

      delay(100); // Adjust the delay as needed
    }

    Serial.println("Client disconnected");
  }
}

```

In your Unity Code you should have something similar to this:

```C#
using System.Net;
using System.Net.Sockets;
using System.Text;
using UnityEngine;

public class TCPServer : MonoBehaviour
{
    private TcpListener tcpListener;
    private TcpClient tcpClient;
    private NetworkStream networkStream;

    public int port = 1234; //same port as referenced in the ESP32 code

    void Start()
    {
        StartServer();
    }

    void StartServer()
    {
        tcpListener = new TcpListener(IPAddress.Any, port);
        tcpListener.Start();
        Debug.Log("Server started and listening on port " + port);
        tcpListener.BeginAcceptTcpClient(OnClientConnected, tcpListener);
    }

    void OnClientConnected(IAsyncResult result)
    {
        tcpClient = tcpListener.EndAcceptTcpClient(result);
        networkStream = tcpClient.GetStream();
        Debug.Log("Client connected");

        // Begin reading data
        byte[] buffer = new byte[1024];
        networkStream.BeginRead(buffer, 0, buffer.Length, OnDataReceived, buffer);
    }

    void OnDataReceived(IAsyncResult result)
    {
        byte[] buffer = (byte[])result.AsyncState;
        int bytesRead = networkStream.EndRead(result);
        string receivedText = Encoding.UTF8.GetString(buffer, 0, bytesRead);
        Debug.Log("Received: " + receivedText);

        // Continue reading data
        networkStream.BeginRead(buffer, 0, buffer.Length, OnDataReceived, buffer);
    }

    void OnApplicationQuit()
    {
        networkStream.Close();
        tcpClient.Close();
        tcpListener.Stop();
    }
}
```

Note: Since you'll need the IPAddress of the ESP32, my advice is to use the mobile Hotspot on your computer. This way, you can get the IP address of the ESP32.   
Another advice, use the IP address in unity scripting as a public variable. This way you can build it before compiling. You can also have it as a input in your scene since only a small thing changes. 

### Through USB
Please Note that this is more complex. This is because we communicate through the hardware layer of the Head Mounted Perceptual Device (HMD)

The setup comprises of three settings;
- Android Module
- Unity Scripting
- ESP32 Code
There are two ways here, one is through rx/tx pins on the ESP32 and the other is through the regular USB on the ESP 32. This will be the last section.  

#### Android
For step one, you need android studio for this. Open android studio and create a module. 

This android module that offers communication between USB and Android via serial communication. 
Intention of the build was having microcontrollers communicate with HMDs via serial, instead of the highly suggested Wireless communication. 

![Screenshot 2024-04-03 163348](https://github.com/leonkoech/USB-Serial-Android-ESP32/assets/39020723/bde7cf76-dff5-42ff-97ce-092ad6770d86)

Outside you application tags, just above them, make sure you have this code for permissions

```xml
 <uses-permission android:name="android.permission.USB_PERMISSION" />
    <uses-feature android:name="android.hardware.usb.host" />
```

Create a `usbconnection.java` file and give it this code;

```java
package com.DefaultCompany.VRAware;
import static androidx.core.content.ContextCompat.getSystemService;
import static androidx.core.content.ContextCompat.registerReceiver;

import android.Manifest;
import android.app.PendingIntent;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.hardware.usb.UsbConstants;
import android.hardware.usb.UsbDevice;
import android.hardware.usb.UsbDeviceConnection;
import android.hardware.usb.UsbEndpoint;
import android.hardware.usb.UsbInterface;
import android.hardware.usb.UsbManager;
import android.os.strictmode.UntaggedSocketViolation;
import android.util.Log;
import android.content.Context;
import android.content.pm.PackageManager;

import java.nio.charset.StandardCharsets;
import java.util.HashMap;

public class usbconnection {
    private static final String TAG = "USBConnection";
    private static final int TIMEOUT = 1000;

    private UsbManager usbManager;
    private Context context;

    private  UsbEndpoint endpointIn;
    private UsbInterface usbInterface;

    public void setContext(Context ctx) {
        context = ctx;
        usbManager = (context != null) ? (UsbManager) context.getSystemService(Context.USB_SERVICE) : null;
    }

    public boolean isContextSet() {
        return context != null;
    }

//    public boolean get(){return usbManager!=null; }

    public String testConnection() {
        return "output";
    }
    public String[] getConnectedDevices() {
        HashMap<String, UsbDevice> deviceList = usbManager.getDeviceList();
        String[] connectedDevices = new String[deviceList.size()];
        int i = 0;
        for (UsbDevice device : deviceList.values()) {
            connectedDevices[i++] = device.getDeviceName();
        }
        return connectedDevices;
    }
    public UsbDevice[] getConnectedDevicesObject() {
        HashMap<String, UsbDevice> deviceList = usbManager.getDeviceList();
        UsbDevice[] connectedDevices = new UsbDevice[deviceList.size()];
        int i = 0;
        for (UsbDevice device : deviceList.values()) {
            connectedDevices[i++] = device;
        }
        return connectedDevices;
    }
    public UsbDevice findUsbDeviceByName(String deviceName) {
        HashMap<String, UsbDevice> deviceList = usbManager.getDeviceList();

        for (UsbDevice device : deviceList.values()) {
            if (device.getDeviceName().equals(deviceName)) {
                return device;
            }
        }

        return null; // Device not found
    }
    public boolean sendData(byte[] data, UsbDevice usbDevice) {
        UsbDeviceConnection usbConnection = usbManager.openDevice(usbDevice);
        if (usbConnection == null) {
            Log.e(TAG, "Failed to open USB device connection");
            return false;
        }

        UsbInterface usbInterface = usbDevice.getInterface(0);
        if (!usbConnection.claimInterface(usbInterface, true)) {
            Log.e(TAG, "Failed to claim interface");
            usbConnection.close();
            return false;
        }

        UsbEndpoint endpointOut = null;
        for (int i = 0; i < usbInterface.getEndpointCount(); i++) {
            UsbEndpoint endpoint = usbInterface.getEndpoint(i);
            if (endpoint.getType() == UsbConstants.USB_ENDPOINT_XFER_BULK && endpoint.getDirection() == UsbConstants.USB_DIR_OUT) {
                endpointOut = endpoint;
                break;
            }
        }

        if (endpointOut == null) {
            Log.e(TAG, "Could not find OUT endpoint");
            usbConnection.releaseInterface(usbInterface);
            usbConnection.close();
            return false;
        }

        int bytesSent = usbConnection.bulkTransfer(endpointOut, data, data.length, TIMEOUT);
        if (bytesSent >= 0) {
            Log.d(TAG, "Sent " + bytesSent + " bytes");
            return true;
        } else {
            Log.e(TAG, "Failed to write to USB device");
            usbConnection.releaseInterface(usbInterface);
            usbConnection.close();
            return false;
        }
    }

    private static final String ACTION_USB_PERMISSION =
            "com.android.example.USB_PERMISSION";
    private final BroadcastReceiver usbReceiver = new BroadcastReceiver() {
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            if (ACTION_USB_PERMISSION.equals(action)) {
                synchronized (this) {
                    UsbDevice device = (UsbDevice)intent.getParcelableExtra(UsbManager.EXTRA_DEVICE);

                    if (intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false)) {
                        if(device != null){
                            // call method to set up device communication
//                            setupDeviceCommunication(device);
                        }
                    }
                    else {
                        Log.d(TAG, "permission denied for device " + device);
                    }
                }
            }
        }
    };

    // Function to register the BroadcastReceiver
    public void registerUSBReceiver(Context context) {
        IntentFilter filter = new IntentFilter(ACTION_USB_PERMISSION);
        context.registerReceiver(usbReceiver, filter);
    }

    // Function to unregister the BroadcastReceiver
    public void unregisterUSBReceiver(Context context) {
        context.unregisterReceiver(usbReceiver);
    }

    // Function to request USB permission
    public String requestUSBPermission(Context context, UsbDevice device) {
        PendingIntent permissionIntent = PendingIntent.getBroadcast(context, 0, new Intent(ACTION_USB_PERMISSION), PendingIntent.FLAG_IMMUTABLE);
        UsbManager usbManager = (UsbManager) context.getSystemService(Context.USB_SERVICE);
        usbManager.requestPermission(device, permissionIntent);
        return "function called";
    }
    public String checkPermission(UsbDevice usbDevice){
        boolean checkPermission = usbManager.hasPermission(usbDevice);
        if(!checkPermission){
            return "permission absent" + "vendor ID: " + usbDevice.getVendorId() + "device ID" + usbDevice.getProductId();
        }
        return "Permission present Vendor ID:" + usbDevice.getVendorId() + "Device ID:" + usbDevice.getProductId();
    }
    public String receiveDataAsString(UsbDevice usbDevice) {
        UsbDeviceConnection usbConnection = usbManager.openDevice(usbDevice);
        checkPermission(usbDevice);
        if (usbConnection == null) {
            Log.e(TAG, "Failed to open USB device connection");
            return "Failed to open USB device connection";
        }

        UsbInterface usbInterface = usbDevice.getInterface(0);
        if (!usbConnection.claimInterface(usbInterface, true)) {
            Log.e(TAG, "Failed to claim interface");
            usbConnection.close();
            return "Failed to claim interface";
        }

        UsbEndpoint endpointIn = null;
        for (int i = 0; i < usbInterface.getEndpointCount(); i++) {
            UsbEndpoint endpoint = usbInterface.getEndpoint(i);
            if (endpoint.getType() == UsbConstants.USB_ENDPOINT_XFER_BULK && endpoint.getDirection() == UsbConstants.USB_DIR_IN) {
                endpointIn = endpoint;
                break;
            }
        }

        if (endpointIn == null) {
            Log.e(TAG, "Could not find IN endpoint");
            usbConnection.releaseInterface(usbInterface);
            usbConnection.close();
            return "Could not find IN endpoint";
        }

        byte[] buffer = new byte[12];
        int bytesRead = usbConnection.bulkTransfer(endpointIn, buffer, buffer.length, TIMEOUT);
        if (bytesRead >= 0) {
            Log.d(TAG, "Received " + bytesRead + " bytes");
            String receivedData = new String(buffer, 0, bytesRead, StandardCharsets.UTF);
            return receivedData;
        } else {
            Log.e(TAG, "Failed to read from USB device");
            usbConnection.releaseInterface(usbInterface);
            usbConnection.close();
            return "Failed to read from USB device";
        }
    }

    public UsbDeviceConnection establishConnection(UsbDevice usbDevice){
        UsbDeviceConnection usbConnection = usbManager.openDevice(usbDevice);
        if (usbConnection == null) {
            Log.e(TAG, "Failed to open USB device connection");
            return null;
        }

        usbInterface = usbDevice.getInterface(0);
        if (!usbConnection.claimInterface(usbInterface, true)) {
            Log.e(TAG, "Failed to claim interface");
            usbConnection.close();
            return null;
        }
         endpointIn =  usbInterface.getEndpoint(0);;


        if (endpointIn == null) {
            Log.e(TAG, "Could not find IN endpoint");
            usbConnection.releaseInterface(usbInterface);
            usbConnection.close();
            return null;
        }
        return usbConnection;
    }
    public String receiveData(UsbDeviceConnection usbConnection, int buffer_size) {
        byte[] buffer = new byte[buffer_size];
        if(usbConnection != null){
            int bytesRead = usbConnection.bulkTransfer(endpointIn, buffer, buffer.length, TIMEOUT);
            if (bytesRead > 0) {
                Log.d(TAG, "Received " + bytesRead + " bytes");
                String receivedData = new String(buffer, 0, bytesRead, StandardCharsets.UTF_8);
                return receivedData;
            } else if (bytesRead == 0) {
                // Transfer was successful, but no data was read
                Log.d(TAG, "Transfer successful, but no data read");
                return "No Data"; // Or handle this case according to your application's logic
            } else {
                // An error occurred during the transfer
                Log.e(TAG, "Failed to read from USB device");
                usbConnection.releaseInterface(usbInterface);
                usbConnection.close();
                return "null";
            }
        }
        else{
            return "isbconnection is null";
        }

    }
}
```

After creating the module, you'll have to build the module and receive an apk. Import this apk into your Unity project; 
- Copy the Android Library Project to your Unity Project’s Assets folder.
- If the Android Library Project root folder doesn’t use the .androidlib extension, add this extension to the folder. For example, mylibrary.androidlib.

#### Unity Scripting 

In your Unity Code you'll need this Script. Name is what you like


```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System.Text;
using TMPro;
using System;
using System.Threading;
using System.Threading.Tasks;

// using com.DefaultCompany.VRAware; 
public class USBManager : MonoBehaviour
{
    public TextMeshProUGUI textMesh;
    private AndroidJavaObject usbCommunicationInstance;
    private AndroidJavaObject deviceObject;
    private AndroidJavaObject deviceObjectConnection;
    private bool connectionEstablished = false;
    private bool activateVRAware;

    private AndroidJavaObject endpointIn;
     public TMP_Text sensorStatus;


    void Start()
    {
        InitializeAndroidPlugin();
        setContext();
    }

    void Update(){
        if (connectionEstablished){
            UpdateUIText("established connection");
            DataFetchLoop(deviceObjectConnection);
        }
    }

    public void setContext()
    {
        AndroidJavaClass unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
        AndroidJavaObject currentActivity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity");
        AndroidJavaObject applicationContext = currentActivity.Call<AndroidJavaObject>("getApplicationContext");

        if (usbCommunicationInstance != null)
        {
            usbCommunicationInstance.Call("setContext", applicationContext);
            bool isContext = usbCommunicationInstance.Call<bool>("isContextSet");
            if (!isContext)
            {
                UpdateUIText("context not set");
            }
            else
            {
                UpdateUIText("context set");
            }
        }
        else{
             UpdateUIText("usb Communication Instance not found");
        }
    }

    public void InitializeAndroidPlugin()
    {
        UpdateUIText("finding Devices");

        usbCommunicationInstance = new AndroidJavaObject("com.DefaultCompany.VRAware.usbconnection");

        if (usbCommunicationInstance == null)
        {
            UpdateUIText("usbCommunicationInstance ==  null");
        }
        else{
             UpdateUIText("usbCommunicationInstance !=  null");
        }
    }
     void DataFetchLoop(AndroidJavaObject currentDeviceConnection)
    {
        if (currentDeviceConnection != null )
        {
            try
            {
                string receivedData = usbCommunicationInstance.Call<string>("receiveDataTest", currentDeviceConnection, endpointIn);
                
                
                if(activateVRAware){
                        // do stuff with received data
                        UpdateUIText(!string.IsNullOrEmpty(receivedData) ? "Received data: " + receivedData : "Received data is empty");
                }
            }
            catch (Exception e)
            {
                UpdateUIText("error: " + e.Message );
            }
        }
        else
        {
            UpdateUIText("Connection or Endpoint is null");
        }
    }


    public string requestPermission(AndroidJavaObject context, AndroidJavaObject usbDevice)
    {
        return usbCommunicationInstance.Call<string>("requestUSBPermission", context, usbDevice); // call the android function
    }

    public void connectUSB()
    {
        if (textMesh != null)
        {
            if (usbCommunicationInstance != null)
            {
                AndroidJavaObject[] Devices = usbCommunicationInstance.Call<AndroidJavaObject[]>("getConnectedDevicesObject");
                if (Devices.Length > 0)
                {
                    UpdateUIText("devices found" + Devices.Length.ToString());
                    AndroidJavaObject currentDevice = Devices[0];
                    AndroidJavaClass unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
                    AndroidJavaObject currentActivity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity");
                    AndroidJavaObject applicationContext = currentActivity.Call<AndroidJavaObject>("getApplicationContext");
                    endpointIn = usbCommunicationInstance.Call<AndroidJavaObject>("getEndpointInTest");
                    UpdateUIText(requestPermission(applicationContext, currentDevice));
                    deviceObject = currentDevice;
                    deviceObjectConnection = establishUSBConnection(deviceObject);
                    if (deviceObjectConnection != null)
                    {
                        
                        if (endpointIn != null)
                        {
                            UpdateUIText("established connection");
                            connectionEstablished = true;
                            sensorStatus.text = "USB Connected";
                        }
                        else
                        {
                            UpdateUIText("IN endpoint not found");
                        }
                    }
                    
                    else
                    {
                        UpdateUIText("Connection not established");
                    }
                }
            }
            else
            {
                Debug.LogError("usbCommunication object is null!");
                UpdateUIText("usbCommunication object is null!");
            }
        }
        else
        {
            UpdateUIText("Text Mesh component is not assigned!");
            Debug.LogWarning("Text Mesh component is not assigned!");
        }
    }

    public string[] GetConnectedDevices()
    {
        if (usbCommunicationInstance != null)
        {
            return usbCommunicationInstance.Call<string[]>("getConnectedDevices");
        }
        else
        {
            Debug.LogError("usbCommunication object is null!");
            return null;
        }
    }

    public AndroidJavaObject establishUSBConnection(AndroidJavaObject currentDevice)
    {
        return usbCommunicationInstance.Call<AndroidJavaObject>("establishConnectionTest", currentDevice);
    }

    public AndroidJavaObject FindDevice(string device_name)
    {
        UpdateUIText("finding device based on name" + device_name);
        return usbCommunicationInstance.Call<AndroidJavaObject>("findUsbDeviceByName", device_name);
    }

    private string ByteToString(byte[] byteArray)
    {
        string str = Encoding.UTF8.GetString(byteArray);
        return str;
    }

  
}
```

For context the above code calls the android code we stated earlier; `DataFetchLoop()` should be the main function.

#### ESP32
Through normal USB or RX TX. For RX TX you'll need a USB cable that exposes these wires and connect the exposed wires to the pins on the ESP32 and the usb to the HMD. 
the code will be the same

```C
#include <HardwareSerial.h>

HardwareSerial SerialUSB(1);

void setup() {
  Serial.begin(115200); // Starts the serial communication
 
    SerialUSB.begin(115200); 
}

void loop() {
    String data_to_send = Get_sensor_data(); //remember to define this
    Serial.println(data_to_send);
    SerialUSB.println(data_to_send);

}
```
