#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const char* ssid = "Pungpond";
const char* password = "pungpond";
const char* url_get = "https://ecourse.cpe.ku.ac.th/exceed03/api/play/get-session/test/hardware/";
const char* url_post = "https://ecourse.cpe.ku.ac.th/exceed03/api/play/sent-score/";


char str[100];
int id;
int done;
int pz;
int is_winner;
//int maxx = 0;
int ledgreen = 19;
int ledred = 18;
int ledyellow = 4;

const int _size = 2*JSON_OBJECT_SIZE(15);

StaticJsonDocument<_size> JSONPost;
StaticJsonDocument<_size> JSONGet;


#define SOUND_PWM_CHANNEL   0
#define SOUND_RESOLUTION    8 // 8 bit resolution
#define SOUND_ON            (1<<(SOUND_RESOLUTION-1)) // 50% duty cycle
#define SOUND_OFF           0                         // 0% duty cycle
#define NOTE_B0  31
#define NOTE_C1  33
#define NOTE_CS1 35
#define NOTE_D1  37
#define NOTE_DS1 39
#define NOTE_E1  41
#define NOTE_F1  44
#define NOTE_FS1 46
#define NOTE_G1  49
#define NOTE_GS1 52
#define NOTE_A1  55
#define NOTE_AS1 58
#define NOTE_B1  62
#define NOTE_C2  65
#define NOTE_CS2 69
#define NOTE_D2  73
#define NOTE_DS2 78
#define NOTE_E2  82
#define NOTE_F2  87
#define NOTE_FS2 93
#define NOTE_G2  98
#define NOTE_GS2 104
#define NOTE_A2  110
#define NOTE_AS2 117
#define NOTE_B2  123
#define NOTE_C3  131
#define NOTE_CS3 139
#define NOTE_D3  147
#define NOTE_DS3 156
#define NOTE_E3  165
#define NOTE_F3  175
#define NOTE_FS3 185
#define NOTE_G3  196
#define NOTE_GS3 208
#define NOTE_A3  220
#define NOTE_AS3 233
#define NOTE_B3  247
#define NOTE_C4  262
#define NOTE_CS4 277
#define NOTE_D4  294
#define NOTE_DS4 311
#define NOTE_E4  330
#define NOTE_F4  349
#define NOTE_FS4 370
#define NOTE_G4  392
#define NOTE_GS4 415
#define NOTE_A4  440
#define NOTE_AS4 466
#define NOTE_B4  494
#define NOTE_C5  523
#define NOTE_CS5 554
#define NOTE_D5  587
#define NOTE_DS5 622
#define NOTE_E5  659
#define NOTE_F5  698
#define NOTE_FS5 740
#define NOTE_G5  784
#define NOTE_GS5 831
#define NOTE_A5  880
#define NOTE_AS5 932
#define NOTE_B5  988
#define NOTE_C6  1047
#define NOTE_CS6 1109
#define NOTE_D6  1175
#define NOTE_DS6 1245
#define NOTE_E6  1319
#define NOTE_F6  1397
#define NOTE_FS6 1480
#define NOTE_G6  1568
#define NOTE_GS6 1661
#define NOTE_A6  1760
#define NOTE_AS6 1865
#define NOTE_B6  1976
#define NOTE_C7  2093
#define NOTE_CS7 2217
#define NOTE_D7  2349
#define NOTE_DS7 2489
#define NOTE_E7  2637
#define NOTE_F7  2794
#define NOTE_FS7 2960
#define NOTE_G7  3136
#define NOTE_GS7 3322
#define NOTE_A7  3520
#define NOTE_AS7 3729
#define NOTE_B7  3951
#define NOTE_C8  4186
#define NOTE_CS8 4435
#define NOTE_D8  4699
#define NOTE_DS8 4978
#define REST      0

int tempo = 140;

int buzzer = 27;


int melody[] = {

  NOTE_FS5,8, NOTE_FS5,8,NOTE_D5,8, NOTE_B4,8, REST,8, NOTE_B4,8, REST,8, NOTE_E5,8, 
  REST,8, NOTE_E5,8, REST,8, NOTE_E5,8, NOTE_GS5,8, NOTE_GS5,8, NOTE_A5,8, NOTE_B5,8,
  
};

int notes = sizeof(melody) / sizeof(melody[0]) / 2;

int wholenote = (60000 * 4) / tempo;

int divider = 0, noteDuration = 0;


void tone(int pin, int frequency, int duration)
{
  ledcSetup(SOUND_PWM_CHANNEL, frequency, SOUND_RESOLUTION);  // Set up PWM channel
  ledcAttachPin(pin, SOUND_PWM_CHANNEL);                      // Attach channel to pin
  ledcWrite(SOUND_PWM_CHANNEL, SOUND_ON);
  delay(duration);
  ledcWrite(SOUND_PWM_CHANNEL, SOUND_OFF);
}

void WiFi_Connect(){
  WiFi.disconnect();
  WiFi.begin(ssid,password);
  while(WiFi.status()!=WL_CONNECTED){
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }

  Serial.println("Connected to the WiFi network");
  Serial.print("IP Address : ");
  Serial.println(WiFi.localIP());
}

void _get() {
  if(WiFi.status() == WL_CONNECTED){
    HTTPClient http;

    http.begin(url_get);

    int httpCode = http.GET();

    if(httpCode == HTTP_CODE_OK){
      String payload = http.getString();
      DeserializationError err = deserializeJson(JSONGet, payload);
      if(err){
        Serial.print(F("deserializeJson() failed with code "));
        Serial.println(err.c_str());
      } else{
        Serial.println(httpCode);
        
        id=JSONGet["session"]["id"];
        done=JSONGet["session"]["done"];
        Serial.println("GET result");
        Serial.printf("id : %d\n",id);
        Serial.printf("done : %d\n",done);
      }
    } else{
      Serial.println(httpCode);
      Serial.println("ERROR on HTTP Request");
    }
  } else{
    WiFi_Connect();
  }
}

void _post(int m){
  if(WiFi.status() == WL_CONNECTED){
    HTTPClient http;
    
    http.begin(url_post);
    http.addHeader("Content-Type", "application/json");

    
    JSONPost["session_id"] = id;
    JSONPost["machine_code"] = "test";
    JSONPost["score"] = m;
    Serial.println(m);

    serializeJson(JSONPost, str);
    int httpCode = http.POST(str);

    if(httpCode == HTTP_CODE_OK){
      String payload = http.getString();
      DeserializationError err = deserializeJson(JSONGet, payload);
      if(err){
        Serial.print(F("deserializeJson() failed with code "));
        Serial.println(err.c_str());
      } else{
        Serial.println(httpCode);
        is_winner= JSONGet["is_winner"];
        Serial.printf("is_winner : %d\n",is_winner);
      }
      Serial.println(httpCode);
      Serial.println("POST result");
      Serial.println(payload);
    } else{
      Serial.println(httpCode);
      Serial.println("ERROR on HTTP Request");
    }
  } else{
    WiFi_Connect();
  }
  delay(100);
}


void setup() {
  // put your setup code here, to run once:
  pinMode(ledgreen, OUTPUT);
  pinMode(ledred, OUTPUT);
  pinMode(ledyellow, OUTPUT);
  digitalWrite(ledgreen, LOW);
  digitalWrite(ledred, LOW);
  digitalWrite(ledyellow, LOW);

  Serial.begin(9600);
  WiFi_Connect();
}

void loop() {
  // put your main code here, to run repeatedly:
  _get();
  while (done != 0){
    _get();
  }
  delay(5000);
 lcd.init(); // initialize the lcd
 lcd.backlight();

 lcd.setCursor(0, 0);         // move cursor to   (0, 0)
 lcd.print("     Start!!!");        // print message at (0, 0)
 lcd.setCursor(2, 1);         // move cursor to   (2, 1)
 lcd.print("    ");
 
  int val1;
  int maxx = 0;
  while (1){
  val1 = analogRead(34);
  Serial.println(val1);
  if ( val1 > maxx){
      maxx = val1;
    }
  if ((val1 <= 400 && maxx >= 400) || (maxx >= 2000 && val1 <= 1500) || (maxx >= 3000 && val1 <= 2000)){
    Serial.print("vibration is ");
    Serial.println(maxx);
    break; 
  }  
  delay(100);
 }
 
 _post(maxx);
 
 lcd.init(); // initialize the lcd
 lcd.backlight();

 lcd.setCursor(0, 0);         // move cursor to   (0, 0)
 lcd.print("   Your Score");        // print message at (0, 0)
 lcd.setCursor(2, 1);         // move cursor to   (2, 1)
 lcd.print("    ");
 lcd.print(maxx);

 if (is_winner == 1){
  for (int thisNote = 0; thisNote < notes * 2; thisNote = thisNote + 2) {

    divider = melody[thisNote + 1];
    if (divider > 0) {
      noteDuration = (wholenote) / divider;
    } else if (divider < 0) {
      noteDuration = (wholenote) / abs(divider);
      noteDuration *= 1.5; // increases the duration in half for dotted notes
    }

    tone(buzzer, melody[thisNote], noteDuration * 0.9);

    delay(noteDuration);

  }  
 }
 
 if (maxx <= 1600){
  digitalWrite(ledred, HIGH);
  delay(3000);
  digitalWrite(ledred, LOW);
 } else if (maxx <= 2800){
  digitalWrite(ledyellow, HIGH);
  delay(3000);
  digitalWrite(ledyellow, LOW);
 } else {
  digitalWrite(ledgreen, HIGH);
  delay(3000);
  digitalWrite(ledgreen, LOW);
 }

}
