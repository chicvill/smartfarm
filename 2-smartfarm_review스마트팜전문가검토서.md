# 스마트팜 시스템 구축 계획서 전문 검토 보고서

## 검토 개요
- **검토 대상**: 500평 규모 스마트팜 시스템 구축 계획서
- **검토 일자**: 2025년 10월 18일
- **검토자**: Claude (AI 시스템 아키텍트)
- **종합 평가**: ⭐⭐⭐⭐☆ (4/5) - 전반적으로 실현 가능한 계획이나 개선 필요

---

## 1. 시스템 아키텍처 검토

### ✅ 강점
- MQTT 기반 pub/sub 패턴은 IoT 시스템에 적합한 선택
- 라즈베리파이를 중앙 서버로 사용하는 것은 비용 대비 효율적
- 계층적 토픽 구조는 확장성과 관리성이 우수

### ⚠️ 개선 필요 사항

#### 1.1 단일 장애점(SPOF) 문제
**문제점**: 라즈베리파이 1대만으로 전체 시스템 운영
```
현재 구조:
[센서] → [라즈베리파이] → [사용자]
         (단일 장애점)

권장 구조:
[센서] → [Main 라즈베리파이] ⇄ [Backup 라즈베리파이] → [사용자]
         (failover 지원)
```

**개선안**:
- 라즈베리파이 2대 구성 (Active-Standby)
- Heartbeat 모니터링 구현
- 자동 failover 메커니즘

#### 1.2 로컬 자율 제어 부재
**문제점**: 중앙 서버 장애 시 모든 제어 중단

**개선안**:
```cpp
// ESP32에 로컬 제어 로직 추가
void local_control_mode() {
  if (!mqtt_connected && millis() - last_mqtt_time > 300000) {
    // 5분간 MQTT 연결 안되면 로컬 모드
    if (temperature > 25) {
      digitalWrite(FAN_PIN, HIGH);
    }
    if (soil_moisture < 60) {
      digitalWrite(VALVE_PIN, HIGH);
      delay(300000);  // 5분
      digitalWrite(VALVE_PIN, LOW);
    }
  }
}
```

#### 1.3 데이터베이스 선택의 문제
**문제점**: SQLite는 시계열 데이터에 최적화되어 있지 않음

**데이터 특성 분석**:
- 센서 노드 16개 × 측정 항목 7개 = 112개 데이터 스트림
- 1분 간격 수집 = 분당 112개 레코드
- 일간 약 161,280개 레코드
- 연간 약 5,900만 개 레코드

**권장 대안**:
1. **InfluxDB** (권장)
   - 시계열 데이터 전문
   - 압축률 우수 (10:1)
   - 내장 다운샘플링
   
2. **TimescaleDB**
   - PostgreSQL 기반 (익숙한 SQL)
   - 자동 파티셔닝
   - 우수한 압축

```python
# InfluxDB 사용 예시
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS

client = InfluxDBClient(url="http://localhost:8086", token="my-token", org="smartfarm")
write_api = client.write_api(write_options=SYNCHRONOUS)

point = Point("sensor_data") \
    .tag("zone", "zone1") \
    .tag("sensor", "temperature") \
    .field("value", 22.5) \
    .time(datetime.utcnow())

write_api.write(bucket="smartfarm", record=point)
```

---

## 2. 하드웨어 구성 검토

### ✅ 강점
- 센서 선택이 적절함 (DHT22, MH-Z19B 등)
- 방수 박스(IP65) 고려는 우수
- UPS 백업 포함

### ⚠️ 개선 필요 사항

#### 2.1 센서 배치 밀도 문제
**문제점**: 500평(1,650m²)을 4개 존으로만 분할
- 1 존당 약 125평(412.5m²) → 약 20m × 20m
- DHT22의 정확한 측정 범위: 반경 5m 내외

**권장 존 분할**:
```
현재: 4개 존 (125평/존)
권장: 8-10개 존 (50-65평/존)

이유:
- 온실 내 미기후(microclimate) 차이 반영
- 구역별 세밀한 제어 가능
- 센서 고장시 영향 범위 축소
```

#### 2.2 토양 센서 부족
**문제점**: Zone당 토양수분 센서 2개만 설치
- 125평 면적에 2개 센서 = 62.5평당 1개
- 관수 편차 발생 가능

**권장 수량**:
- Zone당 4-6개 센서
- 관수 구역별 1개씩 배치

#### 2.3 네트워크 장비의 신뢰성
**문제점**: 일반 소비자용 공유기 사용

**농업 환경 특성**:
- 고온다습 (여름 40°C+, 습도 80%+)
- 분진 및 부식성 가스 (NH3, H2S)
- 24/7 연속 가동

**권장 사양**:
```
소비자용 → 산업용으로 변경

산업용 WiFi AP:
- Ubiquiti UniFi AP AC LR
- IP67 등급 (방수/방진)
- PoE 지원
- 온도 범위: -30°C ~ +70°C
- 가격: 약 250,000원/대

추가 비용: +260,000원 (2대 기준)
```

#### 2.4 센서 캘리브레이션 고려 부재
**문제점**: pH, EC 센서는 정기 캘리브레이션 필수

**권장 사항**:
- pH 7.0, 4.0 표준 용액 구비
- EC 1.413 mS/cm 표준 용액 구비
- 월 1회 캘리브레이션 일정
- 캘리브레이션 로그 기록

---

## 3. 네트워크 설계 검토

### ✅ 강점
- 2.4GHz 선택 적절 (장애물 투과력)
- 고정 IP 할당 계획
- VPN 보안 고려

### ⚠️ 심각한 문제

#### 3.1 WiFi 커버리지 계산 오류
**문제점**: AP 2대로 50m × 33m 커버 불가능

**실제 계산**:
```
이론적 2.4GHz 도달거리: 실내 100m
실제 온실 환경:
- 금속 골조: 신호 감쇠 30-40%
- 비닐/유리: 신호 감쇠 10-20%
- 식물/토양: 신호 흡수 20-30%
→ 실효 거리: 약 30-40m

50m × 33m 온실 커버리지:
- 필요 AP 수: 최소 3-4대
- 권장 배치: 직사각형 모서리 + 중앙
```

**권장 배치도**:
```
[온실 평면도: 50m × 33m]

AP1 -------- AP2 -------- AP3
 |            |            |
 |            |            |
 |           AP4           |
 |            |            |
 |            |            |
센서노드    센서노드      센서노드
```

#### 3.2 MQTT QoS 레벨 부적절
**문제점**: 모든 메시지에 QoS 1 사용

**권장 QoS 정책**:
```
센서 데이터 (주기적):
→ QoS 0 (최대 1회 전달, 빠름)
이유: 1분 후 새 데이터가 오므로 유실 허용

제어 명령 (비주기적):
→ QoS 2 (정확히 1회 전달, 느림)
이유: 밸브 개폐 등 중복 실행 방지

알림 메시지:
→ QoS 1 (최소 1회 전달, 중간)
이유: 중복 알림은 허용, 유실 방지
```

#### 3.3 MQTT 보안 취약
**문제점**: TLS 암호화 미적용

**권장 보안 설정**:
```bash
# Mosquitto 설정 (/etc/mosquitto/mosquitto.conf)
listener 8883
cafile /etc/mosquitto/ca_certificates/ca.crt
certfile /etc/mosquitto/certs/server.crt
keyfile /etc/mosquitto/certs/server.key
require_certificate true

# 사용자 인증
password_file /etc/mosquitto/passwd
allow_anonymous false
```

---

## 4. 소프트웨어 구성 검토

### ✅ 강점
- Python 기반 개발은 생산성이 높음
- Flask 웹 프레임워크 적절
- systemd 프로세스 관리 우수

### ⚠️ 개선 필요 사항

#### 4.1 실시간 처리 성능 문제
**문제점**: 파이썬의 GIL(Global Interpreter Lock)

**시나리오**:
```
112개 센서 × 1분 간격 = 초당 약 2개 메시지
→ 현재 부하는 낮음

그러나 확장시 (1초 간격):
112개 센서 × 1초 간격 = 초당 112개 메시지
+ 제어 명령 20개/초
+ 대시보드 요청 10개/초
= 초당 약 140개 이벤트
→ 파이썬 단일 스레드로 처리 한계
```

**권장 아키텍처**:
```python
# 현재: 단일 프로세스
mqtt_client.loop_forever()

# 권장: 멀티프로세싱
from multiprocessing import Process, Queue

def sensor_processor(queue):
    # 센서 데이터 처리 전용
    pass

def control_processor(queue):
    # 제어 로직 전용
    pass

def web_server():
    # 웹 서버 전용
    pass

if __name__ == "__main__":
    q = Queue()
    Process(target=sensor_processor, args=(q,)).start()
    Process(target=control_processor, args=(q,)).start()
    Process(target=web_server).start()
```

#### 4.2 에러 처리 및 로깅 부재
**문제점**: 제공된 코드에 예외 처리 없음

**권장 개선**:
```python
import logging
from logging.handlers import RotatingFileHandler

# 로그 설정
logger = logging.getLogger('smartfarm')
logger.setLevel(logging.INFO)

handler = RotatingFileHandler(
    'smartfarm.log',
    maxBytes=10*1024*1024,  # 10MB
    backupCount=5
)
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
handler.setFormatter(formatter)
logger.addHandler(handler)

# 에러 처리 예시
def on_message(client, userdata, msg):
    try:
        topic = msg.topic
        value = float(msg.payload.decode())
        save_to_db(topic, value)
        logger.info(f"Saved: {topic} = {value}")
    except ValueError as e:
        logger.error(f"Invalid value: {msg.payload}, Error: {e}")
    except Exception as e:
        logger.critical(f"Unexpected error: {e}", exc_info=True)
```

#### 4.3 데이터 검증(Validation) 누락
**문제점**: 센서 이상값 필터링 없음

**권장 검증 로직**:
```python
def validate_sensor_data(sensor_type, value):
    """센서 데이터 유효성 검증"""
    ranges = {
        'temperature': (-10, 50),      # °C
        'humidity': (0, 100),          # %
        'soil_moisture': (0, 100),     # %
        'light': (0, 2000),            # lux
        'co2': (300, 5000),            # ppm
        'ph': (0, 14),                 # pH
        'ec': (0, 10)                  # mS/cm
    }
    
    if sensor_type not in ranges:
        return False
    
    min_val, max_val = ranges[sensor_type]
    
    if not (min_val <= value <= max_val):
        logger.warning(
            f"Out of range: {sensor_type}={value}, "
            f"expected [{min_val}, {max_val}]"
        )
        return False
    
    return True

def on_message(client, userdata, msg):
    try:
        topic = msg.topic
        sensor_type = topic.split('/')[-1]
        value = float(msg.payload.decode())
        
        if validate_sensor_data(sensor_type, value):
            save_to_db(topic, value)
        else:
            # 이상값 기록
            log_anomaly(topic, value)
    except Exception as e:
        logger.error(f"Error: {e}")
```

---

## 5. 제어 로직 검토

### ✅ 강점
- 상추 최적 재배 환경 수치는 정확
- 기본적인 임계값 기반 제어 포함

### ⚠️ 심각한 문제

#### 5.1 단순 ON/OFF 제어의 한계
**문제점**: 히스테리시스(Hysteresis) 미적용

```python
# 현재 코드의 문제
if temperature > 24:
    turn_on(fan)  # 24.1°C에서 켜짐
elif temperature < 16:
    turn_off(fan)  # 23.9°C에서 꺼짐 → 다시 24.1°C → 켜짐
# → 빈번한 ON/OFF → 액추에이터 수명 단축
```

**권장 개선 (히스테리시스)**:
```python
class TemperatureController:
    def __init__(self):
        self.fan_state = False
        self.target_temp = 20
        self.hysteresis = 2  # ±2°C
    
    def control(self, current_temp):
        if current_temp > self.target_temp + self.hysteresis:
            # 22°C 초과시 켜기
            if not self.fan_state:
                self.turn_on_fan()
                self.fan_state = True
        elif current_temp < self.target_temp - self.hysteresis:
            # 18°C 미만시 끄기
            if self.fan_state:
                self.turn_off_fan()
                self.fan_state = False
```

#### 5.2 PID 제어 부재
**문제점**: 정밀한 환경 제어 불가

**권장 PID 컨트롤러**:
```python
class PIDController:
    def __init__(self, kp, ki, kd, setpoint):
        self.kp = kp  # 비례 게인
        self.ki = ki  # 적분 게인
        self.kd = kd  # 미분 게인
        self.setpoint = setpoint
        self.integral = 0
        self.prev_error = 0
    
    def compute(self, measured_value, dt):
        error = self.setpoint - measured_value
        
        # 비례항
        p = self.kp * error
        
        # 적분항
        self.integral += error * dt
        i = self.ki * self.integral
        
        # 미분항
        derivative = (error - self.prev_error) / dt
        d = self.kd * derivative
        
        # PID 출력
        output = p + i + d
        
        self.prev_error = error
        return output

# 사용 예시
temp_controller = PIDController(
    kp=5.0,    # 튜닝 필요
    ki=0.1,
    kd=1.0,
    setpoint=20.0
)

def control_temperature(current_temp):
    output = temp_controller.compute(current_temp, dt=60)
    
    # 출력을 PWM 신호로 변환 (0-100%)
    fan_speed = max(0, min(100, output))
    set_fan_pwm(fan_speed)
```

#### 5.3 VPD (증기압 차) 제어 미적용
**문제점**: 온도와 습도를 독립적으로 제어

**VPD 중요성**:
- VPD = 포화증기압 - 실제증기압
- 식물 증산작용의 핵심 지표
- 상추 최적 VPD: 0.8-1.2 kPa

**VPD 기반 제어**:
```python
import math

def calculate_vpd(temperature, humidity):
    """VPD 계산 (kPa)"""
    # 포화증기압 계산 (Tetens 공식)
    svp = 0.6108 * math.exp(17.27 * temperature / (temperature + 237.3))
    
    # 실제증기압
    avp = svp * (humidity / 100)
    
    # VPD
    vpd = svp - avp
    return vpd

def control_environment(temp, humidity):
    vpd = calculate_vpd(temp, humidity)
    
    if vpd < 0.8:  # VPD 너무 낮음 (과습)
        # 온도 올리거나 습도 낮추기
        increase_ventilation()
        reduce_misting()
    elif vpd > 1.2:  # VPD 너무 높음 (건조)
        # 온도 낮추거나 습도 높이기
        increase_misting()
        reduce_ventilation()
    else:
        # 최적 범위 유지
        maintain_current()
```

#### 5.4 DLI (일일 광적산량) 관리 부재
**문제점**: 단순 광량 임계값만 체크

**DLI 계산 및 제어**:
```python
class DLIController:
    def __init__(self, target_dli=15):  # mol/m²/day
        self.target_dli = target_dli
        self.accumulated_ppfd = 0  # μmol/m²
        self.last_measurement_time = None
    
    def update(self, current_ppfd, current_time):
        """현재 광량으로 DLI 업데이트"""
        if self.last_measurement_time:
            dt = (current_time - self.last_measurement_time).total_seconds()
            # μmol/m²/s × seconds = μmol/m²
            self.accumulated_ppfd += current_ppfd * dt
        
        self.last_measurement_time = current_time
        
        # μmol/m² → mol/m²/day
        current_dli = self.accumulated_ppfd / 1_000_000
        
        return current_dli
    
    def should_supplement_light(self, current_dli, remaining_hours):
        """보광 필요 여부 판단"""
        required_dli = self.target_dli - current_dli
        
        if required_dli > 0:
            # 남은 시간 동안 필요한 PPFD 계산
            required_ppfd = (required_dli * 1_000_000) / (remaining_hours * 3600)
            return True, required_ppfd
        return False, 0

# 사용 예시
dli_controller = DLIController(target_dli=15)

def light_control_loop():
    current_ppfd = read_light_sensor()
    current_time = datetime.now()
    current_dli = dli_controller.update(current_ppfd, current_time)
    
    # 오후 5시까지 남은 시간
    remaining_hours = (17 - current_time.hour)
    
    need_light, required_ppfd = dli_controller.should_supplement_light(
        current_dli, remaining_hours
    )
    
    if need_light:
        # LED 조명 제어
        set_led_intensity(required_ppfd)
```

---

## 6. 비용 분석 검토

### ⚠️ 주요 문제점

#### 6.1 과소 산정된 비용
**문제점**: 실제 필요한 항목 누락

**재산정 비용**:
```
[추가 필요 항목]
1. 라즈베리파이 백업 서버:      185,000원
2. 추가 WiFi AP (2대):          500,000원
3. 추가 센서 (존 확장):          800,000원
4. 산업용 릴레이 보드:          200,000원
5. 예비 센서 (20%):             268,000원
6. 캘리브레이션 용액:            50,000원
7. 케이블 및 커넥터:            300,000원
8. 테스트 장비 (멀티미터 등):   150,000원

소계:                        2,453,000원

기존 계획:                    5,672,700원
추가 비용:                    2,453,000원
───────────────────────────────────────
재산정 총액:                  8,125,700원
```

#### 6.2 유지보수 비용 과소 산정
**문제점**: 소모품 교체 주기 비현실적

**현실적인 교체 주기**:
```
센서 수명:
- DHT22:            2년
- 토양수분 센서:     1년 (부식)
- pH/EC 센서:       1년 (전극 열화)
- CO2 센서:         5년

연간 교체 비용:
- 토양수분: 8개 × 5,000 = 40,000원
- pH/EC: 2세트 × 75,000 = 150,000원
- DHT22 (50%): 2개 × 8,000 = 16,000원
───────────────────────────────────
소계:                      206,000원

기타 유지보수:
- 필터 교체:              100,000원
- 밸브 O-ring:             50,000원
- 전문가 점검 (분기):      800,000원
───────────────────────────────────
연간 유지보수 총액:      1,156,000원

기존 산정: 1,300,000원 → 적정 범위
```

---

## 7. 보안 검토

### ⚠️ 심각한 취약점

#### 7.1 인증 체계 부재
**문제점**: MQTT 인증만으로는 불충분

**권장 보안 계층**:
```
1. 네트워크 계층:
   - WPA3-Enterprise (개인용 PSK 대신)
   - MAC 주소 필터링
   - VLAN 분리 (IoT / Management)

2. 전송 계층:
   - TLS 1.3 암호화
   - 인증서 기반 상호 인증

3. 애플리케이션 계층:
   - OAuth 2.0 / JWT 토큰
   - RBAC (역할 기반 접근 제어)
   - API Rate Limiting

4. 물리 계층:
   - 서버실 잠금
   - 감시 카메라
```

#### 7.2 펌웨어 업데이트 보안
**문제점**: OTA 업데이트 암호화 미언급

**권장 OTA 구현**:
```cpp
#include <Update.h>
#include <WiFiClientSecure.h>

void performOTA(const char* url) {
    WiFiClientSecure client;
    client.setCACert(ca_cert);  // 인증서 검증
    
    HTTPClient http;
    http.begin(client, url);
    
    int httpCode = http.GET();
    if (httpCode == HTTP_CODE_OK) {
        int contentLength = http.getSize();
        
        // 서명 검증
        if (!verifyFirmwareSignature(http.getStream())) {
            Serial.println("Invalid firmware signature!");
            return;
        }
        
        // 업데이트 실행
        bool canBegin = Update.begin(contentLength);
        if (canBegin) {
            Update.writeStream(http.getStream());
            if (Update.end()) {
                Serial.println("OTA Success!");
                ESP.restart();
            }
        }
    }
}
```

---

## 8. 확장성 및 미래 대비

### ✅ 강점
- 확장 계획 포함
- AI 및 자동화 고려

### ⚠️ 개선 사항

#### 8.1 데이터 분석 및 AI 준비
**권장 추가 기능**:
```python
# 머신러닝 기반 예측 모델
from sklearn.ensemble import RandomForestRegressor
import pandas as pd

class YieldPredictor:
    def __init__(self):
        self.model = RandomForestRegressor()
    
    def train(self, historical_data):
        """과거 데이터로 학습"""
        features = ['avg_temp', 'avg_humidity', 'total_dli', 'days']
        X = historical_data[features]
        y = historical_data['yield']
        self.model.fit(X, y)
    
    def predict_yield(self, current_conditions):
        """현재 조건으로 수확량 예측"""
        return self.model.predict([current_conditions])[0]

# 이상 탐지
from sklearn.ensemble import IsolationForest

class AnomalyDetector:
    def __init__(self):
        self.detector = IsolationForest(contamination=0.1)
    
    def detect(self, sensor_data):
        """센서 이상치 탐지"""
        predictions = self.detector.fit_predict(sensor_data)
        return predictions == -1  # 이상치 인덱스
```

#### 8.2 에너지 최적화
**권장 추가**:
```python
class EnergyOptimizer:
    def __init__(self, electricity_prices):
        self.prices = electricity_prices  # 시간대별 전기요금
    
    def optimize_lighting_schedule(self, required_dli):
        """전기요금이 저렴한 시간대에 보광 집중"""
        cheap_hours = self.get_cheap_hours()
        
        # 저렴한 시간대에 더 많은 광량 공급
        schedule = {}
        for hour in range(24):
            if hour in cheap_hours:
                schedule[hour] = required_dli * 1.2
            else:
                schedule[hour] = required_dli * 0.8
        
        return schedule
```

---

## 9. 규제 및 표준 준수

### ⚠️ 누락 사항

#### 9.1 전기안전 규정
**필요 사항**:
- 전기기사 또는 전기공사업체 시공
- 접지 공사 (농업용 전기 안전)
- 누전차단기 설치 (30mA, 0.03초)
- 정기 전기안전점검

#### 9.2 통신 규정
**필요 사항**:
- 방송통신기자재 적합성 평가 (KC 인증)
- 전파법 준수 (WiFi 2.4GHz 대역)

#### 9.3 농업 관련 규정
**필요 사항**:
- 스마트팜 설치 신고 (지자체)
- 농업용 전력 신청 (한국전력)
- 시설재배업 등록

---

## 10. 종합 평가 및 권장사항

### 📊 항목별 평가

| 항목 | 점수 | 평가 |
|------|------|------|
| 시스템 아키텍처 | 3.5/5 | 기본은 좋으나 이중화 필요 |
| 하드웨어 구성 | 3/5 | 센서 밀도 부족, 산업용 장비 권장 |
| 네트워크 설계 | 2.5/5 | WiFi 커버리지 계산 오류 |
| 소프트웨어 구성 | 3/5 | 기본 기능 충족, 고급 기능 부족 |
| 제어 로직 | 2.5/5 | 단순 ON/OFF, PID/VPD 미적용 |
| 보안 | 2/5 | 기본 보안만 적용, 강화 필요 |
| 비용 산정 | 3/5 | 일부 항목 누락 |
| 유지보수 계획 | 4/5 | 체계적이나 세부 조정 필요 |
| 확장성 | 4/5 | 우수한 확장 계획 |

**총점: 27.5 / 45 = 61%**

### 🎯 우선순위별 개선 권장사항

#### 우선순위 1 (즉시 적용 필요)
1. **WiFi AP 추가**: 2대 → 4대로 증설
2. **InfluxDB 도입**: SQLite → InfluxDB 변경
3. **센서 존 세분화**: 4개 → 8-10개 존
4. **히스테리시스 적용**: 빈번한 ON/OFF 방지
5. **MQTT TLS 암호화**: 보안 강화

#### 우선순위 2 (구축 중 적용)
1. **라즈베리파이 이중화**: failover 구현
2. **PID 컨트롤러**: 정밀 제어
3. **VPD 기반 제어**: 통합 환경 관리
4. **데이터 검증 로직**: 이상값 필터링
5. **로컬 자율 제어**: ESP32 독립 동작

#### 우선순위 3 (운영 후 개선)
1. **AI 예측 모델**: 수확량 예측
2. **에너지 최적화**: 전기요금 절감
3. **고급 대시보드**: Grafana 도입
4. **모바일 앱**: React Native 개발
5. **OTA 업데이트**: 원격 펌웨어 관리

### 💰 재산정 예산

```
초기 구축 비용:
─────────────────────────────
기존 하드웨어:         3,157,000원
추가 하드웨어:         2,453,000원
설치 인건비:           2,500,000원 (증가)
소프트웨어 개발:       1,500,000원 (신규)
예비비 (15%):          1,441,500원
─────────────────────────────
총 초기 비용:          11,051,500원

연간 운영 비용:
─────────────────────────────
유지보수:              1,156,000원
전기료 (추정):         1,200,000원
통신비:                  240,000원
소모품:                  200,000원
─────────────────────────────
연간 운영 비용:         2,796,000원
```

### 📅 수정된 구축 일정

```
Phase 1: 설계 보완 (3주)
- 네트워크 재설계
- 하드웨어 사양 조정
- 소프트웨어 아키텍처 개선

Phase 2: 인프라 구축 (3주)
- 네트워크 장비 설치 (4개 AP)
- 전원 공급 및 배선
- 서버 이중화 구축

Phase 3: 센서/액추에이터 (4주)
- 8-10개 존으로 세분화
- ESP32 펌웨어 개발 (로컬 제어 포함)
- 통합 테스트

Phase 4: 소프트웨어 개발 (4주)
- InfluxDB 구축
- PID/VPD 제어 로직
- 웹 대시보드 개발

Phase 5: 통합 테스트 (3주)
- 시스템 통합
- 부하 테스트
- 보안 점검

Phase 6: 시범 운영 (4주)
- 실제 재배 시작
- 파라미터 튜닝
- 사용자 교육

총 구축 기간: 21주 (약 5개월)
```

---

## 11. 결론

본 스마트팜 시스템 구축 계획서는 **기본적인 구조는 양호**하나, **실제 구축 및 운영에 필요한 세부 사항**에서 여러 개선이 필요합니다.

### 주요 개선 포인트
1. **네트워크 안정성**: WiFi AP 증설 필수
2. **시스템 안정성**: 서버 이중화 및 로컬 제어
3. **제어 정밀도**: PID, VPD, DLI 기반 고급 제어
4. **데이터 관리**: 시계열 DB 도입
5. **보안 강화**: 다층 보안 체계

### 실현 가능성
개선 사항을 반영할 경우, **기술적으로 충분히 실현 가능**하며, 예상되는 효과(노동력 절감, 수확량 증가)도 달성 가능합니다.

### 최종 권고
- **단계적 구축**: 핵심 기능 먼저 구현 후 고도화
- **전문가 자문**: 농업기술센터 또는 스마트팜 컨설턴트 협력
- **파일럿 운영**: 소규모(100평) 시범 운영 후 확대

이상으로 스마트팜 시스템 구축 계획서에 대한 전문 검토를 마칩니다.
