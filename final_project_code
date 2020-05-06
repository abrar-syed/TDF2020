
#include  "MedianFilterLib.h"
#include <Adafruit_NeoPixel.h>
#include <SPI.h>
#include <SD.h>
#include <Adafruit_VS1053.h>

// Music Maker Featherwing Pins
#define VS1053_RESET   -1     // VS1053 reset pin (not used!)

#define VS1053_CS       6     // VS1053 chip select pin (output)
#define VS1053_DCS     10     // VS1053 Data/command select pin (output)
#define CARDCS          5     // Card chip select pin
// DREQ should be an Int pin *if possible* (not possible on 32u4)
#define VS1053_DREQ     9     // VS1053 Data request, ideally an Interrupt pin

Adafruit_VS1053_FilePlayer musicPlayer = 
  Adafruit_VS1053_FilePlayer(VS1053_RESET, VS1053_CS, VS1053_DCS, VS1053_DREQ, CARDCS);
 
/*define the sensor's pins*/
uint8_t trigPin = 12;
uint8_t echoPin = 11;
 
unsigned long timerStart = 0;
int TIMER_TRIGGER_HIGH = 10;
int TIMER_LOW_HIGH = 2;
 
float timeDuration, distance, dist;

int motorPin = A0;

MedianFilter <float> medianFilter (5);
 
/*The states of an ultrasonic sensor*/
enum SensorStates {
  TRIG_LOW,
  TRIG_HIGH,
  ECHO_HIGH
};
 
SensorStates _sensorState = TRIG_LOW;

#define LED_PIN   13
#define LED_COUNT 7
#define DELAYVAL 500

Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

int neopixelTimer = 0;
int soundTimer = 0;
bool flag = false;

uint32_t green = strip.Color(0,128,0);
uint32_t red = strip.Color(255,0,0);
uint32_t yellow = strip.Color(255,255,0);
uint32_t orange = strip.Color(255,165,0);

void startTimer() {
  timerStart = millis();
}
 
bool isTimerReady(int mSec) {
  return (millis() - timerStart) < mSec;
}
 
/*Sets the data rate in bits per second and configures the pins */
void setup() {
  //Serial.begin(9600);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(motorPin, OUTPUT);
  strip.begin();
  strip.show(); // Initialize all pixels to 'off'
  strip.setBrightness(32);
  musicPlayer.begin();
  SD.begin(CARDCS);
  // Set volume for left, right channels. lower numbers == louder volume!
  musicPlayer.setVolume(10,10);
  musicPlayer.useInterrupt(VS1053_FILEPLAYER_PIN_INT);  // DREQ int
  neopixelTimer = millis();
  soundTimer = millis();
}
 
void loop() {
  if ((millis() - neopixelTimer) >= 1000){
    strip.clear();
    strip.setPixelColor(0, red);
    strip.setPixelColor(2, red);
    strip.setPixelColor(4, red);
    strip.setPixelColor(6, red);
    strip.show();
    neopixelTimer = millis();}
  else if ((millis() - neopixelTimer) >= 500){
    strip.clear();
    strip.setPixelColor(1, red);
    strip.setPixelColor(3, red);
    strip.setPixelColor(5, red);
    strip.show();}
  /*Switch between the ultrasonic sensor states*/
  switch (_sensorState) {
    /* Start with LOW pulse to ensure a clean HIGH pulse*/
    case TRIG_LOW: {
        digitalWrite(trigPin, LOW);
        startTimer();
        if (isTimerReady(TIMER_LOW_HIGH)) {
          _sensorState = TRIG_HIGH;
        }
      } break;
      
    /*Triggered a HIGH pulse of 10 microseconds*/
    case TRIG_HIGH: {
        digitalWrite(trigPin, HIGH);
        startTimer();
        if (isTimerReady(TIMER_TRIGGER_HIGH)) {
          _sensorState = ECHO_HIGH;
        }
      } break;
 
    /*Measures the time that ping took to return to the receiver.*/
    case ECHO_HIGH: {
        digitalWrite(trigPin, LOW);
        timeDuration = pulseIn(echoPin, HIGH);
        /*
           distance = time * speed of sound
           speed of sound is 340 m/s => 0.034 cm/us
        */
        dist = timeDuration * 0.034 / 2;
        distance = medianFilter.AddValue(dist);
        //Serial.print("Distance measured is: ");
        //Serial.print(distance);
        //Serial.println(" cm");
        if (distance <= 30) {
          analogWrite(motorPin,255);
          //strip.fill(red, 0);
          //strip.show();
          if ((millis() - soundTimer) >= 2000){
            musicPlayer.playFullFile("/track001.mp3");
            soundTimer = millis();
            flag = true;
            }
          }
        else if (distance <= 60 & distance > 30) {
          analogWrite(motorPin,170); 
          //strip.fill(orange, 0);
          //strip.show();
          if ((millis() - soundTimer) >= 2000){
            musicPlayer.playFullFile("/track001.mp3");
            soundTimer = millis();
            flag = true;
            }
          }
        else if (distance <= 100 & distance > 60) {
          analogWrite(motorPin,85); 
          //strip.fill(yellow, 0);
          //strip.show();
          if ((millis() - soundTimer) >= 2000){
            musicPlayer.playFullFile("/track001.mp3");
            soundTimer = millis();
            flag = true;
            }
          }
        else { analogWrite(motorPin,0); 
          //strip.fill(green, 0);
          //strip.show();
          if ((millis() - soundTimer) >= 2000 & flag){
            musicPlayer.playFullFile("/track002.mp3");
            soundTimer = millis();
            flag = false;
            }
          }
        _sensorState = TRIG_LOW;
      } break;
      
  }//end switch
  
}//end loop
