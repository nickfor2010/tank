# tank
managing tank fresh water

#include <LiquidCrystal.h>
#include <Wire.h>
#include <DS3231.h>
#include <NewPing.h>

LiquidCrystal lcd(4, 5, 6, 7, 9, 10);

#define TRIGGER_PIN  12  // Arduino pin tied to trigger pin on the ultrasonic sensor.
#define ECHO_PIN     11  // Arduino pin tied to echo pin on the ultrasonic sensor.
#define MAX_DISTANCE 200 // Maximum distance we want to ping for (in centimeters). Maximum sensor distance is rated at 400-500cm.

NewPing sonar(TRIGGER_PIN, ECHO_PIN, MAX_DISTANCE); // NewPing setup of pins and maximum distance.
int i = 0;
int cm = 0;
int t = 100;

DS3231 clock;
RTCDateTime dt;

void setup() 
{
  
// LED CHECK

   for (int i=32; i <= 46; i=i+2){
      digitalWrite(i, HIGH);
      delay(t);
   }
   for (int i=46; i >= 32; i=i-2){
      digitalWrite(i, LOW);
      delay(t);
   }
   
// CLOCK BEGIN

  clock.begin();
  clock.armAlarm1(false);
  clock.armAlarm2(false);
  clock.clearAlarm1();
  clock.clearAlarm2();
  // clock.setDateTime(2017, 4, 5, 17, 6, 40);
  clock.setAlarm1(0, 18, 0, 0, DS3231_MATCH_H_M_S);
  clock.setAlarm2(0, 18, 15, DS3231_MATCH_H_M);

// SONAR

  pinMode(8, OUTPUT); // PIN POMP FILL THE TANK
  digitalWrite(8, HIGH);
  pinMode(2, OUTPUT); // PIN POMP CLEAN SURGFACE TANK
  digitalWrite(2, HIGH);
  digitalWrite(46, HIGH);
}

void loop() {

// CHECK ALLARMS ON/OFF CLEANING TANK WATER SURFACE 

  dt = clock.getDateTime();
  if (clock.isAlarm1())
  {
  digitalWrite(2, LOW); // START Clean surface tank
  digitalWrite(34, HIGH); // Clean surface tank LED ON
  }
  if (clock.isAlarm2())
  {
  digitalWrite(2, HIGH); // STOP Clean surface tank
  digitalWrite(34, LOW); // Clean surface tank LED OFF
  }

// PRINT ON LCD

  lcd.begin(16, 2); 
  lcd.clear();
  lcd.print(cm); lcd.print(" cm "); lcd.print(i); lcd.print(" t ");
  lcd.setCursor(11, 0);
  lcd.println(clock.readTemperature()); 
  lcd.setCursor(0, 1);
  lcd.print(clock.dateFormat("d-m-Y H:i:s ", clock.getDateTime()));

// WATER LEVEL CONTROL

  delay(500);  // Wait 500ms between pings (about 2 pings/sec). 29ms should be the shortest delay between pings.
  unsigned int uS = sonar.ping(); // Send ping, get ping time in microseconds (uS).
  
  cm = uS / US_ROUNDTRIP_CM;
 
  if(uS / US_ROUNDTRIP_CM > 9)
  {    // REFIL WATER IN TANK
    
        delay(60000);
        digitalWrite(8, LOW); // START POMP FILL THE TANK
        digitalWrite(32, HIGH); // Refil tank LED
        
        delay(3000);
        i=i+1;
  }else{
        digitalWrite(8, HIGH);
        digitalWrite(32, LOW);
        }
}
