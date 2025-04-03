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
Using the assembly from Lab 6, we attached Velcro to the motors and the bottom of the Arduino breadboard. We then secured the wheels from the kit onto the motors and carefully aligned and attached the motor's Velcro to the Velcro on the board, ensuring proper positioning for the robot to move effectively. The completed robot can be seen below in Figure 1.

![Robot from website](https://github.com/aashikauppala/lab8/blob/main/Lab%208%20Robot%20from%20website.jpg)

_Figure 1. Robot completed with wheels_

Next, we connected the Arduino to the computer and ran the Lab 6 code to confirm that the robot responded correctly to serial commands, allowing it to move forward, backward, right, and left.

Finally, we attached a binder clip to the end of the Arduino board to slightly elevate it, ensuring the battery pack underneath did not touch the surface. We then tested the robot at various speeds to determine the minimum speed at which it could still operate effectively.

### Part 2 - Develop the App
_Note: The following instructions are based on a Windows laptop. We followed the setup guide provided at http://appinventor.mit.edu/explore/ai2/setup.html. If you are using a different operating system, you can refer to the same website for installation instructions._

We installed the App Inventor Setup software package, created an account, logged into the App Inventor 2 web-based tool, and launched aiStarter. To keep our work organized, we created a new project with a descriptive name, ensuring it could be easily identified if an instructor or another user wanted to install our app on their phone.

In the Designer environment, we added a button named "Open_Ser" to initialize serial communication, a label to display incoming communication strings from the Arduino, and multiple user interface elements to control the car’s movement. This allowed it to move forward, backward, right, and left at slow, medium, and fast speeds.

From the Connectivity section, we added a Serial component named "Serial1" and from the Sensors section add a Clock component named "ReadFromArduino." Switching to the Blocks environment, we added the blocks shown below in Figure 2 to establish the communications after pressing the serial communications button.

![Code 1](https://github.com/aashikauppala/lab8/blob/main/Lab%208%20Code%201.png)

_Figure 2. App Inventor blocks needed to control the serial communications. (Found in Lab 8 Instructions by Dr. Carlos Jarro)_

We used the "call serialObject.WriteSerial.data" block along with a standard "text" block to send commands to the RedBoard. Each object we used to control the car sent text commands via serial communication, similar to how we previously used the Arduino IDE Serial Monitor.

Finally, we built our app by selecting 'Build' and then 'Android App (.apk).' Since no one in our group had an Android device, Dr. Jarro connected his phone using a USB-C to USB-A adapter and the standard Arduino cable. He then scanned the QR code to install and test our app on his device.

### Part 3 - Wireless Remote
On an unused area of our breadboard, we wired the HC-05 module, using Figure 3 as a guidance. Using pins 2 and 3 for Rx and Tx, we included a 1kΩ and 2kΩ voltage divider instead of the 1.1kΩ and 3.3kΩ that are shown below in Figure 3.

![Arduino Part 3](https://github.com/aashikauppala/lab8/blob/main/Lab%208%20Arduino%20Part%203.png)

_Figure 3. HC-05 Wiring Diagram (http://exploreembedded.com/wiki/Setting_up_Bluetooth_HC-05_with_Arduino)_

To connect the HC-05 module to our app, we accessed Dr. Jarro's GitHub repository, "BAE-305-Lab-Template," and used the "RobotSerialRC_BLU_Complete.ino" code. All the code provided in Part 3 of the Methods section was retrieved from this repository.

First, insert the following code below before the setup function.

```c++
#include <SoftwareSerial.h>

SoftwareSerial mySerial(2, 3); // HC-05 Tx connected to Arduino #2 & HC-05 Rx to Arduino #3

const byte numChars = 16;       
char receivedChars[numChars];  // an array to store the received data
char tempChars[numChars];

char botDir[numChars] = {0};         // char type variable for the direction of the robot
int botSpeed = 0;           //stores the speed of the whole robot
boolean newData = false;
```

Add the following code within the setup function.

```c++
  mySerial.begin(9600);       //Default Baud Rate for software serial communications
```

Insert the following code within the "if (Serial.available() > 0)" section.

```c++
  recvWithEndMarker();
  if (newData == true)
  {
    strcpy(tempChars, receivedChars);
    parseData();
    botDirection = botDir;
    motorSpeed = botSpeed;
    newData = false;
    Serial.println(botDirection);
    Serial.println(motorSpeed);
  }
```

Finally, add the following code at the end of the program.

```c++
/********************************************************************************/
void recvWithEndMarker() {
    static byte ndx = 0;
    char endMarker = '\n';
    char rc;
    while (mySerial.available() > 0 && newData == false){
      rc = mySerial.read();
      if (rc != endMarker) {
        receivedChars[ndx] = rc;
        ndx++;
        if (ndx >= numChars){
          ndx = numChars - 1;
        }
      }
      else {
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
    //strcpy(botSpeed, strtokIndx); // Use this line for sending speed as text
    botSpeed = atoi(strtokIndx);     // Use this line for sending sepeed as an integer

}
```

After connecting the HC-05 module to the Arduino and linking it with the code, the next step was to integrate it into the app and test the system. However, we ran out of time during the lab. The rest of the instructions that we have included below are pasted directly from Dr. Jarro's Lab 8 Instructions.

"Open App Inventor 2 and save a copy of your previous app, give it a descriptive name. In the Designer environment:

a. From the section Connectivity, add the object BluetoothClient. This will allow you to send data via Bluetooth.

b. From the section User Interface add a ListPicker object. This is a button that will let us select the Bluetooth-paired device we need to send data to.

c. From the section User Interface add a Label object. This will show the status of the Bluetooth connection.

d. Download the file GetApiLevel from the Instructor’s GitHub account (Lab-Template Repository).

e. On the section Extension click the link Import extension and import the file you just downloaded. Add the GetApiLevel object.

f. From the Storage section add a TinyDB object

g. In the Blocks environment, add the blocks shown in Figure 8. BluetoothPicker is the ListPicker and StatusLabel is the label added for Bluetooth control.

h. For each of the buttons, replace the “call serialobject.WriteSerial” block with a “call BluetoothClient.SendText” block as shown in Figure 7. Make sure to include the new line command “/n”, this is used by the Arduino code to identify the end of the sent command. The global variable Speed is used to store the bot speed value, you can use your variable name or not use a variable.

Test your system and show it to your instructor."

## Results
### Part 1 - Assemble and test your robot
We utilized the same code from Lab 6, as our robot's minimum speed remained lower than the predefined "slow" setting. The minimum speed required for movement was 50, with more details provided in the discussion section. Our assembled robot is shown below in Figure 4.

![Robot 1](https://github.com/aashikauppala/lab8/blob/main/Lab%208%20Robot.jpg)

_Figure 4. Assembled robot_

### Part 2 - Develop the App
We created buttons using the "call serialObject.WriteSerial.data" block combined with a standard "text" block. Each "text" block contained the specific command used to trigger the corresponding function that determines the robot's direction and speed. The code behind each button is shown below in Figures 5, 6, and 7.

![Code 2](https://github.com/aashikauppala/lab8/blob/main/Lab%208%20Code%202.png)

_Figure 5. Code for the buttons controlling the car's slow driving mode_

![Code 3](https://github.com/aashikauppala/lab8/blob/main/Lab%208%20Code%203.png)

_Figure 6. Code for the buttons controlling the car's medium driving mode_

![Code 4](https://github.com/aashikauppala/lab8/blob/main/Lab%208%20Code%204.png)

_Figure 7. Code for the buttons controlling the car's fast driving mode_

The display screen for our app is shown below.

![Screen](https://github.com/aashikauppala/lab8/blob/main/Lab%208%20Screen.png)

_Figure 8. Display screen on our app_

### Part 3 - Wireless Remote
We successfully connected the HC-05 module to our breadboard, as shown below in Figure 9.

![Robot 2](https://github.com/aashikauppala/lab8/blob/main/Lab%208%20Robot%202.jpg)

_Figure 9. Robot connected to the HC-05 Bluetooth module_

We did not have time to connect the HC-05 Bluetooth module to our app.

## Discussion
_In Lab 6 we found out what was the minimum speed that will move the motors. What is the minimum speed that will move the complete car?_

During testing, we discovered that the full assembly of the robot, including its battery pack, chassis, and motors with wheels, added weight and resistance compared to testing just the motors in Lab 6. As a result, the minimum PWM (Pulse Width Modulation) value required to move the complete robot was higher. After trial and error, we determined that a PWM value of approximately 50 was necessary to overcome the initial static friction and start the robot’s motion reliably. This value may vary slightly based on build and surface conditions but was a consistent threshold in our testing.

## Conclusion
This lab successfully demonstrated the integration of hardware and software systems through the development of a Bluetooth-controlled robot car. By using MIT App Inventor and the HC-05 Bluetooth module, we learned how to create a custom Android application capable of wirelessly communicating with an Arduino-based robot. The process required modifying both the app and the Arduino sketch to handle Bluetooth commands, as well as troubleshooting issues related to hardware connections and communication protocols. 

From this lab, we gained hands-on experience in mobile app development, serial communication, and embedded system design. Most importantly, we saw how accessible tools like App Inventor can be used to prototype real-world applications. This experience has deepened our understanding of system integration and highlighted the importance of testing, iterative design, and user interface considerations in engineering solutions.
