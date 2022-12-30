---
layout: post
title: "Implemention In-Place FFT Architecture"
author: "Yonghwan Kwon"
tags: "Project"
comments: true
excerpt_separator: <!--more-->
---

이번 프로젝트에서는 `In-Place FFT Architecture`를 `Verilog HDL`을 이용해 `FPGA`에 포팅하고, `GUI`프로그램을 만들어 FPGA와 PC간 `UART 통신`을 이용해 FPGA에서 FFT를 수행하고, PC는 FPGA에서 데이터를 받아 `Spectrogram`으로 출력하였다. <!--more--> 

# 주의
이 포스팅은 코드를 설명하기 위한 용도가 아닙니다. 그러나, [링크](https://github.com/yhkwon6658/Inplace-FFT)를 통해 공개한 GUI 프로그램의 사용법을 제시하고, `Block Diagram` 수준에서 각각의 모듈을 소개합니다. 다음 논문 [L. G. Johnson's Conflict Free Memory Addressing for Dedicated FFT Hardware](https://ieeexplore.ieee.org/document/142032) 에 대한 해석과 위 논문이 가지고 있는 부족한 점을 보완, Spectrogram 에 대한 간단한 설명으로 구성되었습니다. 이 포스팅은 친절하지 않습니다. DIT, DIF, Radix, Twiddle factor 등 FFT에 사용되는 기본적인 용어들과 기초적인 수식은 숙지하고 있는 것을 전제로 작성하였습니다.

# FPGA 및 Reference code
Terasic의 `DE2`를 사용한다. DE2의 경우 SDK로 Intel의 Quartus II가 이용된다. FPGA와 GUI프로그램 모두 [IEEE 754 Single precision(32-bit)](https://ko.wikipedia.org/wiki/IEEE_754) 을 기준으로 한다. `Butterfly`에 사용된 `FPU`의 경우 [링크](https://opencores.org/projects/fpu)의 Usselmannm, Rudolf님의 모듈을 사용하였다. `UART CONTROLLER`에 사용된 `UART RX 및 TX 모듈`의 경우 [링크](https://nandland.com/uart-serial-port-module/)의 Nandland에서 제작한 모듈이다. ALTERA(Xilinx)계열의 보드를 사용하는 경우 Quartus(Vivado)에서 제공하는 Standard IP로 FPU를 교체하면 Syntehsis과정이 더 빨라진다.

# Spectrogram
일반적으로 FFT는 data의 처리를 보다 용이하게 할 수 있도록 사용된다. Time domain의 data를 Frequency domain으로 전환함으로써 모든 data는 `Nyquist frequency`이하로 표현되며 신호처리나 DSP수업을 듣다보면 다음 그림과 같은 그래프를 자주 보게된다.

![FFT](https://www.mwmresearchgroup.org/uploads/3/0/8/6/30861243/editor/image-3-ft.png?1597158887)  
위 이미지는 Fourier Transform의 성격을 잘 보여준다. Time domain에서는 복잡한 곡선의 형태로 보이는 그래프가 Frequency domain에서는 이 data를 구성하고 있는 몇개의 주파수 성분으로 보인다. 일반적으로 위와 같이 변환함으로써 Noise filtering, Quantization등에 쉽게 이용될 수 있는데, FFT를 이용한 data분석에서는 `Spectrogram`이라는 것이 종종 이용된다.  
  
[링크](https://m.blog.naver.com/vmv-tech/220936084562)에서 다음 2개의 그림을 참고하였다.
![그림1](https://mblogthumb-phinf.pstatic.net/MjAxNzAyMTRfMjM1/MDAxNDg3MDQwNzc4NDk4.FLUGC0mJkpN8qLBi6gpjEatqjvsZc163zROfFLw7lr4g.AMzGX7q-mGBJLwYTO2nOd4MqUagOEuCvn7bUd1cuuWwg.JPEG.vmv-tech/f2.jpg?type=w2)  
본 게시글의 저자는 시간에 따라서 10hz, 20hz, 30hz로 변하는 data를 만들었다. 이를 FFT한 결과는 다음과 같다.  
![그림2](https://mblogthumb-phinf.pstatic.net/MjAxNzAyMTRfMjEg/MDAxNDg3MDQwOTE0Nzk1.QOSAnTmtUbhvJvkbgK1HSdIXwqXVqMZspNgzo6BRz5cg.UoFkmxQPuFPbARubGGarerh3bId4gIdIuT03pFu23Qgg.JPEG.vmv-tech/f3.jpg?type=w2)  
앞서 이해한 것과 마찬가지로 3개의 중심 주파수를 같는 그래프를 볼 수 있다. 그러나, FFT는 frequency domain에서 주파수의 distribution만을 제공할 뿐 언제 주파수의 분포가 바뀌는지를 제공하지는 않는다. 이 문제를 해결하기 위한 방법으로 `STFT(Shot-Time-Fourier-Transform)`가 있다. STFT에 대한 설명은 다음 [링크](https://www.audiolabs-erlangen.de/resources/MIR/FMP/C2/C2_STFT-Basic.html)를 참고하기 바란다. 간단한 예시를 들면 8192개의 data에 대하여 8192-point FFT를 진행하는 대신 8번을 나누어 1024-point FFT를 수행하는 것이다. 일반적으로, FFT의 결과는 다음과 같은 형태로 출력한다.  

![그림3](https://miro.medium.com/max/720/1*V2mgZ7y0ngd3q4DZ01xkEQ.webp)  
그림을 보면 x축은 시간, y축은 주파수 성분이다. 시간 t1일 때, FFT를 통해 얻어진 coefficients는 heat map 형식으로 나타난다. 위의 그림을 보면 전반적으로 Low frequency성분이 강한데, 200초 부근을 보면 high frequency성분이 드문 드문 강하게 튀는 것을 볼 수 있다. 이번 연구 과제에서 우리의 역할은 FFT Architecture를 설계하는 것이지만 FFT의 결과는 Spectrogram형태로 우주 공간에서 과학적 분석을 위해 사용된다고 한다. 따라서 이번 프로젝트에서는 이후의 프로젝트에서도 사용할 검증환경을 GUI로 만들었고, 해당 검증환경에서는 FPGA로부터 받을 원본 데이터의 그래프, PC를 통해 FFT를 수행 후 Spectrogram형태로 출력한 그래프, FPGA가 FFT를 수행한 후 PC로 전송한 데이터를 Spectrogram으로 출력한 그래프를 플롯하고, PC에서 계산한 FFT결과와 FPGA에서 전송받은 FFT결과를 비교하여 Log창에 출력한다. 다음 그림은 우리가 만든 GUI의 Sample이미지다.  
  
![그림4](/assets/img/Sample.png)  
본 프로젝트에서 구현한 GUI인 `SPECTRO`는 [링크](https://github.com/yhkwon6658/Inplace-FFT/tree/main/SPECTRO%20-%20Version%20Hardware)에서 다운 받을 수 있다. 해당 프로그램은 보완이 필요하다고 판단하여 현재는 소스 코드를 공개하지 않고 프로그램만 배포한 상태이다. 해당 링크의 파일을 모두 다운 받은 후 압축을 풀어 `Inplace-FFT/SPECTRO - Version Hardware/SPECTRO - Version Hardware.exe`를 실행하여 프로그램을 동작시킬 수 있다. 자세한 사용법은 `How to run, 3. Run SPECTRO - Version Hardware`에서 기술하도록 한다.

# Paper Analysis
하드웨어에서 FFT를 구현하는 방식은 크게 2가지로 구분된다. 첫번째는 `In-Place FFT`로 `Memory-base FFT`라고도 불린다. 하나의 Memory와 PE(Processing-Element)를 기본 구조로 하며, FFT의 redundancy를 활용하여 Area를 줄이는 방식이다. 두번째 방식은 `Pipelined-FFT`라고 한다. In-Place FFT의 경우 Area와 Power를 크게 줄이는 대신 Latency가 길어지면서 `real time`에서는 사용이 어렵게 된다. 이를 극복하기 위해 제안된 방식이며, 이후 논문 리뷰를 통해 다루도록 한다. 본 프로젝트의 배경이 된 [L. G. Johnson's Conflict Free Memory Addressing for Dedicated FFT Hardware](https://ieeexplore.ieee.org/document/142032)는 In-Place FFT를 하드웨어로 구현하기 위해 `CFA(Conflict Free Addressing)`방법을 제안하였으며, Twiddle factor의 exponent를 선택하기 위한 방법도 제안한다. 최신 연구들도 In-Place FFT 방식의 경우 이 논문의 틀에서 크게 벗어나지 않고 있다. 그러나, Johnson의 연구는 Notation에 대한 설명이 부족하고, 이해를 방해하는 typo가 존재한다. DIF의 경우 설명이 부족하거나 하드웨어로 구현하기 위해 수정된 식에서는 address generator, twiddle factor의 exponent generator의 경우 수식 자체가 제시되지 않았다. 이번 프로젝트는 초기 연구에 대한 정확한 이해를 목표로 하고 있기 때문에 DIF방식에 대하여 모든 수식을 완성하고 다소 비효율적인 address generator를 수정하였다. 구현된 시스템 역시 DIF 방식을 적용하였다. 
  
## I. Notation
DFT equation 

![image](https://user-images.githubusercontent.com/120978778/209723250-34dbfd7a-f405-46ce-a1dd-59f774ceb169.png)

Twiddle factor  
![image](https://user-images.githubusercontent.com/120978778/209723374-956e006e-c757-42be-9880-7bac59c40011.png)  

DFT 수식과 Twiddle factor에 대한 설명은 생략한다.  
Decomposition  
![image](https://user-images.githubusercontent.com/120978778/209724382-83c7dd29-e51e-413c-a282-9791878a37eb.png)  
`본 논문은 LHS의 n_R-2-c, k_c 에서 중간에 `,`가 누락되었다.`  
RHS의 F와 LHS의 F를 보면 DFT의 결과로 n이 하나 줄고, k가 하나 늘어난 것을 알 수 있다. Johnson은 flow chart를 기준으로 stage마다 n과 k로 indexing을 달리 하는데, Radix 4, 64-point FFT를 한다고 가정하면, 최초의 stage 0은 {n0,n1,n2}의 modulo 4로 0부터 63까지의 모든 숫자를 표현할 수 있다. stage 1에서는 {n0,n1,k0}로 표기한다. 이때, 각 자리에 대한 index의 종류가 n에서 k로 변하는 것일 뿐 예를 들어 63은 stage에 관계없이 {3,3,3}을 의미한다. 이때, n0와 n1이 0이라고 가정하면 summation은 F(0,0,0), F(0,0,1), F(0,0,2), F(0,0,3)에 대하여 진행된다는 것을 알 수 있다. 이는 우리가 흔히 알고있는 DIT의 flow와 같다.  
k_c
의 자리에 0, 1, 2, 3을 한번씩 넣어보면 올바른 결과를 얻을 수 있을 것이다.

## II. Memory Banking
Johnson은 다음 수식을 통해 `"flow chart상에서 stage에 관계없이 같은 line에 있는 data는 모두 같은 memory bank에 위치하며 같은 address를 갖는다."`를 설명한다.  

![image](https://user-images.githubusercontent.com/120978778/209725419-4063f5be-8a58-4616-84c6-2751ac05933d.png)  
Flow chart를 보면 Stage,그리고 Stage에서 몇번째 Butterfly인지 관계없이 모든 연산은 하나의 Butterfly에 r개의 data가 들어가고 r개의 data가 나가는 형태이다. In-Place는 이 점에 주목하여 r개의 입력과 r개의 출력을 갖는 Butterfly unit을 하나 만들고, Memory를 r개로 banking하여 1회의 연산에 r개의 memory bank에서 각각 1개의 data를 read하여 butterfly에 입력으로 넣고, 그 출력을 다시 r개의 memory bank에 1개씩 write한다. `데이터를 꺼냈던 곳에 다시 넣는 것`이다. N개의 data는 최초에 물리적으로 r개의 bank에 각각 N/r개씩 저장되어 항상 같은 memory bank, address상에 위치하게 되고, 그 이후 항상 같은 자리에 존재한다. 이제 어떻게 데이터를 Banking하고, addressing하는지 하나씩 이해해 보도록 하자.  

앞으로의 설명은 Radix 4, N = 64, DIT 를 가정으로 한다. `(추신: Johnson의 논문이 DIT를 기준으로 작성되어 DIT로 먼저 설명하려 하는데 구글링을 해도 DIT의 flow chart는 나오지가 않습니다. 이 점 양해 부탁 드립니다.)`

![image](https://user-images.githubusercontent.com/120978778/209726246-438a281c-589a-48cf-96b3-b3b867577d28.png)  
최초의 논문리뷰 레포트에서 발췌한 것이다. 위의 Equation 6을 이용하여 Banking 하게 된다. Johnson의 수식보다 간단해 보이는데, 다음의 설명을 이해하도록 하자.  

F(x2,x1,x0) 를 DFT할 때, Stage에 관계없이 하나의 Butterfly에 들어가야 하는 입력은 x2, x1, x0 중 하나의 digit만 다르며, 나머지는 모두 동일하다. 이때, 편의를 위해 x0를 LSB라 하면, stage 1의 첫번째 butterfly는 F(0,0,0), F(0,0,1), F(0,0,2), F(0,0,3) 이 하나의 butterfly에 입력으로 들어가게 되며, 4개의 memory bank에 partition을 하려고 한다면 각각 0번, 1번, 2번, 3번 memory bank에 들어가면 될 것이다. 그 다음 butterfly는 F(0,1,0), F(0,1,1), F(0,1,2), F(0,1,3) 을 입력으로 갖는다. 4번째 data를 0번이 아닌 1번 bank에 넣는 것이 의아할 수도 있는데, stage 2에서는 butterfly의 간격이 4가 되기 때문에 0, 4, 8, 12번째 data가 하나의 butterfly에 입력으로 들어가게 되며, 결과적으로 간격이 1일 때, 간격이 4일 때, 간격이 16일 때는 서로 다른 bank에 위치해야 한다. Radix-4를 사용하고 있기 때문에 Butterfly는 날개의 간격이 1, 4, 16이 된다. 따라서 이 간격에 해당하는 데이터들은 서로 다른 Bank에 위치해야 하고, 이들은 F(x2, x1, x0)에서 단 하나의 digit만 다르기 때문이 서로 다른 하나의 digit을 기준으로 Banking을 해야 하고, Equation 6을 다시 보면 정확하게 이를 표현하고 있다는 것을 이해할 수 있다. 논문에서는 보다 복잡해 보이는 notation을 사용하고 있는데, 동일한 뜻을 갖는다. 이후의 논문들에서는 이를 `modulo-addition` 이라는 용어로 사용되고 있다.

## III. CFA(Conflict-Free-Addressing)
F의 파라미터 개수를 다시 생각해보자. Modulo r로 N개의 data를 표현하기 위해서 필요한 파라미터는 log_r (N) 개이다. 다시 앞서 사용한 Radix 4, 64-point를 가정하면 3개의 파라미터를 갖고, 이는 stage 개수와 동일하다. 이때, Bank는 4개로 구성된다. 각 Bank는 16개의 address를 갖는다. 3개의 파라미터 중 2개의 파라미터를 address로 사용할 수 있는데, 논문에서는 다음과 같이 하였다.
![image](https://user-images.githubusercontent.com/120978778/210101483-533614f7-eae2-41cd-91b8-f5ff2e501460.png)  
이때, c는 stage를 의미하며, index는 0부터 시작한다. R은 log_r (N) 의미한다. 현재의 예시를 가정하면 c는 {0, 1, 2}를 집합으로 하며, R은 3이다. Butterfly에 들어가는 입력은 Stage 0일 때, LSB에 해당하는 파라미터만 다르고, Stage 1일 때는 2번째 파라미터만 다르고, Stage 2일 때는 MSB에 해당하는 파라미터만 다르다. 즉, Stage에 따라서 다른 파라미터는 한 자리씩 옮기게 된다. DIF의 경우는 반대이다. 각 Bank는 16개의 address line을 갖기 때문에 3개의 파라미터 중 2개의 파라미터를 선택하면 모든 address를 표현할 수 있다. 2개의 파라미터는 어떻게 선택하든 관계 없지만 Johnson은 DIT일 때는 하위 2개의 digit, DIF일 때는 상위 2개의 digit을 갖도록 알고리즘을 설계했다.

## IV. Exponent generation
![image](https://user-images.githubusercontent.com/120978778/210103249-b2e8d11a-d003-4219-96fc-fdd7d8387115.png)  
Twiddle factor의 경우 특별한 설명없이 결과식이 제시되었다. 
![image](https://user-images.githubusercontent.com/120978778/210104342-4f0e17bf-b95e-4da8-bcbf-462bcbeac4b6.png)  
Radix-4의 경우 DIT에 해당하는 flow chart가 없어 Radix-2로 가장 간단한 8-Point를 생각해보도록 하자. Stage 1에서는 W_8(0), W_8(2)를 갖는데, Twiddle factor의 성질에 다라서 W_4(0), W_4(1)을 갖는다고 볼 수 있다. Stage 2에서는 W_8(0), W_8(1), W_8(2), W_8(3)을 갖는다. 이를 다시 생각해보면 Stage 를 거칠 때마다 경우의 수가 r배씩 증가하고, 이를 수식에서는 k라고 표현하였는데 stage 1에서 k는 k0, stage 2에서 k는 k1k0를 의미한다. 1개의 digit이 증가하면서 표현할 수 있는 수가 r배 증가했기 때문에 mod r^c 로 stage 마다 modulo 를 r 배씩 증가시킨다. stage 0에 k는 존재하지 않는데, 이 논문에서 존재하지 않는 값은 모두 0으로 간주해야 한다. stage 1에서는 n_1 * (k0 mod 2)가 된다. 제시된 flow chart에서 x(6)을 F(1,1,0)으로 착각할 수 있는데, 이 논문에서는 몇번째 라인에 위치하는지를 기준으로 하기 때문에 x(6)은 F(0,1,1) 이라는 것을 주의해야 한다. 파라미터 n은 stride를 결정짓는 위치에 존재하는 파리미터를 선택하는 것을 알 수 있다. 이는 twiddle factor가 butterfly상에서 2번째 위치에서만 달라지고 1번째 위치에서는 항상 W_N(0)이기 떄문이다. 위의 그림에서 N에 해당하는 값은 2, 4, 8로 증가하고, n은 stride를 결정하는 파라미터, k는 가짓수를 결정하기 위해 사용되어 1, 2, 4가 되도록 한다. 최대한 결과식의 의미를 일반적으로 읽을 수 있도록 해봤는데 수학적인 증명보다는 flow chart를 보면서 그 특징을 찾아 수식화 되었을 가능성이 크다고 생각한다. 이후, 하드웨어로 구현하기 위해서 변형되는 수식에서도 결과식만 제시 되는데 같은 방법으로 해석하면서 여러가지 케이스에 대하여 검증해 보는 것을 추천한다.  
`(추신: DIF일 때, radix-4일 때도 직접 시도해 보는 것을 추천드립니다.)`

## V. For Hardware Implementation
Johnson은 하드웨어로 구현하기 위해 다음 식을 제안한다.
![image](https://user-images.githubusercontent.com/120978778/210106273-0063840c-c0ee-4212-8120-bd65879874d9.png)  
앞서 제안된 F보다 파라미터 수가 1개 줄어들었음을 캐치해야 한다. Memory는 r개로 Banking되었기 때문에 1번에 4개의 data를 모두 read/write할 수 있어야 한다. 여기서 Johnson은 modulo r을 사용하는 R-1개의 digit으로 구성된 b라는 파라미터를 만들었다. b는 modulo r 을 사용하는 up counter인데, 각 stage에 따라서 달라지는 1개의 파라미터는 w로 표현한다. 이 w는 0,1,2,...,r-1을 barrel shifter를 이용하여 계속 rotate시키면서 각 memory bank에 1대1로 대응시킨다.  
![image](https://user-images.githubusercontent.com/120978778/210106799-9420c9f0-ad11-4f39-bffa-a5a0541e66db.png)  
이때, barrel shifter의 selection 신호에 해당하는 s는 b를 모두 modulo addition하여 결정한다. 각각의 memory bank는 s를 통해 w중 0, 1, 2, ... , r-1 중 하나의 값을 가질 수 있다.
![image](https://user-images.githubusercontent.com/120978778/210107059-18baae8e-6a5c-472e-9a61-caadb3ac1741.png)  
앞서 F에 대한 수식과 비슷한데, b에 따라 선택된 s가 1이라면, 3번 bank에는 2라는 w가 들어가야 할 것이다.
![image](https://user-images.githubusercontent.com/120978778/210107230-559eae23-08a9-445d-9180-04458b580e41.png)  
제안된 address generator이다. s에 따른 barrel shifter의 동작은 위에 언급한 수식을 만족할 수 있도록 설계되어야 한다. w는 stage에 따라 달라질 수 있는 1개의 파라미터를 의미한다. 이미 우리가 앞에서 이해한 바에 따르면 Radix-4, 64-Point라면 address는 stage에 관계없이 F(x2, x1, x0)에서 x1x0이다. 이때, b1b0와 stage에 따라 달라지는 w로 이를 표현하려 한다면 stage 0에서는 b1b0w, stage 1에서는 b1wb0, stage 2에서는 wb1b0이다. stage 에 따라 관계없이 하위 2개의 digit이 각각 a1a0이다. a1은 stage 순서대로 b0, w, b1이 된다. a0는 stage 순서대로 w, b0, b0 가 된다. 이를  정리하면 a_i 는 i번째 stage 이전에는 b_(i-1) 를 갖다가 i번째 stage 에는 w, i번째 stage 이후에는 b_i를 갖는다.  
![image](https://user-images.githubusercontent.com/120978778/210112406-03579779-ee05-4aee-b7ce-e921d75a1385.png)  
Johnson은 이를 위와 같이 정리했다. 다소 구간을 나누는 기준이 썩 좋아 보이진 않는다. 따라서 우리는 이후 `VI. Case of DIF`에서 조금 더 깔끔하게 구간을 나누도록 할 것이다.  
![image](https://user-images.githubusercontent.com/120978778/210108050-6bb9c86e-0df5-4546-9fe5-4ff9ffbbcd3b.png)  
b에 따라서 s를 달리하고, barrel shifter를 이용하여 각 memory bank의 address를 결정하여 각각의 bank에서 data가 출력되면, butterfly에 들어가는 data를 mapping해 주는 것도 필요하다. 3번 뱅크에서 0번 날개에 들어가야 할 데이터가 나와 연산이 되었다면, 0번 날개에서의 출력은 다시 3번 뱅크로 들어가야 한다. 이때, butterfly의 입력을 만들기 위한 barrel shifter는 앞서 address generator에서 만든 것과 동일한 것을 사용하면 되지만 butterfly의 출력을 memory bank에 저장하도록 하는 barrel shifter는 이를 거꾸로 하도록 설계해야 한다.  
![image](https://user-images.githubusercontent.com/120978778/210108542-d8efb74f-45d9-423a-9ec9-2c3b5a789149.png)  
3항 중 1항에 대한 해석은 이미 진행 하였다. 1항에서 2항으로의 변환에 대한 해석은 제시되지 않았다. 결과식의 w는 n_(R-1-c)를 의미한다. Twiddle factor의 밑변이 N일 때, 윗변이 r^(R-1-c)인 것은 1항의 r^(c+1)을 의미한다. b_(c-1),...,b_0 는 k mod r^c 를 대응시킨 결과인데, 시작 index가 c-1이기 때문에 c가 0일 때, b는 0이고 , c가 1일 때, b0, c가 2일 때, b1b0가 되고, 앞서 (k mod r^c)로 가짓수를 표현한 것과 동일한 결과라는 것을 알 수 있다.  
![image](https://user-images.githubusercontent.com/120978778/210110793-806afa83-7339-46af-9b27-02bef46e44ee.png)  
Johnson이 제안한 architecture를 보면 exponent를 선택할 때, 0번 날개의 경우 무조건 0이 되기 때문에 직접 표현하지 않았다. w가 2일 때는 left shift 2라고 써두었는데, 표현되지 않았지만 이는 bit단위에서 연산을 의미한다는 것을 우리는 이미 알고 있다. 이 논문에서는 digits과 bits를 특별한 언급없이 사용하고 있기 때문에 이를 주의해야 한다. b는 modulo r counter 이지만 exponent generator는 bit단위로 연산한다는 것만 주의하도록 하자.

## VI. Case of DIF
![image](https://user-images.githubusercontent.com/120978778/210111058-d6168263-55cf-4b45-af88-90266ef51bbc.png)  
DIF는 비교적 간단하게 설명할 것이다. 여기서 n_1은 typo다. n_0로 수정해야 한다. 이 논문이 어려운 이유는 notation에 대한 설명없이 제시된 F라는 식이 DIT, DIF모두 typo를 포함하고 있기 때문이다. 좌변의 f도 F와 동일한 의미로 특별한 차이는 없다.  

Memory banking은 DIT, DIF 관계없이 동일한 알고리즘을 따른다.  

![image](https://user-images.githubusercontent.com/120978778/210111239-c6ccff2e-28ac-4020-998c-cd4e9e6e1e3a.png)  
Addressing은 상위 R-1개의 digits을 선택한다.  
![image](https://user-images.githubusercontent.com/120978778/210111548-6f1a842b-1b50-466e-ade7-2503748ddecc.png)  
DIF일 때는 Twiddle factor의 해석이 더 어려울 수 있는데, DIF에서는 Twiddle factor가 butterfly의 출력으로 next stage의 indexing을 사용한다. 이 부분을 인지하면 DIT때와 똑같은 방식으로 해석할 수 있다.  
![image](https://user-images.githubusercontent.com/120978778/210111790-0832b3bd-2663-4d00-a7b2-1faa665d2535.png)  
Johnson은 하드웨어로 구현하기 위해 제안된 수식이 DIF는 직접 밝히지 않았고, 특정 index가 음수를 가지면 알아서 0으로 이해해야 한다거나 하는 다소 비효율적인 서술 방식을 쓰고 있기 때문에 우리는 DIT와 DIF의 address수식을 변화시킴으로써 Johnson이 제시한 것보다 깔끔한 수식을 제안하였다. Johnson이 제안한 address generator보다는 우리가 제안하고 있는 수식에 따라 구현하는 것이 훨씬 깔끔할 것이라고 생각한다.  
![image](https://user-images.githubusercontent.com/120978778/210112079-e555d6fc-c7c5-4360-8128-da6bd31c1e97.png)  
Johnson은 DIF의 경우 Twiddle factor의 식은 제시하지 않았다. 앞서 DIT에서 해석했던 것과 마찬가지로 w는 k_c를 의미하며, r^c / N 은 r^(R-c)를 의미한다. DIF에서는 digits수가 점점 줄어들어야 하기 때문에 시작 index를 R-2-c로 하면 끝이다. 이때, 시작 index가 음수가 되면 b를 0으로 처리한다.

# Block Diagram
![image](https://user-images.githubusercontent.com/120978778/210112541-7dd62ef1-1478-4905-afbb-6e632adf2d64.png)  
<span style='background-color: #fff5b1'>TESTROM</span>: 실험을 위해 사용할 데이터들을 닮고 있다. 많은 데이터를 메모리에 저장하여 검증할 필요는 없기 때문에 빠르게 사용할 수 있도록 on-chip memory로 구현하였다.  
<span style='background-color: #fff5b1'>UART CONTROLLER</span>: UART는 기본적으로 8bits의 data를 한 번에 주고 받을 수 있다. 우리는 IEEE-754 Single Precision을 사용하고 있기 때문에 real part 32bits, imaginary 32bits를 포함하여 총 64bits를 FPGA에서 PC로 보내야 한다. 이를 위해서 8bits씩 8회를 연속으로 PC로 보내고, PC쪽에서는 Buffer가 Ready상태에 오면 Read를 수행하게 함으로써 64bits를 받을 수 있도록 하였다. 또한, PC에서 ASCII의 'R'을 전송하면 UART CONTROLLER는 FFT CONTROLLER로 `o_tx_ready` 신호를 보내게 된다.  
<span style='background-color: #fff5b1'>FFT CONTROLLER</span>: 2개의 memory bank, 1개의 bank initialize module, 1개의 exponent generator, 1개의 twiddle factor generator, 1개의 butterfly, 1개의 address generator, 1개의 find data module로 구성된다. FFT CONTROLLER는 처음 `i_tx_ready`신호가 high가 되면, TESTROM에서 순서대로 하나씩 data를 받아 `o_fft_data`와 함께 `o_tx_valid`신호를 UART CONTROLLER로 보내 TESTROM의 raw data를 1개 전송한다. UART CONTROLLER에서는 이를 받아 PC로 전송하고, PC에서 다시 'R'을 대답하면 UART CONTROLLER에서 FFT CONTROLLER로 tx_ready 신호가 전달된다. 이 작업은 TESTROM의 모든 data를 하나씩 전달할 때까지 이루어 진다. TESTROM의 모든 raw data를 전달한 후 다시 'R'신호가 들어와 tx_ready 신호가 FFT CONTROLLER로 전달되면, FFT CONTROLLER는 Bank initialize module을 통해 butterfly가 Johnsnon의 Stage 0의 연산을 수행할 수 있도록 TESTROM에서 raw data를 가져와 2개의 memory bank에 집어 넣는 과정을 거친다. 그 다음, FFT를 진행할 때는 address generator를 통해 memory bank의 address를 결정하고, exponent generator와 twiddle factor generator를 통해 butterfly에 입력으로 들어갈 twiddle factor를 결정하게 된다. 1회의 FFT가 끝나게 되면, Find Data Module은 FFT의 출력이 bit-reverse되는 것을 고려하여 오름차순으로 순서대로 memory bank의 address를 생성한다. 그 다음, memory bank에서 나오는 출력을 `o_fft_data`에 넣어 `o_tx_valid`신호를 보낸다. 그러면 UART CONTROLLER는 이를 PC로 보내고 다시 'R'신호를 받아 tx_ready 신호를 FFT CONTROLLER로 전달한다. 이렇게 주고 받는 과정을 FFT-point수만큼 반복한 후 TESTROM의 모든 data에 대하여 FFT를 수행할 때까지 Bank initialize module단계부터 다시 시작하게 된다. 이 모든 과정은 `FSM`을 통해 구현 되었다. 

# How to run
## 1. Data Processing
[Github](https://github.com/yhkwon6658/Inplace-FFT) 에서 모든 코드는 다운로드 받았다고 가정하도록 하겠습니다.  
우선 DATA/Resampler.m 을 열도록 합니다.  

```matlab
lear;
clc;
close all;
%% Put the Input file name and Output file name
%% Output file must be .txt
infile = "input\chopin_etude_op25_no11.mp3";
outfile = "C:output\sample.txt";
[y, Fs_ori] = audioread(infile);
%% Select Option(Start time, End time, Sample rate, Write mode)
START = 5;
END = 6;
Fs = 8192;
mode = 0; % 0 : SW, 1 : HW

y = y(round(START*Fs_ori):round(END*Fs_ori));
x = resample(y,Fs,Fs_ori);
L = length(x)-1;
fileID = fopen(outfile,'w');
if (mode == 0)
    fprintf(fileID,'%f\n',x);
else
    xx = zeros(1,2*L);
    for i = 1:L
        xx(2*i-1) = x(i);
        xx(2*i) = 0;
    end
    formatSpec = '%tx%tx\n';
    fprintf(fileID,formatSpec,xx);
end
fclose(fileID);
fprintf('Sample Rate: %d\n', Fs);
fprintf('# Sample: %d\n',L);
fprintf('DONE\n');
```
1) infile에 입력 데이터의 path, outfile에 출력 data의 path를 써주도록 합니다.  
2) 입력으로 넣은 음원 파일을 Sampling할 시작 시간과 종료 시간을 설정합니다.  
3) Sample Rate(Sampling Frequency)를 입력합니다. 현재의 코드는 8192개의 Sample을 출력합니다.
4) mode를 1로 합니다.  

출력된 결과는 IEEE-754 Single Precision으로 64bits를 hexadecimal로 출력합니다.  
Johnson은 Twiddle factor를 `CORDIC`등을 통해 직접 만드는 것이 아니라 `LUT`에 저장하여 exponent값을 만들어 LUT에서 Twiddle factor를 꺼내서 쓰는 방식을 선택했습니다. 따라서 Twiddle factor의 값을 만들어 내야 합니다. DATA/twiddlegen.c 를 열어 16번 라인 N의 값을 바꿔서 Twiddle factor 값들을 만들어 냅니다. 이때, 21번 라인의 이름을 바꿀 수 있습니다. 64bits를 hexadecimal로 합니다.

## 2. Synthesis and Porting
저희는 radix-2에 맞추어 모듈을 설계하였습니다. 현재 시스템은 256개의 Sample data에 대하여 32-point FFT를 수행하도록 되어 있습니다. 경우에 따라서 `Top.v`의 다음 파라미터들을 수정할 수 있습니다.  
```verilog
module TOP #(
    parameter CLKS_PER_BIT = 434,
    parameter SIG_RUN = 82,
    parameter SIG_STOP = 83,
    parameter DATA_LENGTH = 256,
    parameter R = 5,
    parameter N = 32,
    parameter radix = 2,
    parameter length = 32,
    parameter ROMFILE = "TESTDATA.txt",
    parameter TWIDDLEFILE = "TWIDDLE.txt"
)
```
<span style='background-color: #fff5b1'>CLKS_PER_BIT</span>는 FPGA의 `system clock frequency` 를 UART 통신에 사용할 `Baudrate(Bitrate)`으로 나눈 값입니다.  
<span style='background-color: #fff5b1'>DATA_LENGTH</span>는 Sample Data 개수입니다.  
<span style='background-color: #fff5b1'>R</span>은 log_2(N)에 해당하는 값입니다.  
<span style='background-color: #fff5b1'>N</span>은 FFT-point 수에 해당하는 값입니다.  
<span style='background-color: #fff5b1'>ROMFILE</span>은 Resampler를 통해 만든 데이터 파일 이름입니다.  
<span style='background-color: #fff5b1'>TWIDDLEFILE</span>은 twiddlegen을 통해 만든 twiddle factor 데이터 파일 입니다.

만약, N을 64, R을 6으로 바꿨다면, `Addrgen 모듈`을 수정해야 합니다. 현재는 Addrgen 모듈 내부에 ADDRgen_element가 4개 instantiation 되어 있습니다. 이를 5개로 늘려야 합니다. 현재까지의 글을 모두 읽었다면 Connection은 쉽게 할 수 있을 것이라 생각합니다. 

## 3. Run SPECTRO - Version Hardware
SDK를 통해 synthesis된 파일이 FPGA위에 정상적으로 포팅이 되었다면 SPECTRO.exe를 실행합니다. Datainfo를 누른 후 `Sample length`를 입력합니다(다른 파라미터는 수정할 필요 없습니다). Setting을 누른 후 `Serial Select Port`를 통해 FPGA와 연결된 포트를 선택합니다. `BaudRate`를 설정합니다(115200을 사용하는 것을 추천합니다). Connect를 누른 후 Run을 수행합니다. `Origianl Plot`과 `FFT by PC`가 정상적으로 Plot되면 Stop을 누른 후 다시 Run을 누릅니다. 가끔씩 UART 통신의 문제로 data가 1bit씩 밀려서 전송될 수 있습니다. 이 경우 FPGA를 reset시킨 후 다시 SPECTRO를 실행하면 보통 문제없이 정상적으로 동작합니다.

![DEMO](https://user-images.githubusercontent.com/120978778/210115307-8ddc62be-96ad-420c-a57b-b1d52ca96a3a.gif)  
참고용 데모영상입니다.