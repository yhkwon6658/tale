---
layout: post
title: "Synthesizable SystemVerilog Design 을 위한 이야기"
author: "Yonghwan Kwon"
tags: "tools"
comments: true
excerpt_separator: <!--more-->
---
SystemVerilog 는 흔히 Verification 을 위한 언어로 취급되어 오고 있지만, [링크](https://sutherland-hdl.com/papers/2013-SNUG-SV_Synthesizable-SystemVerilog_paper.pdf) 에 따르면 합성에서도 여러가지 이점을 가지고 있다. Verilog 는 2005 년을 기점으로 새로운 IEEE 규약이 갱신되고 있지 않다. 즉, 2005 년부터 Verilog 는 SystemVerilog 에 통합되었다고 이해하는 것이 좋다. SystemVerilog 는 Verilog 에 비해 Functional Coding 에 있어 코드의 길이를 크게 줄일 수 있다. 또한, Package 와 Interface 등을 Generate 과 함께 사용하여 설계의 자동화에 큰 기여를 할 수 있다. 이 포스티에서는 설계시 Verilog 에 비해 유리하게 사용될 수 있는 SystemVerilog 의 특징들을 탐구해 보도록 할 것이다. <!--more-->

---
이 포스트는 `Sutherland` 님 뿐만 아니라 [Verilog Pro](https://www.verilogpro.com/) 님의 글 들도 많은 도움이 되었으며, 모르는 문법이 등장하면 [Chip verify](https://www.chipverify.com/systemverilog/systemverilog-tutorial) 에서 기술된 간단한 내용도 참고해 보는 것을 추천드립니다. 사실, SystemVerilog 관련 글들은 대부분 verificaiton 에 초점을 맞추고 있다 보니 Synthesis 를 위한 글들은 찾기가 어려울 것입니다. 필자도 위에 언급된 사이트들외에 많은 검색을 해보면서 이 포스트의 내용을 정리하였습니다. SystemVerilog 를 설계에 도입하길 시도하는 누군가에게 도움이 되길 바랍니다.  

---
# Simulation 에는 bit, Design 에는 logic
SystemVerilog 에 도입된 새로운 변수 중 bit 와 logic 이 있다. bit 는 2-state variable 로 0 과 1만을 갖는다. logic 은 4-state veriable 로 X, Z, 0, 1 을 가질 수 있다. logic 은 기존의 wire 와 reg 의 자리에 모두 사용될 수 있다. 즉, SystemVerilog 에서는 wire 와 reg 어떤 것으로 선언할지 고민하지 않아도 된다. 코드가 복잡해 지고 길어지다 보면 간혹 발생할 수 있는 실수를 예방하는데 큰 도움이 된다. bit 의 경우 0 과 1만을 갖는다는 것은 Simulation 단계에서의 이야기다. Synthesis 를 할 때, Compiler 는 4-state variable 로 인식한다.  

# enum
SystemVerilog 에 도입된 enum 은 꽤나 코드 길이를 줄이는데 도움이 된다. 다음의 코드를 보자.  
```verilog
enum {WAITE, LOAD, DONE} State;
```
위의 코드는 type 과 length 를 지정하지 않았다. 이때, enum 은 기본으로 int(2-state, 32-bits) 로 지정된다. 그러나, 역시 합성 시에는 4-state 로 인식한다는 것을 기억하자.  
`SystemVerilog 에는 integer 가 아닌 int 형식이 추가 되었다.`  
다음과 같이 코드를 작성하는 것이 일반적이다.  
```verilog
enum logic [2:0] {WAITE, LOAD, DONE} State;
```
이때, WAITE, LOAD, DONE 은 순서대로 0, 1, 2 의 값을 갖는다. 물론, 다음과 같이 직접 지정하는 것도 가능하다.  

```verilog
enum logic [2:0] {WAITE = 3'b100, LOAD = 3'b010, DONE = 3'b001} State;
```
사실 이게 무슨 이득이 있겠냐 할 수도 있지만, 두 가지 정도 이점이 있다. 먼저, verilog 로 작업할 경우는 다음과 같이 코드를 짤 것이다.  

```verilog
localparam WAITE = 0;
localparam LOAD = 1;
localparam DONE = 2;

reg [2:0] State;
```
딱 봐도 State 가 많아질 경우 Verilog 스타일을 유지할 때보다 코드가 훨씬 간결해 질 수 있다. 또한, 이후에 다루겠지만 SystemVerilog 는 `User-Defined-Type` 을 지원한다. C 의 typedef 를 지원한다는 뜻이다. enum 과 함께 사용될 때 매우 직관적이고 간결한 코딩 스타일을 만들 수 있으며, Simulation 시에 State 의 값을 특별한 설정을 안 해도 WAITE, LOAD, DONE 으로 바로 볼 수 있어 검증도 편해진다.  

# typedef
user-defined-type(UDT) 를 지원하는 것이 SystemVerilog 의 특징이다. UDT 는 enum, struct 와 함께 사용되어 design 을 한결 간편하게 할 수 있도록 도와준다. 다음의 예시를 보도록 하자.  

```verilog
typedef logic [31:0] bus32_t;
typedef enum logic [0:0] {FALSE, TRUE} bool_t;

module mod1 (
    input bus32_t a, b,
    output bool_t aok
);
// mod1 코드
endmodule
```
typedef 와 enum 을 함께 사용할 경우 simulation 단계에서 aok 의 출력은 FALSE 혹은 TRUE 로 표시되는 것이 하나의 장점이기도 하다. 물론, 현재 코드만 봐서 무엇이 특별한 이득인지 잘 모를 것이다. 앞으로 다룰 struct 와 package 와 함께 사용될 때 장점이 발휘되는 것이므로 참고 보도록 하자.  

# struct
SystemVerilog 는 구조체를 지원한다. struct 는 관련이 있는 signal 들을 묶는데 사용된다. struct 들을 묶는 union 도 지원하지만 필자는 union 을 직접적으로 사용해 본 경험은 없다. struct 는 내부에 struct 를 포함할 수 있다. 몇가지 예제에서 입출력에 struct 를 사용하는 것을 보여주지만 필자의 경우 struct 는 보통 포트 아래에 variables 을 선언할 때 사용한다. 또한, 하나의 .sv 파일 안에 여러개의 모듈을 선언할 때, 다음과 같이 사용될 수 있다.  

```verilog
localparam N = 8; // localparam 은 define 처럼 module 밖에서 선언 가능!

typedef struct {
    logic [N-1:0] a;
    logic [N-1:0] b;
} packet;

module top (
    input clk,
    input rst,
    output [N-1:0] c
);

packet p1;

always @(posedge clk) begin
    if(rst) p1 <= '{default: 0}; // p1 의 모든 변수를 0 으로 초기화
    else    p1 <= '{0, 1}; // p1 의 a 는 0, p1 의 b 는 1
end

core u_core (.*); // 이름이 같은 모든 port 를 연결

endmodule

module core (
    input packet p1,
    output [N-1:0] c
);

assign c = p1.a ^ p1.b;

endmodule
```
사실 위의 코드는 예시를 들기 위해서 찾은 코드를 일부 변형한 것일 뿐 위와 같은 코딩 방식을 취하는 사람은 없을 것이다. 위의 코드는 typdef 로 선언한 구조체를 parameterizable 하게 만드는 방법을 제시한 것이다. 주석처리한 내용들은 이 포스팅에서 직접 다룰 내용들은 아니지만 필자가 제시하는 간단한 tip 정도로 생각해 주길 바란다.  

# package
어떻게 보면 SystemVerilog 에서 지원하는 요소 중 Verilog 와 비교하여 가장 큰 이점을 가져오는 것은 package 이다. 우리는 설계없이 무작정 RTL coding 을 하지 않는다. 복잡한 시스템을 설계할 경우 여러 사람이 각자의 역할을 부여 받게 된다. 이때, 각자 사용할 parameter, port list 등을 약속하지 않으면 나중에 큰 문제가 발생할 수 있다. 때문에 가장 첫번째 단계에서는 보통 이런 것들을 약속하게 된다. 이때, package 는 큰 이점을 가져온다. package 는 다음의 것들을 내부에 묶을 수 있다.  

1). parameter, localparam  
2). const  
3). typedef UDT  
4). automatic task, function  

다만, 여기서 4). 의 경우 필자는 package 에 묶지 않고 보통 interface 에 묶어 package 와 interface 의 역할을 달리 한다. 백문이 불여일견이니 package 의 사용 방법을 보도록 하자.  

`package.sv`
```verilog
package TABLE;
    parameter N = 8;
    parameter L = 32;

    typedef logic [N-1:0] bus_t;
    typedef enum logic [1:0] {IDLE, ADDR, CAL, DONE} state_t;

    typedef struct {
        logic [L-1:0] signal_0;
        logic [L-1:0] signal_1;
    } signal_s;
endpackage
```
`core1.sv`
```verilog
module core1 import TABLE::*; // Wildcard import
(
    input bus_t a, b,
    input signal_s signals,
    // port list 선언
);

state_t PS;

always_ff @(posedge clk) begin : STATE_BLOCK
    if(rst) begin : RESET_CODE
        // reset 내용
    end : RESET_CODE
    else begin
        case(PS) : CASE_STATE
        IDLE : begin : IDLE_STATE
        // 내용
        end : IDLE_STATE
        // 나머지 내용
        endcase : CASE_STATE
    end
end : STATE_BLOCK

// 이하 내용
endmodule : core1
```
요즘은 python 을 안 배우는 사람이 없으니 import 가 어색하지 않을 것이다. package 에서 선언된 것들을 사용하기 위해서는 import 가 필요하다. 위의 코드처럼 `::*` 으로 import 하면 package 내의 모든 요소들을 import 해온다. 또한, 이 코드에도 tip 을 넣어 두었다. SystemVerilog 에서는 `열리고 닫히는` 모든 요소들에 이름을 지정할 수 있다. 아무래도 실제 설계에서는 conditional statement 를 잘 이용하더라도 if ... else 와 case 등이 빈번하게 사용되어 복잡한 코딩을 하게 될 가능성이 높다. 이때, 이름을 계속 지정해 주면 실수를 줄일 수 있다.  

`core2.sv`
```verilog
module core2
import TABLE::signal_s; // import item
(
    input TABLE::bus_t a, b, // explicit reference
    input signal_s signals,
    // 이하 내용
);

TABLE::state_t PS;

// 이하 생략
endmodule : core2
```
wildcard import 를 제외한 나머지 방식들이다. 첫번째는 package 내에서 특정 요소만 import 해오는 방법으로 필자는 `import item` 이라고 부른다. 그 다음 방법은 import 없이 참조하는 방법으로 필자는 `explicit reference` 라고 부른다. 이제, 우리는 package 를 이용하여 모든 약속을 기재할 수 있다. 또한, enum 을 사용하여 무분별한 parameter 선언을 줄일 수 있으며, typedef 를 통해 여러가지 varialbe 들의 이름을 정할 수 있다. 또한, struct 를 사용하여 관련된 여러 signal 들을 묶어서 사용할 수 있다!  

**주의사항**  
우리가 package 에서 import 를 해오기 위해서는 package 가 먼저 compile 되어 있어야 한다. 여러 모듈에서 import 를 할 때는 계층이 꼬일 수 있으니 package 를 pre-compile 하도록 하자.  

# Special procedural blocks
많은 곳에서 소개되고 있는 SystemVerilog 의 procedural blocks 이다. 우리는 보통 Verilog 에서 Sequential block 을 만들 때는 `always @(posedge clk)` 등으로 선언하고 Combinational Logic block 을 만들 때는 `always @(*)` 으로 선언하였을 것이다. 이때, 약속처럼 Sequential block 에서는 Non-Blocking 을 사용했고, Combi block 에서는 Blocking 을 사용했을 것이다. 여기서는 mixed-blocking 에 대한 이야기를 할 것은 아니지만 보통 문제는 Combi block 에서 일어난다. 십중 팔구는 Combi block 에서 의도하지 않은 Latch 가 만들어 지는 것이 문제일 가능성이 높고, Prototyping 을 할 때는 cell library 에 Latch 가 없어서 error 를 보고 Latch 가 생긴 것을 파악할 것이다. 이 문제를 다루기 위해 도입된 것이 Special procedural block 이다.  

1). always_ff @(posedge clk) 은 sequential block  
2). always_comb begin 은 combi block, 더이상 @(*) 같은 것은 쓰지 않아도 된다.  
3). always_latch @(a, b) 은 latch block 인데, 필자는 의도적으로 latch 를 만들어 본 적이 없다.  

어찌되었든 always_comb 에서 latch 가 만들어 지면 바로 error 메세지가 나온다. 마찬가지로 always_latch 에서 combi block 이 만들어 지면 바로 error 메세지가 나온다. 사실 이것 외에도 SystemVerilog 는 Verilog 에서 bug 로 인식하고 compile 되던 많은 것들을 error 메세지로 출력한다고 하는데 하나하나 따져 본 것이 아니라 그냥 그렇구나 정도로 이해했다.  

# case...inside
case inside 가 왜 좋은지를 이해하려면 기존 Verilog 체제의 case, casex, casez 를 이해하는 것이 필요하다. 문제는 이것이 거의 유일한 Verilog 의 상당히 난이도 높은 주제가 되는 것이라 애초에 casex, casez 에 대한 논의를 하는 사람이 많지 않다는 것이다. 우선 X-propagation 이라는 것을 먼저 이해할 필요가 있는데, 우리가 설계를 하다보면 특정 조건이 만족되지 않는 경우 회로를 타고 시스템 전체에 X 값이 계속 전달되는 상황을 의미한다. 이때, casex 를 쓰면 회로 자체가 정상적으로 동작하지 않을 수 있다. 조금 더 쉽게 말하면 casex 내의 case item 들이 중복될 수 있다는 뜻이기도 하다. 결국, 이에 대한 해법으로 제시된 것은 casex 를 synthesizable coding 시 사용하지 않는 것이다. 깔끔한 해법이다. casez 의 사용과 위험성만을 이해하면 된다. casez 의 경우 Z 가 입력되면 don't care 로 처리한다. 다음 코드를 보도록 하자.  

```verilog
module test2 (
    input [2:0] sel,
    output reg a, b, c
);

always @(sel) begin
    {a, b, c} = 3'b000;
    casez(sel)
    3'b1?? : a = 1'b1;
    3'b?1? : b = 1'b1;
    3'b??1 : c = 1'b1;
    default : {a,b,c} = 3'b000;
    endcase
end

endmodule
```
casez 의 경우 동시 조건이 만족되면 가장 위의 코드가 우선 순위를 갖게 된다. 그래서 위의 코드는 priority 를 주기 위한 방법으로 제시된 코드이다. 문제는 sel 에 3'b00Z 가 입력된 상황이다. `이때, casez 는 symmetric 하게 Z 를 don't care 로 처리한다고 한다.` 이게 참 헷갈리는 문제라 어려울 수 있는데 Z는 0 과 1을 모두 포함하고 있는 상태로 casez 의 item 이 3'??0 이든 3'b??1 이든 둘 다 만족한다는 것이다. 코드와 결과를 보는 게 정신건강에 이로울 것이다!    

```verilog
`timescale 1ps/1ps
`include "test2.v"

module tb();
reg [2:0] sel = 3'b000;
wire a,b,c;

test2 u_test2 (sel,a,b,c);

initial begin
    $monitor("%0t: %b", $time, c);
    #10;
    sel = 3'b00Z;
    #10;
    $finish;
end

endmodule
```  
```
Time resolution is 1 ps
run -all
0: 0
10: 1
$finish called at time : 20 ps : File "C:/PROJECT/_RTL/Test/test2_tb.v" Line 15
exit
```
**이러면 이제 큰일 났다고 봐야한다.** 만약, 어딘가에 connection 이 안 되어 high z(open circuit) 상태가 되면 이 시스템은 붕괴된다. 이 경우 적어도 문제가 생겼다는 것을 인지할 수 있어야 하는데 이러면 case 를 이용해 완벽하게 모든 조건을 기술하든가 혹은 priority 를 if ... else 로 주는 것이 적합한 선택이 될 것이다. 뭐가 됐든 fancy 한 코딩은 포기해야 한다는 것!  

***이제 case ... inside 를 사용해보자!***  
```verilog
module test2 (
    input [2:0] sel,
    output logic a, b, c
);

always_comb begin
    {a, b, c} = 3'b000;
    case(sel) inside
    3'b1?? : a = 1'b1;
    3'b?1? : b = 1'b1;
    3'b??1 : c = 1'b1;
    default : {a,b,c} = 3'b000;
    endcase
end

endmodule
```
위와 같이 코드를 수정하여 다시 컴파일한 결과는 다음과 같다.  
```
Time resolution is 1 ps
run -all
0: 0
$finish called at time : 20 ps : File "C:/PROJECT/_RTL/Test/test2_tb.sv" Line 15
exit
```
참고자료에서는 case ... inside 는 이를 asymmetric masking 을 하기 때문이라고 하는데, 이번 예제를 통해서 symmetric masking 이 무슨 말인지 알았을 것이다.  

**~~case...inside 의 위용은 아직 끝이 아니다!~~**  
casez 의 위험성을 커버하면서 원래의 역할을 대체한다는 정말 좋은 기능을 가졌지만 사실 한 가지 더 굉장한 기능을 가지고 있다. case 가 오지게 많은 경우의 처리에 능통하다는 것인데, 다음 예시를 한 번 보도록 하자.  

```verilog
module test3(
    input [4:0] sel_state,
    output logic [4:0] next_state
);

always_comb begin
    case(sel_state) inside
    [0 : 15] : next_state = sel_state + 1;
    [16 : 31] : next_state = 0;
    default : next_state = 'x; // 모든 비트 x로 처리
    endcase
end

endmodule
```
다음은 테스트벤치 코드이다.  

```verilog
`timescale 1ps/1ps
`include "test3.sv"

module tb();
logic [4:0] sel_state;
logic [4:0] next_state;

test3 u_test3 (.*);

initial begin
    $monitor("%0t: %b", $time, next_state);
    for(int i=0; i<32; i++) begin 
    /* Simulation 할 때만 이렇게 하자! i++ 은 합성 안 된다. 
    엄밀히 말하면 Design Compiler(DC) 에서 안 된다고 한다. 
    Vivado 는 이것도 처리하긴 한다. */
        #10;
        sel_state = i;
    end
    #10;
    $display("END = %0t", $time);
    $finish();
end

endmodule
```
컴파일 결과는 다음과 같다.  
```
Time resolution is 1 ps
run -all
0: xxxxx
10: 00001
20: 00010
30: 00011
40: 00100
50: 00101
60: 00110
70: 00111
80: 01000
90: 01001
100: 01010
110: 01011
120: 01100
130: 01101
140: 01110
150: 01111
160: 10000
170: 00000
END = 330
$finish called at time : 330 ps : File "C:/PROJECT/_RTL/Test/test3_tb.sv" Line 18
exit
```
i 가 16 이 되는 순간부터 next_state 는 0 을 출력하게 되었다. 특히, parameter 를 사용하는 경우 매우 유용하게 사용될 수 있으며, 다음과 같은 형태도 가능하다.  

```verilog
module test3(
    input [4:0] sel_state,
    output logic [4:0] next_state
);

always_comb begin
    case(sel_state) inside
    5'b0???? : next_state = sel_state + 1;
    5'b1???? : next_state = 0;
    default : next_state = 'x; // 모든 비트 x 로 처리
    endcase
end

endmodule
```
두가지를 마음대로 섞어서 사용해도 된다. 그렇다 할지라도 우리가 머리로 생각한 것이 synthesis/simulation 에 모두 반영될 것이다. 사실, 앞에서 언급한 package 와 함께 사용되는 것들 전부 사용 안 해도 상관없다. 손에 안 맞으면 안 하면 그만이다. 하지만, case ... inside 는 꼭 알아가서 casez 대신 사용했으면 한다. 필자도 case 처리에 대한 부분이 설계시 SystemVerilog 를 사용하는 것으로 마음을 돌리게 된 결정적인 이유이기도 하다!  

# priority 와 unique
앞서 다룬 case ... inside 만큼 어려운 내용은 이제 없으니 안심하고 보자. 우리가 RTL 코딩을 할 때, 가장 주의해야 하는 것은 always_comb 와 case 를 사용할 때이다. 다음과 같은 문제들이 발생할 수 있다는 것을 인지하자.  

1). 원치 않는 Latch 의 발생  
2). case 의 item 이 동시 다발적으로 hit  
3). case 를 만족하는 조건이 없는데, default 가 없는 경우  

위의 세가지 중 2). 의 경우 의도하여 사용할 수도 있지만 그렇지 않은 경우 실수가 될 것이며, 1). 과 3). 은 명백한 코딩 실수이다. 이를 방지하기 위해 추가된 것이 priority 와 unique 이다.  

`priority` : 3). 에 대한 경고  
`unique` : 2). 와 3). 에 대한 경고  

이는 우리가 코딩을 할 때, 2). 와 3). 이 발생하지 않는다고 `의도` 하는 경우에 사용해야 한다. 만약, 경고 메세지가 나온다면 실수가 있는 것이니 확인해야 한다. 만약, 이러한 실수를 인지하지 못한다면 synthesis 단계에서 이를 해결하기 위한 최적화가 일어나고 그 결과 원치 않는 이상한 회로가 튀어 나온다. 일반적으로, if ... else 문을 사용할 때 2). 는 거의 발생하지 않는다. 애초에 설계없이 구현을 하는 경우는 없기 때문에 hierarchy 가 있는 경우 대부분의 엔지니어는 if ... else 를 사용하기 때문이다. 대부분의 문제는 case 에서 발생하므로 실수를 방지하기 위해서 case 앞에 priority 를 붙이는 것을 습관화 하도록 하자. 만약, case 가 동시 다발적으로 hit 되어 첫번째 줄이 수행되도록 의도하는 것이 아니라면 unique 를 case 앞에 붙이는 것을 습관화 하도록 하자. 이 두 가지는 3 개의 문제 중 1). latch 의 발생은 해결하지 못한다. `latch 발생을 피하는 가장 쉬운 방법은 조건문 이전에 출력을 초기화하는 것이다.` 이것들을 습관화하도록 하자.  

**혹시 모를 실수를 방지하기 위해서라도 SystemVerilog 를 사용하는 것이 상당히 유용하다는 것을 알 수 있다. 우리는 아무리 코딩에 자신이 있더라도 사소한 실수가 재앙을 불러올 수 있다는 것을 항상 마음에 새겨야 한다.**  

# task 를 버리고 void function 을 사용하자
사실 Verilog 만을 사용해 온 사람들은 task 나 function 둘 다 RTL 코딩에 사용하지 않았을 것이다. 필자의 경우 가끔 반복적으로 사용되는 조합회로를 instantiation 을 여러번 할 경우 코드가 길어지는 게 보기 싫어 task 를 이용하긴 했었다. Verilog 에서 task 는 return 변수가 없으나 포트에 output 과 inout 을 지정할 수 있어 마치 C 에서 포인터를 이용하여 출력을 받는 것처럼 사용할 수 있었다. function 의 경우 포트는 모두 input 이며 출력 function 자체의 출력 값은 하나였다. 일반적으로 설계시에 task 나 function 은 조합회로를 짜기 위해서 사용되는데 function 의 출력이 하나뿐이라면 모든 엔지니어가 task 를 사용할 것이다. 그런데, synthesis 과정에서 task 는 종종 말썽을 일으킨다고 한다. 솔직히, 자주 쓴 적도 없고 Vivado 를 이용한 합성에서는 특별히 문제가 된 적도 없어서 필자는 잘 모르지만 아무튼 자주 문제가 발생한다고 한다. 일단, Background 는 이 정도고 여기서 언급할 것은 void function 이 가능하다는 것이다. 즉, return 값이 없다. 대신에 SystemVerilog 에서는 function 도 output 이나 inout 으로 포트를 설정할 수 있게 되었다. 즉, 기존에 task 의 역할을 그대로 전수받는 것이다. 이게 합성시에 문제가 안 된다고 하니 void function 을 이용하도록 하자. 다음 예시를 통해서 사용법을 보자.  

`test4.sv`
```verilog
function void f_add (
    input [31:0] a,
    output [31:0] b, c
);
automatic int i = 1; // SystemVerilog 는 default 가 Static 이다. (C 와 반대)
const automatic int j = 1; // const 이지만 warning 보기 싫으면 그냥 automatic 하자.

i = i + 1;
// const 로 선언하였기 때문에 j = j + 1; 은 불가능하다.

b = a + i;
c = a + j;
    
endfunction

module test4 (
    input [31:0] a,
    output logic [31:0] b, c // always_comb 에서 b 와 c 가 지정되므로 logic(reg) 로 선언되어야 한다.
);

always_comb f_add(a,b,c);

endmodule
```
`test4_tb.sv`
```verilog
`timescale 1ns/1ns
`include "test4.sv"

module tb();
logic [31:0] a, b, c;

test4 u_test4(.*);

initial begin
    $monitor("%0t\tb = %0d\tc = %0d", $time, b, c);
    a = 0;
    #10;
    a = 1;
    #10;
    a = 2;
    #10;
    $finish();
end

endmodule
```
`컴파일 결과`
```
Time resolution is 1 ns
run -all
0       b = 2   c = 1
10      b = 3   c = 2
20      b = 4   c = 3
$finish called at time : 30 ns : File "C:/PROJECT/_RTL/Test/test4_tb.sv" Line 17
exit
```
아쉬운 점은 function void automatic 혹은 function automatic void 로 선언은 불가능하다. 그래서 function 에서 사용되는 변수가 있는 경우 function 은 void 로 선언해 주고 변수 형식 앞에 automatic 을 직접 해줘야 한다. Verilog 만 사용한 경우 function 자체를 사용해 본 적이 없는 사람이 많을 것 같기 때문에 automatic 을 지우면 결과가 어떻게 달라지는지 보도록 하자.  

```verilog
function void f_add (
    input [31:0] a,
    output [31:0] b, c
);
int i = 1; // SystemVerilog 는 default 가 Static 이다. (C 와 반대)
const automatic int j = 1; // const 이지만 warning 보기 싫으면 그냥 automatic 하자.

i = i + 1;
// const 로 선언하였기 때문에 j = j + 1; 은 불가능하다.

b = a + i;
c = a + j;
    
endfunction

module test4 (
    input [31:0] a,
    output logic [31:0] b, c // always_comb 에서 b 와 c 가 지정되므로 logic(reg) 로 선언되어야 한다.
);

always_comb f_add(a,b,c);

endmodule
```
int i 앞에 automatic 만 지웠다.  

`컴파일 결과`
```
Time resolution is 1 ns
run -all
0       b = 2   c = 1
10      b = 4   c = 2
20      b = 6   c = 3
$finish called at time : 30 ns : File "C:/PROJECT/_RTL/Test/test4_tb.sv" Line 17
exit
```
SystemVerilog 에서는 C 와 반대로 default 가 static 이기 때문에 f_add 가 호출될 때마다 i 가 1로 초기화 되지 않고 1씩 계속 증가한 것을 알 수 있다. 여기서 흥미로운 것은 always_comb 라는 조합회로에 f_add 를 썼기 때문에 필자는 무한 increment 같은 일이 벌어질 줄 알았지만 xvlog 는 외부에서 호출될 때마다 하나씩 증가하도록 컴파일 하였다. 다만, 이게 DC 에서도 똑같이 컴파일 될 거라는 보장은 없다. 어차피 조합회로를 모델링 하는 것이 목적이라면 애초에 static 으로 선언할 이유도 없긴 하다. `항상 함수는 void 로 내부의 변수는 automatic 으로 선언하도록 하자.`  

# Interface
사실 RTL coding 에서는 Package 와 Interface 는 큰 차이를 보이지 않는다. 다만, Package 는 보통 Shared parameter 나 port list 등을 정의하기 위해 사용한다. Interface 도 비슷한 역할을 할 수 있지만 Package 와 역할을 분리하여 function 들을 선언하기 위해 사용하고 있다. 앞서 사용한 function 예제를 사용하려고 한다. interface 에 사용되는 function 은 automatic 이어야 한다. 그러나, void function 을 선언할 경우 function 을 automatic 으로 선언할 수 없었다. 따라서 function 내부의 변수가 사용될 경우 automatic 으로 선언해 주어야 한다는 것을 잊지 않도록 하자.  

`package.sv`
```verilog
package param_pk;
    parameter N = 32;
endpackage
```
`interface.sv`
```verilog
interface test_if;

import param_pk::N;

function void f_add (
    input [N-1:0] a,
    output [N-1:0] b, c
);
automatic int i = 1; // SystemVerilog 는 default 가 Static 이다. (C 와 반대)
const automatic int j = 1; // const 이지만 warning 보기 싫으면 그냥 automatic 하자.

i = i + 1;
// const 로 선언하였기 때문에 j = j + 1; 은 불가능하다.

b = a + i;
c = a + j;
    
endfunction
    
endinterface //test_if
```
`test5.sv`
```verilog
module test5 import param_pk::N;
(
    input [N-1:0] a,
    output logic [N-1:0] b,c
);
    
test_if itf();

always_comb begin
    itf.f_add(a,b,c);
end

endmodule
```
`test5_tb.sv`
```verilog
`timescale 1ps/1ps
`include "package.sv" // first compile
`include "interface.sv" // second compile
`include "test5.sv" // third compile

module tb();

import param_pk::N;

logic [N-1:0] a,b,c;

test5 u_test5(.*);

initial begin
    $monitor("%0t\tb = %0d\tc = %0d",$time, b,c);
    a = 0;
    #10;
    a = 1;
    #10;
    a = 2;
    #10;
    $finish();
end

endmodule
```
`컴파일 결과`
```
Time resolution is 1 ps
run -all
0       b = 2   c = 1
10      b = 3   c = 2
20      b = 4   c = 3
$finish called at time : 30 ps : File "C:/PROJECT/_RTL/Test/test5_tb.sv" Line 22
exit
```
다음은 PYNQ-Z2 FPGA 위에 합성한 결과이다. 이때, package 의 N = 2 로 설정했다.  

![image](https://user-images.githubusercontent.com/120978778/216857583-140e6b76-b8e9-4a19-b57c-c4ee6d6a72ef.png)  

먼저, Hierarchy 만을 보면 test5 만 보인다.  

![image](https://user-images.githubusercontent.com/120978778/216857843-e2f4cab5-344d-465a-a8f6-fdddaab9efe0.png)  

package 와 interface 는 module 이 아니기 때문에 Libraries 에 할당된다. 이 점을 주의하도록 하자.  

![image](https://user-images.githubusercontent.com/120978778/216858175-771adbc7-f665-4534-a2a7-5cbe84173459.png)  

Implementation 후 본 schematic 이다. FPGA 위에서 동작 검증까지 정상적으로 마무리하였다.  

# generate
사실, generate 은 이미 Verilog-2005 에서도 충분히 활용될 수 있었다. 설계의 자동화를 목표로 할 때, 효과적으로 사용될 수 있다. SystemVerilog 로 다음과 같은 코드를 구현했다고 생각해보자.  

`test6.sv`
```verilog
module test6 (
    input [2:0] a [0:2],
    input [2:0] b [0:2],
    output logic [2:0] c
);

logic [2:0] w [0:2];

assign w[0] = a[0] & b[0];
assign w[1] = a[1] & b[1];
assign w[2] = a[2] & b[2];
    
integer I;
always @(*) begin
    c = w[0];
    for(I=1; I<3; I=I+1) begin
        c = c ^ w[I];
    end
end

endmodule
```
`test7.sv`
```verilog
`timescale 1ps/1ps
`include "test6.sv"

module tb();

logic [2:0] a [0:2];
logic [2:0] b [0:2];
logic [2:0] c;

test6 u_test6 (.*);

initial begin
    a[0] = 3'b001; a[1] = 3'b010; a[2] = 3'b100;
    b[0] = 3'b000; b[1] = 3'b111; b[2] = 3'b100;
    #10;
    $display("%b", c);
    #10;
    $finish;
end

endmodule
```
`컴파일 결과`
```
Time resolution is 1 ps
run -all
110
$finish called at time : 20 ps : File "C:/PROJECT/_RTL/Test/test7.sv" Line 18
exit
```
이제 어떻게 동작 해야 하는지 알았으니 test6.sv 의 코드를 다음과 같이 바꿔 보도록 하자.  

```verilog
module test6 (
    input [2:0] a [0:2],
    input [2:0] b [0:2],
    output logic [2:0] c
);

logic [2:0] w [0:2];

genvar i;
generate
    for(i=0; i<3; i = i + 1) begin
        assign w[i] = a[i] & b[i];
    end
endgenerate
    
int I;
always_comb begin
    c = w[0];
    for(I=1; I<3; I=I+1) begin
        c = c ^ w[I];
    end
end

endmodule
```
결과는 동일하다. generate for loop 내부의 내용을 다음처럼 바꿔보자.  

```verilog
module test6 (
    input [2:0] a [0:2],
    input [2:0] b [0:2],
    output logic [2:0] c
);

logic [2:0] w [0:2];

genvar i;
generate
    for(i=0; i<3; i = i + 1) begin
        always_comb w[i] = a[i] & b[i];
    end
endgenerate

// 생략
```
이래도 결과는 동일하다. 이제 instantiation 도 해보자.  

`and_3bit.sv`
```verilog
module and_3bit (
    input [2:0] a, b,
    output [2:0] c
);

assign c = a & b;

endmodule
```
`test6.sv`
```verilog
module test6 (
    input [2:0] a [0:2],
    input [2:0] b [0:2],
    output logic [2:0] c
);

logic [2:0] w [0:2];

genvar i;
generate
    for(i=0; i<3; i = i + 1) begin
        and_3bit u_and_3bit (a[i], b[i], w[i]);
    end
endgenerate

// 생략
```
마찬가지로 결과는 동일하다.  

![image](https://user-images.githubusercontent.com/120978778/216907837-78d79c6e-d92d-4f3c-b9ee-3f613fdacb3e.png)  

컴파일러가 해석한 블록의 네이밍은 위와 같다. 굳이, 3번의 예시를 모두 보여준 것은 generate for loop 와 alwyas_comb for loop 의 차이를 이해하길 바라기 때문이다.  

# 이 글에서 다루지 않은 것들
각종 system task, assertion 등의 잔잔바리 tip 들은 아직 정리하고 검증해 보지 않았다. 이후에 이 포스트에 추가될 수도 있고, 새로운 게시글로 업로드 될 수도 있다.

