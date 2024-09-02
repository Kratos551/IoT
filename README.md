include <ESP8266WiFi.h>
#include <OneWire.h>
#include <DallasTemperature.h>
#include <ThingSpeak.h>

// WiFi credentials
const char* ssid = "YourWiFiSSID";
const char* password = "YourWiFiPassword";

// ThingSpeak credentials
unsigned long channelID = YOUR_THINGSPEAK_CHANNEL_ID;
const char* writeAPIKey = "YourWriteAPIKey";

// Data pin for the DS18B20 temperature sensors
const int oneWireBus = 5; // D1/GPIO5 on ESP8266

OneWire oneWire(oneWireBus);
DallasTemperature sensors(&oneWire);

DeviceAddress sensor1 = { 0x28, 0xFF, 0x1A, 0x50, 0x97, 0x15, 0x03, 0x4A }; // Replace this with your sensor address
DeviceAddress sensor2 = { 0x28, 0xFF, 0x71, 0x54, 0x97, 0x16, 0x03, 0x3B }; // Replace this with your sensor address

float temperature1, temperature2;

void setup() {
  Serial.begin(115200);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  ThingSpeak.begin(client);

  sensors.begin();
  sensors.setResolution(sensor1, 12);
  sensors.setResolution(sensor2, 12);
}

void loop() {
  sensors.requestTemperatures();
  temperature1 = sensors.getTempC(sensor1);
  temperature2 = sensors.getTempC(sensor2);

  if (isnan(temperature1) || isnan(temperature2)) {
    Serial.println("Failed to read from sensors");
  } else {
    float averageTemp = (temperature1 + temperature2) / 2;
    Serial.print("Temperature 1: ");
    Serial.print(temperature1);
    Serial.print(" °C, Temperature 2: ");
    Serial.print(temperature2);
    Serial.print(" °C, Average Temperature: ");
    Serial.print(averageTemp);
    Serial.println(" °C");

    ThingSpeak.setField(1, temperature1);
    ThingSpeak.setField(2, temperature2);
    ThingSpeak.setField(3, averageTemp);

    int response = ThingSpeak.writeFields(channelID, writeAPIKey);
    if (response == 200) {
      Serial.println("Data sent to ThingSpeak successfully");
    } else {
      Serial.print("Failed to send data. HTTP error code: ");
      Serial.println(response);
    }
  }

  delay(15000); // 15-second delay between readings
}
