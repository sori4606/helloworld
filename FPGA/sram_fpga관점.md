# FPGA 관점에서 SRAM 제어하기

## FPGA에서 SRAM을 제어한다는 것은 **타이밍에 맞춰 GPIO 핀들을 정확히 켜고 끄는 것**.

## 🎯 핵심 개념

```
FPGA의 GPIO 핀 → PCB 배선 → SRAM의 핀

FPGA가 할 일:
1. 주소를 내보낸다 (A0~A14 핀에 출력)
2. 제어신호를 조작한다 (CE, OE, WE)
3. 데이터를 주고받는다 (I/O0~I/O7)
4. **타이밍을 지킨다!** ⏱️
```

---

## 📖 READ 과정 (FPGA 관점)

### **단계별 동작**

```
상태: IDLE → ADDR_SETUP → ADDR_STABLE → ENABLE → WAIT → READ → DISABLE → IDLE
```

### **1. IDLE (대기 상태)**

```vhdl
SRAM_ADDR <= (others => 'Z');  -- 주소 핀 안 쓰는 중
SRAM_CE <= '1';                -- 칩 비활성화 (HIGH)
SRAM_OE <= '1';                -- 출력 비활성화 (HIGH)
SRAM_WE <= '1';                -- 읽기 모드 (HIGH)
SRAM_DATA <= (others => 'Z');  -- 데이터 핀 입력 모드
```

**FPGA는 모든 제어 신호를 비활성 상태로 유지**

---

### **2. ADDR_SETUP (주소 설정)**

```vhdl
SRAM_ADDR <= "000000000010100";  -- 예: 주소 0x0014
SRAM_CE <= '1';                  -- 아직 칩 선택 안 함
SRAM_OE <= '1';                  -- 아직 출력 활성화 안 함
SRAM_WE <= '1';                  -- 읽기 모드
```

**물리적으로 일어나는 일:**

```
FPGA 핀 → PCB 신호선 → SRAM 핀
       (약 5~10ns 전파 지연)
```

**왜 이 단계가 필요한가?**

- 신호가 FPGA에서 SRAM까지 물리적으로 전달되는 시간 필요
- SRAM 내부에서 주소 디코딩 준비 시간 필요

---

### **3. ENABLE (CE, OE 활성화)**

```vhdl
SRAM_ADDR <= "000000000010100";  -- 계속 유지
SRAM_CE <= '0';  -- 🎯 칩 선택! (Active LOW)
SRAM_OE <= '0';  -- 🎯 출력 활성화! (Active LOW)
SRAM_WE <= '1';  -- 읽기 모드
```

**물리적으로 일어나는 일:**

```
SRAM 내부:
1. CE = LOW → 칩 깨어남, 전력 소비 시작
2. OE = LOW → 출력 버퍼 활성화
3. 주소 0x0014의 메모리 셀 접근 시작
4. 데이터를 출력 버퍼로 전송 시작
```

**이 시점부터 tCO(100ns) 타이밍 시작!**

---

### **4. WAIT (tACC, tCO 대기)**

```vhdl
-- 신호 유지, 최소 100ns 대기
SRAM_CE <= '0';  -- 유지
SRAM_OE <= '0';  -- 유지
```

**SRAM 내부 동작:**

```
t = 0ns:   CE/OE 활성화
t = 50ns:  메모리 셀 접근 중...
t = 100ns: 데이터가 I/O 핀에 출력됨! ✅
t = 150ns: 여유 시간
```

---

### **5. READ (데이터 읽기)**

```vhdl
data_buffer <= SRAM_DATA;  -- 🎯 클럭 에지에서 샘플링!
-- 이 순간 I/O0~I/O7 핀의 값을 읽어옴

-- 신호는 계속 유지
SRAM_CE <= '0';
SRAM_OE <= '0';
```

**물리적으로 일어나는 일:**

```
SRAM I/O 핀: 10110101 (0xB5)
      ↓
   PCB 배선
      ↓
FPGA GPIO 핀 ← rising_edge(clk)에서 샘플링
      ↓
data_buffer = 0xB5 ✅
```

**핵심:**

- `rising_edge(clk)` 순간에 FPGA가 핀의 값을 읽어서 내부 레지스터에 저장

---

### **6. DISABLE (신호 비활성화)**

```vhdl
SRAM_CE <= '1';  -- 칩 비활성화
SRAM_OE <= '1';  -- 출력 비활성화
read_done <= '1'; -- 상위 모듈에 완료 신호
```

**SRAM 반응:**

```
t = 0ns:   OE = HIGH
t = 35ns:  I/O 핀이 High-Z로 (tOD)
           → 다른 장치가 버스 사용 가능
```

---

## ✏️ WRITE 과정 (FPGA 관점)

### **READ와의 차이점**

1. **OE는 계속 HIGH** (출력 비활성화!)
2. **데이터는 FPGA가 출력**
3. **WE를 LOW → HIGH로 만들 때 저장됨**

---

### **단계별 동작**

```
IDLE → ADDR_SETUP → DATA_SETUP → ENABLE_CE → WE_LOW → WE_HIGH → HOLD → DONE
```

---

### **1. IDLE**

```vhdl
SRAM_ADDR <= (others => 'Z');
SRAM_DATA <= (others => 'Z');  -- 아직 입력 모드
SRAM_CE <= '1';
SRAM_OE <= '1';  -- ⚠️ 쓰기 내내 HIGH!
SRAM_WE <= '1';
```

---

### **2. ADDR_SETUP (주소 설정)**

```vhdl
SRAM_ADDR <= "000000000010100";  -- 주소 0x0014
SRAM_DATA <= (others => 'Z');    -- 아직 입력 모드
SRAM_CE <= '1';
SRAM_OE <= '1';
SRAM_WE <= '1';
```

**FPGA → PCB → SRAM 전파 시간 확보**

---

### **3. DATA_SETUP (데이터 설정)**

```vhdl
SRAM_ADDR <= "000000000010100";  -- 유지
SRAM_DATA <= "10101011";         -- 🎯 쓸 데이터 0xAB 출력!
SRAM_CE <= '1';
SRAM_OE <= '1';
SRAM_WE <= '1';
```

**물리적으로 일어나는 일:**

```
FPGA I/O 핀: 10101011 (출력 모드로 전환)
      ↓
   PCB 배선
      ↓
SRAM I/O 핀 ← 0xAB 수신 대기
```

**중요:** WE를 LOW로 만들기 최소 40ns 전에 데이터 준비 (tDS)

---

### **4. ENABLE_CE (칩 활성화)**

```vhdl
SRAM_ADDR <= "000000000010100";
SRAM_DATA <= "10101011";
SRAM_CE <= '0';  -- 🎯 칩 선택
SRAM_OE <= '1';  -- ⚠️ 계속 HIGH (중요!)
SRAM_WE <= '1';  -- 아직 쓰기 시작 안 함
```

**왜 OE = HIGH?**

```
만약 OE = LOW면:
SRAM도 I/O 핀에 데이터 출력 시도
    +
FPGA도 I/O 핀에 데이터 출력 중
    ↓
버스 충돌! 🔥 (회로 손상 위험)
```

---

### **5. WE_LOW (쓰기 시작)**

```vhdl
SRAM_ADDR <= "000000000010100";
SRAM_DATA <= "10101011";
SRAM_CE <= '0';
SRAM_OE <= '1';
SRAM_WE <= '0';  -- 🎯 쓰기 모드!
```

**SRAM 내부 동작:**

```
t = 0ns:   WE = LOW (쓰기 시작)
t = 20ns:  주소 디코딩 완료
t = 40ns:  메모리 셀 접근
t = 60ns:  쓰기 회로 준비
t = 75ns:  쓰기 준비 완료 ✅
```

**최소 75ns 동안 WE = LOW 유지 필수! (tWP)**

---

### **6. WE_HIGH (상승 에지 → 저장!)**

```vhdl
SRAM_WE <= '1';  -- 🎯 LOW → HIGH 상승 에지!
-- 다른 신호는 유지
SRAM_CE <= '0';
SRAM_OE <= '1';
SRAM_ADDR <= "000000000010100";
SRAM_DATA <= "10101011";
```

**SRAM 내부:**

```
WE 상승 에지 감지!
    ↓
주소 0x0014의 메모리 셀에
데이터 0xAB 저장 완료! ✅
```

**이 순간이 쓰기의 핵심!**

---

### **7. HOLD (데이터/주소 유지)**

```vhdl
-- 5~10ns 동안 유지 (tDH, tWR)
SRAM_ADDR <= "000000000010100";  -- 유지
SRAM_DATA <= "10101011";         -- 유지
SRAM_CE <= '0';
SRAM_OE <= '1';
SRAM_WE <= '1';
```

**왜 필요한가?**

- SRAM 내부 회로 안정화 시간
- 10MHz 클럭이면 자동으로 만족됨

---

### **8. DONE (완료)**

```vhdl
SRAM_CE <= '1';                  -- 비활성화
SRAM_DATA <= (others => 'Z');    -- 입력 모드로
write_done <= '1';               -- 완료 신호
```

---

## 🎮 FPGA 입장에서의 전체 흐름

### **READ:**

```
1. 주소 핀에 값 출력
2. 10~20ns 기다림 (주소 안정화)
3. CE, OE 핀을 LOW로
4. 100ns 기다림 (SRAM이 데이터 준비하는 시간)
5. 클럭 에지에서 데이터 핀 읽기
6. CE, OE 핀을 HIGH로
```

### **WRITE:**

```
1. 주소 핀에 값 출력
2. 데이터 핀에 값 출력 (출력 모드로!)
3. CE = LOW, OE = HIGH
4. WE를 LOW로 (75ns 이상 유지)
5. WE를 HIGH로 (이 순간 저장!)
6. 5ns 유지
7. CE = HIGH
```

---

## ⚡ 클럭과 타이밍의 관계

### 클럭 주파수와 주기의 관계식

주기(ns) = 1,000,000,000 / 주파수(Hz)
주기(ns) = 1,000 / 주파수(MHz)

또는 반대로

주파수(MHz) = 1,000 / 주기(ns)

### **10MHz 클럭 (100ns 주기)**

```
클럭:  _/‾‾\_/‾‾\_/‾‾\_/‾‾\_/‾‾\_

상태:  IDLE|ADDR|EN |WAIT|READ|
       100ns 100ns 100ns 100ns
```

**각 상태가 1 클럭 = 100ns**

- tACC(100ns) ✅ 만족
- tWP(75ns) ✅ 만족

### **50MHz 클럭 (20ns 주기)는?**

```
❌ 너무 빠름!
1 클럭 = 20ns < tWP(75ns)
→ 4 클럭 필요 (80ns)
```

---

## 📊 타이밍 다이어그램 요약

### READ CYCLE

```
시간:  0    100  200  300  400  500  600  700 (ns)
       |    |    |    |    |    |    |    |

ADDR:  ────X[0x0014]════════════════════X────

CE:    ────────────╲_________________╱────

OE:    ────────────╲_________________╱────

WE:    ──────────────────────────────────
       (HIGH 유지)

DATA:  ──────────────────<==0xB5==>───────
                         ↑
                    데이터 샘플링
```

### WRITE CYCLE

```
시간:  0    100  200  300  400  500  600  700  800 (ns)
       |    |    |    |    |    |    |    |    |

ADDR:  ────X[0x0014]══════════════════════X────

DATA:  ────────X[0xAB]════════════════════X────

CE:    ────────────╲_____________________╱────

WE:    ──────────────────╲______╱─────────────
                         ↑      ↑
                      쓰기 시작  🎯 저장!
                         |←75ns→|

OE:    ──────────────────────────────────────
       (HIGH 유지)
```

---

## 🔧 실전 체크리스트

### ✅ READ 전 확인사항

#### 1. **클럭 속도 확인 (SRAM 데이터시트)**

```
확인할 값들:
- tRC (Read Cycle Time): 100ns → 최대 10MHz
- tACC (Address Access Time): 100ns → 최대 10MHz
- tCO (Chip Enable to Output): 100ns → 최대 10MHz

결정:
클럭 주파수 ≤ 1000 / max(tRC, tACC, tCO) MHz
            = 1000 / 100 = 10MHz ✅

권장: 10MHz 이하 (예: 8MHz, 10MHz)
```

#### 2. **주소 안정화 시간 확보 (실무 관행)**

```
데이터시트 파라미터: 없음 (READ에는)
실전 권장: 10~20ns

방법:
- 주소 설정 후
- 1 클럭 대기 (10MHz = 100ns이면 충분!)
- CE/OE 활성화
```

#### 3. **데이터 유효 시간 확보 (tACC, tCO)**

```
tACC: 100ns (주소 기준)
tCO:  100ns (CE 기준)
tOE:  50ns  (OE 기준)

실제 데이터 유효 = max(tACC, tCO, tOE) = 100ns

방법:
- CE/OE 활성화 후
- 최소 100ns 대기 (1~2 클럭)
- 데이터 읽기
```

#### 4. **WE = HIGH 확인**

```
READ 모드 = WE는 항상 HIGH
(절대 LOW로 만들면 안 됨!)
```

---

### ✅ WRITE 체크리스트

#### 1. **클럭 속도 확인**

```
확인할 값들:
- tWC (Write Cycle Time): 100ns → 최대 10MHz
- tWP (Write Pulse Width): 75ns → 최소 1클럭 필요

결정:
클럭 주파수 ≤ 10MHz
```

#### 2. **주소 안정화 시간 (tAW)**

```
tAW (Address Valid to End of Write): 0ns
→ 이론상 동시 가능
→ 실전: 10~20ns 여유 권장

방법:
- 주소 먼저 설정
- 1 클럭 대기
- WE 조작
```

#### 3. **데이터 Setup/Hold 시간**

```
tDS (Data Setup): 40ns
→ WE 상승 에지 전 40ns 동안 데이터 유효

tDH (Data Hold): 0ns (실전 5~10ns)
→ WE 상승 에지 후 5~10ns 데이터 유지
```

#### 4. **WE 펄스 폭 (tWP) ⭐ 가장 중요**

```
tWP: 75ns
→ WE를 LOW로 유지하는 최소 시간

10MHz (100ns): 1 클럭으로 만족 ✅
20MHz (50ns): 2 클럭 필요 ❌
```

---

## 용어설명

### 1. GPIO (General Purpose Input/Output) (범용 입출력)

: FPGA 칩의 다리들. 전기 신호를 주고받는 물리적인 연결점

### GPIO는 양방향

```
출력 모드:
FPGA → (HIGH/LOW 신호) → 외부 장치

입력 모드:
외부 장치 → (HIGH/LOW 신호) → FPGA
```

## SRAM과 GPIO의 연결

```
FPGA                    PCB 배선                SRAM
┌─────────┐                                  ┌─────────┐
│         │                                  │         │
│ GPIO_0  ├───────────────────────────────── ┤ A0      │
│ GPIO_1  ├───────────────────────────────── ┤ A1      │
│ GPIO_2  ├───────────────────────────────── ┤ A2      │
│   ...   │                                  │  ...    │
│GPIO_20  ├───────────────────────────────── ┤ CE      │
│GPIO_21  ├───────────────────────────────── ┤ OE      │
│GPIO_22  ├───────────────────────────────── ┤ WE      │
│GPIO_23  ├───────────────────────────────── ┤ I/O0    │
│GPIO_24  ├───────────────────────────────── ┤ I/O1    │
│   ...   │                                  │  ...    │
└─────────┘                                  └─────────┘
```

### VHDL에서 GPIO 사용

```vhdl
entity sram_controller is
    port (
        -- 이것들이 실제 FPGA의 GPIO 핀들
        SRAM_ADDR : out std_logic_vector(14 downto 0);  -- GPIO 15개
        SRAM_DATA : inout std_logic_vector(7 downto 0); -- GPIO 8개
        SRAM_CE   : out std_logic;                      -- GPIO 1개
        SRAM_OE   : out std_logic;                      -- GPIO 1개
        SRAM_WE   : out std_logic                       -- GPIO 1개
    );
end entity;
```

### 2. rising_edge : 상승 에지

= 클럭이 0→1로 올라가는 순간

```
클럭 신호:

      ┌───┐   ┌───┐   ┌───┐
      │   │   │   │   │   │
  ────┘   └───┘   └───┘   └───
      ↑   ↑   ↑   ↑   ↑   ↑
      │   │   │   │   │   │
   rising falling rising falling
   edge   edge   edge   edge
   (상승) (하강) (상승) (하강)
```

### VHDL에서 rising_edge

```vhdl
process(clk)
begin
    if rising_edge(clk) then
        -- 이 안의 코드는
        -- 클럭이 0→1로 올라가는 바로 그 순간에만 실행!

        data_buffer <= SRAM_DATA;  -- 이 순간 데이터 캡처!
    end if;
end process;
```
