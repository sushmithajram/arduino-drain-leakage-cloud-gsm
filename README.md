/* author             : j.sushmitha
 * thanks and credits : teachers and seniors of iota batch3
DRAIN WATER LEAKAGE DETECTION 
alert after detecting water leakage in drain pipe
connect usb cabe of arduino to pc
2nd digital pin of uno to water flow sensor
 */
#include <SPI.h>
#include <Ethernet.h>
#include<SoftwareSerial.h>

/*------------> FOR SENSOR AND GSM INTIALIZE<-----------*/
SoftwareSerial mySerial(9,10);
const int buzzer     = 10;
byte statusLed       = 13;
byte sensorInterrupt = 0; 
byte sensorPin       = 2;                                                    
volatile byte pulseCount;  
float flowRate;
unsigned long oldTime;

/*-------------> FOR ETHERNET INTIALIZE<--------------*/
byte mac[] = { 0xD4, 0x28, 0xB2, 0xFF, 0xA0, 0xA1 }; 
char thingSpeakAddress[] = "api.thingspeak.com";
String writeAPIKey = "XXXMX2WYYR0EVZZZ";
const int updateThingSpeakInterval = 16 * 1000; 
long lastConnectionTime = 0; 
boolean lastConnected = false;
int failedCounter = 0;
EthernetClient client;

void setup()
{
  Serial.begin(9600);                                                                 
  pinMode(buzzer,OUTPUT);                                                             
  pinMode(statusLed, OUTPUT);                                                        
                                                   
  pinMode(sensorPin, INPUT);
  digitalWrite(sensorPin, HIGH);                                                     

  pulseCount        = 0;
  flowRate          = 0.0;
  oldTime           = 0;
  
 attachInterrupt(sensorInterrupt, pulseCounter, FALLING);  

startEthernet(); 
}

void loop()
{
   
   if((millis() - oldTime) > 1000)                                                     
  { 
    detachInterrupt(sensorInterrupt);                                                  
    flowRate = ((1000.0 / (millis() - oldTime)) * pulseCount) / 4.5;     
    oldTime = millis();                                                                
                                                 
      
    unsigned int frac;
   
    Serial.print("Flow rate: ");                                                       
    Serial.print(int(flowRate));                                                       
    Serial.print("L/min");
    Serial.print("\t");      
 while(flowRate> 10){                            
    Serial.println("**************** \t\t LEAK \t\t ****************");                
    delay(1000);
    digitalWrite(statusLed, HIGH); 
    tone(buzzer,1000);
    delay(1000);
	
	
	/*----------------> CALLING <---------------*/
{Serial.println("Calling through GSM Modem");
mySerial.begin(9600);
delay(2000);
mySerial.println("ATD9444995601;");
Serial.println("Called ATD9444995601");
delay(30000);
if(mySerial.available())
Serial.write(mySerial.read());}

/*----------------->UPDATING IN THINGSPEAK<----------------*/
 {String digitalValue2 = String(digitalRead(2), DEC);
 if (client.available())
  { char c = client.read();
    Serial.print(c);}
  if (!client.connected() && lastConnected)
  { Serial.println("...disconnected");
    Serial.println();
    client.stop();}
  if(!client.connected() && (millis() - lastConnectionTime > updateThingSpeakInterval))
  {  updateThingSpeak("field1="+analogValue0);}
  if (failedCounter > 3 ) 
  {startEthernet();}
  
  lastConnected = client.connected();
}

void updateThingSpeak(String tsData)
{
  if (client.connect(thingSpeakAddress, 80))
  {         
    client.print("POST /update HTTP/1.1\n");
    client.print("Host: api.thingspeak.com\n");
    client.print("Connection: close\n");
    client.print("X-THINGSPEAKAPIKEY: "+writeAPIKey+"\n");
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    client.print("Content-Length: ");
    client.print(tsData.length());
    client.print("\n\n");

    client.print(tsData);
    
    lastConnectionTime = millis();
    
    if (client.connected())
    {
      Serial.println("Connecting to ThingSpeak...");
      Serial.println(); 
      failedCounter = 0;
    }
    else
    {
      failedCounter++;
      Serial.println("Connection to ThingSpeak failed ("+String(failedCounter, DEC)+")");   
      Serial.println();
    }
    
  }
  else
  {
    failedCounter++;
    Serial.println("Connection to ThingSpeak Failed ("+String(failedCounter, DEC)+")");   
    Serial.println();
    lastConnectionTime = millis(); 
  }
}

void startEthernet()
{
  client.stop();
  Serial.println("Connecting Arduino to network...");
  Serial.println();  
  delay(1000);
  
  // Connect to network amd obtain an IP address using DHCP
  if (Ethernet.begin(mac) == 0)
  {
    Serial.println("DHCP Failed, reset Arduino to try again");
    Serial.println();
  }
  else
  {
    Serial.println("Arduino connected to network using DHCP");
    Serial.println();
  }
  
  delay(1000);
}
   } 
    pulseCount = 0;                                                                 
   attachInterrupt(sensorInterrupt, pulseCounter, FALLING);                         
  }
}

/*---------->Interrupt Service Routine<-----------------*/
void pulseCounter()
{
  pulseCount++;                                                                      
}
