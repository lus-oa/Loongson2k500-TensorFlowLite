# Loongson2k500-TensorFlowLite
移植TensorFow Lite到Loongson-2k500开发板

## 一、TFLM简介
TFLM是TensorFlow Lite for Microcontrollers项目的简称，全称翻译过来就是“适用于微控制器的TensorFlow Lite”。它是一个来自谷歌的边缘AI框架，在单片机上也能够运行。  
来自官方的介绍：  
> TensorFlow Lite for Microcontrollers 是 TensorFlow Lite 的一个实验性移植版本，它适用于微控制器和其他一些仅有数千字节内存的设备。 它可以直接在“裸机”上运行，不需要操作系统支持、任何标准 C/C++ 库和动态内存分配。核心运行时(core runtime)在 Cortex M3 上运行时仅需 16KB，加上足以用来运行语音关键字检测模型的操作，也只需 22KB 的空间。


## 二、TFLM使用方法
