# 500평 규모 스마트팜 시스템 구축 계획서

## 1. 프로젝트 개요

### 1.1 프로젝트 목표
- 500평(약 1,650m²) 규모 온실에서 상추 등 엽채류 재배 자동화
- WiFi 기반 무선 네트워크 구축
- 라즈베리파이 중앙 서버를 통한 통합 관리
- MQTT 프로토콜 기반 IoT 센서/액추에이터 제어
- 원격 모니터링 및 실시간 제어 시스템 구현


### 1.2 기대 효과
- 노동력 30% 절감
- 수확량 20% 증가
- 에너지 효율 15% 개선
- 실시간 데이터 기반 의사결정

---

## 2. 시스템 아키텍처

### 2.1 전체 구조
```
[센서/액추에이터 레이어]
    ↓ WiFi + MQTT
[통신 레이어: ESP32/ESP8266]
    ↓ MQTT
[중앙 서버: 라즈베리파이 4B]
  - MQTT Broker (Mosquitto)
  - 데이터베이스 (SQLite)
  - 제어 로직 (Python)
    ↓
[사용자 인터페이스]
  - 웹 대시보드 (Flask)
  - 모바일 앱
```

### 2.2 통신 프로토콜
- **MQTT 버전**: 3.1.1
- **QoS 레벨**: QoS 1 (최소 1회 전달 보장)
- **토픽 구조**:
  - `smartfarm/zone1/sensor/temperature`
  - `smartfarm/zone1/actuator/irrigation`
  - `smartfarm/zone1/status`

---

## 3. 하드웨어 구성

### 3.1 중앙 서버
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 라즈베리파이 4B | 8GB RAM | 1 | 120,000 | 120,000 |
| MicroSD 카드 | 128GB Class 10 | 1 | 30,000 | 30,000 |
| 전원 어댑터 | 5V 3A | 1 | 15,000 | 15,000 |
| 케이스 | 쿨링팬 포함 | 1 | 20,000 | 20,000 |

### 3.2 센서 노드 (Zone당 1세트, 총 4개 Zone)
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| ESP32 개발보드 | WiFi/Bluetooth | 4 | 15,000 | 60,000 |
| DHT22 센서 | 온습도 센서 | 4 | 8,000 | 32,000 |
| 토양수분 센서 | 정전용량식 | 8 | 5,000 | 40,000 |
| 조도 센서 | BH1750 | 4 | 3,000 | 12,000 |
| CO2 센서 | MH-Z19B | 2 | 45,000 | 90,000 |
| pH 센서 | 아날로그 출력 | 2 | 35,000 | 70,000 |
| EC 센서 | 전기전도도 | 2 | 40,000 | 80,000 |

### 3.3 액추에이터
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 솔레노이드 밸브 | 12V DC | 8 | 25,000 | 200,000 |
| 릴레이 모듈 | 4채널 | 4 | 12,000 | 48,000 |
| 환기팬 | 12V DC | 4 | 35,000 | 140,000 |
| LED 조명 | 식물생장용 | 10 | 80,000 | 800,000 |
| 차광막 모터 | 스텝모터 | 2 | 150,000 | 300,000 |

### 3.4 네트워크 장비
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| WiFi 공유기 | 기가비트, 5GHz | 2 | 120,000 | 240,000 |
| PoE 스위치 | 8포트 | 1 | 180,000 | 180,000 |
| LAN 케이블 | Cat6, 100m | 2 | 50,000 | 100,000 |

### 3.5 전원 및 기타
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| UPS | 1000VA | 1 | 250,000 | 250,000 |
| 배전반 | 누전차단기 포함 | 1 | 150,000 | 150,000 |
| 방수 박스 | IP65 | 6 | 30,000 | 180,000 |

**하드웨어 총 비용: 약 3,157,000원**

---

## 4. 소프트웨어 구성

### 4.1 라즈베리파이 서버
- **OS**: Raspberry Pi OS (64-bit)
- **MQTT Broker**: Mosquitto
- **데이터베이스**: SQLite
- **백엔드**: Python 3.9+
  - paho-mqtt
  - Flask
  - SQLAlchemy
- **웹 서버**: Nginx
- **프로세스 관리**: systemd

### 4.2 ESP32 펌웨어
- **개발환경**: Arduino IDE / PlatformIO
- **라이브러리**:
  - WiFi.h
  - PubSubClient (MQTT)
  - DHT sensor library
  - Adafruit sensor libraries

### 4.3 사용자 인터페이스
- **웹 대시보드**: 
  - Flask (백엔드)
  - Chart.js (데이터 시각화)
  - Bootstrap 5 (UI)
- **모바일 앱**: React Native (선택사항)

### 4.4 MQTT 토픽 설계
```
smartfarm/
  ├── zone1/
  │   ├── sensor/
  │   │   ├── temperature
  │   │   ├── humidity
  │   │   ├── soil_moisture_1
  │   │   ├── soil_moisture_2
  │   │   ├── light
  │   │   └── co2
  │   ├── actuator/
  │   │   ├── irrigation_1/set
  │   │   ├── irrigation_2/set
  │   │   ├── fan/set
  │   │   └── light/set
  │   └── status
  ├── zone2/
  │   └── ...
  ├── zone3/
  │   └── ...
  └── zone4/
      └── ...
```

---

## 5. 네트워크 설계

### 5.1 WiFi 커버리지
- 온실 크기: 약 50m x 33m
- WiFi AP 배치: 2개 (양 끝단)
- 주파수: 2.4GHz (장애물 투과력 우수)
- 채널: 자동 선택 (간섭 최소화)

### 5.2 IP 주소 할당
- **라즈베리파이**: 192.168.1.100 (고정 IP)
- **ESP32 노드**: 192.168.1.101-120 (DHCP 예약)
- **공유기**: 192.168.1.1
- **서브넷 마스크**: 255.255.255.0

### 5.3 보안 설정
- **WiFi 암호화**: WPA2-PSK (AES)
- **MQTT 인증**: Username/Password
- **방화벽**: 필요한 포트만 개방 (1883, 8883, 80, 443)
- **VPN**: 외부 접속시 OpenVPN 사용

---

## 6. 제어 로직 및 자동화

### 6.1 상추 재배 최적 환경
- **온도**: 주간 18-22°C, 야간 12-16°C
- **습도**: 60-70%
- **광량**: 200-300 μmol/m²/s (10-12시간/일)
- **CO2 농도**: 800-1200 ppm
- **토양 수분**: 60-70% (정전용량식 센서 기준)
- **pH**: 5.5-6.5
- **EC**: 1.2-1.8 mS/cm

### 6.2 자동 제어 알고리즘

#### 온도 제어
```python
if temperature > 24:
    turn_on(ventilation_fan)
    open_shade(50%)
elif temperature < 16:
    close_shade(100%)
    turn_on(heating)  # if available
```

#### 관수 제어
```python
if soil_moisture < 60%:
    turn_on(irrigation)
    wait(300)  # 5분
    turn_off(irrigation)
elif soil_moisture > 80%:
    alert("과습 경고")
```

#### 광 제어
```python
if light_level < 200 and is_daytime():
    turn_on(led_grow_light)
elif light_level > 400:
    turn_off(led_grow_light)
```

#### CO2 제어
```python
if co2_level < 800:
    alert("CO2 부족")
    # CO2 발생기 가동 (별도 장비 필요)
```

### 6.3 알림 시스템
- 이메일 알림 (SMTP)
- Telegram Bot 알림
- 대시보드 팝업 알림

---

## 7. 데이터베이스 스키마

### 7.1 센서 데이터 테이블
```sql
CREATE TABLE sensor_data (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    zone VARCHAR(10),
    sensor_type VARCHAR(50),
    value REAL,
    unit VARCHAR(10)
);

CREATE INDEX idx_timestamp ON sensor_data(timestamp);
CREATE INDEX idx_zone_sensor ON sensor_data(zone, sensor_type);
```

### 7.2 액추에이터 로그 테이블
```sql
CREATE TABLE actuator_log (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    zone VARCHAR(10),
    actuator_name VARCHAR(50),
    action VARCHAR(20),
    duration INTEGER
);
```

### 7.3 알림 기록 테이블
```sql
CREATE TABLE alerts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    alert_type VARCHAR(50),
    message TEXT,
    resolved BOOLEAN DEFAULT 0
);
```

---

## 8. 구축 일정

### Phase 1: 준비 및 설계 (2주)
- [x] 요구사항 분석
- [x] 시스템 설계
- [ ] 하드웨어 구매
- [ ] 네트워크 설계

### Phase 2: 인프라 구축 (2주)
- [ ] 네트워크 장비 설치
- [ ] 전원 공급 시스템 구축
- [ ] 라즈베리파이 서버 설정
- [ ] MQTT 브로커 설치 및 설정

### Phase 3: 센서/액추에이터 설치 (3주)
- [ ] ESP32 펌웨어 개발
- [ ] 센서 노드 조립 및 테스트
- [ ] 액추에이터 설치 및 배선
- [ ] 통신 테스트

### Phase 4: 소프트웨어 개발 (3주)
- [ ] 제어 로직 개발
- [ ] 데이터베이스 구축
- [ ] 웹 대시보드 개발
- [ ] 알림 시스템 구현

### Phase 5: 통합 테스트 (2주)
- [ ] 전체 시스템 통합
- [ ] 기능 테스트
- [ ] 안정성 테스트
- [ ] 사용자 교육

### Phase 6: 시범 운영 (4주)
- [ ] 실제 작물 재배 시작
- [ ] 모니터링 및 미세조정
- [ ] 문제점 개선
- [ ] 최종 검수

**총 구축 기간: 약 16주 (4개월)**

---

## 9. 예상 비용

| 항목 | 금액(원) |
|------|----------|
| 하드웨어 | 3,157,000 |
| 소프트웨어 라이선스 | 0 (오픈소스) |
| 설치 인건비 | 2,000,000 |
| 예비비 (10%) | 515,700 |
| **총 예상 비용** | **5,672,700** |

---

## 10. 유지보수 계획

### 10.1 정기 점검 (월 1회)
- 센서 캘리브레이션
- 액추에이터 동작 확인
- 네트워크 상태 점검
- 배터리 백업 확인

### 10.2 예방 정비 (분기 1회)
- 필터 청소
- 배선 연결 상태 확인
- 소프트웨어 업데이트
- 데이터베이스 최적화

### 10.3 예상 유지보수 비용
- 연간 소모품 교체: 약 500,000원
- 센서 교체 (2년마다): 약 300,000원
- 전문가 점검 (분기): 약 200,000원

**연간 유지보수 비용: 약 1,300,000원**

---

## 11. 확장 계획

### 11.1 단기 확장 (1년 이내)
- 기상 관측 시스템 추가
- AI 기반 병충해 감지 카메라
- 자동 수확 로봇 도입 검토

### 11.2 중장기 확장 (2-3년)
- 다단식 재배 시스템으로 확장
- 수경재배 시스템 통합
- 에너지 관리 시스템 (태양광 연계)

---

## 12. 리스크 관리

| 리스크 | 확률 | 영향도 | 대응 방안 |
|--------|------|--------|-----------|
| 네트워크 장애 | 중 | 높음 | 이중화, 로컬 제어 모드 |
| 전원 차단 | 중 | 높음 | UPS 설치, 알림 시스템 |
| 센서 고장 | 높음 | 중 | 예비 센서 확보, 빠른 교체 |
| 해킹/보안 침해 | 낮음 | 높음 | VPN, 암호화, 접근 제어 |
| 소프트웨어 버그 | 중 | 중 | 철저한 테스트, 백업 |

---

## 13. 핵심 기술 구현 예시

### 13.1 ESP32 센서 노드 코드 (예시)
```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

const char* ssid = "SmartFarm_WiFi";
const char* password = "your_password";
const char* mqtt_server = "192.168.1.100";

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  dht.begin();
  setup_wifi();
  client.setServer(mqtt_server, 1883);
}

void setup_wifi() {
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
}

void reconnect() {
  while (!client.connected()) {
    if (client.connect("ESP32_Zone1")) {
      // Connected
    } else {
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
  
  float temp = dht.readTemperature();
  float humid = dht.readHumidity();
  
  if (!isnan(temp) && !isnan(humid)) {
    char tempStr[8];
    char humidStr[8];
    dtostrf(temp, 1, 2, tempStr);
    dtostrf(humid, 1, 2, humidStr);
    
    client.publish("smartfarm/zone1/sensor/temperature", tempStr);
    client.publish("smartfarm/zone1/sensor/humidity", humidStr);
  }
  
  delay(60000); // 1분마다 전송
}
```

### 13.2 라즈베리파이 제어 로직 (Python 예시)
```python
import paho.mqtt.client as mqtt
import sqlite3
from datetime import datetime

BROKER = "localhost"
PORT = 1883

def on_connect(client, userdata, flags, rc):
    print(f"Connected with result code {rc}")
    client.subscribe("smartfarm/+/sensor/#")

def on_message(client, userdata, msg):
    topic = msg.topic
    value = float(msg.payload.decode())
    
    # 데이터베이스 저장
    save_to_db(topic, value)
    
    # 제어 로직
    if "temperature" in topic:
        control_temperature(value)
    elif "soil_moisture" in topic:
        control_irrigation(value)

def save_to_db(topic, value):
    conn = sqlite3.connect('smartfarm.db')
    cursor = conn.cursor()
    
    parts = topic.split('/')
    zone = parts[1]
    sensor_type = parts[3]
    
    cursor.execute('''
        INSERT INTO sensor_data (zone, sensor_type, value)
        VALUES (?, ?, ?)
    ''', (zone, sensor_type, value))
    
    conn.commit()
    conn.close()

def control_temperature(temp):
    if temp > 24:
        # 환기팬 가동
        client.publish("smartfarm/zone1/actuator/fan/set", "ON")
    elif temp < 16:
        # 환기팬 정지
        client.publish("smartfarm/zone1/actuator/fan/set", "OFF")

def control_irrigation(moisture):
    if moisture < 60:
        # 관수 시작
        client.publish("smartfarm/zone1/actuator/irrigation_1/set", "ON")

client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message

client.connect(BROKER, PORT, 60)
client.loop_forever()
```

---

## 14. 결론

본 계획서는 500평 규모의 상추 재배 온실을 위한 스마트팜 시스템의 전체적인 설계를 제시합니다. WiFi 기반 무선 네트워크와 MQTT 프로토콜을 활용하여 안정적이고 확장 가능한 시스템을 구축할 수 있으며, 약 570만원의 합리적인 비용으로 구현 가능합니다.

핵심 성공 요인:
1. 안정적인 네트워크 인프라
2. 센서 데이터의 정확한 수집 및 분석
3. 실시간 제어 로직의 최적화
4. 사용자 친화적인 인터페이스
5. 지속적인 모니터링과 개선

이 시스템을 통해 노동력 절감, 생산성 향상, 자원 효율화를 동시에 달성할 수 있을 것으로 기대됩니다.
