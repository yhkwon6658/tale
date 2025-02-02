---
layout: post
title: "ASIC 설계 레시피"
author: "Yonghwan Kwon"
tags: "tools"
comments: true
excerpt_separator: <!--more-->
---

## 📝 블로그 정리 & 업데이트
블로그 정리를 안 한지 거의 **2~3년**이 다 되었네요. 정말 오랜만에 기존 포스트들을 정리하고, **프로필도 업데이트**하였습니다. 그동안 **과제나 개인 연구**로 참 바쁘게 살아왔습니다. 작년 말부터 **논문 서브미션**을 시작했고, 올해는 **여러 편의 논문을 제출**할 예정입니다.  
🙏 부디 좋은 결과가 있기를 바랍니다.  

작년에는 **Tape-out**을 완료했고, 올해는 **두 번째 칩을 제작**하게 되었습니다. 이 포스트에서는 **ASIC 설계 전반**에 대한 이야기를 담으려고 합니다. 다만, **NDA 문제**로 인해 모든 스크립트를 공개하기는 어렵습니다.  

> 저는 **65nm, 28nm PDK**를 사용하고 있기에,  
> 실제 **Industry 수준**과 비교하면 많이 떨어질 수도 있습니다.  

그럼에도 불구하고, 학교에서는 **ASIC 설계/검증/측정 등 전 과정을 제대로 가르치기 어려운 현실**이 있습니다. 때문에 학부생이나 석사 학생들은 **ASIC 설계를 해보고 싶어도**, 채용 공고에서 이야기하는 **PI / PD 등의 직무에서 무슨 일을 하는지 정확히 알기 어려울 것**입니다.  

이제 막 시작하는 **초심자들을 위한 글**을 쓴다는 생각으로,  
시간이 날 때마다 틈틈이 이 포스팅을 마무리 지을 계획입니다.  
<!--more-->

---

## 🔎 ASIC 설계의 시작
ASIC이라고 하면 많은 분들이 **합성(Synthesis)**이나 **PnR(Place & Route)** 등을 떠올리실 겁니다.  
하지만 **설계에서 가장 중요한 것은 "동작하는 시스템을 만드는 것"**입니다.  

따라서, 설계의 첫 단계는 **측정 시스템을 구축하고, Emulation을 수행하는 것**입니다.  
이 단계를 마치면 **본격적으로 칩을 제작하는 과정**으로 넘어가게 됩니다.  

---

## 🚀 ASIC 설계 단계 개요
이제부터의 포스팅에서는 다음과 같은 순서로 ASIC 설계 과정을 다룰 예정입니다.

1️⃣ **Emulation**  
2️⃣ **Library Preparation**  
3️⃣ **RTL Simulation**  
4️⃣ **Synthesis**  
5️⃣ **Gate-level Simulation**  
6️⃣ **Power Estimation**  
7️⃣ **Physical Implementation**  
8️⃣ **STA (Static Timing Analysis)**  
9️⃣ **Post-Layout Simulation**  
🔟 **Physical Verification**  
🔟+1️⃣ **Chip Finishing**  
🔟+2️⃣ **Packaging**  
🔟+3️⃣ **PCB Development**  
🔟+4️⃣ **Measurement**  

---

## 🏁 들어가기에 앞서
본격적인 내용을 다루기에 앞서, 먼저 기초적인 내용을 짚고 넘어가려 합니다.  
전자공학이나 반도체공학을 전공했거나 공부 중인 분이라면 **Linux OS**를 사용해 본 경험이 있으실 것입니다.  
하지만, **Red Hat**이나 **CentOS**에 대해 깊이 이해하고 있는 분들은 많지 않을 것입니다.  

우리에게 익숙한 **Windows나 macOS에서 ASIC 설계 툴을 사용할 수 있다면** 좋겠지만,  
안타깝게도 앞으로 다룰 거의 모든 **EDA 툴들은 CentOS 7 환경에서 동작합니다.**  

따라서, **CentOS의 기본적인 터미널 명령어**, **Bash 스크립트 작성법**, **Makefile 작성법**  
등에 대해 사전에 익히거나, 툴 사용법을 배우는 과정에서 틈틈이 공부해 두는 것이 필요합니다.  

<div style="text-align: center;">
  <img src="https://github.com/user-attachments/assets/3716a39b-a2ee-4d95-aea3-d73bcbe74aed" 
       alt="System Diagram" height="300">
  <p style="text-align: center;"><em>그림 1. 원격 시스템 구성도</em></p>
</div>

<!-- <div style="text-align: center;">
  <img src="https://github.com/user-attachments/assets/31a6caf3-5123-4b4c-a6ab-5a480e9e20d6" 
       alt="System Diagram" height="300">
  <p style="text-align: center;"><em>그림 1. 원격 시스템 구성도</em></p>
</div> -->

**그림 1**은 일반적으로 ASIC 관련 툴을 사용하는 시스템 구성을 나타냅니다.   
Local의 사용자는 **Mobaxterm**이나 **Putty** 같은 툴을 이용해   **SSH 방식**으로 툴이 설치된 서버에 원격 접속합니다.   

Tool Server는 **내부적으로 라이선스를 구동**하거나,   
**다른 License Server**로부터 **TCP 방식**으로 라이선스를 받아와 사용할 수도 있습니다.

본 포스팅에서는 다음의 주제는 다루지 않을 예정입니다:
- Tool Server에 Local PC를 원격 접속하는 방법  
- License Demon 구동 방법  
- Tool 설치 방법  

이유는 간단합니다.  
- **원격 접속 방법**은 구글링을 통해 쉽게 찾을 수 있는 정보입니다.  
- **EDA 관련 라이선스 설정 및 툴 설치**는 IDEC을 통해 쉽게 배울 수 있기 때문입니다.

다만, 이후 다른 포스팅에서는 **IDEC에서 제공하는 툴 설치 방법 외에**  
사용자가 원하는 버전의 Synopsys 툴을 설치하는 방법을 상세히 다룰 예정입니다.

# 🔍 1. Emulation

**Emulation**은 ASIC 설계 과정에서 **매우 중요한 단계**입니다.  
우리가 설계를 진행하면서 여러 **testbench**를 작성하고 **RTL simulation**을 통해 verification을 수행합니다.  
하지만 실제로 칩을 측정할 때는 **예상치 못한 문제가 발생할 가능성**이 있습니다.

이러한 문제를 **사전에 방지하기 위해**,  
제작할 ASIC chip을 **모델링하여 미리 emulation을 수행**하는 것은 반드시 필요한 과정입니다.

<div style="text-align: center;">
  <img src="https://github.com/user-attachments/assets/4dd0f34f-a943-4e8c-968c-08f8f4ca4e74" 
       alt="Demo GIF" width="600">
  <p style="text-align: center;"><em>그림 2. Emulation 예시</em></p>
</div>


**그림 2**는 제가 설계한 Emulation 시스템을 나타낸 것입니다.  
먼저, ASIC으로 구현하고자 하는 작업을 modeling하여 **FPGA에 포팅**합니다.  
그 다음, **IO Hub 역할을 하는 FPGA**를 추가로 포팅합니다.  
두 FPGA의 IO를 연결한 뒤, **IO Hub 역할을 하는 FPGA는 PC와 연결**합니다.  
PC에서는 시스템 검증을 위한 **간단한 프로그램**을 개발하여 사용합니다.

---

### 시스템 검증 프로세스
제 경우, 아래와 같은 일련의 프로세스를 통해 시스템 검증을 수행합니다:

1. **IO Hub와 연결 설정**:  
   프로그램에서 IO Hub와의 연결을 위해 간단한 설정을 수행합니다.

2. **데이터 생성 및 전송**:  
   **그림 2**에 보이는 것처럼, **RUN 버튼**을 클릭하면 PC에서 데이터를 생성하여 IO Hub로 전송합니다.

3. **데이터 처리**:  
   - IO Hub는 데이터를 수신한 뒤, 이를 **ASIC Model**로 전달합니다.  
   - ASIC Model은 데이터를 수신한 뒤 설계된 연산 작업을 수행하고, 결과를 다시 IO Hub로 전달합니다.

4. **결과 반환 및 분석**:  
   - IO Hub는 ASIC Model로부터 받은 결과를 다시 PC로 전송합니다.  
   - PC는 데이터 생성 시 미리 계산해 둔 **정답과 결과를 비교 분석**하여 검증을 수행합니다.

---

이 과정을 통해 Emulation 시스템이 설계한 대로 정확히 동작하는지 확인할 수 있습니다. Tape-out, Packaging, PCB 등의 작업이 끝나면 PCB 보드를 ASIC model 자리에 그대로 옮겨 시스템 검증을 수행합니다.

---

### IO Hub의 필요성에 대하여
왜 **ASIC Model**과 **PC를 바로 연결하지 않고 IO Hub를 두는지**에 대해 의문이 드실 수 있습니다.  
맞습니다. **신뢰할 수 있는 IO Protocol IP**를 가지고 계신다면 PC와 ASIC Model을 바로 연결하는 것도 가능할 것입니다.  
그러나, 현실적으로 연구실 수준에서는 이러한 IP를 설계하는 것이 쉽지 않습니다.  

이러한 이유로, FPGA에 내장된 **다양한 Hard IP (Device)** 나 **검증된 Soft IP (RTL)** 을 이용하여 PC와 연결하게 됩니다.

---

### ASIC Model과 IO Hub의 연결
ASIC Model과 IO Hub를 연결할 때 사용할 수 있는 일반적인 IO Protocol은 다음과 같습니다:
- **SPI (Serial Peripheral Interface)**  
- **UART (Universal Asynchronous Receiver-Transmitter)**  

하지만, 이 과정에서 **ASIC 설계 시 우리가 얼마나 신뢰할 수 있는 Clock을 사용할 수 있는지**를 고려해야 합니다.

---

### Clock 생성에 대한 문제
**Clock을 생성하는 방법**은 여러 가지가 있지만, 각각의 방법에는 다음과 같은 장단점이 있습니다:

#### 1️⃣ **외부에서 Chip으로 Clock을 직접 전송**
- **장점**: 구현이 간단.
- **단점**:  
  - PAD의 **데이터 전송 속도(data rate)** 한계로 인해 구동 가능한 Clock이 매우 느림.  
  - Clock의 Slew로 인해 **Hold Violation** 등의 문제가 발생.

#### 2️⃣ **Crystal Oscillator**
- **구현**:  
  - 벤더가 제공하는 Crystal 전용 PAD와 데이터시트를 참고해 PCB 수준에서 적절한 RLC 회로 설계.  
  - 일반적으로 **수십 MHz** 수준의 안정적인 Clock 생성 가능.  

- **한계점**: 요즘 논문에서는 **수백 MHz ~ GHz 수준의 Clock**에서 프로세서가 정상적으로 동작한다고 주장.  
  이러한 환경에서는 더 빠른 Clock을 생성해야 하므로 Crystal의 사용은 부적절함.

#### 3️⃣ **Ring Oscillator**
- **구현**: Inverter를 사용하여 Ring Oscillator를 설계.
- **문제점**:  
  - Voltage drop, Ringing 등 예기치 못한 문제가 발생할 수 있음.  
  - 이를 완화하기 위해 **Decap 배치** 및 **강력한 Buffer** 설계 필요.

#### 4️⃣ **PLL (Phase-Locked Loop)**
- **이점**: 가장 이상적인 Clock 회로로, 빠른 Clock 생성 가능.  
- **단점**: 설계와 구현 난이도가 매우 높아 **매우 큰 load를 가진 프로세서**를 구동하는  
 수준의 Silicon Proven된 IP를 확보하기 어려움.

---

위와 같은 일련의 이유로 **수백 MHz ~ GHz 수준의 Clock**을 사용한다면 그 Clock의 신뢰도에 대한 고민이 클 수밖에 없습니다.

---

### IO Protocol에서의 Timing Mismatch 문제
ASIC Chip과 IO Hub 사이의 IO Protocol 구현 시,  
**SPI**나 **UART** 같은 방식은 일반적으로 **Counter 기반의 Timing 맞춤**을 사용합니다.  
이로 인해, **Clock이 예상과 다르게 만들어지면** 두 장치 사이에 **Timing Mismatch**가 발생할 가능성이 존재합니다.

---

### AMBA Protocol의 장점
제가 생각하는 가장 현명한 해결책은 **AMBA 계열의 Protocol**을 사용하는 것입니다.  
AMBA Protocol은 **Handshake 방식**을 이용하여 통신을 진행하며,  
전송 측과 수신 측이 모두 준비된 상태에서만 데이터 전송이 이루어집니다.  
따라서, Clock에 의해 야기되는 Timing Mismatch 문제를 효과적으로 방지할 수 있습니다.

저는 AMBA Protocol을 **Custom Protocol**로 수정하여 사용하고 있습니다.  
이 방식은 위와 같은 상황에서 가장 안전하고 신뢰할 수 있는 솔루션이라고 생각합니다.

---

저는 검증 프로그램을 만들 때 과거에는 C기반의 코딩을 했지만 최근에는 Python기반으로 수정하였습니다. 시스템의 검증이 목표이기 때문에 속도는 느리지만 구현이 쉬운 UART를 이용하여 PC와 IO Hub를 연결하고 있습니다.  

언제까지 하겠다고 약속드리기는 어렵지만 FPGA 포팅이나 검증프로그램 개발은 별도의 Tool 라이센스 없이 Windows 환경에서도 진행할 수 있기 때문에 따라할 수 있는 간단한 toy example을 만들어 보도록 하겠습니다.

---

### FPGA 포팅에 대하여

FPGA 포팅 시, 어떤 FPGA를 사용할 것인지 고민하는 것이 중요합니다.  
일반적으로 **Xilinx**와 **Altera**의 보드를 많이 사용하며, 선택한 보드에 따라 사용하는 툴이 달라집니다:
- **Xilinx 보드**: Vivado 및 Vitis를 사용.
- **Altera 보드**: Quartus를 사용.

---

### SoC 타입 FPGA와 Pure FPGA
FPGA는 크게 **SoC 타입 FPGA**와 **Pure FPGA**로 구분할 수 있습니다.

#### 1️⃣ SoC 타입 FPGA
- **구성**: ARM Core와 FPGA Chip이 결합된 보드.
- **Xilinx SoC FPGA**: Zynq 라인업.
- **특징**:  
  - IO Device는 ARM Core 쪽에 연결되어 있어, **C 프로그래밍**을 통해 IO 포팅을 수행.  
  - 이 과정에서 **Vitis**를 사용.  
  - 최근에는 ARM Core를 Python으로 쉽게 제어할 수 있는 **PYNQ**도 등장.

#### SoC 타입 FPGA의 장점
- **Ethernet, PCIe, DRAM (HBM)** 등을 사용하는 경우 SoC 타입 FPGA가 효과적.  
  - **RTL로 구현하기 어려운 IO Device의 컨트롤**을 SW로 처리할 수 있음.  
  - Core Logic 설계 공간 부족 문제를 완화.

---

#### 2️⃣ Pure FPGA
- **구성**: FPGA만 탑재된 보드.
- **특징**:  
  - SoC 타입 FPGA보다 구현과 검증 시간이 짧음.  
  - 모든 제어를 RTL로 수행.

---

### 시스템 및 Application에 대한 고려
#### IO 및 Hub 구현
- **UART**를 IO로 사용하며, Hub는 데이터 전송 외에 다른 프로세싱을 수행하지 않음.  
- 이러한 이유로, 비교적 간단하게 **RTL로 구현 가능한 Pure FPGA**를 사용.

#### 사용 보드
- **Openvino FPGA**:  
  - IO 포트가 많고, 라이센스가 필요 없음.  
  - Hub 및 가벼운 AISC Model 포팅에 적합.  
- **Xilinx Virtex 계열**:  
  - 무거운 ASIC Model 포팅에 적합.
  - 단, 고가의 License가 필요함.
  - 학교의 경우 University Donotation을 통해 라이센스를 얻을 수 있음.

---

### ASIC Model 포팅 시 고려사항
ASIC Model의 목적은 **설계 중인 ASIC 칩의 RTL 코드를 FPGA에 전부 포팅**하는 것입니다.  

- 필요 시 Custom 회로나 일부 IP는 **FPGA의 IP를 활용하거나 RTL로 모델링**할 수 있음.  
- **Altera FPGA**의 고사양 모델은 SoC 타입 FPGA가 대부분이며, 라이센스 비용 문제가 있을 수 있음.  

ASIC Model 포팅의 경우 Pure FPGA인 **Xilinx Virtex 계열**혹은 **Altera의 Openvino FPGA** 가 좋은 선택이라고 생각합니다.  

---
**그림 2**의 경우 **ASIC Model**과 **IO Hub** 모두 **Openvino FPGA**를 이용하여 구현하였습니다.  

# 🔍 2. Library Preparation

`Emulation` 파트는 중요하고 다룰 내용이 많아 다소 길었습니다.  
이제 **RTL 코딩 및 시뮬레이션**으로 넘어가기 전에, **합성(Synthesis)**과 그 후속 과정들을 위해 **다양한 데이터 타입**에 대해 이해하고, 각 단계에서 사용할 **Library를 준비**해야 합니다.

---

### PDK 벤더가 제공하는 주요 데이터
TSMC나 Samsung 같은 PDK 벤더는 다음과 같은 데이터를 제공합니다:

#### 1️⃣ **Analog PDK**
- **Device 관련**:
  - Device Library (NMOS, PMOS)
  - Device Spice Model
  - gds mapfile, device mapfile, calibreview mapfile
  - Virtuoso techfile
- **Rule Deck 관련**:
  - DRC/LVS/PEX rule deck
  - Dummy Insertion rule deck
  - hcell for LVS/PEX

#### 2️⃣ **Digital IP (EDK)**
- **Standard Cell Library**
- **IO PAD Library**
- **Memory Compiler**
- **Tool-specific Files**:
  - icc/icc2 tech file, gds mapfile, antenna rule file, tlup/tlup mapfile
  - innovus lef tech file, gds mapfile

---

### Analog PDK의 중요성
**디지털 설계**만 한다고 해도 Analog PDK에 대한 이해는 필수적입니다.  
이는 **Physical Verification** 단계에서 사용되는 **Sign-Off DRC/LVS** 과정에 Analog PDK가 필요할 수 있기 때문입니다.

#### 주요 이유:
1. **Mixed IC Design**: Clock Generator, Memory 같은 Hard IP 설계.
2. **Custom Standard Cell**: 특별한 기능을 가진 Standard Cell 설계.
3. **Batch 모드 Sign-Off**: Assura, ICV, Calibre 등 LVS시 rule deck 및 spice 사용.

---

### Digital IP가 제공하는 주요 파일
Digital IP (EDK)에서 제공하는 파일은 다음과 같습니다:

1. **gds**:
   - **물리적 정보**: well, via, metal, poly 등.
   - Foundry에 최종적으로 제출하는 파일.
   - Full-Custom 및 PnR 과정에서 필수적.

2. **lef**:
   - gds에서 metal 및 pin 정보를 추출한 파일.
   - Place 및 Routing을 위한 경량화된 정보.

3. **lib**:
   - **논리적 정보**: area, capacitance, transition time, setup/hold, recovery/removal, operation 등.

4. **spice netlist**:
   - Transistor-level netlist 및 RC 정보 포함.
   - **hspice simulation** 및 **LVS** 수행에 사용.
   - Analog PDK의 device를 Virtuoso로 import하여 schematic 변환 가능.

5. **verilog model**:
   - IP의 동작 모델로, **RTL 시뮬레이션**에 사용.

---

### 파일별 주요 사용 툴 요약
<table style="width: 100%; border-collapse: collapse; text-align: left; font-size: 14px;">
  <thead>
    <tr>
      <th style="border: 1px solid #ddd; padding: 8px;">파일</th>
      <th style="border: 1px solid #ddd; padding: 8px;">주요 내용</th>
      <th style="border: 1px solid #ddd; padding: 8px;">사용 툴</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>gds</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">모든 물리적 정보</td>
      <td style="border: 1px solid #ddd; padding: 8px;">Virtuoso</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>lef</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">metal 및 pin 정보 (경량화)</td>
      <td style="border: 1px solid #ddd; padding: 8px;">Cadence Innovus</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>lib</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">논리적 정보 (area, timing 등)</td>
      <td style="border: 1px solid #ddd; padding: 8px;">Cadence Genus</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>spice netlist</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">Transistor-level 구성</td>
      <td style="border: 1px solid #ddd; padding: 8px;">hspice, primelib, calibre</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>verilog model</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">IP 동작 모델</td>
      <td style="border: 1px solid #ddd; padding: 8px;">VCS, Verdi</td>
    </tr>
  </tbody>
</table>

### 추가 고려 사항
1. **OA(Open Access) 변환**:  
   - PDK ventor는 spice, gds를 따로 따로 virtuoso의 schematic, layout으로 변환해야 하는 번거로움을 줄여 주고자 virtuoso용 라이브러리인 CDK를 제공하기도 함.
   - Virtuoso용 CDK 라이브러리를 OA로 변환해야 함.  
   - 변환 방법은 **나인플러스**의 가이드를 참고.

2. **Analog 설계자처럼 GUI 의존 불가**:  
   - Batch 모드에서 Calibre를 사용하기 위해 명령어 및 Rule Deck의 문법 숙지 필요.

---

이상으로 PDK와 Digital IP의 주요 데이터 및 활용에 대해 정리했습니다.  
추후에 각 파일을 실제 설계 과정에서 어떻게 사용하는지 다루도록 하겠습니다.

---
## Synopsys Tool Library

Synopsys는 Cadence와는 다른 형식의 라이브러리를 사용합니다.  
저는 Synopsys 툴을 주로 사용하므로, 이에 대해 더 자세히 다루겠습니다.  
(추후 Innovus로 전환하게 되면 Innovus에 대한 내용도 다룰 예정입니다.)

---

### Logical Library
Synopsys에서는 **Library Compiler**를 이용해 `lib` 파일을 `db` 형식으로 변환하여 사용합니다:
- **lib**: ASCII 형식, 파일 크기가 크고 무거움.
- **db**: 바이너리 압축 포맷, 더 가볍고 효율적.

벤더는 일반적으로 여러 **VT (Voltage, Temperature) 코너**에 대해 lib와 db를 모두 제공합니다.  
Synopsys 툴을 사용하더라도 엔지니어가 Cell 정보를 확인할 때는 **lib (ASCII)** 형식을 주로 봅니다.

---

### Physical Library
Synopsys는 현재 다음의 3세대 PnR 툴을 공식적으로 판매하고 있습니다:
1. **ICC (1세대)**
2. **ICC2 (2세대)**
3. **Fusion Compiler (3세대)**

각 툴은 서로 다른 Physical Library를 사용하며, 이는 툴이 공존하는 주요 이유 중 하나입니다.

---

#### 1️⃣ Milkyway (ICC용)
ICC는 Synopsys의 초기 PnR 툴인 Astro에서 사용하던 **Milkyway Library**를 사용합니다.  
Milkyway Library를 생성하기 위해 Synopsys는 SolvNet에서 **Reference Methodology (RM)**을 제공합니다.

- **Milkyway Library 생성 방법**:  
  Milkyway Library는 **Milkyway 툴**을 사용해 생성합니다. (포맷이랑 툴 이름이 같아요 😅😄)
  - **lef**를 이용하는 방법 (추천): 변환이 더 안정적.  
  - **gds**를 이용하는 방법.

- **필요 파일**:
  - lef, gds, db, tech file, gds mapfile, clf.  
  - **clf**: Cell의 Antenna Property를 기술하는 파일로,  
    IP 벤더가 제공하거나 **lef**에서 추출하여 생성.  
    (**Bash script**를 만들어 변환 작업 수행.)

- **제약 사항**:
  - 과거에는 Herculues 툴로 clf를 추출했으나,  
    현재는 ICV (IC Validator)로 대체되면서 이 기능이 사라짐.  
  - 따라서, Bash script로 직접 변환하지 않으면  
    Milkyway 생성 시 Antenna Property를 추출할 수 없음.

---

#### 2️⃣ NDM (ICC2용)
ICC2는 기존 Milkyway Library와 달리 **NDM (New Data Model)**이라는 독자적인 라이브러리를 사용합니다.  
NDM은 **Logical Library와 Physical Library를 통합한 All-in-One 스타일**입니다.

- **NDM Library 생성 방법**:  
  NDM Library는 **ICC2에 내장된 Library Manager**를 사용해 생성합니다.
  - **lef** 및 **gds**를 이용.  
  - lef이나 gds가 아닌 기존 **Milkyway**를 기반으로 변환 가능.  

- **필요 파일**: lef, gds, db, tech file, gds mapfile, clf, Milkyway (선택).  

- **장점**:
  - ICC2에 내장된 Library Manager로 NDM 생성 가능.  
  - **ICV**를 사용해 clf 없이도 NDM 생성 시 Antenna Property 추출 가능.

---

### 요약
<table style="width: 100%; border-collapse: collapse; text-align: left; font-size: 14px;">
  <thead>
    <tr>
      <th style="border: 1px solid #ddd; padding: 8px;">툴</th>
      <th style="border: 1px solid #ddd; padding: 8px;">라이브러리 형식</th>
      <th style="border: 1px solid #ddd; padding: 8px;">생성 방법</th>
      <th style="border: 1px solid #ddd; padding: 8px;">주요 특징</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>ICC</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">Milkyway</td>
      <td style="border: 1px solid #ddd; padding: 8px;">Milkyway 툴 사용</td>
      <td style="border: 1px solid #ddd; padding: 8px;">안정적이지만 clf 생성 필요.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>ICC2</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">NDM</td>
      <td style="border: 1px solid #ddd; padding: 8px;">ICC2 Library Manager 사용</td>
      <td style="border: 1px solid #ddd; padding: 8px;">Logical/Physical 통합, clf 없이 가능.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>Fusion Compiler</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">Fusion Lib</td>
      <td style="border: 1px solid #ddd; padding: 8px;">-</td>
      <td style="border: 1px solid #ddd; padding: 8px;">-</td>
    </tr>
  </tbody>
</table>
</br>

Synopsys 툴은 각 세대별로 독자적인 라이브러리를 사용하며,  
Milkyway와 NDM은 각각의 툴에 맞게 생성됩니다.  
- **Milkyway Library**: Milkyway 툴을 사용.  
- **NDM Library**: ICC2의 Library Manager를 사용.  

언젠가 기회가 되면 GPDK를 이용해 Milkyway, NDM을 만드는 방법을 다루도록 하겠습니다.  
물론, GPDK가 충분히 필요한 데이터를 모두 제공한다면...

---

## Tool Selection에 관하여

현재 **Fusion Compiler**에서 사용하는 **Fusion Lib**을 만들기 위한 **Reference Methodology (RM)**은  
아직 공식적으로 배포되지 않았으며, Fusion Compiler 자체도 안정화되지 않은 상태로 보입니다.  
따라서, **현 시점에서는 ICC/ICC2를 선택하는 것이 적합**합니다.

---

### ICC vs ICC2: 어떤 툴을 선택할 것인가?
IP Vendor에서 제공하는 PDK는 일반적으로 **High-Tech 공정**부터 NDM을 공식적으로 지원합니다.  
제가 아는 바에 따르면 한 회사는 **7nm부터 공식적으로 NDM을 지원하고 있습니다**.  
또한, 해외 IP 개발 회사들의 정보를 종합하면 **40nm까지는 ICC로** 구현한 사례들이 있으며,  
**14/16nm 수준부터는 ICC2를 이용한 개발 사례가 많습니다**.

### ✅ ICC vs ICC2 공정별 적합성
<table style="width: 100%; border-collapse: collapse; text-align: left; font-size: 14px;">
  <thead>
    <tr>
      <th style="border: 1px solid #ddd; padding: 8px;">공정 노드</th>
      <th style="border: 1px solid #ddd; padding: 8px;">권장 툴</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>40nm 이상 (≥40nm)</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">ICC 사용 가능</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><b>22nm 이하 (≤22nm)</b></td>
      <td style="border: 1px solid #ddd; padding: 8px;">ICC2 권장</td>
    </tr>
  </tbody>
</table>  

특히, **FinFET이 도입된 공정부터는 ICC2를 사용하는 것이 적합**하다고 생각됩니다.  
예를 들어, **TSMC 22nm**부터는 오픈된 문서에서도 기존 28nm ~ 65nm 세대와 구분을 하고 있는 것으로 확인됩니다.

---

### 저의 PDK 및 Tool Flow 선택
저는 **28nm, 65nm 공정**을 사용하고 있으며,  
현재 설계 플로우는 **ICC를 기준으로 구축**하였습니다.

- **28nm의 경우**, 현재 **Front-End까지만 진행 중**이므로  
  ICC로 28nm까지 완벽히 커버할 수 있을지는 아직 검증되지 않았습니다.
- 따라서, 본 포스팅에서 소개하는 플로우는 **40nm 이상의 공정에서 적합**함을 미리 알려드립니다.

---

## My Selected Tool List
다음은 제가 ASIC 설계를 위해 사용하고 있는 툴들을 정리한 목록입니다.  
**(Emulation 및 PCB 설계 관련 툴은 제외)**

### ✅ **Cadence**
- **Virtuoso** (Full-Custom Layout)
- **Abstract Generator** (LEF 생성)

### ✅ **Siemens**
- **Calibre** (Physical Verification: DRC/LVS/PEX)

### ✅ **Synopsys**
- **VCS** (RTL Simulation)
- **Verdi** (Debug & Waveform Analysis)
- **Design Compiler** (Synthesis)
- **IC Compiler** (PnR for ICC)
- **Library Compiler** (lib → db 변환)
- **Milkyway** (Physical Library Prepare for ICC)
- **PrimeLib** (Cell Library Characterization)
- **PrimeTime** (Static Timing Analysis)
- **StarRC** (Parasitic Extraction)
- **Hspice** (Circuit Simulation)

---

### 추가 고려 사항
- **Custom Standard Cell IP를 사용하지 않는 경우**:
  - **Abstract Generator, PrimeLib, Hspice**는 필수적이지 않음.
- **ICC2 기반으로 전환할 경우**:
  - Milkyway 대신 **ICC2의 Library Manager**를 사용해야 함.
  
---
### Custom Standard Cell Library

저는 연구 목적으로 **특별한 Flip-flop(FF)** 을 설계하여 사용했습니다.  
이를 **Synthesis 및 PnR**에서 사용하려면 **DC 및 ICC**에서 읽을 수 있는 **Custom Library**를 제작해야 합니다.

---

### Custom Library 제작 과정
다음은 Custom Library 제작 과정을 요약한 내용입니다:

1. **Schematic 및 Layout 제작**
   - Virtuoso에서 **Schematic** 및 **Layout**을 완성.
   - **DRC/LVS/PEX**를 수행하며, PEX 출력 형식을 **dspf** 또는 **spef**로 선택.

2. **lib 파일 생성**
   - **PrimeLib**을 이용하여 lib 파일을 생성.  
   - **입력 파일**: PEX에서 출력한 dspf, Analog PDK의 hspice model.
   - **설정 기준**: VT 코너 및 Characterization 기준 정의 (IP 벤더 제공 데이터시트 및 lib 파일 참고).

3. **Characterization**
   - PrimeLib은 **transition time, setup/hold** 등을 Characterization.  
   - **Spice Simulator 선택**:  
     - hspice, primesim, finesim 중 선택 가능.  
     - 제 경우 **PrimeLib 내장 primesim** 또는 hspice 사용.

4. **lib 및 Verilog 모델 추출**
   - PrimeLib에서 lib 파일과 Verilog 모델 추출.
   - 추출된 lib 파일의 타이밍 정보가 schematic simulation 또는 spice simulation 결과와 유사한지 확인.
   - 추출된 Verilog 모델을 **VCS, Verdi**를 이용하여 RTL simulation 수행.

5. **LEF 파일 생성**
   - **Abstract Generator**를 사용하여 Virtuoso의 layout과 lib 파일을 기반으로 **LEF 파일 생성**.

6. **Antenna Property 변환**
   - **LEF 파일**에서 antenna property 정보를 추출해 **CLF 형식으로 변환**.
   - 이를 위해 Batch Script를 만들어야 함.

7. **GDS Streamout**
   - Virtuoso에서 layout을 GDS 형식으로 **streamout**.

8. **Library Compiler 변환**
   - lib 파일을 Library Compiler로 **db 형식으로 변환**.

9. **Milkyway 라이브러리 생성**
   - 생성된 lef, gds, db를 기반으로 Milkyway 라이브러리 생성.
   - ICC에서 Milkyway를 열어 pin, metal 위치가 정확히 추출되었는지 확인.

---

### 제작 시 고려사항
1. **PG Pin의 Pitch**
   - VDD 및 VSS **PG Pin**은 일정한 pitch로 배치되어야 합니다.
   - 보통 **PG 메탈**은 수평으로 직선 배치.

2. **PnR 툴의 배치 방식**
   - PnR 툴은 일정 간격으로 **power rail**을 배치하고, 그 위에 Standard Cell을 배치.
   - VDD, VSS 라인과 핀의 위치는 **power rail 간격에 정확히 맞춰야 함**.
   - 이때 pitch에 대한 정보는 보통 ICC/ICC2 techfile (.tf)의 unit 혹은 tile 등의 이름을 가진 layer에 기술되어 있음.
   - 또는 Standard Cell을 Virtuoso에서 열어 직접 간격을 찾는 것도 가능.

---

위 과정은 요약된 내용으로, 실제로는 매우 복잡하며 다양한 요소를 고려해야 합니다.  
만약 **GPDK**가 앞선 플로우를 재현할 수 있을 만큼 충분한 데이터를 담고 있는 것이 있다면,  
**Custom Cell Library 제작**에 대한 자세한 내용을 다룰 기회를 가지도록 하겠습니다.

---

꽤 길었습니다. 정리하면, Library Preparation 단계를 거쳐 다음과 같은 포맷의 파일들을 가지고 있어야 합니다.  
- lib/lef/gds (Global data format)
- db/Milkyway (Used in Synopsys tool)
- spice; spi, dspf, spef 등 (LVS or Characterization)
- verilog (RTL Simulation)

보통 PDK 벤더는 Std. Cell, IO PAD 등에 대하여 위의 모든 파일을 제공합니다. 기술 유출 방지를 위해 gds를 제공하지 않을 수도 있습니다. Memory Compiler의 경우 Milkyway를 제외한 모든 파일이 기본적으로 생성됩니다. 따라서, Memory Compiler에서 생성된 파일들을 이용하여 Milkyway를 생성해야 합니다. Custom library를 제작하는 경우 위의 언급한 모든 파일을 만들어야 합니다.  

# 🔍 3. RTL Simulation
본격적으로 들어가기 전에 알아야 할 것들이 참 많았죠? 최대한 간략하게 적어 보려고 했는데 워낙 방대한 내용을 다뤄야 하다보니 글이 계속 길어지네요.  

이제 그나마 ASIC 설계 flow에서 가장 고상한(?) RTL에 대하여 이야기 해보도록 하겠습니다.  

(To be continue...)