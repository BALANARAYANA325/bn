#include <DHT.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

// Constants
#define DHTPIN 2      // Pin where the DHT11 is connected
#define DHTTYPE DHT11 // Define DHT type, DHT11
#define SOIL_SENSOR_PIN A0 // Pin for Soil Moisture Sensor

DHT dht(DHTPIN, DHTTYPE);

// WiFi credentials
const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

// Server URL
const char* serverUrl = "http://your_server_ip/sensor_data.php";  // Replace with your server's IP or domain

void setup() {
  // Initialize DHT sensor
  dht.begin();

  // Initialize Serial Monitor
  Serial.begin(115200);

  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
}

void loop() {
  // Read sensor data
  float temperature = dht.readTemperature();  // Read temperature in Celsius
  float humidity = dht.readHumidity();        // Read humidity
  int soilMoistureValue = analogRead(SOIL_SENSOR_PIN); // Read soil moisture
  int soilMoisturePercent = map(soilMoistureValue, 1023, 0, 0, 100); // Convert to percentage

  // Display the data in the Serial Monitor
  Serial.print("Temperature: ");
  Serial.println(temperature);
  Serial.print("Humidity: ");
  Serial.println(humidity);
  Serial.print("Soil Moisture: ");
  Serial.println(soilMoisturePercent);

  // Send data to the PHP server
  if (WiFi.status() == WL_CONNECTED) {  // Check if Wi-Fi is connected
    HTTPClient http;

    // Construct URL with sensor data as GET parameters
    String url = String(serverUrl) + "?temp=" + String(temperature) + "&hum=" + String(humidity) + "&soil=" + String(soilMoisturePercent);
    http.begin(url);  // Specify the URL
    
    // Send the request and get the response
    int httpCode = http.GET();  // Send the request

    // Check the HTTP response code
    if (httpCode > 0) {
      String payload = http.getString();
      Serial.println(payload);  // Print response from server (PHP)
    } else {
      Serial.println("Error in sending request");
    }

    http.end();  // Close the connection
  }

  delay(2000);  // Delay before next reading
}
