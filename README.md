# 🌱 **Project Name: Smart Environmental Data Monitoring System**

## 📖 **Project Overview**
This project is a system that monitors and manages environmental data using **Jetson Nano** and **Arduino sensors**. It collects real-time data on soil moisture, temperature and humidity, and light intensity, and immediately notifies users of water shortage situations via **Discord alerts**. Additionally, it stores sensor data and implements a **Gradio UI-based chatbot** to provide users with easy access to the information.

## 🚀 **Key Features**
- **Data Collection and Storage**:
  - **Soil Moisture Sensor**: Measure the soil's moisture condition and collect data in real time.
  - **DHT11 Sensor**: Collect temperature and humidity data in real time.
  - **Light Sensor**: Collect ambient light intensity data in real time.
  - Save all collected data in **a CSV file**.

- **Automated Notification System**:
  - Immediately notify users via **Discord Notifications** during water shortages.

- **Gradio UI-Based Chatbot**:
  - Provide an interactive **chatbot interface** for users to check sensor information
  - Enhance data visualization and user accessibility through Gradio UI.

## 🛠️ **Technology Stack**
- **Programming Language**: Python, C++
- **Platform/Framework**: Jetson Nano, Arduino, Gradio
- **Hardware**:
  - Jetson Nano
  - Arduino
  - Sensor: Soil Moisture Sensor, DHT11 Sensor, Light Sensor
- **Other tools**: Discord API, CSV Data Saving

## 📂 **Project structure**
```plaintext
📁 Smart Environmental Data Monitoring System/
├── README.md             # Project description
├── src/                  # Source code
│   ├── main_arduino.md   # Main Arduino sensor data measurement code
│   ├── sensor_data.md    # Sensor data collection code
│   ├── discord_alert.md  # Discord Notification system code
│   └── chatbot_ui.md     # Gradio UI-based chatbot code
├── failed_attempts/
│   └── gradio_graph_attempt.md  # Gradio UI graph attempt code
├── data/                 # CSV data storage folder
│   └── sensor_data.csv   # Collected sensor data sensor **data 20241213.csv**.
└── requirements.txt      # Dependency list # Real-time-Environmental-Monitoring-Platform
