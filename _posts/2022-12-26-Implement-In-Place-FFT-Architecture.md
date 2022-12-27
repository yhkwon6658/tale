---
layout: post
title: "Implement In-Place FFT Architecture"
author: "Yonghwan Kwon"
tags: "Project"
comments: true
excerpt_separator: <!--more-->
mathjax: true
---

이번 프로젝트에서는 `In-Place FFT Architecture`를 `Verilog HDL`을 이용해 `FPGA`에 포팅하고, `GUI`프로그램을 만들어 FPGA와 PC간 `UART 통신`을 이용해 FPGA에서 FFT를 수행하고, PC는 FPGA에서 데이터를 받아 `Spectrogram`으로 출력하였다. <!--more--> 
이 포스팅은 모든 코드를 설명하지 않는다. 그러나, [링크](https://github.com/yhkwon6658/Inplace-FFT)를 통해 공개한 GUI 프로그램의 사용법을 제시하고, `Block Diagram` 수준에서 각각의 모듈을 소개한다. 다음 논문 [L. G. Johnson's Conflict Free Memory Addressing for Dedicated FFT Hardware](https://ieeexplore.ieee.org/document/142032) 에 대한 해석과 위 논문이 가지고 있는 부족한 점을 보완, Spectrogram 에 대한 간단한 설명으로 구성된다.

# 주의
이 게시물은 친절하지 않습니다. DIT, DIF, Radix, Twiddle factor 등 FFT에 사용되는 기본적인 용어들과 기초적인 수식은 숙지하고 있는 것을 전제로 작성하였습니다.

# FPGA
Terasic의 `DE2`를 사용하였다.
기본 연산기로 사용되는 FPU의 경우 Quartus II의 Standard IP를 사용하였다.
Xilinx 계열의 보드를 사용하는 경우 Vivado 에서 제공하는 Standard IP를 사용하거나 Open library로 제공되는 FPU 를 사용해야 할 것이다. `(추신: DE2는 Quartus II를 사용합니다. 따라서 Quartus Prime에서 지원되는 FPGA 칩의 경우 Quartus II의 Megawizard로 얻은 FPU가 제대로 동작하지 않을 수 있습니다. 만약, 그럴 경우 Butterfly 내부의 FPU를 교체해야 합니다. FPU는 일반적으로 n clock이 요구되고, 저희가 제작한 모듈의 경우 FFT CONTROLLER를 FSM을 통해 컨트롤하고, Butterfly에서 연산되는 시간을 카운터로 고려하여 다음 stage로 넘어가는 방식을 취하고 있습니다. 만약, FPU를 교체할 경우 이 부분을 고려하여 FSM을 수정해야 합니다.)`

### FPU 교체시 FFT CONTROLLER 에서 수정해야 하는 부분
```verilog
module()

always @() beign
코드
end

endmodule
```

# Spectrogram
일반적으로 FFT는 data의 처리를 보다 용이하게 할 수 있도록 사용된다. Time domain의 data를 Frequency domain으로 전환함으로써 모든 data는 `Nyquist frequency`이하로 표현되며 신호처리나 DSP수업을 듣다보면 다음 그림과 같은 그래프를 자주 보게된다.

![FFT](https://www.mwmresearchgroup.org/uploads/3/0/8/6/30861243/editor/image-3-ft.png?1597158887)  
위 이미지는 Fourier Transform의 성격을 잘 보여준다. Time domain에서는 복잡한 곡선의 형태로 보이는 그래프가 Frequency domain에서는 이 data를 구성하고 있는 몇개의 주파수 성분으로 보인다. 일반적으로 위와 같이 변환함으로써 Noise filtering, Quantization등에 쉽게 이용될 수 있는데, FFT를 이용한 data분석에서는 `Spectrogram`이라는 것이 종종 이용된다.  
  
[링크](https://m.blog.naver.com/vmv-tech/220936084562)에서 다음 2개의 그림을 참고하였다.
![그림1](https://mblogthumb-phinf.pstatic.net/MjAxNzAyMTRfMjM1/MDAxNDg3MDQwNzc4NDk4.FLUGC0mJkpN8qLBi6gpjEatqjvsZc163zROfFLw7lr4g.AMzGX7q-mGBJLwYTO2nOd4MqUagOEuCvn7bUd1cuuWwg.JPEG.vmv-tech/f2.jpg?type=w2)  
본 게시글의 저자는 시간에 따라서 10hz, 20hz, 30hz로 변하는 data를 만들었다. 이를 FFT한 결과는 다음과 같다.
![그림2](https://mblogthumb-phinf.pstatic.net/MjAxNzAyMTRfMjEg/MDAxNDg3MDQwOTE0Nzk1.QOSAnTmtUbhvJvkbgK1HSdIXwqXVqMZspNgzo6BRz5cg.UoFkmxQPuFPbARubGGarerh3bId4gIdIuT03pFu23Qgg.JPEG.vmv-tech/f3.jpg?type=w2)  
앞서 이해한 것과 마찬가지로 3개의 중심 주파수를 같는 그래프를 볼 수 있다. 그러나, FFT는 frequency domain에서 주파수의 distribution만을 제공할 뿐 언제 주파수의 분포가 바뀌는지를 제공하지는 않는다. 이 문제를 해결하기 위한 방법으로 `STFT(Shot-Time-Fourier-Transform)`이 있다. 본 게시글에서는 STFT를 자세하게 설명하지는 않는다. STFT에 대한 설명은 다음 [링크](https://www.audiolabs-erlangen.de/resources/MIR/FMP/C2/C2_STFT-Basic.html)를 참고하기 바란다. 간단한 예시를 들면 8192개의 data에 대하여 8192-point FFT를 진행하는 대신 8번을 나누어 1024-point FFT를 수행하는 것이다. 일반적으로, FFT의 결과는 다음과 같은 형태로 출력한다.  

![그림3](https://miro.medium.com/max/720/1*V2mgZ7y0ngd3q4DZ01xkEQ.webp)  
그림을 보면 x축은 시간, y축은 주파수 성분이다. 시간 t1일 때, FFT를 통해 얻어진 coefficients는 heat map 형식으로 나타난다. 위의 그림을 보면 전반적으로 Low frequency성분이 강한데, 200초 부근을 보면 high frequency성분이 드문 드문 강하게 튀는 것을 볼 수 있다. 이번 프로젝트는 FFT Architecture를 설계하는 것이지만 궁극적으로 FFT의 결과는 Spectrogram형태로 우주 공간에서 과학적 분석을 위해 사용된다고 한다. 이 때문에 이번 프로젝트에서는 이후의 프로젝트에서도 사용할 검증환경을 GUI로 만들었고, 해당 검증환경에서는 FPGA로부터 받을 원본 데이터의 그래프, PC를 통해 FFT를 수행 후 Spectrogram형태로 출력한 그래프, FPGA가 FFT를 수행한 후 PC로 전송한 데이터를 Spectrogram으로 출력한 그래프를 플롯하고, PC에서 계산한 FFT결과와 FPGA에서 전송받은 FFT결과를 비교하여 Log창에 출력한다. 다음 그림은 우리가 만든 GUI의 Sample이미지다.  
  
![그림4](/assets/img/Sample.png)  
본 프로젝트에서 구현한 GUI인 `SPECTRO`는 [링크](https://github.com/yhkwon6658/Inplace-FFT/tree/main/SPECTRO%20-%20Version%20Hardware)에서 다운 받을 수 있다. 해당 프로그램은 보완이 필요하다고 판단하여 현재는 소스 코드를 공개하지 않고 프로그램만 배포한 상태이다. 해당 링크의 파일을 모두 다운 받운 후 압축을 풀어 `Inplace-FFT/SPECTRO - Version Hardware/SPECTRO - Version Hardware.exe`를 실행하여 프로그램을 동작시킬 수 있다. 자세한 사용법은 `How to run, 3. Run SPECTRO - Version Hardware`에서 기술하도록 한다.

# Paper Analysis
하드웨어에서 FFT를 구현하는 방식은 크게 2가지로 구분된다. 첫번째는 `In-Place FFT`로 `Memory-base FFT`라고도 종종 불린다. 하나의 Memory와 PE(Processing-Element)를 기본 구조로 하며, FFT의 redundancy를 활용하여 Area를 극단적으로 줄이는 방식이다. 두번째 방식은 `Pipelined-FFT`라고 한다. In-Place FFT의 경우 Area와 Power를 크게 줄이는 대신 Latency가 길어지면서 `real time`에서는 사용이 어려워 진다. 이를 극복하기 위해 제안된 방식이며, 이후 논문 리뷰를 통해 다루도록 한다. 본 프로젝트의 배경이 된 [L. G. Johnson's Conflict Free Memory Addressing for Dedicated FFT Hardware](https://ieeexplore.ieee.org/document/142032)는 In-Place FFT를 하드웨어로 구현하기 위해 `CFA(Conflict Free Addressing)`방법을 제안하였으며, Twiddle factor의 exponent를 선택하기 위한 방법도 제안한다. radix에 관계없이 일반화하는데 성공하였으며, 현재까지 제안되고 있는 모든 In-Place FFT방식은 본 논문을 기초로 하고 있다. 그러나, Johnson의 초기 논문은 Notation에 대한 설명이 되어 있지 않고, 중간 중간 typo등이 존재한다. 또한, 모든 Algorithm과 Architecture가 `DIT(Decimation-In-Time)`을 기준으로 제시 되어 있고, Twiddle factor의 exponent generation의 경우 그 설명이 빈약하여 하드웨어로 Implementation하기 위한 DIF의 경우 그 결과가 아예 제시되어 있지 않은 것을 알 수 있다. 이번 분석에서는 Johnson의 Notation을 설명하고, `Memory bank Partition`, `CFA`, `Exponent generation`을 DIT관점에서 설명한 후 DIF관점에서도 설명하도록 할 것이다.  
  
## I. Notation
DFT equation 

![image](https://user-images.githubusercontent.com/120978778/209723250-34dbfd7a-f405-46ce-a1dd-59f774ceb169.png)

Twiddle factor  
![image](https://user-images.githubusercontent.com/120978778/209723374-956e006e-c757-42be-9880-7bac59c40011.png)  

DFT 수식과 Twiddle factor에 대한 설명은 생략한다.  
Decomposition  
![image](https://user-images.githubusercontent.com/120978778/209724382-83c7dd29-e51e-413c-a282-9791878a37eb.png)  
LHS의 
$$
n_{R-2-c}k_{c}
$$ 
에서 중간에 `,`가 누락되었다.  
RHS의 F와 LHS의 F를 보면 DFT의 결과로 n이 하나 줄고, k가 하나 늘어난 것을 알 수 있다. Johnson은 flow chart를 기준으로 stage마다 n과 k로 indexing을 달리 하는데, Radix 4, 64-point FFT를 한다고 가정하면, 최초의 stage 1은 {n2,n1,n0}의 modulo 4로 0부터 63까지의 모든 숫자를 표현할 수 있다. stage 2에서는 {k0,n1,n0}로 표기한다. 이때, 각 자리에 대한 index의 종류가 n에서 k로 변하는 것일 뿐 예를 들어 63은 stage에 관계없이 {3,3,3}을 의미한다. 이때, n0와 n1이 0이라고 가정하면 summation은 F(0,0,0), F(0,0,1), F(0,0,2), F(0,0,3)에 대하여 진행된다는 것을 알 수 있다. 이는 우리가 흔히 알고있는 DIT flow와 같은 것을 알 수 있다. 해당 notation만 이해하면 twiddle factor를 이해하는 것은 큰 문제가 되지 않을 것이라 생각한다.  
$$
k_{c}
$$  
의 자리에 0, 1, 2, 3을 한번씩 넣어보면 올바른 결과를 얻을 수 있을 것이다. 이 논문은 Typo와 Notation의 의미만 이해하면 그 이후 과정은 매우 쉽게 이해할 수 있다.

## II. Memory Bank Partition
Johnson은 다음 수식을 통해 `"flow chart상에서 stage에 관계없이 같은 line에 있는 data는 모두 같은 memory bank에 위치하며 같은 address를 갖는다."`를 설명한다.  

![image](https://user-images.githubusercontent.com/120978778/209725419-4063f5be-8a58-4616-84c6-2751ac05933d.png)  
이해를 돕기 위한 예시를 들면, flow chart를 보면 Stage,그리고 Stage에서 몇번째 Butterfly인지 관계없이 모든 연산은 하나의 Butterfly에 r개의 data가 들어가고 r개의 data가 나가는 형태이다. In-Place는 이 점에 주목하여 r개의 입력과 r개의 출력을 갖는 Butterfly unit을 하나 만들고, Memory를 r개로 banking하여 1회의 연산에 r개의 memory bank에서 각각 1개의 data를 read하여 butterfly에 입력으로 넣고, 그 출력을 다시 r개의 memory bank에 1개씩 write한다. 쉽게 말해 `데이터를 꺼냈던 곳에 다시 넣는 것`이다. 그렇기 때문에 N개의 data는 최초에 물리적으로 r개의 bank에 각각 N/r개씩 저장되어 항상 같은 memory bank의 같은 address상에 위치하게 되고, 그 이후 항상 같은 자리를 유지하게 된다. 이제 어떻게 데이터를 partition하고, addressing하는지 하나씩 이해해 보도록 하자.  

`예시`를 통해 이해하는 것이 편한데, 앞으로의 설명은 Radix 4, N = 64, DIT 를 가정으로 한다. `(추신: Johnson의 논문이 DIT를 기준으로 작성되어 DIT로 먼저 설명하려 하는데 구글링을 해도 DIT의 flow chart는 나오지가 않습니다. 모든 flow chart가 DIF로 나오는데 이를 좌우반전 시켰다고 상상하면서 이해하시면 될 것 같습니다. 죄송합니다 ㅠㅠ.)`

![image](https://user-images.githubusercontent.com/120978778/209726246-438a281c-589a-48cf-96b3-b3b867577d28.png)  
최초의 논문리뷰 레포트에서 발췌한 것인데, 위의 Equation 6을 이용하여 partition을 하게 된다. Johnson의 수식보다 훨씬 간단한 것을 알 수 있는데, 다음의 설명을 이해하도록 하자.  

F(x2,x1,x0) 를 DFT할 때, Stage에 관계없이 하나의 Butterfly에 들어가야 하는 입력은 x2, x1, x0 중 하나의 digit만 다르며, 나머지는 모두 동일하다. 이때, 편의를 위해 x0를 LHS라 하면, `예시`의 가정에 따라 stage 1일 때, 첫번째 butterfly는 F(0,0,0), F(0,0,1), F(0,0,2), F(0,0,3) 이 하나의 butterfly에 입력으로 들어가게 되며, 이제 4개의 memory bank에 partition을 하려고 한다면 각각 0번, 1번, 2번, 3번 memory bank에 들어가면 될 것이다. 그 다음 butterfly는 F(0,1,0), F(0,1,1), F(0,1,2), F(0,1,3) 일 것이고, 순서대로 1번, 2번, 3번, 4번에 들어가는 것이 너무도 자명해 보인다. 굳이 4번째 data를 0번이 아닌 1번 bank에 넣는 것이 의아할 수도 있는데, stage 2에서는 butterfly의 간격이 4가 되기 때문에 0, 4, 8, 12번째 data가 하나의 butterfly에 입력으로 들어가게 되며, 결과적으로 간격이 1일 때, 간격이 4일 때, 간격이 16일 때는 서로 다른 bank에 위치해야 한다. 이제 설명이 되었을 것이라 생각한다. 논문에서는 보다 복잡해 보이는 notation을 사용하고 있는데, 동일한 뜻으로 이해하면 된다. 이후의 논문들에서는 이를 `modulo-addition` 혹은 `XOR base banking` 등으로 지칭한다.

## III. CFA(Conflict-Free-Addressing)

## IV. Exponent generation

## V. For Hardware Implementation

## VI. Case of DIF


# Block Diagram

# Experiment Flow

# How to run
## 1. Data Processing

## 2. Synthesis and Porting

## 3. Run SPECTRO - Version Hardware

## 마무리
조만간 완성될 게시물입니다.  
수정일: 2022-12-28 AM 07:35