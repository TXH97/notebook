
sifive官方SDK编译过程
=====================

#. 下载官方 SDK 包；官方的 board 下会给出推荐的工具下载入口。
#. 在 win10 的 linux 子系统中搭建编译环境
   * 安装 make: ``apt install make``
   * 安装 python: ``apt install python3.10-venv``
     
     .. note:: python 解释器也许可以安装其他的，我这个需要修改 ``requirements.txt``
               中的 ``typed-ast`` 版本号到 ``1.4.3`` 才能编译。

   * 安装 gcc: ``sudo apt install gcc``
   * 安装 bc: ``sudo apt-get install bc``
   * 安装 riscv-gcc: ``sudo vim /etc/profile``

     .. note:: Linux 子系统能通过 /mnt 访问到本地的文件，将路径转换到 /mnt 下，再
               写入环境变量即可。工具链是在官网的 Tools 包中的。

#. 执行 ``sudo make pip-cache`` 构建环境
   
   .. note:: 构建环境需要等待一段时间。

#. 添加环境路径：
   
   * ``export FREEDOM_E_SDK_PIP_CACHE_PATH=/mnt/c/Users/TXH15/Desktop/FE310/freedom-e-sdk/pip-cache/``
   * ``export FREEDOM_E_SDK_VENV_PATH=/mnt/c/Users/TXH15/Desktop/FE310/freedom-e-sdk/venv/``
  
#. 编译工程： ``make PROGRAM=sifive-welcome TARGET=qemu-sifive-e31 CONFIGURATION=debug software``
   
   * ``PROGRAM`` 支持的历程在 ``./software`` 目录下
   * ``TARGET`` 支持的目标在 ``./bsp`` 目录下
   
   .. note:: 如果出现在 python 中链接文件打开出错的问题，是因为 python 库版本依赖
      问题导致的，可修改下面文件中出错的部分临时解决：
      ``sudo vim /usr/lib/python3.10/collections/__init__.py``
  
#. 生成的目标文件在： ``./software/sifive-welcome/debug`` 目录下
#. 启动 qemu: ``qemu-system-riscv32.exe -nographic -M sifive_e -cpu sifive-e31 -bios none -gdb tcp::1234 -S``
#. 启动 gdb: ``riscv64-unknown-elf-gdb example-freertos-minimal.elf``
#. 在 gdb 中使用 ``target remote localhost:1234`` 连接目标板，并使用 ``load`` 下载程序
   
   .. note:: Linux 子系统中编译的程序路径不一样，调试时会出现找不到文件的提示；可使用 
      ``set substitute-path /mnt/c/Users/TXH15/Desktop/FE310/freedom-e-sdk C:/Users/TXH15/Desktop/FE310/freedom-e-sdk``
      修改文件路径的指向。

.. note:: 如果需要编译 RTOS 的版本，只需要使用下面这种方法，选择带 RTOS 的编译即可：
   ``make PROGRAM=example-freertos-minimal TARGET=qemu-sifive-e31 CONFIGURATION=debug software``

.. note:: 参考链接：

   * freedom_e_sdk 仓库： ``https://github.com/sifive/freedom-e-sdk``
   * freedom_e_sdk 参考说明： ``https://github.com/sifive/freedom-e-sdk/blob/master/README.md``


Windows10 搭建 Docker
=====================

#. 在 docker 官网下载 windows10 桌面工具，并安装
#. 在微软商店中下载安装 ubuntu 发现版，并点击图标启动，启动时会让填账户与密码。
#. 在 docker 网站 ``https://hub.docker.com/`` 上查找需要的镜像，获取镜像的指令
#. 在本地启动 cmd 等终端，输入指令等待获取即可
#. 使用 ``docker images`` 可查看当前已下载的镜像
#. 执行 ``docker run -itd mec:v2 --name myname`` 启动镜像。使用 ``-it`` 参数会
   立即进入镜像，后面需加 ``bash`` ，表示启动执行的第一个指令。
#. 执行 ``docker exec -it quirky_montalcini bash`` 进行镜像运行。


.. csv-table:: Docker 常用指令
   :header: "指令", "说明"
   :widths: 30, 100
   :align: center

   "docker images": "查看下载的镜像"
   "docker ps": "查看运行的容器"
   "docker stop 名称/id": "停止运行容器"
   "docker rm 名称/id": "删除容器"

.. note::
   * 容器中无法直接访问本地的文件，可在启动容器时添加如下参数：
     ``-v 挂载目录:宿主机目录`` ，中间使用 ``:`` 分割


Windows 与 Linux 、 Mac 跨平台文件的问题
========================================

这三个系统的换行符是不同的，当文件需要跨平台编辑和使用时需要注意换行符的问题。

**踩过的坑**

* 在 windows 中使用 GIT 下载，在 docker 的 Linux 系统中使用，出现 configure 文件
  无法执行的情况；但下载的 SDK 是可以使用的。问题在于使用 GIT 下载时，自动将换行
  符进行替换，导致 Linux 中无法识别。

  .. note:: 解决方法：在 Linux 中下载并在 Linux 中执行；
     使用 ``git config --global core.autocrlf false`` 修改 GIT 上传与检出时不替换。


QEMU 源码编译记录
=================


