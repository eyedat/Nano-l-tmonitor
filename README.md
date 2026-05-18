# Nano-l-tmonitor
Arduino-Nano-NTC-OLED-SolderMonitor
# Arduino Nano OLED Solder Monitor

DIY Temperaturanzeige für Lötkolben / Heizelemente.

## Hardware

- Arduino Nano ATmega328
- SSD1306 OLED I2C
- 100K NTC 3950
- 10K Widerstand

## Features

- Große Temperaturanzeige
- OLED SSD1306
- Aufheizsymbol ↑
- Lötbereit ✓
- Überhitzung ↓
- Statusanzeige
- Serial Monitor Debug

## Verdrahtung

5V -> NTC -> A0 -> 10K -> GND

OLED:
- SDA -> A4
- SCL -> A5

## Libraries

- Adafruit SSD1306
- Adafruit GFX

## Screenshot

(Später Bild einfügen)

## Lizenz

ohne

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <math.h>

// =====================================
// OLED Einstellungen
// =====================================

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1

Adafruit_SSD1306 display(
  SCREEN_WIDTH,
  SCREEN_HEIGHT,
  &Wire,
  OLED_RESET
);

// =====================================
// NTC Einstellungen
// =====================================

#define THERMISTORPIN A0

// 10K Widerstand
#define SERIESRESISTOR 10000

// 100K NTC 3950
#define THERMISTORNOMINAL 100000
#define TEMPERATURENOMINAL 25
#define BCOEFFICIENT 3950

// =====================================

void setup() {

  Serial.begin(9600);

  // OLED starten
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {

    Serial.println("OLED Fehler");

    while(true);
  }

  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);

  display.setTextSize(2);
  display.setCursor(15,20);
  display.println("START");

  display.display();

  delay(1500);
}

// =====================================
// Symbol Funktionen
// =====================================

// Pfeil nach oben
void drawUpArrow(int x, int y) {

  display.drawLine(x, y+10, x, y, SSD1306_WHITE);

  display.drawLine(x, y, x-4, y+4, SSD1306_WHITE);
  display.drawLine(x, y, x+4, y+4, SSD1306_WHITE);
}

// Häkchen
void drawCheck(int x, int y) {

  display.drawLine(x-3, y, x, y+4, SSD1306_WHITE);
  display.drawLine(x, y+4, x+6, y-4, SSD1306_WHITE);
}

// Pfeil nach unten
void drawDownArrow(int x, int y) {

  display.drawLine(x, y, x, y+10, SSD1306_WHITE);

  display.drawLine(x, y+10, x-4, y+6, SSD1306_WHITE);
  display.drawLine(x, y+10, x+4, y+6, SSD1306_WHITE);
}

// =====================================

void loop() {

  int adcValue = analogRead(THERMISTORPIN);

  // Fehlerprüfung
  if(adcValue <= 1 || adcValue >= 1022) {

    display.clearDisplay();

    display.setTextSize(2);
    display.setCursor(0,20);
    display.println("NTC ERROR");

    display.display();

    delay(1000);

    return;
  }

  // =====================================
  // Widerstand berechnen
  // Für:
  // 5V -> NTC -> A0 -> 10K -> GND
  // =====================================

  float resistance;

  resistance = SERIESRESISTOR * ((1023.0 / adcValue) - 1.0);

  // =====================================
  // Temperatur berechnen
  // =====================================

  float steinhart;

  steinhart = resistance / THERMISTORNOMINAL;
  steinhart = log(steinhart);
  steinhart /= BCOEFFICIENT;
  steinhart += 1.0 / (TEMPERATURENOMINAL + 273.15);
  steinhart = 1.0 / steinhart;
  steinhart -= 273.15;

  // =====================================
  // Status Text
  // =====================================

  String statusText = "";

  if(steinhart < 280) {

    statusText = "Aufheizen";

  } else if(steinhart >= 280 && steinhart < 390) {

    statusText = "Bleilot/Bleifrei";

  } else {

    statusText = "UEBERHITZT!";
  }

  // =====================================
  // Serial Ausgabe
  // =====================================

  Serial.print("ADC: ");
  Serial.print(adcValue);

  Serial.print(" | Temperatur: ");
  Serial.print(steinhart,1);

  Serial.println(" C");

  // =====================================
  // OLED Anzeige
  // =====================================

  display.clearDisplay();

  // Titel
  display.setTextSize(1);
  display.setCursor(0,0);
  display.println("Loetstation Monitor");

  // Große Temperatur
  display.setTextSize(3);
  display.setCursor(5,18);

  display.print(steinhart,1);
  display.print("C");

  // =====================================
  // Symbol rechts neben Temperatur
  // =====================================

  if(steinhart < 280) {

    drawUpArrow(118, 28);

  } else if(steinhart >= 280 && steinhart < 390) {

    drawCheck(118, 34);

  } else {

    drawDownArrow(118, 28);
  }

  // =====================================
  // Status unten
  // =====================================

  display.setTextSize(1);
  display.setCursor(0,54);
  display.println(statusText);

  display.display();

  delay(500);
}
