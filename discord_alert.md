---

# **Arduino Sensor Data Discord Notification System**

This project is a Python script that reads **Arduino sensor data** and sends notifications via **Discord webhooks** based on specific conditions. When the soil moisture level falls below a predefined threshold, it automatically sends an alert message.

---

## **1. Project Overview**

### **Features**

1. Reads sensor data via serial communication with Arduino.
2. Sends a notification message to Discord webhooks if **soil moisture levels** are low.
3. Transmits all sensor data to Discord every 10 seconds.
---

## **2. Full Code**

```python
import serial  # For Serial Communication
import requests  # For HTTP Requests
import time  # For Time Delays

# Configuration
WEBHOOK_URL = "please enter your webhook url"  # Enter Discord Webhook URL
SERIAL_PORT = "/dev/ttyUSB0"  # Check Arduino Serial Port (can be checked using ls /dev/tty*)
BAUD_RATE = 9600  # Set the Baud Rate to Match Arduino

# Send Message to Discord
def send_to_discord(message):
    payload = {"content": message}  # Discord Message Format
    try:
        response = requests.post(WEBHOOK_URL, json=payload)
        if response.status_code == 204:
            print("Message successfully sent to Discord!")
        else:
            print(f"Failed to send: {response.status_code} - {response.text}")
    except Exception as e:
        print(f"An error occurred: {e}")

# Read Arduino Data
def read_sensor_data():
    try:
        # Start Serial Communication
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=2) as ser:
            print("Connecting to Arduino...")
            time.sleep(2)  # Wait for Stable Connection
            while True:
                if ser.in_waiting > 0:
                    # Read Data
                    data = ser.readline().decode('utf-8').strip()  # Read Data from Arduino
                    print(f"Receive Sensor Data: {data}")

                    # Parse Data (Example: "Soil Moisture:75, Light Intensity:28, Humidity:15, Temperature:24")
                    try:
                        values = {kv.split(":")[0].strip(): int(kv.split(":")[1].strip())
                                  for kv in data.split(",")}

                        # Check Soil Moisture Data Conditions
                        soil_moisture = values.get("Soil Moisture", 0)  # Use English Keys for Sensor Data
                        if soil_moisture < 30:
                            alert_message = f"⚠️ Soil moisture is low! Please water the plants! Current moisture level: {soil_moisture}"
                            print(alert_message)
                            send_to_discord(alert_message)

                        # Send All Sensor Data to Discord
                        send_to_discord(f"Sensor data alert: {data}")

                    except Exception as parse_error:
                        print(f"Data parsing error: {parse_error}")

                    time.sleep(10)  # Send Data Every 10 Seconds

    except Exception as e:
        print(f"Serial communication error: {e}")

# Run Main Function
if __name__ == "__main__":
    read_sensor_data()
```

---

## **3. Setup and Usage**

### **Environment Setup**
1. **Install Python Libraries**  
   Install the Required Libraries:
   ```bash
   pip install pyserial requests
   ```

2. **Set the Discord Webhook URL**  
   Generate a webhook URL in your Discord channel and enter it in the code under WEBHOOK_URL.

3. **Check Arduino Serial Port**  
   Identify the port compatible with your system and set it in SERIAL_PORT:
   - **Linux/Mac**: `/dev/ttyUSB0` or `/dev/ttyACM0`  
   - **Windows**: `COM3`, `COM4` and so on

4. **Arduino Baud Rate**  
   Set the BAUD_RATE to match the Arduino code.

---

### **Execution**
```bash
python your_script_name.py
```

---

## **4. Description of Key Features**

1. **Serial Communication**  
   Read Arduino sensor data and decode it in UTF-8.

2. **Data Parsing**  
   Parse sensor data when it is in the following format:
   ```
   "토양수분: 250, 조도: 500, 온도: 25"
   ```

3. **Discord Notifications**  
   - **Low Soil Moisture Condition**: Send a warning message to Discord if SoilMoisture < 30.
   - **Periodic Data Transmission**: Notify Discord with all sensor data every 10 seconds.

---

## **5. Example Output**

### **Terminal Output**
```
Connecting to Arduino...
Sensor Data Received: Soil Moisture: 25, Light Intensity: 500, Temperature: 26
⚠️ Soil moisture is low! Please water the plants! Current moisture level: 25
Message Sent to Discord Successfully!
Sensor Data Notification: Soil Moisture: 25, Light Intensity: 500, Temperature: 26
```

### **Example Discord Message**
```
⚠️ Soil moisture is low! Please water the plants! Current moisture level: 25
Sensor Data Notification: Soil Moisture: 25, Light Intensity: 500, Temperature: 26
```

---

## **6. Important Notes**

1. **Discord Webhook Security**: Ensure the webhook URL is not exposed to external sources.
2. **Port Configuration**: The SERIAL_PORT may vary depending on your system; verify it accurately.
3. **Adjust Sensor Values**: Modify condition values (e.g., soilMoisture < 30) to suit your sensors and environment.
---
