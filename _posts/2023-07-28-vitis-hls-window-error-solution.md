---
layout: post
title: "Vitis HLS ERROR: [IMPL 213-28] 해결법"
author: "Yonghwan Kwon"
tags: "tools"
comments: true
excerpt_separator: <!--more-->
---
Vitis HLS Window Version의 경우 ERROR: [IMPL 213-28]를 Tool 공부를 시작하자마자 보게 될 것이다. <!--more-->

---
# 잘못된 해결방법
[링크](https://support.xilinx.com/s/article/76960?language=en_US)의 방법으로 이 문제를 해결할 수 있다는 댓글이 Xilinx Community에 존재한다. 필자도 수행해 봤으나 해당 에러 코드는 해결되지 않았다. 그래도 버그를 패치한다고 하니 Xilinx 폴더 밑에 압축을 풀어준 후 patch.py를 실행시켜 주도록 하자.    

# 해결방법
해당 에러 코드가 언제 왜 발생하는지를 먼저 생각해 보도록 하자. 아마 Export RTL 버튼을 누른 후 Console 창을 확인해 보니 해당 에러 코드가 발생하여 IP를 Export하지 못하는 상황일 것이다. 다시 Export RTL을 누른 후 Configuration을 누르도록 하자. Version에 x.y.z 형식으로 숫자를 기입하도록 하자. 필자는 0.0.0을 기입하고 있다. 이것만 해주면 문제가 해결된다.  

