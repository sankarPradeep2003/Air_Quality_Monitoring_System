#include "FirebaseESP8266.h"
#include <LiquidCrystal_I2C.h>  
LiquidCrystal_I2C lcd(0x3f,16,2);

#define LED D0
#define LED1 D8
#define buzz D6


#define WIFI_SSID "MADHAN"
#define WIFI_PASSWORD "MadRacer"

#define FIREBASE_HOST "https://quality-71330-default-rtdb.firebaseio.com/"
#define FIREBASE_AUTH "VYFVg6yra0rrl6ImQqmF1AG1gr8Xk3XT7yZ4GmpE"

//DHT11 
#include <DHT.h>
#define DHTPIN 0
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);


int mqsensor = A0; 
int mvalue=0;  
float t=0;
float h=0;
const char* host = "maker.ifttt.com";

FirebaseData firebaseData;
FirebaseJson json;



void setup() {
    pinMode(mqsensor,INPUT);
  pinMode(LED,OUTPUT);
  pinMode(LED1,OUTPUT);
  pinMode(buzz,OUTPUT);
  Serial.begin(115200);
   lcd.begin();      
  lcd.backlight(); 

dht.begin();

   WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED)
  {
    Serial.print(".");
    delay(100);
  }
  Serial.println();
  Serial.print("Connected with IP: ");
  Serial.println(WiFi.localIP());
  Serial.println();


  Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH);
  Firebase.reconnectWiFi(true);



  if (Firebase.setFloat(firebaseData, "AirQuality", mvalue))
  {
    Serial.println("PASSED");
    
  }
  else
  {
    Serial.println("FAILED");
    
  }

    digitalWrite(LED,LOW);
  digitalWrite(LED1,LOW);

}

void loop() {
   WiFiClient client; 
      const int httpPort = 80;  
      if (!client.connect(host, httpPort)) 
      {  
         Serial.println("connection failed");  
         return;
      } 
  mvalue=analogRead(mqsensor);
 Firebase.setFloat(firebaseData, "AirQuality", mvalue);
 
 Serial.print(mvalue);
  Serial.print("\t");
 digitalWrite(buzz,LOW);
lcd.setCursor(0,0);
 lcd.print("Air Quality :");
 lcd.print(mvalue);
 lcd.print("%");
 lcd.setCursor(0,1);
 if(mvalue <=75)
 { 
   lcd.print("Density Low");
   digitalWrite(LED1,HIGH);
   digitalWrite(LED,LOW);
   
 }
 else if(mvalue> 75&& mvalue<=150)
 { 
      lcd.print("Good");
      
   Serial.print("Good");
   Serial.print("\n");
      digitalWrite(LED1,HIGH);
      digitalWrite(LED,LOW);
      
 }
 else if(mvalue >150 && mvalue <=200)
 {
   lcd.print("Density High");
   digitalWrite(LED,HIGH);
   digitalWrite(LED1,LOW);
   Serial.print(mvalue);
  Serial.print("\t");
   Serial.print(" Quality is Poor");
   Serial.print("\n");
 }
 
 
 else 
 {
  String url = "/trigger/smp/json/with/key/oxOtyXJ1R-gagMFCNiuEfVYmaxXVGhPbO1YOYKi7n4I";
  client.print(String("GET ") + url + " HTTP/1.1\r\n" + "Host: " + host + "\r\n" + "Connection: close\r\n\r\n");
  digitalWrite(LED1,LOW);
  digitalWrite(LED,HIGH);
  digitalWrite(buzz,HIGH);
  Serial.print(mvalue);
  Serial.print("\t");
  Serial.print(" !!!---Not Fresh Air---!!!");
  Serial.print("\n");
  Serial.print("Density High");
  Serial.print("\n");
 }
 
delay(1000);

 //DHT11
 
  float h=dht.readHumidity();
  float t= dht.readTemperature();
  Firebase.setFloat(firebaseData, "Temperature", t);
  Firebase.setFloat(firebaseData, "Humudity", h);


  Serial.print("Humidity :");
  Serial.print(h);
  Serial.print("\n");
  Serial.print("Temperature :");
  Serial.print(t);
  Serial.print("\n");
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Temp :");
  lcd.print(t);
  lcd.setCursor(0,1);
  lcd.print("Humu :");
  lcd.print(h);
 delay(1000);
 
 //lcd.clear();

}