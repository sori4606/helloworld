# 74 series 예시를 통해 알아보기

## 목차

- [J, D, N ,,, 패키지 분석](./74_series.md#1-j-d-n--패키지)
- [W 패키지 분석](./74_series.md#2-w-패키지)
- [FK 패키지 분석](./74_series.md#3-fk-패키지)
- [NC](./74_series.md#nc)
- [recommended operating conditions 표 분석](./74_series.md#recommended-operating-conditions-5페이지)

---



<a href = "https://www.ti.com/product/SN74LS04?keyMatch=SN74LS04&tisearch=universal_search&usecase=GPN-ALT">예시 74 series (SN74LS04)</a> 사이트 들어가기

<a href = "https://www.ti.com/lit/ds/symlink/sn74ls04.pdf?ts=1760427401138&ref_url=https%253A%252F%252Fen.wikipedia.org%252F">data sheet</a> 열기

<div align = "center">
SN5404 . . . J PACKAGE <br />
SN54LS04, SN54S04 . . . J OR W PACKAGE <br />
SN7404, SN74S04 . . . D, N, OR NS PACKAGE <br />
SN74LS04 . . . D, DB, N, OR NS PACKAGE <br />

<img width="471" height="428" alt="J package" src="https://github.com/user-attachments/assets/4472c0a3-3a29-4a0f-8309-0a17a144849c" />

</div>

<br />
<br />

<div align = "center">
SN5404 . . . W PACKAGE

<img width="457" height="366" alt="W package" src="https://github.com/user-attachments/assets/968730ce-fcf4-4cf3-a030-c69fe527dcb1" />

</div>

<br />
<br />


<div align = "center">
SN54LS04, SN54S04 . . . FK PACKAGE

<img width="426" height="483" alt="FK package" src="https://github.com/user-attachments/assets/2712c983-8a3c-41fa-97e6-56dd03f92cef" />

</div>

<br />
<br />

## package 분석

### 1. J, D, N ... 패키지

- 신호 입력 (1A, 2A, 3A, 4A, 5A, 6A)
- 신호 출력 (1Y, 2Y, 3Y, 4Y, 5Y, 6Y)
- 핀 14 : VCC : 전원 공급 전압 (5V)
- 핀 7 : GND : 접지 (0V 기준점)

### 2. W 패키지

- 신호 입력 (1A, 2A, 3A, 4A, 5A, 6A)
- 신호 출력 (1Y, 2Y, 3Y, 4Y, 5Y, 6Y)
- 핀 4 : VCC : 전원 공급 전압 (5V)
- 핀 11 : GND : 접지 (0V 기준점)

### **왜 중앙 근처에 VCC, GND 배치 ?**

```
✓ 중앙 근처:
  → 최대 거리 최소화

✓ 완벽한 대칭:
  → 위 3개 게이트
  → 아래 3개 게이트

✓ 전원 분배 균등:
  → 전압 강하 최소

✓ 노이즈 감소:
  → 짧은 전원 배선

✓ 열 분산:
  → 중앙에서 양쪽으로
```

---

## 일반 DIP와 비교

### **N Package (끝 배치)**
```
   ┌─────────┐
1A─┤1      14├─VCC  ← 끝
1Y─┤2      13├─6A
2A─┤3      12├─6Y
2Y─┤4      11├─5A
3A─┤5      10├─5Y
3Y─┤6       9├─4A
GND┤7       8├─4Y  ← 끝
   └─────────┘

전원 위치: 7, 14번 (양 끝)
```

### **W Package (중앙 배치)**
```
   ┌─────────┐
1A─┤1      14├─1Y
2Y─┤2      13├─6A
2A─┤3      12├─6Y
VCC┤4     11├─GND  ← 중앙!
3A─┤5      10├─5Y
3Y─┤6       9├─5A
4A─┤7       8├─4Y
   └─────────┘

전원 위치: 4, 11번 (중앙)
```

### 3. FK 패키지

- 신호 입력 (1A, 2A, 3A, 4A, 5A, 6A)
- 신호 출력 (1Y, 2Y, 3Y, 4Y, 5Y, 6Y)
- 핀 20 : VCC : 전원 공급 전압 (5V)
- 핀 10 : GND : 접지 (0V 기준점)
- NC (No connection) : 1, 5, 7, 11, 15, 17

### **NC**
: 아무것도 연결 안되어 있음. 사용 안하는 핀임.

### 왜있음 ?

1. **IC 표준 크기 통일화** (FK 패키지는 20핀 표준)
   - 한 제조 라인으로 여러 IC 생산 가능

2. **열 방출 도움**
   - NC 핀도 금속이라 열 방출에 기여

3. **기계적 안정성**
   - 더 많은 핀 = 진동/충격에 강함

4. **노이즈 차단**
   - NC 핀을 GND 연결하여 전자기 차폐 가능


## 패키지 비교 요약

| 패키지 | 핀 수 | VCC 위치 | GND 위치 | 용도 | NC 핀 |
|--------|-------|----------|----------|------|-------|
| N, D | 14 | 14번 | 7번 | 상업용 | 없음 |
| W | 14 | 4번 | 11번 | 군사용 | 없음 |
| FK | 20 | 20번 | 10번 | 군사용 | 6개 |

**주의**: 패키지마다 핀 번호가 다르므로 PCB 설계 시 확인 필수!


## recommended operating conditions (5페이지)

<img width="1376" height="366" alt="recommended operating conditions" src="https://github.com/user-attachments/assets/08d6d2e6-9573-47d8-92b5-1c37ce450690" />

<br />
<br />

- 권장 동작 조건을 나타냄 (이 범위 안에서 사용하세요 ! 라는 의미)

이 조건 안에서: <br />
✓ 정상 동작 보장 <br />
✓ 스펙대로 작동 <br />
✓ 신뢰성 보장 <br />

이 조건 벗어나면: <br />
✗ 오동작 가능 <br />
✗ 성능 저하 <br />
✗ 고장 위험 <br />

## 표 해석

### **VCC (Supply Voltage) - 전원 전압**
```
SN5404 (군사용):
MIN: 4.5V
NOM: 5V    ← 표준값
MAX: 5.5V

SN7404 (상업용):
MIN: 4.75V
NOM: 5V
MAX: 5.25V

의미:
→ 5V 전원 사용
→ 군사용이 범위 더 넓음 (4.5~5.5V)
→ 상업용은 좁음 (4.75~5.25V)

왜 다른가?
군사용: 극한 환경 대비
상업용: 정밀한 전원 가정
```

### **VIH (High-level Input Voltage) - HIGH 입력 전압**
```
둘 다:
MIN: 2V

의미:
→ 입력에 2V 이상 주면
→ IC가 "1 (HIGH)"로 인식

예시:
입력 2.5V → HIGH ✓
입력 1.5V → 애매함 ✗
```

### **VIL (Low-level Input Voltage) - LOW 입력 전압**
```
둘 다:
MAX: 0.8V

의미:
→ 입력에 0.8V 이하 주면
→ IC가 "0 (LOW)"로 인식

예시:
입력 0.5V → LOW ✓
입력 1.0V → 애매함 ✗
```

### **IOH (High-level Output Current) - HIGH 출력 전류**
```
둘 다:
MAX: -0.4 mA

⚠️ 음수 의미:
→ 전류가 IC에서 "나감"
→ IC가 전류를 공급 (Source)
```

### **출력이 HIGH일 때**
```
상황:
출력 핀 = HIGH (5V)

      5V (출력 HIGH)
       ↓
    [IC 출력핀]
       │
       └─→ 전류 0.4mA만 나갈 수 있음
       ↓
      LED
       ↓
      GND
```


### **IOL (Low-level Output Current) - LOW 출력 전류**
```
둘 다:
MAX: 16 mA

양수 의미:
→ 전류가 IC로 "들어감"
→ IC가 전류를 흡수 (Sink)

의미:
→ 출력 LOW일 때
→ 최대 16mA까지 흡수 가능

예시:
LED 연결 (LOW에서 켜기):
→ 16mA 가능 ✓
→ 직접 연결 가능!
```

### **출력이 LOW일 때**
```
상황:
출력 핀 = LOW (0V)

      5V
       ↓
      LED
       ↓
    [IC 출력핀] ← 16mA까지 받아들일 수 있음
       │
      GND (LOW)
```


### **TA (Operating Temperature) - 동작 온도**
```
SN5404 (군사용):
MIN: -55°C
MAX: 125°C
→ 넓은 범위!

SN7404 (상업용):
MIN: 0°C
MAX: 70°C
→ 좁은 범위

차이:
군사: 사막(-55°C) ~ 엔진 근처(125°C)
상업: 실내 환경
```

## 실제 사용 예시

### **LED 연결 (잘못된 방법)**
```
HIGH로 LED 켜기:

5V ─[출력]─[LED]─[저항]─GND
      ↓
    0.4mA만 공급 ✗
    
LED 필요: 10~20mA
→ 너무 어둡거나 안 켜짐!
```

### **저항** : 전류를 흐르게 하는 수도꼭지

- 전류를 조절하는 안전장치
- IC 보호

### **LED 연결 (올바른 방법)**
```
LOW로 LED 켜기:

5V ─[저항]─[LED]─[출력]─GND
                   ↓
                 16mA 흡수 ✓
                 
LED: 밝게 켜짐!
```

---

## 용어 정리

| 약어 | 풀네임 | 의미 | 단위 |
|------|--------|------|------|
| VCC | Supply Voltage | 전원 전압 | V |
| VIH | High-level Input Voltage | HIGH 입력 전압 (최소) | V |
| VIL | Low-level Input Voltage | LOW 입력 전압 (최대) | V |
| IOH | High-level Output Current | HIGH 출력 전류 (최대) | mA |
| IOL | Low-level Output Current | LOW 출력 전류 (최대) | mA |
| TA | Operating Temperature | 동작 온도 범위 | °C |
