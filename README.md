# fpga-5way-traffic-light-controller
🚦 FPGA 기반 5차로(오거리) 신호등 제어기 설계  
Verilog HDL · FSM 설계 · Vivado Simulation · FPGA 실물 구현

실제 **공덕오거리(5-Way Intersection)** 구조를 기반으로,  
도로 간 신호 충돌을 방지하는 복잡한 신호 제어 FSM을 직접 설계하고  
Verilog HDL로 구현한 FPGA 기반 고난도 디지털 시스템 프로젝트입니다.

Vivado/Active-HDL 시뮬레이션, Clock Divider 설계, Testbench 기반 디버깅,  
브레드보드 LED 회로를 통한 실물 검증까지 수행한 캡스톤 프로젝트입니다.

# 📘 프로젝트 개요

일반적인 2차로 또는 4차로가 아닌,  
**5개의 도로가 만나는 실제 오거리(5-Way Intersection)** 를 제어하기 위한  
신호등 FSM을 Verilog HDL로 설계하고 FPGA로 구현하였습니다.

5개 도로의 **직진/우회전 흐름**,  
각 도로 사이의 **충돌 관계**,  
시간 기반 상태 전환 등을 모두 고려하여  
실제 신호등 패턴과 유사한 흐름을 구현하는 것이 목표였습니다.

본 프로젝트는 다음 핵심 요소를 포함합니다:

- 5차로 신호 흐름 분석 및 그룹화 설계  
- FSM(상태도) 기반 신호 제어 로직 구현  
- Verilog HDL 기반 모듈 설계  
- Clock Divider 설계 및 타이밍 제어  
- Vivado/Active-HDL 기반 Simulation 수행  
- FPGA + 브레드보드 LED 회로 구현  

---

# 🛣 5차로 교차로 설계의 난이도

일반 신호등 예제보다 훨씬 복잡한 이유는 다음과 같습니다:

### 🔹 1) 도로 수 증가 → 상태 폭증  
5개 도로 × 직진/우회전 여부 × 충돌 관계  
→ 상태(FSM)가 단순 증가가 아닌 **기하급수적으로 증가**

### 🔹 2) 특정 도로는 세트로 움직여야 함  
- 1·4번 도로가 같은 흐름  
- 2·5번 도로가 같은 흐름  
- 3번 도로는 단독 직진만 가능

### 🔹 3) 우회전 조건 존재  
- 1 → 2, 1 → 3  
- 4 → 5  
- 5 → 1  

### 🔹 4) 신호 충돌 방지 필요  
예: 3번 도로 GREEN 시 모든 도로 RED 필수

이 요구사항들을 해결하기 위해 FSM을 **도로 그룹화 방식**으로 재설계했습니다.

---

# 🔧 FSM 설계

최종 FSM(상태도)은 다음 순서를 따릅니다:

**YYY → GRR → YRR → RGR → RYR → RRG → RRY → GRR 반복**

상태도는 `/docs/fsm_diagram.png` 에 삽입하면 됩니다.  
(원본 PDF 4페이지 기반)

---

# 💻 Verilog HDL 주요 코드

```verilog
parameter YYY=3'b000, GRR=3'b001, YRR=3'b010, RGR=3'b011,
         RYR=3'b100, RRG=3'b101, RRY=3'b110;

always @(posedge clk or negedge rst) begin
  if (!rst) begin
    state <= YYY;
    TimeCnt <= 0;
  end else begin
    case (state)
      GRR: if (TimeCnt == GRRTime) state <= YRR;
      YRR: if (TimeCnt == YRRTime) state <= RGR;
      RGR: if (TimeCnt == RGRTime) state <= RYR;
      RYR: if (TimeCnt == RYRTime) state <= RRG;
      RRG: if (TimeCnt == RRGTime) state <= RRY;
      RRY: if (TimeCnt == RRYTime) state <= GRR;
    endcase
  end
end
전체 코드는 /src/ 디렉토리에 포함합니다.

🧪 Simulation 결과
Vivado/Active-HDL 시뮬레이션을 통해
각 상태가 타이밍에 맞게 정확히 전환되는 것을 확인했습니다.

Waveform 이미지는 /docs/simulation_waveform.png 에 추가.

🔌 핀 구성(Pin Mapping)
PMOD JA 포트를 사용해 신호등 LED를 매핑했습니다.

핀 매핑 표는 /docs/pinmap.png 또는 PDF 10페이지를 참고.

🔧 FPGA + 브레드보드 구현
FPGA 보드의 출력 신호를 LED 신호등 회로에 연결하여
실제로 점등 순서가 정상적으로 작동함을 검증했습니다.

구현 사진은 /docs/hardware_photo.png 에 추가.

⭐ 배운 점 (문제 → 해결 → 성장)
본 프로젝트는 실제 오거리 교차로 제어라는 고난도 시스템을 설계/구현하면서
많은 문제를 해결한 경험을 포함합니다.

🔹 1) 상태 폭증 문제 → 도로 그룹화로 FSM 재설계
초기 설계는 충돌이 많았으나,
1·4 / 2·5 / 3번 도로 그룹으로 묶어 상태 수를 효과적으로 관리했습니다.

🔹 2) Yellow 전환 타이밍 오류 → Clock Divider + TimeCnt 개선
상태별 OnTime 분리
TimeCnt 초기화 추가
정확한 1Hz 생성으로 실제 신호등과 유사한 타이밍 확보

🔹 3) Testbench 문제 → Reset/Counter 타이밍 수정
초기 시뮬레이션에서 상태가 꼬이는 문제가 있었으나
reset 길이, counter 증가 타이밍을 수정하여 해결

🔹 4) LED 신호 반전 문제 → 핀 매핑 재검증
FPGA 실제 핀번호와 브레드보드 LED가 뒤바뀌는 문제가 있었으나
매핑 테이블을 기반으로 다시 배선하여 정상 동작 구현

🎯 최종적으로 배운 핵심 역량
복잡한 실물 시스템을 FSM으로 추상화하는 능력

Verilog 기반 회로 설계 및 디버깅 경험

Simulation 기반 분석 능력

FPGA-브레드보드 연동 하드웨어 구현 능력

모듈 분리, 분주회로 설계 등 실전 디지털 설계 스킬

🎖 관련 교육 수료증 (AMD FPGA·ASIC 과정)
과정명: Vivado를 활용한 AMD 응용회로설계 프로젝트

기간: 2023.03.05 ~ 2023.12.08

기관: AMD Korea · 리버트론 · 서일대학교
