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
> 실제 **Industry 수준**에는 미치지 못할 수도 있습니다.  

그럼에도 불구하고, 학교에서는 **ASIC 설계/검증/측정 등 전 과정을 제대로 가르치기 어려운 현실**이 있습니다. 때문에 학부생이나 석사 학생들은 **ASIC 설계를 해보고 싶어도**, 채용 공고에서 이야기하는 **RTL / PI / PD / DV 등의 직무에서 무슨 일을 하는지 정확히 알기 어려울 것**입니다.  

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
  <p><em>그림 1. 원격 시스템 구조도</em></p>
</div>

**그림 1**은 일반적으로 ASIC 관련 툴을 사용하는 시스템 구성을 나타냅니다.  
Local의 사용자는 **Mobaxterm**이나 **Putty** 같은 툴을 이용해 **SSH 방식**으로 툴이 설치된 서버에 원격 접속합니다.  

Tool Server는 **내부적으로 라이선스를 구동**하거나, **다른 License Server**로부터 **TCP 방식**으로 라이선스를 받아와 사용할 수도 있습니다.

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
To be continue...