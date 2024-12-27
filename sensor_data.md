---

## Collecting Data from Arduino Sensors (`collect_sensor_data.md`)

This script provides code for collecting real-time data from Arduino sensors using Python and Pandas and saving it as a CSV file.
The code is executed in a Jupyter Notebook within a virtual environment on the Jetson Nano 

---

### **Full Code**

```python
import serial
import time
import pandas as pd

# Serial Communication Setup
port = "/dev/ttyUSB0"  # Arduino's Serial Port
baudrate = 9600        # Same Baud Rate as Arduino
arduino = serial.Serial(port, baudrate, timeout=1)

# List for Data Storage
data = []

# Initialize Current Date
current_date = time.strftime("%Y-%m-%d")

try:
    print("Collecting data from Arduino... Press Ctrl+C to stop.")
    while True:
        if arduino.in_waiting > 0:  # Check if Data Has Arrived on the Serial Port
            line = arduino.readline().decode("utf-8").strip()  # Read and Decode Data
            
            # Data Filtering: Process Sensor Data Only
            if "Soil Moisture:" in line and "Light Intensity:" in line:
                timestamp = time.strftime("%Y-%m-%d %H:%M:%S")  # Save the Current Time in Seconds
                
                # Parse Sensor Data
                parts = line.split(", ")
                soil_moisture = parts[0].split(":")[1].strip()  # Soil Moisture 값
                light_intensity = parts[1].split(":")[1].strip()  # Light Intensity 값
                humidity = parts[2].split(":")[1].strip()  # Humidity 값
                temperature = parts[3].split(":")[1].strip()  # Temperature 값
                
                print(f"{timestamp}, Soil Moisture: {soil_moisture}, Light Intensity: {light_intensity}, Humidity: {humidity}, Temperature: {temperature}")
                
                # Data Saving
                data.append({
                    "Timestamp": timestamp,
                    "Soil Moisture (%)": soil_moisture,
                    "Light Intensity (%)": light_intensity,
                    "Humidity (%)": humidity,
                    "Temperature (C)": temperature
                })

                # Create DataFrame
                df = pd.DataFrame(data)

                # Generate Filename (Based on Current Date)
                file_name = f"sensor_data_{current_date.replace('-', '')}.csv"

                # Save DataFrame as a CSV File
                df.to_csv(file_name, index=False, encoding="utf-8")
                print(f"Data saved to {file_name}")

except KeyboardInterrupt:
    print("Data collection stopped.")

finally:
    # Save Remaining Data
    if data:
        df = pd.DataFrame(data)
        file_name = f"sensor_data_{current_date.replace('-', '')}.csv"
        df.to_csv(file_name, index=False, encoding="utf-8")
        print(f"Final data saved to {file_name}")

    # Close Port
    arduino.close()
```

---

### **Project Overview**

This script performs the following functions:

1. Arduino Serial Communication Setup: Establishes a serial connection to read data from Arduino.
2. Data Filtering and Parsing: Reads sensor data (soil moisture, light intensity, temperature, and humidity) transmitted from Arduino and accurately categorizes it.
3. Real-Time Data Saving: Continuously stores the collected data into a CSV file in real time.
4. Safe Data Preservation on Exit: Ensures any remaining data is safely saved when the program terminates.

This script is designed to provide a seamless solution for real-time sensor data collection and storage, making it ideal for applications requiring continuous monitoring and analysis. 

---

### **Usage Instructions**

1. Upload and run the Arduino code (mainarduino.md).
2. Execute the Python script
```python
python collect_sensor_data.py
```
3. Press "Ctrl+C" to terminate the program during execution.
4. Collected data will be saved in a file named sensor_data_YYYYMMDD.csv.

---

### **Output Example (Terminal)**

```
2024-06-01 14:23:45, Soil Moisture: 45, Light Intensity: 78, Humidity: 60, Temperature: 24
Data saved to sensor_data_20240601.csv
2024-06-01 14:23:47, Soil Moisture: 43, Light Intensity: 80, Humidity: 59, Temperature: 23
Data saved to sensor_data_20240601.csv
```

---

### **File Example (CSV)**

| Timestamp           | Soil Moisture (%) | Light Intensity (%) | Humidity (%) | Temperature (C) |
|---------------------|-------------------|---------------------|-------------|-----------------|
| 2024-06-01 14:23:45 | 45                | 78                  | 60          | 24              |
| 2024-06-01 14:23:47 | 43                | 80                  | 59          | 23              |

---
---
### **Important Notes**

1. **Serial Port Configuration**: Adjust the serial port to match your system, such as /dev/ttyUSB0 (Linux/Mac) or COM3 (Windows).
2. **Baud Rate**: Ensure the baud rate (9600) in the Arduino code matches that in the Python script.
3. **Data Saving on Exit**: When terminating the program with Ctrl+C, all collected data up to that point will be saved.
---
