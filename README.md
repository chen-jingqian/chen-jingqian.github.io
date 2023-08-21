
# **lvgl on linux Frame buffer**

> 主机环境: Ubuntu 18.04  
> lvgl 8.0.1

## **获取源码并运行demo**

1. 获取源码

    需要3样东西来构建demo : 
    1. `lvgl核心控件库`: 从此处获取[源码 lvgl](https://github.com/lvgl/lvgl/releases)，选择版本 [Release v8.0.1](https://github.com/lvgl/lvgl/releases/tag/v8.0.1).
    1. `显示和触摸接口层` : 关联图形库与硬件，[源码 lv_drivers](https://github.com/lvgl/lv_drivers)，选择版本 v8.0.
    1. `lv_demo 源码` : [源码 lv_demos](https://github.com/lvgl/lv_demos), 同样选择版本 v8.0.

    最终得到三个压缩包: 

    - `lvgl-8.0.1.tar.gz`
    - `lv_drivers-release-v8.0.zip`
    - `lv_demos-release-v8.0.zip`

1. 创建文件夹并解压源码:
    ```sh
    mkdir lvgl_linux
    cd lvgl_linux
    # 复制上述压缩包到此处 .

    # 解压
    tar xvf lvgl-8.0.1.tar.gz
    unzip lv_drivers-release-v8.0.zip
    unzip lv_demos-release-v8.0.zip

    # 重命名解压后的文件夹
    mv lvgl-8.0.1 lvgl
    mv lv_drivers-release-v8.0 lv_drivers
    mv lv_demos-release-v8.0 lv_demos

    # delete zip tar
    rm lv_drivers-release-v8.0.zip lv_demos-release-v8.0.zip lvgl-8.0.1.tar.gz

    ls
    lv_demos  lv_drivers  lvgl
    ```

1. 获取配置文件

    - 核心库配置文件 : `lv_conf.h`
    - 驱动配置文件 : `lv_drv_conf.h`
    - demo配置文件 : `lv_demo_conf.h`

    ```sh
    # 基于配置模板创建
    cp lvgl/lv_conf_template.h lv_conf.h
    cp lv_drivers/lv_drv_conf_template.h lv_drv_conf.h
    cp lv_demos/lv_demo_conf_template.h lv_demo_conf.h

    ls
    lv_conf.h  lv_demo_conf.h  lv_demos  lv_drivers  lv_drv_conf.h  lvgl
    ```

    修改配置文件: 

    - 使能，将3个文件 `#if 0` 改为 `#if 1` `/*Set it to "1" to enable content*/`
    - 修改 `lv_drv_conf.h`：

        ```c
        #  define USE_FBDEV           1
        #  define USE_EVDEV           1
        ```
    - 修改 `lv_demo_conf.h` :

        ```c
        #define LV_USE_DEMO_WIDGETS        1
        ```

    - 修改 `lv_conf.h` :

        配置 `Tick interface` ：
        ```c
        #define LV_TICK_CUSTOM     1
        #if LV_TICK_CUSTOM
        #define LV_TICK_CUSTOM_INCLUDE  <stdint.h>         /*Header for the system time function*/
        extern uint32_t custom_tick_get(void);
        #define LV_TICK_CUSTOM_SYS_TIME_EXPR (custom_tick_get())     /*Expression evaluating to current system time in ms*/
        #endif   /*LV_TICK_CUSTOM*/
        ```

        使能 12，16 号字体: 

        ```c
        #define LV_FONT_MONTSERRAT_12    1
        #define LV_FONT_MONTSERRAT_14    1
        #define LV_FONT_MONTSERRAT_16    1
        ```
        
        设置堆大小:

        - 使用 lvgl 内建内存分配方案:
        配置堆大小，实测 2KB 无法启动demo，我给 2MB

            ```c
            #  define LV_MEM_SIZE    (2 * 1024U * 1024U)          /*[bytes]*/
            ```

        - 使用 linux 系统内存分配：使能宏 `#define LV_MEM_CUSTOM      1` 即可


1. 编写主程序和Makefile

    `main.c` 参考自 [lv_port_linux_frame_buffer/blob/release/v8.2/main.c](https://github.com/lvgl/lv_port_linux_frame_buffer/blob/release/v8.2/main.c)

    main.c 中修改 `lv_demo.h` 路径:  

    ```c
    #include "lv_demos/lv_demo.h"
    ```

    `Makefile` 参考自 [lv_port_linux_frame_buffer/blob/release/v8.2/Makefile](https://github.com/lvgl/lv_port_linux_frame_buffer/blob/release/v8.2/Makefile)

    Makefile 做如下修改: 

    ```makefile
    CC = arm-linux-gnueabihf-gcc
    
    include $(LVGL_DIR)/lv_demos/lv_demo.mk

    # CSRCS +=$(LVGL_DIR)/mouse_cursor_icon.c
    ```

1. `make` 生成 `./demo`
