# Arduino-Based-Car-Automatic-Lights
// These constants won't change:
#include <DHT.h>
#include <Wire.h>
#include<LiquidCrystal.h>
const int analogPin = A0;
const int analogPin2 = A1;// pin that the sensor is attached to
const int FULLBEAM = 5; // pin that the LED is attached to
const int LOWBEAM = 6;
const int threshold = 400; // an arbitrary threshold level that's in the range of the analog input
const int trigPin = 9;
const int echoPin = 10;
const int WARNINGL = 4;
const int RIGHTS = 8;
const int LEFTS = 7;
long duration;
int distance;
const int MPU=0x68; //I2C address of MPU
int GyX,GyY,GyZ;
float pitch=0;
float roll=0;
float yaw=0;
float v_pitch;
float v_roll;
float v_yaw;
float a_pitch;
float a_roll;
float a_yaw;
//Constants41
#define DHTPIN 2 // what pin we're connected to
#define DHTTYPE DHT11 // DHT 11 (AM2302)
// Initialize DHT sensor for normal 16mhz Arduino
DHT dht(DHTPIN, DHTTYPE);
int chk;
float hum; //Stores humidity value
float temp; //Stores temperature value
void setup() {
pinMode(trigPin, OUTPUT); // Sets the trigPin as an Output
pinMode(echoPin, INPUT); // Sets the echoPin as an Input
pinMode(WARNINGL, OUTPUT);
pinMode(FULLBEAM, OUTPUT);
pinMode(LOWBEAM, OUTPUT);
pinMode(RIGHTS, OUTPUT);
pinMode(LEFTS, OUTPUT);
Wire.begin();
Wire.beginTransmission(MPU);
Wire.write(0x6B); //power management register 1
Wire.write(0);
Wire.endTransmission(true);
// initialize serial communications:
Serial.begin(9600);
dht.begin();
}
void loop() {
// read the value of the potentiometer:
int analogValue = analogRead(analogPin2);
int analogValue1 = analogRead(analogPin);42
// if the analog value is high enough, turn on the LED:
if (analogValue > threshold) {
digitalWrite(FULLBEAM, LOW);
}
else {
digitalWrite(FULLBEAM, HIGH);
}
if((analogValue < threshold)&&(analogValue1 > threshold)) {
digitalWrite(LOWBEAM, HIGH);
Serial.print(" -> LOW");
}
else {
digitalWrite(LOWBEAM, LOW);}
if ((analogValue < threshold)&&(analogValue1 < threshold)) {
digitalWrite(FULLBEAM, HIGH);
Serial.print(" -> FULL");
}
else {
digitalWrite(FULLBEAM, LOW);
}
// print the analog value:
Serial.print("LDR2:\t");
Serial.println(analogValue);
Serial.print("LDR1:\t");
Serial.println(analogValue1);
digitalWrite(trigPin, LOW);
delayMicroseconds(2);
// Sets the trigPin on HIGH state for 10 micro seconds
digitalWrite(trigPin, HIGH);
delayMicroseconds(10);43
digitalWrite(trigPin, LOW);
// Reads the echoPin, returns the sound wave travel time in microseconds
duration = pulseIn(echoPin, HIGH);
// Calculating the distance
distance= duration*0.034/2;
// Prints the distance on the Serial Monitor
Serial.print("Distance: ");
Serial.println(distance);
if (distance < 20) {
digitalWrite(WARNINGL, HIGH);
delay (100);
digitalWrite(WARNINGL, LOW);
delay (100);
}
else {
digitalWrite(WARNINGL, LOW);
}
//Read data and store it to variables hum and temp
hum = dht.readHumidity();
temp= dht.readTemperature();
//Print temp and humidity values to serial monitor
Serial.print("Humidity: ");
Serial.print(hum);
Serial.print(" %, Temp: ");
Serial.print(temp);
Serial.println(" Celsius");
delay(200); //Delay 0.2 sec.
if (hum > 80) {
digitalWrite(FULLBEAM, HIGH);44
}
else {
digitalWrite(FULLBEAM, LOW);
}
Wire.beginTransmission(MPU);
Wire.write(0x43);
//starts with MPU register 43(GYRO_XOUT_H)
Wire.endTransmission(false);
Wire.requestFrom(MPU,6,true);
//requests 6 registers
GyX=Wire.read()<<8|Wire.read();
GyY=Wire.read()<<8|Wire.read();
GyZ=Wire.read()<<8|Wire.read();
v_pitch=(GyX/131);
if(v_pitch==-1)
//error filtering
{
v_pitch=0;
}
v_roll=(GyY/131);
if(v_roll==1)
//error filtering
{v_roll=0;}
v_yaw=GyZ/131;
a_pitch=(v_pitch*0.046);
a_roll=(v_roll*0.046);
a_yaw=(v_yaw*0.045);
pitch= pitch + a_pitch;
roll= roll + a_roll;
yaw= yaw + a_yaw; Serial.print(" | pitch = ");45
Serial.print(pitch);
Serial.print(" | roll = ");
Serial.print(roll);
Serial.print(" | yaw = ");
Serial.println(yaw);
if (yaw > 0.1) {
digitalWrite(RIGHTS, HIGH);
Serial.print("RIGHTS");
}
else {
digitalWrite(RIGHTS, LOW);
}
if (yaw < -0.1) {
digitalWrite(LEFTS, HIGH);
Serial.print(" LEFTS ");
}
else {
digitalWrite(LEFTS, LOW);
}
}
