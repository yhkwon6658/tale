---
layout: post
title: "Convolution layer의 kernel 개수"
autoh: "Yonghwan Kwon"
tags: "CNN"
comments: true
excerpt_separator: <!--more-->
---

CNN 구조에서 사용되는 Convolution layer의 filter 개수에 대해서 알아보고자 한다. 구글링을 하다 보면 잘못된 정보로 전달되고 있는 경우가 많다. 요점부터 말하면 # kernel = # feature map 이라고 하는 글들이 종종 보이는데 잘못된 내용이므로 이를 인지하고 본문 내용을 보도록 하자.
<!--more-->

## Kernel
Kernel은 filter라는 이름으로도 자주 불린다.

![kernel][addr_kernel]
<br/>
이 그림을 보면 kernel의 동작을 쉽게 이해할 수 있다. Convolution layer의 kernel의 역할은 Fully-Connected layer 에서 하나의 weight에 해당하며, 위의 그림은 3x3 kernel이 5x5 image를 stride 1로 shifting 하면서 2D convolution을 진행하는 것을 보여준다. 2D Convolution 동작에 대한 설명은 [링크](https://gruuuuu.github.io/machine-learning/cnn-doc/) 에서 쉽게 다루고 있다. 여기서 강조하고자 하는 것은 Convolution layer에서 kernel 하나는 FC layer에서 weight 하나에 해당한다는 것이다.

## Kernel 의 개수
본론으로 들어가 Kernel의 개수에 대하여 설명하고자 한다. 설명을 위해 사용된 코드는 [링크](https://sdc-james.gitbook.io/onebook/4.-and/5.4.-tensorflow/5.4.2.-cnn-convolutional-neural-network)를 참조하였다. 이 글에서는 코드에 대한 자세한 설명은 생략한다.

### Code
```python
import sys
import tensorflow as tf
import keras
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten
from keras.layers.convolutional import Conv2D, MaxPooling2D
import numpy as np
# data
img_rows = 28
img_cols = 28
(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()
input_shape = (img_rows, img_cols, 1)
x_train = x_train.reshape(x_train.shape[0], img_rows, img_cols, 1)
x_test = x_test.reshape(x_test.shape[0], img_rows, img_cols, 1)
x_train = x_train.astype('float32') / 255.
x_test = x_test.astype('float32') / 255.
print('x_train shape:', x_train.shape)
print(x_train.shape[0], 'train samples')
print(x_test.shape[0], 'test samples')
num_classes = 10
y_train = keras.utils.to_categorical(y_train, num_classes)
y_test = keras.utils.to_categorical(y_test, num_classes)

# model
model = Sequential()
model.add(Conv2D(32, kernel_size=(5, 5), strides=(1, 1), padding='same', activation='relu', input_shape=input_shape))
model.add(MaxPooling2D(pool_size=(2, 2), strides=(2, 2)))
model.add(Conv2D(64, (2, 2), activation='relu', padding='same'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))
model.add(Flatten())
model.add(Dense(1000, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(num_classes, activation='softmax'))
model.summary()
model.compile(loss='categorical_crossentropy', optimizer='adam', metrics=['accuracy'])

# weights
weights = model.get_weights()
print('weight check')
print(f'first convolution layer:\t{weights[0].T.shape}') # 32 x 1 x 5 x 5
print(f'second convolution layer:\t{weights[2].T.shape}') # 64 x 32 x 2 x 2
```
본 코드에서는 첫번째 Conv2D와 두 번째 Conv2D의 wiehgts만을 보고 있다. 이미 주석처리를 해서 알 수 있는 사실이지만 두번째 Conv2D의 weights[2].T.shape는 64 x 32 x 2 x 2 가 되는 것을 알 수 있다. kernel 하나의 size를 2x2로 설정했기 때문에 kernel의 개수는 총 64 x 32 개이다. 즉, convolution layer의 kernel 개수는 # input image x # feature map으로 정해진다. 생각해 보건데 feature map의 개수와 kernel의 개수가 동일하다는 착각은 tensorflow 모델 함수의 파라미터 이름 때문일 것이라고 추측된다. Conv2D 함수의 첫번째 파라미터의 이름은 `filters`이다. 우리는 kernel과 filter를 동일한 용어로 지칭하고 있기 때문에 두번째 Conv2D의 filters가 64라면 kernel의 개수가 64개라는 착각을 충분히 할 수 있다고 생각된다. 아마도 입력 이미지 한 장에 대한 kernel의 개수를 나타내기 위한 표현이었을 것이라 생각된다. <br/>

## 마무리
본 포스팅은 Convolution layer의 kernel 개수에 대한 오개념을 바로잡고자 작성되었습니다. 오류 및 지적은 언제든 환영합니다. 구글링을 해보면 많은 분들께서 back-propagation을 잘 설명하고 있는 것 같습니다. 그러나, 몇몇 포스팅을 보니 CNN에서의 back-propagation은 지나치게 복잡하고 이해하기 어렵게 설명된 경우가 많은 것 같습니다. 다음 게시글은 나름 `CNN back-propagation`을 쉽게 설명할 수 있도록 해보려고 합니다. 감사합니다.





























[addr_kernel]: https://user-images.githubusercontent.com/15958325/58780750-defb7480-8614-11e9-943c-4d44a9d1efc4.gif