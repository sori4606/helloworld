# CPU 회로로 공부하기

## 목차

- [CPU](./cpu회로.md#cpu-c8051f023)
- [reset 회로](./cpu회로.md#1-reset-회로)
- [clock 회로](./cpu회로.md#2-clock-회로)
- [I2C 회로](./cpu회로.md#3-i2c-회로)
- [I2C 통신방법](./cpu회로.md#2-i2c-의-통신방법)
- [JTAG](./cpu회로.md#jtag)
- [rs232 driver](./cpu회로.md#rs232-driver-max3222eeup)
- [rs232 통신방법](./cpu회로.md#1-rs-232-통신방법)
- [buffer](./cpu회로.md#buffer)
- [Connector](./cpu회로.md#connector)
- [블록 다이어그램](./cpu회로.md#블록-다이어그램-그려보기)

## CPU (C8051F023)

<a href = "https://www.alldatasheet.com/datasheet-pdf/download/102988/SILABS/C8051F023.html">data sheet 다운로드하러가기</a>

### 분석할 CPU 회로도

<img width="2247" height="1361" alt="Image" src="https://github.com/user-attachments/assets/6112333f-d287-47bd-9a72-97900f24b9fa" />


### 1. Reset 회로

오른쪽 아래에 달려있다.

<img width="909" height="626" alt="Image" src="https://github.com/user-attachments/assets/5598e6b5-3f96-4d12-9c3b-4b01ee565a73" />


### `왜 있을까?`

-> CPU를 안정적인 초기 상태로 만들어주는 회로. 전원이 켜질 때나 시스템 오류시 CPU가 정해진 상태에서 시작하도록 보장

### `회로도 분석`

- 풀업저항이 걸려있어서 평상시에 HIGH 로 유지
- `SW1(스위치)`를 누르면 `reset`핀이 `GND`에 연결되고, reset 기능이 동작한다. ([reset 방법](./cpu회로.md#1-reset의-방법들-power-on-reset--manual-reset--soft-reset)에는 `power-on reset` , `manual reset` , `soft reset` 이 있음)
- C5, C6 (커패시터)의 충전, 방전을 이용해서 리셋 신호를 발생시킨다. ([커패시터가 있을때와 없을때의 차이 ?](./cpu회로.md#2-커패시터가-있을때와-없을때의-차이))
- R8 저항은 저항없이 전원에 직접 연결하면 과전류가 흘러서 부품이 타거나 파손될 수 있으므로 달아줬다.

### 추가 개념 사항

### 1. reset의 방법들. `power-on reset` , `manual reset` , `soft reset`

### 1. `power-on reset`

#### `개념`

- 전원이 처음 켜질 때 자동으로 발생하는 리셋

#### `원리`

전원이 천천히 상승할 때, 전압이 완전히 안정되기 전에 CPU가 실행을 시작하면, 엉뚱한 데이터가 레지스터에 들어갈 수 있다. <br />
그래서 전원이 정상 수준에 도달하고 전압이 안정될 때까지 리셋을 유지하는 회로가 필요하다.

이를 담당하는 게 Power-on Reset 회로다. 보통 내부에 RC 지연 회로나 전압 감지 비교기(Comparator)가 있어서
전압이 기준치 이상이 되면 리셋 신호를 해제한다.

#### `특징`

- **자동으로 발생** (사용자 개입 불필요)
- 모든 레지스터와 메모리를 **초기 상태**로 설정
- MCU, FPGA, CPU 등 모든 칩 기본 내장 기능
- 전압이 안정될 때까지 리셋 신호 유지

### 2. `manual reset`

#### `개념`

- 사람이 직접 누르거나 외부 신호로 리셋을 거는 방식.

#### `원리`

보드에는 보통 `RESET` 버튼이 하나 달려 있다.
그걸 누르면 `RST` 핀이 `GND`로 떨어지고, `CPU`는 다시 초기 상태로 돌아간다.
혹은 외부 회로(예: `watchdog timer`)가 오류를 감지했을 때 리셋 신호를 줄 수도 있다.

`watchdog` ? -> cpu 가 주기적으로 `WDI(watchdog input)` 핀에 펄스를 보내야하는데, `watchdog timeout` 동안 신호를 안보내면 `MCU`가 멈췄다고 생각하고 `reset`을 한다.

`mcu (Microcontroller Unit)` ? CPU + 메모리 + 주변 장치(Peripheral) 를 하나로 묶은 마이크로컴퓨터.

#### `특징`

- 수동 또는 외부 트리거 방식
- 회로적으로는 Power-on Reset과 같은 리셋 핀 공유
- 보통 디버깅, 펌웨어 업데이트 중, 시스템 이상 시 사용
- 전원은 그대로 유지, 단지 CPU만 초기화

### 3. `soft reset`

#### `개념`

- 소프트웨어 명령으로 수행되는 리셋.
- 프로그램 코드로 시스템 재시작

#### `원리`

이건 하드웨어 신호가 아니라 CPU 내부 명령 또는 레지스터 조작으로 일어나는 리셋이다. <br />
예를 들어 MCU에서는 특정 레지스터(예: `SYSRESETREQ`)에 1을 쓰면
CPU가 내부적으로 리셋 시퀀스를 수행한다.

또는 운영체제(예: 리눅스, 안드로이드)에서 “재부팅” 명령을 내리면
소프트웨어적으로 내부 주변장치 초기화 후 하드 리셋 신호를 생성한다.

#### `특징`

- 프로그램 코드로 실행 가능
- 전원은 끊기지 않음 (메모리 일부 유지 가능)
- 보통 펌웨어 업데이트 후, 설정 변경 후 자동 재부팅 등에 사용

### 2. 커패시터가 있을때와 없을때의 차이

전원을 켜거나 끌 때, 커패시터의 충전, 방전을 이용해서 리셋 신호를 발생시킨다고 하였다. <br />
그런데, 커패시터가 없으면 바운스 현상이 일어난다.

#### `바운스 현상의 개념`

기계식 스위치나 버튼의 `접점(contact)`은 금속판 두 개가 닿았다 떨어졌다 하면서 동작한다. <br />
그런데 실제로 닿는 순간, 금속이 완벽히 “착” 하고 붙는 게 아니라, <br />
미세한 탄성 진동 때문에 수 밀리초(ms) 동안 여러 번 “탁탁탁” 튀는 것이다.

즉, 버튼을 한 번 눌렀는데, 회로 입장에서는 여러 번 눌린 것처럼 보이는 것이 `바운스 현상`이다.

그래서 다시 커패시터가 없을때로 돌아가면, <br />
이상적인 스위치 동작은 아래의 그림과 같이 되어야한다.

<img width="1309" height="441" alt="Image" src="https://github.com/user-attachments/assets/159b6342-a69e-448b-8ff2-544fc692fafc" />

하지만, 바운스 현상이 발생하면, 아래의 그림과 같이 스위치를 한번만 눌렀음에도 불구하고, MCU가 여러번 입력한 것으로 인식한다.

<img width="1345" height="513" alt="Image" src="https://github.com/user-attachments/assets/9e951b07-20e7-4e98-b474-d6a86ee77706" />

즉, MCU나 FPGA의 입장에서는
“한 번 눌렀는데 신호가 여러 번 변한 것처럼 보이는” 현상이 발생한다.
결국 **논리적으로 노이즈**처럼 보인다.

그래서 `커패시터를 병렬`로 달면, 회로의 반응을 의도적으로 느리게 만들어서 전류를 흡수·방출하면서 신호를 부드럽게 필터링해준다. 그 결과 아래의 그림과 같이 (바운스 제거 후 파형) 바운스 신호의 미세한 진동이 무시되고, 한 번의 깔끔한 신호로 보인다.

<img width="1318" height="445" alt="Image" src="https://github.com/user-attachments/assets/a0a74f8e-dc8f-4b73-9155-b31d81d04f82" />


### 2. clock 회로

왼쪽 아래에 달려있다.

<img width="946" height="471" alt="Image" src="https://github.com/user-attachments/assets/b2597f3e-190e-4c8c-8e49-9876b9d952e8" />


### `왜 있을까?`

-> CPU 전체의 동작 리듬을 결정하는 `심장` 같은 존재. 클럭 회로는 CPU의 모든 동작 타이밍을 맞춰주는 기준 신호 발생기이고, 
클럭이 없으면 CPU는 언제 데이터를 읽고 쓸지 몰라서 동작이 붕괴한다.

### `회로도 분석`

- Y1( SCO-103 22.1184MHz) (crystal Oscillator) (크리스탈 발진자) : 특정주파수 신호를 만들어내는데 사용한다. 여기서는 22.1184MHz 의 속도로 클럭을 발생시킴
- FT1 (NFW31SP107X1E4) (EMI Filter) : 고속 디지털 신호회로에서 노이즈를 억제하기 위해 설계된 필터. 클럭 출력에는 노이즈가 포함되는데, 노이즈가 cpu로 전달되면 cpu 오동작, 다른 회로 간섭 등의 문제가 발생할 수 있기 때문에 달아준다. ([clock 구형파, crystal 파 ?](./cpu회로.md#1-clock-구형파-crystal파))
- BC6 는 `Bypass필터` 또는 `Decoupling필터` 라고 부른다. 전원 노이즈 제거에 사용되고, 자주 사용되니 꼭 알아두어야 한다.

### 추가 개념 사항

### 1. Clock 구형파, crystal파

### 1. Clock 구형파

CPU나 FPGA가 사용하는 `클럭(clock)` 은 대부분 `구형파(square wave)` 형태이다. 즉, 시간에 따라 전압이 빠르게 HIGH ↔ LOW로 바뀌는 신호.

```
파형:  ▔╲▁▁▁╱▔╲▁▁▁╱▔  (각진 사각파)
```

이 구형파는 단순히 `0(Low)` 과 `1(High)` 만 반복되는것이 아니라,
아주 정밀한 주기(주파수) 로 반복되어 CPU 내부의 모든 타이밍 기준이 된다. (예를 들어 50MHz 클럭이면 1초에 5천만 번 HIGH-LOW를 반복한다는 뜻)

그런데 이 구형파는 “이상적인 수학적 형태”가 아니고, 실제로는 매우 빠른 에지 때문에 고주파 성분(고조파) 이 잔뜩 섞여 있다. 이것이 바로 EMI(전자파 간섭)의 근원이다.

### 2. Crystal파

대부분의 시스템은 처음부터 구형파를 만들지 못한다. 
그래서 `Quartz Crystal(수정 발진기)` 를 사용해 아주 정확한 `사인파(sine wave)` 나 `약한 진동파`를 만든다. <br />
( `Quartz Crystal` = `발진기(oscillator)`의 핵심 공진 소자, `Crystal Oscillator` = 그 소자를 포함한 완성된 발진 회로)

```
파형:  ∿∿∿∿∿∿  (부드러운 사인파)
```

이건 `“기본 주파수(fundamental frequency)”`를 만드는 기준 신호 역할을 한다. <br />
예를 들어 `25MHz crystal`을 `CPU` 옆에 붙이면,
그걸 `PLL(Phase Locked Loop)` 회로가 받아서 더 높은 주파수로 변환한다. <br />
(`PLL` ? -> 기준 주파수를 입력받아, 위상을 맞추며 주파수를 배수/분주로 변환해내는 회로. 즉, 클럭을 ‘복제하되 더 빠르거나 느리게’ 만들면서, ‘동기화는 유지하는’ 똑똑한 제어 시스템)

#### `특징`

- **사인파(Sine Wave)** 형태
- **매우 순수한** 주파수 성분
- **노이즈가 거의 없음**
- **고조파(Harmonic) 성분이 적음**

#### `정리`

- `Crystal`의 부드러운 사인파를 디지털 회로가 이해할 수 있는 0과 1의 구형파로 변환하기 위해 `Crystal Oscillator`가 동작하고, 이렇게 만들어진 기준 클럭은 `PLL`에 의해 필요한 배수로 채배되어 `CPU`나 `다른 회로`로 분배된다. 이 과정에서 발생하는 고주파 노이즈를 억제하기 위해 `EMI 필터`를 추가한다.

### 3. I2C 회로

왼쪽 위에 달려있다.

<img width="801" height="406" alt="Image" src="https://github.com/user-attachments/assets/7a1632d4-dec1-47e0-a889-32ca3f6a4a32" />

### `왜 있을까 ?`

- 다른 여러 장치와 통신하기 위해 존재한다. 단 2개의 선으로 통신할 수 있다.

### `회로도 분석`

- SDA, SCL : 각각 Data 신호, 클럭 신호 핀이다. 데이터를 주고 받을 때 사용한다.
- R1, R3 : [damping저항](./cpu회로.md#1-damping-저항)이다. 주로 신호선과 클럭선에 사용한다.
- TP1, TP2 : 테스트 핀. 신호 측정할 때 사용하고, 디버깅용이다.

### 추가 개념 사항

### 1. damping 저항

-> 말 그대로 진동 (damping) 을 완화해주는 완충 장치같은 존재. 주로 고속 신호선이나 클럭 라인에서 [반사(reflection)](./cpu회로.md#반사) 와 [링잉(ringing)](./cpu회로.md#링잉) 을 줄이는 역할을 한다.

#### `왜 필요한가`

고속 디지털 신호(예: 클럭, 데이터 라인)는 사실상 짧은 `전송선(transmission line)` 이다. 그런데 전송선의 임피던스가 맞지 않으면, 신호가 종단에서 `반사(reflection)` 돼서 돌아온다. <br />
이 반사가 원 신호와 겹치면 `오버슈트(overshoot)`, `언더슈트(undershoot)`, `링잉(ringing)` 같은 현상이 생김. <br />
결과적으로: <br />
- 클럭이 이상하게 튀어서 타이밍 오류 발생
- FPGA, CPU 입력이 ‘1인지 0인지’ 오판
- `EMI(전자파 간섭)` 증가

### `반사`
: 신호가 전송선 끝에서 임피던스가 맞지 않으면, 일부가 되돌아오는 현상

#### `이유`
: 전송선의 임피던스(보통 50 Ω) 와 수신단(혹은 종단)의 임피던스가 다를 때, 파형의 일부가 반사되어 소스 쪽으로 돌아간다.

#### `결과`
: 신호가 겹치면서 파형이 찌그러지고, 클럭 에지가 흐트러져 오동작을 일으킨다.

### `오버슈트`
: 신호가 목표 전압을 넘어서 튀어오르는 현상

#### `원인`
: 전송선의 반사 에너지가 원래 파형과 합쳐지면 전압이 일시적으로 더 커진다. <br />
그게 `오버슈트`.

#### `문제점`
: 입력 핀의 절대 최대 전압 초과 → ESD 보호 다이오드 손상 <br />
클럭 노이즈 증가 → 신호 왜곡

### `언더슈트`
: 신호가 GND(0V)보다 더 아래로 떨어지는 현상.<br />
즉, 반대로 ‘아래로 튐.’

#### `원인`
: 반사파의 위상이 반대로 겹칠 때 발생. <br />
특히 드라이버의 `구동력(Drive Strength)`이 클 때 잘 나타남.

#### `문제점`
: 입력단 보호 다이오드 역전압 인가 → 수명 단축 <br />
전원 레일에 노이즈 주입 → 시스템 불안정

### `링잉`
: 신호가 목표 전압에 도달한 뒤에도 진동하며 덜덜 떨리는 현상.

#### `원인`
: 전송선의 인덕턴스(L) 와 부하의 커패시턴스(C) 가 LC 공진을 일으키기 때문이다. <br />
즉, 회로가 “미세한 스프링처럼” 진동하는 셈이다.

#### `문제점`
: 신호의 안정화 시간 증가 (settling time) <br />
타이밍 마진 축소 → 오동작 가능

### 2. I2C 의 통신방법

CPU (C8051F023) 의 datasheet 의 185p (SMBus Protocol) 를 보면 알 수 있다. (SMBus는 I2C를 기반으로 만들어진 규격이다.)

여기 써있는 내용을 먼저 살펴보면, <br />
2가지 타입의 데이터 통신이 가능하고, `마스터(MASTER)` 송신기 -> `지정된 슬레이브(Slave)` 수신기의 데이터이동인 `WRITE` , `지정된 슬레이브 송신기` -> `마스터` 수신기로의 데이터이동인 `READ`가 있다. <br />
`MASTER`는 데이터를 전송하고, 클럭을 생성하고, `START` 조건을 생성하고, 주소를 보낸다. 어떠한 장치든 `START`신호와 주소를 보내면 그 순간 `MASTER`가 되고, 고정된 `MASTER`는 없다. <br />
버스에 여러 `MASTER`가 있으면, 반드시 한쪽 `MASTER`가 중재방식을 통해 승리한다. <br />
일반적인 SMBus 데이터 전송은 `START` 조건으로 시작해서, 주소 바이트(7비트 슬레이브 주소 + 1비트 읽기/쓰기 방향) 를 보내고, 그 다음에 하나 이상의 데이터 바이트를 전송한 후 `STOP` 조건으로 끝난다. <br />
응답에는 `ACK`, `NACK` 가 있고, 바이트를 받는 쪽이 ACK/NACK을 낸다.<br />
( `READ`: 데이터를 받는 쪽 = MASTER → MASTER가 ACK/NACK. `WRITE`: 데이터를 받는 쪽 = SLAVE → SLAVE가 ACK/NACK.)
`ACK` 는 잘 받았다고 주는 신호이고, `SDA = LOW` 상태, `CLK = HIGH` 이다. <br />
`NACK`은 마지막 바이트임을 알려주거나 (READ에서) 에러가 났다는 신호이고, `SDA = HIGH`, `CLK = HIGH` 이다. <br />
전송속도는 이 CPU는 SMBus version 1.1 이므로, I2C 표준규격에 의해 최대 100kHz 까지 가능하다. <br />

이정도로 사전정보를 마쳤으니, 이제 `WRITE` 와 `READ` 과정에 대해 살펴보자.

<img width="1865" height="582" alt="Image" src="https://github.com/user-attachments/assets/587eeefd-3278-4cc5-abde-35ba8bccf555" />

### 1. `READ`

위 그림을 보면, 클럭은 0과 1로 진동하고 있다.
1. SDA 핀에서 High -> Low 로 떨어질 때 `START` 조건이 생성된다.
2. `MASTER`가 주소 + R를 전송한다. 이때, `SCL`이 Low일때 `SDA`가 비트를 바꾸고 (WRITE) , `SCL`이 High 일때 `slave`가 그 비트를 읽는다. 
3. `slave`가 `ACK` 응답을 보낸다. (주소가 왔다 !)
4. `slave`가 데이터를 전송한다
5. `master`가 `ACK` 응답을 보낸다. 
6. 4번, 5번을 반복한다.
7. `master`가 `NACK` 응답을 보낸다 (`SCL = HIGH`일 때, `SDA = HIGH` 유지) → "더 이상 데이터 필요없음" 신호 
8. `master`가 `STOP` 신호를 보낸다.

### 2. `WRITE`

`READ`의 그림과 비슷한데, 

1. `master`가 `start` 신호를 생성한다 (`SCL=HIGH`, `SDA=High->Low` 일 때 조건 생성) 
2. `master`가 주소 + W을 전송한다. 이때, `SCL`이 Low일때 `SDA`가 비트를 바꾸고 (WRITE) , `SCL`이 High 일때 `slave`가 그 비트를 읽는다. 
3. `slave`가 `ACK` 응답을 보낸다 (주소왔다 !) (`SCL=HIGH` 에서 `SDA=LOW`로 유지되면 `ACK` , `SDA=HIGH`로 유지되면 `NACK`) (`SDA`를 `high-z` 상태로 놓고, `slave`가 `SDA`를 `LOW`로 끌어당기는 방식.)
4. `master`가 데이터를 전송한다. 
5. `slave`가 `ACK` 응답을 보낸다 
6. 4번, 5번을 반복한다 
7. `master`가 `STOP` 신호를 보낸다.

#### `high-z`
: High Impedance = 고임피던스 = 초고저항 = "전류가 거의 흐르지 못할 만큼 높은 임피던스 상태” — 즉, 전기적으로 단절(open) 된 상태

High-Z는 회로가 “신호선에서 손을 떼는 상태” <br />
다른 장치가 신호를 내보낼 수 있게 라인을 개방(open)하여 버스 충돌을 막는다.

## JTAG

### 분석할 회로도

<img width="795" height="641" alt="Image" src="https://github.com/user-attachments/assets/cf746d76-fd83-4316-97d3-75aed4464be2" />


### `왜 있을까?`
1. 디버깅 : 개발중인 하드웨어의 문제를 해결하고, 코드를 실행
2. 테스트 : PCB의 이상 유무를 검사
3. 프로그래밍 : 마이크로컨트롤러나 FPGA 같은 칩의 펌웨어나 코드를 다운로드

### `회로도 분석`

- `TMS(Test Mode Select)` : 어떤 동작 모드로 들어갈지 선택할 때 쓴다.
- `TDI(Test Data In)` : 디버거 -> CPU 로 데이터 입력할 때 쓴다. 명령이나 데이터를 CPU에 전송한다.
- `TCK(Test Clock)` : JTAG의 클럭신호
- `TDO(Test Data out)` : CPU -> 디버거로 데이터 출력할 때 쓴다. CPU 내부의 정보를 내보낼 떄 쓴다.
-  R5는 TCK 라인을 안정시키는 풀업 저항. 논리 안정화용.

## RS232 Driver (MAX3222EEUP)

<a href = "https://www.alldatasheet.co.kr/datasheet-pdf/download/203248/MAXIM/MAX3222EEUP.html">data sheet 다운로드하러가기</a>

### 분석할 회로도

<img width="1384" height="650" alt="Image" src="https://github.com/user-attachments/assets/bef4e9e4-b9b3-441c-9bce-0af196c6cc73" />

### `왜 있을까?`
: CPU와 다른 장치간에 데이터 송수신을 위해 필요. `UART`가 만든 TTL 레벨(0V / 5V)의 TX/RX (송신, 수신) 신호를 장거리 전송에 적합한 전압 레벨로 변환하는 표준 인터페이스다.

### `UART(Universal Asynchronous Receiver / Transmitter)`
: CPU 내부의 병렬 데이터를 직렬 신호로 변환해 송신하고, 외부에서 들어온 직렬 신호를 병렬 데이터로 복원해 수신하는 하드웨어 회로.

### Feature 분석 (MAX3222EEUP)

<img width="845" height="1199" alt="Image" src="https://github.com/user-attachments/assets/421e95a8-7b69-4187-b26b-92e7ea18d6c5" />

- ESD 보호 (정전기 보호가 된다) (왜있나 ? -> RS-232 포트는 사람이 직접 케이블을 꽂고 빼는 동작이 자주 일어나는데, 손끝의 정전기가 TX, RX 핀으로 순간 유입될 수 있다. 정전기 스파이크는 수천 V를 만들기 때문에, 회로에 들어간다면 매우 치명적임) (사람 몸에서 방전될 수 있는 수준(±15 kV)까지 견딜 수 있음)
- 데이터 신호를 안정적으로 전송할 수 있는 최대속도는 250kbps
- Latchup Free (CMOS 회로가 정전기나 과전압으로 내부가 단락되는 현상이 절대 안일어난다)
- 수신기는 깨어있는 상태로 shutdown 모드 실행하는데, 1μA 전류소모

#### `핵심개념`

- "Asynchronous" (비동기): 별도의 클럭 신호 없이, 송신자와 수신자가 약속된 속도(baud rate)에 맞춰 통신한다.
- "Serial" (직렬): 데이터를 한 비트씩 순서대로 전송한다.

즉, CPU 내부에서 8비트씩 처리되는 데이터를 <br />
UART가 직렬 신호(TX: 전송, RX: 수신)로 바꾸어 외부로 내보내는 것이다.

#### `기본구조`
```css
[CPU Data Bus] ⇄ [UART] ⇄ TX/RX 선 ⇄ [다른 UART]
```

### `TTL(Transistor–Transistor Logic)`
: UART 신호가 실제로 전달되는 ‘전압 레벨’ 규격.

#### `전압 기준`

| 논리            | **TTL / UART 레벨 (3.3V 기준)** | **RS-232 레벨 (±전압 기준)** | 설명                            |
| ------------- | --------------------------- | ---------------------- | ----------------------------- |
| **HIGH (1)**  | 약 3.3 V (또는 5 V)            | 약 −3 V ~ −15 V         | **논리 1**. RS-232에서는 음전압으로 표현됨 |
| **LOW (0)**   | 약 0 V                       | 약 +3 V ~ +15 V         | **논리 0**. RS-232에서는 양전압으로 표현됨 |
| **Idle 상태**   | HIGH (= 3.3 V)              | −전압 (약 −6 V 근처)        | 데이터 없을 때 라인 유지 상태             |
| **Start bit** | LOW (= 0 V)                 | +전압 (약 +6 V 근처)        | 프레임 시작 신호 — 논리 전환 점           |

UART가 만든 TX/RX 신호는 TTL 레벨이다. <br />
문제는 TTL은 ±전압이 아니라 “0~3.3V” 범위라서,
장거리 전송(수 m 이상)에서는 노이즈에 약하다.

그래서 RS-232 드라이버(MAX3222EEUP)가 등장한다. <br />
이 회로가 TTL 신호(0~3.3V) 를 RS-232 신호(±7~10V) 로 변환해서
멀리 있는 장치(PC, 모뎀 등)와 안정적으로 통신할 수 있게 한다.

여기서 조금 더 살펴볼점은, `rs-232` 와 `TTL` 의 논리가 반대라는 점이다. 논리가 반대된다 라는말은, `TTL`에서 1은 전압 높음이고, `RS-232`에서 1은 전압 낮음 (음전압) 이라는 뜻이다. <br />
대부분의 MCU의 출력은 +3.3V 혹은 +5V 이기 때문에, `MAX3222` 같은 칩이 필요한 이유가 전압을 변환해주는 것도 있지만, 논리를 반전시켜주기도 하는 장치여서 꼭 필요하다.


### `회로도 분석`

MAX3222 datasheet 7페이지에 나와있다.

- R1IN : RS-232 수신 입력 (+/- 10V)
- R1OUT : MCU쪽으로 변환된 RX 출력 (3.3V TTL)
- T1IN : MCU의 TX 입력 (3.3V TTL)
- T1OUT : RS-232 송신 출력 (+/- 10V)
- C1+, C1-, C2+, C2- : 내부 charge pump용 커패시터 연결
- V+ : 생성된 + 전압
- V- : 생성된 - 전압
- EN : Receiver Enable (Active Low = Low 일때 수신기 작동)
- SHDN : shutdown (절전모드)
- VCC : 공급전원
- NC : no connection (안쓰는 회로)
- C3 : 생성된 + 전압 평활용 (평활 ? = 전압의 요동(리플)을 줄여서 전압을 매끄럽게 일정하게 만드는 것)
- C4 : 생성된 - 전압 평활용
- BC5 : 앞에서 언급했던 `bypass 필터` or `decoupling 필터`
- R2, R4 : `damping 저항`


#### `왜 +전압, -전압 둘다 있지 ?`

전압은 전하를 움직이는 힘이다. 여기서 "힘"을 "힘 자체"로 보면 안되고, `전위차` , 즉, 전하가 어느 방향으로 움직일지를 결정짓는 “언덕의 높이 차이”라고 보는 게 더 정확하다. <br />
예를 들어, <br />
```
A점 = +5V
B점 = 0V (GND)
```
이라면, 전하(+전자)가 A → B 방향으로 흐른다.
왜냐면 전자는 “낮은 전위 쪽으로 굴러가려는” 성질이 있기 때문이다. <br />
그래서 아무튼, `rs-232` 에서는 내부적으로 <br />

```
TX High 상태 → T1OUT = -6V  
TX Low 상태  → T1OUT = +6V
```

이렇게 동작한다. 신호의 두 가지 상태(High / Low)를 전압의 높낮이가 아니라 전류의 방향(극성)으로 표현한다.

RS-232가 ±전압을 쓰는 이유는, <br />
① 장거리 전송에서도 노이즈 영향을 줄이고, <br />
② 서로 다른 GND 간 전위차(전압의 차이)를 흡수하며, <br />
③ 유휴 상태(사용되지 않을때의 상태)를 안정적으로 유지하기 위해서다.

그래서 “TX High = −전압”, “TX Low = +전압”으로 동작하며,
논리 기준이 TTL과 반대다.



### 추가 개념 사항

### 1. RS-232 통신방법

- 비동기 (클럭이 없는) 직렬 통신이다 <br />
-> 데이터는 한줄(TX)로 `1비트씩 순서대로` 보내지고, 수신기는 하강 에지 (HIGH -> LOW)를 기준으로 타이밍을 계산해서 데이터를 읽는다. <br />
예를들어, 전송속도 (Baud rate) 가 9600bps(1초에 9600개의 비트를 전송)에 8N1( 8 = 데이터 비트 수 , N = No parity (패리티 없음 = 짝수 홀수 검증 없음) , 1 = Stop 비트 수) 이라고 한다면, <br />
한 비트가 차지하는 시간 = 1/9600 = 0.000104166..초 = 104.17μs (마이크로초) 이고, 이는 한 비트의 유지시간을 나타낸다. <br />
따라서, 하나의 문자를 보내는데 걸리는 시간은 10 bits (start bit까지 10 bits) * 104.17μs = 1.04ms 이다.


#### `통신 상태`

| 설명           | 상태   | 설명   |
| ------------  | -------------- | ---- |
| **Idle**      | 1 (HIGH) | 대기상태 |
| **start bit** | 0 (LOW)  | 전송 시작 신호 |
| **data bits** | 0 or 1 | 실제 데이터 |
| **stop bit**  | 1 (HIGH) | 프레임 종료 |


#### `그림으로 살펴보기`

<img width="1348" height="467" alt="Image" src="https://github.com/user-attachments/assets/c802572d-4582-4baa-8e5b-48d05b725c9e" />

위 그림은 'A' (0x41 = 0b01000001)를 전송하는 그림이다. <br />
하강 에지 (HIGH -> LOW)를 기준으로 start bit가 생성되고, 104.17μs 마다 데이터를 전송한다. 8개의 데이터 비트의 전송이 끝나면, 상승 에지 (LOW -> HIGH)를 기준으로 stop bit가 생성된다. <br />
`rs-232`는 LSB First (Least Significant Bit First) 방식이므로, 작은 비트 -> 큰 비트 순으로 데이터를 전송한다. 수신기는 각 비트의 중앙에서 값을 샘플링하므로, 타이밍 오차에 강하고 안정적인 수신이 가능하다. <br />

<img width="1300" height="628" alt="Image" src="https://github.com/user-attachments/assets/7beb27cc-c09c-4475-8d13-0ff59bab4b47" />

위 그림은 수신기 동작 원리이다. <br />
1. IDLE 상태 감지 - TX 라인이 HIGH 유지
2. Start Bit 감지 - HIGH → LOW 하강 에지 감지
3. 타이밍 계산 - Baud rate로 비트 간격 계산
4. 비트 샘플링 - 각 비트 중앙에서 값 읽기 (가장 안정적)
5. 데이터 수집 - 8개 비트 수집 (LSB → MSB)
6. Stop Bit 확인 - HIGH인지 검증
7. 프레임 완료 - 다음 Start Bit 대기

## Buffer

### 분석할 회로도

<img width="1590" height="1103" alt="Image" src="https://github.com/user-attachments/assets/37f530bc-7497-44ad-8db6-ab3fa128c68f" />

### `왜 있을까 ?`

- 여러 장치를 동시에 구동하면, 긴 선을 연결할 때 신호가 약해지는데, 버퍼가 신호를 증폭해서 강하게 만들어준다.
- 또한 외부 회로에서 문제가 생겨도 CPU를 보호해주는 방패 역할을 해준다

### `회로도 분석`

- 입력단자 8개, 출력단자 8개
- R20 ~ R23 풀업저항을 달아놨다. 풀업저항으로 HIGH 고정시켜서 안정화에 쓰인다. (floating 방지)

## power

### 분석할 회로도

<img width="2090" height="873" alt="Image" src="https://github.com/user-attachments/assets/0dcb0e54-dfa6-4047-a7e1-45969d8d1573" />

### `왜 있을까 ?`

- 외부에서 5V 공급받아서 3.3V 로 변환시켜서 내보내준다. (다른곳의 필요전압이 3.3V 이므로)

### `회로도 분석`

- J4 : 전원 입력 커넥터
- U5 : LDO 레귤레이터이다. 5V 를 3.3V로 변환해준다. ([LDO vs switching 방식](./cpu회로.md#1-dc-dc-의-ldo-vs-switching))
- L1 (NLC453232T-1R0K) : 전원 라인에서 고속 스위칭이나 전류 급변으로 발생하는 EMI(전자파 간섭) 를 억제하기 위한 페라이트 비드(ferrite bead) 로, 고주파 노이즈를 제거하는 고주파 필터 소자.
- F1 (퓨즈) : 과전류가 흐르는것을 막아준다. 비정상적으로 큰 전류가 흐르면 선을 끊어버려서 막는다. (정격(정해진 규격 전류) 3A일 때, 3A 이상이 흐르면 퓨즈 끊김) 과전류 보호, 부품 보호, 안정성의 목적.
- C7 : 입력 전압 안정화
- C8 : 출력 전압 안정화
- BC9 : bypass필터
- C9 : 최종 출력 대용량 안정화
- 아래 전원표시 LED회로가 있다. 시각적 확인용을 위하여 달려있다.

### 추가 개념 사항

### 1. DC-DC 의 LDO vs Switching 

#### `DC와 AC의 차이`

<img width="2470" height="880" alt="Image" src="https://github.com/user-attachments/assets/a56c2f34-1b00-4636-ab68-42fcfb34c8c2" />

DC(직류)는 전류방향이 한 방향으로 일정하게 흐른다. (직선형태) 대부분의 전자부품은 DC 전원을 사용한다. <br />


<img width="2473" height="866" alt="Image" src="https://github.com/user-attachments/assets/88fcda96-993c-48a2-9859-b854c13eb6c8" />

AC(교류)는 전류의 방향과 크기가 주기적으로 바뀐다. (sin 파형) 전력 송전이나 변압에 유리하다.

#### `DC-DC 변환이 필요한 이유`

시스템에는 다양한 전압이 필요하다. <br />

| 부품    | 필요 전압       |
| ----- | ----------- |
| MCU   | 3.3V        |
| FPGA  | 1.0V ~ 1.8V |
| RF 회로 | 5V          |
| 센서    | 3.3V or 5V  |


전원은 하나(예: 12V 어댑터)인데, 이걸 각 부품에 맞는 전압으로 “변환”해야 한다. <br />
→ 그래서 DC-DC 컨버터가 필요하다.

#### `그래서 LDO vs switching ?`

#### `LDO (Low Dropout Regulator)`

입력 전압을 낮은 전압으로 “직접” 줄이는 선형 방식의 DC-DC 변환기

⚙️ 동작 원리

내부 트랜지스터로 전압을 “저항처럼 떨어뜨린다.” <br />
즉, 남는 전압을 열로 버림 → P(loss) = (Vin - Vout) × Iout

🔹 예시

12V → 5V, 1A 부하일 때 <br />
→ (12 - 5) × 1 = 7W 손실 (발열)

🔹 장점

- 회로 단순

- 출력 리플(노이즈)이 거의 없음

- 응답 속도 빠름 (Transient Response 좋음)

🔹 단점

- 전압차가 크면 발열 심함

- 효율 낮음
(예: 12V→3.3V 변환 시 효율 ≈ 27%)

🔹 주 용도

- 저전류, 저노이즈가 중요한 회로 (RF, ADC, 센서 전원 등)

#### `switching Regulator`

트랜지스터를 고속으로 ON/OFF 스위칭 하여 평균 전압을 제어하는 방식. <br />
내부 인덕터(L)와 커패시터(C)가 에너지를 저장·방출하며 원하는 전압을 만들어낸다.

⚙️ 원리 요약

1. 트랜지스터 ON → 인덕터에 에너지 충전

2. 트랜지스터 OFF → 인덕터의 에너지가 부하로 방출

3. ON/OFF 비율(Duty ratio)을 조절하여 평균 출력 전압 결정
→ Vout = Vin × Duty (Buck일 때)

🔹 장점

- 고효율 (보통 85~95%)

- 발열 적음

- 입력전압이 높고 출력전류가 큰 경우 적합

🔹 단점

- 회로 복잡

- 스위칭 동작에 따른 리플·EMI 노이즈 발생

- 외부 필터 소자 필요 (L, C, 다이오드 등)

🔹 노이즈 종류

- 리플(Ripple) : 스위칭 주파수에 따른 주기적 파형 잔류

- EMI(Electromagnetic Interference, 전자파 간섭) : 고속 스위칭이나 전류 급변으로 인해 발생하는 방사(Radiated)/전도(Conducted) 형태의 전자기 간섭 현상. (`방사형 EMI` : 공기 중으로 퍼져나가는 전자파 간섭, `전도형 EMI` : 전원선이나 신호선을 통해 전기적으로 퍼지는 간섭)

🔹 주 용도

- 고전류 구동 회로 (FPGA, CPU 전원)

- 배터리 기반 시스템

- 효율이 중요한 휴대용 장비

### `LDO Regulator` , `Switching Regulator` Feature 분석

### `LDO Regulator Features` (EZ1085)

<a href = "https://www.alldatasheet.co.kr/datasheet-pdf/download/42415/SEMTECH/EZ1085.html">datasheet 다운로드 받으러 가기</a>

<img width="1063" height="650" alt="Image" src="https://github.com/user-attachments/assets/cc9f7135-19e6-449d-83ea-f6971fc7bab6" />


- 입력전압이 출력보다 최소 1.3V 가 높아야한다. (출력 5V를 쓰고싶다면, 최소 6.3V 이상 넣어줘야함)
- 라인(입력전압)과 온도가 바뀌어도 전류를 안정적으로 낼 수 있다.
- 부하가 갑자기 변할 때 (전원을 소비하는 쪽의 전류) 출력전압이 출렁이지 않고 빠르게 복원된다.
- 라인(임력전압), 로드(부하전류), 온도가 변해도 출력전압이 목표값 (+/-)2% 이내로 유지된다.
- Adjustable 타입 가능. ADj 핀에서 나오는 전류가 최대 90μA 라서 출력 전압이 정확히 유지된다. (Adjustable ? : Fixed는 3.3V / 5V 처럼 정해진 전압, Adjustable은 3.7V 등 일반적으로 쓰지 않는 전압을 출력해주고 싶을 때)
- 입력전압이 바뀌어도 출력변화가 0.015% 수준 (매우 낮다)
- 부하전류가 변할 때 출력이 변하는 비율은 0.05% 수준 (매우 낮다)

### `Switching Regulator Features` (LM2596)

<a href = "https://www.alldatasheet.co.kr/datasheet-pdf/download/2032213/ONSEMI/LM2596.html">datasheet 다운로드 받으러 가기</a>

<img width="873" height="638" alt="Image" src="https://github.com/user-attachments/assets/c9aaa2f2-cd4b-4478-898e-cdf4e385186f" />

- 출력전압 1.23V ~ 37V 까지 설계
- 3A 연속 전류를 버틸 수 있도록 설계
- 40V까지 입력전압으로 받을 수 있다.
- 내부에 스위칭 트랜지스터를 1초에 150000번 켜고 끄는 오실레이터 존재
- TTL 논리 레벨 신호로 칩을 켜고 끌 수 있다.
- shutdown 상태 소비전류 평균 80μA
- 과열보호, 과전류 보호 기능 있음
- 보상 네트워크가 내부에 설계 (보상 네트워크 ? : 스위칭 회로가 흔들리지 않게 잡아주는 안정화 회로. 보통 외부에 콘덴서와 저항을 달아서 조절하는데, 내부에 설계되어 있어서 안정도 계산 없이 부품값만 맞추면 된다는 의미)
- 습기에 영향을 받지 않음. (MSL = 1) (`MSL = 1` : 영향 거의 없음. `MSL = 2~5` : 건조 필요 `MSL 6` : 매우 민감)
- 납 포함 X


## Connector

### `왜 필요한가 ?`

-> 외부 장치와의 연결 (CPU 보드와 다른 모듈/장치 연결) 할때 필요하다. <br />
각 기능별로 분리된 커넥터가 있다.

### `회로도 분석`

#### `CON RS-232`

<img width="806" height="316" alt="Image" src="https://github.com/user-attachments/assets/8610e9fa-e5a7-4722-a23e-f97e6e3e5aa2" />


- rs232 커넥터
- TX, RX (송신, 수신)선 존재
- TXD(송신) : MAX3222 T1OUT 연결
- RXD(수신) : MAX3222 R1IN 연결

#### `용도`
- PC나 외부 장치와 시리얼 통신
- 디버깅, 설정, 제어 명령 송수신
  
#### `신호 레벨`
- RS-232: ±12V (또는 ±5.5V)
- MAX3222가 TTL(3.3V) ↔ RS-232 변환

#### `회로 흐름`

```
CPU(UART) ─ TTL(3.3V) -→ MAX3222 ─ RS-232(±12V) -→ CON RS-232 -→ 외부 장치
         ←- TTL(3.3V) ←-         ←- RS-232(±12V) ←-            ←-
```

#### `CON DN I2C`

<img width="822" height="270" alt="Image" src="https://github.com/user-attachments/assets/59f6d12d-6b03-4e57-87a9-026aad3de296" />


- I2C 통신 커넥터
- SDA, SCL (데이터, 클럭)선 존재

#### `CON select`

<img width="1106" height="551" alt="Image" src="https://github.com/user-attachments/assets/b9f707c0-67e1-4a85-8216-b441005d8a75" />


- LOC / REM (Local , Remote 모드 선택) (`Local` : 장비 패널의 버튼/스위치로 직접 제어, `Remote` : 외부 제어 시스템에서 명령으로 제어)
- C_DN_PATH_SEL : 하향 경로 선택
- C_UP_PATH_SEL : 상향 경로 선택
- LED A/B : 각 경로의 상태 LED (상향 경로, 하향 경로)
- **풀업저항 10kΩ 역할:**
     - 스위치 OPEN → HIGH (3.3V) 안정적
     - 스위치 CLOSE → LOW (0V) 안정적
     - Floating 방지 → 노이즈 내성 확보


### 블록 다이어그램 그려보기

<img width="2107" height="1445" alt="Image" src="https://github.com/user-attachments/assets/108017fe-4a85-4a59-91f0-0cca7092a9fb" />