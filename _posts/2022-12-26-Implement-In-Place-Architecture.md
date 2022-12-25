---
layout: post
title: "Implement In-Place FFT Architecture"
tags: FFT
---

이번 프로젝트에서는 `In-Place FFT Architecture`를 `Verilog HDL`을 이용해 `FPGA`에 포팅하고, `GUI`프로그램을 만들어 FPGA와 PC간 `UART 통신`을 이용해 FPGA에서 FFT를 수행하고, PC는 FPGA에서 데이터를 받아 `Spectrogram`으로 출력하였다. 이 포스팅은 모든 코드를 설명하지 않는다. 그러나, [링크](https://github.com/yhkwon6658/Inplace-FFT)를 통해 공개한 GUI프로그램의 사용법과 `Block Diagram` 수준에서 각각의 모듈을 소개한다. 전반적으로 다음 논문 [L. G. Johnson's Conflict Free Memory Addressing for Dedicated FFT Hardware](https://ieeexplore.ieee.org/document/142032) 에 대한 해석과 프로젝트 과정에서 진행한 재해석 및 Spectrogram 에 대한 설명으로 구성된다.

# Link
{% highlight markdown %}
[GitHub](https://github.com/yhkwon6658/Inplace-FFT)
{% endhighlight %}

# FPGA
{% highlight markdown %}
Terasic의 `DE2`를 사용하였다. 기본 연산기로 사용되는 FPU의 경우 Quartus II의 Standard IP를 사용하였다. Xilinx 계열의 보드를 사용하는 경우 Vivado 에서 제공하는 Standard IP를 사용하거나 Open library로 제공되는 FPU 를 사용하는 것을 추천한다.  

# Spectrogram

# Paper Analysis

# Block Diagram

# Flow

# How to run
## 1. Data Processing

## 2. Synthesis and Porting

## 3. Run Spectro - Version Hardware