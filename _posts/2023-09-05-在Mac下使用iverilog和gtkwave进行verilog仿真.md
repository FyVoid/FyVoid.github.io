---
layout: single
title: "在Mac下使用iverilog和gtkwave进行verilog仿真"
category: BUAA
tags: MacOS, verilog, ComputerOrganization
toc: true
toc_sticky: true
---

> 由于**iverilog**和**gtkwave**均为跨平台开源工具，进行配置后本教程同样适用于windows和linux平台

由于在Mac端无法使用ISE和VCS，我在搜索后找到了iverilog和gtkwave作为Mac端原生编译和运行调试verilog的工具，在此做一个分享（编辑器采用VSCode）。

## 环境配置

首先，你需要**homebrew**来安装这些工具，如果你没有安装homebrew可以参考这个博客。

> https://zhuanlan.zhihu.com/p/146001246

进入terminal后输入以下命令

* 安装iverilog：

```
brew install icarus-verilog
```

* 安装gtkwave：

```
brew install caskroom/cask/gtkwave
```

## 编写和运行verilog代码

确认已经安装好需要的工具后，我们可以开始编写verilog代码了。（此处以教程中时序电路练习_1 编号：1109-400为例）

首先，在vscode中新建一个目录，目录中新建文件counter.v用于编写counter模块 

> vscode中可以自己搜索并安装verilog相关扩展提供代码补全和高亮，我目前使用的是
>
> Verilog-HDL/SystemVerilog/Bluespec SystemVerilog
>
> 不是太好用

counter.v 内容（并非最佳实现）：

```
module counter(
    input[1:0] num,
    input clk,
    output ans
);

    reg[1:0] Status;

    initial begin
        Status = 2'b00;
    end

    always @(posedge clk) begin
        case(Status)
            2'b00: begin
                case(num)
                    2'b01: Status <= 2'b01;
                    2'b10: Status <= 2'b00;
                    2'b11: Status <= 2'b00;
                endcase
            end
            2'b01: begin
                case(num)
                    2'b01: Status <= 2'b01;
                    2'b10: Status <= 2'b10;
                    2'b11: Status <= 2'b00;
                endcase
            end
            2'b10: begin
                case(num)
                    2'b01: Status <= 2'b01;
                    2'b10: Status <= 2'b00;
                    2'b11: Status <= 2'b11;
                endcase
            end
        endcase
    end

    assign ans = Status == 2'b11 ? 1 : 0;

endmodule
```

编写完成后，在目录中新建counter_tb.v 用于编写testbench

counter_tb.v 内容：

```
module counter_tb;

    reg clk;
    reg[31:0] input_nums;
    wire[1:0] num;
    reg[1:0] in_num;
    wire ans;
    reg in_count;

    counter c(num, clk, ans);
    
    initial begin
        input_nums = 32'b01_01_11_10_10_01_10_11_11_10_11_10_01_01_10_01;
        clk = 0;
        in_num = 2'b00;
    end

    /*iverilog */
    initial
    begin            
        $dumpfile("wave.vcd");        //生成的vcd文件名称
        $dumpvars(0, counter_tb);     //tb模块名称
    end
    /*iverilog */
    
    always 
    #5 clk = ~clk;

    always @(posedge clk) begin
        if(input_nums == 32'b0)
            $finish;
        in_num <= input_nums[31:30];
        input_nums = input_nums << 2;
    end

    assign num = in_num;
    
endmodule
```

现在，我们需要在terminal手动进行编译，生成波形文件，并用gtkwave查看生成的波形。

### 编译

在terminal输入：

```
iverilog -o wave counter.v counter_tb.v
```

-o指定生成文件的名称（wave），后面的参数表示编译的源文件，完成编译后，可以看到目录中多出了wave文件

### 生成波形文件

输入：

```
vvp -n wave -lxt2
```

运行后，可以看到目录产生了wave.vcd文件

### 用gtkwave查看波形

gtkwave可以**直接打开软件**，打开波形文件查看。

更推荐的做法是，在terminal输入gtkwave wave.vcd

如图，gtkwave中就可以查看仿真的波形，可以看到和预想的结果一致。

![](/Assets/2023-09-05-在Mac下使用iverilog和gtkwave进行verilog仿真/gtkwave.png)

## 偷懒

*程序员诞生的目的就是为了偷懒。*

既然我们需要多个控制台命令完成这个过程，不如写一个shell脚本来自动化这个过程

在目录新建一个verilog_compile.sh 文件，内容为：

```
echo 'start compiling'
iverilog -o wave $*
# 如果直接运行shell脚本进行编译，注释掉上一行，使用下一行
# iverilog -o wave # 这里填入需要编译的文件
echo 'compiling complete'

echo 'generate wave file'
vvp -n wave -lxt2
# 如果要生成lxt 文件
# cp wave.vcd wave.lxt

echo 'opening wave file'
gtkwave wave.vcd
```

通过在终端将这个shell脚本当作编译命令使用，可以直接完成上一节的所有过程。

![](/Assets/2023-09-05-在Mac下使用iverilog和gtkwave进行verilog仿真/shell_gtkwave.png)

也可以修改shell脚本中相关部分，直接在vscode中运行shell脚本。
