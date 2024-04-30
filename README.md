# Loongson2k500-TensorFlowLite
移植TensorFow Lite到Loongson-2k500开发板

## 一、TFLM简介
TFLM是TensorFlow Lite for Microcontrollers项目的简称，全称翻译过来就是“适用于微控制器的TensorFlow Lite”。它是一个来自谷歌的边缘AI框架，在单片机上也能够运行。  
来自官方的介绍：  
> TensorFlow Lite for Microcontrollers 是 TensorFlow Lite 的一个实验性移植版本，它适用于微控制器和其他一些仅有数千字节内存的设备。 它可以直接在“裸机”上运行，不需要操作系统支持、任何标准 C/C++ 库和动态内存分配。核心运行时(core runtime)在 Cortex M3 上运行时仅需 16KB，加上足以用来运行语音关键字检测模型的操作，也只需 22KB 的空间。


## 二、TFLM使用方法  
接下来准备在PC上编译TFLM，并运行基准测试。

首先下载TFLM代码，使用如下命令：  
```
git clone https://github.com/tensorflow/tflite-micro.git
```
TFLM是一个边缘AI推理框架，可以简单理解为一个计算库；另外，TFLM项目内提供了基准测试，用于对框架进行简单的测试，可以实现用一个AI模型在不同设备上的进行推理，并就各自推理性能进行对比。  
### 2.1 基准测试简介
TFLM代码仓顶层的README.md中给出了基准测试文档链接：

https://github.com/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/benchmarks/README.md  

![image](https://github.com/lus-oa/Loongson2k500-TensorFlowLite/assets/122666739/49e7521c-dc0e-4716-bb02-e9bb35761b0f)  
通过这个目录我们可以知道，TFLM提供了两个基准测试（实际有三个），分别是：

- 关键词基准测试
&emsp; - 关键词基准测试使用的是程序运行时生产的随机数据作为输入，所以它的输出是没有意义的
- 人体检测基准测试
&emsp;- 人体检测基准测试使用了两张bmp图片作为输入
&emsp;- 具体位于tensorflow\lite\micro\examples\person_detection\testdata子目录

