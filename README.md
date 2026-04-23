# READER-STATUS-CHECK
Source Code to check weather rfid reader is talking to Arduino Mega
#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN 53
#define RST_PIN 9

MFRC522 rfid(SS_PIN, RST_PIN);

void setup() {
  Serial.begin(9600);
  while (!Serial); // wait for serial monitor (important on Mega)

  SPI.begin();        // start SPI bus
  rfid.PCD_Init();    // initialize RFID reader

  Serial.println("==================================");
  Serial.println(" RFID COMMUNICATION TEST STARTED ");
  Serial.println("==================================");

  // Check firmware version (best communication test)
  byte v = rfid.PCD_ReadRegister(MFRC522::VersionReg);

  Serial.print("MFRC522 Firmware Version: 0x");
  Serial.println(v, HEX);

  if (v == 0x00 || v == 0xFF) {
    Serial.println("❌ ERROR: RFID reader NOT detected!");
    Serial.println("Check wiring and power (3.3V only).");
  } else {
    Serial.println("✅ RFID reader detected successfully!");
    Serial.println("Ready to scan cards...");
  }

  Serial.println("----------------------------------");
}

void loop() {
  // Just keep checking if card communication works
  if (!rfid.PICC_IsNewCardPresent()) return;
  if (!rfid.PICC_ReadCardSerial()) return;

  Serial.println("🎯 Card detected!");

  Serial.print("UID: ");
  for (byte i = 0; i < rfid.uid.size; i++) {
    if (rfid.uid.uidByte[i] < 0x10) Serial.print("0");
    Serial.print(rfid.uid.uidByte[i], HEX);
    Serial.print(" ");
  }
  Serial.println();

  rfid.PICC_HaltA();
  rfid.PCD_StopCrypto1();
}
