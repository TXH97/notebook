================================================================================
ARMv6-M 芯片架构
================================================================================


#. ARMv6-M 仅支持 Thumb 指令执行状态；参考：



寄存器
================================================================================

专用程序状态寄存器： `xPSR <https://developer.arm.com/documentation/ddi0419/c/System-Level-Architecture/System-Level-Programmers--Model/Registers/The-special-purpose-program-status-registers--xPSR?lang=en>`_


异常
================================================================================

官方参考文档： `异常模型 <https://developer.arm.com/documentation/ddi0419/c/System-Level-Architecture/System-Level-Programmers--Model/ARMv6-M-exception-model?lang=en>`_

在 ARMv6-M 手册中描述，矢量表偏移寄存器中的值由实现来定义的；
根据这个查阅 Cortex-M0 手册，手册中描述：向量表固定在地址 0x00000000 。
因此判断基于 Cortex-M0 的芯片，向量表都在 0x0 地址处。
对于在 0x0 地址处的向量表，是无法在运行时进行修改的。

.. note:: 要实现向量表在运行时的修改，需要将向量表放置在 RAM 等区域。
    也就意味着需要支持向量表偏移设置。

* ARMv6-M 支持 2 位优先级字段，提供四个优先级。
* ARMv6-M 不支持可配置的优先级分组
* 当两个挂起异常具有相同的组优先级时，作为优先级优先级规则的一部分，较低的挂起异
  常编号优先于较高的挂起编号。



