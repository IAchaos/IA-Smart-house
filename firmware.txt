#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

// WiFi Configuration
const char* ssid = "Wokwi-GUEST";
const char* password = "";

// MQTT Configuration
const char* mqtt_server = "broker.hivemq.com";
const int mqtt_port = 1883;

// FIX: Use a UNIQUE topic prefix with your name to avoid conflicts
// CHANGE THIS to your own unique identifier!
const char* topic_prefix = "smarthome_allam_imad_2025";

// Pin Definitions
#define DHTPIN 15
#define DHTTYPE DHT22
#define PIR_PIN 27
#define LDR_PIN 34
#define LED1_PIN 16
#define LED2_PIN 17
#define BUZZER_PIN 26

// System Constants
const long sensor_interval = 2000;
const long pir_interval = 500;
const int light_threshold = 2000;

// Initialize DHT sensor
DHT dht(DHTPIN, DHTTYPE);

// WiFi and MQTT clients
WiFiClient espClient;
PubSubClient client(espClient);

// System state variables
float temperature = 20.0;
float humidity = 50.0;
int lightLevel = 0;
bool motionDetected = false;
bool systemArmed = true;
bool nightTime = false;

// Timing variables
unsigned long lastSensorReading = 0;
unsigned long lastPIRCheck = 0;
unsigned long lastReconnectAttempt = 0;
const long reconnectInterval = 5000;

// Topic buffers
char topic_temp[100];
char topic_hum[100];
char topic_light[100];
char topic_security[100];
char topic_light_status[100];
char topic_status[100];
char topic_commands[100];
char topic_alarm[100];

void setup_topics() {
  // Build all topic strings with unique prefix
  snprintf(topic_temp, sizeof(topic_temp), "%s/temperature", topic_prefix);
  snprintf(topic_hum, sizeof(topic_hum), "%s/humidity", topic_prefix);
  snprintf(topic_light, sizeof(topic_light), "%s/lightlevel", topic_prefix);
  snprintf(topic_security, sizeof(topic_security), "%s/security/status", topic_prefix);
  snprintf(topic_light_status, sizeof(topic_light_status), "%s/light/status", topic_prefix);
  snprintf(topic_status, sizeof(topic_status), "%s/status", topic_prefix);
  snprintf(topic_commands, sizeof(topic_commands), "%s/commands", topic_prefix);
  snprintf(topic_alarm, sizeof(topic_alarm), "%s/security/alarm", topic_prefix);
  
  Serial.println("\n=== MQTT Topics ===");
  Serial.print("Temperature: ");
  Serial.println(topic_temp);
  Serial.print("Humidity: ");
  Serial.println(topic_hum);
  Serial.print("Commands: ");
  Serial.println(topic_commands);
  Serial.println("==================\n");
}

void setup_wifi() {
  delay(10);
  Serial.println("Connecting to WiFi");
  
  WiFi.begin(ssid, password);
  
  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 20) {
    delay(500);
    Serial.print(".");
    attempts++;
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWiFi connected");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
    digitalWrite(LED2_PIN, HIGH);
  } else {
    Serial.println("\nFailed to connect to WiFi");
    digitalWrite(LED2_PIN, LOW);
  }
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  
  String message;
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println(message);
  
  if (strcmp(topic, topic_commands) == 0) {
    if (message == "arm") {
      systemArmed = true;
      Serial.println("System armed");
      client.publish(topic_security, "armed", false);
      digitalWrite(LED2_PIN, HIGH);
    } 
    else if (message == "disarm") {
      systemArmed = false;
      Serial.println("System disarmed");
      client.publish(topic_security, "disarmed", false);
      digitalWrite(LED2_PIN, LOW);
    }
    else if (message == "lighton") {
      digitalWrite(LED1_PIN, HIGH);
      client.publish(topic_light_status, "on", false);
    }
    else if (message == "lightoff") {
      digitalWrite(LED1_PIN, LOW);
      client.publish(topic_light_status, "off", false);
    }
  }
}

boolean reconnect() {
  unsigned long now = millis();
  if (now - lastReconnectAttempt > reconnectInterval) {
    lastReconnectAttempt = now;
    
    Serial.print("Attempting MQTT connection...");
    
    // Create unique client ID
    String clientId = "ESP32_SmartHome_";
    clientId += String(random(0xffff), HEX);
    
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      
      // FIX: Publish with retain=false to prevent storing old values
      client.publish(topic_status, "online", false);
      client.subscribe(topic_commands);
      
      client.publish(topic_security, systemArmed ? "armed" : "disarmed", false);
      client.publish(topic_light_status, digitalRead(LED1_PIN) ? "on" : "off", false);
      
      return true;
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again later");
    }
  }
  return false;
}

void statusBlink(int count) {
  for (int i = 0; i < count; i++) {
    digitalWrite(LED2_PIN, HIGH);
    delay(200);
    digitalWrite(LED2_PIN, LOW);
    delay(200);
  }
}

void triggerAlarm() {
  if (systemArmed) {
    Serial.println("ALARM ACTIVATED!");
    client.publish(topic_alarm, "alarm", false);
    
    for (int i = 0; i < 5; i++) {
      digitalWrite(BUZZER_PIN, HIGH);
      digitalWrite(LED2_PIN, HIGH);
      delay(200);
      digitalWrite(BUZZER_PIN, LOW);
      digitalWrite(LED2_PIN, LOW);
      delay(200);
    }
    
    digitalWrite(LED2_PIN, HIGH);
  } else {
    Serial.println("Motion detected but system is disarmed");
  }
}

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("\n\n=== SMART HOME SYSTEM STARTING ===");
  
  pinMode(PIR_PIN, INPUT);
  pinMode(LDR_PIN, INPUT);
  pinMode(LED1_PIN, OUTPUT);
  pinMode(LED2_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, LOW);
  digitalWrite(BUZZER_PIN, LOW);
  
  // Setup unique topics
  setup_topics();
  
  Serial.println("Initializing DHT22 sensor...");
  dht.begin();
  
  delay(2000);
  
  Serial.println("Reading initial DHT22 values...");
  for (int attempt = 0; attempt < 10; attempt++) {
    float t = dht.readTemperature();
    float h = dht.readHumidity();
    
    Serial.print("Attempt ");
    Serial.print(attempt + 1);
    Serial.print(": Temp=");
    Serial.print(t);
    Serial.print("°C, Hum=");
    Serial.print(h);
    Serial.print("%");
    
    if (!isnan(t) && !isnan(h) && t >= -40 && t <= 80 && h >= 0 && h <= 100) {
      temperature = t;
      humidity = h;
      Serial.println(" [VALID]");
      break;
    } else {
      Serial.println(" [INVALID - RETRYING]");
      delay(500);
    }
  }
  
  Serial.print("Initial values set: Temp=");
  Serial.print(temperature);
  Serial.print("°C, Hum=");
  Serial.print(humidity);
  Serial.println("%");
  
  statusBlink(3);
  
  setup_wifi();
  
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
  
  String clientId = "ESP32_SmartHome_";
  clientId += String(random(0xffff), HEX);
  
  if (client.connect(clientId.c_str())) {
    client.publish(topic_status, "online", false);
    client.subscribe(topic_commands);
  }
  
  Serial.println("=== SYSTEM READY ===\n");
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
  
  unsigned long currentMillis = millis();
  
  if (currentMillis - lastPIRCheck > pir_interval) {
    lastPIRCheck = currentMillis;
    
    int pirValue = digitalRead(PIR_PIN);
    if (pirValue == HIGH && !motionDetected) {
      motionDetected = true;
      Serial.println("Motion detected!");
      
      if (nightTime) {
        digitalWrite(LED1_PIN, HIGH);
        client.publish(topic_light_status, "on", false);
      }
      
      triggerAlarm();
    } 
    else if (pirValue == LOW && motionDetected) {
      motionDetected = false;
      Serial.println("Motion ended");
      
      if (nightTime) {
        delay(5000);
        if (!motionDetected) {
          digitalWrite(LED1_PIN, LOW);
          client.publish(topic_light_status, "off", false);
        }
      }
    }
  }
  
  if (currentMillis - lastSensorReading > sensor_interval) {
    lastSensorReading = currentMillis;
    
    Serial.println("\n[Sensor Reading]");
    
    float newTemp = dht.readTemperature();
    float newHum = dht.readHumidity();
    
    Serial.print("Raw DHT: Temp=");
    Serial.print(newTemp);
    Serial.print("°C, Hum=");
    Serial.print(newHum);
    Serial.print("%");
    
    bool validReading = !isnan(newTemp) && !isnan(newHum) && 
                       newTemp >= -40 && newTemp <= 80 && 
                       newHum >= 0 && newHum <= 100;
    
    if (validReading) {
      temperature = newTemp;
      humidity = newHum;
      Serial.println(" [VALID]");
    } else {
      Serial.println(" [INVALID - using old values]");
    }
    
    // FIX: Format values properly and publish with retain=FALSE
    char tempStr[10];
    char humStr[10];
    dtostrf(temperature, 4, 1, tempStr);
    dtostrf(humidity, 4, 1, humStr);
    
    Serial.print("Publishing: Temp=");
    Serial.print(tempStr);
    Serial.print("°C to topic: ");
    Serial.println(topic_temp);
    
    Serial.print("Publishing: Hum=");
    Serial.print(humStr);
    Serial.print("% to topic: ");
    Serial.println(topic_hum);
    
    // FIX: The third parameter (false) means DO NOT RETAIN
    client.publish(topic_temp, tempStr, false);
    client.publish(topic_hum, humStr, false);
    
    lightLevel = analogRead(LDR_PIN);
    Serial.print("Light level: ");
    Serial.println(lightLevel);
    
    nightTime = (lightLevel < light_threshold);
    
    client.publish(topic_light, String(lightLevel).c_str(), false);
    
    if (nightTime && motionDetected) {
      digitalWrite(LED1_PIN, HIGH);
      client.publish(topic_light_status, "on", false);
    } else if (!motionDetected && nightTime) {
      digitalWrite(LED1_PIN, LOW);
      client.publish(topic_light_status, "off", false);
    }
    
    Serial.println("---");
  }
}