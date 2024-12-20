# EmbeddedNote
1. 🌱 研究方向：个人兴趣方向:嵌入式系统开发；研究生研究方向:混沌系统和图像加密算法等。
2. 📫 联系方式：邮箱：delelemonwzx@163.com
3. 🌱 当前正在学习：C/C++、MATLAB、STM32/GD32/AT32/APM32/CW32,嵌入式Linux系统等。
4. 💬 电子信息工程专业学生，希望成为嵌入式工程师,嵌入式软件工程师,电子工程师。
![](pic/open.jpg)
## 教育背景

**西安邮电大学 -电子工程学院-电子信息(硕士)  2022.09 - 2025.06**

研究生研究方向：混沌系统设计与图像加密算法设计，发表SCI论文2篇，参与中国研究生数学建模竞赛获全国三等奖.

**Design and analysis of image encryption based on memristor chaotic systems with hidden attractors**

SCI论文 Physica Scripta 2024 [查看论文](https://iopscience.iop.org/article/10.1088/1402-4896/ad56cf)

**Generation multi-scroll chaotic attractors using composite sine function and its application in image encryption**

SCI论文 Physica Scripta 2024 [查看论文](https://iopscience.iop.org/article/10.1088/1402-4896/ad2b3f)

**蚌埠学院 - 电子与电气工程学院-电子信息工程(本科)2018.09 - 2022.06**

本科期间作为电子协会和电子竞赛负责人，参与各项电子类竞赛，包括电子设计大赛、单片机、物联网等，获得过奖项若干。

## 专业技能

1. 熟悉C/Matlab/Python开发，熟悉基本数据结构，熟练使用主流嵌入式开发工具，具有基本软硬设件调试能力；

2. 熟悉嵌入式MCU芯片开发(STM32、GD32、CW32、AT32、瑞萨RA系列等)，熟悉MCU基本外设使用；

3. 具有RTOS(FreeRTOS、OneOS、RT-Thread)应用开发经验；了解嵌入式Linux系统应用与驱动开发；

4. 熟悉Verilog和FPGA开发软件使用，具有Altera FPGA、国产紫光同创FPGA开发经验；

## 奖项证书
2022年第十九届中国研究生数学建模竞赛三等奖

2020年安徽省机器人大赛单片机与嵌入式系统竞赛一等奖 

2020年安徽省高校物联网应用创新大赛技能赛 二等奖

2020年第十五届全国大学生智能车竞赛创意组 三等奖

## 项目经历

### 基于极海APM32的多功能环境监测仪装置设计
#### APM32F407 FreeRTOS 2024 

![](pic/gd.jpg)

**Embedded-APM32-Board-Template 极海半导体APM32**

**https://gitee.com/End-ING/embedded-apm32-board**

> 本项目设计了基于极海半导体APM32F4系列微控制器的多功能环境检测装置，通过对多种环境传感器数据的采集，实现环境参数实时监测，使用显示屏幕模块和按键模块进行用户信息展示与交互。同时连接物联网云平台，将传感器数据通发送到云端，实现远程监测。项目移植实时操作系统FreeRTOS实现多任务管理。移植FreeModbus协议，实现Modbus通讯功能。
#### 主要职责：
1. 负责APM32单片机的软件开发工作，项目分层设计，以芯片固件库层(APM32固件库)，模块驱动层（环境传感器、屏幕驱动等）、芯片外设驱动层（GPIO、UART、I2C、SPI、ADC、Timer、RTC等）、中间件层(组件层FreeRTOS、Multi-Button、EasyLogger、LVGL、FreeModbus等)、应用控制层。
2. 主要负责编写传感器驱动代码(温湿度AHT10、光强BH1750、气压等传感器BMP280、气体传感器AGS10等)，实现环境参数的读取与数据处理，移植控制模块代码(LED驱动、按键驱动、Flash储存驱动、RTC驱动等)。
3. 移植LCD模块(SPI接口)代码，移植开源图形库LVGL进行系统界面UI设计(显示传感器数据，系统状态信息等)；
4. 通过无线模组实现连接云端 ，将环境传感器数据发送到物联网平台，修改APP软件，使用户在APP上实现对各种参数监测进行远程监测与控制.
5. 移植FreeRTOS实时操作系统，通过FreeRTOS的任务调度实现(传感器数据采集任务，数据显示任务等)多任务管理；
6. 移植FreeModbus开源协议，编写串口上位机测试程序，实现串口上位机与Modbus通讯基本功能(测试基本功能码)，同时进行软件与硬件联合调试，编写技术文档笔记。

### 基于CW32的无刷电机驱动控制系统
#### CW32 BLDC 2023

![](pic/cw32-bldc.jpg)

**Embedded-CW32-Board-Template 武汉芯源CW32** 

**https://gitee.com/End-ING/embedded-cw32-board-template**

> 本项目设计一个基于CW32的无刷电机控制系统，其核心控制单元采用CW32F030单片机。该系统实现了方波有感控制和方波无感控制两种模式，以满足不同应用场景下的需求。通过ADC接口实时读取电流和电压值，通过电位器调整电机速度，通过温度传感器获取温度值，并通过OLED显示屏展示系统状态、参数等信息。
##### 主要职责：

1. 负责武汉芯源CW32无刷电机控制开发板设计与调试。(无刷电机控制板包括：MCU主控电路、OLED屏幕接口电路、电位器电路、指示灯电路、按键电路、下载接口电路、电源输入电路（输入电压12~36V，输出电流3A）、电源稳压电路(输入36V转15V转5V，5V转3.3V)、三相逆变电路、栅极驱动电路、电压电流检测电路、反电动势检测电路（单电阻采样）、电机接口、霍尔接口等）;
2. 负责武汉芯源CW32单片机的软件开发工作，编写GPIO、ADC、I2C、Timer(ATIM、GTIM、BTIM) 等外设驱动代码。通过ADC接口读取电压值，编写传感器数据采集代码，移植OLED显示屏的驱动程序，实现OLED显示系统的转速，电压等状态信息。同时对控制系统进行调试，编写测试程序与技术文档。
3. 编写有感方波控制代码，使用ATIM定时器产生PWM波驱动上桥电路，下桥采用电平控制，使用GTIM定时器的输入捕获功能获取霍尔传感器状态，通过判断霍尔传感器的组合状态完成转子位置检测，实现电机的六步换相。
4. 编写无感方波控制代码，使用反电势法检测方法，通过过零信号的组合状态判断转子的位置，从而实现电机换相控制。

### 基于紫光同创FPGA的混沌信号的实现
#### FPGA MATLAB Verilog 2024
![](pic/fpga.jpg)

> 项目简述：项目来源于研究生科研项目，研究方向为混沌信号的硬件实现与图像加密应用。通过将连续混沌系统微分方程进行数值迭代运算，利用MATLAB进行仿真分析绘制系统运动轨迹。同时，使用Verilog编写混沌系统离散化代码，在Modelsim软件上进行仿真验证。最后，使用DAC转换模块和国产紫光同创PGL50H-FPGA开发板，将系统产生的数字信号转换为模拟信号，并在示波器上显示系统的运动轨迹。
##### 主要职责：
1. 使用数值方法求解微分方程(Euler、Runge-Kutta)，利用MATLAB进行仿真分析绘制系统运动轨迹。
2. 使用模拟电路元器件，完成混沌电路系统的模拟电路设计验证。利用国厂紫光同创FPGA开发板进行硬件数字电路验证，编写Verilog程序(Euler、Runge-Kutta算法，状态机、浮点数转换定点数，查找表)，使用DAC模块将数字信号转换成模拟信号,完成混沌信号的生成。

### 无接触安防控制系统
#### STM32 K210 IoT OTA

#### B站 视频https://www.bilibili.com/video/BV1gt4y1Y78u/

> 项目采用STM32单片机作为主控制器，负责整体系统的控制与传感器数据获取与处理，嘉楠科技的RISC-V架构处理器K210作为副处理器,负责图像采集并对图像数据进行处理,实现人脸识别与检测。系统通过WIFI模组与机智云物联网平台对接,实现传感器数据的上传与控制命令的下发,利用云平台进行远程OTA升级。修改机智云APP软件,使用户在APP上实现对各种参数监测,支持RFID刷卡与语音播报功能,从而实现无接触安防控制系统功能。
##### 主要职责：
1. 负责STM32单片机的软件开发工作,编写传感器驱动代码与应用层控制代码,实现传感器数据的读取与处理,编写通信代码,完成与副处理器K210通信。同时进行软件与硬件调试等,编写测试程序与技术文档。
2. 实现WIFI无线模组与机智云物联网平台连接,实现数据上传与下发。利用机智云物联网云平台OTA功能,实现系统远程固件升级。同时修改APP源码,完成APP上位机设计,并与微控制器进行联调测试.
## 自我评价
1. 有良好的自学能力和解决问题的能力，能承受一定的工作压力；
2. 具备良好的团队合作能力，能够与团队紧密协作，共同完成项目目标；
3. 能够快速接受新知识和新技术。

## 求职意向

嵌入式工程师 嵌入式软件工程师 电子工程师

## Blog
Blog: https://openblog-coderend.top/

Gitee:https://gitee.com/End-ING

CSDN: https://delehub.blog.csdn.net/



