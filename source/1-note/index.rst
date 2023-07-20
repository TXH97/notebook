================================================================================
日志记录
================================================================================

sifive官方SDK编译过程
================================================================================

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
   
   * ``export FREEDOM_E_SDK_PIP_CACHE_PATH=
     /mnt/c/Users/TXH15/Desktop/FE310/freedom-e-sdk/pip-cache/``
   * ``export FREEDOM_E_SDK_VENV_PATH=
     /mnt/c/Users/TXH15/Desktop/FE310/freedom-e-sdk/venv/``
  
#. 编译工程： 
   ``make PROGRAM=sifive-welcome TARGET=qemu-sifive-e31 CONFIGURATION=debug software``
   
   * ``PROGRAM`` 支持的历程在 ``./software`` 目录下
   * ``TARGET`` 支持的目标在 ``./bsp`` 目录下
   
   .. note:: 如果出现在 python 中链接文件打开出错的问题，是因为 python 库版本依赖
      问题导致的，可修改下面文件中出错的部分临时解决：
      ``sudo vim /usr/lib/python3.10/collections/__init__.py``
  
#. 生成的目标文件在： ``./software/sifive-welcome/debug`` 目录下
#. 启动 qemu: ``qemu-system-riscv32.exe -nographic -M sifive_e -cpu sifive-e31
   -bios none -gdb tcp::1234 -S``
#. 启动 gdb: ``riscv64-unknown-elf-gdb example-freertos-minimal.elf``
#. 在 gdb 中使用 ``target remote localhost:1234`` 连接目标板，并使用 ``load`` 下载程序
   
   .. note:: Linux 子系统中编译的程序路径不一样，调试时会出现找不到文件的提示；可使用 
      ``set substitute-path /mnt/c/Users/TXH15/Desktop/FE310/freedom-e-sdk 
      C:/Users/TXH15/Desktop/FE310/freedom-e-sdk``
      修改文件路径的指向。

.. note:: 如果需要编译 RTOS 的版本，只需要使用下面这种方法，选择带 RTOS 的编译即可：
   ``make PROGRAM=example-freertos-minimal TARGET=qemu-sifive-e31 CONFIGURATION=debug software``

.. note:: 参考链接：

   * freedom_e_sdk 仓库： `源码 <https://github.com/sifive/freedom-e-sdk>`_
   * freedom_e_sdk 参考说明： `使用说明 <https://github.com/sifive/freedom-e-sdk/blob/master/README.md>`_



--------------------------------------------------------------------------------

Windows10 搭建 Docker
================================================================================

使用前说明：

* 在 docker 官网下载 windows10 桌面工具，并安装
* 在微软商店中下载安装 ubuntu 发现版，并点击图标启动，启动时会让填账户与密码。
* 在 docker 网站 ``https://hub.docker.com/`` 上查找需要的镜像，获取镜像的指令
* 在本地启动 cmd 等终端，输入指令等待获取即可

安装镜像
########

#. 进入 Docker 镜像网站 `DockerHub <https://hub.docker.com/>`_ 
   搜寻所需的镜像，并获取安装命令。
#. 在主机电脑的 CMD 中执行获取到的命令（也可自己组装命令）。
#. 使用 ``docker images`` 查看已安装的镜像。

创建容器
########

#. 在主机电脑的 CMD 中执行 ``docker run -itd 镜像名:镜像TAG`` 创建并启动容器。

   * ``-itd`` 是启动并分离容器。
   * 也可直接使用 ``docker run -it 镜像名:镜像TAG bash`` 创建启动并进入容器。
   * 如果需要容器与本地电脑实现文件共享，可挂载本地文件夹到容器中；
     使用 ``docker run -itd -v 本地路径:挂载路径 镜像名:镜像TAG``

#. 查看所有容器： ``docker ps -a``
#. 查看运行容器： ``docker ps``
#. 进入容器： ``docker exec -it 容器名/容器ID bash``
#. 推出容器： 窗口执行 ``exit``
#. 停止容器： ``docker stop 容器名/容器ID``
#. 运行容器： ``docker start 容器名/容器ID``
#. 删除容器： ``docker rm 容器名/容器ID``

Docker 命令
###########

.. csv-table:: Docker 常用指令
   :header: "指令", "说明"
   :widths: 30, 100
   :align: center

   "docker images", "查看下载的镜像"
   "docker ps", "查看运行的容器"
   "docker stop 名称/id", "停止运行容器"
   "docker rm 名称/id", "删除容器"



--------------------------------------------------------------------------------

Windows 与 Linux 、 Mac 跨平台文件的问题
================================================================================

这三个系统的换行符是不同的，当文件需要跨平台编辑和使用时需要注意换行符的问题。

**踩过的坑**

* 在 windows 中使用 GIT 下载，在 docker 的 Linux 系统中使用，出现 configure 文件
  无法执行的情况；但下载的 SDK 是可以使用的。问题在于使用 GIT 下载时，自动将换行
  符进行替换，导致 Linux 中无法识别。

  .. note:: 解决方法：在 Linux 中下载并在 Linux 中执行；
     使用 ``git config --global core.autocrlf false`` 修改 GIT 上传与检出时不替换。



--------------------------------------------------------------------------------

QEMU 源码编译记录
================================================================================

使用 docker 搭建的基于 Fedora 系统的环境；

具体操作流程参考qemu官网::
   https://wiki.qemu.org/Hosts/W32

在环境配置中会遇到一些工具链的问题，需要手动安装，还有一些库页需要手动安装，
官网可能没有提供安装方法。

未完待续...



--------------------------------------------------------------------------------

VIRTIO 设备
================================================================================

**记录在 qemu-riscv32 上的 virt 平台上模拟 virtio 设备的记录。**

virtio 设备是按一定的大小顺序，从 virt 平台中的 virtio 基地址开始排列。



--------------------------------------------------------------------------------

VSCODE 调试
================================================================================

WATCH 窗口技巧

================== =============================================================
需求               解决方法
================== =============================================================
打印数组           在地址前添加 ``*(uint32_t(*)[16])``
显示十六进制       在变量后添加 ``,x``
================== =============================================================



--------------------------------------------------------------------------------

任务切换流程
================================================================================

#. 触发异常， CPU 将一些寄存器压入前一个栈
#. 在异常处理中，手动压入其他寄存器到前一个栈，并更新前一个任务的栈地址
#. 获取新的任务栈，并弹出手动压入的寄存器
#. 异常处理退出， CPU 出栈后一个栈中寄存器
#. 异常退出后就进入下一个栈中的 PC 当中。

* 触发异常时，将 PC.SP.R0~R3.R12 自动压入前一个任务栈。
* CPU 切换 SP 为主栈
* 进入异常处理函数， 软件将 R4~R11. 手动入上一个任务栈。
* 软件更新上一个任务的任务栈地址。
* 获取下一个任务栈地址，并出栈 R4~R11 。
* 更新 psp 为下一个任务栈。
* 异常退出， CPU 切换 SP 为任务栈。
* CPU 自动出栈 PC.SP.R0~R3.R12 。

.. note:: 这个过程中在异常处理中使用的主栈，并配置任务栈的内容与没有自动保存的寄
   存器，退出后自动切换到任务栈运行程序。



--------------------------------------------------------------------------------

内联汇编
================================================================================

__asm__ __volatile__(汇编语句模板: 输出部分: 输入部分: 破坏描述部分)

共四个部分：汇编语句模板，输出部分，输入部分，破坏描述部分，各部分使用":"格开，
汇编语句模板必不可少，其他三部分可选，如果使用了后面的部分，而前面部分为空，也需
要用":"格开，相应部分内容为空。

* 输出部分：输出部分描述输出操作数，不同的操作数描述符之间用逗号格开，每个操作数
  描述符由限定字符串和 C 语言变量组成。每个输出操作数的限定字符串必须包含"="表示
  他是一个输出操作数。



--------------------------------------------------------------------------------

链接文件
================================================================================

* ENTRY( 入口函数 ) ：指定程序入口
* MEMORY( 记忆存储 ) ：指定内存区域
* SECTIONS( 部分 ) ：程序分布
* KEEP( 保持 ) ：保留内容，没被链接也保留的内容
* PROVIDE ：为在任何链接目标中没有定义但是被引用的一个符号,而在链接脚本定义一个符号。

.. note:: 可参考：
   `链接文件的描述 <https://blog.csdn.net/weixin_44889430/article/details/128041438>`_



--------------------------------------------------------------------------------

QEMU 使用 TAP 联网
================================================================================

#. 创建本地 TAP 设备
   
   **LINUX**

   ``sudo ip tuntap add tap0 mode tap``

   ``sudo ip link set tap0 up``
   
   **WIN7/WIN10**

   安装 TAP 驱动程序：首先，您需要安装 TAP 驱动程序。在 Windows 7 上，您可以使用 
   OpenVPN 提供的 TAP-Windows 驱动程序。您可以从 OpenVPN 官方网站下载并安装适用于 
   Windows 7 的 TAP 驱动程序。

   创建 TAP 适配器：安装完 TAP 驱动程序后，您需要创建一个 TAP 适配器。打开 Windows 
   的设备管理器，找到"网络适配器"部分，右键单击并选择"添加 Legacy 硬件"。在向导中，
   选择"手动选择硬件"，然后选择"网络适配器"，然后从列表中选择" TAP-Windows Adapter V9 "
   作为您的虚拟网络适配器。完成后，将会创建一个新的 TAP 适配器。

#. 启动 QEMU

   ``qemu-system-arm -M vexpress-a9 -net nic -net tap,ifname=tap``

   上述命令假设您已经创建了名为 "tap" 的 TAP 设备，并且通过 "-ifname" 参数将其与
   虚拟机的网络设备关联。

#. 编写驱动

   Vexpress 使用的是 LAN9118 网卡。编写对应的驱动。

#. 参考

  * TAP 安装文档： `windows <https://blog.csdn.net/baidu_25117757/article/details/128302530>`_

  * OpenVPN 下载地址： `win7 与 win10 <https://build.openvpn.net/downloads/releases/>`_ 
  
  * WIN10 TAP 下载地址： `Win10 TAP  <https://build.openvpn.net/downloads/releases/tap-windows-9.24.7-I601-Win10.exe>`_ 

  * QEMU-TAP 配置文档： `TAP <https://blog.csdn.net/Mculover666/article/details/105664454/>`_

.. note:: 网络参数参考文档：
   `-net/-nic <https://www.codenong.com/cs106723798/>`_



--------------------------------------------------------------------------------

QEMU Vexpress-a9 Linux 开发搭建
================================================================================

Vexpress-a9 Linux 网络配置配置
##############################

参考博文： `ifconfig配置IP <https://www.python100.com/html/96023.html>`_

* QEMU 参数： ``qemu-system-arm -M vexpress-a9 -m 512M 
  -kernel C:\Users\TXH15\Desktop\linux\zImage 
  -dtb C:\Users\TXH15\Desktop\linux\vexpress-v2p-ca9.dtb -nographic --append "rw console=ttyAMA0" 
  -initrd C:\Users\TXH15\Desktop\linux\busybox.cpio 
  -smp 4 -serial COM1 -net nic -net tap,ifname=tap``

* 设置 TAP 虚拟网卡 IP 为： ``192.168.137.62``

* Linux 内核设置网卡 IP: ``ifconfig eth0 192.168.137.61``

* Linux 查看路由： ``route -n``

* Linux 设置默认路由： ``route add default gw 192.168.137.62 dev eth0``


搭建 nfs 网络文件服务
#####################

* 修改根文件系统
  
  ``qemu-system-arm -M vexpress-a9 -m 512M 
  -kernel C:\Users\TXH15\Desktop\linux\zImage 
  -dtb C:\Users\TXH15\Desktop\linux\vexpress-v2p-ca9.dtb -nographic --append "root=/dev/mmcblk0 rw console=ttyAMA0" 
  -sd C:\Users\TXH15\Desktop\linux\rootfs0328.ext3 
  -smp 4 -serial COM1 -net nic -net tap,ifname=tap``

  可不修改，这儿修改是为了在 Linux 上运行程序

* 查看是否支持 nfs 服务

  ``cat /proc/filesystems``

  如果结果显示 ``nodev nfs`` 则表示支持

* 搭建本地 nfs 服务器
  
  可参考博客： `在Windows上搭建NFS服务器 <https://blog.csdn.net/Wu_GuiMing/article/details/115872995>`_

  需要在本地服务器上设置客户端的地址，以及目录。这个过程中可能需要重启服务。

* Linux上启动 nfs 客户端

  ``mount -o nolock,addr=192.168.200.1 -t nfs 192.168.200.1:/d/nfs``

  * ``192.168.200.1`` 是本地主机地址 (TAP 虚拟网卡 IP)
  * ``192.168.200.1:/d/nfs`` 是本地主机上共享的文件夹



--------------------------------------------------------------------------------

交叉编译 Iperf2.0.9
================================================================================

可参考博文： 
`iperf交叉编译与简单使用 <https://blog.csdn.net/feitingfj/article/details/106923384>`_ 与 
`iperf使用与交叉编译 <https://www.yii666.com/blog/344762.html?action=onAll>`_


测试环境： ubuntu 16.0.4.6

#. 下载源码

   直接使用官方源码即可

#. 解压

   ``tar -vxf iperf-2.0.9-source.tar.gz``

#. 进入解压出的文件夹
#. 进行 configure
   
   ``./configure --host=arm-linux-gnueabihf CC=arm-linux-gnueabihf-gcc CFLAGS=-static CXX=arm-linux-gnueabihf-g++ CXXFLAGS=-static``

#. 编译

   ``make`` 与 ``make install``

#. 得到目标文件

   目标文件在 src 目录下

.. note:: 注意事项

   * 在配置时如果报 C++ 有问题，大概率是安装的交叉编译工具链有问题，只有 GCC 可以
     使用， G++ 是无法使用的，可在命令行中用命令验证 G++ 编译器。
   
   * 配置中出现问题也可能是权限的问题， 可以加上 ``sudo`` 进行配置核编译。
   * 尽量使用静态编译与链接，因为目标环境的 Linux 中可能不支持共享库等。



--------------------------------------------------------------------------------

屏障
================================================================================

**内存屏障**

内存屏障： rmb(), wmb(), mb() 可以防止硬件上的指令重排。
除了编译器，有的 CPU 也支持对指令进行重排来优化程序执行效率，这几个函数就是去防
止 CPU 去做这些事情。

* rmb() 是读访问内存屏障，它保证在屏障（调用 rmb() 的位置处）之后的任何读操作在
  执行之前，屏障之前的所有读操作都已经完成。
* wmb() 对应写操作，意思同上。
* mb() 就同时包含读和写操作，意思同上。

内存屏障是和硬件即 CPU 特性相关的，那么，如果你的 CPU 没有指令重排的能力，也就没
有必要防止指令重排了。例如，一款 CPU 不支持写指令重排，那么系统中的 wmb() 就直接
被定义成了 barrier()。


**优化屏障**

优化屏障： barrier() 防止编译器对内存访问的优化。
类似 volatile 关键字对于访问变量的作用。它告诉编译器，在插入 barrier() 的位置处，
内存中的内容都被更新了，你想读变量、映射到内存的寄存器等内容都需要真正到内存里去
读，这样就能保证 barrier 之后的读指令不会被优化掉。


**SMP 系统**

还有SMP系统中使用的 smp_rmb(), smp_wmb(), smp_mb()。
它们只用于 SMP 系统。在单处理器上它们被定义成 barrier()。


**总结**

#. wmb() / smp_wmb()

  * wmb() 是用于在写操作之后确保对其他处理器可见的内存屏障宏。
  * smp_wmb() 是用于在多处理器（ SMP ）系统中确保对其他处理器可见的内存屏障宏。

#. rmb() / smp_rmb()

  * rmb() 是用于在读操作之前确保读取最新数据的内存屏障宏。
  * smp_rmb() 是用于在多处理器（ SMP ）系统中确保读取最新数据的内存屏障宏。

#. mb() / smp_mb()

  * mb() 是用于在读写操作之前或之后确保顺序性的内存屏障宏。
  * smp_mb() 是用于在多处理器（ SMP ）系统中确保顺序性的内存屏障宏。


--------------------------------------------------------------------------------

编译器相关备注
================================================================================

* 似乎不是所有的交叉编译工具链都可以混用。对于不同的交叉编译工具链，其可能拥有不
  同的库，支持平台情况，优化情况都有可能有差距；同时，其遵循的标准可能也会有差异，
  从而导致汇编与源码之间调用存在差异。
* 对于函数调用相关的规则，有 AAPCS 与 EABI 规范等，规范了函数调用过程中行为。
* 不同编译器可能支持不同的功能，对代码的行为也可能会存在一定的差异，例如对函数添
  加一些特殊的标识等等。


