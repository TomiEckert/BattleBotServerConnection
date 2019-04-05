# Battlebot Server Communication
### Arduino
##### Table of Contents  
 - [Basics](#Basics)  
 - [Codes](#Codes)
    -  [Top](#Top)
    -  [Setup](#Setup)
    -  [Loop](#Loop)
    -  [Bottom](#Bottom)
 - [Important](#Important)
 - [Example](#Example)

### Basics
Paster the following codes to the corresponding parts of your code. Change the necessary settings. Read through the comments to understand what it does.

> **Note:** you can modify the code if you think you can. But the parts in the **loop** method, has to run with the maximum delay of 50ms. _bottom of the page for more info_

---
### Codes
> Paste to following code snippets into your program.

#### Top
Paste this to the top of the file:
```java
#include <SoftwareSerial.h> // Importing the SoftwareSerial library

#define RxD A0 // === IMPORTANT === Change this to Receiving pin
#define TxD A1 // === IMPORTANT === Change this to Transmitting pin

#define m_RL 2 // === IMPORTANT === Change this to motor BACK  LEFT
#define m_FL 3 // === IMPORTANT === Change this to motor FRONT LEFT
#define m_RR 4 // === IMPORTANT === Change this to motor BACK  RIGHT
#define m_FR 9 // === IMPORTANT === Change this to motor FRONT RIGHT

SoftwareSerial serial = SoftwareSerial(RxD,TxD); // Setting up software serial (bluetooth service)
```
---
#### Setup
Paste this into the **setup** method:
```javascript
serial.begin(38400); // Starting software serial

// set up motors
pinMode(m_RL, OUTPUT);
pinMode(m_FL, OUTPUT);
pinMode(m_RR, OUTPUT);
pinMode(m_FR, OUTPUT);
```
> **Note:** it doesn't matter if you have an already set up motor scheme. It won't conflict.
---
#### Loop
Paste this into the **loop** method:
```javascript
// Gets the Game Mode into the gameMode variable
String gameMode = getGameMode();


String serverMessage = GetServerMessage(); // Gets the message from the server
ProcessServer(serverMessage); // Parses the message and sets the motors to the correct speed

// This is the same as the two lines above, just shorter
ProcessServer(GetServerMessage());
```
---
#### Bottom
Paste this to the bottom of the code:
```javascript
String getGameMode() {
  // game modes:
  // 0    None
  // 1    Obstacle race
  // 2    Follow the line
  // 3    Waypoints
  // 4    Sumo
  // 5    Football
  if(serial.available() > 1) {
    String serverMessage = ""; // Creates variable for serverMessage
    serverMessage.concat((char)serial.read()); // Reads the character
    if(serverMessage.length() == 1) { // Checks if message is only one character
      return serverMessage; // returns serverMessage
    }
  }
}

String GetServerMessage() {
  String serverMessage = ""; // Creates variable for serverMessage
  while (serial.available() > 0) { // Reads while the server is sending stuff
    serverMessage.concat((char)serial.read()); // Gets a character from the server
  }
  
  if(serverMessage != ""){ // Checks if there was a serverMessage received
    if (serverMessage.length() == 8){ // Checks if the serverMessage was 8 characters long
      if(serverMessage == "00000000") // Checks if it's "00000000", it came to a stop
        serial.write("stop"); // Sends "stop" signal to server
      else
        serial.write("move"); // Sends "move" signal to the server
    }
  }

  return serverMessage; // Returns the serverMessage
}

void ProcessServer(String serverMessage) {
  // Parsing logic: example: 12512500
  //   125           125           0             0
  // FrontLeft | FrontRight | ReverseLeft | ReverseRight
  
  int FL = serverMessage.substring(0, 3).toInt();
  int FR = serverMessage.substring(3, 6).toInt();
  int RL = serverMessage.substring(6, 7).toInt();
  int RR = serverMessage.substring(7).toInt();
  analogWrite(m_FL, FL);
  analogWrite(m_FR, FR);
  if(RL != 0)
    digitalWrite(m_RL, HIGH);
  else
    digitalWrite(m_RL, LOW);
  if(RR != 0)
    digitalWrite(m_RR, HIGH);
  else
    digitalWrite(m_RR, LOW);
}
```
# Important
 - For the manual control the delay must be 50 or less. (ms)
 - The **_getGameMode_** method must have a delay less than 500. (ms)
 - This is possible if you remove every other method call from the **loop** method, with a _switch/if_ statement.
 
#### Example
_The loop checks if we only need the manual control first, then ignores the rest of the code._

 ```javascript
boolean manualControl = true; // we want manual control

void loop() {
  if(manualControl) {
    ProcessServer(GetServerMessage());
    delay(25);
    return;
  }

  // These are here to demostrate your code :)
  GetSensor();          // Fake code
  AvoidObject();        // Fake code
  BlackStripeWhere();   // Fake code
  FollowWaypoint();     // Fake code
  Fly();                // Fake code
  delay(325);           // Long bad delay
}
 ```
 [Got to the top](#Arduino)