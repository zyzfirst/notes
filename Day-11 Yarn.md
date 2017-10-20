---
title: Day-11 Yarn
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


## yarn的基本架构
>YARN是Hadoop 2.0中的资源管理系统，它的基本设计思想是将MRv1中的JobTracker拆分成了两个独立的服务：**一个全局的资源管理器ResourceManager和每个应用程序特有的ApplicationMaster。
其中ResourceManager负责整个系统的资源管理和分配，而ApplicationMaster负责单个应用程序的管理。**

## yarn的基本组成结构
>YARN总体上仍然是Master/Slave结构，在整个资源管理框架中，ResourceManager为Master，NodeManager为Slave，ResourceManager负责对各个NodeManager上的资源进行统一管理和调度。当用户提交一个应用程序时，需要提供一个用以跟踪和管理这个程序的ApplicationMaster，它负责向ResourceManager申请资源，并要求NodeManger启动可以占用一定资源的任务。由于不同的ApplicationMaster被分布到不同的节点上，因此它们之间不会相互影响。在本小节中，我们将对YARN的基本组成结构进行介绍。
>1.  **ResourceManager（RM）**
RM是一个全局的资源管理器，负责整个系统的资源管理和分配。它主要由两个组件构成：调度器（Scheduler）和应用程序管理器（Applications Manager，ASM）。
（1）**调度器**
调度器根据容量、队列等限制条件（如每个队列分配一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各个正在运行的应用程序。
需要注意的是，该调度器是一个“纯调度器”，它不再从事任何与具体应用程序相关的工作，比如不负责监控或者跟踪应用的执行状态等，也不负责重新启动因应用执行失败或者硬件故障而产生的失败任务，这些均交由应用程序相关的ApplicationMaster完成。调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位用一个抽象概念“资源容器”（Resource Container，简称Container）表示，Container是一个动态资源分配单位，它将内存、CPU、磁盘、网络等资源封装在一起，从而限定每个任务使用的资源量。此外，该调度器是一个可插拔的组件，用户可根据自己的需要设计新的调度器，YARN提供了多种直接可用的调度器，比如Fair Scheduler和Capacity Scheduler等。
（2） **应用程序管理器**
应用程序管理器负责管理整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动ApplicationMaster、监控ApplicationMaster运行状态并在失败时重新启动它等。

>2. **ApplicationMaster（AM）**
用户提交的每个应用程序均包含1个AM，主要功能包括：
与RM调度器协商以获取资源（用Container表示）；
将得到的任务进一步分配给内部的任务；
与NM通信以启动/停止任务；
监控所有任务运行状态，并在任务运行失败时重新为任务申请资源以重启任务。
当前YARN自带了两个AM实现，一个是用于演示AM编写方法的实例程序distributedshell，它可以申请一定数目的Container以并行运行一个Shell命令或者Shell脚本；另一个是运行MapReduce应用程序的AM—MRAppMaster，我们将在第8章对其进行介绍。此外，一些其他的计算框架对应的AM正在开发中，比如Open MPI、Spark等。

**>3. NodeManager（NM）**
>NM是每个节点上的资源和任务管理器，一方面，它会定时地向RM汇报本节点上的资源使用情况和各个Container的运行状态；另一方面，它接收并处理来自AM的Container启动/停止等各种请求。

**>4. Container**
>Container是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，当AM向RM申请资源时，RM为AM返回的资源便是用Container表示的。YARN会为每个任务分配一个Container，且该任务只能使用该Container中描述的资源。
需要注意的是，Container不同于MRv1中的slot，它是一个动态资源划分单位，是根据应用程序的需求动态生成的。截至本书完成时，YARN仅支持CPU和内存两种资源，且使用了轻量级资源隔离机制Cgroups进行资源隔离。

## yarn的工作流程
>**当用户向YARN中提交一个应用程序后，YARN将分两个阶段运行该应用程序：
第一个阶段是启动ApplicationMaster；
第二个阶段是由ApplicationMaster创建应用程序，为它申请资源，并监控它的整个运行过程，直到运行完成。**

>步骤1　用户向YARN中提交应用程序，其中包括ApplicationMaster程序、启动ApplicationMaster的命令、用户程序等。
步骤2　ResourceManager为该应用程序分配第一个Container，并与对应的Node-Manager通信，要求它在这个Container中启动应用程序的ApplicationMaster。
步骤3　ApplicationMaster首先向ResourceManager注册，这样用户可以直接通过ResourceManager查看应用程序的运行状态，然后它将为各个任务申请资源，并监控它的运行状态，直到运行结束，即重复步骤4~7。
步骤4　ApplicationMaster采用轮询的方式通过RPC协议向ResourceManager申请和领取资源。
步骤5　一旦ApplicationMaster申请到资源后，便与对应的NodeManager通信，要求它启动任务。
步骤6　NodeManager为任务设置好运行环境（包括环境变量、JAR包、二进制程序等）后，将任务启动命令写到一个脚本中，并通过运行该脚本启动任务。
步骤7　各个任务通过某个RPC协议向ApplicationMaster汇报自己的状态和进度，以让ApplicationMaster随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务。
 在应用程序运行过程中，用户可随时通过RPC向ApplicationMaster查询应用程序的当前运行状态。
步骤8　应用程序运行完成后，ApplicationMaster向ResourceManager注销并关闭自己。

![enter description here][1]


  [1]:https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508513100546.jpg