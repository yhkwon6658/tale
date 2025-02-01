---
layout: post
title: "CNN Backpropagation"
author: "Yonghwan Kwon"
tags: "CNN"
comments: true
excerpt_separator: <!--more-->
---
CNN은 일반적으로 Convolution Layer와 Fully-Connected Layer로 이루어진 형태를 기본 구조로 하고 있다. FC Layer의 경우 여러 매체에서 Back-Propagation을 자세히 설명하고 있다. 이번 포스팅에서는 Convolution Layer의 Back-Propagation을 FC Layer와 동일한 방식으로 설명하고자 한다.  
<!--more-->

# Back-Propagation
Neural Network(NN)는 학습을 위해 Back-Propagation 알고리즘을 사용한다. 직역하면 `"뒤로부터 전달."`정도로 이해하면 될 것인데, 나름 이유가 있다. 항상 NN을 학습하기 위해서는 `Label` 이라 하는 정답지가 필요하다. NN에 입력을 넣어 얻은 출력과 답지를 비교하여 `Cost(Error)` 를 구하게 되는데, 당연히 Cost 는 작으면 작을수록 좋을 것이다. 각 Layer의 모든 Weights와 Bias는 이 Cost 에 영향을 미치게 된다. 필자는 이를 `"기여한다."`라고 표현한다. Cost Function이 f라면 x라는 변수에 의한 값 f(x)는 최소가 되도록 기여하는 x를 찾는 것이 NN의 학습 메커니즘이다. 이 방법을 [Gradient Descent(경사 하강법)](https://angeloyeo.github.io/2020/08/16/gradient_descent.html) 이라 한다. 문제는 NN은 매우 많은 Weights와 Bias를 가지고 있기 때문에 이 f에 기여하는 파라미터가 너무나 많다는 것이다. 그런데, 각 Layer의 상관 관계를 생각해 보면 항상 앞 Layer의 출력은 뒷 Layer의 입력이 되고, 이는 우리가 고등학교때부터 열심히 배우는 합성함수로 표현 가능하다. 예를 들어, y=f(x), z=g(y) 라는 함수이면, 우리는 z=g(f(x)) 로 표현한다. 이때, z가 cost이고, x가 weight라고 가정해 보자.  

![image](https://user-images.githubusercontent.com/120978778/210169159-bdcd1988-8c41-4deb-807d-af2334666f3c.png)  

그러면 z에 대한 x의 기여도를 구할 때, `반드시` z에 대한 y의 기여도를 구할 수 밖에 없다. 이와 같은 이유 때문에 항상 가장 뒤의 Layer부터 Cost에 대한 gradient를 구하여 이를 앞의 Layer로 전달함으로써 모든 Weights와 Bias의 gradient를 구한 후 update를 진행하게 된다.  
  
이제, 몇가지 Case를 보도록 하자.  

## Case 1. Addition
아까와 마찬가지로, z=g(y), y=f(x) 인데, f를 f(x1,x2) = x1 + x2 라고 가정해 보도록 하자. 두 변수 x1과 x2를 더하는 경우이다. 이때, x1의 z에 대한 기여도는 다음과 같다.  

![image](https://user-images.githubusercontent.com/120978778/210169454-b4db2ff1-dfb4-4572-a5a5-fe163a3aa512.png)  

이미 이 글을 검색하여 들어온 많은 사람들은 알고 있는 사실일 것이지만 파라미터가 몇 개가 되든 덧셈을 하는 경우는 항상 전달 받은 gradient를 그대로 갖게 된다.  

## Case 2. Multiplication
이번에는 f(x1,x2) = x1 * x2 라고 가정해 보자. 이때, x1의 z에 대한 기여도는 다음과 같다.  

![image](https://user-images.githubusercontent.com/120978778/210169804-6a936261-31cf-43a5-9426-aa7b2d2348b6.png)  

이 경우에 x1은 곱해진 모든 변수를 전달 받은 gradient에 곱하는 것으로 자신의 gradient를 갖게 된다.  

## Case 3. Activation
일반적으로 NN은 h = x1 * w1 + x2 * w2 + x3 * w3, z = activation(h) 의 형태를 갖게 된다. 현재 수준에서는 일반적인 DNN을 가정하도록 하자. 이후 같은 논리로 Convolution Layer에 대해서도 해석할 것이다. 이때, `Activation function`은 여러 형태가 될 수 있는데, 기초적인 수준에서 가장 자주 언급되는 `Sigmoid, ReLu, Softmax`의 gradient를 구해보도록 하자.  
  
1). Sigmoid  
Sigmoid 함수는 다음과 같이 정의한다.  

![image](https://user-images.githubusercontent.com/120978778/210169955-6f7c65de-1486-4fa3-a1ce-67a14d4d65bf.png)  

이때, y = sigmoid(x) 라고 가정하면 y에 대한 x의 기여도는 다음과 같이 구해진다.  

![image](https://user-images.githubusercontent.com/120978778/210170113-b065f373-bb48-498c-a949-74ef6e3e7d07.png)  

Sigmoid 함수의 경우 x에 관계없이 항상 y * (1 - y) 를 gradient로 갖기 때문에 gradient 방법을 사용하기가 상당히 편리하다.  

2). ReLu  
ReLu 함수는 다음과 같이 정의한다.  

![image](https://user-images.githubusercontent.com/120978778/210170153-00c421ca-ae90-4edc-bd2b-695bca25f904.png)  

max 함수는 두 변수 중 큰 값을 출력하므로 ReLu는 x가 0보다 작은 경우는 0, 그렇지 않은 경우는 y = x 형태로 출력된다.  

![image](https://user-images.githubusercontent.com/120978778/210170173-e76a930b-70a2-4497-ae46-da01e2566376.png)  

ReLu의 경우 y혹은 x에 따라서 기울기가 달라진다. y에 대하여 정리하면 gradient는 다음과 같다.  

![image](https://user-images.githubusercontent.com/120978778/210170237-132e1a05-57a0-4eee-98b1-14b22141e764.png)  

물론, x로 정의할 수도 있지만 필자는 y로 정리하는 것을 선호한다.  

3). Softmax  
Softmax 함수는 다음과 같이 정의한다.  

![image](https://user-images.githubusercontent.com/120978778/210170264-98076023-def6-48b5-952a-b91309584c2d.png)  

별로 미분하고 싶지 않은 형태이다. 이런 경우에 우리는 고등학교에서 log를 취하라고 배웠다.  

![image](https://user-images.githubusercontent.com/120978778/210170443-d8d14b2c-4175-4a24-adef-e9daa7b04b3e.png)  

이제 양변을 xi 로 미분해 보도록 하자.  

![image](https://user-images.githubusercontent.com/120978778/210170524-df8afaa0-2a86-4ccf-ac6d-fc276f2585fb.png)  

결국 Softmax 함수의 gradient 도 다음과 같이 정리된다.  

![image](https://user-images.githubusercontent.com/120978778/210170591-f109d47a-70cf-4b8b-ac5c-c9329ee85ae2.png)  

4). Example  
![image](https://user-images.githubusercontent.com/120978778/210170892-26d89765-44fe-4b2b-8248-163416cb16d7.png)  

z1 에 대한 w1, w2, w3 의 gradient 를 각각 구해보도록 하자.  
  
# Convolution Layer
Example 까지 풀어 봤다면 이제 기본적인 FC layer 에서의 Back-Propagation 은 이해를 했을 것이다. 그러나, 많은 게시물에서 Convolution Layer 의 경우 직관적인 이해가 어려울 만큼 복잡하게 설명하고 있다. 이전 포스팅인 [Convolution Layer의 Kernel 개수](https://yhkwon6658.github.io/2022-12-28/Convolution-layer%EC%9D%98-Kernel-%EA%B0%9C%EC%88%98) 에서 Convolution Layer 의 Kernel 을 FC Layer 의 Weight 하나와 동일하게 생각하라고 이야기 했다. 이번에도 마찬가지다. Convolution Layer 의 Kernel 하나를 FC Layer 의 Weight 하나와 동일하게 생각하고 다음 내용을 읽어 나가도록 하자.  

![image](https://user-images.githubusercontent.com/120978778/210171592-43dc8d98-f53b-4a28-a27d-e855f71c28e0.png)  

굳이 예시를 복잡하게 들 필요는 없기 때문에 위와 같이 간단한 형태의 구조를 통해 이해해 보도록 하자. 입력 이미지의 크기를 28x28, kernel과의 연산에서 zero-padding 을 적용했다고 가정하면, C 의 경우 28x28, S 의 경우 14x14 가 된다.  
  
1). C  
C의 경우 S와 Average pooling 관계에 있다. 그러나, S를 통해 C의 gradient를 구해야 하는데, un-pooling 외에는 마땅한 방법이 없다. 대다수의 논문에서도 다음과 같이 기술한다.  

![image](https://user-images.githubusercontent.com/120978778/210172015-1bd6ae01-85c9-43fb-8c15-047e076bacb2.png)  

특별한 어려움은 없을 것이라 생각된다.  

2). k  
이제 C를 바탕으로 k를 구해야 한다. k의 notation만 간략하게 언급하면 다음과 같다.  

![image](https://user-images.githubusercontent.com/120978778/210172159-44a35fe7-2c9a-4b7a-8ff7-1f9181cca88f.png)  

윗첨자는 layer, 아래첨자는 순서대로 input node, output node를 나타낸다. 우리의 예시는 input node 가 1장이기 때문에 p는 모두 1이고, output node 가 2장이기 때문에 q는 1, 2이다.  
  
![image](https://user-images.githubusercontent.com/120978778/210172891-c5fd2d09-10e9-407c-b1ad-8917324e8c2e.png)  

사실 파라미터들이 위아래 인덱스가 붙으면서 보기에만 복잡해 보일 뿐이지 sigmoid(xy) 를 x로 미분한 것과 동일한 결과인 것을 알 수 있다. 다만, 여기서 완벽하게 서술하지 않은 것은 2D-Convolution 을 어떻게 정의할 것인지에 따라 표기에 영향을 줄 수 있어서인데, 일반적인 경우에 Convolution을 다음과 같이 정의할 수 있다.  

![image](https://user-images.githubusercontent.com/120978778/210173279-e5992df1-8e1d-4b6a-a8bb-3b9e65e025c6.png)  

이때, k는 -1을 시작 index로 하고, I는 1을 시작 index로 하기로 가정했다.  

![image](https://user-images.githubusercontent.com/120978778/210173390-d65b71a7-282e-46a8-9f57-a29f8a9b45be.png)  

이제, gradient 식은 다음과 같이 정리할 수 있다.  

![image](https://user-images.githubusercontent.com/120978778/210173494-e513e575-584f-483f-bb51-1021bc3aa714.png)  

앞서 우리가 정의한 Convolution 식을 만들기 위해 I를 180도 뒤집을 것이다. x축을 뒤집고, y축을 뒤집도록 하자.  

![image](https://user-images.githubusercontent.com/120978778/210173572-52fc77e2-5a06-4447-812a-e2cf6ee1a45d.png)  

정리하면 우리는 다음 식을 얻을 수 있다.  

![image](https://user-images.githubusercontent.com/120978778/210173644-791e4b9f-4178-4738-ab34-5eae508cee30.png)  

식을 깔끔하게 정리하기 위한 가정이었을 뿐 실제 연산은 곱셈과 덧셈으로 이루어 지므로 이 과정을 굳이 거치지 않아도 특별히 문제는 되지 않는다. 또한, CNN에 한해서는 I(x-u,y-v)가 아닌 I(x+u,y+v) 로 Convolution 을 정의하는 경우가 상당히 많은데, 본문에서 제시한 예제의 Convolution 정의를 I(x+u, y+v) 로 가정하고, k의 gradient 를 구해보는 연습을 하는 것을 추천한다.  
  
또한, 위의 연습을 마무리 한 후에는 [링크](https://zzutk.github.io/docs/reports/2016.10%20-%20Derivation%20of%20Backpropagation%20in%20Convolutional%20Neural%20Network%20(CNN).pdf)  의 글을 꼭 읽어보는 것을 추천한다.  
