//Created by Ryan Glaser & Donald Glaser for long-term soil temperature and humidity profiles
//Please cite Glaser et al. 2021. Astrobiology. DOI:

//Necessary libraries for hardware and software functionality
#include <HIH61Donny.h>
#include <TimeLib.h>
#include <Wire.h>
#include <Narcoleptic.h>
#include <TimerOne.h>
#include "SD.h"
#include "SPI.h"
#include <DS1307RTC.h>

//Creates each sensor address; place in order from top to bottom of array. Ensure numSensors equals array size of sensors[]
HIH61Donny sensors[] = {HIH61Donny(0x22),HIH61Donny(0x23),HIH61Donny(0x24),HIH61Donny(0x25),HIH61Donny(0x26),HIH61Donny(0x27)};

double values[2]; //Creates array for placement of sensor data (humidity & temperature)
int counter = 0;
String fileName = "";
bool skip = true;
String output;
int compDay;

//Input Sensor Array parameters here:
int numSensors = 6; //Number of sensors in array
int numMeasurements = 10; //Number of measurements per measurement cycle
int delay = 20; // length of delay between measurements (in minutes)
delay = delay * 60000; //convert delay to milliseconds
String sampleSite = "TEST1"; //Name of site

void setup() {

// Nuts & Bolts functions
pinMode(13, OUTPUT);
pinMode(A3, INPUT);
pinMode(10, OUTPUT);
pinMode(6, OUTPUT);
pinMode(LED_BUILTIN, OUTPUT);
Wire.begin();
Serial.begin(9600);
Serial.println("SERIAL OK");
digitalWrite(6, HIGH);
delay(500);
setSyncProvider(RTC.get);  //set function to call when sync required
compDay = int(day());
Serial.println("Time OK");
digitalWrite(6, LOW);
long startTime = millis();
sampleSite += "\n";
fileName = String(year()) + printDigitz(month()) + printDigitz(day());
fileName.remove(0, 2);
fileName = String(fileName);
fileName += ".csv";
output = "Time";
output += ", Number";
output += ", Light";
for(int i=1; i<numSensors + 1; i++){
  output += ", Humidity";
  output += i;
  output += ", Temperature";
  output += i;
}
output += "\n";
skip = false;


if (!SD.begin(10)) { // Starts SD card communication
  while(!SD.begin(10)) { //if SD card doesnt work, LED will blink 3 times repeatedly.
    Serial.println("SD FAIL");
    digitalWrite(LED_BUILTIN, HIGH);
    delay(100);
    digitalWrite(LED_BUILTIN, LOW);
    delay(50);
    digitalWrite(LED_BUILTIN, HIGH);
    delay(100);
    digitalWrite(LED_BUILTIN, LOW);
    delay(50);
    digitalWrite(LED_BUILTIN, HIGH);
    delay(100);
    digitalWrite(LED_BUILTIN, LOW);
    delay(1000);
  }
  return;
}
File sensorData = SD.open(fileName, FILE_WRITE);
sensorData.print(sampleSite);
sensorData.println(output);
sensorData.close();
delay(500);
for(int i=0; i<20; i++) { //blink LED 20 times to indicate setup worked correctly
  digitalWrite(LED_BUILTIN, HIGH);
  delay(100);
  digitalWrite(LED_BUILTIN, LOW);
  delay(50);
  Serial.println("SETUP OK");
}
Serial.end(9600);
}

// Main loop that measures and saves the T, RH, and light data
void loop(){
  digitalWrite(6, HIGH);
  delay(500);
  setSyncProvider(RTC.get);  //set function to call when sync required
  if(compDay != int(day())) {
    fileName = String(year()) + printDigitz(month()) + printDigitz(day());
    fileName.remove(0, 2);
    fileName = String(fileName);
    compDay = int(day());
  }

  else {
    for (int i=0; i<10; i++) {
      counter = i;
      readSensor();
      delay(500);
    }
    counter = 0;
    digitalWrite(6, LOW);
    delay(500);
    Narcoleptic.delay(delay); //milliseconds
  }
}

// Support functions
void readSensor (){
  saveData(timeStamp(), true);
  saveData(", " + String(counter), true);
  saveData(", " + String(analogRead(A3)), true);
  delay(500);
  for(int i=0;i<numSensors;i++){
    //  start the sensor
    sensors[i].start();
    sensors[i].update();
    if(sensors[i].error() == 0){
      values[0]=sensors[i].humidity()*100; //Measurement of humidity (fraction * 100 = %)
      values[1]=sensors[i].temperature(); //Measurement of temperature (ºC)
      sensors[i].stop();
    }
    else{
      values[0]=0;
      values[1]=0;
      sensors[i].stop();
    }
    saveData("",false);
    delay(50);
  }
  saveData("\n",true);
}

String timeStamp (){ //Places unix timestamp for each measurement
  String temp;
  temp = printDigitz(hour());
  temp += printDigits(minute()) + printDigits(second());
  temp += " " + printDigitz(day()) + " " + printDigitz(month()) + " " + String(year());
  return temp;
}

String printDigits(int digits){ // utility function for digital clock display: prints preceding colon and leading 0 for minutes and seconds
  String temp = ":";
  if(digits < 10)
    temp += "0";
  temp += String(digits);
  return temp;
}

String printDigitz(int digits){ // utility function for digital clock display: prints leading 0 for hour, month, and day
  String temp2 = "";
  if(digits < 10)
    temp2 += "0";
  temp2 += String(digits);
  return temp2;
}

void saveData(String text, bool format){
  if(SD.exists(fileName)){ // check the card is still there
  // now append new data file
    File sensorData = SD.open(fileName, FILE_WRITE);
    if (sensorData){
      if(format){
      sensorData.print(text);
      }
      else{
        sensorData.print(", ");
        sensorData.print(values[0]) ;
        sensorData.print(", ");
        sensorData.print(values[1]);
      }
      sensorData.close(); // close the file
    }
  }
}
