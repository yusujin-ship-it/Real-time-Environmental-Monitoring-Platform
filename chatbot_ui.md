---

# **Sensor Data Retrieval and Gradio UI Implementation Project**

This code processes Arduino sensor data either in real-time or from a file and provides a user interface using OpenAI API and Gradio UI.
---

## **Project Setup**

### **Required Library Installation**

The following libraries need to be installed:

```bash
!pip install openai
!pip install gradio
!pip install pyserial
```
---

## **2. Environment Variable Configuration**
### **Default Settings and Library Import**

```python
# 필수 라이브러리 설치
!pip install openai
!pip install gradio
!pip install pyserial

# Import
import gradio as gr
import pandas as pd
import serial
import time
import json
from openai import OpenAI
import os

os.environ['OPENAI_API_KEY'] = 'please enter your key'

OpenAI.api_key = os.getenv("OPENAI_API_KEY")
import time
import serial
```
### **Soil Moisture Sensor Function**
```python
def moisture_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    Function to process and return soil moisture sensor data.

    Parameters:
    - mode (str): 'real-time' or 'file'. Processes data in real-time or from a file.
    - file_path (str): Path to the CSV file used in 'file' mode.
    - port (str): Serial port used in 'real-time' mode.
    - baudrate (int): Serial communication speed.

    Returns:
    - dict: Dictionary containing Soil Moisture value or an error message.
    """
    if mode == 'real-time':
        try:
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)
            arduino.reset_input_buffer()
            attempts = 5
            while attempts > 0:
                if arduino.in_waiting > 0:
                    data = arduino.readline().decode('utf-8', errors='ignore').strip()
                    if "Soil Moisture:" in data:
                        try:
                            soil_moisture = int(data.split(", ")[0].split(":")[1].strip())
                            arduino.close()
                            return {
                                'timestamp': time.strftime("%Y-%m-%d %H:%M:%S"),
                                'Soil Moisture (%)': soil_moisture
                            }
                        except (IndexError, ValueError):
                            pass
                time.sleep(1)
                attempts -= 1
            arduino.close()
            return {'error': 'No valid data received from sensor after multiple attempts.'}
        except Exception as e:
            return {'error': str(e)}
    elif mode == 'file':
        try:
            if not file_path:
                return {'error': 'File path is required for "file" mode.'}
            df = pd.read_csv(file_path)
            if df.empty:
                return {'error': 'The CSV file is empty.'}
            latest_entry = df.iloc[-1]
            return {
                'timestamp': latest_entry['Timestamp'],
                'Soil Moisture (%)': latest_entry['Soil Moisture (%)']
            }
        except Exception as e:
            return {'error': str(e)}
    else:
        return {'error': 'Invalid mode. Use "real-time" or "file".'}
```

---

### **Light Sensor Function**
```python
def light_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    Function to process and return light sensor data.
    """
    if mode == 'real-time':
        try:
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)
            arduino.reset_input_buffer()
            attempts = 5
            while attempts > 0:
                if arduino.in_waiting > 0:
                    data = arduino.readline().decode('utf-8', errors='ignore').strip()
                    if "Light Intensity:" in data:
                        try:
                            light_intensity = int(data.split(", ")[1].split(":")[1].strip())
                            arduino.close()
                            return {
                                'timestamp': time.strftime("%Y-%m-%d %H:%M:%S"),
                                'Light Intensity (%)': light_intensity
                            }
                        except (IndexError, ValueError):
                            pass
                time.sleep(1)
                attempts -= 1
            arduino.close()
            return {'error': 'No valid data received from sensor after multiple attempts.'}
        except Exception as e:
            return {'error': str(e)}
    elif mode == 'file':
        try:
            if not file_path:
                return {'error': 'File path is required for "file" mode.'}
            df = pd.read_csv(file_path)
            if df.empty:
                return {'error': 'The CSV file is empty.'}
            latest_entry = df.iloc[-1]
            return {
                'timestamp': latest_entry['Timestamp'],
                'Light Intensity (%)': latest_entry['Light Intensity (%)']
            }
        except Exception as e:
            return {'error': str(e)}
    else:
        return {'error': 'Invalid mode. Use "real-time" or "file".'}
```
---

---

### **DHT11 Sensor Function**
```python
def dht_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    Function to process and return temperature and humidity sensor data.
    """
    if mode == 'real-time':
        try:
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)
            arduino.reset_input_buffer()
            attempts = 5
            while attempts > 0:
                if arduino.in_waiting > 0:
                    data = arduino.readline().decode('utf-8', errors='ignore').strip()
                    if "Humidity:" in data and "Temperature:" in data:
                        try:
                            humidity = int(data.split(", ")[2].split(":")[1].strip())
                            temperature = int(data.split(", ")[3].split(":")[1].strip())
                            arduino.close()
                            return {
                                'timestamp': time.strftime("%Y-%m-%d %H:%M:%S"),
                                'Humidity (%)': humidity,
                                'Temperature (C)': temperature
                            }
                        except (IndexError, ValueError):
                            pass
                time.sleep(1)
                attempts -= 1
            arduino.close()
            return {'error': 'No valid data received from sensor after multiple attempts.'}
        except Exception as e:
            return {'error': str(e)}
    elif mode == 'file':
        try:
            if not file_path:
                return {'error': 'File path is required for "file" mode.'}
            df = pd.read_csv(file_path)
            if df.empty:
                return {'error': 'The CSV file is empty.'}
            latest_entry = df.iloc[-1]
            return {
                'timestamp': latest_entry['Timestamp'],
                'Humidity (%)': latest_entry['Humidity (%)'],
                'Temperature (C)': latest_entry['Temperature (C)']
            }
        except Exception as e:
            return {'error': str(e)}
    else:
        return {'error': 'Invalid mode. Use "real-time" or "file".'}
```        
---

---
## **Sensor Function Definitions (`sensor_functions`)**

`sensor_functions` is a JSON structure defined to allow OpenAI API to call specific functions. Each function is designed to retrieve data from a particular sensor either in real-time or from a saved file.

### **Code**

```python
sensor_functions = [
    {
        "type": "function",
        "function": {
            "name": "moisture_sensor_info",
            "description": "Retrieves soil moisture data either in real-time from a sensor or from a file.",
            "parameters": {
                "type": "object",
                "properties": {
                    "mode": {
                        "type": "string",
                        "description": "'real-time' for live data from the sensor or 'file' for data from a CSV file.",
                        "enum": ["real-time", "file"]
                    },
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing soil moisture data (required if mode is 'file').",
                        "nullable": True
                    },
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the sensor is connected (required if mode is 'real-time'). Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    }
                },
                "required": ["mode"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "light_sensor_info",
            "description": "Retrieves light intensity data either in real-time from a sensor or from a file.",
            "parameters": {
                "type": "object",
                "properties": {
                    "mode": {
                        "type": "string",
                        "description": "'real-time' for live data from the sensor or 'file' for data from a CSV file.",
                        "enum": ["real-time", "file"]
                    },
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing light intensity data (required if mode is 'file').",
                        "nullable": True
                    },
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the sensor is connected (required if mode is 'real-time'). Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    }
                },
                "required": ["mode"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "dht_sensor_info",
            "description": "Retrieves temperature and humidity data either in real-time from a sensor or from a file.",
            "parameters": {
                "type": "object",
                "properties": {
                    "mode": {
                        "type": "string",
                        "description": "'real-time' for live data from the sensor or 'file' for data from a CSV file.",
                        "enum": ["real-time", "file"]
                    },
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing temperature and humidity data (required if mode is 'file').",
                        "nullable": True
                    },
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the sensor is connected (required if mode is 'real-time'). Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    }
                },
                "required": ["mode"]
            }
        }
    }
]
---

---





