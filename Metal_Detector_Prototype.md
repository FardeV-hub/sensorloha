# โครงงานจำลองระบบตรวจจับโลหะแจ้งเตือนผ่านเสียง (Metal Detector Alarm System Prototype)

## 📌 ภาพรวมโครงงาน
โปรโตไทป์ระบบตรวจจับโลหะด้วยเซนเซอร์ Inductive Proximity (LJ12A3 / LJ18A3) ควบคุมด้วยบอร์ด Arduino UNO R3 พร้อมระบบแจ้งเตือนด้วยเสียง (Active Buzzer) เหมาะสำหรับการประยุกต์ใช้ในระบบคัดแยกวัสดุบนสายพาน หรือระบบรักษาความปลอดภัยเบื้องต้น

## 🛒 รายการอุปกรณ์และงบประมาณ
| ลำดับ | รายการอุปกรณ์ | ราคาประมาณ (บาท) |
|---|---|---|
| 1 | Arduino UNO R3 พร้อมสาย USB | 165 - 220 |
| 2 | Power Adapter 9V 2A (5.5x2.5mm) | 120 |
| 3 | MB-102 Breadboard 830 Point | 25 |
| 4 | สายไฟจัมเปอร์ (ผู้-ผู้, ผู้-เมีย, เมีย-เมีย) | 75 |
| 5 | ตัวต้านทาน 10k Ohm (แพ็ค 10 ชิ้น) | 10 |
| 6 | Active Buzzer Module | 15 |
| 7 | LJ12A3 / LJ18A3 Inductive Proximity Sensor | 90 - 150 |
| **รวม** | | **ประมาณ 500 - 615 บาท** |

## 🔌 การต่อวงจร (Wiring Guide)

**1. เซนเซอร์ (Inductive Proximity Sensor)**
*   **สายสีน้ำตาล (Brown):** ต่อไฟเลี้ยง `+5V` หรือ `+9V` (อ้างอิงตามสเปกแรงดันของเซนเซอร์)
*   **สายสีน้ำเงิน (Blue):** ต่อ `GND` ของ Arduino
*   **สายสีดำ (Black / Signal):** ต่อผ่านวงจรแบ่งแรงดัน (Voltage Divider ด้วยตัวต้านทาน 10k) แล้วเข้าที่ขา **Digital Pin 2** (D2) ของ Arduino เพื่อป้องกันไฟเกิน 5V

**2. โมดูลเสียง (Active Buzzer)**
*   **VCC:** ต่อเข้า `+5V`
*   **GND:** ต่อเข้า `GND`
*   **I/O (Signal):** ต่อเข้าขา **Digital Pin 8** (D8)

## 💻 โค้ด Arduino สำหรับใช้งานจริง
โค้ดชุดนี้มีการจัดการ Debounce เพื่อป้องกันสัญญาณรบกวน (Noise) และใช้ millis() ในการสร้างจังหวะเสียงเตือนโดยไม่บล็อกการทำงานหลัก (Non-blocking)

```cpp
// กำหนดขาเชื่อมต่อ
const int sensorPin = 2;   // ขาสัญญาณจาก Inductive Proximity Sensor
const int buzzerPin = 8;   // ขาสัญญาณไปยัง Active Buzzer

// ตัวแปรสำหรับเก็บสถานะ
int currentSensorState = LOW;
int lastSensorState = LOW;

// ตัวแปรสำหรับการทำ Debounce
unsigned long lastDebounceTime = 0;  
const unsigned long debounceDelay = 50; 

// ตัวแปรสำหรับควบคุมจังหวะเสียงเตือน
unsigned long previousMillis = 0;
const long beepInterval = 150; 
int buzzerState = LOW;

void setup() {
  pinMode(sensorPin, INPUT);
  pinMode(buzzerPin, OUTPUT);
  Serial.begin(115200); 
  Serial.println("System Ready: Metal Detector Started.");
}

void loop() {
  int reading = digitalRead(sensorPin);

  if (reading != lastSensorState) {
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {
    if (reading != currentSensorState) {
      currentSensorState = reading;
      
      if (currentSensorState == HIGH) {
         Serial.println("ALERT: Metal object detected!");
      } else {
         Serial.println("Status: Clear.");
         digitalWrite(buzzerPin, LOW); 
         buzzerState = LOW;
      }
    }
  }

  if (currentSensorState == HIGH) {
    unsigned long currentMillis = millis();
    
    if (currentMillis - previousMillis >= beepInterval) {
      previousMillis = currentMillis;
      buzzerState = !buzzerState;
      digitalWrite(buzzerPin, buzzerState);
    }
  }

  lastSensorState = reading;
}
```

## 🛠️ ขั้นตอนการประกอบและทดสอบ
1.  **ต่อวงจร:** ยึด Arduino และอุปกรณ์ลงบน Breadboard เดินสายไฟตามคำแนะนำ
2.  **อัปโหลด:** เปิด Arduino IDE เลือกบอร์ด `Arduino UNO` พอร์ตให้ถูกต้อง และทำการอัปโหลดโค้ด
3.  **ทดสอบการทำงาน:** 
    *   นำวัตถุที่เป็นโลหะ (เช่น กุญแจ, ไขควง, เหรียญ) เข้าใกล้หน้าสัมผัสของเซนเซอร์ในระยะ 4-8 มม.
    *   บอร์ดจะต้องส่งสัญญาณให้ Buzzer ดังเป็นจังหวะ และเมื่อเปิด `Serial Monitor` (ตั้งค่า 115200 baud) จะแสดงข้อความแจ้งเตือน "ALERT: Metal object detected!"
