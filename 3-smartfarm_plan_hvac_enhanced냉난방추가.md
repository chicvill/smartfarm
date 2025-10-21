# 500평 스마트팜 시스템 구축 계획서 (냉난방·통풍 시스템 강화판)

## 추가된 주요 시스템
✅ 냉난방 시스템 (HVAC)
✅ 통합 환기 시스템
✅ 에너지 효율 최적화
✅ 다단계 온도 제어

---

## 1. 냉난방 및 통풍 시스템 개요

### 1.1 시스템 목표
- 사계절 안정적인 온도 유지 (상추 최적 온도: 18-22°C)
- 에너지 효율적인 냉난방
- 자연 환기와 기계식 환기의 통합 제어
- CO2 농도 관리를 위한 통풍 최적화

### 1.2 온실 열부하 계산

#### 난방 부하 (겨울철)
```
온실 규모: 1,650m² (500평)
온실 높이: 4m
체적: 6,600m³

겨울철 조건:
- 외기 온도: -10°C
- 목표 온도: 18°C
- 온도차: 28°C

열손실 계산:
Q = U × A × ΔT

측면/지붕 (비닐 2중):
- U값: 3.5 W/m²·K
- 면적: 약 2,500m²
- 손실: 3.5 × 2,500 × 28 = 245,000W

바닥 (지중):
- U값: 0.5 W/m²·K
- 면적: 1,650m²
- 손실: 0.5 × 1,650 × 15 = 12,375W

환기 열손실 (1회/시간):
- 공기 열용량: 1,200 J/m³·K
- 손실: 6,600 × 1,200 × 28 / 3,600 = 61,600W

총 난방 부하: 약 320 kW (안전율 1.2 적용)
```

#### 냉방 부하 (여름철)
```
여름철 조건:
- 외기 온도: 35°C
- 일사량: 800 W/m² (피크)
- 목표 온도: 22°C

일사 취득:
- 투과율: 70%
- 취득: 1,650 × 800 × 0.7 = 924,000W

외피 취득:
- 2,500 × 3.5 × 13 = 113,750W

내부 발열 (조명):
- 10개 LED × 100W = 1,000W

총 냉방 부하: 약 1,040 kW (피크 시)
```

---

## 2. 냉난방 시스템 하드웨어 구성

### 2.1 난방 시스템

#### A. 주 난방 시스템 - 온수 보일러 + 방열기
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 농업용 온수 보일러 | 400,000 kcal/h | 1 | 15,000,000 | 15,000,000 |
| 순환 펌프 | 3HP | 2 | 800,000 | 1,600,000 |
| 팬코일 유닛 | 10,000 kcal/h | 8 | 450,000 | 3,600,000 |
| 온수 배관 | PE-RT 32mm | 300m | 15,000 | 4,500,000 |
| 보일러실 설치비 | - | 1 | 3,000,000 | 3,000,000 |

#### B. 보조 난방 - 전기 히팅 케이블
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 토양 가온선 | 20W/m | 500m | 8,000 | 4,000,000 |
| 난방 조절기 | 온도센서 포함 | 4 | 350,000 | 1,400,000 |
| 전기 판넬 히터 | 2kW, 예비용 | 4 | 180,000 | 720,000 |

**난방 시스템 소계: 33,820,000원**

### 2.2 냉방 시스템

#### A. 주 냉방 시스템 - 증발 냉각 (Evaporative Cooling)
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 쿨링 패드 시스템 | 10m × 2m | 2 | 4,500,000 | 9,000,000 |
| 순환 펌프 | 1HP | 2 | 450,000 | 900,000 |
| 물탱크 | 500L | 2 | 350,000 | 700,000 |
| 배관 및 스프레이 노즐 | - | 1 | 800,000 | 800,000 |

#### B. 보조 냉방 - 포그 시스템 (Fog System)
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 고압 포그 펌프 | 70bar, 1HP | 1 | 2,800,000 | 2,800,000 |
| 스테인리스 노즐 | 0.2mm | 40 | 25,000 | 1,000,000 |
| 고압 배관 | 10mm | 200m | 12,000 | 2,400,000 |
| 필터 시스템 | 5μm | 1 | 450,000 | 450,000 |

#### C. 차광 시스템 (냉방 보조)
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 차광막 | 55% 차광율 | 1,700m² | 8,000 | 13,600,000 |
| 자동 개폐 모터 | AC 220V | 4 | 850,000 | 3,400,000 |
| 컨트롤러 | 타이머+센서 | 2 | 450,000 | 900,000 |

**냉방 시스템 소계: 35,950,000원**

### 2.3 환기 시스템

#### A. 자연 환기
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 천창 개폐기 | 전동식, 200kg | 6 | 1,200,000 | 7,200,000 |
| 측창 개폐기 | 전동식, 150kg | 8 | 900,000 | 7,200,000 |
| 풍향풍속계 | - | 1 | 650,000 | 650,000 |

#### B. 강제 환기
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 환기팬 (대형) | 48인치, 3상 | 4 | 1,800,000 | 7,200,000 |
| 환기팬 (소형) | 16인치, 단상 | 8 | 280,000 | 2,240,000 |
| 인버터 | 팬 속도 제어 | 4 | 450,000 | 1,800,000 |
| 차압 센서 | 환기 모니터링 | 2 | 180,000 | 360,000 |

#### C. 순환팬 (공기 교반)
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 순환팬 | 20인치 | 12 | 180,000 | 2,160,000 |
| 매달기 부품 | - | 12 | 30,000 | 360,000 |

**환기 시스템 소계: 29,170,000원**

### 2.4 제어 및 센서 (추가분)
| 항목 | 사양 | 수량 | 단가(원) | 금액(원) |
|------|------|------|----------|----------|
| 온도 센서 (추가) | PT100 | 8 | 45,000 | 360,000 |
| 습도 센서 (추가) | 산업용 | 4 | 85,000 | 340,000 |
| 풍속 센서 | 초음파식 | 2 | 450,000 | 900,000 |
| 일사량 센서 | 파이라노미터 | 1 | 1,200,000 | 1,200,000 |
| 외기 온습도 센서 | 방수형 | 2 | 120,000 | 240,000 |

**센서 추가 소계: 3,040,000원**

---

## 3. 통합 냉난방·환기 제어 시스템

### 3.1 제어 전략

#### 계절별 제어 모드

```python
class HVACMode(Enum):
    HEATING = "heating"          # 난방 모드 (겨울)
    COOLING = "cooling"          # 냉방 모드 (여름)
    VENTILATION = "ventilation"  # 환기 모드 (봄/가을)
    TRANSITION = "transition"    # 전환 모드

class SeasonalController:
    def __init__(self):
        self.current_mode = None
        self.outdoor_temp_history = []
    
    def determine_mode(self, outdoor_temp, indoor_temp, solar_radiation):
        """계절 및 시간대별 최적 모드 결정"""
        
        # 7일 평균 외기온도로 계절 판단
        avg_outdoor = self.get_weekly_avg_outdoor_temp()
        
        if avg_outdoor < 10:
            # 겨울철 - 난방 중심
            return HVACMode.HEATING
        elif avg_outdoor > 25:
            # 여름철 - 냉방 중심
            return HVACMode.COOLING
        elif 10 <= avg_outdoor <= 25:
            # 봄/가을 - 자연환기 중심
            return HVACMode.VENTILATION
```

#### 다단계 온도 제어 알고리즘

```python
class MultiStageTemperatureControl:
    """다단계 온도 제어 시스템"""
    
    def __init__(self):
        self.target_temp = 20.0  # 목표 온도
        self.deadband = 1.0       # 불감대 ±1°C
        
        # 난방 단계
        self.heating_stages = [
            {"name": "순환팬", "threshold": -1.0, "action": self.stage_circulation},
            {"name": "토양가온", "threshold": -2.0, "action": self.stage_soil_heating},
            {"name": "팬코일", "threshold": -3.0, "action": self.stage_fancoil},
            {"name": "보일러", "threshold": -4.0, "action": self.stage_boiler},
        ]
        
        # 냉방 단계
        self.cooling_stages = [
            {"name": "자연환기", "threshold": 1.0, "action": self.stage_natural_vent},
            {"name": "순환팬증속", "threshold": 2.0, "action": self.stage_circulation_high},
            {"name": "강제환기", "threshold": 3.0, "action": self.stage_forced_vent},
            {"name": "차광막", "threshold": 4.0, "action": self.stage_shading},
            {"name": "증발냉각", "threshold": 5.0, "action": self.stage_evap_cooling},
            {"name": "포그시스템", "threshold": 6.0, "action": self.stage_fogging},
        ]
    
    def control(self, current_temp, outdoor_temp, humidity, solar_radiation):
        """통합 온도 제어"""
        temp_error = self.target_temp - current_temp
        
        if abs(temp_error) < self.deadband:
            # 목표 범위 내 - 현재 상태 유지
            return "maintain"
        
        elif temp_error > 0:
            # 난방 필요
            return self.activate_heating_stages(temp_error, outdoor_temp)
        
        else:
            # 냉방 필요
            return self.activate_cooling_stages(
                abs(temp_error), outdoor_temp, humidity, solar_radiation
            )
    
    def activate_heating_stages(self, temp_deficit, outdoor_temp):
        """난방 단계별 활성화"""
        actions = []
        
        for stage in self.heating_stages:
            if temp_deficit >= abs(stage["threshold"]):
                actions.append(stage["action"]())
        
        # 외기온이 너무 낮으면 보일러 우선
        if outdoor_temp < -5:
            actions.append(self.stage_boiler_priority())
        
        return actions
    
    def activate_cooling_stages(self, temp_excess, outdoor_temp, 
                                 humidity, solar_radiation):
        """냉방 단계별 활성화"""
        actions = []
        
        for stage in self.cooling_stages:
            if temp_excess >= stage["threshold"]:
                action_result = stage["action"](
                    outdoor_temp, humidity, solar_radiation
                )
                if action_result:
                    actions.append(action_result)
        
        return actions
    
    # 난방 단계별 동작
    def stage_circulation(self):
        """1단계: 순환팬 가동"""
        return {
            "device": "circulation_fans",
            "action": "on",
            "speed": 50,
            "power": "1.2 kW"
        }
    
    def stage_soil_heating(self):
        """2단계: 토양 가온선"""
        return {
            "device": "soil_heating_cable",
            "action": "on",
            "zones": [1, 2, 3, 4],
            "power": "10 kW"
        }
    
    def stage_fancoil(self):
        """3단계: 팬코일 유닛"""
        return {
            "device": "fancoil_units",
            "action": "on",
            "units": 8,
            "power": "24 kW"
        }
    
    def stage_boiler(self):
        """4단계: 보일러 가동"""
        return {
            "device": "boiler",
            "action": "on",
            "setpoint": 70,  # 온수 온도 70°C
            "power": "320 kW"
        }
    
    def stage_boiler_priority(self):
        """극한 저온 - 보일러 우선 가동"""
        return {
            "device": "boiler",
            "action": "on",
            "mode": "priority",
            "setpoint": 75,
            "power": "350 kW"
        }
    
    # 냉방 단계별 동작
    def stage_natural_vent(self, outdoor_temp, humidity, solar_radiation):
        """1단계: 자연 환기"""
        # 외기가 실내보다 서늘하고, 습하지 않을 때만
        if outdoor_temp < self.target_temp - 2 and humidity < 80:
            return {
                "device": "roof_windows",
                "action": "open",
                "opening_ratio": 30,
                "side_windows": 20
            }
        return None
    
    def stage_circulation_high(self, outdoor_temp, humidity, solar_radiation):
        """2단계: 순환팬 고속"""
        return {
            "device": "circulation_fans",
            "action": "on",
            "speed": 100
        }
    
    def stage_forced_vent(self, outdoor_temp, humidity, solar_radiation):
        """3단계: 강제 환기"""
        # 외기 온도 확인
        if outdoor_temp < 30:
            return {
                "device": "exhaust_fans",
                "action": "on",
                "fans": [1, 2, 3, 4],
                "speed": 70,
                "roof_windows": 50,
                "power": "4.8 kW"
            }
        return None
    
    def stage_shading(self, outdoor_temp, humidity, solar_radiation):
        """4단계: 차광막 전개"""
        # 일사량이 높을 때만
        if solar_radiation > 600:
            # 일사량에 비례한 차광율
            shade_ratio = min(100, (solar_radiation - 400) / 4)
            return {
                "device": "shade_screen",
                "action": "deploy",
                "ratio": shade_ratio
            }
        return None
    
    def stage_evap_cooling(self, outdoor_temp, humidity, solar_radiation):
        """5단계: 증발 냉각"""
        # 습도가 너무 높으면 비활성
        if humidity < 75:
            return {
                "device": "cooling_pad",
                "action": "on",
                "water_flow": "high",
                "exhaust_fans": [1, 2],
                "power": "2.4 kW"
            }
        return None
    
    def stage_fogging(self, outdoor_temp, humidity, solar_radiation):
        """6단계: 포그 시스템 (최후 수단)"""
        if humidity < 70:
            return {
                "device": "fog_system",
                "action": "on",
                "duration": 30,  # 30초 분무
                "interval": 300,  # 5분 간격
                "power": "0.75 kW"
            }
        return None
```

### 3.2 환기 제어 알고리즘

```python
class VentilationController:
    """통합 환기 제어 시스템"""
    
    def __init__(self):
        self.target_co2 = 1000  # ppm
        self.target_humidity = 65  # %
        self.min_fresh_air = 0.5  # 시간당 환기 횟수 (최소)
    
    def control_ventilation(self, indoor_temp, outdoor_temp, 
                           co2_level, humidity, wind_speed):
        """환기 제어 메인 로직"""
        
        # 우선순위 1: CO2 농도 제어
        if co2_level > 1500:
            return self.emergency_ventilation()
        
        # 우선순위 2: 습도 제어
        if humidity > 80:
            return self.dehumidification_ventilation()
        
        # 우선순위 3: 온도 제어 보조
        if indoor_temp > self.target_temp + 2:
            return self.cooling_ventilation(outdoor_temp, wind_speed)
        
        # 일반 환기
        return self.normal_ventilation(co2_level, humidity)
    
    def emergency_ventilation(self):
        """긴급 환기 - CO2 과다"""
        return {
            "mode": "emergency",
            "roof_windows": 100,
            "side_windows": 100,
            "exhaust_fans": "all_on",
            "speed": 100,
            "duration": 600  # 10분
        }
    
    def dehumidification_ventilation(self):
        """제습 환기"""
        return {
            "mode": "dehumidification",
            "strategy": "temperature_differential",
            "roof_windows": 40,
            "exhaust_fans": [1, 3],
            "speed": 60,
            "heating": "on"  # 가온 제습
        }
    
    def cooling_ventilation(self, outdoor_temp, wind_speed):
        """냉각 환기"""
        if outdoor_temp < indoor_temp - 3:
            # 외기가 충분히 시원함
            opening = min(80, 50 + wind_speed * 5)
            return {
                "mode": "natural_cooling",
                "roof_windows": opening,
                "side_windows": opening - 10,
                "exhaust_fans": "off"
            }
        else:
            # 강제 환기 필요
            return {
                "mode": "forced_cooling",
                "roof_windows": 60,
                "exhaust_fans": "all_on",
                "speed": 80
            }
    
    def normal_ventilation(self, co2_level, humidity):
        """일반 환기"""
        # CO2 기반 환기량 계산
        ventilation_rate = self.calculate_ventilation_rate(co2_level)
        
        return {
            "mode": "normal",
            "roof_windows": ventilation_rate * 0.5,
            "side_windows": ventilation_rate * 0.3,
            "exhaust_fans": self.select_fans(ventilation_rate),
            "speed": min(60, ventilation_rate)
        }
    
    def calculate_ventilation_rate(self, co2_level):
        """CO2 농도 기반 환기량 계산"""
        if co2_level < 800:
            return 20  # 최소 환기
        elif co2_level < 1000:
            return 30
        elif co2_level < 1200:
            return 50
        else:
            return 70
    
    def smart_window_control(self, wind_speed, wind_direction, rain):
        """지능형 창문 제어"""
        if rain:
            # 비올 때는 천창만 부분 개방
            return {
                "roof_windows": 20,
                "side_windows": 0,
                "note": "rain_protection"
            }
        
        if wind_speed > 10:  # m/s
            # 강풍 시 개방 제한
            return {
                "roof_windows": 30,
                "side_windows": 10,
                "note": "strong_wind_protection"
            }
        
        # 풍향 고려한 개방
        if wind_direction in ["north", "northeast"]:
            return {
                "roof_windows": 50,
                "side_windows": {"south": 40, "north": 20}
            }
        
        return None
```

### 3.3 에너지 최적화 제어

```python
class EnergyOptimizer:
    """에너지 효율 최적화"""
    
    def __init__(self):
        # 전기 요금 (시간대별)
        self.electricity_rates = {
            "off_peak": 60,      # 23:00-09:00
            "mid_peak": 150,     # 09:00-12:00, 13:00-17:00
            "on_peak": 240       # 12:00-13:00, 17:00-23:00
        }
        
        self.thermal_mass = 6600 * 1.2 * 1000  # J/K (온실 열용량)
    
    def optimize_heating_schedule(self, weather_forecast):
        """난방 스케줄 최적화"""
        
        schedule = []
        
        for hour in range(24):
            rate = self.get_rate_for_hour(hour)
            forecast_temp = weather_forecast[hour]["temperature"]
            
            if rate == "off_peak":
                # 야간 저렴한 전기로 예열
                schedule.append({
                    "hour": hour,
                    "action": "preheat",
                    "target": 21,  # 1도 높게
                    "reason": "thermal_storage"
                })
            elif rate == "on_peak":
                # 피크 시간대는 최소 난방
                schedule.append({
                    "hour": hour,
                    "action": "minimal_heating",
                    "target": 19,  # 1도 낮게
                    "reason": "cost_saving"
                })
            else:
                # 일반 시간대
                schedule.append({
                    "hour": hour,
                    "action": "normal_heating",
                    "target": 20
                })
        
        return schedule
    
    def hybrid_heating_control(self, current_temp, outdoor_temp, 
                                electricity_rate):
        """하이브리드 난방 제어 (경유 보일러 + 전기)"""
        
        # 전기 요금이 비싼 시간대
        if electricity_rate == "on_peak":
            return {
                "primary": "boiler",  # 경유 보일러 우선
                "secondary": "none",
                "electric_heating": "off"
            }
        
        # 전기 요금이 저렴한 시간대
        elif electricity_rate == "off_peak":
            temp_deficit = 20 - current_temp
            
            if temp_deficit < 2:
                # 작은 부하 - 전기 난방
                return {
                    "primary": "electric",
                    "devices": ["soil_heating", "panel_heater"],
                    "boiler": "standby"
                }
            else:
                # 큰 부하 - 보일러 + 전기
                return {
                    "primary": "boiler",
                    "secondary": "electric",
                    "boiler_setpoint": 65,
                    "electric_supplement": True
                }
        
        # 중간 요금 시간대
        else:
            return {
                "primary": "boiler",
                "setpoint": 70,
                "electric_heating": "minimal"
            }
    
    def calculate_energy_cost(self, operations):
        """에너지 비용 계산"""
        total_cost = 0
        
        for op in operations:
            device = op["device"]
            duration = op["duration"]  # hours
            power = op["power"]  # kW
            rate = op["electricity_rate"]  # won/kWh
            
            if device == "boiler":
                # 경유 보일러 (8,000 kcal/L, 1,500원/L)
                fuel_consumption = power * duration * 0.086 / 8000  # L
                cost = fuel_consumption * 1500
            else:
                # 전기
                energy = power * duration  # kWh
                cost = energy * rate
            
            total_cost += cost
        
        return total_cost
    
    def predict_daily_energy_cost(self, weather_forecast):
        """일일 에너지 비용 예측"""
        
        hourly_costs = []
        
        for hour in range(24):
            outdoor_temp = weather_forecast[hour]["temperature"]
            solar_rad = weather_forecast[hour]["solar_radiation"]
            
            # 난방 부하 예측
            heating_load = self.estimate_heating_load(outdoor_temp)
            
            # 냉방 부하 예측
            cooling_load = self.estimate_cooling_load(outdoor_temp, solar_rad)
            
            # 비용 계산
            rate = self.get_rate_for_hour(hour)
            hour_cost = self.calculate_hourly_cost(
                heating_load, cooling_load, rate
            )
            
            hourly_costs.append({
                "hour": hour,
                "heating": heating_load,
                "cooling": cooling_load,
                "cost": hour_cost
            })
        
        daily_cost = sum(h["cost"] for h in hourly_costs)
        
        return {
            "daily_total": daily_cost,
            "hourly_breakdown": hourly_costs,
            "peak_hours": self.find_peak_cost_hours(hourly_costs)
        }
```

### 3.4 통합 제어 플로우

```python
class IntegratedHVACController:
    """통합 냉난방·환기 제어 시스템"""
    
    def __init__(self):
        self.temp_controller = MultiStageTemperatureControl()
        self.vent_controller = VentilationController()
        self.energy_optimizer = EnergyOptimizer()
        self.seasonal_controller = SeasonalController()
    
    def main_control_loop(self):
        """메인 제어 루프 (1분마다 실행)"""
        
        # 센서 데이터 수집
        indoor_temp = self.read_sensor("indoor_temperature")
        outdoor_temp = self.read_sensor("outdoor_temperature")
        humidity = self.read_sensor("humidity")
        co2 = self.read_sensor("co2")
        solar_rad = self.read_sensor("solar_radiation")
        wind_speed = self.read_sensor("wind_speed")
        wind_direction = self.read_sensor("wind_direction")
        rain = self.read_sensor("rain_detector")
        
        # 현재 전기 요금 시간대
        current_rate = self.energy_optimizer.get_current_rate()
        
        # 계절 모드 결정
        hvac_mode = self.seasonal_controller.determine_mode(
            outdoor_temp, indoor_temp, solar_rad
        )
        
        # 온도 제어 명령 생성
        temp_actions = self.temp_controller.control(
            indoor_temp, outdoor_temp, humidity, solar_rad
        )
        
        # 환기 제어 명령 생성
        vent_actions = self.vent_controller.control_ventilation(
            indoor_temp, outdoor_temp, co2, humidity, wind_speed
        )
        
        # 에너지 최적화 적용
        optimized_actions = self.energy_optimizer.optimize_actions(
            temp_actions, vent_actions, current_rate
        )
        
        # 충돌 해결 (온도 vs 환기)
        final_actions = self.resolve_conflicts(
            optimized_actions, hvac_mode
        )
        
        # 명령 실행
        self.execute_actions(final_actions)
        
        # 로깅
        self.log_operations(final_actions, indoor_temp, outdoor_temp)
    
    def resolve_conflicts(self, actions, hvac_mode):
        """제어 명령 충돌 해결"""
        
        # 예: 난방 중인데 환기 명령이 온 경우
        if hvac_mode == HVACMode.HEATING:
            # 환기는 최소화, 난방 우선
            if "ventilation" in actions:
                actions["ventilation"]["opening"] = min(
                    actions["ventilation"]["opening"], 30
                )
        
        # 예: 냉방 중인데 CO2가 높은 경우
        elif hvac_mode == HVACMode.COOLING:
            if actions.get("co2") == "high":
                # 환기 우선, 냉방은 강화
                actions["cooling"]["intensity"] = "max"
                actions["ventilation"]["priority"] = True
        
        return actions
    
    def execute_actions(self, actions):
        """MQTT를 통한 명령 실행"""
        
        for device, command in actions.items():
            topic = f"smartfarm/actuator/{device}/set"
            payload = json.dumps(command)
            
            self.mqtt_client.publish(topic, payload, qos=2)
            
            # 실행 확인 대기 (타임아웃 5초)
            if not self.wait_for_confirmation(device, timeout=5):
                self.handle_actuator_failure(device)
```

---

## 4. ESP32 펌웨어 (온도 제어용)

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

// 핀 정의
#define DHT_PIN 4
#define SOIL_HEATING_RELAY 5
#define PANEL_HEATER_RELAY 18
#define FAN_PWM_PIN 19

// 디바이스 설정
DHT dht(DHT_PIN, DHT22);
WiFiClient espClient;
PubSubClient mqtt(espClient);

// 제어 변수
bool soil_heating_on = false;
bool panel_heater_on = false;
int fan_speed = 0;  // 0-100%

void setup() {
  Serial.begin(115200);
  
  pinMode(SOIL_HEATING_RELAY, OUTPUT);
  pinMode(PANEL_HEATER_RELAY, OUTPUT);
  pinMode(FAN_PWM_PIN, OUTPUT);
  
  // PWM 설정 (팬 속도 제어)
  ledcSetup(0, 25000, 8);  // 25kHz, 8-bit
  ledcAttachPin(FAN_PWM_PIN, 0);
  
  dht.begin();
  
  setup_wifi();
  mqtt.setServer("192.168.1.100", 1883);
  mqtt.setCallback(mqtt_callback);
}

void mqtt_callback(char* topic, byte* payload, unsigned int length) {
  String message;
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  
  // 토양 가온선 제어
  if (String(topic) == "smartfarm/zone1/actuator/soil_heating/set") {
    if (message == "ON") {
      digitalWrite(SOIL_HEATING_RELAY, HIGH);
      soil_heating_on = true;
    } else {
      digitalWrite(SOIL_HEATING_RELAY, LOW);
      soil_heating_on = false;
    }
  }
  
  // 패널 히터 제어
  else if (String(topic) == "smartfarm/zone1/actuator/panel_heater/set") {
    if (message == "ON") {
      digitalWrite(PANEL_HEATER_RELAY, HIGH);
      panel_heater_on = true;
    } else {
      digitalWrite(PANEL_HEATER_RELAY, LOW);
      panel_heater_on = false;
    }
  }
  
  // 순환팬 속도 제어 (PWM)
  else if (String(topic) == "smartfarm/zone1/actuator/circulation_fan/speed") {
    fan_speed = message.toInt();
    int pwm_value = map(fan_speed, 0, 100, 0, 255);
    ledcWrite(0, pwm_value);
  }
}

void loop() {
  if (!mqtt.connected()) {
    reconnect();
  }
  mqtt.loop();
  
  // 1분마다 센서 데이터 전송
  static unsigned long last_send = 0;
  if (millis() - last_send > 60000) {
    send_sensor_data();
    last_send = millis();
  }
  
  // 로컬 안전 제어 (MQTT 연결 끊김 시)
  local_safety_control();
}

void send_sensor_data() {
  float temp = dht.readTemperature();
  float humid = dht.readHumidity();
  
  if (!isnan(temp) && !isnan(humid)) {
    char tempStr[8];
    char humidStr[8];
    dtostrf(temp, 1, 2, tempStr);
    dtostrf(humid, 1, 2, humidStr);
    
    mqtt.publish("smartfarm/zone1/sensor/temperature", tempStr);
    mqtt.publish("smartfarm/zone1/sensor/humidity", humidStr);
    
    // 상태 정보도 전송
    String status = String("{\"soil_heating\":") + soil_heating_on + 
                    ",\"panel_heater\":" + panel_heater_on +
                    ",\"fan_speed\":" + fan_speed + "}";
    mqtt.publish("smartfarm/zone1/status", status.c_str());
  }
}

void local_safety_control() {
  // MQTT 연결이 5분 이상 끊기면 로컬 제어
  static unsigned long last_mqtt_time = 0;
  
  if (mqtt.connected()) {
    last_mqtt_time = millis();
    return;
  }
  
  if (millis() - last_mqtt_time < 300000) {
    return;  // 5분 이내면 대기
  }
  
  // 로컬 제어 모드
  float temp = dht.readTemperature();
  
  if (temp < 16) {
    // 긴급 난방
    digitalWrite(SOIL_HEATING_RELAY, HIGH);
    digitalWrite(PANEL_HEATER_RELAY, HIGH);
  } else if (temp > 24) {
    // 긴급 냉각
    ledcWrite(0, 255);  // 팬 최대
  }
}
```

---

## 5. 업데이트된 총 비용

### 5.1 하드웨어 비용 재산정

| 카테고리 | 기존 비용 | 추가 비용 | 합계 |
|----------|-----------|-----------|------|
| 기본 시스템 | 3,157,000 | 2,453,000 | 5,610,000 |
| 난방 시스템 | 0 | 33,820,000 | 33,820,000 |
| 냉방 시스템 | 0 | 35,950,000 | 35,950,000 |
| 환기 시스템 | 140,000 | 29,030,000 | 29,170,000 |
| 추가 센서 | 0 | 3,040,000 | 3,040,000 |
| **소계** | **3,297,000** | **104,293,000** | **107,590,000** |

### 5.2 시공 및 기타 비용

| 항목 | 금액(원) |
|------|----------|
| 배관 시공비 | 5,000,000 |
| 전기 공사비 | 4,000,000 |
| 보일러실 구축 | 3,000,000 |
| 시운전 및 조정 | 2,000,000 |
| 소프트웨어 개발 | 3,000,000 |
| 예비비 (10%) | 12,459,000 |
| **소계** | **29,459,000** |

### 5.3 총 투자 비용

**총 초기 투자: 137,049,000원 (약 1억 3,700만원)**

### 5.4 연간 운영 비용

| 항목 | 월간(원) | 연간(원) |
|------|----------|----------|
| 전기료 (평균) | 800,000 | 9,600,000 |
| 난방 연료 (경유, 겨울 4개월) | 2,000,000 | 8,000,000 |
| 수도료 | 150,000 | 1,800,000 |
| 유지보수 | 200,000 | 2,400,000 |
| 소모품 교체 | 100,000 | 1,200,000 |
| **합계** | **3,250,000** | **23,000,000** |

---

## 6. 에너지 효율 개선 효과

### 6.1 난방 에너지 절감

```
기존 단순 보일러 시스템:
- 연간 경유 소비: 약 30,000L
- 비용: 45,000,000원

개선된 통합 시스템:
- 다단계 제어로 10% 절감
- 축열 운전으로 15% 절감
- 단열 개선 효과 5% 절감
- 총 절감: 30%

절감 후 비용: 31,500,000원
연간 절감액: 13,500,000원
```

### 6.2 냉방 에너지 절감

```
기존 단순 환기 시스템:
- 연간 전기 소비: 약 15,000 kWh
- 비용: 2,250,000원

개선된 증발냉각 시스템:
- 전기 소비 40% 절감
- 차광 제어로 20% 절감
- 총 절감: 60%

절감 후 비용: 900,000원
연간 절감액: 1,350,000원
```

### 6.3 ROI 분석

```
총 초기 투자: 137,049,000원
연간 절감액: 14,850,000원 (난방+냉방)
연간 생산성 향상: 약 10,000,000원 (수확량 20% 증가)

연간 총 효과: 24,850,000원

투자 회수 기간: 5.5년
```

---

## 7. 시스템 운영 시나리오

### 시나리오 1: 한겨울 (-10°C 외기)

```
시간: 오전 6시
외기: -10°C
실내: 17°C (목표 18°C)

제어 시퀀스:
1. 보일러 가동 (우선)
2. 팬코일 전체 가동
3. 토양 가온선 ON
4. 순환팬 50% 가동
5. 모든 창문 닫힘
6. 차광막 격납 (일사 취득)

예상 소비 전력: 350 kW
예상 난방 비용: 약 50,000원/일
```

### 시나리오 2: 한여름 (35°C 외기, 강한 일사)

```
시간: 오후 2시
외기: 35°C
실내: 26°C (목표 22°C)

제어 시퀀스:
1. 차광막 80% 전개
2. 천창 60% 개방
3. 대형 환기팬 전체 가동
4. 증발냉각 시스템 ON
5. 포그 시스템 간헐 가동 (30초/5분)
6. 순환팬 100% 가동

예상 소비 전력: 50 kW
예상 냉방 비용: 약 18,000원/일
```

### 시나리오 3: 봄/가을 (쾌적한 날씨)

```
시간: 오후 3시
외기: 20°C
실내: 21°C
일사: 보통

제어 시퀀스:
1. 천창 30% 개방 (자연 환기)
2. 순환팬 30% 가동
3. 난방/냉방 OFF
4. CO2 모니터링 (필요시 환기)

예상 소비 전력: 5 kW
예상 비용: 약 1,800원/일
```

---

## 8. 유지보수 체크리스트

### 일일 점검
- [ ] 보일러 압력 확인
- [ ] 순환 펌프 작동 확인
- [ ] 온도 센서 정상 작동
- [ ] 대시보드 알람 확인

### 주간 점검
- [ ] 쿨링 패드 청소
- [ ] 포그 노즐 점검
- [ ] 환기팬 필터 점검
- [ ] 차광막 동작 확인

### 월간 점검
- [ ] 보일러 연소 상태 점검
- [ ] 배관 누수 점검
- [ ] 릴레이 접점 확인
- [ ] 센서 캘리브레이션

### 계절별 점검
**여름 전 (5월)**
- [ ] 증발냉각 시스템 전체 점검
- [ ] 차광막 교체 (필요시)
- [ ] 환기팬 베어링 점검

**겨울 전 (10월)**
- [ ] 보일러 전체 정비
- [ ] 토양 가온선 저항 측정
- [ ] 단열재 점검
- [ ] 동파 방지 대책

---

## 9. 결론

냉난방 및 통풍 시스템을 포함한 완전한 스마트팜 구축에는 **약 1억 3,700만원**의 초기 투자가 필요하지만, 다음과 같은 이점이 있습니다:

### 주요 장점
1. **사계절 안정적 생산**: 연중 최적 환경 유지
2. **에너지 효율**: 30-60% 에너지 절감
3. **생산성 향상**: 20% 이상 수확량 증가
4. **투자 회수**: 약 5.5년

### 핵심 기술
- 다단계 온도 제어
- 지능형 환기 시스템
- 에너지 최적화 알고리즘
- 통합 모니터링

이 시스템은 **기술적으로 검증된 방식**들을 조합한 것으로, 실현 가능성이 높습니다.
