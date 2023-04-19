
wifi 模块的使用
===============

以 i.Mx-RT 1052 为例

#. 选用带 wifi 模块的 board 
#. 在 guiconfig 中勾选 ``WIFI/cyw43362`` 外设，并使能 ``STA`` 、 ``AP`` 与 ``DHCP``
#. 继续勾选 shell 下的 wifi 指令
#. 编译工程并下载到开发板
#. 在 shell 中输入 ``wifi power cyw43362 on`` 开启wifi
#. 在 shell 中输入 ``wifi join netname password`` 连接wifi


wifi 结构分析
=============

分为 wifi 子系统、 wifi 驱动、 wiced 组件四部分。

wifi 子系统
-----------

wifi 子系统 将 wifi 设备的方法分为 基础方法、sta 方法、 ap 方法、 日志方法、 漫游
方法等， 具体方法参考 ``awb_wifi_ops_t`` 。

wifi 子系统将 wifi 设备注册到 设备文件系统中， 设备文件系统可通过设备的访问方式
访问 wifi 模块，具体的访问细节可参考 ``aw_wifi_ioctl.h`` 中的定义。


wiced 组件
----------

wiced 组件是 CYPRESS （赛普拉斯）公司定义的，对于该公司生产的 wifi 模块，都可以
使用 wiced 组件。

wiced 可以看作是对 wifi 芯片指令的封装。

wifi 驱动
---------

wifi 驱动在框架的公共驱动中实现。
可分为 可将该驱动划分为两部分： wifi 设备驱动与 wiced 底层适配层。

**WiFi 设备驱动**

该部分代码实现在 ``awb_cyw43xxx.c`` 文件中，
其内容是实例一个 wifi 设备，实例 wifi 设备的 基础方法、 sta 方法、 ap 方法 等等，
并将 wifi 设备注册进 wifi 子系统中。
根据使能的 wifi 设备类型， STA 或 AP 设备，实例一个网卡设备并注册到网络子系统中。

wifi 设备方法的实现是在 wiced 组件的基础上完成的，调用 wiced 的接口来实现方法的功能。

.. note:: 主要用到的是 wiced 中的 wwd 部分，且 wiced 组件在以前已做过适配。

**wiced 底层适配层**

wiced 组件与系统相关的适配在 wiced 组件内部已完成。
对于 wiced 组件底层相关的适配，在驱动中来实现的。
具体代码参考 ``awb_cyw43362_wiced.c``  文件。

适配内容包括以下几个部分：

#. wiced 底层硬件的初始化、上电、复位、中断等操作
#. wiced 底层硬件发送与接收数据接口的适配
#. wifi 固件操作接口的适配

.. note:: 该部分使用了一个任务来处理连接改变的事务，是为了解决死锁的问题。

