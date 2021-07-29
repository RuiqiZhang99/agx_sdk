# Agilex Robot Platform SDK

![GitHub Workflow Status](https://github.com/westonrobot/wrp_sdk/workflows/Cpp/badge.svg)
![GitHub Workflow Status](https://github.com/westonrobot/wrp_sdk/workflows/ROS/badge.svg)

Copyright (c) 2020 [AgilexRobot](http://www.agilex.ai/)

## Introduction

Supported platforms/所支持的平台

* **Scout**: skid-steer mobile base
* **Hunter**: ackermann mobile base
* **Bunker**: caterpillar mobile base
* **Tracer**: skid-steer mobile base
* **Scout-mini**: skid-steer mobile base

这个软件包提供了一个C++接口，用于与AGILX机器人的移动平台通信，用于向机器人发送命令并获取最新的机器人状态。SDK可以在x86和ARM平台上工作。

通常，您只需要实例化robot基类的一个对象（如ScoutBase），然后使用该对象以编程方式控制robot。在内部，基类管理两个后台线程，一个用于处理机器人状态的CAN/UART消息并相应地更新机器人状态数据结构中的状态变量，另一个用于维护50Hz循环并向机器人库发送最新命令。用户可以在主线程中迭代执行任务，并检查robot状态或设置控制命令。

有关示例，请参阅“src/apps”。

## Hardware Interface

### Setup CAN-To-USB adapter 
 
1. Enable gs_usb kernel module
    ```
    $ sudo modprobe gs_usb
    ```
2. Bringup can device
   ```
   $ sudo ip link set can0 up type can bitrate 500000
   ```
3. If no error occured during the previous steps, you should be able to see the can device now by using command
   ```
   $ ifconfig -a
   ```
4. Install and use can-utils to test the hardware/用 can-utils 测试实物硬件
    ```
    $ sudo apt install can-utils
    ```
5. Testing command/测试指令
    ```
    # receiving data from can0
    $ candump can0
    # send data to can0
    $ cansend can0 001#1122334455667788
    ```

“/scripts”文件夹中提供了两个脚本，便于设置。您可以运行“/setup\u can2usb.bash”进行首次安装，并在每次拔下并重新插入适配器时运行“/bringup\u can2usb.bash”启动设备。

### Setup UART

通常情况下，您的UART2USB应自动识别为“/dev/ttyUSB0”或类似类型，并可随时使用。如果您得到错误“... permission denied ...”尝试打开端口时，您需要将端口访问权限授予您的用户帐户：

```
$ sudo usermod -a -G dialout $USER
```
您需要重新登录以使更改生效。

## Build SDK

### Install dependent libraries

```
$ sudo apt install build-essential cmake
$ sudo apt install libasio-dev
```

### I. Use the package with ROS

```
$ cd <your-catkin-ws>/src
$ git clone https://github.com/agilexrobotics/agx_sdk.git
$ cd ..
$ catkin_make
```

### II. Use the package without ROS
You will need to upgrade CMake to a newer version in this case. Follow instructions on this page: https://apt.kitware.com/ 

Here is a brief summary

```
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates gnupg software-properties-common wget
$ wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | sudo tee /etc/apt/trusted.gpg.d/kitware.gpg >/dev/null
```

Ubuntu 20.04 

```
$ sudo apt-add-repository 'deb https://apt.kitware.com/ubuntu/ focal main'
```

Ubuntu 18.04 

```
$ sudo apt-add-repository 'deb https://apt.kitware.com/ubuntu/ bionic main'
```

Ubuntu 16.04 

```
$ sudo apt-add-repository 'deb https://apt.kitware.com/ubuntu/ xenial main'
```

```
$ sudo apt-get update
$ sudo apt-get install kitware-archive-keyring
$ sudo rm /etc/apt/trusted.gpg.d/kitware.gpg
$ sudo apt-get install cmake
```

If you want to build the TUI monitor tool, additionally install libncurses

```
$ sudo apt install libncurses5-dev
```

Configure and build

```
$ cd agx_sdk 
$ mkdir build
$ cd build
$ cmake ..
$ make
```

## Run Demo Apps
### Run Scout Demo

The demo code expects one parameter for the CAN bus mode.

```
$ ./app_scout_demo can0
```

Both the port name and baud rate need to be provided when using the RS232 interface.

```
$./app_scout_demo /dev/ttyUSB0 115200
```

If you've installed "libncurses5-dev" and built "app_scout_monitor", you can run it in a similar way:

```
$ ./app_scout_monitor can0
```

or

```
$./app_scout_monitor /dev/ttyUSB0 115200
```
Both the port name and baud rate need to be provided when using the RS232 interface.
### Run Hunter Demo
The demo code expects one parameter for the CAN bus mode.

```
$ ./app_hunter_demo can0
```

Note: the monitor app is not built by default if you use this SDK with ROS.

## Known Limitations

<!-- 
1. The CAN interface requires the hardware to appear as a CAN device in the system. You can use the command "ifconfig" to check the interface status. For example, you may see something like

    ```
    can1: flags=193<UP,RUNNING,NOARP>  mtu 16
            unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 10  (UNSPEC)
            RX packets 4751634  bytes 38013072 (36.2 MiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 126269  bytes 1010152 (986.4 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
            device interrupt 43
    ```
    
    If you use your own CAN-to-USB adapter, make sure it supports slcan or can be brought up as a native CAN device. Some adapters may use a custom-defined protocol and appear as a serial device in Linux. In such a case, you will have to translate the byte stream between CAN and UART by yourself. It would be difficult for us to provide support for them since not all manufacturers define this protocol in the same way.

1. Release v0.1 of this SDK provided a serial interface to talk with the robot. Front/rear light on the robot cannot be controlled and only a small subset of all robot states can be acquired through that interface. Full support of the serial interface is still under development and requires additional work on both the SDK and firmware sides.
-->

## Reference

* [CAN command reference in Linux](https://wiki.rdu.im/_pages/Notes/Embedded-System/Linux/can-bus-in-linux.html)
