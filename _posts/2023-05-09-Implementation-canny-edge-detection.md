---
layout: post
title: "Canny edge detection을 Python으로 구현해보자"
author: "Yonghwan Kwon"
tags: "ISP"
comments: true
excerpt_separator: <!--more-->
---
학교 영상신호처리 과목에서 canny edge detection을 배웠다. non-maximum suppresion이라고 해서 직역하면 극대가 아닌 점을 죽이는 방법이다. 즉, 전처리 과정을 통해서 edge를 극댓값으로 만드는 방법이다. 구글링을 통해 쉽게 참조할만한 [게시글](https://towardsdatascience.com/canny-edge-detection-step-by-step-in-python-computer-vision-b49c3a2d8123)을 찾았는데 코드가 안 돌아간다. 에러를 찾아서 수정한 코드를 다음 [링크](https://github.com/yhkwon6658/canny_edge_detection.git)에 첨부한다. <!--more-->

# Gaussian Lowpass filtering
Edge는 일반적으로 High freq성분이다. Edge를 얻기 위해 High freq성분을 남기고 나머지를 죽이려고 하다보면, 필연적으로 noise가 딸려 나온다. 그래서, Edge detection의 경우 noise boosting effect를 줄이기 위한 Low pass 작업을 하게 된다. Canny edge detection의 경우 2D gaussian을 적용하여 Lowpass filtering을 진행한다.  

# High frequency boosting
이제 High feature를 꺼내기 위해 작업이 필요하다. 학교 수업에서는 Deravative of gaussian을 이용하였다.  

![image](https://user-images.githubusercontent.com/120978778/237044142-98fcbaf4-bbdd-41e8-8ac0-ee4c451d4734.png)  

위 그림에서 볼 수 있는 것처럼 Gaussian을 미분하면 기함수가 된다. 이 기함수 filter를 적용하면 마지막 사진처럼 edge에서 극댓값을 갖게 되는 것을 알 수 있다. 이때, Gaussian은 [Separable filter](https://en.wikipedia.org/wiki/Separable_filter)이기 때문에 computing complexity를 낮추기 위해 x-direction, y-direction을 따로 작업하여 Ximg, Yimg를 뽑아낸다. 그 다음, abs(Ximg) + abs(Yimg)를 통해 `Edge map`을 형성한다. Edge map에서 Non-maximum suppression에 의해 최종 Edge를 결정한다.  

# Non-maximum suppression
arctan(Yimg/Ximg)을 통해 `gradient map`을 형성한다. gradient map은 1, 2, 3, 4, -1, -2, -3, -4 를 0, 45, 90, 135 도로 45도씩 돌면서 quantized graident가 mapping된 형태이다. `Edge map`과 `gradient map`의 동일 픽셀 위치에서 gradient의 수직 방향으로 양옆의 픽셀과 비교하여 극댓값인 지점을 Edge로 하고, 나머지는 죽인다.  

# Implementation in Python
구글링을 하면서 보니 대부분의 코드는 Sobel filter를 이용하였다. zero-crossing을 이용하여 판단하는 대신 일단 high pass filter로 sobel을 적용한 후 `gradient map`을 추출한다. 이때, `gradient map`은 앞서 설명한 것처럼 Quantize된 형태를 출력하지 않고, arctan를 적용하여 실제 각도를 추출한 후 그 각도를 0, 45, 90, 135 도를 기준으로 case를 나누어 비교할 주변 픽셀을 결정한다. 비교할 주변 픽셀의 선택이 끝나면 극댓값인지 아닌지를 판별한다. 사실상 뚜렷한 차이는 high pass filter로 deravative of gradient가 아닌 sobel을 사용한 것이다. 깃허브 코드는 `top.py`를 실행하여 결과를 볼 수 있다.  

# Result
한 가지 실험을 진행하였는데, 다음 코드를 보자.  
```python
def sobel_filters2(img):
    Kx = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]], np.float32)
    Ky = np.array([[1, 2, 1], [0, 0, 0], [-1, -2, -1]], np.float32)
    
    Kx = Kx.reshape(3,3,1)
    Ky = Ky.reshape(3,3,1)
    
    Ix = ndimage.convolve(img, Kx)
    Iy = ndimage.convolve(img, Ky)
    
    G = np.hypot(Ix, Iy)
    G = G / G.max() * 255
    theta = np.arctan2(Ix, Iy)
    
    return (G, theta)
```

마지막 gradient를 구할 때, 기울기의 역수를 취했다. 대부분의 Image는 Symmetric하고, Sobel filter자체도 x, y로 direction만 다를 뿐이므로 형태에 따라서는 arctan(Ix/Iy)가 더 좋은 결과를 낼 수도 있지 않을까 하는 것이었다.  

![image](https://user-images.githubusercontent.com/120978778/237050409-4be160f8-d0dc-4645-82bf-e65787ce930c.png)  

위 실험 결과는 사실 비슷하게 보인다. 그런데, 코드를 실행하고 자세히 보면, arctan(x/y)를 했을 때, 털부분의 edge가 더 잘 사는 것이 보인다. 결과적으로, gradient는 x/y와 y/x 두 방향 모두 실험해 보고 결과가 더 잘 나오는 것을 선택하는 것이 적합해 보인다. `실제로는 gradient map을 뽑고 pixel 별로 comparation을 하는 것은 cost가 너무 크기 때문에 Hysteresis detection을 이용한다.`  

