# 스마트팜 IoT 시스템 구축 가이드
## 라즈베리파이 + MQTT + ESP32로 저렴하고 확장 가능한 시스템 만들기

---

## 목차
1. [시스템 아키텍처 개요](#1-시스템-아키텍처-개요)
2. [하드웨어 구성](#2-하드웨어-구성)
3. [소프트웨어 스택](#3-소프트웨어-스택)
4. [단계별 구현 가이드](#4-단계별-구현-가이드)
5. [비용 분석](#5-비용-분석)
6. [확장 전략](#6-확장-전략)
7. [유지보수 및 모니터링](#7-유지보수-및-모니터링)

---

## 1. 시스템 아키텍처 개요

### 1.1 전체 구조

```
┌─────────────────────────────────────────────────────────┐
│                    클라우드 (선택)                       │
│               InfluxDB Cloud / Grafana Cloud            │
└──────────────────────┬──────────────────────────────────┘
                       │ (인터넷)
                       │
┌──────────────────────┴──────────────────────────────────┐
│              라즈베리파이 (중앙 서버)                    │
│  ┌────────────────────────────────────────────────┐     │
│  │ MQTT Broker (Mosquitto)                        │     │
│  │ - 포트: 1883 (일반), 8883 (TLS)               │     │
│  │ - 메시지 중계                                  │     │
│  └────────────┬───────────────────────────────────┘     │
│               │                                          │
│  ┌────────────┴───────────────────────────────────┐     │
│  │ Node-RED                                       │     │
│  │ - 비주얼 프로그래밍                            │     │
│  │ - 자동화 로직                                  │     │
│  │ - 대시보드                                     │     │
│  └────────────┬───────────────────────────────────┘     │
│               │                                          │
│  ┌────────────┴───────────────────────────────────┐     │
│  │ InfluxDB (로컬)                                │     │
│  │ - 시계열 데이터베이스                          │     │
│  │ - 센서 데이터 저장                             │     │
│  └────────────────────────────────────────────────┘     │
│                                                          │
│  ┌──────────────────────────────────────────────┐       │
│  │ Grafana (선택)                               │       │
│  │ - 시각화 대시보드                            │       │
│  └──────────────────────────────────────────────┘       │
└──────────────────────┬──────────────────────────────────┘
                       │ WiFi Network
                       │
        ┌──────────────┼──────────────┬──────────────┐
        │              │              │              │
   ┌────┴───┐    ┌────┴───┐    ┌────┴───┐    ┌────┴───┐
   │ ESP32  │    │ ESP32  │    │ ESP32  │    │ ESP32  │
   │ Zone 1 │    │ Zone 2 │    │ Zone 3 │    │ Zone N │
   └────┬───┘    └────┬───┘    └────┬───┘    └────┬───┘
        │             │             │             │
   ┌────┴─────┐  ┌───┴──────┐ ┌───┴──────┐ ┌───┴──────┐
   │ 센서     │  │ 센서     │ │ 센서     │ │ 센서     │
   │ 릴레이   │  │ 릴레이   │ │ 릴레이   │ │ 릴레이   │
   └──────────┘  └──────────┘ └──────────┘ └──────────┘
```

### 1.2 시스템 특징

#### ✅ 장점
```
1. 확장성
   - ESP32 노드 무제한 추가 가능
   - Zone 기반 관리
   - 모듈화된 구조

2. 저비용
   - 오픈소스 기반
   - 저렴한 하드웨어
   - 라이선스 비용 없음

3. 안정성
   - MQTT QoS 지원
   - 자동 재연결
   - 오프라인 동작 가능

4. 유연성
   - 다양한 센서 지원
   - 맞춤형 로직 구현
   - 클라우드 연동 선택 가능

5. 유지보수성
   - 원격 관리 가능
   - OTA 펌웨어 업데이트
   - 실시간 모니터링
```

#### 💡 핵심 개념
```
MQTT (Message Queue Telemetry Transport):
- 경량 메시징 프로토콜
- Publish/Subscribe 모델
- IoT에 최적화
- 저전력, 저대역폭

Topic 구조:
smartfarm/
├── zone1/
│   ├── sensors/temp
│   ├── sensors/humidity
│   ├── sensors/soil
│   └── control/relay1
├── zone2/
└── status/...
```

---

## 2. 하드웨어 구성

### 2.1 중앙 서버: 라즈베리파이

#### 권장 모델
```
┌────────────────┬──────────────┬──────────┬────────────┐
│ 모델           │ 가격         │ 권장도   │ 비고       │
├────────────────┼──────────────┼──────────┼────────────┤
│ RPi 5 (8GB)    │ ~120,000원   │ ★★★★★    │ 최고성능   │
│ RPi 4 (4GB)    │ ~80,000원    │ ★★★★☆    │ 가성비 최고│
│ RPi 4 (2GB)    │ ~60,000원    │ ★★★☆☆    │ 소규모 추천│
│ RPi Zero 2W    │ ~25,000원    │ ★★☆☆☆    │ 테스트용   │
└────────────────┴──────────────┴──────────┴────────────┘

★ 권장: Raspberry Pi 4 (4GB)
  - 500평 기준 충분한 성능
  - 동시 연결: 50개 이상
  - 여유 있는 리소스
```

#### 필수 주변기기
```
1. 마이크로SD 카드
   - 용량: 32GB 이상 (권장 64GB)
   - 속도: Class 10, A1 등급
   - 가격: ~15,000원
   - 추천: Samsung EVO Plus 64GB

2. 전원 어댑터
   - 출력: 5V/3A (USB-C for Pi 4/5)
   - 공식 어댑터 권장
   - 가격: ~15,000원

3. 케이스
   - 방열 케이스 권장
   - 팬 포함 권장
   - 가격: ~10,000원

4. 이더넷 케이블 (선택)
   - WiFi보다 안정적
   - Cat 6 이상
   - 가격: ~5,000원

총 비용: 약 125,000원
```

### 2.2 센서 노드: ESP32

#### ESP32 모델 선택
```
┌─────────────────────┬─────────┬──────────┬──────────┐
│ 모델                │ 가격    │ 권장도   │ 특징     │
├─────────────────────┼─────────┼──────────┼──────────┤
│ ESP32 DevKit V1     │ 5,000원 │ ★★★★★    │ 범용     │
│ ESP32-WROOM-32      │ 6,000원 │ ★★★★☆    │ 안정성   │
│ ESP32-C3            │ 4,000원 │ ★★★☆☆    │ 저렴     │
│ ESP32-S3            │ 8,000원 │ ★★★★☆    │ 고성능   │
└─────────────────────┴─────────┴──────────┴──────────┘

★ 권장: ESP32 DevKit V1
  - 충분한 GPIO 핀 (30개)
  - WiFi 내장
  - 저렴한 가격
  - 풍부한 자료
```

#### Zone당 센서 구성 (예시)

```
┌──────────────────────────────────────────────────────┐
│          1개 Zone (150m² = 약 45평)                  │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ESP32 × 1개                          5,000원       │
│  ─────────────────────────────────────────────       │
│                                                      │
│  센서:                                               │
│  ├─ DHT22 (온습도)          × 1      5,000원       │
│  ├─ Soil Moisture (토양)    × 2      6,000원       │
│  ├─ LDR (조도)              × 1      1,000원       │
│  └─ CO2 (MH-Z19B) (선택)    × 1     35,000원       │
│                                                      │
│  액추에이터:                                         │
│  ├─ 릴레이 모듈 (4CH)       × 1      8,000원       │
│  ├─ 팬 (환기)               × 2     10,000원       │
│  └─ 전자밸브 (관수)         × 1     15,000원       │
│                                                      │
│  기타:                                               │
│  ├─ 점퍼 와이어                      3,000원       │
│  ├─ 브레드보드                       2,000원       │
│  ├─ 5V 전원 (3A)                     8,000원       │
│  └─ 방수 박스                       10,000원       │
│                                                      │
│  ─────────────────────────────────────────────       │
│  기본 구성 (CO2 제외):     총 73,000원              │
│  풀 구성 (CO2 포함):       총 108,000원             │
└──────────────────────────────────────────────────────┘

500평 (1,650m²) 기준:
- Zone 수: 10개
- 총 비용 (기본): 730,000원
- 총 비용 (풀): 1,080,000원
```

### 2.3 네트워크 장비

```
WiFi 공유기:
- 범위: 500평 커버 가능
- 표준: WiFi 5 (802.11ac) 이상
- 동시 연결: 50개 이상
- 가격: 50,000~100,000원
- 추천: ipTIME A3004NS-M

메시 WiFi (대규모 시):
- 3팩 기준: 200,000원
- 추천: ipTIME Giga WiFi Mesh

이더넷 스위치 (선택):
- 8포트 기가비트
- 가격: 30,000원
```

### 2.4 총 하드웨어 비용 (500평 기준)

```
┌─────────────────────────┬──────────────┐
│ 항목                    │ 금액         │
├─────────────────────────┼──────────────┤
│ 라즈베리파이 세트       │   125,000원  │
│ ESP32 Zone (10개)       │   730,000원  │
│ WiFi 공유기             │    80,000원  │
│ 예비 부품 (10%)         │    93,500원  │
├─────────────────────────┼──────────────┤
│ **총 하드웨어 비용**    │**1,028,500원**│
│                         │**(약 100만원)**│
└─────────────────────────┴──────────────┘

★ 기존 계획서 IoT 비용: 14,150,000원
★ 이 방식: 1,028,500원
★ 절감: 13,121,500원 (93% 절감!) ✓✓✓
```

---

## 3. 소프트웨어 스택

### 3.1 라즈베리파이 소프트웨어

#### OS 설치
```bash
# Raspberry Pi OS Lite (64-bit) 권장
# 이유: 
# - GUI 불필요 (리소스 절약)
# - 안정적
# - 장기 지원

다운로드: 
https://www.raspberrypi.com/software/

설치 도구:
Raspberry Pi Imager 사용
- WiFi 설정 사전 구성
- SSH 활성화
- 사용자 계정 설정
```

#### 필수 소프트웨어 스택

```
┌────────────────────────────────────────┐
│  Operating System                      │
│  Raspberry Pi OS (Debian-based)        │
└────────────┬───────────────────────────┘
             │
┌────────────┴───────────────────────────┐
│  MQTT Broker                           │
│  Eclipse Mosquitto                     │
│  - 포트: 1883 (MQTT)                   │
│  - 포트: 8883 (MQTTS - TLS)            │
└────────────┬───────────────────────────┘
             │
┌────────────┴───────────────────────────┐
│  Automation & Dashboard                │
│  Node-RED                              │
│  - 포트: 1880                          │
│  - 비주얼 프로그래밍                   │
└────────────┬───────────────────────────┘
             │
┌────────────┴───────────────────────────┐
│  Time-Series Database                  │
│  InfluxDB 2.x                          │
│  - 포트: 8086                          │
└────────────┬───────────────────────────┘
             │
┌────────────┴───────────────────────────┐
│  Visualization (선택)                  │
│  Grafana                               │
│  - 포트: 3000                          │
└────────────────────────────────────────┘
```

### 3.2 ESP32 개발 환경

#### Arduino IDE 방식 (초보자 권장 ★★★★★)

```
장점:
✓ 쉬운 설치
✓ 풍부한 라이브러리
✓ 많은 예제 코드
✓ 큰 커뮤니티

단점:
✗ 메모리 관리 제한
✗ 고급 기능 제한

추천 대상:
- 프로그래밍 초보자
- 빠른 프로토타이핑
- 간단한 로직
```

#### PlatformIO 방식 (중급 이상 ★★★★☆)

```
장점:
✓ 프로젝트 관리 우수
✓ 라이브러리 관리 자동화
✓ 여러 보드 동시 관리
✓ VSCode 통합

단점:
✗ 학습 곡선 있음
✗ 초기 설정 복잡

추천 대상:
- 경험 있는 개발자
- 대규모 프로젝트
- 팀 협업
```

#### MicroPython 방식 (실험적 ★★★☆☆)

```
장점:
✓ Python 문법 사용
✓ 빠른 개발
✓ 대화형 REPL

단점:
✗ 성능 낮음
✗ 메모리 사용량 큼
✗ 라이브러리 제한적

추천 대상:
- Python 개발자
- 교육 목적
- 간단한 프로토타입
```

### 3.3 필수 라이브러리

#### ESP32 라이브러리
```cpp
// Arduino IDE 라이브러리 매니저에서 설치

1. PubSubClient
   - 용도: MQTT 통신
   - 작성자: Nick O'Leary
   - 버전: 2.8 이상

2. ArduinoJson
   - 용도: JSON 데이터 처리
   - 작성자: Benoit Blanchon
   - 버전: 6.x

3. DHT sensor library
   - 용도: DHT22 센서
   - 작성자: Adafruit
   - 버전: 1.4.x

4. WiFi (내장)
   - 용도: WiFi 연결

5. ESPAsyncWebServer (선택)
   - 용도: 웹 설정 페이지
   - GitHub: me-no-dev

6. ArduinoOTA (선택)
   - 용도: 무선 펌웨어 업데이트
   - 내장 라이브러리
```

---

## 4. 단계별 구현 가이드

### Phase 1: 라즈베리파이 설정 (2시간)

#### Step 1.1: OS 설치

```bash
# 1. Raspberry Pi Imager로 SD카드에 OS 설치
# 2. 부팅 후 업데이트

sudo apt update
sudo apt upgrade -y
sudo apt install -y git curl vim

# 3. 시간대 설정
sudo timedatectl set-timezone Asia/Seoul

# 4. 고정 IP 설정 (선택, 권장)
sudo nano /etc/dhcpcd.conf

# 추가:
# interface eth0
# static ip_address=192.168.1.100/24
# static routers=192.168.1.1
# static domain_name_servers=8.8.8.8

# 재부팅
sudo reboot
```

#### Step 1.2: Mosquitto 설치

```bash
# 1. Mosquitto 설치
sudo apt install -y mosquitto mosquitto-clients

# 2. 서비스 활성화
sudo systemctl enable mosquitto
sudo systemctl start mosquitto

# 3. 설정 파일 수정
sudo nano /etc/mosquitto/mosquitto.conf

# 다음 내용 추가:
# ─────────────────────────────────
# 기본 설정
listener 1883
protocol mqtt

# 모든 IP 허용
bind_address 0.0.0.0

# 익명 접근 허용 (테스트용)
allow_anonymous true

# 로그 설정
log_dest file /var/log/mosquitto/mosquitto.log
log_type all
log_timestamp true

# 지속성
persistence true
persistence_location /var/lib/mosquitto/

# 보안 (나중에 설정)
# password_file /etc/mosquitto/passwd
# ─────────────────────────────────

# 4. 서비스 재시작
sudo systemctl restart mosquitto

# 5. 테스트
# 터미널 1:
mosquitto_sub -h localhost -t test/topic

# 터미널 2:
mosquitto_pub -h localhost -t test/topic -m "Hello MQTT"
```

#### Step 1.3: Node-RED 설치

```bash
# 1. Node.js 및 Node-RED 설치
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)

# 2. 서비스 활성화
sudo systemctl enable nodered.service
sudo systemctl start nodered.service

# 3. 접속 확인
# 브라우저에서: http://[라즈베리파이IP]:1880

# 4. 필수 노드 설치
# Node-RED 인터페이스 > 메뉴 > Manage palette > Install
# - node-red-dashboard (대시보드)
# - node-red-contrib-influxdb (InfluxDB)
# - node-red-node-email (알림)
```

#### Step 1.4: InfluxDB 설치

```bash
# 1. InfluxDB 2.x 설치
wget -q https://repos.influxdata.com/influxdata-archive_compat.key
echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null

echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list

sudo apt update
sudo apt install -y influxdb2

# 2. 서비스 시작
sudo systemctl enable influxdb
sudo systemctl start influxdb

# 3. 초기 설정
# 브라우저: http://[라즈베리파이IP]:8086
# - 사용자 생성
# - Organization: smartfarm
# - Bucket: sensors
# - API 토큰 생성 및 저장
```

#### Step 1.5: Grafana 설치 (선택)

```bash
# 1. Grafana 설치
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_10.2.3_arm64.deb
sudo dpkg -i grafana_10.2.3_arm64.deb

# 2. 서비스 시작
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

# 3. 접속
# 브라우저: http://[라즈베리파이IP]:3000
# 기본 로그인: admin/admin

# 4. InfluxDB 데이터 소스 추가
# Configuration > Data Sources > Add > InfluxDB
```

### Phase 2: ESP32 개발 환경 (1시간)

#### Step 2.1: Arduino IDE 설치 및 설정

```bash
# Windows/Mac/Linux: 
# https://www.arduino.cc/en/software

# 설치 후:

1. 파일 > 환경설정 > 추가 보드 매니저 URLs:
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

2. 도구 > 보드 > 보드 매니저 > "esp32" 검색 > 설치

3. 도구 > 보드 > ESP32 Arduino > ESP32 Dev Module

4. 드라이버 설치:
   - CP210x: Silicon Labs
   - CH340: WCH
   
5. 포트 선택:
   도구 > 포트 > COM포트 선택
```

#### Step 2.2: 필수 라이브러리 설치

```
스케치 > 라이브러리 포함하기 > 라이브러리 관리

설치할 라이브러리:
1. PubSubClient (Nick O'Leary)
2. ArduinoJson (6.x)
3. DHT sensor library (Adafruit)
4. Adafruit Unified Sensor
```

### Phase 3: 첫 번째 센서 노드 구현 (3시간)

#### Step 3.1: ESP32 기본 코드

```cpp
// smartfarm_node.ino
// 기본 센서 노드 코드

#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>
#include <ArduinoJson.h>

// ============================================
// 설정 (각 노드마다 변경)
// ============================================
const char* WIFI_SSID = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
const char* MQTT_SERVER = "192.168.1.100";  // 라즈베리파이 IP
const int MQTT_PORT = 1883;
const char* NODE_ID = "zone1";  // 각 노드마다 고유값

// ============================================
// 핀 설정
// ============================================
#define DHT_PIN 4           // DHT22 센서 핀
#define SOIL1_PIN 34        // 토양 센서 1 (ADC)
#define SOIL2_PIN 35        // 토양 센서 2 (ADC)
#define LDR_PIN 32          // 조도 센서 (ADC)
#define RELAY1_PIN 16       // 릴레이 1 (관수)
#define RELAY2_PIN 17       // 릴레이 2 (환기팬)
#define RELAY3_PIN 18       // 릴레이 3 (예비)
#define RELAY4_PIN 19       // 릴레이 4 (예비)

// ============================================
// 객체 생성
// ============================================
DHT dht(DHT_PIN, DHT22);
WiFiClient espClient;
PubSubClient mqtt(espClient);

// ============================================
// 타이머 변수
// ============================================
unsigned long lastSensorRead = 0;
unsigned long lastStatusSend = 0;
const long SENSOR_INTERVAL = 30000;    // 30초마다 센서 읽기
const long STATUS_INTERVAL = 60000;    // 1분마다 상태 전송

// ============================================
// WiFi 연결
// ============================================
void setupWiFi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
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
    Serial.println();
    Serial.println("WiFi connected");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
  } else {
    Serial.println();
    Serial.println("WiFi connection failed!");
  }
}

// ============================================
// MQTT 콜백
// ============================================
void mqttCallback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  
  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println(message);

  // JSON 파싱
  StaticJsonDocument<200> doc;
  DeserializationError error = deserializeJson(doc, message);
  
  if (error) {
    Serial.print("JSON parse failed: ");
    Serial.println(error.c_str());
    return;
  }

  // 릴레이 제어
  // Topic: smartfarm/zone1/control/relay1
  String topicStr = String(topic);
  
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
  // 추가 릴레이도 동일하게...
}

// ============================================
// MQTT 재연결
// ============================================
void reconnectMQTT() {
  while (!mqtt.connected()) {
    Serial.print("Attempting MQTT connection...");
    
    String clientId = "ESP32_";
    clientId += NODE_ID;
    
    if (mqtt.connect(clientId.c_str())) {
      Serial.println("connected");
      
      // 제어 토픽 구독
      String controlTopic = "smartfarm/" + String(NODE_ID) + "/control/#";
      mqtt.subscribe(controlTopic.c_str());
      Serial.print("Subscribed to: ");
      Serial.println(controlTopic);
      
      // 연결 상태 발행
      publishStatus("online");
      
    } else {
      Serial.print("failed, rc=");
      Serial.print(mqtt.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

// ============================================
// 센서 읽기
// ============================================
void readSensors() {
  // DHT22 읽기
  float temp = dht.readTemperature();
  float humidity = dht.readHumidity();
  
  // 토양 수분 읽기 (0-4095 -> 0-100%)
  int soil1Raw = analogRead(SOIL1_PIN);
  int soil2Raw = analogRead(SOIL2_PIN);
  float soil1 = map(soil1Raw, 0, 4095, 0, 100);
  float soil2 = map(soil2Raw, 0, 4095, 0, 100);
  
  // 조도 읽기 (0-4095 -> 0-100%)
  int ldrRaw = analogRead(LDR_PIN);
  float light = map(ldrRaw, 0, 4095, 0, 100);
  
  // 센서 오류 체크
  if (isnan(temp) || isnan(humidity)) {
    Serial.println("Failed to read from DHT sensor!");
    temp = -999;
    humidity = -999;
  }
  
  // JSON 생성
  StaticJsonDocument<300> doc;
  doc["node"] = NODE_ID;
  doc["timestamp"] = millis();
  
  JsonObject sensors = doc.createNestedObject("sensors");
  sensors["temperature"] = temp;
  sensors["humidity"] = humidity;
  sensors["soil1"] = soil1;
  sensors["soil2"] = soil2;
  sensors["light"] = light;
  
  // MQTT 발행
  char buffer[512];
  serializeJson(doc, buffer);
  
  String topic = "smartfarm/" + String(NODE_ID) + "/sensors";
  mqtt.publish(topic.c_str(), buffer);
  
  Serial.println("Sensor data published:");
  Serial.println(buffer);
}

// ============================================
// 상태 발행
// ============================================
void publishStatus(const char* status) {
  StaticJsonDocument<200> doc;
  doc["node"] = NODE_ID;
  doc["status"] = status;
  doc["uptime"] = millis() / 1000;
  doc["rssi"] = WiFi.RSSI();
  doc["ip"] = WiFi.localIP().toString();
  
  char buffer[256];
  serializeJson(doc, buffer);
  
  String topic = "smartfarm/" + String(NODE_ID) + "/status";
  mqtt.publish(topic.c_str(), buffer);
}

// ============================================
// Setup
// ============================================
void setup() {
  Serial.begin(115200);
  Serial.println();
  Serial.println("SmartFarm Node Starting...");
  Serial.print("Node ID: ");
  Serial.println(NODE_ID);
  
  // 핀 모드 설정
  pinMode(RELAY1_PIN, OUTPUT);
  pinMode(RELAY2_PIN, OUTPUT);
  pinMode(RELAY3_PIN, OUTPUT);
  pinMode(RELAY4_PIN, OUTPUT);
  
  // 초기 상태: 모든 릴레이 OFF
  digitalWrite(RELAY1_PIN, LOW);
  digitalWrite(RELAY2_PIN, LOW);
  digitalWrite(RELAY3_PIN, LOW);
  digitalWrite(RELAY4_PIN, LOW);
  
  // DHT 센서 초기화
  dht.begin();
  
  // WiFi 연결
  setupWiFi();
  
  // MQTT 설정
  mqtt.setServer(MQTT_SERVER, MQTT_PORT);
  mqtt.setCallback(mqttCallback);
  mqtt.setBufferSize(512);  // 버퍼 크기 증가
  
  Serial.println("Setup complete!");
}

// ============================================
// Loop
// ============================================
void loop() {
  // WiFi 재연결
  if (WiFi.status() != WL_CONNECTED) {
    setupWiFi();
  }
  
  // MQTT 재연결
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
  
  // 짧은 딜레이
  delay(10);
}
```

#### Step 3.2: 배선도

```
ESP32 DevKit V1 배선:

센서:
├─ DHT22
│  ├─ VCC  → 3.3V
│  ├─ DATA → GPIO 4
│  └─ GND  → GND
│
├─ Soil Moisture 1
│  ├─ VCC  → 3.3V
│  ├─ AOUT → GPIO 34 (VP)
│  └─ GND  → GND
│
├─ Soil Moisture 2
│  ├─ VCC  → 3.3V
│  ├─ AOUT → GPIO 35 (VN)
│  └─ GND  → GND
│
└─ LDR (조도)
   ├─ VCC  → 3.3V
   ├─ AOUT → GPIO 32
   └─ GND  → GND

릴레이 모듈 (4CH):
├─ VCC  → 5V
├─ GND  → GND
├─ IN1  → GPIO 16
├─ IN2  → GPIO 17
├─ IN3  → GPIO 18
└─ IN4  → GPIO 19

전원:
- ESP32: 5V 2A (USB 또는 VIN)
- 릴레이: 별도 5V 3A 권장

주의사항:
⚠ ADC2 핀(GPIO 0,2,4,12-15,25-27)은 WiFi 사용 시 불안정
✓ ADC1 핀(GPIO 32-39) 사용 권장
✓ 릴레이는 HIGH = OFF, LOW = ON (반대일 수도 있음)
```

### Phase 4: Node-RED 자동화 (2시간)

#### Step 4.1: 기본 Flow 예시

```json
[
  {
    "id": "mqtt_in",
    "type": "mqtt in",
    "topic": "smartfarm/+/sensors",
    "broker": "mqtt_broker",
    "name": "Sensor Data",
    "outputs": 1
  },
  {
    "id": "json_parse",
    "type": "json",
    "name": "Parse JSON"
  },
  {
    "id": "influx_write",
    "type": "influxdb out",
    "database": "sensors",
    "name": "Save to InfluxDB"
  },
  {
    "id": "dashboard_gauge",
    "type": "ui_gauge",
    "group": "sensors_group",
    "name": "Temperature"
  }
]

설명:
1. MQTT 구독 → 2. JSON 파싱 → 3. InfluxDB 저장
                                 └→ 4. 대시보드 표시
```

#### Step 4.2: 자동 제어 로직

```javascript
// Function 노드 예시: 자동 관수

// 입력: 센서 데이터
// msg.payload = {
//   node: "zone1",
//   sensors: {
//     soil1: 30,
//     soil2: 35
//   }
// }

const soil1 = msg.payload.sensors.soil1;
const soil2 = msg.payload.sensors.soil2;
const avgSoil = (soil1 + soil2) / 2;

// 토양 수분이 40% 이하이면 관수
if (avgSoil < 40) {
    msg.payload = {
        state: true
    };
    msg.topic = "smartfarm/" + msg.payload.node + "/control/relay1";
    node.status({fill:"green",shape:"dot",text:"관수 ON"});
    return msg;
} 
// 토양 수분이 70% 이상이면 중지
else if (avgSoil > 70) {
    msg.payload = {
        state: false
    };
    msg.topic = "smartfarm/" + msg.payload.node + "/control/relay1";
    node.status({fill:"red",shape:"dot",text:"관수 OFF"});
    return msg;
}

// 40-70% 사이면 유지
return null;
```

---

## 5. 비용 분석

### 5.1 하드웨어 비용 비교

```
┌──────────────────────────┬─────────────┬─────────────┐
│ 항목                     │ 기존 계획   │ 이 방식     │
├──────────────────────────┼─────────────┼─────────────┤
│ 중앙 서버                │             │             │
│ - 라즈베리파이 + SD      │      0원    │  125,000원  │
│                          │             │             │
│ 제어 시스템              │             │             │
│ - 라즈베리파이 × 2       │ 8,650,000원 │       0원   │
│ - 소프트웨어 개발        │ 3,000,000원 │       0원   │
│ - 설치 및 시운전         │ 2,500,000원 │       0원   │
│                          │             │             │
│ 센서 노드 (10개 Zone)    │             │             │
│ - ESP32 기반             │      0원    │  730,000원  │
│ - 상용 IoT 컨트롤러      │ (포함 위)   │       0원   │
│                          │             │             │
│ 네트워크                 │             │             │
│ - WiFi 장비              │      0원    │   80,000원  │
│                          │             │             │
│ 예비 부품                │      0원    │   93,500원  │
├──────────────────────────┼─────────────┼─────────────┤
│ **합계**                 │**14,150,000**│**1,028,500**│
│                          │**(1,415만원)**│**(103만원)**│
├──────────────────────────┼─────────────┼─────────────┤
│ **절감액**               │      -      │ 13,121,500원│
│ **절감율**               │      -      │    **92.7%**│
└──────────────────────────┴─────────────┴─────────────┘

💰 1,312만원 절감!
```

### 5.2 소프트웨어 비용

```
┌──────────────────────────┬─────────────┬─────────────┐
│ 항목                     │ 상용 솔루션 │ 이 방식     │
├──────────────────────────┼─────────────┼─────────────┤
│ MQTT Broker              │ 무료~수백만 │    무료     │
│ Node-RED                 │     -       │    무료     │
│ InfluxDB (오픈소스)      │     -       │    무료     │
│ Grafana (오픈소스)       │     -       │    무료     │
│ 개발/컨설팅              │ 3,000,000원 │    무료     │
│                          │             │ (직접 구현) │
├──────────────────────────┼─────────────┼─────────────┤
│ **합계**                 │   3,000,000+│       0원   │
└──────────────────────────┴─────────────┴─────────────┘
```

### 5.3 운영 비용 (연간)

```
┌──────────────────────────┬─────────────┬─────────────┐
│ 항목                     │ 기존 계획   │ 이 방식     │
├──────────────────────────┼─────────────┼─────────────┤
│ 전기료                   │             │             │
│ - 라즈베리파이 4 (15W)   │      0원    │   65,000원  │
│ - ESP32 × 10 (1W each)   │      0원    │   43,000원  │
│ - WiFi 공유기 (10W)      │      0원    │   43,000원  │
│ 소계                     │      0원    │  151,000원  │
│                          │             │             │
│ 유지보수                 │             │             │
│ - 부품 교체              │  500,000원  │  100,000원  │
│ - 소프트웨어 업데이트    │  300,000원  │       0원   │
│ 소계                     │  800,000원  │  100,000원  │
│                          │             │             │
│ 통신                     │             │             │
│ - 인터넷 (기존 사용)     │       0원   │       0원   │
│                          │             │             │
├──────────────────────────┼─────────────┼─────────────┤
│ **연간 합계**            │  800,000원  │  251,000원  │
├──────────────────────────┼─────────────┼─────────────┤
│ **절감액**               │      -      │  549,000원  │
└──────────────────────────┴─────────────┴─────────────┘

💡 연간 55만원 절감
```

### 5.4 총 비용 (5년 기준)

```
┌──────────────────────────┬─────────────┬─────────────┐
│ 항목                     │ 상용 방식   │ DIY 방식    │
├──────────────────────────┼─────────────┼─────────────┤
│ 초기 투자                │ 14,150,000원│  1,028,500원│
│ 운영비 × 5년             │  4,000,000원│  1,255,000원│
├──────────────────────────┼─────────────┼─────────────┤
│ **5년 총 비용**          │**18,150,000원**│**2,283,500원**│
├──────────────────────────┼─────────────┼─────────────┤
│ **절감액**               │       -     │15,866,500원 │
│ **절감율**               │       -     │   **87.4%** │
└──────────────────────────┴─────────────┴─────────────┘

🎯 5년간 1,587만원 절감!
```

---

## 6. 확장 전략

### 6.1 단계별 확장

```
Phase 1: 최소 시스템 (1-2개 Zone)
├─ 라즈베리파이 + Mosquitto
├─ ESP32 × 2
├─ 기본 센서
└─ 비용: ~250,000원

Phase 2: 기본 시스템 (5개 Zone)
├─ Phase 1 +
├─ Node-RED 대시보드
├─ InfluxDB
├─ ESP32 × 3 추가
└─ 비용: ~500,000원

Phase 3: 완전 시스템 (10개 Zone)
├─ Phase 2 +
├─ Grafana
├─ ESP32 × 5 추가
├─ 고급 센서 (CO2 등)
└─ 비용: ~1,000,000원

Phase 4: 고급 기능
├─ Phase 3 +
├─ 카메라 모니터링
├─ AI 분석 (Edge TPU)
├─ 클라우드 백업
└─ 비용: +500,000원
```

### 6.2 Zone 확장 방법

```python
# 새 Zone 추가 시:

1. ESP32 구매
   - 비용: ~70,000원 (센서 포함)
   
2. 코드 수정
   const char* NODE_ID = "zone11";  # 변경
   
3. 배선 및 설치
   - 소요 시간: 2시간
   
4. Node-RED Flow 복제
   - 기존 Flow 복사
   - Topic만 수정
   - 소요 시간: 10분
   
5. 완료!

추가 비용: 70,000원
추가 시간: 2.5시간
```

### 6.3 기능 확장

#### 카메라 추가
```
ESP32-CAM 모듈:
- 비용: 8,000원
- 기능: 실시간 영상, 타임랩스
- 통합: MQTT로 이미지 전송
- 저장: Node-RED → 파일시스템

활용:
- 작물 생육 모니터링
- 병해충 감지 (AI 연동)
- 원격 확인
```

#### 날씨 데이터 통합
```javascript
// Node-RED에서 OpenWeatherMap API 사용
// 무료: 60 calls/min

http request 노드:
URL: http://api.openweathermap.org/data/2.5/weather
      ?q=Seoul&appid=YOUR_API_KEY

활용:
- 외부 온도와 비교
- 강우 예측
- 자동 환기 제어
```

#### 알림 시스템
```javascript
// Telegram Bot 통합
// Node-RED telegram 노드

알림 조건:
- 온도 이상
- 토양 수분 부족
- ESP32 오프라인
- 릴레이 고장

구현:
npm install node-red-contrib-telegrambot
```

---

## 7. 유지보수 및 모니터링

### 7.1 일상 점검 (5분/일)

```
Node-RED 대시보드 확인:
□ 모든 Zone 온라인 상태
□ 센서 값 정상 범위
□ 릴레이 동작 상태
□ 시스템 리소스 (CPU, 메모리)

이상 발견 시:
→ MQTT topic 확인
→ ESP32 시리얼 모니터 확인
→ 센서/릴레이 물리적 확인
```

### 7.2 주간 점검 (30분/주)

```
□ 라즈베리파이 재부팅
  sudo reboot
  
□ 로그 확인
  tail -100 /var/log/mosquitto/mosquitto.log
  
□ 디스크 공간 확인
  df -h
  
□ InfluxDB 데이터 확인
  - 오래된 데이터 삭제 (3개월 이상)
  
□ 백업
  - SD카드 이미지 백업 (월 1회)
```

### 7.3 문제 해결 가이드

```
문제 1: ESP32가 MQTT 연결 안됨
원인:
- WiFi 문제
- Mosquitto 중단
- IP 주소 변경

해결:
1. WiFi 신호 확인
2. Mosquitto 재시작: sudo systemctl restart mosquitto
3. 라즈베리파이 IP 확인: ifconfig
4. ESP32 코드의 MQTT_SERVER 확인

─────────────────────────────────────

문제 2: 센서 값이 이상함
원인:
- 센서 고장
- 배선 불량
- 전원 부족

해결:
1. 시리얼 모니터에서 원시 값 확인
2. 센서 재연결
3. 전원 전압 측정 (멀티미터)
4. 센서 교체

─────────────────────────────────────

문제 3: 릴레이 동작 안함
원인:
- GPIO 핀 문제
- 릴레이 모듈 고장
- 로직 반전

해결:
1. digitalWrite 테스트 (시리얼 모니터)
2. 릴레이 LED 확인
3. HIGH/LOW 반전 시도
4. 릴레이 교체

─────────────────────────────────────

문제 4: 라즈베리파이 느림
원인:
- SD카드 수명
- 메모리 부족
- 프로세스 과다

해결:
1. free -h (메모리 확인)
2. top (프로세스 확인)
3. SD카드 교체
4. 불필요한 서비스 중지
```

### 7.4 성능 최적화

```bash
# 1. Mosquitto 설정 최적화
sudo nano /etc/mosquitto/mosquitto.conf

# 추가:
max_connections 100
max_queued_messages 1000
message_size_limit 10240

# 2. InfluxDB 데이터 보존 정책
# 3개월 이상 데이터 자동 삭제
influx bucket update \
  --name sensors \
  --retention 2160h  # 90 days

# 3. Node-RED 메모리 제한
sudo systemctl edit nodered.service

# 추가:
[Service]
Environment="NODE_OPTIONS=--max-old-space-size=512"

# 4. 로그 로테이션
sudo nano /etc/logrotate.d/mosquitto

# 내용:
/var/log/mosquitto/*.log {
    daily
    rotate 7
    compress
    missingok
}
```

---

## 8. 보안 고려사항

### 8.1 기본 보안

```bash
# 1. Mosquitto 인증 활성화
sudo mosquitto_passwd -c /etc/mosquitto/passwd smartfarm
# 비밀번호 입력

sudo nano /etc/mosquitto/mosquitto.conf
# 수정:
allow_anonymous false
password_file /etc/mosquitto/passwd

sudo systemctl restart mosquitto

# 2. ESP32 코드에 인증 추가
mqtt.connect(clientId.c_str(), "smartfarm", "your_password")

# 3. Node-RED 접근 제한
cd ~/.node-red
npm install node-red-admin

node-red-admin hash-pw
# 비밀번호 입력 → 해시 복사

nano settings.js
# 수정:
adminAuth: {
    type: "credentials",
    users: [{
        username: "admin",
        password: "[복사한 해시]",
        permissions: "*"
    }]
}

# 4. 방화벽 설정
sudo apt install ufw
sudo ufw allow 22    # SSH
sudo ufw allow 1883  # MQTT (내부망만)
sudo ufw allow 1880  # Node-RED (내부망만)
sudo ufw enable
```

### 8.2 TLS/SSL 설정 (고급)

```bash
# MQTTS (포트 8883) 설정
# Let's Encrypt 인증서 사용

sudo apt install certbot
sudo certbot certonly --standalone -d yourdomain.com

sudo nano /etc/mosquitto/mosquitto.conf
# 추가:
listener 8883
cafile /etc/letsencrypt/live/yourdomain.com/chain.pem
certfile /etc/letsencrypt/live/yourdomain.com/cert.pem
keyfile /etc/letsencrypt/live/yourdomain.com/privkey.pem

sudo systemctl restart mosquitto
```

---

## 9. 참고 자료

### 9.1 공식 문서
```
라즈베리파이:
https://www.raspberrypi.com/documentation/

ESP32:
https://docs.espressif.com/

Mosquitto:
https://mosquitto.org/documentation/

Node-RED:
https://nodered.org/docs/

InfluxDB:
https://docs.influxdata.com/
```

### 9.2 커뮤니티
```
한국 라즈베리파이 사용자 모임:
https://cafe.naver.com/raspigamer

아두이노/ESP32 포럼:
https://forum.arduino.cc/

MQTT 한국 사용자:
(Facebook 그룹)

스마트팜 코리아:
https://smartfarmkorea.net/
```

### 9.3 추천 YouTube 채널
```
- 공대남자 (한글)
- Andreas Spiess (영문)
- DroneBot Workshop (영문)
- The Hook Up (영문)
```

---

## 10. 결론

### 10.1 핵심 정리

```
✅ 이 시스템의 장점:

1. 비용 효율
   - 초기: 103만원 (상용 대비 93% 절감)
   - 운영: 연 25만원
   - 5년: 228만원 (상용 대비 87% 절감)

2. 확장성
   - Zone 추가: 7만원, 2.5시간
   - 센서 추가: 자유롭게
   - 기능 확장: 무제한

3. 유연성
   - 오픈소스 기반
   - 맞춤 제작 가능
   - 다양한 센서 지원

4. 학습 가치
   - IoT 전반 이해
   - 프로그래밍 실력 향상
   - 문제 해결 능력

5. 독립성
   - 외부 의존 없음
   - 라이선스 문제 없음
   - 장기 유지 가능
```

### 10.2 성공을 위한 팁

```
🎯 단계별로 진행하세요:
   - 한 번에 모두 구축 ×
   - 1-2개 Zone부터 시작 ○
   - 안정화 후 확장 ○

📚 학습에 투자하세요:
   - 공식 문서 읽기
   - 예제 코드 실습
   - 커뮤니티 활용

🔧 문제 해결 능력:
   - 시리얼 모니터 활용
   - 로그 분석
   - 체계적 디버깅

💾 백업의 중요성:
   - 코드 버전 관리 (Git)
   - SD카드 정기 백업
   - 설정 문서화

🤝 도움 요청하기:
   - 커뮤니티 질문
   - 전문가 컨설팅
   - 동료 농가 협력
```

### 10.3 최종 권장사항

```
이 시스템을 추천하는 경우:
✓ 예산이 제한적
✓ 기술에 관심 있음
✓ DIY 정신
✓ 시간 투자 가능
✓ 장기적 관점

다른 방법을 고려할 경우:
⚠ 기술적 지식 전무
⚠ 시간 없음
⚠ 즉시 사용 필요
⚠ 대규모 (1,000평 이상)
⚠ 전문 A/S 필요

하이브리드 접근:
◎ 작은 규모로 시작
◎ 성공 후 확장
◎ 필요시 전문가 지원
```

---

**이 가이드를 통해 저렴하고 확장 가능한 스마트팜 IoT 시스템을 구축하실 수 있습니다!**

**초기 투자: 103만원으로 500평 스마트팜 IoT 완성** ✓✓✓

**궁금한 점이 있으면 언제든지 질문해주세요!**

---

작성일: 2025년 10월 19일
버전: 1.0
