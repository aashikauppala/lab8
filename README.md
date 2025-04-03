# Lab 8 - There’s an App for That: App Development
Megan Fister, Aashika Uppala

4/2/2025
## Introduction
The goal of this lab was to design and implement a mobile app capable of wirelessly controlling a robot car using Bluetooth communication. This project builds upon previous labs by integrating hardware components such as motors, sensors, and the SparkFun RedBoard with software tools including Arduino IDE and MIT App Inventor. By the end of the lab, the robot car should be able to move forward, backward, left, and right at varying speeds via commands sent from a smartphone.
To achieve this, we first assembled the robot car and tested basic motion through wired serial communication. We then used MIT App Inventor to create a simple Android app capable of sending directional commands to the car. In the final stage, we implemented Bluetooth functionality using the HC-05 module and modified both the Arduino sketch and the app to support wireless communication. 

## Methods
### Instruments
• Computer running Arduino IDE, and Chrome Browser.

• A smartphone running an Android OS with the MIT AI2 Companion app installed.

• SparkFun Inventor’s kit

&nbsp; &nbsp; &nbsp; &nbsp; o RedBoard

&nbsp; &nbsp; &nbsp; &nbsp; o Ultrasonic sensor

&nbsp; &nbsp; &nbsp; &nbsp; o Two motors

&nbsp; &nbsp; &nbsp; &nbsp; o Motor Driver

&nbsp; &nbsp; &nbsp; &nbsp; o Battery pack

• A cable/adaptor to connect the smartphone to the RedBoard.

• An HC-05 Bluetooth UART Module

### Part 1 - Assemble and test your robot
Using the assembly from lab 6, add velco to the motors as well as on the bottom of the Arduino breadboard. Attach the wheels in the kit to the motors and connect the velcro of the motor to the velcro on the board so that the two wheels are placed so that the board can move as a robot, being careful with the alignment.
Connect the Arduino to the computer and run the code from Lab 6 to verify that the robot moves forward, backward, right, and left according to the commands sent via serial communications.
Attach a binder clip to the end of the Arduino board so that the board is lifted off the surface and the battery pack on the bottom of the robot will not touch the surface.

### Part 2 - Develop the App

### Part 3 - Wireless Remote



## Results
Code
```c++
/*
  SparkFun Inventor’s Kit
  Circuit 5B - Remote Control Robot

  Control a two wheeled robot by sending direction commands through the serial monitor.
  This sketch was adapted from one of the activities in the SparkFun Guide to Arduino.
  Check out the rest of the book at
  https://www.sparkfun.com/products/14326

  This sketch was written by SparkFun Electronics, with lots of help from the Arduino community.
  This code is completely free for any use.

  View circuit diagram and instructions at: https://learn.sparkfun.com/tutorials/sparkfun-inventors-kit-experiment-guide---v40
  Download drawings and code at: https://github.com/sparkfun/SIK-Guide-Code

  Modified by:
  Carlos Jarro for University of Kentucky's BAE305 Lab 6
  02/28/2024
*/
#include <SoftwareSerial.h>

SoftwareSerial mySerial(2, 3); // HC-05 Tx connected to Arduino #2 & HC-05 Rx to Arduino #3

const byte numChars = 6;       
char receivedChars[numChars];  // an array to store the received data
char tempChars[numChars];

char botDir[1];         // char type variable for the direction of the robot
//                         botDirection[0] = 'f' (forward), 'b' (backwards)
//                         'r' (right), 'l' (left), 's' (stop)
int botSpeed = 0;           //stores the speed of the whole robot
boolean newData = false;

//the right motor will be controlled by the motor A pins on the motor driver
const int AIN1 = 13;           //control pin 1 on the motor driver for the right motor
const int AIN2 = 12;            //control pin 2 on the motor driver for the right motor
const int PWMA = 11;            //speed control pin on the motor driver for the right motor

//the left motor will be controlled by the motor B pins on the motor driver
const int PWMB = 10;           //speed control pin on the motor driver for the left motor
const int BIN2 = 9;           //control pin 2 on the motor driver for the left motor
const int BIN1 = 8;           //control pin 1 on the motor driver for the left motor

const int trigPin = 6;        //trigger pin for distance snesor
const int echoPin = 7;        //echo pin for distance sensor
const int RED = 5;
const int GREEN = 4;


String botDirection;           //the direction that the robot will drive in (this change which direction the two motors spin in)
String motorSpeedStr;

int motorSpeed;               //speed integer for the motors
float duration, distance;     //duration and distance for the distance sensor

/********************************************************************************/
void setup()
{
  mySerial.begin(9600);       //Default Baud Rate for software serial communications

  //set the motor control pins as outputs
  pinMode(AIN1, OUTPUT);
  pinMode(AIN2, OUTPUT);
  pinMode(PWMA, OUTPUT);

  pinMode(BIN1, OUTPUT);
  pinMode(BIN2, OUTPUT);
  pinMode(PWMB, OUTPUT);
  
  //set the distance sensor trigger pin as output and the echo pin as input
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  pinMode(RED,OUTPUT);
  pinMode(GREEN,OUTPUT);

  Serial.begin(9600);           //begin serial communication with the computer

  //prompt the user to enter a command
  Serial.println("Enter a direction followed by speed.");
  Serial.println("f = forward, b = backward, r = turn right, l = turn left, s = stop");
  Serial.println("Example command: f 50 or s 0");
}

/********************************************************************************/
void loop()
{
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH);
  distance = (duration*340/10000)/2; // Units are cm
  //Serial.print("Distance: ");
  //Serial.println(distance);
    delayMicroseconds(50);

  if (Serial.available() > 0)                         //if the user has sent a command to the RedBoard
  {
    botDirection = Serial.readStringUntil(' ');       //read the characters in the command until you reach the first space
    motorSpeedStr = Serial.readStringUntil(' ');           //read the characters in the command until you reach the second space
    motorSpeed = motorSpeedStr.toInt();
    Serial.println(botDirection);
  }

  recvWithEndMarker();
  if (newData == true)
  {
    strcpy(tempChars, receivedChars);
    parseData();
    motorSpeed = botSpeed;
    newData = false;
    Serial.println(botDir[0]);
    Serial.println(motorSpeed);
   
  }

  if (distance > 10)
  {                                                     //if the switch is in the ON position
    if ((botDir[0] == 'f') || (botDirection == "f"))                         //if the entered direction is forward
    {
      rightMotor(-motorSpeed);                                //drive the right wheel forward
      leftMotor(motorSpeed);                                 //drive the left wheel forward
      digitalWrite(GREEN,HIGH);
      digitalWrite(RED,LOW);
    }
    else if ((botDir[0] == 'b') || (botDirection == "b"))                    //if the entered direction is backward
    {
      rightMotor(motorSpeed);                               //drive the right wheel forward
      leftMotor(-motorSpeed);                                //drive the left wheel forward
      digitalWrite(GREEN,HIGH);
      digitalWrite(RED,LOW);
    }
    else if ((botDir[0] == 'r') || (botDirection == "r"))                     //if the entered direction is right
    {
      rightMotor(motorSpeed);                               //drive the right wheel forward
      leftMotor(motorSpeed);                                 //drive the left wheel forward
      digitalWrite(GREEN,HIGH);
      digitalWrite(RED,LOW);
    }
    else if ((botDir[0] == 'l') || (botDirection == "l"))                   //if the entered direction is left
    {
      rightMotor(-motorSpeed);                                //drive the right wheel forward
      leftMotor(-motorSpeed);                                //drive the left wheel forward
      digitalWrite(GREEN,HIGH);
      digitalWrite(RED,LOW);
    }
    else if ((botDir[0] == 's') || (botDirection == "s"))
    {
      rightMotor(0);
      leftMotor(0);
      digitalWrite(GREEN,LOW);
      digitalWrite(RED,HIGH);
    }
  }
  else if (distance < 10)
  {
    if ((botDir[0] == 'b') || (botDirection == "b"))
    {
      rightMotor(motorSpeed);                               //drive the right wheel forward
      leftMotor(-motorSpeed);                                //drive the left wheel forward
    }
    else
    {
      Serial.print("Object Detected at ");
      Serial.print(distance);
      Serial.println(" cm");
      rightMotor(0);                                  //turn the right motor off
      leftMotor(0);                                   //turn the left motor off
    }
  }
}
/********************************************************************************/
void rightMotor(int motorSpeed)                       //function for driving the right motor
{
  if (motorSpeed > 0)                                 //if the motor should drive forward (positive speed)
  {
    digitalWrite(AIN1, HIGH);                         //set pin 1 to high
    digitalWrite(AIN2, LOW);                          //set pin 2 to low
  }
  else if (motorSpeed < 0)                            //if the motor should drive backward (negative speed)
  {
    digitalWrite(AIN1, LOW);                          //set pin 1 to low
    digitalWrite(AIN2, HIGH);                         //set pin 2 to high
  }
  else                                                //if the motor should stop
  {
    digitalWrite(AIN1, LOW);                          //set pin 1 to low
    digitalWrite(AIN2, LOW);                          //set pin 2 to low
  }
  analogWrite(PWMA, abs(motorSpeed));                 //now that the motor direction is set, drive it at the entered speed
}

/********************************************************************************/
void leftMotor(int motorSpeed)                        //function for driving the left motor
{
  if (motorSpeed > 0)                                 //if the motor should drive forward (positive speed)
  {
    digitalWrite(BIN1, HIGH);                         //set pin 1 to high
    digitalWrite(BIN2, LOW);                          //set pin 2 to low
  }
  else if (motorSpeed < 0)                            //if the motor should drive backward (negative speed)
  {
    digitalWrite(BIN1, LOW);                          //set pin 1 to low
    digitalWrite(BIN2, HIGH);                         //set pin 2 to high
  }
  else                                                //if the motor should stop
  {
    digitalWrite(BIN1, LOW);                          //set pin 1 to low
    digitalWrite(BIN2, LOW);                          //set pin 2 to low
  }
  analogWrite(PWMB, abs(motorSpeed));                 //now that the motor direction is set, drive it at the entered speed
}

/********************************************************************************/
void recvWithEndMarker() {
    static byte ndx = 0;
    char endMarker = '\n';
    char rc;
    while (mySerial.available() > 0 && newData == false)
    {
      rc = mySerial.read();

      if (rc != endMarker)
      {
        receivedChars[ndx] = rc;
        ndx++;
        if (ndx >= numChars)
        {
          ndx = numChars - 1;
        }
      }
      else
      {
        receivedChars[ndx] = '\0'; // terminate the string
        ndx = 0;
        newData = true;
      }
    }
}
/*****************************************************************************************/
void parseData() {      // split the data into its parts

    char * strtokIndx; // this is used by strtok() as an index

    strtokIndx = strtok(tempChars," ");      // get the first part - the string
    strcpy(botDir, strtokIndx); // copy it to messageFromPC
 
    strtokIndx = strtok(NULL, " "); // this continues where the previous call left off
    botSpeed = atoi(strtokIndx);     // convert this part to an integer
}
```


## Discussion
_In Lab 6 we found out what was the minimum speed that will move the motors. What is the minimum speed that will move the complete car?_

During testing, we discovered that the full assembly of the robot, including its battery pack, chassis, and motors with wheels, added weight and resistance compared to testing just the motors in Lab 6. As a result, the minimum PWM (Pulse Width Modulation) value required to move the complete robot was higher. After trial and error, we determined that a PWM value of approximately 50 was necessary to overcome the initial static friction and start the robot’s motion reliably. This value may vary slightly based on build and surface conditions but was a consistent threshold in our testing.

## Conclusion
This lab successfully demonstrated the integration of hardware and software systems through the development of a Bluetooth-controlled robot car. By using MIT App Inventor and the HC-05 Bluetooth module, we learned how to create a custom Android application capable of wirelessly communicating with an Arduino-based robot. The process required modifying both the app and the Arduino sketch to handle Bluetooth commands, as well as troubleshooting issues related to hardware connections and communication protocols. 

From this lab, we gained hands-on experience in mobile app development, serial communication, and embedded system design. Most importantly, we saw how accessible tools like App Inventor can be used to prototype real-world applications. This experience has deepened our understanding of system integration and highlighted the importance of testing, iterative design, and user interface considerations in engineering solutions.
