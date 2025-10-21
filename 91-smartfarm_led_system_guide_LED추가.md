# 스마트팜 LED 보광 시스템 가이드
## 조도 기반 자동 LED 제어로 생산량 증대

---

## 목차
1. [LED 보광의 중요성](#1-led-보광의-중요성)
2. [LED 시스템 설계](#2-led-시스템-설계)
3. [하드웨어 구성](#3-하드웨어-구성)
4. [ESP32 코드 (LED 제어 포함)](#4-esp32-코드-led-제어-포함)
5. [자동화 로직](#5-자동화-로직)
6. [비용 분석](#6-비용-분석)
7. [설치 가이드](#7-설치-가이드)
8. [생산량 증대 효과](#8-생산량-증대-효과)

---

## 1. LED 보광의 중요성

### 1.1 왜 LED 보광이 필요한가?

```
자연광 부족 시나리오:
├─ 겨울철 일조량 감소 (하루 6시간 → 3시간)
├─ 흐린 날씨 지속
├─ 온실 피복재로 인한 광량 손실 (20-30%)
└─ 작물 밀식으로 인한 하부 광부족

결과:
❌ 생육 지연 (작기 25일 → 35일)
❌ 생산량 감소 (30-40%)
❌ 품질 저하 (연약, 도장)
❌ 병해 증가

LED 보광 효과:
✅ 생육 촉진 (작기 단축)
✅ 생산량 20-30% 증가
✅ 품질 향상 (색상, 당도)
✅ 연중 안정 생산
```

### 1.2 상추 재배 광 요구량

```
상추 최적 광 조건:
┌────────────────────────────────────────────────┐
│ 항목              │ 값                         │
├────────────────────────────────────────────────┤
│ 광포화점          │ 300-400 μmol/m²/s (PPFD)  │
│ 광보상점          │ 30-50 μmol/m²/s            │
│ 최적 일적산광량   │ 12-17 mol/m²/day           │
│ 광주기            │ 12-16시간/일               │
│ 적색:청색 비율    │ 3:1 ~ 5:1                  │
└────────────────────────────────────────────────┘

PPFD (Photosynthetic Photon Flux Density):
- 광합성에 유효한 광량 지표
- 단위: μmol/m²/s
- 측정 파장: 400-700nm

Lux와 PPFD 관계 (대략):
- 태양광: 50-75 lux = 1 μmol/m²/s
- LED (적색): 65-75 lux = 1 μmol/m²/s
- LED (백색): 70-85 lux = 1 μmol/m²/s
```

### 1.3 시간대별 자연광 vs. LED 보광

```
겨울철 하루 광량 시뮬레이션:

시간   │자연광(lux)│ LED보광 │ 합계(lux)│ PPFD(μmol)│비고
───────┼───────────┼─────────┼──────────┼───────────┼──────
06:00  │   1,000   │ 20,000  │  21,000  │   300     │ LED ON
07:00  │   5,000   │ 20,000  │  25,000  │   357     │
08:00  │  15,000   │ 20,000  │  35,000  │   500     │
09:00  │  30,000   │    0    │  30,000  │   429     │ LED OFF
10:00  │  50,000   │    0    │  50,000  │   714     │
11:00  │  60,000   │    0    │  60,000  │   857     │
12:00  │  65,000   │    0    │  65,000  │   929     │
13:00  │  60,000   │    0    │  60,000  │   857     │
14:00  │  50,000   │    0    │  50,000  │   714     │
15:00  │  30,000   │    0    │  30,000  │   429     │
16:00  │  15,000   │ 20,000  │  35,000  │   500     │ LED ON
17:00  │   5,000   │ 20,000  │  25,000  │   357     │
18:00  │   1,000   │ 20,000  │  21,000  │   300     │
19:00  │       0   │ 20,000  │  20,000  │   286     │
20:00  │       0   │    0    │      0   │     0     │ LED OFF

LED 점등 시간: 8시간 (06-09시, 16-20시)
일적산광량: 약 15 mol/m²/day (최적 범위)
```

---

## 2. LED 시스템 설계

### 2.1 LED 타입 선택

```
┌────────────────────────────────────────────────────────────┐
│ LED 타입별 비교                                            │
├────────────┬──────────┬──────────┬──────────┬─────────────┤
│ 타입       │ 가격/W   │ 효율     │ 수명     │ 권장        │
├────────────┼──────────┼──────────┼──────────┼─────────────┤
│ 일반 백색  │ 100원    │ 100lm/W  │ 25,000h  │ ★★☆☆☆      │
│ LED                                                        │
├────────────┼──────────┼──────────┼──────────┼─────────────┤
│ 식물생장   │ 300원    │ 150lm/W  │ 30,000h  │ ★★★★☆      │
│ LED (백색)                                                 │
├────────────┼──────────┼──────────┼──────────┼─────────────┤
│ 풀스펙트럼 │ 500원    │ 180lm/W  │ 35,000h  │ ★★★★★      │
│ (Full Spec)                                                │
├────────────┼──────────┼──────────┼──────────┼─────────────┤
│ Red+Blue   │ 400원    │ 2.0μmol/J│ 40,000h  │ ★★★★☆      │
│ (R:B=4:1)                                                  │
├────────────┼──────────┼──────────┼──────────┼─────────────┤
│ 고효율     │ 800원    │ 2.5μmol/J│ 50,000h  │ ★★★☆☆      │
│ 식물재배용                                        (고가)   │
└────────────┴──────────┴──────────┴──────────┴─────────────┘

★ 권장: 풀스펙트럼 LED (Full Spectrum)
  이유:
  ✓ 자연광에 가까운 스펙트럼
  ✓ 작물 관찰 용이 (Red+Blue는 보라색)
  ✓ 균형잡힌 생육
  ✓ 합리적 가격
```

### 2.2 Zone당 LED 배치 설계

```
Zone 크기: 150m² (약 45평)
재배 면적: 120m² (통로 제외)
천장 높이: 2.5m
작물 높이: 0.3m (상추)
LED 설치 높이: 1.5m (작물 상부 1.2m)

필요 광량 계산:
├─ 목표 PPFD: 200-300 μmol/m²/s (보광)
├─ 면적: 120 m²
├─ 총 필요: 24,000-36,000 μmol/s
└─ LED 효율: 2.0 μmol/J (풀스펙트럼)

필요 전력:
= 30,000 μmol/s ÷ 2.0 μmol/J
= 15,000 W
= 15 kW (이론치)

실제 설계:
- 겹치는 조사 고려
- 벽면 반사 고려
- 실제 필요: 약 8-10 kW
```

### 2.3 LED 바 구성 (추천 ★★★★★)

```
LED 바 사양:
┌─────────────────────────────────────────┐
│ 형태: LED Bar (일자형)                  │
│ 길이: 1.2m                              │
│ 전력: 80W / bar                         │
│ PPFD: 400 μmol/m²/s @ 0.5m             │
│ 색온도: 3000K (풀스펙트럼)              │
│ 가격: 약 50,000원 / bar                 │
└─────────────────────────────────────────┘

배치 계획 (150m² Zone):
┌─────────────────────────────────────────┐
│ 길이 방향: 10m                          │
│ 너비 방향: 12m                          │
│                                         │
│ LED Bar 배치 (가로 방향):               │
│ ═══════════════  (1.2m × 8개 = 9.6m)   │
│ ═══════════════                         │
│ ═══════════════                         │
│ ═══════════════  ← 1.5m 간격           │
│ ═══════════════                         │
│ ═══════════════                         │
│ ═══════════════                         │
│ ═══════════════                         │
│                                         │
│ 세로 방향 줄 수: 8줄                    │
│ 가로 방향 Bar/줄: 8개                   │
│ 총 LED Bar: 64개                        │
│ 총 전력: 5,120W (5.1kW)                 │
│ 총 비용: 3,200,000원                    │
└─────────────────────────────────────────┘

최적화 방안 (비용 절감):
├─ 핵심 재배 구역만 설치 (80m²)
├─ LED Bar: 40개
├─ 전력: 3.2kW
└─ 비용: 2,000,000원 ✓
```

### 2.4 대안: COB LED 방식

```
COB (Chip On Board) LED:
┌─────────────────────────────────────────┐
│ 형태: 원형 모듈                         │
│ 전력: 100-200W / module                 │
│ PPFD: 500 μmol/m²/s @ 0.5m             │
│ 커버 면적: 2m² @ 1.2m 높이             │
│ 가격: 30,000-60,000원 / module          │
└─────────────────────────────────────────┘

배치 계획 (150m² Zone):
필요 모듈 수: 120m² ÷ 2m² = 60개
총 전력: 60 × 150W = 9kW
총 비용: 60 × 45,000원 = 2,700,000원

장점:
✓ 높은 광량
✓ 넓은 조사각
✓ 균일한 광분포

단점:
✗ 발열 많음 (방열판 필수)
✗ 전력 소비 큼
✗ 설치 복잡
```

### 2.5 최종 권장: LED Bar 방식

```
선택 이유:
✓ 설치 간편
✓ 균일한 광분포
✓ 발열 관리 용이
✓ 확장 용이
✓ 유지보수 편리

최종 구성 (Zone당):
├─ LED Bar 80W × 40개
├─ 전력: 3.2kW
├─ 비용: 2,000,000원
└─ 커버: 80m² (핵심 재배 구역)

500평 전체 (10 Zone):
├─ LED Bar: 400개
├─ 전력: 32kW
└─ 비용: 20,000,000원

★ 단계적 도입 권장:
  - 1단계: 5개 Zone (1,000만원)
  - 효과 검증 후 확장
```

---

## 3. 하드웨어 구성

### 3.1 LED 제어 회로

```
ESP32 → LED 제어 방식:

방법 1: 릴레이 직접 제어 (간단 ★★★★★)
┌──────────────────────────────────────┐
│                                      │
│  ESP32                               │
│  GPIO 21 ──→ SSR (고체릴레이)       │
│              └─→ AC 220V             │
│                   └─→ LED Driver     │
│                        └─→ LED Bar   │
│                                      │
│  장점: 간단, 저렴                    │
│  단점: ON/OFF만 가능                 │
└──────────────────────────────────────┘

방법 2: PWM 조광 (중급 ★★★★☆)
┌──────────────────────────────────────┐
│                                      │
│  ESP32                               │
│  GPIO 21 ──→ PWM 신호 (0-10V)       │
│              └─→ 조광 LED Driver     │
│                   └─→ LED Bar        │
│                                      │
│  장점: 밝기 조절 가능 (0-100%)       │
│  단점: 조광 드라이버 필요 (고가)     │
└──────────────────────────────────────┘

방법 3: DMX512 제어 (고급 ★★★☆☆)
└─ 대규모 시설용, 복잡, 고가

★ 권장: 방법 1 (릴레이)
  - 초기: ON/OFF 충분
  - 추후 PWM 업그레이드 가능
```

### 3.2 부품 리스트 (LED 제어 추가)

```
Zone당 추가 부품:

1. LED Bar (80W)
   - 수량: 40개
   - 가격: 50,000원 × 40 = 2,000,000원
   - 사양: 1.2m, 3000K, IP65

2. LED Driver (전원)
   - 수량: 40개 (Bar당 1개)
   - 가격: 15,000원 × 40 = 600,000원
   - 사양: 80W, AC 220V → DC 24V

3. 고체 릴레이 (SSR)
   - 수량: 4개 (10개 LED Bar/그룹)
   - 가격: 15,000원 × 4 = 60,000원
   - 사양: 40A, AC 220V

4. 배선 및 커넥터
   - 전선: 2.5mm² (220V용)
   - 커넥터: 방수형
   - 가격: 약 100,000원

5. 설치 부자재
   - 행거, 체인, 볼트 등
   - 가격: 약 200,000원

6. 배전반 및 차단기
   - 3상 차단기 20A × 2
   - 가격: 150,000원

─────────────────────────────────────
Zone당 LED 시스템 총 비용: 3,110,000원

★ 10개 Zone 전체: 31,100,000원

비용 절감 방안:
1. 중고 LED 활용: 50% 절감
2. 단계적 도입: 5개 Zone만 우선
3. 저가형 LED: 30% 절감
```

### 3.3 전기 용량 계산

```
Zone당 전력 소비:

기존 시설:
├─ 환기팬: 200W × 2 = 400W
├─ 관수펌프: 300W
├─ ESP32: 5W
└─ 기타: 100W
합계: 805W

LED 추가:
└─ LED Bar: 80W × 40 = 3,200W

Zone당 총 전력: 4,005W ≈ 4kW
동시 사용률: 80%
실제 소비: 3.2kW

500평 전체 (10 Zone):
├─ 총 설비: 40kW
├─ 동시 사용: 32kW
└─ 필요 전기 용량: 50kW (여유 포함)

기존 농업용 전기:
├─ 일반: 10-30kW
├─ LED 추가 시: 50-60kW 필요
└─ → 증설 필요 (한전 신청)

전기 증설 비용:
├─ 전력 추가: 30kW
├─ 공사비: 약 3,000,000원
└─ 분담금: 약 2,000,000원
합계: 약 5,000,000원

★ 총 LED 시스템 구축 비용:
  = LED 장비(31.1M) + 전기증설(5M)
  = 36,100,000원
```

---

## 4. ESP32 코드 (LED 제어 포함)

### 4.1 개선된 전체 코드

```cpp
// smartfarm_node_with_led.ino
// LED 보광 제어 기능 추가

#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>
#include <ArduinoJson.h>

// ============================================
// 설정
// ============================================
const char* WIFI_SSID = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
const char* MQTT_SERVER = "192.168.1.100";
const int MQTT_PORT = 1883;
const char* NODE_ID = "zone1";

// ============================================
// 핀 설정
// ============================================
#define DHT_PIN 4
#define SOIL1_PIN 34
#define SOIL2_PIN 35
#define LDR_PIN 32          // 조도 센서
#define RELAY1_PIN 16       // 관수
#define RELAY2_PIN 17       // 환기팬
#define RELAY3_PIN 18       // 예비
#define LED_RELAY_PIN 21    // LED 조명 릴레이 ★ 추가

// ============================================
// LED 제어 설정
// ============================================
// 조도 임계값 (lux)
const int LED_ON_THRESHOLD = 15000;   // 15,000 lux 이하면 LED ON
const int LED_OFF_THRESHOLD = 25000;  // 25,000 lux 이상이면 LED OFF

// LED 운영 시간대 (24시간 기준)
const int LED_START_HOUR = 6;    // 06:00
const int LED_END_HOUR = 20;     // 20:00

// LED 상태
bool ledState = false;
bool ledAutoMode = true;  // true: 자동, false: 수동

// ============================================
// 시간 동기화
// ============================================
#include <time.h>
const char* ntpServer = "pool.ntp.org";
const long gmtOffset_sec = 9 * 3600;  // GMT+9 (한국)
const int daylightOffset_sec = 0;

// ============================================
// 객체 생성
// ============================================
DHT dht(DHT_PIN, DHT22);
WiFiClient espClient;
PubSubClient mqtt(espClient);

// ============================================
// 타이머
// ============================================
unsigned long lastSensorRead = 0;
unsigned long lastStatusSend = 0;
unsigned long lastLedCheck = 0;

const long SENSOR_INTERVAL = 30000;    // 30초
const long STATUS_INTERVAL = 60000;    // 1분
const long LED_CHECK_INTERVAL = 10000; // 10초

// ============================================
// 함수 선언
// ============================================
void setupWiFi();
void setupTime();
void reconnectMQTT();
void mqttCallback(char* topic, byte* payload, unsigned int length);
void readSensors();
void publishStatus(const char* status);
void controlLED();
int getCurrentHour();
float readLux();

// ============================================
// Setup
// ============================================
void setup() {
  Serial.begin(115200);
  Serial.println("\n=================================");
  Serial.println("SmartFarm Node with LED Control");
  Serial.println("=================================");
  Serial.print("Node ID: ");
  Serial.println(NODE_ID);
  
  // 핀 모드
  pinMode(RELAY1_PIN, OUTPUT);
  pinMode(RELAY2_PIN, OUTPUT);
  pinMode(RELAY3_PIN, OUTPUT);
  pinMode(LED_RELAY_PIN, OUTPUT);
  
  // 초기 상태
  digitalWrite(RELAY1_PIN, LOW);
  digitalWrite(RELAY2_PIN, LOW);
  digitalWrite(RELAY3_PIN, LOW);
  digitalWrite(LED_RELAY_PIN, LOW);
  
  dht.begin();
  setupWiFi();
  setupTime();
  
  mqtt.setServer(MQTT_SERVER, MQTT_PORT);
  mqtt.setCallback(mqttCallback);
  mqtt.setBufferSize(512);
  
  Serial.println("Setup complete!");
}

// ============================================
// WiFi 연결
// ============================================
void setupWiFi() {
  delay(10);
  Serial.print("\nConnecting to WiFi: ");
  Serial.println(WIFI_SSID);
  
  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  
  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 20) {
    delay(500);
    Serial.print(".");
    attempts++;
  }
  
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWiFi connected!");
    Serial.print("IP: ");
    Serial.println(WiFi.localIP());
  } else {
    Serial.println("\nWiFi failed!");
  }
}

// ============================================
// 시간 동기화
// ============================================
void setupTime() {
  Serial.print("Setting up time... ");
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  
  struct tm timeinfo;
  int attempts = 0;
  while (!getLocalTime(&timeinfo) && attempts < 10) {
    delay(500);
    Serial.print(".");
    attempts++;
  }
  
  if (getLocalTime(&timeinfo)) {
    Serial.println("\nTime synchronized!");
    Serial.println(&timeinfo, "%Y-%m-%d %H:%M:%S");
  } else {
    Serial.println("\nTime sync failed!");
  }
}

// ============================================
// 현재 시각 가져오기
// ============================================
int getCurrentHour() {
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) {
    return -1;
  }
  return timeinfo.tm_hour;
}

// ============================================
// 조도 읽기 (Lux 변환)
// ============================================
float readLux() {
  int ldrRaw = analogRead(LDR_PIN);
  
  // LDR 저항값 → Lux 변환 (근사식)
  // 실제 센서에 따라 보정 필요
  // 이 식은 일반적인 LDR 기준
  
  float voltage = ldrRaw * (3.3 / 4095.0);
  float resistance = (3.3 - voltage) / voltage * 10000; // 10kΩ 풀다운 가정
  float lux = pow(10, (log10(resistance / 10000) / -0.7)) * 500;
  
  // 현실적인 범위로 제한
  if (lux < 0) lux = 0;
  if (lux > 100000) lux = 100000;
  
  return lux;
}

// ============================================
// LED 자동 제어 로직
// ============================================
void controlLED() {
  if (!ledAutoMode) {
    return; // 수동 모드면 자동 제어 안함
  }
  
  // 1. 시간대 체크
  int currentHour = getCurrentHour();
  if (currentHour < 0) {
    Serial.println("Time not available");
    return;
  }
  
  bool withinOperatingHours = (currentHour >= LED_START_HOUR && 
                               currentHour < LED_END_HOUR);
  
  if (!withinOperatingHours) {
    // 운영 시간 외 → LED OFF
    if (ledState) {
      digitalWrite(LED_RELAY_PIN, LOW);
      ledState = false;
      Serial.println("LED OFF (outside operating hours)");
      publishLedStatus("auto_off_time");
    }
    return;
  }
  
  // 2. 조도 체크
  float lux = readLux();
  
  Serial.print("Current lux: ");
  Serial.println(lux);
  
  // 히스테리시스 적용 (떨림 방지)
  if (!ledState && lux < LED_ON_THRESHOLD) {
    // LED OFF 상태에서 어두우면 → ON
    digitalWrite(LED_RELAY_PIN, HIGH);
    ledState = true;
    Serial.print("LED ON (lux: ");
    Serial.print(lux);
    Serial.println(")");
    publishLedStatus("auto_on");
    
  } else if (ledState && lux > LED_OFF_THRESHOLD) {
    // LED ON 상태에서 밝으면 → OFF
    digitalWrite(LED_RELAY_PIN, LOW);
    ledState = false;
    Serial.print("LED OFF (lux: ");
    Serial.print(lux);
    Serial.println(")");
    publishLedStatus("auto_off");
  }
}

// ============================================
// LED 상태 발행
// ============================================
void publishLedStatus(const char* reason) {
  StaticJsonDocument<200> doc;
  doc["node"] = NODE_ID;
  doc["led_state"] = ledState;
  doc["led_mode"] = ledAutoMode ? "auto" : "manual";
  doc["reason"] = reason;
  doc["lux"] = readLux();
  doc["hour"] = getCurrentHour();
  
  char buffer[256];
  serializeJson(doc, buffer);
  
  String topic = "smartfarm/" + String(NODE_ID) + "/led/status";
  mqtt.publish(topic.c_str(), buffer);
}

// ============================================
// MQTT 콜백
// ============================================
void mqttCallback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message: [");
  Serial.print(topic);
  Serial.print("] ");
  
  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println(message);
  
  StaticJsonDocument<200> doc;
  DeserializationError error = deserializeJson(doc, message);
  
  if (error) {
    Serial.print("JSON error: ");
    Serial.println(error.c_str());
    return;
  }
  
  String topicStr = String(topic);
  
  // 기존 릴레이 제어
  if (topicStr.endsWith("/relay1")) {
    bool state = doc["state"];
    digitalWrite(RELAY1_PIN, state ? HIGH : LOW);
    Serial.print("Relay 1: ");
    Serial.println(state ? "ON" : "OFF");
  }
  else if (topicStr.endsWith("/relay2")) {
    bool state = doc["state"];
    digitalWrite(RELAY2_PIN, state ? HIGH : LOW);
    Serial.print("Relay 2: ");
    Serial.println(state ? "ON" : "OFF");
  }
  // ★ LED 제어 추가
  else if (topicStr.endsWith("/led")) {
    if (doc.containsKey("mode")) {
      String mode = doc["mode"].as<String>();
      if (mode == "auto") {
        ledAutoMode = true;
        Serial.println("LED: Auto mode");
      } else if (mode == "manual") {
        ledAutoMode = false;
        Serial.println("LED: Manual mode");
      }
    }
    
    if (doc.containsKey("state") && !ledAutoMode) {
      bool state = doc["state"];
      digitalWrite(LED_RELAY_PIN, state ? HIGH : LOW);
      ledState = state;
      Serial.print("LED manual: ");
      Serial.println(state ? "ON" : "OFF");
      publishLedStatus("manual");
    }
  }
}

// ============================================
// MQTT 재연결
// ============================================
void reconnectMQTT() {
  while (!mqtt.connected()) {
    Serial.print("MQTT connecting... ");
    
    String clientId = "ESP32_" + String(NODE_ID);
    
    if (mqtt.connect(clientId.c_str())) {
      Serial.println("connected");
      
      String controlTopic = "smartfarm/" + String(NODE_ID) + "/control/#";
      mqtt.subscribe(controlTopic.c_str());
      Serial.print("Subscribed: ");
      Serial.println(controlTopic);
      
      publishStatus("online");
      publishLedStatus("init");
      
    } else {
      Serial.print("failed, rc=");
      Serial.println(mqtt.state());
      delay(5000);
    }
  }
}

// ============================================
// 센서 읽기
// ============================================
void readSensors() {
  float temp = dht.readTemperature();
  float humidity = dht.readHumidity();
  
  int soil1Raw = analogRead(SOIL1_PIN);
  int soil2Raw = analogRead(SOIL2_PIN);
  float soil1 = map(soil1Raw, 0, 4095, 0, 100);
  float soil2 = map(soil2Raw, 0, 4095, 0, 100);
  
  float lux = readLux();
  
  if (isnan(temp) || isnan(humidity)) {
    Serial.println("DHT read failed!");
    temp = -999;
    humidity = -999;
  }
  
  StaticJsonDocument<400> doc;
  doc["node"] = NODE_ID;
  doc["timestamp"] = millis();
  
  JsonObject sensors = doc.createNestedObject("sensors");
  sensors["temperature"] = temp;
  sensors["humidity"] = humidity;
  sensors["soil1"] = soil1;
  sensors["soil2"] = soil2;
  sensors["lux"] = lux;  // ★ 조도 추가
  
  JsonObject status = doc.createNestedObject("status");
  status["led_state"] = ledState;
  status["led_mode"] = ledAutoMode ? "auto" : "manual";
  
  char buffer[512];
  serializeJson(doc, buffer);
  
  String topic = "smartfarm/" + String(NODE_ID) + "/sensors";
  mqtt.publish(topic.c_str(), buffer);
  
  Serial.println("Sensor data:");
  Serial.println(buffer);
}

// ============================================
// 상태 발행
// ============================================
void publishStatus(const char* status) {
  StaticJsonDocument<300> doc;
  doc["node"] = NODE_ID;
  doc["status"] = status;
  doc["uptime"] = millis() / 1000;
  doc["rssi"] = WiFi.RSSI();
  doc["ip"] = WiFi.localIP().toString();
  doc["led_state"] = ledState;
  doc["led_mode"] = ledAutoMode ? "auto" : "manual";
  
  char buffer[400];
  serializeJson(doc, buffer);
  
  String topic = "smartfarm/" + String(NODE_ID) + "/status";
  mqtt.publish(topic.c_str(), buffer);
}

// ============================================
// Loop
// ============================================
void loop() {
  // WiFi 체크
  if (WiFi.status() != WL_CONNECTED) {
    setupWiFi();
  }
  
  // MQTT 체크
  if (!mqtt.connected()) {
    reconnectMQTT();
  }
  mqtt.loop();
  
  unsigned long currentMillis = millis();
  
  // 센서 읽기
  if (currentMillis - lastSensorRead >= SENSOR_INTERVAL) {
    lastSensorRead = currentMillis;
    readSensors();
  }
  
  // 상태 전송
  if (currentMillis - lastStatusSend >= STATUS_INTERVAL) {
    lastStatusSend = currentMillis;
    publishStatus("alive");
  }
  
  // ★ LED 자동 제어
  if (currentMillis - lastLedCheck >= LED_CHECK_INTERVAL) {
    lastLedCheck = currentMillis;
    controlLED();
  }
  
  delay(10);
}
```

### 4.2 MQTT Topic 구조 (LED 추가)

```
smartfarm/
├── zone1/
│   ├── sensors              # 센서 데이터 (lux 포함)
│   ├── control/
│   │   ├── relay1          # 관수
│   │   ├── relay2          # 환기
│   │   └── led             # ★ LED 제어
│   ├── led/
│   │   └── status          # ★ LED 상태
│   └── status              # 노드 상태

LED 제어 메시지:
{
  "mode": "auto",      // "auto" or "manual"
  "state": true        // true=ON, false=OFF (manual mode only)
}

예시:
1. 자동 모드로 전환:
   Topic: smartfarm/zone1/control/led
   Payload: {"mode": "auto"}

2. 수동 ON:
   Topic: smartfarm/zone1/control/led
   Payload: {"mode": "manual", "state": true}

3. 수동 OFF:
   Topic: smartfarm/zone1/control/led
   Payload: {"mode": "manual", "state": false}
```

---

## 5. 자동화 로직

### 5.1 Node-RED Flow (LED 제어)

```javascript
// Function 노드: LED 자동 제어 로직

// 입력: msg.payload.sensors.lux
const lux = msg.payload.sensors.lux;
const node = msg.payload.node;
const currentHour = new Date().getHours();

// 운영 시간: 06:00 - 20:00
const START_HOUR = 6;
const END_HOUR = 20;

// 조도 임계값
const LED_ON_LUX = 15000;
const LED_OFF_LUX = 25000;

// 현재 시간 체크
if (currentHour < START_HOUR || currentHour >= END_HOUR) {
    // 운영 시간 외 → OFF
    msg.payload = {
        mode: "auto"
    };
    msg.topic = `smartfarm/${node}/control/led`;
    flow.set(`${node}_led_state`, false);
    node.status({fill:"grey", shape:"dot", text:"시간외"});
    return msg;
}

// 현재 LED 상태 가져오기
let ledState = flow.get(`${node}_led_state`) || false;

// 조도에 따른 제어
if (!ledState && lux < LED_ON_LUX) {
    // 어두우면 ON
    msg.payload = {
        mode: "auto",
        state: true
    };
    flow.set(`${node}_led_state`, true);
    node.status({fill:"green", shape:"dot", text:`ON (${lux} lux)`});
    
} else if (ledState && lux > LED_OFF_LUX) {
    // 밝으면 OFF
    msg.payload = {
        mode: "auto",
        state: false
    };
    flow.set(`${node}_led_state`, false);
    node.status({fill:"red", shape:"dot", text:`OFF (${lux} lux)`});
    
} else {
    // 유지
    return null;
}

msg.topic = `smartfarm/${node}/control/led`;
return msg;
```

### 5.2 일출/일몰 연동 (고급)

```javascript
// Function 노드: 일출/일몰 시간 계산

// SunCalc 라이브러리 사용
// npm install suncalc

const SunCalc = require('suncalc');

// 위치 정보 (예: 서울)
const lat = 37.5665;
const lon = 126.9780;

const now = new Date();
const times = SunCalc.getTimes(now, lat, lon);

const sunrise = times.sunrise.getHours();
const sunset = times.sunset.getHours();

// LED 운영: 일출 1시간 전 ~ 일몰 2시간 후
const LED_START = sunrise - 1;
const LED_END = sunset + 2;

flow.set('led_start_hour', LED_START);
flow.set('led_end_hour', LED_END);

msg.payload = {
    sunrise: sunrise,
    sunset: sunset,
    led_start: LED_START,
    led_end: LED_END
};

node.status({fill:"blue", shape:"dot", 
             text:`${LED_START}:00 ~ ${LED_END}:00`});

return msg;
```

### 5.3 생육 단계별 제어

```javascript
// Function 노드: 생육 단계별 LED 제어

// 입력: 정식 후 일수
const daysAfterPlanting = msg.payload.days_after_planting;

let targetDLI;  // Daily Light Integral (mol/m²/day)
let ledHours;

if (daysAfterPlanting < 7) {
    // 초기 육묘
    targetDLI = 10;
    ledHours = 12;
    
} else if (daysAfterPlanting < 14) {
    // 생육 초기
    targetDLI = 14;
    ledHours = 14;
    
} else {
    // 생육 후기
    targetDLI = 17;
    ledHours = 16;
}

msg.payload = {
    target_dli: targetDLI,
    led_hours: ledHours,
    stage: daysAfterPlanting < 7 ? "초기" :
           daysAfterPlanting < 14 ? "중기" : "후기"
};

flow.set('target_dli', targetDLI);
flow.set('led_hours', ledHours);

return msg;
```

---

## 6. 비용 분석

### 6.1 LED 시스템 총 비용

```
┌─────────────────────────┬──────────────┬───────────┐
│ 항목                    │ Zone당       │ 10 Zone   │
├─────────────────────────┼──────────────┼───────────┤
│ LED Bar (80W × 40개)    │ 2,000,000원  │20,000,000 │
│ LED Driver (40개)       │   600,000원  │ 6,000,000 │
│ 고체 릴레이 (4개)       │    60,000원  │   600,000 │
│ 배선 및 커넥터          │   100,000원  │ 1,000,000 │
│ 설치 부자재             │   200,000원  │ 2,000,000 │
│ 배전반                  │   150,000원  │ 1,500,000 │
├─────────────────────────┼──────────────┼───────────┤
│ **LED 장비 소계**       │**3,110,000원**│**31,100,000**│
├─────────────────────────┼──────────────┼───────────┤
│ 전기 증설 (한전)        │       -      │ 5,000,000 │
│ 설치 공사비             │       -      │ 2,000,000 │
├─────────────────────────┼──────────────┼───────────┤
│ **총 투자 비용**        │       -      │**38,100,000**│
│                         │              │**(3,810만원)**│
└─────────────────────────┴──────────────┴───────────┘

단계적 도입:
├─ 1단계 (5 Zone): 19,050,000원
├─ 2단계 (5 Zone 추가): 15,550,000원
└─ 합계: 34,600,000원
```

### 6.2 운영 비용 (LED 추가 후)

```
전기료 계산:

LED 사용:
├─ 전력: 32kW (10 Zone)
├─ 일 사용: 8시간 (평균)
├─ 일 전력량: 32kW × 8h = 256kWh
├─ 월 전력량: 256 × 30 = 7,680kWh
└─ 월 전기료: 7,680 × 120원 = 921,600원

기존 시설:
└─ 월 전기료: 151,000원 (기존)

총 전기료:
├─ 월: 1,072,600원
└─ 연: 12,871,200원

┌─────────────────────┬─────────────┬──────────────┐
│ 항목                │ LED 미사용  │ LED 사용     │
├─────────────────────┼─────────────┼──────────────┤
│ 전기료 (연)         │   151,000원 │ 12,871,200원 │
│ 부품 교체 (연)      │   100,000원 │    300,000원 │
│ LED 유지보수        │         0원 │    500,000원 │
├─────────────────────┼─────────────┼──────────────┤
│ **연간 운영비**     │   251,000원 │ 13,671,200원 │
├─────────────────────┼─────────────┼──────────────┤
│ **증가분**          │       -     │ 13,420,200원 │
└─────────────────────┴─────────────┴──────────────┘

⚠️ 전기료 증가: 연 1,342만원
```

### 6.3 비용 대비 효과 분석

```
생산량 증대 효과:

기존 (LED 없음):
├─ 연 생산량: 140톤
├─ 작기: 25일
└─ 연 작기 수: 14회

LED 보광 후:
├─ 작기 단축: 25일 → 22일 (12% 단축)
├─ 연 작기 수: 16회 (2회 증가)
├─ 생산량 증가: 20% (품질 개선)
└─ 연 생산량: 140톤 × 1.2 = 168톤

증가 생산량: 28톤

추가 매출:
├─ 28톤 × 평균 단가 5,080원/kg
├─ = 28,000kg × 5,080원
└─ = 142,240,000원

추가 비용:
├─ 전기료 증가: 13,420,200원
├─ 종자/비료 증가: 7,000,000원
├─ 포장/배송 증가: 5,000,000원
└─ 합계: 25,420,200원

순 이익 증가:
= 142,240,000 - 25,420,200
= 116,819,800원/년

투자 회수 기간:
= 38,100,000 / 116,819,800
= 0.33년 = **약 4개월**

┌─────────────────────────────────────────┐
│ LED 투자 수익성 분석                    │
├─────────────────────────────────────────┤
│ 초기 투자: 3,810만원                    │
│ 연 순이익 증가: 1억 1,682만원           │
│ 투자 회수: 4개월 ★★★★★                 │
│ ROI (1년): 307% ★★★★★                   │
│ 5년 누적 이익: 5억 8,410만원            │
└─────────────────────────────────────────┘

💰 결론: 매우 우수한 투자!
```

---

## 7. 설치 가이드

### 7.1 LED Bar 설치 순서

```
Day 1: 준비 작업
□ 전기 증설 (한전 사전 신청 필요)
□ 배전반 설치
□ 3상 배선 인입
□ 차단기 설치

Day 2-3: 구조물 설치
□ LED Bar 행거 설치 (천장)
  - 간격: 1.5m
  - 높이: 2.5m (바닥 기준)
□ 체인 설치 (높이 조절용)
□ 안전 확인

Day 4-5: LED 설치
□ LED Bar 장착 (40개/Zone)
□ LED Driver 연결
□ 배선 정리
□ 방수 처리

Day 6: 전기 연결
□ LED 그룹별 회로 구성
  - 10개 Bar = 1그룹
  - 4개 그룹/Zone
□ SSR (고체 릴레이) 연결
□ 접지 확인
□ 절연 테스트

Day 7: 제어 시스템
□ ESP32 LED 릴레이 연결
□ 코드 업로드
□ MQTT 통신 테스트
□ 조도 센서 캘리브레이션

Day 8: 시운전
□ 수동 ON/OFF 테스트
□ 자동 제어 테스트
□ 타이머 동작 확인
□ 전력 측정

Day 9-10: 최적화
□ LED 높이 조정
□ 조도 분포 측정
□ 임계값 튜닝
□ 문서화

총 소요: 10일 (1 Zone 기준)
```

### 7.2 안전 수칙

```
⚠️ 전기 안전:
□ 차단기 OFF 후 작업
□ 절연 장갑 착용
□ 접지 필수
□ 누전 차단기 설치
□ 정기 점검

⚠️ 고소 작업:
□ 안전대 착용
□ 2인 1조 작업
□ 사다리 고정
□ 공구 낙하 방지

⚠️ LED 취급:
□ 정전기 방지
□ 충격 주의
□ 방수 처리 확인
□ 과전류 방지
```

### 7.3 배선도

```
전기 계통도:

3상 220V (한전)
    │
    ├─ 주 차단기 (50A)
    │   │
    │   ├─ Zone 1 차단기 (20A)
    │   │   │
    │   │   ├─ LED 그룹 1 (10개 Bar)
    │   │   │   └─ SSR 1 ← ESP32 GPIO 21
    │   │   │
    │   │   ├─ LED 그룹 2 (10개 Bar)
    │   │   │   └─ SSR 2 ← ESP32 GPIO 22
    │   │   │
    │   │   ├─ LED 그룹 3 (10개 Bar)
    │   │   │   └─ SSR 3 ← ESP32 GPIO 23
    │   │   │
    │   │   └─ LED 그룹 4 (10개 Bar)
    │   │       └─ SSR 4 ← ESP32 GPIO 25
    │   │
    │   ├─ Zone 2 차단기 (20A)
    │   │   └─ (동일 구조)
    │   │
    │   └─ ... (Zone 3-10)
    │
    └─ 예비 차단기

ESP32 핀 할당:
├─ GPIO 21: LED 그룹 1
├─ GPIO 22: LED 그룹 2
├─ GPIO 23: LED 그룹 3
└─ GPIO 25: LED 그룹 4
```

---

## 8. 생산량 증대 효과

### 8.1 작기별 비교

```
┌──────────────────┬─────────┬─────────┬────────┐
│ 구분             │ LED 없음│ LED 있음│ 개선율 │
├──────────────────┼─────────┼─────────┼────────┤
│ 정식 후 수확일   │  10일   │   8일   │  20%↑  │
│ 육묘 기간        │  15일   │  14일   │   7%↑  │
│ 총 작기          │  25일   │  22일   │  12%↑  │
├──────────────────┼─────────┼─────────┼────────┤
│ 연간 작기 수     │  14회   │  16회   │ +2회   │
│ 평균 수확 중량   │ 250g    │ 280g    │  12%↑  │
│ 폐기율           │   2%    │  1.5%   │ 0.5%↓  │
├──────────────────┼─────────┼─────────┼────────┤
│ Zone당 생산량    │  14톤   │ 16.8톤  │  20%↑  │
│ (150m²)          │         │         │        │
├──────────────────┼─────────┼─────────┼────────┤
│ 전체 생산량      │ 140톤   │ 168톤   │  20%↑  │
│ (10 Zone)        │         │         │        │
└──────────────────┴─────────┴─────────┴────────┘

증가량: 28톤/년
```

### 8.2 품질 개선 효과

```
품질 지표:

┌─────────────────┬─────────┬─────────┐
│ 항목            │ LED 없음│ LED 있음│
├─────────────────┼─────────┼─────────┤
│ 엽장 (cm)       │  15-18  │  18-22  │
│ 엽폭 (cm)       │  12-15  │  15-18  │
│ 색상            │  보통   │  진한색 │
│ 조직감          │  연약   │  치밀   │
│ 병해 발생률     │   5%    │   2%    │
│ 상품화율        │  90%    │  95%    │
└─────────────────┴─────────┴─────────┘

가격 프리미엄:
├─ 일반 상추: 3,500원/kg
├─ 고품질 상추: 4,500원/kg
└─ 차이: +1,000원/kg (29%)

추가 매출:
= 168톤 × 0.5 (고품질 비율) × 1,000원
= 84,000,000원
```

### 8.3 계절별 효과

```
월별 LED 기여도:

월   │ 자연광 │LED 보광│ 비중 │ 효과
─────┼────────┼────────┼──────┼─────
 1월 │  부족  │  필수  │ 70%  │ 높음
 2월 │  부족  │  필수  │ 70%  │ 높음
 3월 │  보통  │  보조  │ 50%  │ 중간
 4월 │  충분  │  최소  │ 30%  │ 낮음
 5월 │  충분  │  없음  │  0%  │ 없음
 6월 │  과다  │  없음  │  0%  │ 없음
 7월 │  충분  │  보조  │ 20%  │ 낮음
 8월 │  충분  │  보조  │ 20%  │ 낮음
 9월 │  보통  │  보조  │ 40%  │ 중간
10월 │  보통  │  필수  │ 50%  │ 중간
11월 │  부족  │  필수  │ 70%  │ 높음
12월 │  부족  │  필수  │ 70%  │ 높음

연평균 LED 기여: 약 40%
```

### 8.4 최종 수익 비교

```
┌──────────────────────┬─────────────┬─────────────┐
│ 항목                 │ LED 없음    │ LED 있음    │
├──────────────────────┼─────────────┼─────────────┤
│ 연 생산량            │ 140톤       │ 168톤       │
│ 평균 판매가          │ 5,080원/kg  │ 5,580원/kg  │
│                      │             │ (품질↑)     │
├──────────────────────┼─────────────┼─────────────┤
│ **매출**             │ 711,200,000 │ 937,440,000 │
├──────────────────────┼─────────────┼─────────────┤
│ 변동비               │ 283,995,955 │ 320,000,000 │
│ 고정비               │ 261,844,203 │ 275,264,403 │
│ (전기료 포함)        │             │             │
├──────────────────────┼─────────────┼─────────────┤
│ **영업이익**         │ 165,359,842 │ 342,175,597 │
│ **당기순이익**       │ 132,287,874 │ 273,740,478 │
├──────────────────────┼─────────────┼─────────────┤
│ **순이익 증가**      │      -      │ 141,452,604 │
│                      │             │ (+107%)     │
└──────────────────────┴─────────────┴─────────────┘

💰 LED 투자 효과:
  - 순이익 증가: 연 1억 4,145만원
  - LED 투자액: 3,810만원
  - 회수 기간: 3.2개월
  - ROI: 371% (연간)
```

---

## 9. 결론

### 9.1 LED 보광 투자 가치

```
✅ 투자 타당성: ★★★★★ (매우 높음)

핵심 지표:
├─ 초기 투자: 3,810만원
├─ 회수 기간: 3.2개월
├─ 연간 ROI: 371%
├─ 5년 수익: 7억원
└─ 생산량 증가: 20%

권장 사항:
✓ 반드시 도입 필요
✓ 단계적 접근 가능
✓ 5개 Zone부터 시작
✓ 효과 검증 후 확장
```

### 9.2 단계별 실행 계획

```
Phase 1: 테스트 (1개 Zone)
├─ 투자: 311만원
├─ 기간: 2주
├─ 목표: 효과 검증
└─ 판단 기준: 작기 단축, 품질

Phase 2: 부분 도입 (5개 Zone)
├─ 투자: 1,906만원 (전기 포함)
├─ 기간: 2개월
├─ 목표: 수익성 확인
└─ 예상 수익: 월 590만원

Phase 3: 전면 도입 (10개 Zone)
├─ 투자: 1,546만원 (추가)
├─ 기간: 1개월
├─ 목표: 최대 생산
└─ 예상 수익: 월 1,180만원

총 기간: 3.5개월
총 투자: 3,810만원
```

### 9.3 성공 요인

```
기술적 요인:
✓ ESP32 자동 제어
✓ 조도 센서 정밀 측정
✓ MQTT 실시간 통신
✓ Node-RED 자동화

경제적 요인:
✓ 빠른 투자 회수
✓ 안정적 수익
✓ 낮은 운영 비용
✓ 높은 ROI

운영적 요인:
✓ 쉬운 관리
✓ 원격 제어
✓ 자동화
✓ 확장 용이
```

### 9.4 최종 권장

```
LED 보광 시스템:
★★★★★ (5/5) 강력 추천

이유:
1. 탁월한 경제성 (ROI 371%)
2. 빠른 투자 회수 (3.2개월)
3. 생산량 20% 증가
4. 품질 대폭 개선
5. 연중 안정 생산

주의사항:
⚠ 전기 증설 필수
⚠ 초기 투자 3,810만원
⚠ 전기료 증가 (월 92만원)
⚠ 단계적 도입 권장

결론:
LED 보광은 스마트팜의 필수 투자!
```

---

## 부록

### A. LED 제품 추천

```
1. 삼성 호티 LED LM301H
   - 효율: 2.7 μmol/J
   - 가격: 높음
   - 품질: 최고

2. 국산 풀스펙트럼 LED Bar
   - 효율: 2.0 μmol/J
   - 가격: 중간
   - 품질: 우수 ★ 권장

3. 중국산 LED Bar
   - 효율: 1.5 μmol/J
   - 가격: 저렴
   - 품질: 보통
```

### B. 전기 증설 절차

```
1. 한전 신청 (온라인)
2. 현장 조사 (2주)
3. 공사 견적 (1주)
4. 공사 (2주)
5. 준공 검사 (1주)

총 기간: 약 6주
```

### C. 참고 자료

```
- 농촌진흥청 LED 연구 자료
- 스마트팜코리아 매뉴얼
- ESP32 공식 문서
- MQTT 가이드
```

---

**LED 보광으로 수익 2배 증가!**
**3.2개월 만에 투자 회수!**
**지금 바로 시작하세요!**

작성일: 2025년 10월 19일
