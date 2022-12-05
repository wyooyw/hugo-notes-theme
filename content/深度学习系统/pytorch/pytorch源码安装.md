---
linktitle: pytorch源码安装
summary: Summary of pytorch源码安装
type: book
---
# pytorch源码安装
https://github.com/pytorch/pytorch#from-source
### 1.准备环境
- gcc7.5、g++7.5(要求版本都在8以下)
- cuda11.3
由于服务器上各个版本的gcc都被放在了/usr/bin目录下，所以通过软链接将7.5版本的链接到一个文件夹里（方便后续设环境变量）：
```shell
ln
```

### 2.安装依赖包
按照[readme.md](https://github.com/pytorch/pytorch#from-source) 里的要求，安装mkl-include和magma-cuda113
```shell
conda install mkl mkl-include
conda install -c pytorch magma-cuda113
```
若服务器无法联网，可从https://anaconda.org将安装包下载到本机，再上传到服务器，再安装，例如：
```shell
conda install magma-cuda113-2.5.2-1.tar.bz2
```

### 3.拉取pytorch代码
由于服务器上不能连接外网，所以先在本地下载好源码和依赖的子模块，然后再一起上传到服务器。
```shell
git clone --recursive https://github.com/pytorch/pytorch
cd pytorch
git submodule sync
git submodule update --init --recursive --jobs 0
```

### 4.编译
脚本如下：
```shell
# CUDA
export CUDA_HOME=/usr/local/cuda-11.3
export PATH=/usr/local/cuda-11.3/bin:$PATH

# gcc and g++
export CC=/home/yiou/opensource/gcc7.5/gcc
export CXX=/home/yiou/opensource/gcc7.5/g++
export GCC=/home/yiou/opensource/gcc7.5/gcc
export GXX=/home/yiou/opensource/gcc7.5/g++
export PATH=/home/yiou/opensource/gcc7.5:$PATH

#These options can prevent some error in building process.
export LD_LIBRARY_PATH=/usr/local/cuda-11.3/lib64:$LD_LIBRARY_PATH
export CXXFLAGS="${CXXFLAGS} -D__STDC_FORMAT_MACROS -DTHRUST_IGNORE_CUB_VERSION_CHECK"

# If you want to compile a debug mode pytorch, set this to 1
export DEBUG=0

#If you don't use these component,just don't build them.
#With the following options,you can imporve the compile speed,and reduce the disk usage.
#More details are in https://github.com/pytorch/pytorch/blob/master/CONTRIBUTING.md#build-only-what-you-need
export USE_DISTRIBUTED=0
export USE_MKLDNN=0
export BUILD_TEST=0
export USE_FBGEMM=0
export USE_NNPACK=0
export USE_QNNPACK=0
export USE_XNNPACK=0
export BUILD_CAFFE2_OPS=0
export USE_NCCL=0

#This option can improve the compile speed.
export MAX_JOBS=10

# Use 'develop' the build result will stay in the current dictionary
python setup.py develop
```

