#include <SimpleDHT.h>

// Pin definitions
const int soilMoisturePin = A1; // Soil moisture sensor connected to analog pin
const int lightSensorPin = A0;  // Light sensor connected to analog pin
const int dhtPin = 2;           // DHT11 connected to digital pin

// Constants for soil moisture sensor
const int dryValue = 520;  // Calibrated dry soil value (represents no moisture in the soil)
const int wetValue = 260;  // Calibrated wet soil value (represents fully saturated soil)

// DHT11 sensor initialization
SimpleDHT11 dht11;

// Function to calculate the average of multiple readings from a sensor
// Parameters:
//   pin - Analog pin connected to the sensor
//   samples - Number of readings to average
// Returns:
//   The averaged sensor value
int getAverageSensorValue(int pin, int samples) {
  long total = 0;
  for (int i = 0; i < samples; i++) {
    total += analogRead(pin);  // Read the sensor value
    delay(10); // Delay between readings to ensure stability
  }
  return total / samples; // Calculate the average
}

// Function to get a stable and averaged reading from the light sensor
// Removes extreme values (outliers) to reduce noise in the data
// Returns:
//   The averaged light intensity value
int getStableLightReading() {
  const int NUM_SAMPLES = 10;  // Number of readings to collect
  int samples[NUM_SAMPLES];

  // Collect readings from the light sensor
  for (int i = 0; i < NUM_SAMPLES; i++) {
    samples[i] = analogRead(lightSensorPin);
    delay(10);
  }

  // Sort readings in ascending order to remove outliers
  for (int i = 0; i < NUM_SAMPLES - 1; i++) {
    for (int j = i + 1; j < NUM_SAMPLES; j++) {
      if (samples[i] > samples[j]) {
        int temp = samples[i];
        samples[i] = samples[j];
        samples[j] = temp;
      }
    }
  }

  // Remove the two smallest and two largest readings
  long sum = 0;
  for (int i = 2; i < NUM_SAMPLES - 2; i++) {
    sum += samples[i];
  }

  // Return the average of the remaining readings
  return sum / (NUM_SAMPLES - 4);
}

void setup() {
  Serial.begin(9600); // Initialize serial communication at 9600 baud rate
  Serial.println("Starting Combined Sensor Test!"); // Debug message
}

void loop() {
  // 1. Soil Moisture Sensor
  int soilValue = getAverageSensorValue(soilMoisturePin, 10); // Get averaged sensor value
  int soilMoisturePercent = map(soilValue, dryValue, wetValue, 0, 100); // Map the sensor value to a percentage
  soilMoisturePercent = constrain(soilMoisturePercent, 0, 100); // Constrain the value to a valid range (0-100%)

  // 2. Light Sensor
  int lightValue = getStableLightReading(); // Get stabilized light sensor reading
  int lightPercent = map(lightValue, 0, 1023, 0, 100); // Map the light intensity to a percentage

  // 3. DHT11 Sensor
  byte temperature = 0; // Variable to store temperature
  byte humidity = 0;    // Variable to store humidity
  int dhtErr = dht11.read(dhtPin, &temperature, &humidity, NULL); // Read temperature and humidity from DHT11

  // Output all sensor data
  Serial.print("Soil Moisture:");
  Serial.print(soilMoisturePercent);
  Serial.print("%, Light Intensity:");
  Serial.print(lightPercent);
  Serial.print("%, ");

  // Check if DHT11 data was successfully read
  if (dhtErr == SimpleDHTErrSuccess) {
    Serial.print("Humidity:");
    Serial.print(humidity);
    Serial.print("%, Temperature:");
    Serial.println(temperature);
  } else {
    Serial.println("Humidity:Error, Temperature:Error");
  }

  delay(2000); // Wait for 2 seconds before the next reading
}
