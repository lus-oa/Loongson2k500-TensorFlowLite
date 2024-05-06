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

&emsp; - 人体检测基准测试使用了两张bmp图片作为输入  
&emsp; - 具体位于tensorflow\lite\micro\examples\person_detection\testdata子目录  
### 2.2 安装依赖的软件
由于TFLM的过程中，需要下载一些测试数据，并使用Pillow库将部分测试图片转化为C代码。因此，编译TFLM之前需要先安装Pillow库，以及一些命令行工具。

运行TFLM基准测试之前，使用如下命令先安装依赖的一些软件：  
```
sudo apt install python3 python3-pip git unzip wget build-essential
```
Pillow是一个Python库，因此如果PC的Linux系统上还没有Python则需要安装。  
#### 2.2.1 设置pip源
将pip源设置为国内源，可以加速pip包安装，执行如下命令：
```shell
pip config set global.index-url <http://mirrors.aliyun.com/pypi/simple/>
pip config set global.trusted-host mirrors.aliyun.com
pip config set global.timeout 120
```
#### 2.2.2 安装Pillow库
执行如下命令，安装pillow库：  
```
pip install pillow
```
安装过程会编译pillow包中的C/C++源代码文件，速度较慢，耐心等待。

如果Pillow安装过程报错：The headers or library files could not be found for jpeg

需要先安装libjpeg库：

`apt-get install libjpeg-dev zlib1g-dev`  

### 2.3 基准测试命令
参考”Run on x86”，在x86 PC上运行关键词基准测试的命令是：  
```
make -f tensorflow/lite/micro/tools/make/Makefile run_keyword_benchmark
```
在PC上运行人体检测基准测试的命令是：  
```
make -f tensorflow/lite/micro/tools/make/Makefile run_person_detection_benchmark
```
执行这两个命令，会依次执行如下步骤：  
&emsp; - 调用几个下载脚本，下载依赖库和数据集；
&emsp; - 编译测试程序；
&emsp; - 运行测试程序；
`tensorflow/lite/micro/tools/make/Makefile`代码片段中，可以看到调用了几个下载脚本：

