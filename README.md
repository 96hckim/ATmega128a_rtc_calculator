# ATmega128A 기반 I2C RTC 시계 및 복합 계산기 시스템

> **데이터시트 분석 및 I2C 통신 프로토콜 구현을 통한 비차단 멀티태스킹 임베디드 시스템 통합 프로젝트**

---

## 1. 프로젝트 개요

* **목적**: 칩셋 규격서 레벨 해석을 통한 하드웨어 제어, I2C 통신 구현, 1ms 타이머 인터럽트 기반 비차단 스케줄러 내 복합 계산기 파싱 엔진 및 RTC 시계 기능 최적화 통합
* **환경**: ATmega128A (8-bit AVR), Microchip Studio, I2C, 4-bit Parallel Bus, 오실로스코프 (하드웨어 파형 디버깅)

---

## 2. 하드웨어 구성 (H/W)

| 시스템 결선 외관 (Breadboard) | 시스템 논리 회로도 (KiCad) |
| :---: | :---: |
| ![Breadboard](assets/images/breadboard.jpg) | ![Schematic](assets/images/schematic.png) |

* **부품 명세**: LCD1602 (시각 출력), DS1307 (I2C RTC 데이터 공급), 4x4 Keypad (수식 입력), Push Switch x5 (모드 제어 및 수정)

---

## 3. 소프트웨어 설계 및 주요 기능 (S/W)

### ⚙️ 3.1 시스템 아키텍처 및 흐름 제어
* **FSM**: 시계 ↔ 시간 수정 ↔ 계산기 모드 유기적 전환 및 **1분간 미사용 방치 시 시계 자동 복귀** 예외 처리 엔진 탑재
* **계층 구조**: `main.c`(스케줄러) → `ui.c`(표현층) → `cal.c`(연산 로직층) → HAL 드라이버 구조로 설계하여 계층 간 결합도 최소화

| 유한 상태 머신 (System FSM) | 파일 단위 계층 구조도 (Architecture) |
| :---: | :---: |
| ![FSM](assets/images/system_fsm.png) | ![Architecture](assets/images/project_architecture.png) |

---

### 📺 3.3 핵심 모듈 구현 및 하드웨어 파형 분석
* **LCD1602 (4-bit)**: 전원 인가 초기화를 위한 Software Reset(`0x30` 3회 주입) 및 1.64ms 실행 타이밍을 준수하여 상/하위 니블 분할 전송 안정화
* **DS1307 (I2C rtc)**: BCD-10진수 포맷 변환 및 CH(Clock Halt) 비트 제어를 통한 실시간 버스 데이터 동기화
* **복합 계산기**: 연산자 우선순위 및 소괄호 연산 처리를 위한 시스템 스택 기반 **재귀 하향 파서(Recursive Descent Parser)** 엔진 자력 설계

| 1. 주소 지정 파형 (`0xD0->0x00`) | 2. READ 시작 파형 (`0xD1->0x54`) | 3. 분/시/요일/일 읽기 파형 |
| :---: | :---: | :---: |
| ![i2c_wave1](assets/images/i2c_wave1.png) | ![i2c_wave2](assets/images/i2c_wave2.png) | ![i2c_wave3](assets/images/i2c_wave3.png) |
| **4. 월/년 읽기 및 마감 파형** | **5. I2C 버스 종합 파형** | **6. 최종 LCD 표출 결과** |
| ![i2c_wave4](assets/images/i2c_wave4.png) | ![i2c_wave5](assets/images/i2c_wave5.png) | ![rtc_result](assets/images/rtc_result.png) |

---

## 4. 트러블슈팅 및 구현 후기

* **LCD 하드웨어 시프트 타이밍 최적화**
  * 명령어의 물리적 실행 시간(1.64ms) 제약을 고려하여 4비트 인터페이스의 타이밍 동기화를 정밀하게 제어함으로써, 긴 수식 입력 시 화면이 부드럽게 밀리는 하드웨어 스크롤 기능을 성공적으로 구현했습니다.
* **오버플로우 방지 및 자체 출력 버퍼 설계**
  * 정수 범위 한계를 극복하기 위해 내부 연산을 64비트 확장 자료형(`int64_t`)으로 설계하고, 컴파일러의 출력 제한을 자체 구현한 문자열 변환 버퍼 함수(`int64_to_str`)로 제어하여 큰 자릿수도 깨짐 없이 정상 계산되도록 완성했습니다.
* **데이터시트 및 I2C 통신 후기**
  * 데이터시트의 타이밍 스펙을 엄격히 준수하는 것이 시스템 신뢰성의 핵심임을 깨달았고, 오실로스코프로 I2C 버스의 실시간 BCD 파형을 직접 계측 및 분석해 보며 임베디드 통신 제어의 깊이를 더한 값진 경험이었습니다.

---

## 5. 시스템 구동 시연 동영상

| 1. Reset 동작 시연 | 2. 계산기 대기 동작 시연 |
| :---: | :---: |
| [Reset 동작 영상 보기](https://github.com/user-attachments/assets/b78caf9c-ed95-4711-8a38-911cbf39adca) | [계산기 대기 영상 보기](https://github.com/user-attachments/assets/9526dc14-fb07-40c4-8e4b-e8f7e06477e3) |
| **3. 계산기 Shift 동작 시연** | **4. 시스템 전체 동작 시연** |
| [계산기 Shift 영상 보기](https://github.com/user-attachments/assets/d007d798-954c-4ccf-9ef9-4953d11a8a9e) | [전체 동작 영상 보기](https://github.com/user-attachments/assets/0f2ee930-3377-46e6-80b0-7c4bed6878ca) |
