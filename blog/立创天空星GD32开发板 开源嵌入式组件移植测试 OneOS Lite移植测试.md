## 立创天空星GD32开发板 开源嵌入式组件移植测试 OneOS Lite移植测试

#### 移植目标硬件（开发板/芯片/模组）

​		OneOS Lite支持ARM Cortex M核芯片和RISC-V内核的芯片的移植，比如STM32 基于Cortex M核全系列、GD32 基于Cortex M核全列、NXP 基于Cortex M核全系列等。本教程将使用立创开发板GD32F407进行示例移植，其他 ARM Cortex M系列开发板和芯片移植方法类似。调试ARM Cortex M核还需要仿真器，如果您的开发板或者芯片模组没有板载仿真器，就需要连接外置的仿真器，如DAPLink。

### OneOS Lite

​		OneOS Lite是中国移动针对物联网领域推出的轻量级操作系统，具有可裁剪、跨平台、低功耗、高安全等特点，支持ARM Cortex-A和 Cortex-M、MIPS、RISC-V等主流芯片架构，兼容POSIX、CMSIS等标准接口，支持Javascript、MicroPython等高级语言开发模式，提供图形化开发工具，能够有效提升开发效率、降低开发成本，帮助用户快速开发稳定可靠、安全易用的物联网应用。

OneOS Lite总体架构采用分层设计，主体由驱动、内核、组件、安全框架组成。采用一个轻量级内核加多个系统组件的模式，加上海量硬件的适配支持，使OneOS Lite 具备极高的可伸缩性与易用性。

![](pic/oneos-01.jpg)

#### OneOS系统特点

灵活裁剪：抢占式的实时多任务RTOS内核，支持多任务处理、软件定时器、信号量、互斥锁、消息队列、邮箱和实时调度等特性，RAM和ROM资源占用极小。可灵活裁剪，搭配丰富组件，适应不同客户需求。

跨芯片平台：应用程序可无缝移植，大幅提高软件复用率。支持的主流芯片架构有：ARM Cortex-A和Cortex-M、MIPS、RISC-V等。支持几乎所有的MCU和主流的NB-IOT、4G、WIFI、蓝牙通信芯片。

组件丰富：提供丰富的组件功能，如互联互通、端云融合、远程升级、室内外定位、低功耗控制等。同时提供开放的第三方组件管理工具，支持添加各类第三方组件，以便扩展系统功能。

低功耗设计：支持MCU和外围设备的功耗管理，用户可以根据业务场景选择相应低功耗方案，系统会自动采用相应功耗控制策略，进行休眠和调频调压，有效降低设备整体功耗。

安全设计：针对物联网设备资源受限、海量连接、网络异构等特点，在系统安全、通信安全、数据安全等方面提供多维度安全防护能力。

#### OneOS开发资料

1. OneOS源码开源地址

   https://gitee.com/cmcc-oneos/OneOS

   ![](pic/oneos-03.jpg)

2. 开发文档官网

   https://os.iot.10086.cn/v2/doc/homePage

   ![](pic/oneos-00.jpg)

3. 开发工具源码下载

   https://os.iot.10086.cn/download/

   ![](pic/oneos-02.jpg)

4. 开发工具OneOS_Cube

   https://os.iot.10086.cn/download/tool

   ![](pic/oneos-04.jpg)

### OneOS-GD32开发板测试

1. 下载源码

   ```bash
   git clone https://gitee.com/cmcc-oneos/OneOS.git
   ```

   ![](pic/oneos-05.jpg)

   ![](pic/oneos-06.jpg)

   ##### 内核启动简单分析

   ![](pic/oneos.jpg)

   内核启动主要有下面几个步骤：

   1. 系统先从启动文件开始运行，然后进入 OneOS Lite的内核启动函数 os_kernel_init()和os_kernel_start()；
   2. os_kernel_init中调用k_run_init_call函数执行OS_INIT_LEVEL_PRE_KERNEL_1，进行内核启动前的第一阶段的初始化；
   3. 初始化内核各模块，如tick队列，调度器，定时器等
   4. 创建recycle，idle，timer，sys系统任务；
   5. 启动调度器，最后会运行sys任务，sys任务调用k_run_init_call函数依次执行其他自动化宏注册的函数，最后调用main函数进入用户程序入口。

2. 安装OneOS_Cube工具

   OneOS-Cube工具地址 https://os.iot.10086.cn/download/tool

   下载工具请点击[OneOS-Cube下载](https://os.iot.10086.cn/download/tool)

   直接点击下载的OneOS-Cube工具进行安装，安装路径不支持含有中文和空格字符。

   OneOS-Cube工具使用方法：进入到代码工程目录，任意空白处点击右键，再找到“OneOS_Cube”执行，即可打开OneOS-Cube的命令行操作界面。

   ![](pic/oneos-07.jpg)

   这样OneOS-Cube成功启动了

   ![](pic/oneos-08.jpg)

   OneOS-Cube常用的用户指令非常简单，OneOS 3.0及以上版本的常用指令简单介绍如下：

   | 命令            | 说明                                                         |
   | --------------- | ------------------------------------------------------------ |
   | oos project     | 创建工程命令，需在project目录下使用，提供菜单交互环境创建工程 |
   | oos config      | 系统配置命令，需在芯片工程目录下使用，提供菜单交互环境对系统功能宏进行控制，宏配置结果自动保存到oneos_config.h文件中 |
   | oos build       | 代码编译命令，需在芯片工程目录下使用，将根据编译配置文件的描述进行代码编译 |
   | oos clean       | 清理工程命令，对前期编译生成的中间文件和二进制结果文件进行删除 |
   | oos init -i XXX | 生成IDE工程命令，其中"XXX"代表目标IDE环境，具体支持的IDE参考oos help init命令说明。例如生成Keil工程：oos init -i keil |
   | oos pack        | 打开OneOS组件仓库，包含调式、网络、应用等组件，按需下载获取；支持OneOS Lite 3.2.0及以上版本，请使用CUBE 2.1.0版本 |
   | oos             | 查看oos命令选项说明                                          |

3. 工程模板创建

   1. 创建工程

      在OS源码根目录中，打开project文件夹。启动CUBE工具，输入创建工程命令  **oos project**

      ![](pic/oneos-09.jpg)
      
      ![](pic/oneos-10.jpg)
      
      默认是STM32系列芯片，因此需要选择GD32芯片品牌系列型号等信息，**按S键进行配置保存，按ESC键退出并开始生成工程代码。**
      
      **在这里我们选择的立创天空星开发板对应的芯片，GD32F407VET6作为案例进行测试。**
      
      ![](pic/oneos-11.jpg)
      
      ![](pic/oneos-12.jpg)
      
      ![](pic/oneos-13.jpg)
      
      ![](pic/oneos-14.jpg)
      
      ![](pic/oneos-15.jpg)
      
      保存.config配置文件，ESC退出开始生成代码。
      
      ![](pic/oneos-16.jpg)
      
      ![](pic/oneos-17.jpg)
      
      ![](pic/oneos-18.jpg)

4. 生成代码

   ![](pic/oneos-19.jpg)

5. 编译代码

   1. 使用cube工具编译代码

      需要切换到GD32F407VE的工程目录中，输入oos build命令，编译代码

      ![](pic/oneos-20.jpg)

      ![](pic/oneos-21.jpg)

      ![](pic/oneos-22.jpg)

   2. 使用MDK工具编译代码

      输入oos init -i keil命令，选择Project工程,进行编译代码

      ![](pic/oneos-23.jpg)

      ![](pic/oneos-24.jpg)

      ![](pic/oneos-25.jpg)

6. 修改代码

   由于默认工程生成的代码与实际我们用到的板子是不一样的，因此我们需要进行修改，目前我们只修改立创天空星开发板板载的LED模块。将board.c代码中默认的LED的IO端口PA8修改成PB2.

   ![](pic/oneos-26.jpg)

   ![](pic/oneos-27.jpg)

   在主函数添加部分串口打印代码,在操作系统内部一般使用os_kprintf进行打印功能，这个函数C库中printf功能一样。

   ```c
   #include <board.h>
   #include "stdio.h"
   static void user_task(void *parameter)
   {
       int i = 0;
   
       for (i = 0; i < led_table_size; i++)
       {
           os_pin_mode(led_table[i].pin, PIN_MODE_OUTPUT);
       }
   
       while (1)
       {
           for (i = 0; i < led_table_size; i++)
           {
               os_pin_write(led_table[i].pin, led_table[i].active_level);
   					  os_kprintf("gd32-lckfb bsp_led on \r\n");
               os_task_msleep(500);
   
               os_pin_write(led_table[i].pin, !led_table[i].active_level);
   					  os_kprintf("gd32-lckfb bsp_led off \r\n");
               os_task_msleep(500);
           }
       }
   }
   
   int main(void)
   {
       os_task_id task;
   	  os_kprintf("gd32f407 lckfb-board hardware_init [ok] \r\n");
   		os_kprintf("gd32f407 lckfb-board 2024-07-28 [ok]\r\n");
   		os_kprintf("gd32f407 lckfb-board oneos-lite testing [ok]\r\n");
       task = os_task_create(OS_NULL, OS_NULL, 256, "user", user_task, OS_NULL, 3);
       OS_ASSERT(task);
       os_task_startup(task);
   
       return 0;
   }
   ```

7. 下载代码

   理论上完成上面的操作就可以直接下载代码了，但是我在实际操作过程中发现下载程序没有任何反应，查找资料发现修改部分配置文件才可以。参考文章 https://blog.csdn.net/weixin_46158019/article/details/137949236

   输入oos config命令，进入配置工具，选择Drivers里面boot，配置部分东西。

   ![](pic/oneos-28.jpg)

   ![](pic/oneos-30.jpg)

   ![](pic/oneos-29.jpg)

   修改boot里面的起始地址与size大小，这个东西和MDK-KEIL的配置差不多。

   ![](pic/oneos-35.jpg)

   ![](pic/oneos-31.jpg)

   ![](pic/oneos-32.jpg)

   **修改前：**

   ![](pic/oneos-33.jpg)

   **修改后：**

   ![](pic/oneos-34.jpg)

   退出ESC，保存即可，重新编译即可。

   ![](pic/oneos-36.jpg)

   第一种下载方法使用MDK-KEIL下载，简单方便，我这里使用的DAPlink进行下载。

   ![](pic/oneos-38.jpg)

   **下载完成效果，打印正常，shell正常，led正常闪烁。**

   ![](pic/oneos-37.jpg)

   第二种下载方法使用PyocD进行下载，使用的工具还是DAPlink。

   进入工程目录下面的out文件夹里面，输入两个命令进行烧录代码

   ```bash
   pyocd erase -c -t gd32f407ve --config pyocd.yaml
   pyocd load oneos.bin -t gd32f407ve --config pyocd.yaml
   ```

   ![](pic/oneos-39.jpg)

   ![](pic/oneos-40.jpg)

   ![](pic/oneos-41.jpg)

   ![](pic/oneos-42.jpg)

   **下载完成效果**

   ![](pic/oneos-43.jpg)

   ### 总结

   ​	完成上面的移植之后，接下来就可以开始学习OneOS-lite的系统内核与设备驱动和物联网组件相关部分了，特别是设备驱动部分，这是精华部分，绝大多数的RTOS内核其实的大差不差，例如FreeRTOS，UCOS等，但是OneOS-lite提供了设备驱动层的东西，大量的利用面向对象的思想，这是值得学习。

   ![](pic/oneos-44.jpg)

   ​	OneOS Lite 提供了丰富的基础驱动支持，驱动部分通过对底层代码和厂家 SDK 的统一封装，以 device 设备的方式对外提供统一的接口和较为全面的协议支持，更便于客户上手开发。目前 OneOS Lite 已经支持了包括 STM32、NXP、MM32、HC32、GD32 等多个厂家及芯片平台，并且针对其不同系列做了深度适配，覆盖了常用的 GPIO、SPI、QSPI、ADC、DAC、CAN、I2C、ETH、SAI 等接口，同时配套了相关的协议和应用示例，如各类传感器、屏幕显示、网络链接、音频等，后续仍在不断丰富和完善，力求为客户提供一个便捷高效的开发环境。

   OneOS Lite 在各类外设的基础上抽象出了设备驱动模型，有效提高了代码可复用性、可移植性，模块分层解耦，降低了各层的开发难度。设备驱动框架设计遵循以下原则：

   1. 下层向上层提供统一的访问接口

   2. 分层设计，屏蔽硬件差异

   3. 每一层只跟其相邻的层建立联系

   ##### 各层的作用如下：

   1. 系统调用接口提供统一的对外接口，实现对设备的访问。
   2. 设备管理层实现了对设备驱动程序的封装，向上对接系统调用接口。
   3. 设备驱动框架层是对同类硬件设备驱动的抽象，将不同厂家的同类硬件设备驱动中相同的部分抽取出来，将不同部分留出接口，由驱动程序实现。
   4. 设备驱动层是一组驱使硬件设备工作的程序，实现访问硬件设备的功能，它负责创建和注册设备。

   

