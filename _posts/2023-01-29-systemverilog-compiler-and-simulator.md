---
layout: post
title: "Window 에서 사용 가능한 System Verilog 컴파일러와 시뮬레이터"
author: "Yonghwan Kwon"
tags: "tools"
comments: true
excerpt_separator: <!--more-->
---
이제껏 HDL 로는 verilog 만을 사용해 오다가 HDL 과 HVL 기능을 모두 가지고 있는 sytem verilog 로 넘어 오게 되었다. 그런데, system verilog 의 경우 compile 과 simulation 을 진행할 수 있는 tool 에 대한 소개가 별로 없는 것 같아 포스팅을 작성한다.  
<!--more-->

# 소개
먼저, 여기서 말하는 complie 은 cell, power, timing, technology 같은 constraint 이 전혀없는 elaboration 을 위한 것이라고 보면 좋을 것 같다. 여러 삽질을 한 결과 verilog 와 icarus 의 포지션으로 사용할 수 있는 tool 은 vivado 밖에 없다고 결론 내렸다. 몇몇 이유가 있는데 일단 vivado 는 대부분의 작업자들의 local에 깔려 있다. 그리고, modelsim 이 system verilog 에서 몇가지 제약이 있다고 한다. 또한, icarus 는 현재까지는 system veirlog 를 사용하는 것이 거의 불가능하다. [링크](https://blog.naver.com/doksg/221979883906) 에 `verilator` 라는 툴을 이용해 system verilog 를 컴파일 할 수 있다고는 하는데 그 다음 튜토리얼 글을 읽어보니 배보다 배꼽이 더 큰 것처럼 보인다. 아무튼 vivado 로 compile 과 simulation 을 하기 위해 다음 과정을 따르도록 하자.

---
# 시작
1. Vivado 설치  
[링크](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive.html) 로 이동하여 Web installer 로 설치하도록 하자. Vivado 는 2022.2 까지 나왔지만 각종 버그 등의 문제도 있고, 최신 버전에서 툴에 몇가지 변화가 있었는데 그게 마음에 안 들어서 필자는 2020.2 버전을 사용하고 있다.  
<br/>
2. 환경변수  
윈도우 버튼 - 시스템 환경 변수 편집 - 고급 - 환경 변수  
사용자 변수의 Path 편집 - 새로 만들기 - C:\Xilinx\Vivado\2020.2\bin - 확인  
`만약, 기본 경로가 아니라면 <설치경로>\Vivado\2020.2\bin 을 넣으면 된다.`  
<br/>  
3. 터미널 명령어 활용하기  
vivado 는 gui 를 켜서 컴파일과 시뮬레이션을 하기에는 너무 많은 시간이 걸린다. 최대한 작업을 덜 귀찮게(?) 하기 위해 터미널 명령어를 잘 활용하자. 필자는 text editor 로 `vscode` 를 사용하고 있다.

### `mem.sv`
```verilog
module mem #(
    parameter 
    width = 0,
    addr_bits = 0,
    height = 0
) (
    input i_clk,
    input i_en,
    input [width-1:0] i_data,
    input [addr_bits-1:0] i_addr,
    output logic [width-1:0] o_data
);
// varialbes
logic [width-1:0] mem [0:height-1];

// initialization
int i;
initial begin
    for(i=0; i<height; i=i+1) begin
        mem[i] = 0;
    end
end

// design
always_ff @( posedge i_clk ) begin : memory
    if(i_en) begin
        mem[i_addr] <= i_data;
    end
    else begin
        o_data <= mem[i_addr];
    end
end
    
endmodule
```
### `mem_tb.sv`
```verilog
`timescale 1ns/1ns
`include "mem.sv"

`define width 32
`define addr_bits 8
`define height 256

module tb();
logic i_clk;
logic i_en;
logic [`width-1:0] i_data;
logic [`addr_bits-1:0] i_addr;
wire [`width-1:0] o_data; // net out must be wire

// instantiaion
mem #(`width, `addr_bits, `height) 
u_mem (i_clk, i_en, i_data, i_addr, o_data);

// initialization
initial begin
    i_clk = 0;
    i_en = 0;
    i_data = 0;
    i_addr = 0;
end

// clock generation
always #5 i_clk = ~i_clk; // 100MHz

// testbench
initial begin
    $display("time\ti_en\ti_data\ti_addr\to_data");
    $monitor("%0t\t%0b\t%0d\t%0d\t%0d",$time, i_en, i_data, i_addr, o_data);
    i_en = 1;
    i_data = 128;
    i_addr = 3;
    #10;
    i_en = 0;
    #50;
    $finish;
end

endmodule
```
테스트를 위해서 만든 코드다. 터미널에 모니터만 하는 용도로 만들어졌다. testbench 를 작성하다 보면 나의 경우 주로 waveform 을 보기 보다는 hdl 로 module 을 구현하기 전에 modeling 한 algorithm 의 출력과 작성한 module 의 output 을 비교하여 error 가 있는지만을 모니터한다. systemverilog 를 선택한 건 첫번째로 random signal 을 입력으로 줄 수 있다는 점, 두번째는 내가 굳이 modeling 할 때, 포인터를 사용하지만 않으면 C/C++ 로 modeling 하던 것과 동일하게 systemverilog 로 짤 수 있어 여러모로 작업의 단계를 줄일 수 있다고 생각했기 때문이다. 어쨌든 다음 두 가지 케이스에 대한 명령어만 알면 된다.  

### 커맨드 창에 모니터만 하는 경우
Ctrl + ` 를 눌러 terminal 을 키자. Window Powershell 이나 cmd 창을 열면 될 것이다. 필자는 Widnow Powershell 이 기본으로 되어 있어서 그냥 Powershell 에다가 입력한다.  
```
xvlog -sv mem_tb.sv
xelab -R tb
```
순서대로 입력하면 된다. mem_tb.sv 에서 이미 mem_sv 를 `include 했기 때문에 xvlog -sv mem_tb.sv mem_v 를 할 필요가 없다. xelab 에서는 top module 의 이름을 지정하면 된다.  

### Waveform 보는 경우
```
xvlog -sv mem_tb.sv
xelab --debug wave tb
xsim -g tb
```
이렇게 입력하면 다음과 같은 화면이 뜰 것이다.  

![image](https://user-images.githubusercontent.com/120978778/215289481-d5252df7-7932-48b6-9451-966fd0541a7e.png)  

좌측 상단의 File - Simulation Waveform - New Configuration 을 누르자.  
![image](https://user-images.githubusercontent.com/120978778/215290055-db9a368b-560d-44b9-a85c-7e9eb1e0273c.png)  

그 다음 Objects 의 모든 신호를 Add to Wave Window 해주도록 하자.  
![image](https://user-images.githubusercontent.com/120978778/215290146-c3a29b56-f638-4aa1-b3a8-50019547cf5e.png)  

이제 F3 을 눌러 Run 시키도록 하자.  
![image](https://user-images.githubusercontent.com/120978778/215290184-12f963b0-1c88-43ef-a045-80001c76001e.png)  

최종 결과물  

### Clean
하나를 깜박했다. 터미널에서 명령어를 치면 뭔가 이상한 파일들이 많이 만들어 진다. 뭐 원래 compile 을 하면 그 부산물들이 만들어지기 마련인데, xvlog, xelab, xsim 을 하면서 뭐가 정말 많이 만들어진다. 필자의 경우 다음과 같은 것들이 만들어 졌다.  

![image](https://user-images.githubusercontent.com/120978778/215290423-91f81cc0-5304-45cc-943f-25a12787a4c3.png)  

뭔가 굉장히 지저분한데, 리눅스를 쓸 때는 아예 clean 파일을 만들어서 밀어 버리지만.. Powershell 을 쓸 때는 clean 파일을 어떻게 만드는지도 모르겠다. 그래서 명령어를 쳐야 하는데 다음과 같이 입력하도록 하자.  
```
rm *xsim*, *webtalk*, *wdb*, *xelab*, *xvlog*, *Xil*
```
리눅스와 다르게 중간에 콤마(,) 를 한 번씩 찍어줘야 한다. 결과물은 다음과 같다.  

![image](https://user-images.githubusercontent.com/120978778/215290530-961e2afc-e738-4ef4-b29e-5b4817fccf47.png)  
