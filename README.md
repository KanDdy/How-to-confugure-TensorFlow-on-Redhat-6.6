# How-to-confugure-TensorFlow-on-Redhat-6.6
## 记录我在RedHat 6.6下配置TensorFLow所遇到的坑.

我这里是按照[官网](https://www.tensorflow.org/get_started/os_setup)的方法,这里选择的是最方便的pip安装.
### 1.Python3安装  
由于我的系统自带的python版本是2.6.6,所以我需要重新装一个高版本(2.7/3.3+),因此首先下载python(由于之前在Ubuntu上用的是2.7.6,所以这次选择了Python3)
```
wget https://www.python.org/ftp/python/3.4.1/Python-3.4.1.tgz
tar -zxf  Python-3.4.1.tgz
cd Python 3.4.1
./configure
make -j8 && make install
```
这样装好之后,由于系统原本自带python,需要python3才能调出,因此需要调整一下(不调整也行，每次输python3即可):

python命令通常存在
/usr/bin/python和/usr/local/bin/python下面,一般是符号链接,优先使用后者.

如果有2.6和3.4两个版本,并且（用`ll /usr/bin/python*`查看）  
/usr/bin/python --> /usr/bin/python2.6  
/usr/local/bin/python --> /usr/local/bin/python3.4  
此时系统默认使用3.4, 如果想切换至2.6
直接用 
```
sudo ln -s /usr/local/bin/python /usr/bin/python2.6
```
即可.  
注意到Python3源码中自带pip3，所以不用我们自己装pip了.  

### 2.安装CUDA和cudnn
**2.1**.TensorFLow目前更新至rc0.12.0,推荐CUDA和cudnn版本分别为8.0和v5.1,公司服务器的硬件为GTX 1080,所以这里CUDA版本选择是8.0.从网上下载CUDA的安装包(这里是用的是.run，见[官网](https://developer.nvidia.com/cuda-downloads)),关于CUDA8.0的安装这里就不赘述了,自行网上搜索即可(建议参照[官方文档](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#axzz4YcvVyZKO)）

**2.2**.cudnn[下载地址](https://developer.nvidia.com/cudnn),下载之后,执行如下操作(假设你的CUDA toolkit已安装至/usr/local/cuda,即默认配置)
```
tar xvzf cudnn-8.0-linux-x64-v5.1-ga.tgz
sudo cp -P cuda/include/cudnn.h /usr/local/cuda/include
sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```
### 3.安装TensorFlow
直接用如下命令：
```
pip install tensorflow-gpu
```
系统会直接安装.如果上述命令无法安装,则根据下面的地址选择适合自己系统的链接,在终端输入:
```
# Ubuntu/Linux 64-bit, CPU only, Python 2.7
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.12.1-cp27-none-linux_x86_64.whl

# Ubuntu/Linux 64-bit, GPU enabled, Python 2.7
# Requires CUDA toolkit 8.0 and CuDNN v5. For other versions, see "Installing from sources" below.
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-0.12.1-cp27-none-linux_x86_64.whl

# Mac OS X, CPU only, Python 2.7:
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-0.12.1-py2-none-any.whl

# Mac OS X, GPU enabled, Python 2.7:
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/mac/gpu/tensorflow_gpu-0.12.1-py2-none-any.whl

# Ubuntu/Linux 64-bit, CPU only, Python 3.4
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.12.1-cp34-cp34m-linux_x86_64.whl

# Ubuntu/Linux 64-bit, GPU enabled, Python 3.4
# Requires CUDA toolkit 8.0 and CuDNN v5. For other versions, see "Installing from sources" below.
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-0.12.1-cp34-cp34m-linux_x86_64.whl

# Ubuntu/Linux 64-bit, CPU only, Python 3.5
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.12.1-cp35-cp35m-linux_x86_64.whl

# Ubuntu/Linux 64-bit, GPU enabled, Python 3.5
# Requires CUDA toolkit 8.0 and CuDNN v5. For other versions, see "Installing from sources" below.
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-0.12.1-cp35-cp35m-linux_x86_64.whl

# Mac OS X, CPU only, Python 3.4 or 3.5:
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-0.12.1-py3-none-any.whl

# Mac OS X, GPU enabled, Python 3.4 or 3.5:
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/mac/gpu/tensorflow_gpu-0.12.1-py3-none-any.whl
```
然后
```
# Python 2
$ sudo pip install --upgrade $TF_BINARY_URL

# Python 3
$ sudo pip3 install --upgrade $TF_BINARY_URL
```
等待安装完成即可.

### 4.TensorFlow无法import
**4.1**.GLIBC升级. 
安装完成之后,在验证TensorFlow时,发现无法import,根据报错信息发现是缺少CLIBC版本过低,需要GLIBC>=2.16,而系统自带的是2.12,因此需要升级GLIBC.   
GLIBC是GNU发布的LIBC库,即C运行库.GLIBC是Linux系统中最底层的API,几乎其它任何运行库都会依赖于GLIBC(mv,ls等命令就依赖于GLIBC).GLIBC除了封装Linux操作系统所提供的系统服务外,它本身也提供了许多其它一些必要功能服务的实现.按照下面的方法升级GLIBC：
```
wget http://ftp.gnu.org/gnu/glibc/glibc-2.19.tar.gz

tar -xvf  glibc-2.19.tar.gz

mkdir glibc-build-2.19

cd glibc-build-2.19

../glibc-2.19/configure  --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin

make & make install  
```
make是成功后，build目录下编译出了一个新的libc.so.6 （/glibc-build-2.15/libc.so.6， 我们会发现这实际上也是一个软连接，真实的lib文件时libc.so  
```
cd glibc-build-2.19 
ll libc.so.6
```
查看库文件，可以看到2.12的旧库文件还在，多了2.19版本的库文件，而且软链接文件全部指向了2.19版本
```
ll /lib64/libc*
```
此时，查看glibc支持的版本可以看到GLIBC的版本已经成功升级到2.19.
```
strings libc.so | grep GLIBC
```  
**4.2**.GLIBCXX升级  
继续import,发现又报错这次则提示我缺少GLIBCXX3.4.19,用
```
strings /usr/lib64/libstdc++.so.6|grep GLIBCXX
```  
发现系统只支持到3.4.13,因此又需要升级GLIBCXX,实际上是GCC版本过低(系统自带版本为4.4.1)，升级GCC即可  
下面的方法参照[这里](https://github.com/qiwsir/ITArticles/blob/master/Linux/upgrade_gcc_on_Centos.md)    
```
下载GCC最新版
wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.1/gcc-4.8.1.tar.gz

然后解压到文件夹
tar -xvzf gcc-4.8.1.tar.gz

进入解压缩之后的目录
cd gcc-4.8.1
然后执行下面的命令
./contrib/download_prerequisites
再返回上一层,建立build_gcc_4.8.1目录,这个目录和gcc-4.8.1平行
cd ..
mkdir build_gcc_4.8.1
进入刚建立的目录,并执行编译过程
cd build_gcc_4.8.1
../gcc-4.8.1/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib  
make -j23
make install
OK,在build_gcc_4.8.1中将gcc已经安装完成

确定新安装的GCC的路径,之前安装时记下最后mv时的路径即可,我是默认安在了/usr/local/bin

ls /usr/local/bin | grep gcc
执行
/usr/sbin/update-alternatives --install  /usr/bin/gcc gcc /usr/local/bin/x86_64-unknown-linux-gnu-gcc-4.8.1 40

gcc --version      #查看版本

/usr/sbin/update-alternatives --install /usr/bin/g++ g++ /usr/local/bin/g++ 40

g++ --version     #查看版本
```
然后进入目录是,  .libs是隐藏的:
～/build_gcc_4.8.1/x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs
用下面的命令查看GLIBCXX的版本
```
Strings libstdc++.so.6.0.18|grep GLIBCXX
```
一般来说,里面就有满足需要的GLIBCXX版本了
然后,把该文件拷贝到/usr/lib64下
然后将libstdc++.so.6指向libstdc++.so.6.0.18:
```
cp libstdc++.so.6.0.18 /usr/lib64 
ln -sf libstdc++.so.6.0.18 libstdc++.so.6
```
