---
layout: post
title: "A Pipelined FFT Architecture for Real-Valed Signals"
author: "Yonghwan Kwon"
tags: "Review"
comments: true
excerpt_separator: <!--more-->
---
기존의 FFT Architecture 에 대한 연구는 complex fourier fast transform (CFFT) 에 대하여 주로 이루어 졌다. `Phari` 교수의 연구진은 실제로 FFT 에 사용되는 입력이 대부분 Real Value 라는데 주목하여 RFFT 의 flow chart 를 제안하였으며, Pipelined RFFT Architecture 와 In-place RFFT Architecture 를 모두 제안하였다. 이번 논문은 Pipelined RFFT Architecture 를 다루고 있으며, 논문의 핵심 쟁점인 `Conjugate symmetry`, `RFFT flow chart`, `Pipelined RFFT Architecture` 에 대하여 정리하고자 한다. <!--more-->  

# Link
https://ieeexplore-ieee-org-ssl.webgate.khu.ac.kr/document/4799153

# Conjugate Symmetry
![image](https://user-images.githubusercontent.com/120978778/212918653-91f369d2-59d3-487c-bc1b-faca86923b65.png)  
위의 식은 일반적으로 알려진 DFT 수식이다. 이때, 입력이 Real value 이면 다음 식이 성립한다.  
![image](https://user-images.githubusercontent.com/120978778/212919161-d63d9b35-af34-4606-8168-4829c51533c1.png)  
위의 식은 Twiddle factor 의 주기성에 의해 DFT 식에 X[N-k] 와 X*[k] 를 대입하면 증명된다.  

# RFFT flow chart
본 논문에서는 Conjugate Symmetry 에 근거하여 RFFT 의 algorithm 으로 제안되었던 몇가지 방법을 소개하고, 해당 방법들이 갖는 한계점을 간략하게 지적한다. 이번 포스팅에서는 그것들을 다루기 보다는 해당 논문에서 제안된 RFFT algorithm 에 근거한 flow chart 에 집중하도록 한다. <br/>
Conjugate Symmetry 의 결론은 출력단에서 (N-2)/2 개의 데이터는 서로 대칭이라는 것이다. RFFT 의 핵심은 이 출력의 절반`(거의)`을 flow chart 에서 제거하는 것이다. 문제는 flow chart 상에서 살릴 절반과 죽일 절반을 선택하는 것인데, 기존의 연구들은 살릴 절반 k 를 [0, N/2] 혹은 [0, N/4] U [N/2, 3N/4] 로 선택하였다. 
![image](https://user-images.githubusercontent.com/120978778/212921607-564e5b51-c3b5-4b1c-80b7-491ab4e9390a.png)  
위의 그림은 DIF 의 flow chart 이다. 여기서 k = [0, N/2], k = [0, N/4] U [N/2, 3N/4] 로 선택하여 선택되지 못한 나머지 출력 부분을 지우고 이에 연결된 선들을 지워 보자. 그렇게 할 경우 지워지는 영역은 별로 크지 않다. 본 논문에서는 이 점에 주목했다. 최대한 많은 영역을 지우고자 할 때, floaw chart 를 기준으로 절반을 지우고 싶겠지만 그것은 불가능하다. 다시 flow chart 의 출력단을 보도록 하자. 0과 8은 conjugate symmetry 에 해당하지 않는다. 4의 대칭은 12이고, 2와 10의 대칭은 14와 6이다. 1, 9, 5, 13 의 대칭은 15, 7, 11, 3 이다. Logarithmatic 으로 대칭이 되는 개수가 늘어나는 것을 알 수 있다. 조금만 생각해 보면 Logarithmatic 에 따라 선택하는 것이 최대한 많은 영역을 지울 수 있다는 것을 눈치챌 수 있을 것이다. 논문에서는 이를 수식으로 나타내고 있지만 이번 포스팅에서는 굳이 언급하진 않도록 하겠다.  
![image](https://user-images.githubusercontent.com/120978778/212924558-356bfad7-8c21-43c9-a9bc-331b189490c6.png)  
Logarithmatic 으로 절반을 선택하여 죽은 부분을 지운 flow chart 이다. 저자는 여기서 멈추지 않고 flow chart 를 더 수정한다. Twiddle factor 의 주기성을 활용하는 것인데, 다음 수식을 먼저 보도록 하자.  
![image](https://user-images.githubusercontent.com/120978778/212925778-0bf8572d-ca07-4668-9e6d-507588cb0037.png)  
여기서 i = 0, 1, 2, 3 이며, 좌변은 2번째 stage 의 출력을 의미한다. 이때, 다음이 성립한다.  
![image](https://user-images.githubusercontent.com/120978778/212928831-8255b2cf-e696-4679-82a5-aada36a7961c.png)  
이 식에 따라 X_2 의 식은 다음과 같이 바뀐다.  
![image](https://user-images.githubusercontent.com/120978778/212930665-656687c7-0318-4cdf-8fc2-f2f799ca7f41.png)  
이를 통해 Twiddle factor 를 곱하는 연산이 한 번 줄어들게 되는데, j 를 곱하는 의미를 잘 생각해 보면, real value 가 imaginary value 로 바뀔 뿐 값에는 변화가 없다. 저자는 이 점을 이용하여 다음과 같이 flow chart 를 변형한다.  
![image](https://user-images.githubusercontent.com/120978778/212931371-5fdf55c9-062f-481a-a17c-b6d941057eed.png)  
넘버링이 된 박스의 윗 라인은 real value 를 아래 라인은 imaginary value 를 의미한다. 박스 안의 숫자는 두 개의 입력에 각각 곱해질 Twiddle factor 의 exponent 를 의미한다. 이를 이용하여 Conjugate symmetry 에 의해 지워진 부분을 imaginary path 로 만드는 선택을 했다. 사실 다음 포스팅에서 다룰 In-place RFFT Architecture 와 이번 논문에서 다룰 Pipelined RFFT Architecture 는 Architecture 자체가 놀랍다기 보다는 위의 RFFT flow chart 를 만들어 내는 과정이 더 주목 받는다. 이후의 논문들은 이 flow chart 가 가진 단점을 보완하기 위한 접근을 시도하고, 이 논문에서 제시된 Architecture 를 보완할 수 있는 방법을 찾고자 노력한다. 그러나, 최근의 연구들이 지난 Project 글에서 다룬 Johnson 의 [Conflict Free Memory Addressing for Dedicated FFT Hardware](https://ieeexplore.ieee.org/document/142032) 논문에서 제안된 In-Place Architecture 를 크게 벗어나지 못하는 것처럼 이 논문 이후의 연구들도 틀 자체를 벗어날 만큼의 시도는 하지 못하였다. 
 
# Pipelined RFFT Architecture
RFFT flow chart 는 일반적인 형태의 Butterfly, 2 종류의 넘버링 된 박스, 특별한 연산 없이 그냥 패스되는 부분으로 구성된다. 본 논문에서는 다음과 같은 architecture 를 제안한다.  
![image](https://user-images.githubusercontent.com/120978778/212939827-ef84f534-9586-4c6f-aceb-0a285a0bdc80.png)  
여기서 고려해야 하는 것이 이 논문에서는 R2 와 ROTATOR 로 표시했지만 R2 는 Mux 에 의해 일반적인 Butterfly 와 연산이 없는 패싱을 선택적으로 수행하고, ROTATOR 는 Mux 에 의해 imaginary value 에 -1 을 곱한 후 Twiddle factor 를 곱하는 연산과 그냥 Twiddle factor 를 곱하는 연산을 선택적으로 수행한다.  
또한, Stage 2 부터 다음 Stage 사이에 2개의 Mux 와 FIFO 로 구성된 회로가 존재하는데, 논문에서는 이를 `Shuffling structure` 라고 지칭한다. Shuffling structure 는 stage 가 변함에 따라 butterfly 의 stride 가 변하는 것에 대응하기 위해 제안된 구조이다.  
stage 2 와 stage 3 사이의 Shuffling 은 다음과 같이 일어난다.  
![image](https://user-images.githubusercontent.com/120978778/212940889-ef8b8057-91f7-46bb-a611-39349ecad2e3.png)  
이 논문에서는 selection 신호의 발생 조건을 따로 언급하지는 않고 있다. 이 논문을 바탕으로 하여 제안된 논문들은 보통 selection 신호의 조건을 제안하고 있는 것으로 보인다.  
![image](https://user-images.githubusercontent.com/120978778/212941469-56f06192-88c1-431c-ad34-deac2958534a.png)  
위 그림은 전체적인 flow 를 만족시키기 위해 직접 그려본 것인데, 출력을 보면 0, 1, 2, 3, 4, 5, 6, 7 은 순서대로 출력이 되지만 8, 9, 12, 13 그리고 10, 11, 14, 15 가 출력되어 순서대로 출력되지 않는 것을 알 수 있다. 이 논문에서 SWITCH 의 동작 조건을 특별히 언급하고 있지 않기 때문에 임의로 조건을 달아 flow 를 만족시킬 수 있도록 했는데, path 상 가장 상단 라인과 가장 하단 라인은 SWITCH 의 영향을 받지 않기 때문에 8, 9, 12, 13 그리고 10, 11, 14, 15 가 출력되는 것은 여기서 임의로 단 조건이 틀리지 않음을 알려준다. 논문에서는 모두 순서대로 출력된다고 언급하고 있는데 실수가 있었던 것으로 보인다. 이를 해결하기 위해서 하단의 마지막 Shuffling structure 앞에 SWITCH 를 달아 10, 11 과 12, 13 을 SWITCHING 시키면 이 문제를 해결할 수 있다.  

# Limitation
사실 이 논문은 꽤나 많은 문제점이 존재한다. 앞서 언급하였지만 R2 와 ROTATOR 의 세부 구조를 제안하고 있지 않다. 그러나, 이후에 제안된 논문들을 바탕으로 선택적으로 동작함을 알 수 있고, 그에 따라 해석하면 동작을 이해할 수 있었다. 구조가 제안된 Shuffling structure 또한 selection 신호의 조건이 언급되지 않았다. 전반적인 Control 조건이 제대로 기술되지 않은 것이다. 또한, RFFT flow chart 의 경우 중간을 기준으로 비대칭적인 구조를 가지고 있기 떄문에 single path 로 architecture 를 제안하는데 어려움이 있었던 것으로 보인다. 물론, multi path 인지 single path 인지는 area 와 performance 중에서 우선이 되는 기준에 맞추어 선택하는 것이지만 현재의 flow chart 는 single path 를 선택할 수 있는 방법이 제안되지 않은 것을 알 수 있다. 지금까지 언급한 부분들은 이후의 후속 연구들에서 전반적으로 해결되었다. 그러나, 다음의 문제는 꽤나 치명적이다. 결국, 이 구조의 선택에 제약 사항이 되는데, Imaginary path 를 따로 두면서 CFFT 일 때는 DIF 는 출력 쪽이 DIT 는 입력 쪽이 bit reverse 되었던 패턴이 사라지게 되었다. 문제는 이 출력을 어떤 기준으로 읽어서 처리할 지가 되는데, 이는 온전히 알고리즘으로 정리하기가 매우 어려워 보인다. 예를 들어, X[3] 이 필요하다면 우리는 X[13] 을 선택했기 때문에 R13 과 I13 을 꺼내야 한다. 이때, R13 은 13번 I13 은 15번이다. R과 I가 존재하기 때문에 한 번에 2개의 데이터씩 꺼내는 것이 합리적이라 생각되지만 애석하게도 X[0] 과 X[N/2] 는 real value 만 갖기 때문에 최소 한 번은 동시에 3개의 데이터를 꺼내야 한다. 이 출력을 꺼내는 기준을 일반화할 수 있는 방법이 현재까지는 제시되지 않았다. 아무래도, RFFT Architecture 의 출력을 순서대로 일단 메모리에 저장해 두고 Read 조건을 만들어 메모리에서 데이터를 순서대로 꺼내 필요한 곳으로 전달해야 할 것인데, 이 쪽의 오버헤드가 너무 크고 일반화할 방법이 제시되지 않았다. 이는 이후의 논문들에서도 해결되지 않은 문제점인데, 이를 바탕으로 포인트 수가 매우 많아지면 뒤 쪽의 오버헤드가 매우 커지기 때문에 실제로 이를 선택하는 것은 매우 어려운 일이 된다. 마지막으로, ROTATOR 에 사용되는 Twiddle factor 의 exponent 생성 조건은 거의 대부분의 Pipelined Architecture 에서는 언급하고 있지 않다. Pipelined Architecture 가 real time 에 강한 것은 맞지만 결국 point 수가 커지면서 stage 가 늘어나 area 가 크게 증가하고, RFFT 의 경우 출력의 read pattern 과 exponent generation 조건이 일반화 되지 않는 한 사용이 꺼려지는 것이 사실이다.  

# Appendix
이 논문에서는 특별히 언급하고 있진 않지만 다음과 같은 DIT RFFT flow chart 를 보여주고 있다.  
![image](https://user-images.githubusercontent.com/120978778/212948549-9e167567-74aa-48d6-8106-85a414e807f7.png)  
이 논문에서 얻을 수 있는 하나의 교훈같은 것인데, 기존의 우리가 알고 있던 DIT 는 DIF 와 대칭적인 형태였던 것을 마치 DIF 와 동일해 보이는 형태로 쓰고 있는 것이다. 이는 사실, DIF 이든 DIT 이든 STAGE 1 에서는 0 번과 8 번이 만나야 하기 때문에 기존의 DIT chart 에서는 입력쪽을 bit reverse 시켰던 것인데, DIF 처럼 그려두고 0 과 8 을 묶어도 전혀 문제가 되지 않는다. Twiddle factor 가 표시되는 위치만 변하게 되는 것이다. 이는 논문에서는 특별히 언급하고 있지 않지만 꽤나 괜찮은 접근이라 생각되어 따로 언급하도록 한다. <br/>
하나 더 언급되는 것이 있는데, Appendix 의 bit-reverse 회로의 대체에 대한 언급이다. bit-reverse 를 언급하고 있는데 다소 의아한 부분이다. 애초에 논문에서 완성된 flow chart 는 real value path 와 imaginary value path 가 나뉘어 출력되기 때문에 bit-reverse 는 깨지게 된다. 아무래도, Appendix 로 뺀 것은 이 때문으로 보이는데, imaginary path 를 따로 두기 전 출력의 절반만 선택해서 잘라낸 flow chart 를 기준으로 생각하면 문제가 되지 않는다.  
![image](https://user-images.githubusercontent.com/120978778/212954603-b651315e-0254-400b-b961-9736c92261ad.png)  
이는 32-point 의 예시이다. 일단, OUTPUT FREQUENCIES 에서 16을 넘는 숫자들은 모두 16을 빼서 OUTPUT ORDER 를 0 부터 16 까지의 숫자들로 고려한다. 그 다음 표시된 박스와 화살표를 따라가면 출력 결과가 bit-reverse 된 형태로 나오게 된다. 이를 다시 표현한 것이 다음 그림이다.  
![image](https://user-images.githubusercontent.com/120978778/212955652-cd464efa-1d58-4872-b233-871c2a29526e.png)  
대충 보면 알겠지만 한 칸이 바뀌거나 두 칸이 바뀐다. 이는 앞서 사용했던 Shuffling structure 를 이용하여 해결할 수 있다. 그림만 봐도 알 수 있는데, L= 1, L = 2, L = 2, L = 1 짜리 Shuffling structure 를 이어붙인 후 위 그림에 따라 Selection 신호를 줘서 Shuffling 시키면 bit-reverse 된 형태로 결과가 나타나게 된다.