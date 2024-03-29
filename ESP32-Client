#include <WiFi.h>
#include <WiFiClient.h>
#include <WiFiServer.h>

const char* ssid = "Alight Fullerton";
const char* password = "5mvazj3t";
const int serverPort = 5423;

const int LED1Pin = 25; // Pin for LED1
const int LED2Pin = 26; // Pin for LED2
const int temperaturePin = 34;

bool subscriptionActive = false; // Flag to indicate an active subscription
int numberOfReadings = 0;
int interval = 0;
int isCelsius = 1; // Global variable to track temperature mode

WiFiServer server(serverPort);
WiFiClient client;

void toggleLED(const String &led) {
    Serial.print("Toggling LED based on command: ");
    Serial.println(led);
    
    if (led == "LED1") {
        digitalWrite(LED1Pin, !digitalRead(LED1Pin)); // Toggle LED1
    } else if (led == "LED2") {
        digitalWrite(LED2Pin, !digitalRead(LED2Pin)); // Toggle LED2
    } else {
        Serial.println("Invalid command for toggling LED");
        // Handle unrecognized commands or notify the PC client
    }
}

void readTemperature() {
    int sensorValue = analogRead(temperaturePin);
    Serial.print("Read ADC: ");
    Serial.println(sensorValue);
    client.print(sensorValue); // Send the current ADC value to the client without a new line
    client.println(); // Add a new line separately
}

void subscriptionTask(void *parameter) {
    subscriptionActive = true;

    for (int i = 0; i < numberOfReadings; ++i) {
        if (!client.connected()) {
            subscriptionActive = false;
            vTaskDelete(NULL); // End the task if the client disconnects
        }

        // Read temperature
        int sensorValue = analogRead(temperaturePin);
        Serial.print("Read ADC: ");
        Serial.println(sensorValue);
        
        // Send ADC value to client during subscription
        client.print("Read ADC during subscription: ");
        client.print(sensorValue); // Send the current ADC value to the client without a new line
        client.println(); // Add a new line separately

        // Toggle LED1
        digitalWrite(LED1Pin, !digitalRead(LED1Pin));
        Serial.println("Toggled LED1");

        // Toggle LED2
        digitalWrite(LED2Pin, !digitalRead(LED2Pin));
        Serial.println("Toggled LED2");

        vTaskDelay(interval * 1000 / portTICK_PERIOD_MS); // Delay for the specified interval
    }

    subscriptionActive = false;
    vTaskDelete(NULL); // End the task after completing the specified readings
}



void handleSubscribeCommand(const String &command) {
    if (!subscriptionActive) {
        // Parse the subscribe command to extract readings and interval values
        sscanf(command.c_str(), "subscribe %d %d", &numberOfReadings, &interval);

        xTaskCreatePinnedToCore(subscriptionTask, "SubscriptionTask", 10000, NULL, 1, NULL, 1);
    } else {
        client.println("Another subscription is already active");
    }
}

void handleCommands() {
    if (client) {
        while (client.connected()) {
            if (client.available()) {
                String command = client.readStringUntil('\n');
                command.trim();

                Serial.print("Received command: ");
                Serial.println(command);

                if (command.startsWith("read")) {
                    readTemperature();
                    client.println(analogRead(temperaturePin));
                } else if (command.startsWith("toggle")) {
                    command.remove(0, 7);
                    toggleLED(command);
                    client.println("LED toggled");
                } else if (command.startsWith("mode")) {
                    command.remove(0, 5);
                    if (command == "celsius") {
                        isCelsius = 1;
                        Serial.println("Temperature mode set to Celsius.");
                    } else if (command == "fahrenheit") {
                        isCelsius = 0;
                        Serial.println("Temperature mode set to Fahrenheit.");
                    } else {
                        client.println("Invalid mode command. Use 'mode celsius' or 'mode fahrenheit'");
                    }
                } else if (command.startsWith("subscribe")) {
                    handleSubscribeCommand(command);
                } else if (command.startsWith("exit")) {
                    client.stop();
                    return;
                } else {
                    Serial.println("Unknown command");
                    client.println("Unknown command");
                }
            }
        }
        client.stop();
    }
}


void setup() {
    Serial.begin(115200);
    delay(1000);

    pinMode(LED1Pin, OUTPUT);
    pinMode(LED2Pin, OUTPUT);

    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.println("Connecting to WiFi...");
    }
    Serial.println("Connected to WiFi");

    server.begin();
    Serial.println("Server started");
}

void loop() {
    client = server.available();

    if (client) {
        Serial.println("Client connected");
        handleCommands();
        Serial.println("Client disconnected");
    }
}
