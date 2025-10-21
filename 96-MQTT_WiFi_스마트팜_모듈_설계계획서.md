# MQTT 기반 WiFi 플러그앤플레이 스마트팜 제어 모듈
## 설계 계획서

**프로젝트명**: SmartFarm WiFi Control Module (SFWCM)  
**버전**: 1.0  
**작성일**: 2025년 10월 20일

---

## 📋 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [시스템 아키텍처](#2-시스템-아키텍처)
3. [하드웨어 설계](#3-하드웨어-설계)
4. [소프트웨어 설계](#4-소프트웨어-설계)
5. [MQTT 통신 프로토콜](#5-mqtt-통신-프로토콜)
6. [PCB 설계](#6-pcb-설계)
7. [모듈 종류 및 사양](#7-모듈-종류-및-사양)
8. [제작 및 비용](#8-제작-및-비용)
9. [설치 및 사용법](#9-설치-및-사용법)
10. [테스트 계획](#10-테스트-계획)

---

## 1. 프로젝트 개요

### 1.1 목적
**전원선만 연결하면 WiFi로 센서와 액추에이터를 자동으로 제어할 수 있는 플러그앤플레이 모듈 개발**

### 1.2 핵심 개념
```
전원 연결 → WiFi 자동 접속 → MQTT Broker 연결 → 센서 데이터 송신/제어 명령 수신
```

### 1.3 주요 특징
- ✅ **진정한 플러그앤플레이**: 전원만 꽂으면 자동 작동
- ✅ **WiFi 통신**: 배선 없이 무선으로 연결
- ✅ **MQTT 프로토콜**: 경량, 저전력 통신
- ✅ **모듈식 설계**: 센서/액추에이터 모듈 교체 가능
- ✅ **자동 복구**: WiFi/MQTT 연결 끊김 시 자동 재연결
- ✅ **OTA 업데이트**: 무선으로 펌웨어 업데이트
- ✅ **저비용**: 모듈당 5,000원~15,000원

### 1.4 적용 분야
- 스마트팜 환경 모니터링
- 자동 관개 시스템
- 온실 환경 제어
- 양액 관리 시스템
- 조명 제어
- 환풍기/차광막 제어

---

## 2. 시스템 아키텍처

### 2.1 전체 시스템 구성도

```
┌─────────────────────────────────────────────────────────────┐
│                    중앙 제어 시스템                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │   Raspberry Pi 4 (MQTT Broker + 대시보드)           │   │
│  │   - Mosquitto MQTT Broker                           │   │
│  │   - InfluxDB (데이터 저장)                          │   │
│  │   - Grafana (시각화)                                │   │
│  │   - Node-RED (자동화)                               │   │
│  └─────────────────────────────────────────────────────┘   │
│                            ↕ WiFi Network                    │
└─────────────────────────────────────────────────────────────┘
                               ↕
        ┌──────────────────────┼──────────────────────┐
        ↓                      ↓                      ↓
┌───────────────┐      ┌───────────────┐     ┌───────────────┐
│  센서 모듈 1  │      │  센서 모듈 2  │     │  액추에이터   │
│  (온습도)     │      │  (EC/pH)      │     │  모듈 1       │
│               │      │               │     │  (릴레이)     │
│  ESP32-C3     │      │  ESP32-C3     │     │  ESP32-C3     │
│  + DHT22      │      │  + EC센서     │     │  + 4채널릴레이│
│  + 전원       │      │  + pH센서     │     │  + 전원       │
└───────────────┘      └───────────────┘     └───────────────┘
      WiFi                   WiFi                  WiFi
      MQTT                   MQTT                  MQTT
```

### 2.2 통신 플로우

```
1. 모듈 부팅
   ↓
2. WiFi 연결 (저장된 SSID/Password 또는 SmartConfig)
   ↓
3. MQTT Broker 연결 (Raspberry Pi)
   ↓
4. 고유 Topic 등록 (예: smartfarm/zone1/sensor/temperature)
   ↓
5. 데이터 송신 또는 제어 명령 수신 대기
   ↓
6. 주기적 상태 보고 (Heartbeat)
```

### 2.3 모듈 분류

**센서 모듈 (Publisher)**
- 환경 데이터 수집
- 주기적으로 MQTT Topic에 Publish
- 저전력 설계 (Deep Sleep 지원)

**액추에이터 모듈 (Subscriber)**
- 제어 명령 수신
- MQTT Topic에서 Subscribe
- 릴레이/모터/밸브 제어

**하이브리드 모듈 (Pub/Sub)**
- 센서 + 액추에이터 통합
- 양방향 통신

---

## 3. 하드웨어 설계

### 3.1 핵심 MCU 선정

#### ESP32-C3 선택 이유
| 특징 | 사양 | 장점 |
|------|------|------|
| CPU | RISC-V 32-bit @ 160MHz | 저비용, 고성능 |
| WiFi | 802.11 b/g/n | 안정적 무선 통신 |
| Bluetooth | BLE 5.0 | 추가 확장 가능 |
| GPIO | 22개 | 충분한 I/O |
| ADC | 12-bit, 6채널 | 아날로그 센서 지원 |
| 전력 | 활성 ~80mA, Sleep ~5μA | 저전력 |
| 가격 | $1.5~2 | 매우 저렴 |
| 크기 | QFN32 (5×5mm) | 소형 |

**대안**: ESP32-S3 (카메라 필요 시), ESP8266 (초저비용)

### 3.2 표준 모듈 하드웨어 블록

```
┌────────────────────────────────────────────────────┐
│              표준 WiFi 제어 모듈                   │
├────────────────────────────────────────────────────┤
│                                                    │
│  ┌──────────┐    ┌──────────────┐                │
│  │ 전원부   │───▶│   ESP32-C3   │                │
│  │ 5V→3.3V  │    │              │                │
│  │ AMS1117  │    │  WiFi + BLE  │                │
│  └──────────┘    │              │                │
│                  │  RISC-V MCU  │                │
│  ┌──────────┐    └──────┬───────┘                │
│  │  LED     │           │                        │
│  │ 상태표시 │◀──────────┤                        │
│  └──────────┘           │                        │
│                         │                        │
│  ┌──────────┐           │                        │
│  │ 센서/    │◀──────────┤                        │
│  │액추에이터│           │                        │
│  │ 인터페이스│           │                        │
│  └──────────┘           │                        │
│      ↕                  │                        │
│  ┌──────────┐           │                        │
│  │ 커넥터   │           │                        │
│  │ (선택)   │           │                        │
│  └──────────┘    ┌──────┴───────┐                │
│                  │   플래시      │                │
│  전원 입력       │   4MB        │                │
│  5V DC          └──────────────┘                │
│  (Micro USB                                      │
│   또는 단자대)                                    │
└────────────────────────────────────────────────────┘
```

### 3.3 전원 설계

#### 입력 전원
- **입력 전압**: 5V DC (USB 또는 외부 전원)
- **최대 전류**: 500mA~2A (모듈 종류에 따라)
- **커넥터**: Micro USB 또는 단자대 (Phoenix Connector)

#### 전압 레귤레이터
- **3.3V LDO**: AMS1117-3.3 (1A 출력)
- **입력 보호**: 역전압 보호 다이오드
- **필터링**: 
  - 입력: 100μF 전해 + 100nF 세라믹
  - 출력: 22μF 전해 + 100nF 세라믹

#### 전력 소비
| 모드 | 전류 | 설명 |
|------|------|------|
| Active (WiFi TX) | ~160mA | 데이터 송신 중 |
| Active (WiFi RX) | ~80mA | 대기 중 |
| Modem Sleep | ~20mA | WiFi 연결 유지 |
| Light Sleep | ~800μA | 주기적 Wake-up |
| Deep Sleep | ~5μA | 거의 꺼짐 |

### 3.4 센서 인터페이스

#### 디지털 센서 (I2C, 1-Wire, UART)
```
ESP32-C3 GPIO
  ├─ GPIO 4, 5: I2C (SDA, SCL) - Pull-up 4.7kΩ
  ├─ GPIO 6: 1-Wire (Dallas DS18B20 등)
  ├─ GPIO 20, 21: UART (TX, RX)
  └─ GPIO 8, 9: SPI (MOSI, MISO) - 선택적
```

#### 아날로그 센서
```
ESP32-C3 ADC
  ├─ GPIO 0~4: ADC1 (12-bit, 0~3.3V)
  │   - 토양 수분 센서
  │   - 조도 센서 (포토다이오드)
  │   - 전압/전류 측정
  └─ 주의: WiFi 사용 시 ADC2 사용 불가
```

### 3.5 액추에이터 인터페이스

#### 릴레이 제어
```
ESP32-C3 GPIO → 트랜지스터 → 릴레이 코일
  - GPIO 출력 (3.3V, 최대 40mA)
  - NPN 트랜지스터 (2N2222, BC547)
  - 플라이백 다이오드 (1N4007)
  - 릴레이: 5V 코일, 10A 접점
```

**회로 예시**:
```
GPIO 10 ──┬─ 1kΩ ─── BC547 Base
          │               │ Collector
          └─ 10kΩ ─── GND │
                          │
                     Relay Coil (+5V)
                          │
                     1N4007 (Flyback)
                          │
                         GND
```

#### 모터/밸브 제어
```
ESP32-C3 GPIO → MOSFET → 모터/솔레노이드
  - Logic-level MOSFET (IRLZ44N, 2A 이상)
  - 또는 L298N 모터 드라이버 IC
```

---

## 4. 소프트웨어 설계

### 4.1 펌웨어 구조

```
┌─────────────────────────────────────────┐
│           Application Layer             │
│  - 센서 읽기 / 액추에이터 제어         │
│  - 비즈니스 로직                        │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│          MQTT Client Layer              │
│  - Publish / Subscribe                  │
│  - QoS 관리                             │
│  - 메시지 큐                            │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│           WiFi Manager                  │
│  - WiFi 연결 관리                       │
│  - SmartConfig / WPS                    │
│  - 자동 재연결                          │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│        ESP-IDF / Arduino Core           │
│  - Hardware Abstraction Layer           │
└─────────────────────────────────────────┘
```

### 4.2 핵심 기능 구현

#### 4.2.1 WiFi 자동 연결

```cpp
// WiFi 설정 구조체
struct WiFiConfig {
    char ssid[32];
    char password[64];
    bool saved;
};

// WiFi 연결 함수
void connectWiFi() {
    WiFiConfig config;
    
    // 저장된 설정 로드 (NVS)
    if (loadWiFiConfig(&config) && config.saved) {
        // 저장된 SSID로 연결 시도
        WiFi.begin(config.ssid, config.password);
        
        // 30초 대기
        int timeout = 30;
        while (WiFi.status() != WL_CONNECTED && timeout-- > 0) {
            delay(1000);
            Serial.print(".");
        }
        
        if (WiFi.status() == WL_CONNECTED) {
            Serial.println("WiFi Connected!");
            return;
        }
    }
    
    // 연결 실패 시 SmartConfig 모드
    Serial.println("Starting SmartConfig...");
    WiFi.beginSmartConfig();
    
    // SmartConfig 대기
    while (!WiFi.smartConfigDone()) {
        delay(500);
        Serial.print(".");
    }
    
    Serial.println("SmartConfig done!");
    
    // 새 설정 저장
    strcpy(config.ssid, WiFi.SSID().c_str());
    strcpy(config.password, WiFi.psk().c_str());
    config.saved = true;
    saveWiFiConfig(&config);
}

// WiFi 재연결 (백그라운드)
void WiFiKeepAlive(void *parameter) {
    while (1) {
        if (WiFi.status() != WL_CONNECTED) {
            Serial.println("WiFi disconnected. Reconnecting...");
            WiFi.reconnect();
        }
        vTaskDelay(10000 / portTICK_PERIOD_MS); // 10초마다 체크
    }
}
```

#### 4.2.2 MQTT 클라이언트 구현

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

WiFiClient espClient;
PubSubClient mqtt(espClient);

// MQTT 설정
const char* MQTT_SERVER = "192.168.1.100"; // Raspberry Pi IP
const int MQTT_PORT = 1883;
const char* MQTT_USER = "smartfarm";       // 선택적
const char* MQTT_PASS = "password";        // 선택적

// Topic 설정 (모듈 ID 기반)
String MODULE_ID = "zone1_temp";
String TOPIC_PUB = "smartfarm/" + MODULE_ID + "/data";
String TOPIC_SUB = "smartfarm/" + MODULE_ID + "/control";
String TOPIC_STATUS = "smartfarm/" + MODULE_ID + "/status";

// MQTT 콜백 (메시지 수신)
void mqttCallback(char* topic, byte* payload, unsigned int length) {
    Serial.print("Message received on topic: ");
    Serial.println(topic);
    
    // Payload를 문자열로 변환
    String message = "";
    for (int i = 0; i < length; i++) {
        message += (char)payload[i];
    }
    
    Serial.print("Message: ");
    Serial.println(message);
    
    // JSON 파싱 및 제어 로직
    handleControlMessage(message);
}

// MQTT 연결
void connectMQTT() {
    while (!mqtt.connected()) {
        Serial.print("Connecting to MQTT broker...");
        
        String clientId = "ESP32-" + MODULE_ID;
        
        if (mqtt.connect(clientId.c_str(), MQTT_USER, MQTT_PASS)) {
            Serial.println("connected");
            
            // 온라인 상태 발행
            mqtt.publish(TOPIC_STATUS.c_str(), "online", true);
            
            // Subscribe (액추에이터 모듈만)
            mqtt.subscribe(TOPIC_SUB.c_str());
            
            Serial.println("MQTT ready");
        } else {
            Serial.print("failed, rc=");
            Serial.print(mqtt.state());
            Serial.println(" retry in 5 seconds");
            delay(5000);
        }
    }
}

// MQTT 초기화
void setupMQTT() {
    mqtt.setServer(MQTT_SERVER, MQTT_PORT);
    mqtt.setCallback(mqttCallback);
    mqtt.setKeepAlive(60); // Heartbeat 60초
}

// MQTT 루프 (연결 유지)
void mqttLoop() {
    if (!mqtt.connected()) {
        connectMQTT();
    }
    mqtt.loop();
}
```

#### 4.2.3 센서 데이터 발행 예시

```cpp
#include <DHT.h>
#include <ArduinoJson.h>

#define DHTPIN 4
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

unsigned long lastPublish = 0;
const long PUBLISH_INTERVAL = 60000; // 1분

void publishSensorData() {
    unsigned long now = millis();
    
    if (now - lastPublish >= PUBLISH_INTERVAL) {
        lastPublish = now;
        
        // 센서 읽기
        float temp = dht.readTemperature();
        float humi = dht.readHumidity();
        
        if (isnan(temp) || isnan(humi)) {
            Serial.println("Failed to read sensor!");
            return;
        }
        
        // JSON 생성
        StaticJsonDocument<200> doc;
        doc["temperature"] = temp;
        doc["humidity"] = humi;
        doc["timestamp"] = millis() / 1000;
        
        String json;
        serializeJson(doc, json);
        
        // MQTT 발행
        mqtt.publish(TOPIC_PUB.c_str(), json.c_str());
        
        Serial.print("Published: ");
        Serial.println(json);
    }
}

void loop() {
    mqttLoop();
    publishSensorData();
    delay(100);
}
```

#### 4.2.4 액추에이터 제어 예시

```cpp
#define RELAY1_PIN 10
#define RELAY2_PIN 11
#define RELAY3_PIN 12
#define RELAY4_PIN 13

void setupActuator() {
    pinMode(RELAY1_PIN, OUTPUT);
    pinMode(RELAY2_PIN, OUTPUT);
    pinMode(RELAY3_PIN, OUTPUT);
    pinMode(RELAY4_PIN, OUTPUT);
    
    // 초기 상태 OFF
    digitalWrite(RELAY1_PIN, LOW);
    digitalWrite(RELAY2_PIN, LOW);
    digitalWrite(RELAY3_PIN, LOW);
    digitalWrite(RELAY4_PIN, LOW);
}

void handleControlMessage(String message) {
    // JSON 파싱
    StaticJsonDocument<200> doc;
    DeserializationError error = deserializeJson(doc, message);
    
    if (error) {
        Serial.println("JSON parse failed!");
        return;
    }
    
    // 제어 명령 처리
    if (doc.containsKey("relay1")) {
        bool state = doc["relay1"];
        digitalWrite(RELAY1_PIN, state ? HIGH : LOW);
        Serial.print("Relay 1: ");
        Serial.println(state ? "ON" : "OFF");
    }
    
    if (doc.containsKey("relay2")) {
        bool state = doc["relay2"];
        digitalWrite(RELAY2_PIN, state ? HIGH : LOW);
    }
    
    // 상태 피드백 발행
    publishActuatorStatus();
}

void publishActuatorStatus() {
    StaticJsonDocument<200> doc;
    doc["relay1"] = digitalRead(RELAY1_PIN);
    doc["relay2"] = digitalRead(RELAY2_PIN);
    doc["relay3"] = digitalRead(RELAY3_PIN);
    doc["relay4"] = digitalRead(RELAY4_PIN);
    
    String json;
    serializeJson(doc, json);
    
    mqtt.publish(TOPIC_PUB.c_str(), json.c_str());
}
```

### 4.3 OTA (Over-The-Air) 업데이트

```cpp
#include <ArduinoOTA.h>

void setupOTA() {
    ArduinoOTA.setHostname(MODULE_ID.c_str());
    ArduinoOTA.setPassword("admin"); // 보안 비밀번호
    
    ArduinoOTA.onStart([]() {
        String type;
        if (ArduinoOTA.getCommand() == U_FLASH) {
            type = "sketch";
        } else { // U_SPIFFS
            type = "filesystem";
        }
        Serial.println("Start updating " + type);
    });
    
    ArduinoOTA.onEnd([]() {
        Serial.println("\nEnd");
    });
    
    ArduinoOTA.onProgress([](unsigned int progress, unsigned int total) {
        Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
    });
    
    ArduinoOTA.onError([](ota_error_t error) {
        Serial.printf("Error[%u]: ", error);
        if (error == OTA_AUTH_ERROR) Serial.println("Auth Failed");
        else if (error == OTA_BEGIN_ERROR) Serial.println("Begin Failed");
        else if (error == OTA_CONNECT_ERROR) Serial.println("Connect Failed");
        else if (error == OTA_RECEIVE_ERROR) Serial.println("Receive Failed");
        else if (error == OTA_END_ERROR) Serial.println("End Failed");
    });
    
    ArduinoOTA.begin();
}

void loop() {
    ArduinoOTA.handle();
    mqttLoop();
    publishSensorData();
}
```

---

## 5. MQTT 통신 프로토콜

### 5.1 Topic 구조 설계

```
smartfarm/
├── zone1/                      # 구역 1
│   ├── sensor/
│   │   ├── temperature         # 온도 센서
│   │   ├── humidity            # 습도 센서
│   │   ├── ec                  # EC 센서
│   │   └── ph                  # pH 센서
│   ├── actuator/
│   │   ├── pump1/control       # 펌프 제어 (Subscribe)
│   │   ├── pump1/status        # 펌프 상태 (Publish)
│   │   ├── valve1/control
│   │   └── valve1/status
│   └── status/
│       ├── module1             # 모듈 온라인/오프라인
│       └── module2
│
├── zone2/                      # 구역 2
│   └── ...
│
├── control/                    # 전역 제어
│   ├── all/emergency_stop      # 긴급 정지
│   └── broadcast               # 전체 방송
│
└── system/                     # 시스템 메타
    ├── discovery               # 자동 발견
    ├── config                  # 설정 배포
    └── ota                     # OTA 업데이트
```

### 5.2 메시지 포맷 (JSON)

#### 센서 데이터
```json
{
    "sensor_id": "zone1_temp_humi",
    "type": "DHT22",
    "timestamp": 1697845200,
    "data": {
        "temperature": 25.3,
        "humidity": 65.2
    },
    "unit": {
        "temperature": "°C",
        "humidity": "%"
    },
    "quality": "good"
}
```

#### 제어 명령
```json
{
    "command": "set",
    "actuator_id": "zone1_pump1",
    "action": "turn_on",
    "duration": 300,
    "timestamp": 1697845200
}
```

#### 상태 피드백
```json
{
    "module_id": "zone1_pump1",
    "status": "running",
    "uptime": 86400,
    "wifi_rssi": -45,
    "free_heap": 180000,
    "timestamp": 1697845200
}
```

### 5.3 QoS 레벨 설정

| Topic | QoS | 이유 |
|-------|-----|------|
| sensor/* | 0 | 주기적 데이터, 손실 허용 |
| actuator/*/control | 1 | 제어 명령, 최소 1회 전달 |
| status/* | 1 | 상태 정보, 신뢰성 필요 |
| control/all/* | 2 | 긴급 명령, 정확히 1회 |

### 5.4 Retained Message 활용

```cpp
// 모듈 온라인 상태 (LWT 설정)
mqtt.publish("smartfarm/zone1/status/module1", "online", true);

// Last Will and Testament (연결 끊김 시 자동 발행)
mqtt.setWill("smartfarm/zone1/status/module1", "offline", true, 1);
```

---

## 6. PCB 설계

### 6.1 PCB 사양

| 항목 | 사양 |
|------|------|
| 크기 | 50mm × 40mm (표준형) |
| 층수 | 2층 (Top, Bottom) |
| 기판 재질 | FR-4, 1.6mm |
| 동박 두께 | 1oz (35μm) |
| 최소 선폭/간격 | 0.2mm / 0.2mm |
| 실크스크린 | 양면 |
| 솔더마스크 | 녹색 (양면) |
| 표면처리 | HASL (무연) |

### 6.2 PCB 레이아웃 가이드

```
┌──────────────────────────────────────────────┐
│  [전원 입력]  Micro USB 또는 단자대          │
│                                              │
│  ┌─────────┐         ┌──────────────┐      │
│  │ 전원부  │────────▶│   ESP32-C3   │      │
│  │ LDO     │         │              │      │
│  │ 3.3V    │         │   (중앙배치) │      │
│  └─────────┘         └──────────────┘      │
│                             │               │
│  ┌─────────┐         ┌──────┴──────┐       │
│  │ LED     │◀────────│   GPIO      │       │
│  │ 상태등  │         │   헤더      │       │
│  └─────────┘         └─────────────┘       │
│                                              │
│  [센서/액추에이터 커넥터]                    │
│   - JST-XH 커넥터 (3~6핀)                   │
│   - 또는 나사식 단자대                       │
│                                              │
│  [프로그래밍 헤더]                           │
│   - 2.54mm 핀헤더 (TX, RX, GND, 3.3V)      │
└──────────────────────────────────────────────┘
```

### 6.3 레이어별 설계

**Top Layer (부품면)**
```
- ESP32-C3 모듈 (중앙)
- LDO 레귤레이터
- 디커플링 커패시터 (ESP32 주변에 근접 배치)
- LED, 저항
- 커넥터 (전원, 센서)
```

**Bottom Layer (납땜면)**
```
- GND Plane (전체 면적의 80% 이상)
- 전원 라인 (굵게, 20mil 이상)
- 리턴 패스 최소화
```

### 6.4 안테나 설계 고려사항

**ESP32-C3 내장 안테나 사용 시**
```
- PCB 가장자리에 배치
- 안테나 아래 50mm × 20mm 영역 Keep-Out
- GND Plane 없음 (안테나 성능 저하 방지)
- 금속 케이스 사용 시 외부 안테나 필요
```

**외부 안테나 (선택적)**
```
- U.FL / IPEX 커넥터
- 2.4GHz 전용 안테나 (3dBi~5dBi)
- 50Ω 임피던스 매칭
```

### 6.5 회로도 예시 (센서 모듈)

```
                    +5V
                     │
                  ┌──┴──┐
                  │     │
                 100μF 100nF
                  │     │
                  └──┬──┘
                     │
                  ┌──▼───┐
  Micro USB ─────▶│AMS1117│
  5V               │ 3.3V │
                   └───┬──┘
                       │ +3.3V
                    ┌──┴──┐
                    │     │
                   22μF 100nF
                    │     │
                    └──┬──┘
                       │
            ┌──────────▼──────────────┐
            │      ESP32-C3           │
            │                         │
  DHT22 ────┤ GPIO4 (Data)           │
            │                         │
  I2C SDA ──┤ GPIO5 (SDA)            │
  I2C SCL ──┤ GPIO6 (SCL)            │
            │                         │
  LED ──────┤ GPIO8 (Status)         │
            │                         │
  Button ───┤ GPIO9 (Boot/Reset)     │
            │                         │
            └─────────────────────────┘
                       │
                      GND
```

---

## 7. 모듈 종류 및 사양

### 7.1 센서 모듈

#### 7.1.1 온습도 센서 모듈

**사양**
- MCU: ESP32-C3
- 센서: DHT22 또는 SHT31
- 측정 범위:
  - 온도: -40~80°C (정확도 ±0.5°C)
  - 습도: 0~100% RH (정확도 ±2%)
- 샘플링: 1분 간격
- 전력: 평균 80mA

**BOM (Bill of Materials)**
| 부품 | 수량 | 단가 | 합계 |
|------|------|------|------|
| ESP32-C3 모듈 | 1 | 3,000원 | 3,000원 |
| DHT22 센서 | 1 | 2,000원 | 2,000원 |
| AMS1117-3.3 | 1 | 300원 | 300원 |
| 커패시터, 저항 | - | 500원 | 500원 |
| PCB | 1 | 2,000원 | 2,000원 |
| 커넥터 | 2 | 500원 | 1,000원 |
| **총계** | | | **8,800원** |

---

#### 7.1.2 EC/pH 센서 모듈

**사양**
- MCU: ESP32-C3
- 센서: 
  - EC 센서 (TDS 측정)
  - pH 센서 (BNC 커넥터)
- 측정 범위:
  - EC: 0~10 mS/cm
  - pH: 0~14
- 샘플링: 5분 간격
- 보정: 자동 온도 보상
- 전력: 평균 100mA

**BOM**
| 부품 | 수량 | 단가 | 합계 |
|------|------|------|------|
| ESP32-C3 모듈 | 1 | 3,000원 | 3,000원 |
| EC 센서 | 1 | 8,000원 | 8,000원 |
| pH 센서 | 1 | 12,000원 | 12,000원 |
| BNC 커넥터 | 2 | 1,000원 | 2,000원 |
| 신호 조정 회로 | 1 | 3,000원 | 3,000원 |
| 기타 부품 | - | 2,000원 | 2,000원 |
| **총계** | | | **30,000원** |

---

#### 7.1.3 토양 센서 모듈

**사양**
- MCU: ESP32-C3
- 센서: 용량형 토양 수분 센서
- 측정: 토양 수분 (0~100%)
- 샘플링: 10분 간격
- 전력: 평균 60mA

**BOM**
| 부품 | 수량 | 단가 | 합계 |
|------|------|------|------|
| ESP32-C3 모듈 | 1 | 3,000원 | 3,000원 |
| 토양 수분 센서 | 1 | 2,500원 | 2,500원 |
| 기타 부품 | - | 2,000원 | 2,000원 |
| **총계** | | | **7,500원** |

---

### 7.2 액추에이터 모듈

#### 7.2.1 4채널 릴레이 모듈

**사양**
- MCU: ESP32-C3
- 릴레이: 4채널, 5V 코일
- 접점: 10A @ 250V AC / 30V DC
- 용도: 펌프, 환풍기, 조명 제어
- 보호: 플라이백 다이오드, LED 표시
- 전력: 평균 150mA

**제어 예시**
```json
{
    "relay1": true,   // 펌프 ON
    "relay2": false,  // 환풍기 OFF
    "relay3": true,   // 조명 ON
    "relay4": false   // 예비
}
```

**BOM**
| 부품 | 수량 | 단가 | 합계 |
|------|------|------|------|
| ESP32-C3 모듈 | 1 | 3,000원 | 3,000원 |
| 5V 릴레이 | 4 | 1,500원 | 6,000원 |
| 트랜지스터 | 4 | 200원 | 800원 |
| 다이오드 | 4 | 100원 | 400원 |
| 단자대 | 5 | 500원 | 2,500원 |
| 기타 부품 | - | 2,000원 | 2,000원 |
| **총계** | | | **14,700원** |

---

#### 7.2.2 솔레노이드 밸브 제어 모듈

**사양**
- MCU: ESP32-C3
- 출력: 2채널 고전류 MOSFET
- 전류: 최대 5A per channel
- 용도: 양액 공급, 물 공급
- 보호: 과전류 차단, 역전압 방지

**BOM**
| 부품 | 수량 | 단가 | 합계 |
|------|------|------|------|
| ESP32-C3 모듈 | 1 | 3,000원 | 3,000원 |
| MOSFET (IRLZ44N) | 2 | 800원 | 1,600원 |
| 전류 센서 | 2 | 2,000원 | 4,000원 |
| 기타 부품 | - | 2,000원 | 2,000원 |
| **총계** | | | **10,600원** |

---

### 7.3 하이브리드 모듈

#### 7.3.1 올인원 환경 제어 모듈

**사양**
- MCU: ESP32-S3 (더 많은 GPIO)
- 센서:
  - DHT22 (온습도)
  - BH1750 (조도)
  - MH-Z19B (CO2)
- 액추에이터:
  - 2채널 릴레이 (환풍기, 히터)
- 디스플레이: OLED 128×64 (선택)

**BOM**
| 부품 | 수량 | 단가 | 합계 |
|------|------|------|------|
| ESP32-S3 모듈 | 1 | 5,000원 | 5,000원 |
| DHT22 | 1 | 2,000원 | 2,000원 |
| BH1750 | 1 | 1,500원 | 1,500원 |
| MH-Z19B | 1 | 18,000원 | 18,000원 |
| 릴레이 | 2 | 1,500원 | 3,000원 |
| OLED (선택) | 1 | 3,000원 | 3,000원 |
| 기타 부품 | - | 3,000원 | 3,000원 |
| **총계** | | | **35,500원** |

---

## 8. 제작 및 비용

### 8.1 PCB 제작 (10개 기준)

**중국 업체 (JLCPCB, PCBWay)**
| 항목 | 사양 | 가격 |
|------|------|------|
| PCB | 50×40mm, 2층, 10개 | $5~10 |
| 배송 | DHL (5일) | $15~20 |
| **총계** | | **$20~30 (2.5~4만원)** |
| **개당** | | **2,500~4,000원** |

**국내 업체 (예: 코코넛)**
| 항목 | 사양 | 가격 |
|------|------|------|
| PCB | 50×40mm, 2층, 10개 | 30,000원 |
| 배송 | 국내 (2일) | 무료 |
| **총계** | | **30,000원** |
| **개당** | | **3,000원** |

### 8.2 부품 구매

**AliExpress / 타오바오**
- 배송: 2~4주
- 가격: 가장 저렴 (50% 할인)
- 품질: 검증 필요

**국내 업체 (ICbanQ, 디바이스마트)**
- 배송: 1~3일
- 가격: 중간
- 품질: 신뢰성 높음

**권장**: 프로토타입은 국내, 양산은 해외

### 8.3 조립

**수동 조립**
- 도구: 인두기, 멀티미터, 핀셋
- 시간: 모듈당 30분~1시간
- 비용: 무료 (직접 작업)

**PCBA 서비스**
- JLCPCB SMT 서비스
- BOM 업로드 + 부품 자동 장착
- 추가 비용: $30~50 (10개 기준)

### 8.4 총 제작 비용 (10개 기준)

#### 온습도 센서 모듈
| 항목 | 단가 | 10개 |
|------|------|------|
| PCB | 3,000원 | 30,000원 |
| 부품 | 5,800원 | 58,000원 |
| 조립 | 무료 | 무료 |
| **총계** | **8,800원** | **88,000원** |

#### 4채널 릴레이 모듈
| 항목 | 단가 | 10개 |
|------|------|------|
| PCB | 3,000원 | 30,000원 |
| 부품 | 11,700원 | 117,000원 |
| 조립 | 무료 | 무료 |
| **총계** | **14,700원** | **147,000원** |

---

## 9. 설치 및 사용법

### 9.1 초기 설정 (최초 1회)

**Step 1: WiFi 설정 (SmartConfig)**

1. 모듈에 전원 연결 (5V)
2. LED가 빠르게 깜빡임 (SmartConfig 모드)
3. 스마트폰에 "ESP Touch" 앱 설치
   - iOS: App Store
   - Android: Google Play
4. 앱에서 WiFi SSID/Password 입력
5. 모듈이 자동으로 WiFi 연결
6. LED가 천천히 깜빡임 (정상 동작)

**Step 2: MQTT Broker 설정**

모듈은 하드코딩된 MQTT Broker IP로 자동 연결
```cpp
const char* MQTT_SERVER = "192.168.1.100"; // 수정 필요 시 재컴파일
```

또는 웹 설정 페이지 제공 (선택적)
```
http://192.168.1.xxx  (모듈 IP)
→ WiFi/MQTT 설정 변경
```

### 9.2 일상 사용

**전원만 연결하면 끝!**
```
1. 전원 공급 (5V DC)
   ↓
2. 자동 WiFi 연결
   ↓
3. 자동 MQTT 연결
   ↓
4. 센서 데이터 자동 전송 또는 제어 대기
```

### 9.3 대시보드에서 확인

**Grafana 대시보드**
```
http://192.168.1.100:3000
- 온도/습도 그래프
- 릴레이 상태
- 모듈 온라인 상태
```

**Node-RED 제어**
```
http://192.168.1.100:1880
- 수동 제어 버튼
- 자동화 룰 설정
- 알림 설정
```

### 9.4 문제 해결

| 문제 | LED 상태 | 해결 방법 |
|------|---------|----------|
| WiFi 연결 실패 | 빠른 깜빡임 | SmartConfig 재시도 |
| MQTT 연결 실패 | 느린 깜빡임 | Broker IP 확인 |
| 센서 오류 | 항상 켜짐 | 센서 연결 확인 |
| 전원 부족 | 불규칙 깜빡임 | 전원 공급 확인 |

---

## 10. 테스트 계획

### 10.1 하드웨어 테스트

#### 전원 테스트
```
□ 입력 전압 범위 테스트 (4.5V~5.5V)
□ 출력 전압 안정성 (3.3V ±0.1V)
□ 리플 노이즈 측정 (<100mV)
□ 전류 소비 측정 (각 모드별)
□ 과전류 보호 동작 확인
```

#### GPIO 테스트
```
□ 모든 GPIO 입출력 동작 확인
□ ADC 정확도 테스트 (멀티미터 대조)
□ I2C/UART 통신 확인
□ PWM 출력 확인
```

#### 센서 테스트
```
□ DHT22 온습도 정확도 (표준 센서 대조)
□ EC/pH 센서 보정 및 정확도
□ 장시간 안정성 테스트 (24시간)
```

#### 릴레이 테스트
```
□ 개폐 동작 확인
□ 접점 전압/전류 테스트
□ 수명 테스트 (10,000회 반복)
```

### 10.2 소프트웨어 테스트

#### WiFi 테스트
```
□ 자동 연결 성공률 (10회 중 10회)
□ 재연결 시간 측정 (<10초)
□ 신호 세기별 안정성 (-30dBm ~ -80dBm)
□ 라우터 재부팅 후 자동 복구
```

#### MQTT 테스트
```
□ Broker 연결 성공률
□ QoS 0, 1, 2 메시지 전달 확인
□ 메시지 지연 시간 측정 (<100ms)
□ 네트워크 끊김 후 재연결
□ LWT (Last Will) 동작 확인
```

#### 데이터 정확성 테스트
```
□ JSON 파싱 오류 없음
□ 타임스탬프 정확도
□ 센서 값 범위 검증
□ 이상값 필터링
```

### 10.3 통합 테스트

#### 시나리오 1: 센서 → 대시보드
```
1. 센서 모듈 전원 ON
2. WiFi/MQTT 자동 연결 확인
3. Grafana에서 실시간 데이터 확인
4. 1분마다 업데이트 확인
```

#### 시나리오 2: 대시보드 → 액추에이터
```
1. Node-RED에서 릴레이 ON 명령
2. MQTT 메시지 전송 확인
3. 릴레이 모듈 동작 확인 (LED)
4. 상태 피드백 수신 확인
```

#### 시나리오 3: 자동화
```
1. Node-RED에서 룰 설정 (온도 > 28°C → 환풍기 ON)
2. 온도 센서에 열풍기로 가열
3. 자동으로 환풍기 릴레이 ON 확인
4. 온도 하강 후 릴레이 OFF 확인
```

### 10.4 환경 테스트

#### 온도 테스트
```
□ 저온: -10°C (실외 동작)
□ 고온: 60°C (온실 내부)
□ 온도 충격: -10°C ↔ 60°C 반복
```

#### 습도 테스트
```
□ 건조: 10% RH
□ 고습: 90% RH
□ 결로 방지 확인 (방수 코팅)
```

#### 전자파 적합성 (EMC)
```
□ 전자파 방출 (EMI) 측정
□ 전자파 내성 (EMS) 테스트
□ WiFi 간섭 최소화 확인
```

### 10.5 신뢰성 테스트

#### 장기 안정성
```
□ 연속 운전: 7일 무중단
□ 메모리 누수 없음
□ WiFi/MQTT 재연결 횟수 기록
```

#### 전원 사이클
```
□ 1,000회 전원 ON/OFF 반복
□ 부팅 시간 일관성
□ 설정 저장 확인 (NVS)
```

---

## 11. 확장 기능

### 11.1 웹 설정 페이지

**ESP32 AsyncWebServer 사용**
```cpp
#include <ESPAsyncWebServer.h>

AsyncWebServer server(80);

void setupWebServer() {
    // 설정 페이지
    server.on("/", HTTP_GET, [](AsyncWebServerRequest *request){
        String html = R"(
        <html>
        <body>
            <h1>SmartFarm WiFi Module</h1>
            <form action="/config" method="POST">
                WiFi SSID: <input name="ssid"><br>
                Password: <input name="pass"><br>
                MQTT Server: <input name="mqtt"><br>
                <input type="submit" value="Save">
            </form>
        </body>
        </html>
        )";
        request->send(200, "text/html", html);
    });
    
    // 설정 저장
    server.on("/config", HTTP_POST, [](AsyncWebServerRequest *request){
        String ssid = request->arg("ssid");
        String pass = request->arg("pass");
        String mqtt = request->arg("mqtt");
        
        // NVS에 저장
        saveConfig(ssid, pass, mqtt);
        
        request->send(200, "text/plain", "Saved! Rebooting...");
        delay(1000);
        ESP.restart();
    });
    
    server.begin();
}
```

### 11.2 자동 발견 (Discovery)

**mDNS (Multicast DNS)**
```cpp
#include <ESPmDNS.h>

void setupMDNS() {
    if (MDNS.begin("smartfarm-module1")) {
        Serial.println("mDNS responder started");
        MDNS.addService("mqtt", "tcp", 1883);
    }
}

// 네트워크에서 발견:
// smartfarm-module1.local
```

### 11.3 LED 상태 표시

```cpp
#define LED_PIN 8

enum LEDStatus {
    OFF,              // 전원 없음
    SLOW_BLINK,       // WiFi 연결 중 (1초 간격)
    FAST_BLINK,       // MQTT 연결 중 (0.2초 간격)
    HEARTBEAT,        // 정상 동작 (2초마다 짧게)
    SOLID_ON          // 오류
};

void updateLED(LEDStatus status) {
    static unsigned long lastToggle = 0;
    static bool ledState = false;
    unsigned long now = millis();
    
    switch (status) {
        case SLOW_BLINK:
            if (now - lastToggle > 1000) {
                ledState = !ledState;
                digitalWrite(LED_PIN, ledState);
                lastToggle = now;
            }
            break;
        // ... 기타 상태
    }
}
```

### 11.4 배터리 지원 (선택)

**리튬 배터리 + 충전 모듈**
```
- 18650 리튬 배터리 (3.7V, 3000mAh)
- TP4056 충전 모듈
- 배터리 전압 모니터링 (ADC)
- Deep Sleep으로 수명 연장
```

**예상 배터리 수명**
| 모드 | 전류 | 시간 | 수명 |
|------|------|------|------|
| 연속 활성 | 80mA | 24시간 | ~37시간 |
| Sleep 60초 | 10mA | 평균 | ~12일 |
| Deep Sleep 10분 | 1mA | 평균 | ~4개월 |

---

## 12. 부록

### 12.1 필요 도구 목록

**하드웨어 제작**
```
□ 인두기 (온도 조절식, 300~400°C)
□ 인두기 팁 (0.5mm 뾰족한 팁)
□ 땜납 (무연, 0.8mm)
□ 플럭스
□ 핀셋 (ESD 안전)
□ 멀티미터
□ 오실로스코프 (선택)
□ 전원 공급기 (5V, 2A)
□ 돋보기 또는 현미경
```

**소프트웨어 개발**
```
□ Arduino IDE 또는 PlatformIO
□ USB-UART 변환기 (CH340, CP2102)
□ Git (버전 관리)
□ MQTT 클라이언트 (MQTT Explorer)
```

### 12.2 참고 자료

**공식 문서**
- ESP32-C3 Datasheet: https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_en.pdf
- Arduino-ESP32: https://github.com/espressif/arduino-esp32
- PubSubClient: https://pubsubclient.knolleary.net/

**오픈소스 프로젝트**
- Tasmota: https://tasmota.github.io/
- ESPHome: https://esphome.io/
- OpenMQTTGateway: https://github.com/1technophile/OpenMQTTGateway

**커뮤니티**
- ESP32 Forum: https://www.esp32.com/
- Arduino Forum: https://forum.arduino.cc/
- Reddit r/esp32: https://www.reddit.com/r/esp32/

### 12.3 샘플 코드 저장소

**GitHub 저장소 구조**
```
smartfarm-wifi-modules/
├── hardware/
│   ├── pcb/                  # KiCad 프로젝트
│   │   ├── sensor_module/
│   │   ├── relay_module/
│   │   └── library/          # 공통 심볼/풋프린트
│   ├── bom/                  # BOM 파일
│   └── gerber/               # Gerber 파일
│
├── firmware/
│   ├── common/               # 공통 라이브러리
│   │   ├── wifi_manager/
│   │   ├── mqtt_client/
│   │   └── ota_update/
│   ├── sensor_modules/
│   │   ├── temp_humidity/
│   │   ├── ec_ph/
│   │   └── soil_moisture/
│   ├── actuator_modules/
│   │   ├── relay_4ch/
│   │   └── valve_controller/
│   └── platformio.ini
│
├── server/                   # Raspberry Pi 설정
│   ├── mosquitto/            # MQTT Broker
│   ├── node-red/             # 자동화 플로우
│   ├── grafana/              # 대시보드
│   └── docker-compose.yml
│
├── docs/                     # 문서
│   ├── assembly_guide.md
│   ├── user_manual.md
│   └── api_reference.md
│
└── README.md
```

### 12.4 라이센스

**하드웨어**
- CERN Open Hardware Licence v2 - Strongly Reciprocal (CERN-OHL-S-2.0)

**소프트웨어**
- MIT License

**문서**
- Creative Commons Attribution 4.0 International (CC BY 4.0)

---

## 13. 로드맵

### Phase 1: 프로토타입 (1개월)
- [x] 시스템 설계 완료
- [ ] PCB 설계 및 제작 (2주)
- [ ] 펌웨어 개발 (2주)
- [ ] 초기 테스트

### Phase 2: 베타 테스트 (2개월)
- [ ] 10개 모듈 제작
- [ ] 실제 온실에서 테스트
- [ ] 피드백 수집 및 개선

### Phase 3: 양산 준비 (3개월)
- [ ] 최종 디자인 확정
- [ ] CE/FCC 인증 (선택)
- [ ] 사용자 매뉴얼 작성
- [ ] 판매 시작

### Phase 4: 추가 모듈 개발
- [ ] 카메라 모듈 (ESP32-CAM)
- [ ] LoRa 장거리 통신 모듈
- [ ] 태양광 전원 모듈

---

## 14. 결론

### 14.1 프로젝트 장점

✅ **진정한 플러그앤플레이**
- 전원만 연결하면 자동으로 모든 것이 설정됨
- 사용자가 복잡한 설정 불필요

✅ **저비용 고효율**
- 모듈당 5,000원~30,000원
- 기존 상용 제품 대비 1/5~1/10 비용

✅ **모듈식 설계**
- 필요에 따라 모듈 추가/교체 가능
- 확장성 우수

✅ **오픈소스**
- 하드웨어/소프트웨어 모두 공개
- 커뮤니티 기여 가능

✅ **무선 통신**
- 배선 없이 WiFi로 통신
- 설치 간편

### 14.2 적용 효과

**500평 스마트팜 기준**

| 항목 | 기존 유선 | WiFi 모듈 | 절감 효과 |
|------|---------|-----------|----------|
| 센서 노드 | 10개 × 150,000원 | 10개 × 10,000원 | **1,400,000원** |
| 배선 비용 | 500,000원 | 0원 | **500,000원** |
| 설치 시간 | 2일 | 4시간 | **인건비 절감** |
| 유지보수 | 어려움 | 쉬움 | **시간 절감** |
| **총 절감** | | | **약 200만원** |

### 14.3 향후 개선 사항

1. **보안 강화**: TLS/SSL 암호화, 인증서 기반 인증
2. **메시 네트워크**: ESP-MESH로 장거리 확장
3. **AI 통합**: Edge AI로 로컬 의사결정
4. **에너지 하베스팅**: 태양광 패널 통합
5. **케이스 디자인**: 방수/방진 케이스 설계

---

**프로젝트 연락처**  
이메일: smartfarm@example.com  
GitHub: https://github.com/smartfarm-wifi-modules

**마지막 업데이트**: 2025년 10월 20일
