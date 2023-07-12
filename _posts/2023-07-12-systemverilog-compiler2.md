---
layout: post
title: "Window 에서 사용 가능한 System Verilog 컴파일러와 시뮬레이터2"
author: "Yonghwan Kwon"
tags: "tools"
comments: true
excerpt_separator: <!--more-->
---
일전에 Window에서 사용 가능한 System Verilog Compiler로 [Vivado를 사용하는 방법을 소개하였다](https://yhkwon6658.github.io/2023-01-29/systemverilog-compiler-and-simulator). 그러나, 계속 사용을 하다보니 Vivado는 새로운 버젼이 나올 때마다 버그가 계속 존재하고 있는 것이 확인되었다. 그래서, modelsim/questasim으로 갈아타게 되었다. 이번 포스트에서는 modelsim에서 CLI를 사용하는 방법을 소개하고자 한다. <!--more-->

---
# 설치
modelsim은 Mentor Graphics가 Simense로 상호를 변경하면서 questasim으로 이름이 바뀌었다. questasim의 경우 이제는 설치하자마자 바로 사용할 수 없고, intel에 계정을 등록하고 license를 받아야 한다. 그런데, 과정이 다소 복잡하고, 분명 intel에서는 메일을 보내줄 것이라고 써있지만 메일이 오지 않는다. 몇 시간 지나서 다시 로그인 해보면 또 계정이 등록되어 있다. 그래도 questasim을 설치하여 사용해 보고 싶은 사람이 있다면, 다음의 [링크](https://fun-teaching-goodkook.blogspot.com/2023/07/questasimmodelsim.html)를 참고하여 설치해보자. modelsim과 questasim은 같은 엔진을 사용하고 있기 때문에 방법은 거의 동일하다. modelsim의 경우 다음의 [링크](https://www.intel.com/content/www/us/en/software-kit/711920/intel-quartus-ii-subscription-edition-design-software-version-13-0sp1-for-windows.html?)에서 Downloads탭의 Individual Files에서 설치할 수 있다. 더 상위 버젼이 존재하지만 필자는 위 링크의 버젼을 쓰고 있어 상위 버젼에서 라이센스 등록이 필요한지 모른다. 아무튼, 이 버젼은 라이센스 등록없이 설치하여 쓸 수 있다.  

---
# test code
### `add4.sv"`
```systemverilog
module add4 (
    input [7:0] din [0:3],
    output [7:0] dout
);

assign dout = din[0] + din[1] + din[2] + din[3];
    
endmodule
```
### `tb.sv`
```systemverilog
`timescale 1ps/1ps

module tb();

logic [7:0] din [0:3];
logic [7:0] dout;

add4 DUT (.*);

initial begin
    for(int i=0; i<4; i++) begin
        for(int j=0; j<4; j++) begin
            din[j] = $random();
        end
        #1;
    end    
    #1;
    $finish();
end

endmodule
```

---
# questasim
questasim이 정상적으로 설치가 되었다면 일단 실행해 보도록 하자. 다음과 같은 화면이 나올 것이다.  

![image](https://github.com/yhkwon6658/f2q/assets/120978778/11235cd6-b6ee-464f-ba10-3dff5925642b)

이제 좌측 상단의 File에서 Change Directory를 누른 후 테스트할 코드가 있는 디렉토리로 이동하자.  

여기서 하단의 Transcript에 다음과 같은 명령어를 순서대로 치도록 하자.

```
vlib work
vlog *.sv
```

![image](https://github.com/yhkwon6658/f2q/assets/120978778/09d9f7ad-ec05-4917-9e4f-fa729e2f9a01)  

그러면 이렇게 Compile 결과를 볼 수 있을 것이다. questa로 바뀌면서 object를 load하는 옵션의 default값이 object를 아무것도 불러오지 않는 것으로 변경되었다. 그래서, CLI로 한 번에 vsim까지 수행할 수 없어졌다. 상단의 Simulate에서 start simulation을 누르도록 하자.  

![image](https://github.com/yhkwon6658/f2q/assets/120978778/253708d2-097c-4fe7-bfa4-db84e73e6428)  

여기서 가장 위의 work가 내가 만든 work directory이다. +버튼을 누른 후 testbench인 tb를 찾도록 하자. tb를 선택한 후 우측 하단의 Optimization Options를 누르도록 하자.  

![image](https://github.com/yhkwon6658/f2q/assets/120978778/fa8b7c2e-fcb1-47b7-bd4a-595bd38a28f0)  

이때, `Design Object Visibility가 No design object visibility로 되어 있을 것이다.` Apply full visibility to all modules로 바꾸도록 하자. OK, OK.  

![image](https://github.com/yhkwon6658/f2q/assets/120978778/3d0a084d-2437-45c6-a1d2-4f722063d6a4)  

이러면 보고 싶은 Objects를 선택하고 Simulation을 수행하면 된다.  

# modelsim
questa는 과정이 조금 번잡스럽다는 것을 알 수 있다. 이번에는 modelsim을 사용해보도록 하자. 필자는 vscode를 사용한다. Window에서 작업할 때 확실히 vscode가 다른 편집기에 비해 편하다.  

![image](https://github.com/yhkwon6658/f2q/assets/120978778/58bdfaf3-12a1-4e0f-bec8-1fffa622c97b)

questasim을 이용해 만든 work 디렉토리와 vsim.wlf 파일이 생성된 것을 확인할 수 있다. 툴 버젼이 달라 뭔가 충돌이 일어날 수 있으니 저것들은 지워 주도록 하자. vscode에서 terminal를 열기 위한 단축키는 `Ctrl + grave key` 이다. 

![image](https://github.com/yhkwon6658/f2q/assets/120978778/96a1f027-d5f4-4aa2-a3de-8074a9670515)

리눅스와 달리 콤마(,)를 찍어줘야 한다.  

어찌되었든 파일은 잘 지워졌다. 이제 다음의 CLI를 순서대로 입력하도록 하자.  

```
vlib work
vlog *.sv
vsim tb
```

![image](https://github.com/yhkwon6658/f2q/assets/120978778/b6bf1223-ce62-4a78-8a4c-4e46c1451940)

modelsim은 default option이 object를 load하도록 되어 있어 커맨드 3개를 연달아 치면 된다. 필자가 사용해본 결과 modelsim이 compile과 simulation 모두 questasim에 비해 빠르다. 그래픽은 questasim이 조금 더 좋다. questasim의 경우 verilog-a나 systemC도 compile이 가능할 것 같은데 아직 확인은 해보지 않았다. verilog-a가 compile이 잘 되면, 이후에 veilog-a를 이용해 analog circuit을 modeling하고 simulation하는 방법을 소개해 보려고 한다.  
