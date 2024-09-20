# ECG-MONITORING-SYSTEM
Overview
In one of my previous tutorial, I explained how to interface AD8232 ECG Sensor with Arduino and monitor the ECG waveform on Serial Plotter. You can read my previous guide here: ECG Monitoring with AD8232 ECG Sensor & Arduino with ECG Graph.

But today we will learn how to monitor the same ECG graph online on any IoT cloud platform. For that, we will interface AD8232 ECG Sensor with ESP32. And then we will generate an ECG signal by connecting ECG leads to chest or hand. Using Ubidots parameters like API Key or Token we will send the ECG graph to cloud using MQTT Broker. This project can also be done using NodeMCU ESP8266 Board but connections and program need to be modified.

Medical uses of ECG
An electrocardiogram can be a useful way to find out whether your high blood pressure has caused any damage to your heart or blood vessels. Because of this, you may be asked to have an ECG when you are first diagnosed with high blood pressure.

Some of the things an ECG reading can detect are:
1. cholesterol clogging up your heart’s blood supply
2. a heart attack in the past
3. enlargement of one side of the heart
4. abnormal heart rhythms


>> **5. AD8232 ECG Sensor***
This sensor is a cost-effective board used to measure the electrical activity of the heart. This electrical activity can be charted as an ECG or Electrocardiogram and output as an analog reading. ECGs can be extremely noisy, the AD8232 Single Lead Heart Rate Monitor acts as an op amp to help obtain a clear signal from the PR and QT Intervals easily.

AD8232 ECG Sensor:-
The AD8232 is an integrated signal conditioning block for ECG and other biopotential measurement applications. It is designed to extract, amplify, and filter small biopotential signals in the presence of noisy conditions, such as those created by motion or remote electrode placement.

The AD8232 module breaks out nine connections from the IC that you can solder pins, wires, or other connectors to. SDN, LO+, LO-, OUTPUT, 3.3V, GND provide essential pins for operating this monitor with an Arduino or other development board. Also provided on this board are RA (Right Arm), LA (Left Arm), and RL (Right Leg) pins to attach and use your own custom sensors. Additionally, there is an LED indicator light that will pulsate to the rhythm of a heartbeat.

AD8232 ECG Sensor Placement on Body
It is recommended to snap the sensor pads on the leads before application to the body. The closer to the heart the pads are, the better the measurement. The cables are color coded to help identify proper placement.
Red: RA (Right Arm)
Yellow: LA (Left Arm)
Green: RL (Right Leg)


Setting Up Ubidots Account
To publish the data to IoT Cloud, we need some IoT platform. So Ubidots is one such platform. Ubidots offers a platform for developers that enables them to easily capture sensor data and turn it into useful information. Use the Ubidots platform to send data to the cloud from any Internet-enabled device.
Ubidots
Step 1: Creating Ubidots Account
Go to ubidots.com and create and account. You will get a trial period of 30 days.
Step 2: Creating Device & Adding Variables
Now setup an Ubidots Device. To create it, go to the Devices section (Devices > Devices). Create a new Device with name esp32.
Once the device is created, create a new variable by renaming the variabale to sensor.
Step 3: Creating Dashboards
Let’s setup an Ubidots’ Dashboard. To create it, go to the Dashboard section (Data > Dashboard)
Step 4: Adding New Widgets

> Click on the + sign in the right side and “Add new Widget”, and select your widget.
> Now, Select the type of widget desired to be displayed. In my case, I choose the “Line Chart”:
> Then, select the variable desired to display the data. Ubidots allows you assign a customize widget name, color, period of data to be displayed and much more. To finish the widget creation, press the green icon.
> Selct your previously created Device and Variables as shown in the figure below.

Source Code/Program
The source code for IoT Based ECG Monitoring with AD8232 ECG Sensor & ESP32 is given below. Copy this code and change the following Parameters.

1. WIFISSID:</strong> Your WiFi SSID
2. PASSWORD:</strong> Your WiFi password
3. TOKEN:</strong> Your Ubidots TOKEN (Check the video below to find about it)
4. MQTT_CLIENT_NAME:</strong> Your own 8-12 alphanumeric character ASCII string.

 
#include <WiFi.h>
#include <PubSubClient.h>
 
#define WIFISSID "Alexahome" // Put your WifiSSID here
#define PASSWORD "hngzhowxiantan" // Put your wifi password here
#define TOKEN "BBFF-RJ8ABBbh6G2ECGU0rkjRSOdXqhUnvj" // Put your Ubidots' TOKEN
#define MQTT_CLIENT_NAME "alexnewton" // MQTT client Name, please enter your own 8-12 alphanumeric character ASCII string; 
                                           //it should be a random and unique ascii string and different from all other devices
 
/****************************************
 * Define Constants
 ****************************************/
#define VARIABLE_LABEL "sensor" // Assing the variable label
#define DEVICE_LABEL "esp32" // Assig the device label
 
#define SENSOR A0 // Set the A0 as SENSOR
 
char mqttBroker[]  = "industrial.api.ubidots.com";
char payload[100];
char topic[150];
// Space to store values to send
char str_sensor[10];
 
/****************************************
 * Auxiliar Functions
 ****************************************/
WiFiClient ubidots;
PubSubClient client(ubidots);
 
void callback(char* topic, byte* payload, unsigned int length) {
  char p[length + 1];
  memcpy(p, payload, length);
  p[length] = NULL;
  Serial.write(payload, length);
  Serial.println(topic);
}
 
void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.println("Attempting MQTT connection...");
    
    // Attemp to connect
    if (client.connect(MQTT_CLIENT_NAME, TOKEN, "")) {
      Serial.println("Connected");
    } else {
      Serial.print("Failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 2 seconds");
      // Wait 2 seconds before retrying
      delay(2000);
    }
  }
}
 
/****************************************
 * Main Functions
 ****************************************/
void setup() {
  Serial.begin(115200);
  WiFi.begin(WIFISSID, PASSWORD);
  // Assign the pin as INPUT 
  pinMode(SENSOR, INPUT);
 
  Serial.println();
  Serial.print("Waiting for WiFi...");
  
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }
  
  Serial.println("");
  Serial.println("WiFi Connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  client.setServer(mqttBroker, 1883);
  client.setCallback(callback);  
}
 
void loop() {
  if (!client.connected()) {
    reconnect();
  }
 
  sprintf(topic, "%s%s", "/v1.6/devices/", DEVICE_LABEL);
  sprintf(payload, "%s", ""); // Cleans the payload
  sprintf(payload, "{\"%s\":", VARIABLE_LABEL); // Adds the variable label
  
  float sensor = analogRead(SENSOR); 
  
  /* 4 is mininum width, 2 is precision; float value is copied onto str_sensor*/
  dtostrf(sensor, 4, 2, str_sensor);
  
  sprintf(payload, "%s {\"value\": %s}}", payload, str_sensor); // Adds the value
  Serial.println("Publishing data to Ubidots Cloud");
  client.publish(topic, payload);
  client.loop();
  delay(500);
}
