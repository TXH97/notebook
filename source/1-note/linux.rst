================================================================================
Linux 使用笔记
================================================================================

常用命令
================================================================================

交叉编译工具链的安装与卸载： arm-linux-gnueabihf
################################################

参考博文： `Ubuntu 18.04安装arm-linux-gcc交叉编译器的两种方法 <https://blog.csdn.net/FL1623863129/article/details/129692467>`_

编译器资源： `Linaro Releases <https://releases.linaro.org/components/toolchain/binaries/>`_

验证环境： ubuntu 16.0.4

* 安装 arm-linux-gnueabihf-gcc

  ``sudo apt-get install gcc-arm-linux-gnueabihf``

* 安装 arm-linux-gnueabihf-g++

  ``sudo apt-get install g++-arm-linux-gnueabihf``

* 卸载 arm-linux-gnueabihf-gcc

  ``sudo apt-get remove gcc-arm-linux-gnueabihf``

* 卸载 arm-linux-gnueabihf-g++

  ``sudo apt-get remove g++-arm-linux-gnueabihf``


压缩解压文件
############

* 解压文件

  ``tar zxvf iperf-2.0.9-source.tar.gz``

* 压缩文件
  
  ``tar czvf a.tgz b``


查看文件属性
############

* 查看可执行文件
  
  ``file iperf``


Ubuntu apt源更换
################

* 备份
  
  ``sudo cp /etc/apt/sources.list  /etc/apt/sources.list_save``

* 跟换源文件
  
  ``sudo vim /etc/apt/sources.list``

* 更新缓存
  
  ``sudo apt-get update``

* 更新软件
  
  ``sudo apt-get upgrade``

反编译 dtb 文件
###############

* 安装编译器
  
  ``sudo apt-get install device-tree-compiler``

* 执行反编译

  ``dtc -I dtb -O dts -o hello.dts ./test_dtb.dtb``

