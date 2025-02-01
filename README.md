
الفكرة العامة:
الهدف من السؤال هو تصميم نظام أتمتة للبيت الزجاجي (Greenhouse) للحفاظ على الظروف المثلى لنمو النباتات. النظام يستخدم أجهزة استشعار (Sensors) لمراقبة درجة الحرارة والرطوبة ورطوبة التربة، بالإضافة إلى مشغلات (Actuators) مثل محرك سيرفو (Servo Motor) لفتح وإغلاق فتحات التهوية، ومضخة مياه (Water Pump) لري النباتات، وسخان (Heater) لتدفئة البيئة عند الحاجة.

المطلوب:
1. التحكم في درجة الحرارة:
   - إذا تجاوزت درجة الحرارة 30°C، يتم فتح فتحة التهوية باستخدام محرك سيرفو.
   - إذا انخفضت درجة الحرارة عن 20°C، يتم تشغيل السخان (الممثل بمصباح LED أحمر).
2. التحكم في الرطوبة:
   - إذا انخفضت الرطوبة عن 50%، يتم إرسال تنبيه للمستخدم عبر الشاشة التسلسلية (Serial Monitor).
3. التحكم في رطوبة التربة:
   - إذا انخفضت رطوبة التربة عن 30%، يتم تشغيل مضخة المياه (الممثلة بمصباح LED أزرق) لمدة 10 ثوانٍ.

---

2- شرح الكود بشكل أجزاء:

 1. إضافة المكتبات وتعريف المتغيرات:

```cpp
#include <DHT.h>
#include <Servo.h>

#define DHTPIN 2      
#define DHTTYPE DHT22 

#define RED_LED 10    
#define BLUE_LED 11   
#define SOIL_MOISTURE_PIN A0 
#define SERVO_PIN 9   

DHT dht(DHTPIN, DHTTYPE);
Servo ventServo;
```

- المكتبات:
  - DHT.h: مكتبة لقراءة بيانات الحساس DHT22 الذي يقيس درجة الحرارة والرطوبة.
  - Servo.h: مكتبة للتحكم بمحرك السيرفو لفتح وإغلاق فتحة التهوية.
  
- تعريف المتغيرات:
  - DHTPIN: المنفذ الذي يتصل به حساس DHT22.
  - DHTTYPE: نوع الحساس (DHT22).
  - RED_LED و `BLUE_LED`: المنافذ التي تتصل بها المصابيح الحمراء والزرقاء.
  - SOIL_MOISTURE_PIN: المنفذ الذي يتصل به حساس رطوبة التربة (Potentiometer).
  - SERVO_PIN: المنفذ الذي يتصل به محرك السيرفو.
  - dht: كائن من مكتبة DHT لقراءة البيانات.
  - ventServo: كائن من مكتبة Servo للتحكم بمحرك السيرفو.

---

الثوابت والمتغيرات العامة:

```cpp
const int TEMP_HIGH = 30;
const int TEMP_LOW = 20;
const int HUMIDITY_LOW = 50;
const int MOISTURE_LOW = 30;

unsigned long waterPumpStartTime = 0;
const unsigned long waterPumpDuration = 10000; 
```

- الثوابت:
  - `TEMP_HIGH` و `TEMP_LOW`: الحدود العليا والدنيا لدرجة الحرارة.
  - `HUMIDITY_LOW`: الحد الأدنى للرطوبة.
  - `MOISTURE_LOW`: الحد الأدنى لرطوبة التربة.
  - `waterPumpDuration`: مدة تشغيل مضخة المياه (10 ثوانٍ).

- المتغيرات:
  - `waterPumpStartTime`: وقت بدء تشغيل مضخة المياه.

---

 3. دالة الإعداد (`setup`):

```cpp
void setup() {
  Serial.begin(9600);
  dht.begin();
  ventServo.attach(SERVO_PIN);
  pinMode(RED_LED, OUTPUT);
  pinMode(BLUE_LED, OUTPUT);
  ventServo.write(0); 
}
```

- الوظيفة:
  - تهيئة الشاشة التسلسلية (`Serial.begin(9600)`) لعرض البيانات.
  - بدء تشغيل حساس DHT (`dht.begin()`).
  - توصيل محرك السيرفو بالمنفذ المحدد (`ventServo.attach(SERVO_PIN)`).
  - تعيين منافذ المصابيح الحمراء والزرقاء كمخرجات (`pinMode`).
  - إغلاق فتحة التهوية عند بدء التشغيل (`ventServo.write(0)`).



4. دالة الحلقة الرئيسية (loop):

```cpp
void loop() {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();
  int soilMoisture = analogRead(SOIL_MOISTURE_PIN);

  // Temperature Control
  if (temperature > TEMP_HIGH) {
    ventServo.write(90);
    digitalWrite(RED_LED, LOW); 
    Serial.println("درجة الحرارة مرتفعة جدًا! يجب فتح فتحة التهوية.");
  } else if (temperature < TEMP_LOW) {
    ventServo.write(0);
    digitalWrite(RED_LED, HIGH); 
    Serial.println("درجة الحرارة منخفضة جدًا! يجب تشغيل السخان.");
  } else {
    ventServo.write(0);
    digitalWrite(RED_LED, LOW); 
    Serial.println(" درجة الحرارة معتدلة، يجب  إطفاء السخان وإغلاق فتحة التهوية");
  }

  // Humidity Control
  if (humidity < HUMIDITY_LOW) {
    Serial.println("  تنبية: الرطوبة منخفضة!! يرجى التحقق ");
  }

  // Soil Moisture Control
  int moistureLevel = map(soilMoisture, 0, 1023, 0, 100);
  if (moistureLevel < MOISTURE_LOW && (millis() - waterPumpStartTime) >= waterPumpDuration) {
    digitalWrite(BLUE_LED, HIGH); 
    waterPumpStartTime = millis();
    Serial.println("رطوبة التربة منخفضة! يجب تشغيل مضخة المياه.");
  } else if ((millis() - waterPumpStartTime) < waterPumpDuration) {
    digitalWrite(BLUE_LED, HIGH); 
    Serial.println("مضخة المياه قيد التشغيل الآن");
  } else {
    digitalWrite(BLUE_LED, LOW); 
    Serial.println("مضخة المياه متوقفة.");
  }

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.print("°C, Humidity: ");
  Serial.print(humidity);
  Serial.print("%, Soil Moisture: ");
  Serial.print(moistureLevel);
  Serial.println("%");
  delay(1000);
}
```

- الوظيفة:
  - قراءة درجة الحرارة والرطوبة من حساس DHT.
  - قراءة رطوبة التربة من الحساس وتحويل القيمة إلى نسبة مئوية باستخدام `map`.
  - التحكم في درجة الحرارة:
    - إذا كانت درجة الحرارة أعلى من 30°C، يتم فتح فتحة التهوية وإطفاء السخان.
    - إذا كانت درجة الحرارة أقل من 20°C، يتم إغلاق فتحة التهوية وتشغيل السخان.
    - إذا كانت درجة الحرارة معتدلة، يتم إغلاق فتحة التهوية وإطفاء السخان.
  - التحكم في الرطوبة:
    - إذا كانت الرطوبة أقل من 50%، يتم إرسال تنبيه للمستخدم عبر الشاشة التسلسلية.
  - التحكم في رطوبة التربة:
    - إذا كانت رطوبة التربة أقل من 30% ولم تكن مضخة المياه تعمل، يتم تشغيل المضخة لمدة 10 ثوانٍ.
    - إذا كانت المضخة تعمل، يتم إبقائها قيد التشغيل حتى انتهاء المدة.
    - بعد انتهاء المدة، يتم إيقاف المضخة.
  - طباعة قراءات درجة الحرارة والرطوبة ورطوبة التربة على الشاشة التسلسلية.
  - تأخير لمدة ثانية (`delay(1000)`) قبل تكرار الحلقة.

---
