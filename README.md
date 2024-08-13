# nodemcu
node mcu for line notify


#include <TridentTD_LineNotify.h>

#define SSID        "NMP-WiFi"  // แก้ไข SSID ที่ผิดพลาด
#define PASSWORD    ""          // เพิ่มรหัสผ่านของ WiFi ที่นี่
#define LINE_TOKEN  "OmQJFgSSQFdfVJlltvMv4602cWJ7tdyMzciEDgsrRLp" // LINE Notify token
#define PirPin D6

bool beep_state = false;
uint32_t ts, ts1;
const int pirThreshold = 10; // ปรับค่านี้ตามความไวของ PIR sensor

// ตัวแปรที่เก็บจำนวนครั้งที่ส่งข้อความไปยัง LINE Notify
int messageCount = 0;

void setup() {
  Serial.begin(115200);
  Serial.println();
  Serial.println(LINE.getVersion());
  pinMode(PirPin, INPUT);
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LED_BUILTIN, HIGH);
  WiFi.begin(SSID, PASSWORD);
  Serial.printf("WiFi connecting to %s\n",  SSID);
  
  while(WiFi.status() != WL_CONNECTED) { 
    Serial.print("."); 
    delay(400); 
  }
  
  Serial.printf("\nWiFi connected\nIP: ");
  Serial.println(WiFi.localIP());  
  delay(5000);
  
  ts = ts1 = millis();
  LINE.setToken(LINE_TOKEN);
}

void loop() {
  ts = millis();

  if (WiFi.status() == WL_CONNECTED) {
    digitalWrite(LED_BUILTIN, LOW);
  } else {
    digitalWrite(LED_BUILTIN, HIGH);   
  }
  
  // รีเซ็ตสถานะ beep_state หลังจากผ่านไป 5 วินาที
  if ((ts - ts1 >= 5000) && (beep_state == true)) {
    beep_state = false;
  } 

  // ตรวจจับการเคลื่อนไหวจาก PIR sensor และส่งข้อความไปยัง LINE Notify
  if ((digitalRead(PirPin) == HIGH) && (beep_state == false) && (WiFi.status() == WL_CONNECTED)) {
    while (digitalRead(PirPin) == HIGH) delay(100);
    Serial.println("Detect !");
    LINE.notify("ส่งการบ้านแล้ว | จำนวนครั้งที่ส่ง: " + String(messageCount)); // แก้ไขข้อความให้ชัดเจนขึ้น
    
    beep_state = true;
    ts1 = ts; // อัพเดตเวลาเพื่อใช้ในการรีเซ็ต beep_state

    // เพิ่มจำนวนครั้งที่ส่งข้อความไปยัง LINE Notify
    messageCount++;
    Serial.print("Messages sent: ");
    Serial.println(messageCount);
  }
  
  delay(10);
}

