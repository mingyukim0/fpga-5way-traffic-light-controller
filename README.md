# fpga-5way-traffic-light-controller
🚦 FPGA 기반 5차로(오거리) 신호등 제어기 설계  
Verilog HDL · FSM 설계 · Vivado Simulation · FPGA 실물 구현

실제 **공덕오거리(5-Way Intersection)** 구조를 기반으로,  
도로 간 신호 충돌을 방지하는 복잡한 신호 제어 FSM을 직접 설계하고  
Verilog HDL로 구현한 FPGA 기반 고난도 디지털 시스템 프로젝트입니다.

Vivado/Active-HDL 시뮬레이션, Clock Divider 설계, Testbench 기반 디버깅,  
브레드보드 LED 회로를 통한 실물 검증까지 수행한 캡스톤 프로젝트입니다.

---

# 📘 프로젝트 개요

일반적인 2차로 또는 4차로가 아닌,  
**5개의 도로가 만나는 실제 오거리(5-Way Intersection)** 를 제어하기 위한  
신호등 FSM을 Verilog HDL로 설계하고 FPGA로 구현하였습니다.

프로젝트 핵심 요소는 다음과 같습니다:

- 5차로 신호 흐름 분석 및 그룹화 설계  
- FSM(상태도) 기반 신호 제어 로직 구현  
- Verilog HDL 기반 모듈 설계  
- Clock Divider 및 타이밍 제어  
- Vivado/Active-HDL Simulation 수행  
- FPGA + 브레드보드 LED 회로 구현  

---

# 🛣 5차로 교차로 설계 난이도

일반 신호등 예제보다 훨씬 복잡한 이유는 다음과 같습니다:

### 🔹 1) 도로 수 증가 → 상태 폭증  
직진/우회전 포함 시 상태가 기하급수적으로 증가함

### 🔹 2) 특정 도로는 세트로 제어해야 함  
- 1·4번 도로 그룹  
- 2·5번 도로 그룹  
- 3번 도로는 독립 직진 전용

### 🔹 3) 우회전 경로가 복잡  
1→2, 1→3, 4→5, 5→1 등

### 🔹 4) 신호 충돌 방지 필요  
특정 조합끼리는 절대 동시에 GREEN일 수 없음

이를 해결하기 위해 FSM을 **도로 그룹 기반**으로 재설계했습니다.

---

# 🔧 FSM 설계

최종 신호 전환 순서는 다음과 같습니다:

**YYY → GRR → YRR → RGR → RYR → RRG → RRY → GRR 반복**

상태도 이미지는 `/docs/fsm_diagram.png` 에 추가합니다.

---

# 💻 Verilog HDL 코드

본 프로젝트의 전체 Verilog 코드는 `/src` 디렉토리에 포함됩니다:

- [`traffic.v`](./src/traffic.v)
- [`top_module.v`](./src/top_module.v)
- [`clock_divider.v`](./src/clock_divider.v)
- [`testbench.v`](./src/testbench.v)

아래는 FSM 상태 전환을 보여주는 핵심 코드 일부입니다:

```verilog
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
```

전체 코드는 `/src` 폴더에서 확인할 수 있습니다.

---

# 🧪 Simulation 결과
Vivado/Active-HDL 시뮬레이션을 통해
각 상태가 타이밍에 맞게 정확히 전환되는 것을 확인했습니다.

Waveform 이미지는 /docs/simulation_waveform.png 에 추가합니다.

---

# 🔌 핀 구성(Pin Mapping)
PMOD JA 포트를 기반으로 신호등 LED를 매핑했습니다.

핀 매핑 표 이미지는 /docs/pinmap.png 에 추가합니다.

---


# 🔧 FPGA + 브레드보드 구현
FPGA 보드의 출력 신호를 브레드보드 LED 회로로 연결해
신호등 패턴이 정상적으로 작동하는 것을 검증했습니다.

하드웨어 사진은 /docs/hardware_photo.png 에 추가합니다.

---

# ⭐ 배운 점 (문제 → 해결 → 성장)
🔹 1) 상태 폭증 문제 → 도로 그룹화 설계로 해결
복잡도가 높아 충돌이 발생했으나
1·4 / 2·5 / 3번 도로 그룹으로 재구성하여 문제 해결.

🔹 2) Yellow 타이밍 오류 → Clock Divider/Counter 재설계
상태별 OnTime을 분리하고 TimeCnt 초기화 로직을 도입해
실제 신호등과 동일한 타이밍 확보.

🔹 3) Testbench 문제 → Reset/Counter 타이밍 수정
상태가 반복되는 오류를 reset 길이 조정 및 counter 동작 수정으로 해결.

🔹 4) LED 점등 오류 → 핀 매핑 재확인
FPGA 실제 핀과 LED 위치 불일치 문제를 pinmap 문서 기반으로 재배선하여 해결.

---

# 🎯 최종적으로 배운 핵심 역량
복잡한 실물 시스템을 FSM으로 추상화하는 능력

Verilog 기반 디지털 회로 설계 및 디버깅 능력

Simulation 기반 원인 분석 및 문제 해결 능력

FPGA–브레드보드 연동 하드웨어 구현 능력

모듈 설계, 분주회로 설계 등 임베디드 실전 경험

---

# 🎖 관련 교육 수료증
과정명: Vivado를 활용한 AMD 응용회로설계 프로젝트

교육기간: 2023.03.05 ~ 2023.12.08

교육기관: AMD Korea · 리버트론 · 서일대학교

수료증 이미지는 /certificate/amd_certificate.png 에 추가합니다.
