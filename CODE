#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// RFID Setup
#define SS_PIN 10
#define RST_PIN 9
MFRC522 rfid(SS_PIN, RST_PIN);

// Servo Setup
Servo servo1;
Servo servo2;
int servo1Pin = 2;
int servo2Pin = 5;

// LEDs, Buzzer
int greenLED = 7;
int redLED = 6;
int buzzer = 3;

// LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Card UID (replace with your actual UID)
byte validCard[4] = {0x71, 0x4A, 0xA6, 0x08};

// Balance and Toll
int balance = 10000;    // Initial balance
int tollCharge = 25;    // Per entry

void setup() {
  Serial.begin(9600);
  SPI.begin();
  rfid.PCD_Init();

  servo1.attach(servo1Pin);
  servo2.attach(servo2Pin);

  // Initial positions for opposite rotation
  servo1.write(0);     // Start at 0°
  servo2.write(180);   // Start at 180°

  pinMode(greenLED, OUTPUT);
  pinMode(redLED, OUTPUT);
  pinMode(buzzer, OUTPUT);

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("RFID Toll System");
  delay(2000);
  lcd.clear();
}

void loop() {
  if (!rfid.PICC_IsNewCardPresent() || !rfid.PICC_ReadCardSerial()) return;

  Serial.print("Scanned UID: ");
  for (byte i = 0; i < 4; i++) {
    Serial.print(rfid.uid.uidByte[i], HEX);
    Serial.print(" ");
  }
  Serial.println();

  if (isValidCard(rfid.uid.uidByte)) {
    if (balance >= tollCharge) {
      balance -= tollCharge;
      openGate();
    } else {
      showInsufficientBalance();
    }
  } else {
    denyAccess();
  }

  rfid.PICC_HaltA();
  rfid.PCD_StopCrypto1();
  delay(2000);
}

bool isValidCard(byte *uid) {
  for (byte i = 0; i < 4; i++) {
    if (uid[i] != validCard[i]) return false;
  }
  return true;
}

void openGate() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Access Granted");
  lcd.setCursor(0, 1);
  lcd.print("Bal: Rs ");
  lcd.print(balance);

  digitalWrite(greenLED, HIGH);
  digitalWrite(redLED, LOW);
  digitalWrite(buzzer, LOW);

  // Rotate servos
  servo1.write(180);  // Clockwise
  servo2.write(0);    // Anti-clockwise
  delay(3000);        // Keep gate open for 3 seconds

  // Return to original positions
  servo1.write(0);
  servo2.write(180);

  digitalWrite(greenLED, LOW);
  lcd.clear();
}

void denyAccess() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Access Denied");

  digitalWrite(redLED, HIGH);
  digitalWrite(buzzer, HIGH);
  delay(2000);
  digitalWrite(redLED, LOW);
  digitalWrite(buzzer, LOW);

  lcd.clear();
}

void showInsufficientBalance() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Low Balance!");
  lcd.setCursor(0, 1);
  lcd.print("Add Funds");

  digitalWrite(redLED, HIGH);
  delay(3000);
  digitalWrite(redLED, LOW);

  lcd.clear();
}
