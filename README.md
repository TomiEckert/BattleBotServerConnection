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

### Basics
Paster the following codes to the corresponding parts of your code. Change the necessary settings. Read through the comments to understand what it does.

> **Note:** you can modify the code if you think you can. But the parts in the **loop** method, has to run with the maximum delay of 50ms. _bottom of the page for more info_

---
### Codes
> Paste to following code snippets into your program.

#### Top
Paste this to the top of the file:
```java
#include <SoftwareSerial.h>

#define RxD A0 // === IMPORTANT === Change this to Receiving pin
#define TxD A1 // === IMPORTANT === Change this to Transmitting pin

#define m_RL 2 // === IMPORTANT === Change this to motor BACK  LEFT
#define m_FL 3 // === IMPORTANT === Change this to motor FRONT LEFT
#define m_RR 4 // === IMPORTANT === Change this to motor BACK  RIGHT
#define m_FR 9 // === IMPORTANT === Change this to motor FRONT RIGHT

SoftwareSerial serial = SoftwareSerial(RxD,TxD); // sets up bluetooth communication
```
---
#### Setup
Paste this into the **setup** method:
```javascript
void setup() {
  // starts bluetooth communication
  serial.begin(38400);
  
  // set up motors
  pinMode(m_RL, OUTPUT);
  pinMode(m_FL, OUTPUT);
  pinMode(m_RR, OUTPUT);
  pinMode(m_FR, OUTPUT);

  // your code goes here
}
```
> **Note:** it doesn't matter if you have an already set up motor scheme. It won't conflict.
---
#### Loop
Paste this into the **loop** method:
```javascript
int gameMode = 0;
void loop() {
  int temp = GetServerMessage();
  if(temp != -1)
    gameMode = temp;
  if(gameMode == 3 || gameMode == 4 || gameMode == 5){
    delay(50);
    return;
  }

  // your code goes here
  // the variable 'gameMode' has the current game mode.

  // Available game modes:
  // 0    None
  // 1    Obstacle race
  // 2    Follow the line
  // 3    Waypoints     -       manual
  // 4    Sumo          -       manual
  // 5    Football      -       manual
}
```
---
#### Bottom
Paste this to the bottom of the code:
```javascript
int GetServerMessage() {
  if(serial.available() < 9)
    return -1;
  String serverMessage = "";
  for (int i = 0; i < 9; i++) {
    serverMessage.concat((char)serial.read());
  }
  if(serverMessage.indexOf(':') > -1)
    serialFlush();

  return ProcessServer(serverMessage);
}

int ProcessServer(String serverMessage) {
  if (serverMessage.length() != 9)
    return -1;
  int FL = serverMessage.substring(0, 3).toInt();
  int FR = serverMessage.substring(3, 6).toInt();
  int RL = serverMessage.substring(6, 7).toInt();
  int RR = serverMessage.substring(7, 8).toInt();
  int gameMode = serverMessage.substring(8).toInt();

  if(gameMode != 4 && gameMode != 5) {
    return gameMode;
  }
  
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
  return gameMode;
}

void serialFlush(){
  while(serial.available() > 0) {
    serial.read();
  }
} 
```
# Important
 - For the manual control the delay in the loop must be 50 or less. (ms)
 - Use a _switch/if_ statement to check which game mode you need, and run the appropriate methods for it.
 
 [Got to the top](#Arduino)
