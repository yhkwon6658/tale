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
  <img src="https://github.com/user-attachments/assets/31a6caf3-5123-4b4c-a6ab-5a480e9e20d6" 
       alt="System Diagram" height="300">
  <p style="text-align: center;"><em>그림 1. 원격 시스템 구성도</em></p>
</div>

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

# 🔍 2. Library Preparation
