# KAB
Smart agribusiness and climate defence system for macadamia and citrus farming


#SOURCE CODE FOR THE AGROSHIELD 

#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

// Pins
int soilPin = A0;
int pumpPin = 8;
int trigPin = 9;
int echoPin = 10;

int crop = 0;
float tankHeight = 30.0;

void setup() {
  Serial.begin(9600);
  pinMode(pumpPin, OUTPUT);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  digitalWrite(pumpPin, LOW);

  lcd.init();
  lcd.backlight();
  lcd.print("SMART FARMING");
  lcd.setCursor(0,1);
  lcd.print("1=Tom 2=Pota");

  Serial.println("SMART FARMING - Choose crop: 1=Tomato 2=Potato");
}

void loop() {
  if (crop == 0 && Serial.available() > 0) {
    crop = Serial.parseInt();
    if (crop != 1 && crop != 2) {
      Serial.println("Invalid choice.");
      crop = 0;
    } else {
      Serial.println(crop == 1 ? "Tomato selected" : "Potato selected");
    }
  }

  if (crop == 1 || crop == 2) {
    int rawValue = analogRead(soilPin);
    int moisture = map(rawValue, 800, 300, 0, 100);
    moisture = constrain(moisture, 0, 100);

    float distance = getDistance();
    float waterLevel = tankHeight - distance;
    waterLevel = constrain(waterLevel, 0, tankHeight);
    int tankPercent = (waterLevel / tankHeight) * 100;

    bool irrigating = false;
    if (crop == 1 && moisture < 60 && tankPercent > 10) irrigating = true;
    if (crop == 2 && moisture < 70 && tankPercent > 10) irrigating = true;

    digitalWrite(pumpPin, irrigating ? HIGH : LOW);

    // Serial
    Serial.print("Soil: "); Serial.print(moisture);
    Serial.print("% Tank: "); Serial.print(tankPercent);
    Serial.println(irrigating ? "% PUMP ON" : "% PUMP OFF");

    // LCD - 16x2 so we show in 2 screens
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print(crop == 1 ? "TOMATO " : "POTATO ");
    lcd.print(moisture); lcd.print("%");

    lcd.setCursor(0,1);
    lcd.print("Tank:"); lcd.print(tankPercent); lcd.print("% ");
    lcd.print(irrigating ? "ON" : "OFF");
    
    delay(2000);
  }
}

float getDistance() {
  digitalWrite(trigPin, LOW); delayMicroseconds(2);
  digitalWrite(trigPin, HIGH); delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  long duration = pulseIn(echoPin, HIGH, 30000);
  if (duration == 0) return tankHeight;
  return duration * 0.0343 / 2;
}


GREATINGS TO ALL!!
Today we are presenting Agro Shield Twin, our smart climate-defense solution for macadamia and citrus farming in Mpumalanga.
Farmers face changing environmental conditions such as high temperatures and insufficient soil moisture. The challenge is that these conditions may not affect the entire farm equally. One area can be dry while another area still has enough moisture.
Our solution is designed to identify exactly which farming zone needs attention.
Agro Shield Twin uses soil-moisture and temperature sensors to continuously monitor different zones of the farm. The sensor information is processed locally by an ESP-32
When the system detects low soil moisture, it can automatically activate a relay and irrigation pump to provide water to the affected zone. When high temperatures are detected, it alertsC to the particular situation so that they  shade the farm to help protect the crops.
But our project goes further.
We connect the physical farm to a 3D digital twin. The digital twin represents the different farm zones and displays their current conditions. If Zone 2 becomes too dry, for example, the digital twin highlights Zone 2 as stressed and shows that irrigation is active.
Our system therefore follows four important steps:
Sense. Decide. Act. Protect.
We also designed the system to handle sensor faults, record historical data, and allow manual intervention when necessary.
Our prototype demonstrates how edge computing, sensors, automation, and 3D visualisation can work together to support climate-smart agriculture.
Our vision is not to replace the farmer, but to give the farmer better information and faster, targeted responses.
Agro Shield Twin — Sense. Decide. Act. Protect.
Thank you!!
