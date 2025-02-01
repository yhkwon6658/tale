---
layout: post
title: "ASIC 설계 레시피"
author: "Yonghwan Kwon"
tags: "tools"
comments: true
excerpt_separator: <!--more-->
---

## 📝 블로그 정리 & 업데이트
블로그 정리를 안 한지 거의 **2~3년**이 다 되었네요. 정말 오랜만에 기존 포스트들을 정리하고, **프로필도 업데이트**하였습니다.  
그동안 **과제나 개인 연구**로 참 바쁘게 살아왔습니다. 작년 말부터 **논문 서브미션**을 시작했고, 올해는 **여러 편의 논문을 제출**할 예정입니다.  
🙏 부디 좋은 결과가 있기를 바랍니다.  

작년에는 **Tape-out**을 완료했고, 올해는 **두 번째 칩을 제작**하게 되었습니다.  
이 포스트에서는 **ASIC 설계 전반**에 대한 이야기를 담으려고 합니다.  
다만, **NDA 문제**로 인해 모든 스크립트를 공개하기는 어렵습니다.  

> 저는 **65nm, 28nm 수준의 PDK**를 사용하고 있기에,  
> 실제 **Industry 수준**에는 미치지 못할 수도 있습니다.  

그럼에도 불구하고,  
학교에서는 **ASIC 설계/검증/측정 등 전 과정을 제대로 가르치기 어려운 현실**이 있습니다.  
때문에 학부생이나 석사 학생들은 **ASIC 설계를 해보고 싶어도**,  
채용 공고에서 이야기하는 **RTL / PI / PD / DV 등의 직무에서 무슨 일을 하는지 정확히 알기 어려울 것**입니다.  

이제 막 시작하는 **초심자들을 위한 글**을 쓴다는 생각으로,  
시간이 날 때마다 틈틈이 이 포스팅을 마무리 지을 계획입니다.  
<!--more-->

---

## 🔎 ASIC 설계의 시작
ASIC이라고 하면 많은 분들이 **합성(Synthesis)**이나 **PnR(Place & Route)** 등을 떠올리실 겁니다.  
하지만 **설계에서 가장 중요한 것은 "동작하는 시스템을 만드는 것"**입니다.  

### 🎯 ASIC 설계의 첫 단계
설계의 첫 단계는 **측정 시스템을 구축하고, Emulation을 수행하는 것**입니다.  
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

---

# 🔍 1. Emulation
