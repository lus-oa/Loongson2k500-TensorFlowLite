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
 - 具体位于tensorflow\lite\micro\examples\person_detection\testdata子目录  
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
![image](https://github.com/lus-oa/Loongson2k500-TensorFlowLite/assets/122666739/811b67e3-f633-4d66-a6c1-f0984e31652a)  
flatbuffers_download.sh和kissfft_download.sh脚本第一次执行时，会将相应的压缩包下载到本地，并解压，具体细节参见代码内容；

pigweed_download.sh脚本会克隆一个代码仓，再检出一个特定版本：    
![image](https://github.com/lus-oa/Loongson2k500-TensorFlowLite/assets/122666739/e7462f80-2690-4a9d-a9aa-516f40dcd671)  
这里需要注意的是，代码仓https://pigweed.googlesource.com/pigweed/pigweed 国内一般无法访问（因为域名googlesource.com被禁了）。将此连接修改为克隆好的代码仓：https://github.com/xusiwei/pigweed.git 可以解决因为国内无法访问googlesource.com而无法下载pigweed测试数据的问题

### 2.4 基准测试构建规则
`tensorflow/lite/micro/tools/make/Makefile`文件是Makefile总入口文件，该文件中定义了一些makefile宏函数，并通过include引入了其他文件，包括定义了两个基准测试编译规则的`tensorflow/lite/micro/benchmarks/Makefile.inc`文件：  
```
KEYWORD_BENCHMARK_SRCS := \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/benchmarks/keyword_benchmark.cc

KEYWORD_BENCHMARK_GENERATOR_INPUTS := \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/models/keyword_scrambled.tflite

KEYWORD_BENCHMARK_HDRS := \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/benchmarks/micro_benchmark.h

KEYWORD_BENCHMARK_8BIT_SRCS := \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/benchmarks/keyword_benchmark_8bit.cc

KEYWORD_BENCHMARK_8BIT_GENERATOR_INPUTS := \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/models/keyword_scrambled_8bit.tflite

KEYWORD_BENCHMARK_8BIT_HDRS := \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/benchmarks/micro_benchmark.h

PERSON_DETECTION_BENCHMARK_SRCS := \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/benchmarks/person_detection_benchmark.cc

PERSON_DETECTION_BENCHMARK_GENERATOR_INPUTS := \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/examples/person_detection/testdata/person.bmp \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/examples/person_detection/testdata/no_person.bmp

ifneq ($(CO_PROCESSOR),ethos_u)
  PERSON_DETECTION_BENCHMARK_GENERATOR_INPUTS += \
    $(TENSORFLOW_ROOT)tensorflow/lite/micro/models/person_detect.tflite
else
  # Ethos-U use a Vela optimized version of the original model.
  PERSON_DETECTION_BENCHMARK_SRCS += \
  $(GENERATED_SRCS_DIR)$(TENSORFLOW_ROOT)tensorflow/lite/micro/models/person_detect_model_data_vela.cc
endif

PERSON_DETECTION_BENCHMARK_HDRS := \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/examples/person_detection/model_settings.h \
$(TENSORFLOW_ROOT)tensorflow/lite/micro/benchmarks/micro_benchmark.h

# Builds a standalone binary.
$(eval $(call microlite_test,keyword_benchmark,\
$(KEYWORD_BENCHMARK_SRCS),$(KEYWORD_BENCHMARK_HDRS),$(KEYWORD_BENCHMARK_GENERATOR_INPUTS)))

# Builds a standalone binary.
$(eval $(call microlite_test,keyword_benchmark_8bit,\
$(KEYWORD_BENCHMARK_8BIT_SRCS),$(KEYWORD_BENCHMARK_8BIT_HDRS),$(KEYWORD_BENCHMARK_8BIT_GENERATOR_INPUTS)))

$(eval $(call microlite_test,person_detection_benchmark,\
$(PERSON_DETECTION_BENCHMARK_SRCS),$(PERSON_DETECTION_BENCHMARK_HDRS),$(PERSON_DETECTION_BENCHMARK_GENERATOR_INPUTS)))
```
从这里可以看到，实际上有三个基准测试程序，比文档多了一个 keyword_benchmark_8bit ，应该是 keword_benchmark的8bit量化版本。另外，可以看到有三个tflite的模型文件。  
### 2.5 Keyword基准测试
关键词基准测试使用的模型较小，比较适合在STM32 F3/F4这类主频低于100MHz的MCU。

这个基准测试的模型比较小，计算量也不大，所以在PC上运行这个基准测试的耗时非常短：
![image](https://github.com/lus-oa/Loongson2k500-TensorFlowLite/assets/122666739/f4b43928-1a73-4315-aed2-07757732cb11)   
可以看到，在PC上运行关键词唤醒的速度非常快，10次时间不到1毫秒。

模型文件路径为：./tensorflow/lite/micro/models/keyword_scrambled.tflite

### 2.6 Person detection基准测试
人体检测基准测试的计算量相对要大一些，运行的时间也要长一些：  
```shell
loongson@ubuntu:~/Desktop/tflite-micro$ make -f tensorflow/lite/micro/tools/make/Makefile run_person_detection_benchmark
tensorflow/lite/micro/tools/make/downloads/flatbuffers already exists, skipping the download.
tensorflow/lite/micro/tools/make/downloads/kissfft already exists, skipping the download.
tensorflow/lite/micro/tools/make/downloads/pigweed already exists, skipping the download.
g++ -std=c++11 -fno-rtti -fno-exceptions -fno-threadsafe-statics -Werror -fno-unwind-tables -ffunction-sections -fdata-sections -fmessage-length=0 -DTF_LITE_STATIC_MEMORY -DTF_LITE_DISABLE_X86_NEON -Wsign-compare -Wdouble-promotion -Wshadow -Wunused-variable -Wunused-function -Wswitch -Wvla -Wall -Wextra -Wmissing-field-initializers -Wstrict-aliasing -Wno-unused-parameter  -DTF_LITE_USE_CTIME -Os -I. -Itensorflow/lite/micro/tools/make/downloads/gemmlowp -Itensorflow/lite/micro/tools/make/downloads/flatbuffers/include -Itensorflow/lite/micro/tools/make/downloads/ruy -Itensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/ -Itensorflow/lite/micro/tools/make/downloads/kissfft -c tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/tensorflow/lite/micro/examples/person_detection/testdata/person_image_data.cc -o tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/obj/core/tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/tensorflow/lite/micro/examples/person_detection/testdata/person_image_data.o
g++ -std=c++11 -fno-rtti -fno-exceptions -fno-threadsafe-statics -Werror -fno-unwind-tables -ffunction-sections -fdata-sections -fmessage-length=0 -DTF_LITE_STATIC_MEMORY -DTF_LITE_DISABLE_X86_NEON -Wsign-compare -Wdouble-promotion -Wshadow -Wunused-variable -Wunused-function -Wswitch -Wvla -Wall -Wextra -Wmissing-field-initializers -Wstrict-aliasing -Wno-unused-parameter  -DTF_LITE_USE_CTIME -Os -I. -Itensorflow/lite/micro/tools/make/downloads/gemmlowp -Itensorflow/lite/micro/tools/make/downloads/flatbuffers/include -Itensorflow/lite/micro/tools/make/downloads/ruy -Itensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/ -Itensorflow/lite/micro/tools/make/downloads/kissfft -c tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/tensorflow/lite/micro/examples/person_detection/testdata/no_person_image_data.cc -o tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/obj/core/tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/tensorflow/lite/micro/examples/person_detection/testdata/no_person_image_data.o
g++ -std=c++11 -fno-rtti -fno-exceptions -fno-threadsafe-statics -Werror -fno-unwind-tables -ffunction-sections -fdata-sections -fmessage-length=0 -DTF_LITE_STATIC_MEMORY -DTF_LITE_DISABLE_X86_NEON -Wsign-compare -Wdouble-promotion -Wshadow -Wunused-variable -Wunused-function -Wswitch -Wvla -Wall -Wextra -Wmissing-field-initializers -Wstrict-aliasing -Wno-unused-parameter  -DTF_LITE_USE_CTIME -Os -I. -Itensorflow/lite/micro/tools/make/downloads/gemmlowp -Itensorflow/lite/micro/tools/make/downloads/flatbuffers/include -Itensorflow/lite/micro/tools/make/downloads/ruy -Itensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/ -Itensorflow/lite/micro/tools/make/downloads/kissfft -c tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/tensorflow/lite/micro/models/person_detect_model_data.cc -o tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/obj/core/tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/tensorflow/lite/micro/models/person_detect_model_data.o
g++ -std=c++11 -fno-rtti -fno-exceptions -fno-threadsafe-statics -Werror -fno-unwind-tables -ffunction-sections -fdata-sections -fmessage-length=0 -DTF_LITE_STATIC_MEMORY -DTF_LITE_DISABLE_X86_NEON -Wsign-compare -Wdouble-promotion -Wshadow -Wunused-variable -Wunused-function -Wswitch -Wvla -Wall -Wextra -Wmissing-field-initializers -Wstrict-aliasing -Wno-unused-parameter  -DTF_LITE_USE_CTIME -I. -Itensorflow/lite/micro/tools/make/downloads/gemmlowp -Itensorflow/lite/micro/tools/make/downloads/flatbuffers/include -Itensorflow/lite/micro/tools/make/downloads/ruy -Itensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/ -Itensorflow/lite/micro/tools/make/downloads/kissfft -o tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/bin/person_detection_benchmark tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/obj/core/tensorflow/lite/micro/benchmarks/person_detection_benchmark.o tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/obj/core/tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/tensorflow/lite/micro/examples/person_detection/testdata/person_image_data.o tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/obj/core/tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/tensorflow/lite/micro/examples/person_detection/testdata/no_person_image_data.o tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/obj/core/tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/genfiles/tensorflow/lite/micro/models/person_detect_model_data.o tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/lib/libtensorflow-microlite.a -Wl,--fatal-warnings -Wl,--gc-sections -lm
tensorflow/lite/micro/tools/make/gen/linux_x86_64_default/bin/person_detection_benchmark non_test_binary linux
InitializeBenchmarkRunner took 192 ticks (0 ms).

WithPersonDataIterations(1) took 32299 ticks (32 ms)
DEPTHWISE_CONV_2D took 895 ticks (0 ms).
DEPTHWISE_CONV_2D took 895 ticks (0 ms).
CONV_2D took 1801 ticks (1 ms).
DEPTHWISE_CONV_2D took 424 ticks (0 ms).
CONV_2D took 1465 ticks (1 ms).
DEPTHWISE_CONV_2D took 921 ticks (0 ms).
CONV_2D took 2725 ticks (2 ms).
DEPTHWISE_CONV_2D took 206 ticks (0 ms).
CONV_2D took 1367 ticks (1 ms).
DEPTHWISE_CONV_2D took 423 ticks (0 ms).
CONV_2D took 2540 ticks (2 ms).
DEPTHWISE_CONV_2D took 102 ticks (0 ms).
CONV_2D took 1265 ticks (1 ms).
DEPTHWISE_CONV_2D took 205 ticks (0 ms).
CONV_2D took 2449 ticks (2 ms).
DEPTHWISE_CONV_2D took 204 ticks (0 ms).
CONV_2D took 2449 ticks (2 ms).
DEPTHWISE_CONV_2D took 243 ticks (0 ms).
CONV_2D took 2483 ticks (2 ms).
DEPTHWISE_CONV_2D took 202 ticks (0 ms).
CONV_2D took 2481 ticks (2 ms).
DEPTHWISE_CONV_2D took 203 ticks (0 ms).
CONV_2D took 2489 ticks (2 ms).
DEPTHWISE_CONV_2D took 52 ticks (0 ms).
CONV_2D took 1222 ticks (1 ms).
DEPTHWISE_CONV_2D took 90 ticks (0 ms).
CONV_2D took 2485 ticks (2 ms).
AVERAGE_POOL_2D took 8 ticks (0 ms).
CONV_2D took 3 ticks (0 ms).
RESHAPE took 0 ticks (0 ms).
SOFTMAX took 2 ticks (0 ms).

NoPersonDataIterations(1) took 32148 ticks (32 ms)
DEPTHWISE_CONV_2D took 906 ticks (0 ms).
DEPTHWISE_CONV_2D took 924 ticks (0 ms).
CONV_2D took 1762 ticks (1 ms).
DEPTHWISE_CONV_2D took 446 ticks (0 ms).
CONV_2D took 1466 ticks (1 ms).
DEPTHWISE_CONV_2D took 897 ticks (0 ms).
CONV_2D took 2692 ticks (2 ms).
DEPTHWISE_CONV_2D took 209 ticks (0 ms).
CONV_2D took 1366 ticks (1 ms).
DEPTHWISE_CONV_2D took 427 ticks (0 ms).
CONV_2D took 2548 ticks (2 ms).
DEPTHWISE_CONV_2D took 102 ticks (0 ms).
CONV_2D took 1258 ticks (1 ms).
DEPTHWISE_CONV_2D took 208 ticks (0 ms).
CONV_2D took 2473 ticks (2 ms).
DEPTHWISE_CONV_2D took 210 ticks (0 ms).
CONV_2D took 2460 ticks (2 ms).
DEPTHWISE_CONV_2D took 203 ticks (0 ms).
CONV_2D took 2461 ticks (2 ms).
DEPTHWISE_CONV_2D took 230 ticks (0 ms).
CONV_2D took 2443 ticks (2 ms).
DEPTHWISE_CONV_2D took 203 ticks (0 ms).
CONV_2D took 2467 ticks (2 ms).
DEPTHWISE_CONV_2D took 51 ticks (0 ms).
CONV_2D took 1224 ticks (1 ms).
DEPTHWISE_CONV_2D took 89 ticks (0 ms).
CONV_2D took 2412 ticks (2 ms).
AVERAGE_POOL_2D took 7 ticks (0 ms).
CONV_2D took 2 ticks (0 ms).
RESHAPE took 0 ticks (0 ms).
SOFTMAX took 2 ticks (0 ms).

WithPersonDataIterations(10) took 326947 ticks (326 ms)

NoPersonDataIterations(10) took 352888 ticks (352 ms)
```
可以看到，人像检测模型运行10次的时间是三百多毫秒，一次平均三十几毫秒。这是在配备AMD标压R7 4800 CPU的Win10虚拟机下运行的结果。

模型文件路径为：./tensorflow/lite/micro/models/person_detect.tflite  

## TFLM交叉编译
前面说明了如何在PC上编译TFLM，以及运行TFLM基准测试。由于是在PC平台上直接编译和运行的，因此生成的可执行文件和库文件都是x86平台的。

如果要生成LoongArch的库和可执行程序，则需要进行交叉编译。

### 3.1 配置loongarch64-linux-gnu-gcc环境
配置loongarch64-linux-gnu-gcc环境比较简单，基本上只需要如下几步即可：

- 将龙芯交叉编译工具链的压缩包解压；
- 再将龙芯交叉编译工具链所在目录添加到PATH环境变量中；
具体操作龙芯开发板手册中有详细描述，这里不再赘述。

配置成功后，可以使用如下命令进行测试：
```shell
loongarch64-linux-gnu-gcc -v
```
能够成功输出版本信息，则表示配置正确。


