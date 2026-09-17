# KAB
Smart agribusiness and climate defence system for macadamia and citrus farming


#SOURCE CODE FOR THE 

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
