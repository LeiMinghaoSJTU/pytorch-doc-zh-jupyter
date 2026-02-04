# 分布式通信包 
- torch.distributed [¶](#distributed-communication-package-torch-distributed "永久链接到此标题")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/distributed>
>
> 原始地址：<https://pytorch.org/docs/stable/distributed.html>




!!! note "笔记"

    请参阅[PyTorch分布式概述](https://pytorch.org/tutorials/beginner/dist_overview.html)，简要介绍与分布式训练相关的所有功能。


## 后端 [¶](#backends "此标题的永久链接")


`torch.distributed` 支持三个内置后端，每个后端都有不同的功能。下表显示了哪些函数可与 CPU/CUDA tensor一起使用。仅当用于构建 PyTorch 的实现支持 CUDA 时，MPI 才支持 CUDA。


| 	 Backend	  | 	`gloo`	 | 	`mpi`	 | 	`nccl`	 |
| --- | --- | --- | --- |
| 	 Device	  | 	 CPU	  | 	 GPU	  | 	 CPU	  | 	 GPU	  | 	 CPU	  | 	 GPU	  |
| 	 send	  | 	 ✓	  | 	 ✘	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |
| 	 recv	  | 	 ✓	  | 	 ✘	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |
| 	 broadcast	  | 	 ✓	  | 	 ✓	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |
| 	 all_reduce	  | 	 ✓	  | 	 ✓	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |
| 	 reduce	  | 	 ✓	  | 	 ✘	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |
| 	 all_gather	  | 	 ✓	  | 	 ✘	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |
| 	 gather	  | 	 ✓	  | 	 ✘	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |
| 	 scatter	  | 	 ✓	  | 	 ✘	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |
| 	 reduce_scatter	  | 	 ✘	  | 	 ✘	  | 	 ✘	  | 	 ✘	  | 	 ✘	  | 	 ✓	  |
| 	 all_to_all	  | 	 ✘	  | 	 ✘	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |
| 	 barrier	  | 	 ✓	  | 	 ✘	  | 	 ✓	  | 	 ?	  | 	 ✘	  | 	 ✓	  |


### PyTorch 附带的后端 [¶](#backends-that-c​​ome-with-pytorch "永久链接到此标题")


 PyTorch 分布式包支持 Linux(稳定)、MacOS(稳定)和 Windows(原型)。默认情况下，Linux 会构建 Gloo 和 NCCL 后端并包含在 PyTorchdistributed 中(仅当使用 CUDA 构建时才包含 NCCL)。 MPI 是一个可选后端，仅当您从源代码构建 PyTorch 时才能包含该后端。 (例如，在安装了 MPI 的主机上构建 PyTorch。)




!!! note "笔记"

    从 PyTorch v1.8 开始，Windows 支持除 NCCL 之外的所有集体通信后端，如果 [`init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group") 的 init_method 参数指向对于文件，它必须遵循以下架构：



* 本地文件系统，`init_method="file:///d:/tmp/some_file"`
* 共享文件系统，`init_method="file://////{machine_name} /{share_folder_name}/some_file"`


 与Linux平台相同，您可以通过设置环境变量MASTER_ADDR和MASTER_PORT来启用TcpStore。


### 使用哪个后端？ [¶](#which-backend-to-use"此标题的永久链接")


 过去，我们经常被问到：“我应该使用哪个后端？”。



* 经验法则



+ 使用 NCCL 后端进行分布式 **GPU** 训练 
+ 使用 Gloo 后端进行分布式 **CPU** 训练。
* 具有 InfiniBand 互连的 GPU 主机



+ 使用 NCCL，因为它是当前唯一支持 InfiniBand 和 GPUDirect 的后端。
* 具有以太网互连的 GPU 主机



+ 使用NCCL，因为它目前提供最好的分布式GPU训练性能，特别是对于多进程单节点或多节点分布式训练。如果您在使用 NCCL 时遇到任何问题，请使用 Gloo 作为后备选项。 (请注意，Gloo 目前的 GPU 运行速度比 NCCL 慢。)
* 具有 InfiniBand 互连的 CPU 主机



+ 如果您的 InfiniBand 启用了 IP over IB，请使用 Gloo，否则请使用 MPI。我们计划在即将发布的版本中添加对 Gloo 的 InfiniBand 支持。
* 具有以太网互连的 CPU 主机



+ 使用 Gloo，除非您有特定原因使用 MPI。


### 通用环境变量 [¶](#common-environment-variables "永久链接到此标题")


#### 选择要使用的网络接口 [¶](#choosing-the-network-interface-to-use "Permalink to this header")


 默认情况下，NCCL 和 Gloo 后端都会尝试找到要使用的正确网络接口。如果自动检测到的接口不正确，您可以使用以下环境变量覆盖它(适用于各自的后端)：



* **NCCL_SOCKET_IFNAME** ，例如 `export NCCL_SOCKET_IFNAME=eth0`
* **GLOO_SOCKET_IFNAME** ，例如 `export GLOO_SOCKET_IFNAME=eth0`


 如果您使用的是 Gloo 后端，您可以通过用逗号分隔来指定多个接口，如下所示： `export GLOO_SOCKET_IFNAME=eth0,eth1,eth2,eth3` 。后端将以循环方式分派操作跨这些接口的时尚。所有进程都必须在此变量中指定相同数量的接口。


#### 其他 NCCL 环境变量 [¶](#other-nccl-environment-variables "永久链接到此标题")


**调试** 
- 如果 NCCL 失败，您可以设置 `NCCL_DEBUG=INFO` 来打印显式警告消息以及基本 NCCL 初始化信息。


 您还可以使用“NCCL_DEBUG_SUBSYS”来获取有关 NCCL 特定方面的更多详细信息。例如，“NCCL_DEBUG_SUBSYS=COLL”将打印集合调用的日志，这在调试挂起时可能会有所帮助，特别是由于集合类型或消息大小不匹配而导致的挂起。如果拓扑检测失败，设置“NCCL_DEBUG_SUBSYS=GRAPH”将有助于检查详细的检测结果并保存以供需要 NCCL 团队进一步帮助时参考。


**性能调优** 
- NCCL 根据其拓扑检测进行自动调优，以节省用户的调优工作。在某些基于套接字的系统上，用户仍然可以尝试调整“NCCL_SOCKET_NTHREADS”和“NCCL_NSOCKS_PERTHREAD”以增加套接字网络带宽。 NCCL 已针对某些云提供商(例如 AWS 或 GCP)预先调整了这两个环境变量。


 NCCL环境变量的完整列表请参考【NVIDIA NCCL官方文档】(https://docs.nvidia.com/deeplearning/sdk/nccl-developer-guide/docs/env.html)


## 基础知识 [¶](#basics "此标题的永久链接")


 torch.distributed 包为在一台或多台机器上运行的多个计算节点之间的多进程并行性提供了 PyTorch 支持和通信原语。类 [`torch.nn.parallel.DistributedDataParallel()`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 在此功能的基础上构建以提供同步分布式训练作为任何 PyTorch 模型的包装器。这与 [Multiprocessing 包 
- torch.multiprocessing](multiprocessing.html) 和 [`torch.nn.DataParallel()`](generated/torch.nn.DataParallel.html#torch.nn.DataParallel) 提供的并行类型不同“torch.nn.DataParallel”)，因为它支持多个网络连接的机器，并且用户必须为每个进程显式启动主训练脚本的单独副本。


 单机同步情况下，torch.distributed 或 [`torch.nn.parallel.DistributedDataParallel()`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 包装器可能仍然比其他数据并行方法具有优势，包括 [`torch.nn.DataParallel()`](generated/torch.nn.DataParallel.html#torch.nn.DataParallel "torch.nn.DataParallel") :



* 每个进程都维护自己的优化器，并在每次迭代时执行完整的优化步骤。虽然这可能看起来多余，但由于梯度已经被收集在一起并在进程之间求平均值，因此每个进程都是相同的，这意味着不需要参数广播步骤，从而减少了在节点之间传输tensor所花费的时间。*每个进程都包含一个独立的 Python 解释器，消除了从单个 Python 进程驱动多个执行线程、模型副本或 GPU 带来的额外解释器开销和“GIL 抖动”。这对于大量使用 Python 运行时的模型尤其重要，包括具有循环层或许多小组件的模型。


## 初始化 [¶](#initialization "此标题的永久链接")


 在调用任何其他方法之前，需要使用 [`torch.distributed.init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group") 函数初始化该包。这会阻塞，直到所有进程都加入。


 火炬分布式。


 可用


 ( ) [[source]](_modules/torch/distributed.html#is_available)[¶](#torch.distributed.is_available "此定义的永久链接")


 如果分布式包可用，则返回“True”。否则，“torch.distributed”不会公开任何其他 API。目前，“torch.distributed”可在 Linux、MacOS 和 Windows 上使用。设置“USE_DISTRIBUTED=1”以在从源构建 PyTorch 时启用它。目前，对于 Linux 和 Windows，默认值为“USE_DISTRIBUTED=1”，对于 MacOS，默认值为“USE_DISTRIBUTED=0”。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 火炬分布式。


 初始化_进程_组


 (*后端



 =
 


 无
* , *init_method



 =
 


 无
* , *超时



 =
 


 datetime.timedelta(秒=1800)
* , *world_size



 =
 


 -1
* , *排名



 =
 


 -1
* , *存储



 =
 


 无
* , *组_name



 =
 


 ''
* , *pg_选项



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#init_process_group)[¶](#torch.distributed.init_process_group "此定义的永久链接")


 初始化默认的分布式进程组，这也将初始化分布式包。


 初始化进程组有两种主要方法： 1. 显式指定 `store` 、 `rank` 和 `world_size` 。 2. 初始化进程组。指定“init_method”(一个 URL 字符串)，指示在何处/如何发现对等点。可以选择指定“rank”和“world_size”，或者对 URL 中的所有必需参数进行编码并省略它们。


 如果两者均未指定，则假定 `init_method` 为“env://”。


 参数 
* **backend** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*或
* [*Backend
* ](#torch.distributed.Backend "torch.distributed.Backend")*,
* *可选
* ) – 要使用的后端。根据构建时配置，有效值包括 `mpi` 、 `gloo` 、 `nccl` 和 `ucc` 。如果未提供后端，则将创建“gloo”和“nccl”后端，请参阅下面的注释了解如何管理多个后端。该字段可以作为小写字符串给出(例如， `"gloo"` )，也可以通过 [`Backend`](#torch.distributed.Backend "torch.distributed.Backend") 属性(例如，`后端.GLOO`)。如果每台机器使用带有“nccl”后端的多个进程，则每个进程必须对其使用的每个 GPU 具有独占访问权限，因为进程之间共享 GPU 可能会导致死锁。 `ucc` 后端是实验性的。
* **init_method** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") *,
* *可选
* ) – 指定如何初始化进程组的 URL。如果未指定 `init_method` 或 `store`，则默认为“env://”。与 `store` 互斥。
* **world_size** ( [*int*](https://docs. python.org/3/library/functions.html#int "(Python v3.12)")*,
* *可选
* ) – 参与作业的进程数。如果指定了 `store`，则为必需。
* **rank** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") *,
* *可选
* ) – 当前进程的排名(应该是 0 到 `world_size` -1 之间的数字)。如果指定了 `store`，则为必需。
* **store** ( [*Store*] (#torch.distributed.Store "torch.distributed.Store")*,
* *可选
* ) – 所有工作人员均可访问的键/值存储，用于交换连接/地址信息。与 `init_method` 互斥。
* 
* *timeout** ( *timedelta
* *,
* *可选
* ) – 针对进程组执行的操作超时。默认值等于 30 分钟。这适用于 `gloo` 后端。对于“nccl”，仅当环境变量“NCCL_BLOCKING_WAIT”或“NCCL_ASYNC_ERROR_HANDLING”设置为 1 时才适用。当设置“NCCL_BLOCKING_WAIT”时，这是持续时间该进程将阻塞并等待集合完成，然后抛出异常。当设置“NCCL_ASYNC_ERROR_HANDLING”时，这是集合将异步中止并且进程将崩溃的持续时间。 `NCCL_BLOCKING_WAIT` 将为用户提供可以捕获和处理的错误，但由于其阻塞性质，它会产生性能开销。另一方面，“NCCL_ASYNC_ERROR_HANDLING”的性能开销非常小，但会因错误而导致进程崩溃。这样做是因为 CUDA 执行是异步的，并且继续执行用户代码不再安全，因为失败的异步 NCCL 操作可能会导致后续 CUDA 操作在损坏的数据上运行。只需设置这两个环境变量之一。对于 `ucc` ，支持类似于 NCCL 的阻塞等待。然而，异步错误处理的方式有所不同，因为使用 UCC 我们有进度线程而不是看门狗线程。
* **group_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*,
* *可选
* *,
* *已弃用
* ) – 组名称。该参数被忽略
* **pg_options** ( *ProcessGroupOptions
* *,
* *可选
* ) – 进程组选项，指定在构建特定进程组期间需要传入哪些附加选项。截至目前，我们支持的唯一选项是“nccl”后端的“ProcessGroupNCCL.Options”，可以指定“is_high_priority_stream”，以便 nccl 后端在有计算内核等待时可以拾取高优先级 cuda 流。



!!! note "笔记"

    要启用 `backend == Backend.MPI` ，需要在支持 MPI 的系统上从源代码构建 PyTorch。


!!! note "笔记"

    对多个后端的支持是实验性的。目前，当未指定后端时，将创建 `gloo` 和 `nccl` 后端。 `gloo` 后端将用于具有 CPU tensor的集合，而 `nccl` 后端将用于具有 CUDA tensor的集合。可以通过传递格式为“<device_type>:<backend_name>,<device_type>:<backend_name>”的字符串来指定自定义后端，例如“cpu:gloo,cuda:custom_backend” 。


 火炬分布式。


 已初始化


 ( ) [[source]](_modules/torch/distributed/distributed_c10d.html#is_initialized)[¶](#torch.distributed.is_initialized "此定义的永久链接")


 检查默认进程组是否已初始化


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 火炬分布式。


 可用_mpi_


 ( ) [[source]](_modules/torch/distributed/distributed_c10d.html#is_mpi_available)[¶](#torch.distributed.is_mpi_available "此定义的永久链接")


 检查 MPI 后端是否可用。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 火炬分布式。


 可用_nccl_


 ( ) [[source]](_modules/torch/distributed/distributed_c10d.html#is_nccl_available)[¶](#torch.distributed.is_nccl_available "此定义的永久链接")


 检查 NCCL 后端是否可用。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 火炬分布式。


 可用


 ( ) [[source]](_modules/torch/distributed/distributed_c10d.html#is_gloo_available)[¶](#torch.distributed.is_gloo_available "此定义的永久链接")


 检查 Gloo 后端是否可用。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 火炬分布式。


 已_torchelastic_发射


 ( ) [[source]](_modules/torch/distributed/distributed_c10d.html#is_torchelastic_launched)[¶](#torch.distributed.is_torchelastic_launched "此定义的永久链接")


 检查此进程是否是使用“torch.distributed.elastic”(又名 torchelastic)启动的。 `TORCHELASTIC_RUN_ID` 环境变量的存在用作代理来确定当前进程是否是使用 torchelastic 启动的。这是一个合理的代理，因为“TORCHELASTIC_RUN_ID”映射到集合点 id，该集合点 id 始终为非空值，指示用于对等发现目的的作业 id。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)



---


 目前支持三种初始化方法：


### TCP 初始化 [¶](#tcp-initialization "永久链接到此标题")


 有两种使用 TCP 进行初始化的方法，都需要所有进程都可访问的网络地址和所需的 `world_size` 。第一种方法需要指定属于 0 级进程的地址。这种初始化方法要求所有进程都手动指定等级。


 请注意，最新的分布式软件包中不再支持多播地址。 `group_name` 也已被弃用。


```
import torch.distributed as dist

# Use address of one of the machines
dist.init_process_group(backend, init_method='tcp://10.1.1.20:23456',
                        rank=args.rank, world_size=4)

```


### 共享文件系统初始化 [¶](#shared-file-system-initialization "永久链接到此标题")


 另一种初始化方法利用一个组中所有机器共享且可见的文件系统，以及所需的“world_size”。 URL 应以“file://”开头，并包含共享文件系统上不存在的文件(在现有目录中)的路径。如果该文件不存在，文件系统初始化将自动创建该文件，但不会删除该文件。因此，您有责任确保在同一文件路径上的下一个 [`init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group") 调用之前清理该文件/姓名。


 请注意，最新的分布式软件包中不再支持自动排名分配，并且“group_name”也已弃用。


!!! warning "警告"

     此方法假设文件系统支持使用“fcntl”锁定 
- 大多数本地系统和 NFS 支持它。


!!! warning "警告"

     此方法将始终创建该文件，并尽力在程序结束时清理和删除该文件。换句话说，每次使用 file init 方法进行初始化都需要一个全新的空文件才能成功初始化。如果再次使用先前初始化使用的相同文件(碰巧没有被清理)，这是意外行为，并且通常会导致死锁和失败。因此，即使此方法会尽力清理文件，但如果自动删除不成功，您有责任确保在训练结束时删除该文件，以防止在训练期间再次重复使用同一文件。下一次。如果您计划对同一文件名多次调用 [`init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group")，这一点尤其重要。换句话说，如果文件是未删除/清理并且您再次对该文件调用 [`init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group") ，预计会失败。这里的经验法则是，确保每次调用 [`init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group") 时该文件不存在或为空。



```
import torch.distributed as dist

# rank should always be specified
dist.init_process_group(backend, init_method='file:///mnt/nfs/sharedfile',
                        world_size=4, rank=args.rank)

```


### 环境变量初始化 [¶](#environment-variable-initialization "永久链接到此标题")


 此方法将从环境变量中读取配置，允许用户完全自定义获取信息的方式。需要设置的变量有：



* `MASTER_PORT` 
- 必需；必须是等级 0
* `MASTER_ADDR` 机器上的空闲端口 
- 必需(等级 0 除外)；等级 0 节点的地址
* `WORLD_SIZE` 
- 必需；可以在此处设置，也可以在对 init 函数的调用中设置
* `RANK` 
- 必需；可以在这里设置，也可以在调用 init 函数时设置


 等级 0 的机器将用于建立所有连接。


 这是默认方法，意味着不必指定 `init_method` (或者可以是 `env://` )。


## 初始化后 [¶](#post-initialization "永久链接到此标题")


 一旦运行了 [`torch.distributed.init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group")，就可以使用以下函数。要检查进程组是否已初始化，请使用 [`torch.distributed.is_initialized()`](#torch.distributed.is_initialized "torch.distributed.is_initialized") 。


*班级*


 火炬分布式。


 后端


 ( *name
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#Backend)[¶](#torch.distributed.Backend "此定义的永久链接")


 类似枚举的可用后端：GLOO、NCCL、UCC、MPI 和其他注册后端。


 此类的值是小写字符串，例如 `"gloo"` 。它们可以作为属性访问，例如“Backend.NCCL”。


 可以直接调用该类来解析字符串，例如`Backend(backend_str)`会检查`backend_str`是否有效，如果有效则返回解析后的小写字符串。它还接受大写字符串，例如 `Backend("GLOO")` 返回 `"gloo"` 。




!!! note "笔记"

    条目“Backend.UNDEFINED”存在，但仅用作某些字段的初始值。用户不应直接使用它，也不应假设它的存在。


*类方法*


 注册_后端


 (*名称*，*功能*，*扩展_api



 =
 


 错误*、*设备



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#Backend.register_backend)[¶](#torch.distributed.Backend.register_backend "此定义的永久链接")


 使用给定名称和实例化函数注册一个新后端。


 第三方“ProcessGroup”扩展使用此类方法来注册新后端。


 参数 
* **name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – ` 的后端名称ProcessGroup` 扩展。它应该与 `init_process_group()` 中的匹配。
* **func** ( *function
* ) – 实例化后端的函数处理程序。该函数应在后端扩展中实现，并采用四个参数，包括 `store ` 、 `rank` 、 `world_size` 和 `timeout`.
* **extend_api** ( [*bool*](https://docs.python.org/3/library/functions.html# bool "(Python v3.12)")*,
* *可选
* ) – 后端是否支持扩展参数结构。默认: `False` 。如果设置为 `True` ，后端将获取 `c10d::DistributedBackendOptions` 的实例，以及后端实现定义的进程组选项对象。
* **device** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*或
* [*list*](https://docs.python.org/3/library/stdtypes.html #list "(在 Python v3.12 中)")*of
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)") *,
* *可选
* ) – 该后端支持的设备类型，例如“cpu”、“cuda”等。如果没有，则假设“cpu”和“cuda”



!!! note "笔记"

    对第 3 方后端的支持是实验性的，可能会发生变化。


 火炬分布式。


 获取_后端


 ( *团体



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#get_backend)[¶](#torch.distributed.get_backend "此定义的永久链接")


 返回给定进程组的后端。


 Parameters


**group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。默认是通用主进程组。如果指定了另一个特定组，则调用进程必须是“group”的一部分。


 退货


 给定进程组的后端作为小写字符串。


 Return type


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 火炬分布式。


 获得_排名


 ( *团体



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#get_rank)[¶](#torch.distributed.get_rank "此定义的永久链接")


 返回当前进程在所提供的“组”或默认组(如果未提供)中的排名。


 Rank 是分配给分布式进程组中每个进程的唯一标识符。它们始终是从 0 到 `world_size` 的连续整数。


 Parameters


**group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。


 退货


 进程 group-1 的等级(如果不是该组的一部分)


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 火炬分布式。


 获取世界大小


 ( *团体



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#get_world_size)[¶](#torch.distributed.get_world_size "此定义的永久链接")


 返回当前进程组中的进程数


 Parameters


**group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。


 退货


 进程 group-1 的世界大小(如果不是该组的一部分)


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)




---


## 分布式键值存储 [¶](#distributed-key-value-store "此标题的永久链接")


 分布式包附带一个分布式键值存储，可用于在组中的进程之间共享信息，以及在 [`torch.distributed.init_process_group()`](#torch. Distribution.init_process_group "torch.distributed.init_process_group")(通过显式创建存储作为指定 `init_method` 的替代方法。)键值存储有 3 种选择： [`TCPStore`](#torch.distributed.TCPStore " torch.distributed.TCPStore") 、 [`FileStore`](#torch.distributed.FileStore "torch.distributed.FileStore") 和 [`HashStore`](#torch.distributed.HashStore "torch.distributed.HashStore") 。


*班级*


 火炬分布式。


 存储[¶](#torch.distributed.Store"此定义的永久链接")


 所有存储实现的基类，例如 PyTorchdistributed 提供的 3 个： ( [`TCPStore`](#torch.distributed.TCPStore "torch.distributed.TCPStore") , [`FileStore`](#torch.distributed.FileStore " torch.distributed.FileStore") 和 [`HashStore`](#torch.distributed.HashStore "torch.distributed.HashStore") )。


*班级*


 火炬分布式。


 TCPStore [¶](#torch.distributed.TCPStore"此定义的永久链接")


 基于 TCP 的分布式键值存储实现。服务器存储保存数据，而客户端存储可以通过 TCP 连接到服务器存储并执行诸如“set()”插入键值对、“get()”检索键值对等操作。始终是初始化一个服务器存储，因为客户端存储将等待服务器建立连接。


 参数 
* **host_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 主机名或服务器存储应运行的 IP 地址。
* **端口** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)" ) ) – 服务器存储应侦听传入请求的端口。
* **world_size** ( [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)")*,
* *可选
* ) – 商店用户总数(客户端数量 
+ 服务器数量 1)。默认为 None(None 表示商店用户数量不固定)。
* **is_master** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)")*,
* *可选
* ) – 初始化服务器存储时为 True，对于客户端存储为 False。默认值为 False。
* **timeout** ( *timedelta
* *,
* *可选
* ) – 存储在初始化期间以及“get()”和“wait()”等方法使用的超时。默认值为 timedelta(seconds=300)
* **wait_for_worker** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.1 中) 12)")*,
* *可选
* ) – 是否等待所有工作人员连接到服务器存储。这仅适用于 world_size 为固定值的情况。默认值为 True。
* **multi_tenant** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*, 
* *可选
* ) – 如果为 True，当前进程中具有相同主机/端口的所有 `TCPStore` 实例将使用相同的底层 `TCPServer` 。默认值为 False。
* **master_listen_fd** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") *,
* *可选
* ) – 如果指定，底层 `TCPServer` 将侦听此文件描述符，该文件描述符必须是已绑定到 `port` 的套接字。在某些情况下有助于避免端口分配竞争。默认值为 None (意味着服务器创建一个新套接字并尝试将其绑定到 `port` )。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> # Run on process 1 (server)
>>> server_store = dist.TCPStore("127.0.0.1", 1234, 2, True, timedelta(seconds=30))
>>> # Run on process 2 (client)
>>> client_store = dist.TCPStore("127.0.0.1", 1234, 2, False)
>>> # Use any of the store methods from either the client or server after initialization
>>> server_store.set("first_key", "first_value")
>>> client_store.get("first_key")

```


*班级*


 火炬分布式。


 HashStore [¶](#torch.distributed.HashStore"此定义的永久链接")


 基于底层哈希图的线程安全存储实现。该存储可以在同一进程内使用(例如，由其他线程使用)，但不能跨进程使用。


 例子：：




```
>>> import torch.distributed as dist
>>> store = dist.HashStore()
>>> # store can be used from other threads
>>> # Use any of the store methods after initialization
>>> store.set("first_key", "first_value")

```


*班级*


 火炬分布式。


 FileStore [¶](#torch.distributed.FileStore"此定义的永久链接")


 使用文件来存储底层键值对的存储实现。


 参数 
* **file_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 路径用于存储键值对的文件
* **world_size** ( [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3.1 中) 12)")*,
* *可选
* ) – 使用存储的进程总数。默认值为-1(负值表示商店用户数量不固定)。


 例子：：




```
>>> import torch.distributed as dist
>>> store1 = dist.FileStore("/tmp/filestore", 2)
>>> store2 = dist.FileStore("/tmp/filestore", 2)
>>> # Use any of the store methods from either the client or server after initialization
>>> store1.set("first_key", "first_value")
>>> store2.get("first_key")

```


*班级*


 火炬分布式。


 PrefixStore [¶](#torch.distributed.PrefixStore "此定义的永久链接")


 围绕 3 个键值存储中任意一个的包装器( [`TCPStore`](#torch.distributed.TCPStore "torch.distributed.TCPStore") 、 [`FileStore`](#torch.distributed.FileStore "torch.distributed. FileStore") 和 [`HashStore`](#torch.distributed.HashStore "torch.distributed.HashStore") ) 为插入存储的每个键添加前缀。


 参数 
* **prefix** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 前缀字符串在插入存储之前添加到每个键的前面。
* **store** ( *torch.distributed.store
* ) – 形成底层键值存储的存储对象。


 火炬.分布式.商店。



 set
 


 ( *自己



 :
 


 火炬._C._distributed_c10d.Store
* , *arg0



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")
* , *arg1



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)*)


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.distributed.Store.set"此定义的永久链接")


 根据提供的“key”和“value”将键值对插入到存储中。如果存储中已经存在“key”，它将用新提供的“value”覆盖旧值。


 参数 
* **key** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要添加的密钥
* **value** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 值与要添加到商店的“key”关联。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> store = dist.TCPStore("127.0.0.1", 0, 1, True, timedelta(seconds=30))
>>> store.set("first_key", "first_value")
>>> # Should return "first_value"
>>> store.get("first_key")

```


 火炬.分布式.商店。



 get
 


 ( *自己



 :
 


 火炬._C._distributed_c10d.Store
* , *arg0



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)*)


 → [字节](https://docs.python.org/3/library/stdtypes.html#bytes“(Python v3.12)”)


[¶](#torch.distributed.Store.get"此定义的永久链接")


 检索与存储中给定“key”关联的值。如果存储中不存在“key”，该函数将等待“timeout”(在初始化存储时定义)，然后抛出异常。


 Parameters


**key** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 该函数将返回关联的值用这把钥匙。


 退货


 如果“key”在商店中，则与“key”关联的值。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> store = dist.TCPStore("127.0.0.1", 0, 1, True, timedelta(seconds=30))
>>> store.set("first_key", "first_value")
>>> # Should return "first_value"
>>> store.get("first_key")

```


 火炬.分布式.商店。



 add
 


 ( *自己



 :
 


 火炬._C._distributed_c10d.Store
* , *arg0



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")
* , *arg1



 :
 


[int](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)")
* )


 → [int](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)")


[¶](#torch.distributed.Store.add"此定义的永久链接")


 对给定“key”的第一次调用 add 会创建一个与存储中的“key”关联的计数器，并初始化为“amount”。后续使用相同的“key”调用 add 会将计数器增加指定的“amount”。使用已通过“set()”在存储中设置的密钥调用“add()”将导致异常。


 参数 
* **key** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 存储中的密钥其计数器将递增。
* **amount** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) –计数器将增加的数量。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> # Using TCPStore as an example, other store types can also be used
>>> store = dist.TCPStore("127.0.0.1", 0, 1, True, timedelta(seconds=30))
>>> store.add("first_key", 1)
>>> store.add("first_key", 6)
>>> # Should return 7
>>> store.get("first_key")

```


 火炬.分布式.商店。


 比较_set


 ( *自己



 :
 


 火炬._C._distributed_c10d.Store
* , *arg0



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")
* , *arg1



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")
* , *arg2



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)*)


 → [字节](https://docs.python.org/3/library/stdtypes.html#bytes“(Python v3.12)”)


[¶](#torch.distributed.Store.compare_set"此定义的永久链接")


 根据提供的“key”将键值对插入到存储中，并在插入之前对“expected_value”和“desired_value”进行比较。仅当“key”的“expected_value”已存在于存储中或者“expected_value”为空字符串时，才会设置“desired_value”。


 参数 
* **key** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要检查的键在商店中。
* **预期_值** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) –插入之前要检查的与 `key` 关联的值。
* **desired_value** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 与要添加到存储中的 `key` 关联的值。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> store = dist.TCPStore("127.0.0.1", 0, 1, True, timedelta(seconds=30))
>>> store.set("key", "first_value")
>>> store.compare_set("key", "first_value", "second_value")
>>> # Should return "second_value"
>>> store.get("key")

```


 火炬.分布式.商店。



 wait
 


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [¶](#torch.distributed.Store.wait "此定义的永久链接")


 重载功能。


1. wait(self: torch._C._distributed_c10d.Store, arg0: List[str]) -
> None


 等待“keys”中的每个密钥添加到存储中。如果在“timeout”(在存储初始化期间设置)之前未设置所有键，则“wait”将引发异常。


 Parameters


**keys** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)") ) – 等待的键列表直到它们在商店里设置完毕。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> # Using TCPStore as an example, other store types can also be used
>>> store = dist.TCPStore("127.0.0.1", 0, 1, True, timedelta(seconds=30))
>>> # This will throw an exception after 30 seconds
>>> store.wait(["bad_key"])

```


2. wait(self: torch._C._distributed_c10d.Store, arg0: List[str], arg1: datetime.timedelta) -
> None


 等待“keys”中的每个键添加到存储中，如果提供的“timeout”尚未设置键，则抛出异常。


 参数 
* **keys** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)") ) – 其上的键列表等待它们在存储中设置。
* **timeout** ( *timedelta
* ) – 在抛出异常之前等待添加键的时间。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> # Using TCPStore as an example, other store types can also be used
>>> store = dist.TCPStore("127.0.0.1", 0, 1, True, timedelta(seconds=30))
>>> # This will throw an exception after 10 seconds
>>> store.wait(["bad_key"], timedelta(seconds=10))

```


 火炬.分布式.商店。


 数量_键


 ( *自己



 :
 


 火炬._C._distributed_c10d.Store
* )


 → [int](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)")


[¶](#torch.distributed.Store.num_keys"此定义的永久链接")


 返回商店中设置的密钥数量。请注意，该数字通常比“set()”和“add()”添加的键数大一，因为一个键用于协调使用商店的所有工作人员。


!!! warning "警告"

     当与 [`TCPStore`](#torch.distributed.TCPStore "torch.distributed.TCPStore") 一起使用时， `num_keys` 返回写入底层文件的键的数量。如果存储被破坏并使用同一文件创建另一个存储，则原始密钥将被保留。


 退货


 商店中存在的钥匙数量。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> # Using TCPStore as an example, other store types can also be used
>>> store = dist.TCPStore("127.0.0.1", 0, 1, True, timedelta(seconds=30))
>>> store.set("first_key", "first_value")
>>> # This should return 2
>>> store.num_keys()

```


 火炬.分布式.商店。


 删除_key


 ( *自己



 :
 


 火炬._C._distributed_c10d.Store
* , *arg0



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)*)


 → [bool](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)")


[¶](#torch.distributed.Store.delete_key "此定义的永久链接")


 从存储中删除与“key”关联的键值对。如果键已成功删除，则返回 true，否则返回 false。


!!! warning "警告"

     `delete_key` API 仅受 [`TCPStore`](#torch.distributed.TCPStore "torch.distributed.TCPStore") 和 [`HashStore`](#torch.distributed.HashStore "torch.distributed.哈希存储”)。将此 API 与 [`FileStore`](#torch.distributed.FileStore "torch.distributed.FileStore") 一起使用将导致异常。


 Parameters


**key** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要从店铺


 退货


 如果删除了 `key` 则为 True ，否则为 False 。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> # Using TCPStore as an example, HashStore can also be used
>>> store = dist.TCPStore("127.0.0.1", 0, 1, True, timedelta(seconds=30))
>>> store.set("first_key")
>>> # This should return true
>>> store.delete_key("first_key")
>>> # This should return false
>>> store.delete_key("bad_key")

```


 火炬.分布式.商店。


 设置_超时


 ( *自己



 :
 


 火炬._C._distributed_c10d.Store
* , *arg0



 :
 


[datetime.timedelta](https://docs.python.org/3/library/datetime.html#datetime.timedelta“(在Python v3.12中)”)*)


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.distributed.Store.set_timeout"此定义的永久链接")


 设置商店的默认超时。此超时在初始化期间以及“wait()”和“get()”中使用。


 Parameters


**timeout** ( *timedelta
* ) – 在商店中设置的超时。


 例子：：




```
>>> import torch.distributed as dist
>>> from datetime import timedelta
>>> # Using TCPStore as an example, other store types can also be used
>>> store = dist.TCPStore("127.0.0.1", 0, 1, True, timedelta(seconds=30))
>>> store.set_timeout(timedelta(seconds=10))
>>> # This will throw an exception after 10 seconds
>>> store.wait(["bad_key"])

```


## 组 [¶](#groups "此标题的永久链接")


 默认情况下，集体在默认组(也称为世界)上运行，并要求所有进程进入分布式函数调用。然而，某些工作负载可以受益于更细粒度的通信。这就是分布式团体发挥作用的地方。 [`new_group()`](#torch.distributed.new_group "torch.distributed.new_group") 函数可用于创建新组，其中包含所有进程的任意子集。它返回一个不透明的组句柄，可以作为所有集体的“组”参数给出(集体是分布式函数，用于以某些众所周知的编程模式交换信息)。


 火炬分布式。


 新_组


 (*排名



 =
 


 无
* , *超时



 =
 


 datetime.timedelta(秒=1800)
* , *后端



 =
 


 无
* , *pg_options



 =
 


 无
* , *使用_本地_同步



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#new_group)[¶](#torch.distributed.new_group "此定义的永久链接")


 创建一个新的分布式组。


 此功能要求主组中的所有进程(即属于分布式作业的所有进程)进入此功能，即使它们不会成为该组的成员。此外，应在所有进程中以相同的顺序创建组。


!!! warning "警告"

     同时使用多个进程组与“NCCL”后端是不安全的，用户应该在其应用程序中执行显式同步，以确保一次仅使用一个进程组。这意味着来自一个进程组的集合应该在设备上完成执行(而不仅仅是自 CUDA 执行 isasync 起入队)，然后再将来自另一个进程组的集合入队。请参阅[同时使用多个 NCCL 通信器](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/communicators.html#使用多个 nccl-communicators-concurrently)了解更多详细信息。


 参数 
* **排名** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)")*[
* [*int
* ](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*]
* ) – 群组成员的等级列表。如果为“None”，则将设置为所有等级。默认值为“无”。
* **超时** ( *timedelta
* *,
* *可选
* ) – 针对进程组执行的操作的超时。默认值等于 30 分钟。这适用于 `gloo` 后端。对于“nccl”，仅当环境变量“NCCL_BLOCKING_WAIT”或“NCCL_ASYNC_ERROR_HANDLING”设置为 1 时才适用。当设置“NCCL_BLOCKING_WAIT”时，这是持续时间该进程将阻塞并等待集合完成，然后抛出异常。当设置“NCCL_ASYNC_ERROR_HANDLING”时，这是集合将异步中止并且进程将崩溃的持续时间。 `NCCL_BLOCKING_WAIT` 将为用户提供可以捕获和处理的错误，但由于其阻塞性质，它会产生性能开销。另一方面，“NCCL_ASYNC_ERROR_HANDLING”的性能开销非常小，但会因错误而导致进程崩溃。这样做是因为 CUDA 执行是异步的，并且继续执行用户代码不再安全，因为失败的异步 NCCL 操作可能会导致后续 CUDA 操作在损坏的数据上运行。仅应设置这两个环境变量之一。
* **backend** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*或
* [*Backend*](#torch.distributed.Backend "torch.distributed.Backend")*,
* *可选
* ) – 要使用的后端。根据构建时配置，有效值为 `gloo` 和 `nccl` 。默认情况下使用与全局组相同的后端。该字段应以小写字符串形式给出(例如，`"gloo"`)，也可以通过 [`Backend`](#torch.distributed.Backend "torch.distributed.Backend") 属性(例如，`Backend.格洛`)。如果传入 None ，则使用默认进程组对应的后端。默认值为 `None`.
* **pg_options** ( *ProcessGroupOptions
* *,
* *可选
* ) – 进程组选项，指定在构建特定进程组期间需要传入哪些附加选项。即对于`nccl`后端，可以指定`is_high_priority_stream`，以便进程组可以拾取高优先级cuda流。
* **使用_local_synchronization** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 在进程组创建结束时执行 group-localbarrier。不同的是，非会员等级不需要调用API，也不加入屏障。


 退货


 分布式组的句柄，可以提供给集体调用，如果排名不是“ranks”的一部分，则为 None 。


 注意： use_local_synchronization 不适用于 MPI。


 注意：虽然 use_local_synchronization=True 对于较大的集群和小型进程组可以显着加快，但必须小心，因为它会改变集群行为，因为非成员等级不会加入组 Barrier()。


 注意：当每个级别创建多个重叠进程组时，use_local_synchronization=True 可能会导致死锁。为了避免这种情况，请确保所有排名都遵循相同的全局创建顺序。


 火炬分布式。


 获取_group_rank


 ( *group
* , *global_rank
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#get_group_rank)[¶](#torch.distributed.get_group_rank "此定义的永久链接")


 将全球排名转换为团体排名。


`global_rank` 必须是 `group` 的一部分，否则会引发 RuntimeError。


 参数 
* **group** ( *ProcessGroup
* ) – 用于查找相对排名的 ProcessGroup。
* **global_rank** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 要查询的全球排名。


 退货


 “global_rank”相对于“group”的组排名


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 注意：在默认进程组上调用此函数返回标识


 火炬分布式。


 获取_全球_排名


 ( *group
* , *group_rank
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#get_global_rank)[¶](#torch.distributed.get_global_rank "此定义的永久链接")


 将团体排名转化为全球排名。


`group_rank` 必须是组的一部分，否则会引发 RuntimeError。


 参数 
* **group** ( *ProcessGroup
* ) – 用于查找全局排名的 ProcessGroup。
* **group_rank** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 要查询的组排名。


 退货


 “group_rank”相对于“group”的全局排名


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 注意：在默认进程组上调用此函数返回标识


 火炬分布式。


 获取_process_group_ranks


 ( *group
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#get_process_group_ranks)[¶](#torch.distributed.get_process_group_ranks "此定义的永久链接")


 获取与“group”关联的所有排名。


 Parameters


**group** ( *ProcessGroup
* ) – 从中获取所有等级的 ProcessGroup。


 退货


 按组排名排序的全球排名列表。


## 点对点通信 [¶](#point-to-point-communication "固定链接到此标题")


 火炬分布式。



 send
 


 ( *tensor
* , *dst
* , *组



 =
 


 无
* , *标签



 =
 


 0
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#send)[¶](#torch.distributed.send "此定义的永久链接")


 同步发送tensor。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 要发送的tensor。
* **dst** ( [*int*](https://docs.python.org/3/library/functions.html#int "(Python v3.12)") ) – 目标排名。目标等级不应相同
* **进程。** ( *作为
* *当前
* 的等级
* ) –
* **组** ( *ProcessGroup
* *,
* *可选
* ) – 进程组继续努力。如果没有，将使用默认进程组。
* **tag** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12 )")*,
* *可选
* ) – 用于将发送与远程接收相匹配的标记


 火炬分布式。



 recv
 


 (*tensor*，*src



 =
 


 无
* , *组



 =
 


 无
* , *标签



 =
 


 0
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#recv)[¶](#torch.distributed.recv "此定义的永久链接")


 同步接收tensor。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 用于填充接收到的数据的tensor。
* **src** ( [*int*](https ://docs.python.org/3/library/functions.html#int "(Python v3.12)")*,
* *可选
* ) – 源排名。如果未指定，将从任何进程接收。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **tag** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12 )")*,
* *可选
* ) – 用于将接收与远程发送相匹配的标记


 退货


 发件人排名为 1(如果不是该组的一部分)


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


[`isend()`](#torch.distributed.isend "torch.distributed.isend") 和 [`irecv()`](#torch.distributed.irecv "torch.distributed.irecv") 返回分布式请求对象用过的。一般来说，该对象的类型是未指定的，因为它们永远不应该手动创建，但它们保证支持两种方法：



* `is_completed()` 
- 如果操作已完成，则返回 True 
* `wait()` 
- 将阻塞进程，直到操作完成。 `is_completed()` 一旦返回就保证返回 True。


 火炬分布式。



 isend
 


 ( *tensor
* , *dst
* , *组



 =
 


 无
* , *标签



 =
 


 0
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#isend)[¶](#torch.distributed.isend "此定义的永久链接")


 异步发送tensor。


!!! warning "警告"

     在请求完成之前修改“tensor”会导致未定义的行为。


!!! warning "警告"

    NCCL 后端不支持“tag”。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 要发送的tensor。
* **dst** ( [*int*](https://docs.python.org/3/library/functions.html#int "(Python v3.12)") ) – 目标排名。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 进程小组进行工作。如果没有，将使用默认进程组。
* **tag** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12 )")*,
* *可选
* ) – 用于将发送与远程接收相匹配的标记


 退货


 分布式请求对象。如果不是组的一部分，则无


 Return type


*工作*


 火炬分布式。



 irecv
 


 (*tensor*，*src



 =
 


 无
* , *组



 =
 


 无
* , *标签



 =
 


 0
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#irecv)[¶](#torch.distributed.irecv "此定义的永久链接")


 异步接收tensor。


!!! warning "警告"

    NCCL 后端不支持“tag”。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 用于填充接收到的数据的tensor。
* **src** ( [*int*](https ://docs.python.org/3/library/functions.html#int "(Python v3.12)")*,
* *可选
* ) – 源排名。如果未指定，将从任何进程接收。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **tag** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12 )")*,
* *可选
* ) – 用于将接收与远程发送相匹配的标记


 退货


 分布式请求对象。如果不是组的一部分，则无


 Return type


*工作*


 火炬分布式。


 批_isend_irecv


 ( *p2p_op_list
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#batch_isend_irecv)[¶](#torch.distributed.batch_isend_irecv "此定义的永久链接")


 异步发送或接收一批tensor并返回请求列表。


 处理`p2p_op_list`中的每个操作并返回相应的请求。目前支持 NCCL、Gloo 和 UCC 后端。


 Parameters


**p2p_op_list** – 点对点操作列表(每个操作符的类型为 `torch.distributed.P2POp` )。列表中 isend/irecv 的顺序很重要，它需要与远程端相应的 isend/irecv 匹配。


 退货


 调用op_list中对应op返回的分布式请求对象列表。


 例子


```
>>> send_tensor = torch.arange(2) + 2 * rank
>>> recv_tensor = torch.randn(2)
>>> send_op = dist.P2POp(dist.isend, send_tensor, (rank + 1)%world_size)
>>> recv_op = dist.P2POp(dist.irecv, recv_tensor, (rank - 1 + world_size)%world_size)
>>> reqs = batch_isend_irecv([send_op, recv_op])
>>> for req in reqs:
>>>     req.wait()
>>> recv_tensor
tensor([2, 3]) # Rank 0
tensor([0, 1]) # Rank 1

```


!!! note "笔记"

    请注意，当此 API 与 NCCL PG 后端一起使用时，用户必须使用 torch.cuda.set_device 设置当前 GPU 设备，否则会导致意外挂起问题。


 另外，如果该 API 是传递给 dist.P2POp 的 group 中的第一个集体调用，则该 group 的所有行列都必须参与该 API 调用；否则，行为是未定义的。如果此 API 调用不是“组”中的第一个集体调用，则允许仅涉及“组”的排名子集的批量 P2P 操作。


*班级*


 火炬分布式。



 P2POp
 


 (*操作*，*tensor*，*对等*，*组



 =
 


 无
* , *标签



 =
 


 0
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#P2POp)[¶](#torch.distributed.P2POp"此定义的永久链接")


 为 `batch_isend_irecv` 构建点对点操作的类。


 该类构建 P2P 操作的类型、通信缓冲区、对等等级、进程组和标记。此类的实例将被传递到“batch_isend_irecv”以进行点对点通信。


 参数 
* **op** ( *Callable
* ) – 向对等进程发送数据或从对等进程接收数据的函数。`op` 的类型是 `torch.distributed.isend` 或 `torch.distributed.irecv`.
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 要发送或接收的tensor。
* **peer** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 目标或源排名。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的流程组。如果没有，将使用默认进程组。
* **tag** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12 )")*,
* *可选
* ) – 用于将发送与接收相匹配的标记。


## 同步和异步集体操作 [¶](#synchronous-and-asynchronous-collective-operations "永久链接到此标题")


 每个集合操作函数都支持以下两种操作，具体取决于传递到集合中的“async_op”标志的设置：


**同步操作** 
- 默认模式，当`async_op`设置为`False`时。当函数返回时，保证执行集体操作。在 CUDA 操作的情况下，不能保证 CUDA 操作完成，因为 CUDA 操作是异步的。对于 CPU 集合，利用集合调用的输出的任何进一步的函数调用都将按预期运行。对于 CUDA 集合，利用同一 CUDA 流上的输出的函数调用将按预期运行。用户在不同流下运行的场景下必须注意同步。有关 CUDA 语义(例如流同步)的详细信息，请参阅 [CUDA 语义](https://pytorch.org/docs/stable/notes/cuda.html)。请参阅下面的脚本以查看 CPU 和 CUDA 的这些语义差异的示例运营。


**异步操作** 
- 当 `async_op` 设置为 True 时。集合操作函数返回一个分布式请求对象。一般来说，不需要手动创建，并且保证支持两种方法：



* `is_completed()` 
- 对于 CPU 集合，如果完成则返回 `True`。在 CUDA 操作的情况下，如果操作已成功排队到 CUDA 流上，并且输出可以在默认流上使用而无需进一步同步，则返回“True”。*“wait()” 
- 在 CPU 集体的情况下，将阻塞进程直到操作完成。在 CUDA 集合的情况下，将阻塞，直到操作成功排队到 CUDA 流上，并且可以在默认流上使用输出而无需进一步同步。
* `get_future()` 
- 返回 `torch._C.Future`目的。支持 NCCL，还支持 GLOO 和 MPI 上的大多数操作(点对点操作除外)。注意：随着我们继续采用 Futures 和合并 API，`get_future()` 调用可能会变得多余。


**例子**


 以下代码可以作为使用分布式集合时 CUDA 操作语义的参考。它显示了在不同 CUDA 流上使用集合输出时明确需要同步：


```
# Code runs on each rank.
dist.init_process_group("nccl", rank=rank, world_size=2)
output = torch.tensor([rank]).cuda(rank)
s = torch.cuda.Stream()
handle = dist.all_reduce(output, async_op=True)
# Wait ensures the operation is enqueued, but not necessarily complete.
handle.wait()
# Using result on non-default stream.
with torch.cuda.stream(s):
    s.wait_stream(torch.cuda.default_stream())
    output.add_(100)
if rank == 0:
    # if the explicit call to wait_stream was omitted, the output below will be
    # non-deterministically 1 or 101, depending on whether the allreduce overwrote
    # the value after the add completed.
    print(output)

```


## 集体函数 [¶](#collective-functions "此标题的固定链接")


 火炬分布式。


 播送


 (*tensor*，*src*，*组



 =
 


 无
* , *异步_op



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#broadcast)[¶](#torch.distributed.broadcast "此定义的永久链接")


 将tensor广播给整个组。


“tensor”在参与集体的所有进程中必须具有相同数量的元素。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 如果 `src` 是当前进程的排名，则要发送的数据，以及用于保存的tensor否则收到数据。
* **src** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 源排名.
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 火炬分布式。


 广播_object_list


 ( *object_list
* , *src



 =
 


 0
* , *组



 =
 


 无
* , *设备



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#broadcast_object_list)[¶](#torch.distributed.broadcast_object_list "此定义的永久链接")


 将 `object_list` 中的可挑选对象广播给整个组。与 [`broadcast()`](#torch.distributed.broadcast "torch.distributed.broadcast") 类似，但可以传入 Python 对象。请注意，`object_list` 中的所有对象都必须是可挑选的才能进行广播。


 参数 
* **object_list** ( *List
* *[
* *Any
* *]
* ) – 要广播的输入对象列表。每个对象都必须是可挑选的。只有“src”等级上的对象才会被广播，但每个等级必须提供相同大小的列表。
* **src** ( [*int*](https://docs.python.org/3/library/functions. html#int "(in Python v3.12)") ) – 从中广播 `object_list` 的源级别。
* **group** – (ProcessGroup，可选)：要处理的进程组。如果没有，将使用默认进程组。默认值为 `None`.
* **device** ( `torch.device` ，可选) – 如果不是 None，对象将被序列化并转换为tensor，并在广播之前移动到 `device`。默认为“无”。


 退货


‘无’。如果排名是组的一部分，则“object_list”将包含来自“src”排名的广播对象。



!!! note "笔记"

    对于基于 NCCL 的进程组，对象的内部tensor表示必须在通信发生之前移动到 GPU 设备。在这种情况下，使用的设备由“torch.cuda.current_device()”给出，用户有责任通过“torch.cuda.set_device”确保设置为每个等级都有一个单独的 GPU ()`。


!!! note "笔记"

    请注意，此 API 与 [`all_gather()`](#torch.distributed.all_gather "torch.distributed.all_gather") 集体略有不同，因为它不提供 `async_op` 句柄，因此将是一个阻塞称呼。


!!! warning "警告"

    [`broadcast_object_list()`](#torch.distributed.broadcast_object_list "torch.distributed.broadcast_object_list") 隐式使用 `pickle` 模块，已知这是不安全的。可以构建恶意 pickledata，在 unpickle 过程中执行任意代码。仅使用您信任的数据调用此函数。


!!! warning "警告"

     使用 GPU tensor调用 [`broadcast_object_list()`](#torch.distributed.broadcast_object_list "torch.distributed.broadcast_object_list") 没有得到很好的支持，并且效率低下，因为它会导致 GPU -
> CPU 传输，因为tensor会被腌制。请考虑使用 [`broadcast()`](#torch.distributed.broadcast "torch.distributed.broadcast") 代替。


 例子：：




```
>>> # Note: Process group initialization omitted on each rank.
>>> import torch.distributed as dist
>>> if dist.get_rank() == 0:
>>>     # Assumes world_size of 3.
>>>     objects = ["foo", 12, {1: 2}] # any picklable object
>>> else:
>>>     objects = [None, None, None]
>>> # Assumes backend is not NCCL
>>> device = torch.device("cpu")
>>> dist.broadcast_object_list(objects, src=0, device=device)
>>> objects
['foo', 12, {1: 2}]

```


 火炬分布式。


 全部_reduce


 ( *tensor
* , *op=<RedOpType.SUM: 0>
* , *group=None
* , *async_op=False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#all_reduce)[ ¶](#torch.distributed.all_reduce "此定义的永久链接")


 以所有机器都得到最终结果的方式减少所有机器上的tensor数据。


 调用“tensor”之后，所有进程中的“tensor”将按位相同。


 支持复杂tensor。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 集体的输入和输出。该函数就地操作。
* **op** ( *可选
* ) – `torch.distributed.ReduceOp` 枚举中的值之一。指定用于逐元素缩减的操作。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 例子


```
>>> # All tensors below are of torch.int64 type.
>>> # We have 2 process groups, 2 ranks.
>>> tensor = torch.arange(2, dtype=torch.int64) + 1 + 2 * rank
>>> tensor
tensor([1, 2]) # Rank 0
tensor([3, 4]) # Rank 1
>>> dist.all_reduce(tensor, op=ReduceOp.SUM)
>>> tensor
tensor([4, 6]) # Rank 0
tensor([4, 6]) # Rank 1

```



```
>>> # All tensors below are of torch.cfloat type.
>>> # We have 2 process groups, 2 ranks.
>>> tensor = torch.tensor([1+1j, 2+2j], dtype=torch.cfloat) + 2 * rank * (1+1j)
>>> tensor
tensor([1.+1.j, 2.+2.j]) # Rank 0
tensor([3.+3.j, 4.+4.j]) # Rank 1
>>> dist.all_reduce(tensor, op=ReduceOp.SUM)
>>> tensor
tensor([4.+4.j, 6.+6.j]) # Rank 0
tensor([4.+4.j, 6.+6.j]) # Rank 1

```


 火炬分布式。


 减少


 ( *tensor
* , *dst
* , *op=<RedOpType.SUM: 0>
* , *group=None
* , *async_op=False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html #reduce)[¶](#torch.distributed.reduce "此定义的永久链接")


 减少所有机器上的tensor数据。


 只有排名为“dst”的进程才会收到最终结果。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 集体的输入和输出。该函数就地运行。
* **dst** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) –目标排名
* **op** ( *可选
* ) – `torch.distributed.ReduceOp` 枚举中的值之一。指定用于逐元素缩减的操作。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 火炬分布式。


 全部_聚集


 ( *tensor_list
* , *tensor
* , *组



 =
 


 无
* , *异步_op



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#all_gather)[¶](#torch.distributed.all_gather "此定义的永久链接")


 将整个组的tensor收集到列表中。


 支持复杂tensor。


 参数 
* **tensor_list** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)")*[
* [
* Tensor*](tensors.html#torch.Tensor "torch.Tensor")*]
* ) – 输出列表。它应该包含用于集体输出的正确大小的tensor。
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 要从当前进程广播的tensor.
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 例子


```
>>> # All tensors below are of torch.int64 dtype.
>>> # We have 2 process groups, 2 ranks.
>>> tensor_list = [torch.zeros(2, dtype=torch.int64) for _ in range(2)]
>>> tensor_list
[tensor([0, 0]), tensor([0, 0])] # Rank 0 and 1
>>> tensor = torch.arange(2, dtype=torch.int64) + 1 + 2 * rank
>>> tensor
tensor([1, 2]) # Rank 0
tensor([3, 4]) # Rank 1
>>> dist.all_gather(tensor_list, tensor)
>>> tensor_list
[tensor([1, 2]), tensor([3, 4])] # Rank 0
[tensor([1, 2]), tensor([3, 4])] # Rank 1

```



```
>>> # All tensors below are of torch.cfloat dtype.
>>> # We have 2 process groups, 2 ranks.
>>> tensor_list = [torch.zeros(2, dtype=torch.cfloat) for _ in range(2)]
>>> tensor_list
[tensor([0.+0.j, 0.+0.j]), tensor([0.+0.j, 0.+0.j])] # Rank 0 and 1
>>> tensor = torch.tensor([1+1j, 2+2j], dtype=torch.cfloat) + 2 * rank * (1+1j)
>>> tensor
tensor([1.+1.j, 2.+2.j]) # Rank 0
tensor([3.+3.j, 4.+4.j]) # Rank 1
>>> dist.all_gather(tensor_list, tensor)
>>> tensor_list
[tensor([1.+1.j, 2.+2.j]), tensor([3.+3.j, 4.+4.j])] # Rank 0
[tensor([1.+1.j, 2.+2.j]), tensor([3.+3.j, 4.+4.j])] # Rank 1

```


 火炬分布式。


 全部聚集到tensor中


 ( *输出_tensor
* , *输入_tensor
* , *组



 =
 


 无
* , *异步_op



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#all_gather_into_tensor)[¶](#torch.distributed.all_gather_into_tensor "此定义的永久链接")


 收集所有级别的tensor并将它们放入单个输出tensor中。


 参数 
* **output_tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输出tensor以容纳所有等级的tensor元素。它的大小必须正确，才能具有以下形式之一：(i)所有输入tensor沿主维度的串联；有关“串联”的定义，请参阅“torch.cat()”；(ii) 沿主维度的所有输入tensor的堆栈；有关“堆栈”的定义，请参阅“torch.stack()”。下面的示例可以更好地解释支持的输出形式。
* **input_tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 从当前等级收集的tensor。与 `all 不同_gather` API，此 API 中的输入tensor在所有级别上必须具有相同的大小。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 例子


```
>>> # All tensors below are of torch.int64 dtype and on CUDA devices.
>>> # We have two ranks.
>>> device = torch.device(f'cuda:{rank}')
>>> tensor_in = torch.arange(2, dtype=torch.int64, device=device) + 1 + 2 * rank
>>> tensor_in
tensor([1, 2], device='cuda:0') # Rank 0
tensor([3, 4], device='cuda:1') # Rank 1
>>> # Output in concatenation form
>>> tensor_out = torch.zeros(world_size * 2, dtype=torch.int64, device=device)
>>> dist.all_gather_into_tensor(tensor_out, tensor_in)
>>> tensor_out
tensor([1, 2, 3, 4], device='cuda:0') # Rank 0
tensor([1, 2, 3, 4], device='cuda:1') # Rank 1
>>> # Output in stack form
>>> tensor_out2 = torch.zeros(world_size, 2, dtype=torch.int64, device=device)
>>> dist.all_gather_into_tensor(tensor_out2, tensor_in)
>>> tensor_out2
tensor([[1, 2],
 [3, 4]], device='cuda:0') # Rank 0
tensor([[1, 2],
 [3, 4]], device='cuda:1') # Rank 1

```


!!! warning "警告"

     Gloo 后端不支持此 API。


 火炬分布式。


 所有_聚集_对象


 ( *object_list
* , *obj
* , *group



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#all_gather_object)[¶](#torch.distributed.all_gather_object "此定义的永久链接")


 将整个组中的可腌制对象收集到一个列表中。与 [`all_gather()`](#torch.distributed.all_gather "torch.distributed.all_gather") 类似，但可以传入 Python 对象。请注意，该对象必须是可picklable 的才能被收集。


 参数 
* **object_list** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)")*[
* *任何
* *]
* ) – 输出列表。它的大小应正确设置为该集合的组的大小，并将包含输出。
* **obj** ( *Any
* ) – 要从当前进程广播的可选取 Python 对象。
* **group** ( *ProcessGroup 
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。默认为“无”。


 退货


 没有任何。如果调用rank是该组的一部分，则该集合的输出将被填充到输入“object_list”中。如果调用的rank不是该组的一部分，则传入的“object_list”将保持不变。



!!! note "笔记"

    请注意，此 API 与 [`all_gather()`](#torch.distributed.all_gather "torch.distributed.all_gather") 集体略有不同，因为它不提供 `async_op` 句柄，因此将是一个阻塞称呼。


!!! note "笔记"

    对于基于 NCCL 的处理组，对象的内部tensor表示必须在通信发生之前移动到 GPU 设备。在这种情况下，使用的设备由 `torch.cuda.current_device()` 给出，用户有责任确保通过 `torch.cuda.set_device 进行设置，以便每个等级都有一个单独的 GPU ()`。


!!! warning "警告"

    [`all_gather_object()`](#torch.distributed.all_gather_object "torch.distributed.all_gather_object") 隐式使用 `pickle` 模块，已知这是不安全的。可以构造恶意的 pickle 数据，该数据将在 unpickling 期间执行任意代码。仅使用您信任的数据调用此函数。


!!! warning "警告"

     使用 GPU tensor调用 [`all_gather_object()`](#torch.distributed.all_gather_object "torch.distributed.all_gather_object") 没有得到很好的支持，而且效率很低，因为它会导致 GPU -
> CPU 传输，因为tensor会被腌制。请考虑使用 [`all_gather()`](#torch.distributed.all_gather "torch.distributed.all_gather") 代替。


 例子：：




```
>>> # Note: Process group initialization omitted on each rank.
>>> import torch.distributed as dist
>>> # Assumes world_size of 3.
>>> gather_objects = ["foo", 12, {1: 2}] # any picklable object
>>> output = [None for _ in gather_objects]
>>> dist.all_gather_object(output, gather_objects[dist.get_rank()])
>>> output
['foo', 12, {1: 2}]

```


 火炬分布式。


 收集


 ( *tensor
* , *收集_list



 =
 


 无
* , *dst



 =
 


 0
* , *组



 =
 


 无
* , *异步_op



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#gather)[¶](#torch.distributed.gather "此定义的永久链接")


 在单个进程中收集tensor列表。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。
* **gather_list** ( [*list*](https:///docs.python.org/3/library/stdtypes.html#list "(Python v3.12)")*[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*] 
* *,
* *可选
* ) – 用于收集数据的适当大小的tensor列表(默认为无，必须在目标排名上指定)
* **dst** ( [*int*](https://docs. python.org/3/library/functions.html#int "(在 Python v3.12 中)")*,
* *可选
* ) – 目标排名(默认为 0)
* **group** ( *ProcessGroup
* *, 
* *可选
* ) – 要处理的流程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 火炬分布式。


 收集_object


 ( *obj
* , *object_gather_list



 =
 


 无
* , *dst



 =
 


 0
* , *组



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#gather_object)[¶](#torch.distributed.gather_object "此定义的永久链接")


 在单个进程中收集整个组中的可picklable对象。类似于 [`gather()`](#torch.distributed.gather "torch.distributed.gather") ，但可以传入Python对象。注意，该对象必须是可腌制以便收集。


 参数 
* **obj** ( *Any
* ) – 输入对象。必须是可挑选的。
* **object_gather_list** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)") *[
* *任意
* *]
* ) – 输出列表。在“dst”等级上，它的大小应该与该集合的组的大小正确，并且将包含输出。非 dstrank 上必须为“None”。 (默认为 `None` )
* **dst** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")
* ,
* *可选
* ) – 目的地排名。 (默认值为 0)
* **group** –(ProcessGroup，可选)：要处理的进程组。如果没有，将使用默认进程组。默认为“无”。


 退货


 没有任何。在“dst”等级上，“object_gather_list”将包含集体的输出。



!!! note "笔记"

    请注意，此 API 与收集集合体略有不同，因为它不提供 async_op 句柄，因此将是阻塞调用。


!!! note "笔记"

    对于基于 NCCL 的处理组，对象的内部tensor表示必须在通信发生之前移动到 GPU 设备。在这种情况下，使用的设备由 `torch.cuda.current_device()` 给出，用户有责任确保通过 `torch.cuda.set_device 进行设置，以便每个等级都有一个单独的 GPU ()`。


!!! warning "警告"

    [`gather_object()`](#torch.distributed.gather_object "torch.distributed.gather_object") 隐式使用 `pickle` 模块，已知这是不安全的。可以构造恶意的 pickle 数据，该数据将在 unpickling 期间执行任意代码。仅使用您信任的数据调用此函数。


!!! warning "警告"

     使用 GPU tensor调用 [`gather_object()`](#torch.distributed.gather_object "torch.distributed.gather_object") 没有得到很好的支持，并且效率低下，因为它会导致 GPU -
> CPU 传输，因为tensor会被腌制。请考虑使用 [`gather()`](#torch.distributed.gather "torch.distributed.gather") 代替。


 例子：：




```
>>> # Note: Process group initialization omitted on each rank.
>>> import torch.distributed as dist
>>> # Assumes world_size of 3.
>>> gather_objects = ["foo", 12, {1: 2}] # any picklable object
>>> output = [None for _ in gather_objects]
>>> dist.gather_object(
...     gather_objects[dist.get_rank()],
...     output if dist.get_rank() == 0 else None,
...     dst=0
... )
>>> # On rank 0
>>> output
['foo', 12, {1: 2}]

```


 火炬分布式。


 分散


 ( *tensor
* , *分散_list



 =
 


 无*，*src



 =
 


 0
* , *组



 =
 


 无
* , *异步_op



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#scatter)[¶](#torch.distributed.scatter "此定义的永久链接")


 将tensor列表分散到组中的所有进程。


 每个进程都会接收一个tensor并将其数据存储在“tensor”参数中。


 支持复杂tensor。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输出tensor。
* **scatter_list** ( [*list*](https:///docs.python.org/3/library/stdtypes.html#list "(Python v3.12)")*[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*] 
* ) – 要分散的tensor列表(默认为无，必须在源等级上指定)
* **src** ( [*int*](https://docs.python.org/3/library/functions.html #int "(Python v3.12)") ) – 源等级(默认为 0)
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分



!!! note "笔记"

    请注意，scatter_list 中的所有tensor必须具有相同的大小。


 例子：：




```
>>> # Note: Process group initialization omitted on each rank.
>>> import torch.distributed as dist
>>> tensor_size = 2
>>> t_ones = torch.ones(tensor_size)
>>> t_fives = torch.ones(tensor_size) * 5
>>> output_tensor = torch.zeros(tensor_size)
>>> if dist.get_rank() == 0:
>>>     # Assumes world_size of 2.
>>>     # Only tensors, all of which must be the same size.
>>>     scatter_list = [t_ones, t_fives]
>>> else:
>>>     scatter_list = None
>>> dist.scatter(output_tensor, scatter_list, src=0)
>>> # Rank i gets scatter_list[i]. For example, on rank 1:
>>> output_tensor
tensor([5., 5.])

```


 火炬分布式。


 分散_object_list


 ( *scatter_object_output_list
* , *scatter_object_input_list
* , *src



 =
 


 0
* , *组



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#scatter_object_list)[¶](#torch.distributed.scatter_object_list "此定义的永久链接")


 将 `scatter_object_input_list` 中的可pickl对象分散到整个组中。与 [`scatter()`](#torch.distributed.scatter "torch.distributed.scatter") 类似，但可以传入 Python 对象。每个rank，分散的对象将存储为 `scatter\ 的第一个元素_object_output_list` 。请注意，“scatter_object_input_list”中的所有对象都必须是可挑选的才能进行分散。


 参数 
* **scatter_object_output_list** ( *List
* *[
* *Any
* *]
* ) – 非空列表，其第一个元素将存储分散到该排名的对象。
* **scatter_object\ _input_list** ( *List
* *[
* *Any
* *]
* ) – 要分散的输入对象列表。每个对象都必须是可挑选的。只有 `src` 等级上的对象才会被分散，对于非 src 等级，参数可以为 `None`。
* **src** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 用于分散 `scatter_object_input_list` 的源排名.
* **group** – (ProcessGroup，可选)：进程小组进行工作。如果没有，将使用默认进程组。默认为“无”。


 退货


‘无’。如果排名是该组的一部分，则“scatter_object_output_list”将其第一个元素设置为该排名的分散对象。



!!! note "笔记"

    请注意，此 API 与分散集合体略有不同，因为它不提供 `async_op` 句柄，因此将是阻塞调用。


!!! warning "警告"

    [`scatter_object_list()`](#torch.distributed.scatter_object_list "torch.distributed.scatter_object_list") 隐式使用 `pickle` 模块，已知这是不安全的。可以构建恶意 pickledata，在 unpickle 过程中执行任意代码。仅使用您信任的数据调用此函数。


!!! warning "警告"

     使用 GPU tensor调用 [`scatter_object_list()`](#torch.distributed.scatter_object_list "torch.distributed.scatter_object_list") 并没有得到很好的支持，而且效率低下，因为它会导致 GPU -
> CPU 传输，因为tensor会被腌制。请考虑使用 [`scatter()`](#torch.distributed.scatter "torch.distributed.scatter") 代替。


 例子：：




```
>>> # Note: Process group initialization omitted on each rank.
>>> import torch.distributed as dist
>>> if dist.get_rank() == 0:
>>>     # Assumes world_size of 3.
>>>     objects = ["foo", 12, {1: 2}] # any picklable object
>>> else:
>>>     # Can be any list on non-src ranks, elements are not used.
>>>     objects = [None, None, None]
>>> output_list = [None]
>>> dist.scatter_object_list(output_list, objects, src=0)
>>> # Rank i gets objects[i]. For example, on rank 2:
>>> output_list
[{1: 2}]

```


 火炬分布式。


 减少_分散


 ( *output
* , *input_list
* , *op=<RedOpType.SUM: 0>
* , *group=None
* , *async_op=False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#reduce_scatter)[¶](#torch.distributed.reduce_scatter "此定义的永久链接")


 减少tensor列表，然后将其分散到组中的所有进程。


 参数 
* **output** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输出tensor。
* **input_list** ( [*list*](https:///docs.python.org/3/library/stdtypes.html#list "(Python v3.12)")*[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*] 
* ) – 要减少和分散的tensor列表。
* **op** ( *可选
* ) – `torch.distributed.ReduceOp` 枚举中的值之一。指定用于逐元素缩减的操作。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作。


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分。


 火炬分布式。


 减少_分散_tensor


 ( *输出
* , *输入
* , *op=<RedOpType.SUM: 0>
* , *group=None
* , *async_op=False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html #reduce_scatter_tensor)[¶](#torch.distributed.reduce_scatter_tensor "此定义的永久链接")


 减少tensor，然后将其分散到组中的所有等级。


 参数 
* **output** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输出tensor。它在所有等级上应该具有相同的大小。
* **input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 要减少和分散的输入tensor。它的大小应该是输出tensor大小乘以世界大小。输入tensor可以具有以下形状之一：(i)沿主维度串联输出tensor，或(ii)沿主维度输出tensor的堆栈。有关“串联”的定义，请参阅“torch.cat” ()` 。有关“stack”的定义，请参阅 `torch.stack()` 。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作。


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分。


 例子


```
>>> # All tensors below are of torch.int64 dtype and on CUDA devices.
>>> # We have two ranks.
>>> device = torch.device(f'cuda:{rank}')
>>> tensor_out = torch.zeros(2, dtype=torch.int64, device=device)
>>> # Input in concatenation form
>>> tensor_in = torch.arange(world_size * 2, dtype=torch.int64, device=device)
>>> tensor_in
tensor([0, 1, 2, 3], device='cuda:0') # Rank 0
tensor([0, 1, 2, 3], device='cuda:1') # Rank 1
>>> dist.reduce_scatter_tensor(tensor_out, tensor_in)
>>> tensor_out
tensor([0, 2], device='cuda:0') # Rank 0
tensor([4, 6], device='cuda:1') # Rank 1
>>> # Input in stack form
>>> tensor_in = torch.reshape(tensor_in, (world_size, 2))
>>> tensor_in
tensor([[0, 1],
 [2, 3]], device='cuda:0') # Rank 0
tensor([[0, 1],
 [2, 3]], device='cuda:1') # Rank 1
>>> dist.reduce_scatter_tensor(tensor_out, tensor_in)
>>> tensor_out
tensor([0, 2], device='cuda:0') # Rank 0
tensor([4, 6], device='cuda:1') # Rank 1

```


!!! warning "警告"

     Gloo 后端不支持此 API。


 火炬分布式。


 全部_到_全部_单个


 (*输出*、*输入*、*输出_分割_大小



 =
 


 无
* , *输入_split_sizes



 =
 


 无
* , *组



 =
 


 无
* , *异步_op



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#all_to_all_single)[¶](#torch.distributed.all_to_all_single "此定义的永久链接")


 每个进程都会分割输入tensor，然后将分割列表分散到组中的所有进程。然后连接从组中所有进程接收到的tensor并返回单个输出tensor。


 支持复杂tensor。


 参数 
* **output** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 收集的串联输出tensor。
* **input** ( [*Tensor*](tensors.html #torch.Tensor "torch.Tensor") ) – 要分散的输入tensor。
* **output_split_sizes** – (list[Int]，可选)：dim 0 的输出分割大小(如果指定) None 或为空，dim 0 `output` tensor的必须除以 `world_size` 。
* **input_split_sizes** – (list[Int]，可选)：dim 0 的输入分割大小(如果指定 None 或空，则 `input 的 dim 0) ` tensor必须除以 `world_size` 。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作。


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分。


!!! warning "警告"

     all_to_all_single 是实验性的，可能会发生变化。


 例子


```
>>> input = torch.arange(4) + rank * 4
>>> input
tensor([0, 1, 2, 3]) # Rank 0
tensor([4, 5, 6, 7]) # Rank 1
tensor([8, 9, 10, 11]) # Rank 2
tensor([12, 13, 14, 15]) # Rank 3
>>> output = torch.empty([4], dtype=torch.int64)
>>> dist.all_to_all_single(output, input)
>>> output
tensor([0, 4, 8, 12]) # Rank 0
tensor([1, 5, 9, 13]) # Rank 1
tensor([2, 6, 10, 14]) # Rank 2
tensor([3, 7, 11, 15]) # Rank 3

```



```
>>> # Essentially, it is similar to following operation:
>>> scatter_list = list(input.chunk(world_size))
>>> gather_list  = list(output.chunk(world_size))
>>> for i in range(world_size):
>>>     dist.scatter(gather_list[i], scatter_list if i == rank else [], src = i)

```



```
>>> # Another example with uneven split
>>> input
tensor([0, 1, 2, 3, 4, 5]) # Rank 0
tensor([10, 11, 12, 13, 14, 15, 16, 17, 18]) # Rank 1
tensor([20, 21, 22, 23, 24]) # Rank 2
tensor([30, 31, 32, 33, 34, 35, 36]) # Rank 3
>>> input_splits
[2, 2, 1, 1] # Rank 0
[3, 2, 2, 2] # Rank 1
[2, 1, 1, 1] # Rank 2
[2, 2, 2, 1] # Rank 3
>>> output_splits
[2, 3, 2, 2] # Rank 0
[2, 2, 1, 2] # Rank 1
[1, 2, 1, 2] # Rank 2
[1, 2, 1, 1] # Rank 3
>>> output = ...
>>> dist.all_to_all_single(output, input, output_splits, input_splits)
>>> output
tensor([ 0, 1, 10, 11, 12, 20, 21, 30, 31]) # Rank 0
tensor([ 2, 3, 13, 14, 22, 32, 33]) # Rank 1
tensor([ 4, 15, 16, 23, 34, 35]) # Rank 2
tensor([ 5, 17, 18, 24, 36]) # Rank 3

```



```
>>> # Another example with tensors of torch.cfloat type.
>>> input = torch.tensor([1+1j, 2+2j, 3+3j, 4+4j], dtype=torch.cfloat) + 4 * rank * (1+1j)
>>> input
tensor([1+1j, 2+2j, 3+3j, 4+4j]) # Rank 0
tensor([5+5j, 6+6j, 7+7j, 8+8j]) # Rank 1
tensor([9+9j, 10+10j, 11+11j, 12+12j]) # Rank 2
tensor([13+13j, 14+14j, 15+15j, 16+16j]) # Rank 3
>>> output = torch.empty([4], dtype=torch.int64)
>>> dist.all_to_all_single(output, input)
>>> output
tensor([1+1j, 5+5j, 9+9j, 13+13j]) # Rank 0
tensor([2+2j, 6+6j, 10+10j, 14+14j]) # Rank 1
tensor([3+3j, 7+7j, 11+11j, 15+15j]) # Rank 2
tensor([4+4j, 8+8j, 12+12j, 16+16j]) # Rank 3

```


 火炬分布式。


 全部_到_全部


 ( *输出_tensor_列表
* , *输入_tensor_列表
* , *组



 =
 


 无
* , *异步_op



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#all_to_all)[¶](#torch.distributed.all_to_all "此定义的永久链接")


 每个进程将输入tensor列表分散到组中的所有进程，并在输出列表中返回聚集的tensor列表。


 支持复杂tensor。


 参数 
* **output_tensor_list** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)")*[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*]
* ) – 每个等级要收集的tensor列表。
* **input_tensor_list** ( [*list*]( https://docs.python.org/3/library/stdtypes.html#list "(Python v3.12)")*[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor" )*]
* ) – 每个等级分散一个的tensor列表。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作。


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分。


!!! warning "警告"

     all_to_all 是实验性的，可能会发生变化。


 例子


```
>>> input = torch.arange(4) + rank * 4
>>> input = list(input.chunk(4))
>>> input
[tensor([0]), tensor([1]), tensor([2]), tensor([3])] # Rank 0
[tensor([4]), tensor([5]), tensor([6]), tensor([7])] # Rank 1
[tensor([8]), tensor([9]), tensor([10]), tensor([11])] # Rank 2
[tensor([12]), tensor([13]), tensor([14]), tensor([15])] # Rank 3
>>> output = list(torch.empty([4], dtype=torch.int64).chunk(4))
>>> dist.all_to_all(output, input)
>>> output
[tensor([0]), tensor([4]), tensor([8]), tensor([12])] # Rank 0
[tensor([1]), tensor([5]), tensor([9]), tensor([13])] # Rank 1
[tensor([2]), tensor([6]), tensor([10]), tensor([14])] # Rank 2
[tensor([3]), tensor([7]), tensor([11]), tensor([15])] # Rank 3

```



```
>>> # Essentially, it is similar to following operation:
>>> scatter_list = input
>>> gather_list  = output
>>> for i in range(world_size):
>>>     dist.scatter(gather_list[i], scatter_list if i == rank else [], src=i)

```



```
>>> input
tensor([0, 1, 2, 3, 4, 5]) # Rank 0
tensor([10, 11, 12, 13, 14, 15, 16, 17, 18]) # Rank 1
tensor([20, 21, 22, 23, 24]) # Rank 2
tensor([30, 31, 32, 33, 34, 35, 36]) # Rank 3
>>> input_splits
[2, 2, 1, 1] # Rank 0
[3, 2, 2, 2] # Rank 1
[2, 1, 1, 1] # Rank 2
[2, 2, 2, 1] # Rank 3
>>> output_splits
[2, 3, 2, 2] # Rank 0
[2, 2, 1, 2] # Rank 1
[1, 2, 1, 2] # Rank 2
[1, 2, 1, 1] # Rank 3
>>> input = list(input.split(input_splits))
>>> input
[tensor([0, 1]), tensor([2, 3]), tensor([4]), tensor([5])] # Rank 0
[tensor([10, 11, 12]), tensor([13, 14]), tensor([15, 16]), tensor([17, 18])] # Rank 1
[tensor([20, 21]), tensor([22]), tensor([23]), tensor([24])] # Rank 2
[tensor([30, 31]), tensor([32, 33]), tensor([34, 35]), tensor([36])] # Rank 3
>>> output = ...
>>> dist.all_to_all(output, input)
>>> output
[tensor([0, 1]), tensor([10, 11, 12]), tensor([20, 21]), tensor([30, 31])] # Rank 0
[tensor([2, 3]), tensor([13, 14]), tensor([22]), tensor([32, 33])] # Rank 1
[tensor([4]), tensor([15, 16]), tensor([23]), tensor([34, 35])] # Rank 2
[tensor([5]), tensor([17, 18]), tensor([24]), tensor([36])] # Rank 3

```



```
>>> # Another example with tensors of torch.cfloat type.
>>> input = torch.tensor([1+1j, 2+2j, 3+3j, 4+4j], dtype=torch.cfloat) + 4 * rank * (1+1j)
>>> input = list(input.chunk(4))
>>> input
[tensor([1+1j]), tensor([2+2j]), tensor([3+3j]), tensor([4+4j])] # Rank 0
[tensor([5+5j]), tensor([6+6j]), tensor([7+7j]), tensor([8+8j])] # Rank 1
[tensor([9+9j]), tensor([10+10j]), tensor([11+11j]), tensor([12+12j])] # Rank 2
[tensor([13+13j]), tensor([14+14j]), tensor([15+15j]), tensor([16+16j])] # Rank 3
>>> output = list(torch.empty([4], dtype=torch.int64).chunk(4))
>>> dist.all_to_all(output, input)
>>> output
[tensor([1+1j]), tensor([5+5j]), tensor([9+9j]), tensor([13+13j])] # Rank 0
[tensor([2+2j]), tensor([6+6j]), tensor([10+10j]), tensor([14+14j])] # Rank 1
[tensor([3+3j]), tensor([7+7j]), tensor([11+11j]), tensor([15+15j])] # Rank 2
[tensor([4+4j]), tensor([8+8j]), tensor([12+12j]), tensor([16+16j])] # Rank 3

```


 火炬分布式。


 障碍


 ( *团体



 =
 


 无
* , *异步_op



 =
 


 假
* , *设备_ids



 =
 


 无
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#barrier)[¶](#torch.distributed.barrier "此定义的永久链接")


 同步所有进程。


 如果 async_op 为 False，或者在 wait() 上调用异步工作句柄，则该集合会阻塞进程，直到整个组进入该函数。


 参数 
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作
* **device_ids** ( *[
* [*int*](https://docs.python.org/3 /library/functions.html#int "(在 Python v3.12 中)")*]
* *,
* *可选
* ) – 设备/GPU ID 列表。


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 火炬分布式。


 受监控的_barrier


 ( *团体



 =
 


 无
* , *超时



 =
 


 无
* , *等待_all_ranks



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#monitored_barrier)[¶](#torch.distributed.monitored_barrier "此定义的永久链接")


 同步与“torch.distributed.barrier”类似的所有进程，但采用可配置的超时，并且能够报告在该超时内未通过此屏障的排名。具体来说，对于非零等级，将阻塞，直到从等级 0 处理发送/接收。等级 0 将阻塞，直到处理来自其他等级的所有发送/接收，并将报告未能及时响应的等级的失败。请注意，如果一个等级未达到受监控的屏障(例如由于挂起)，则所有其他等级都将在受监控的屏障中失败。


 该集体将阻止组中的所有进程/等级，直到整个组成功退出该功能，这对于调试和同步很有用。但是，它可能会对性能产生影响，并且只能用于调试或需要主机端完全同步点的场景。出于调试目的，可以在应用程序的集体调用之前插入此屏障，以检查是否有任何等级不同步。




!!! note "笔记"

    请注意，此集合仅受 GLOO 后端支持。


 参数 
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果 `None` ，将使用默认进程组。
* **timeout** ( [*datetime.timedelta*](https://docs.python.org/3/library/datetime.html#datetime.timedelta " (在 Python v3.12 中)")*,
* *可选
* ) – 受监控_barrier 超时。如果 `None` ，将使用默认进程组超时。
* **wait_all_ranks** ( [
* bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 是否收集所有失败的排名。默认情况下，这是“False”，并且等级 0 上的“monitored_barrier”将抛出到它遇到的第一个失败的等级上，以便快速失败。通过设置 wait_all_ranks=True``monitored_barrier` 将收集所有失败的排名并抛出包含所有失败排名信息的错误。


 退货


‘无’。


 例子：：




```
>>> # Note: Process group initialization omitted on each rank.
>>> import torch.distributed as dist
>>> if dist.get_rank() != 1:
>>>     dist.monitored_barrier() # Raises exception indicating that
>>> # rank 1 did not call into monitored_barrier.
>>> # Example with wait_all_ranks=True
>>> if dist.get_rank() == 0:
>>>     dist.monitored_barrier(wait_all_ranks=True) # Raises exception
>>> # indicating that ranks 1, 2, ... world_size - 1 did not call into
>>> # monitored_barrier.

```


*班级*


 火炬分布式。


 ReduceOp [¶](#torch.distributed.ReduceOp "此定义的永久链接")


 用于可用归约运算的类似枚举的类：`SUM`、`PRODUCT`、`MIN`、`MAX`、`BAND`、`BOR`、`BXOR` 和 `PREMUL_SUM` 。


使用“NCCL”后端时，“BAND”、“BOR”和“BXOR”缩减不可用。


“AVG”将值除以世界大小，然后再按等级求和。 “AVG”仅适用于“NCCL”后端，并且仅适用于 NCCL 2.10 或更高版本。


`PREMUL_SUM` 在归约之前将输入乘以给定的本地标量。 `PREMUL_SUM` 仅适用于 `NCCL` 后端，并且仅适用于 NCCL 2.11 或更高版本。用户应该使用 `torch.distributed._make_nccl_premul_sum` 。


 此外，复杂tensor不支持“MAX”、“MIN”和“Product”。


 此类的值可以作为属性访问，例如，`ReduceOp.SUM`。它们用于指定归约集合的策略，例如，[`reduce()`](#torch.distributed.reduce "torch.distributed. reduce") 、 [`all_reduce_multigpu()`](#torch.distributed.all_reduce_multigpu "torch.distributed.all_reduce_multigpu") 等


 此类不支持 `__members__` 属性。


*班级*


 火炬分布式。


 reduce_op [¶](#torch.distributed.reduce_op "此定义的永久链接")


 已弃用用于归约运算的类似枚举的类：`SUM`、`Product`、`MIN` 和 `MAX` 。


建议使用 [`ReduceOp`](#torch.distributed.ReduceOp "torch.distributed.ReduceOp") 代替。


## 分析集体通信 [¶](#profiling-collective-communication "此标题的固定链接")


 请注意，您可以使用“torch.profiler”(推荐，仅在 1.8.1 之后可用)或“torch.autograd.profiler”来分析此处提到的集体通信和点对点通信 API。支持所有开箱即用的后端(`gloo`、`nccl`、`mpi`)，并且集体通信使用情况将在分析输出/跟踪中按预期呈现。分析代码与任何常规的 torch 操作员相同：


```
import torch
import torch.distributed as dist
with torch.profiler():
    tensor = torch.randn(20, 10)
    dist.all_reduce(tensor)

```


 请参阅 [探查器文档](https://pytorch.org/docs/main/profiler.html) 以获取探查器功能的完整概述。


## 多 GPU 集合函数 [¶](#multi-gpu-collective-functions "此标题的固定链接")


!!! warning "警告"

     多 GPU 功能将被弃用。如果您必须使用它们，请稍后重新访问我们的文档。


 如果每个节点上有多个 GPU，则在使用 NCCL 和 Gloo 后端时，[`broadcast_multigpu()`](#torch.distributed.broadcast_multigpu "torch.distributed.broadcast_multigpu")[`all_reduce_multigpu ()`](#torch.distributed.all_reduce_multigpu "torch.distributed.all_reduce_multigpu")[`reduce_multigpu()`](#torch.distributed.reduce_multigpu "torch.distributed.reduce_multigpu")[`all_gather_multigpu ()`](#torch.distributed.all_gather_multigpu "torch.distributed.all_gather_multigpu") 和 [`reduce_scatter_multigpu()`](#torch.distributed.reduce_scatter_multigpu "torch.distributed.reduce_scatter_multigpu") 支持分布式集体操作每个节点内有多个 GPU。这些函数可以潜在地提高整体分布式训练性能，并且可以绕过tensor列表轻松使用。传递的tensor列表中的每个tensor都需要位于调用该函数的主机的单独 GPU 设备上。请注意，所有分布式进程中tensor列表的长度需要相同。另请注意，目前仅 NCCL 后端支持多 GPU 集体功能。


 例如，如果我们用于分布式训练的系统有2个节点，每个节点有8个GPU。在 16 个 GPU 中的每一个上，都有一个我们想要全部约简的tensor。下面的代码可以作为参考：


 代码在节点0上运行


```
import torch
import torch.distributed as dist

dist.init_process_group(backend="nccl",
                        init_method="file:///distributed_test",
                        world_size=2,
                        rank=0)
tensor_list = []
for dev_idx in range(torch.cuda.device_count()):
    tensor_list.append(torch.FloatTensor([1]).cuda(dev_idx))

dist.all_reduce_multigpu(tensor_list)

```


 代码在节点1上运行


```
import torch
import torch.distributed as dist

dist.init_process_group(backend="nccl",
                        init_method="file:///distributed_test",
                        world_size=2,
                        rank=1)
tensor_list = []
for dev_idx in range(torch.cuda.device_count()):
    tensor_list.append(torch.FloatTensor([1]).cuda(dev_idx))

dist.all_reduce_multigpu(tensor_list)

```


 调用后，两个节点上的所有 16 个tensor将具有全归约值 16


 火炬分布式。


 广播_multigpu


 ( *tensor_list
* , *src
* , *组



 =
 


 无
* , *异步_op



 =
 


 假
* , *src_tensor



 =
 


 0
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#broadcast_multigpu)[¶](#torch.distributed.broadcast_multigpu "此定义的永久链接")


 将tensor广播到具有多个 GPU tensor节点的整个组。


“tensor”在参与集合的所有进程的所有 GPU 中必须具有相同数量的元素。列表中的每个tensor必须位于不同的 GPU 上


 目前仅支持 nccl 和 gloo 后端tensor应该只能是 GPU tensor


 参数 
* **tensor_list** ( *List
* *[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*]
* ) – 参与集合操作的tensor。如果“src”是等级，则“tensor_list”(“tensor_list[src_tensor]”)中指定的“src_tensor”元素将被广播到 src 进程中的所有其他tensor(在不同的 GPU 上)并且其他非 src 进程的 `tensor_list` 中的所有tensor。您还需要确保调用此函数的所有分布式进程的 `len(tensor_list)` 是相同的。
* **src** ( [
* int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 来源排名。
* **group** ( *ProcessGroup
* *, 
* *可选
* ) – 要处理的流程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作
* **src_tensor** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*,
* *可选
* ) – `tensor_list` 中的源tensor排名


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 火炬分布式。


 所有_reduce_multigpu


 ( *tensor_list
* , *op=<RedOpType.SUM: 0>
* , *group=None
* , *async_op=False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#all_reduce_multigpu )[¶](#torch.distributed.all_reduce_multigpu "此定义的永久链接")


 以所有机器都得到最终结果的方式减少所有机器上的tensor数据。该函数减少了每个节点上的tensor数量，而每个tensor驻留在不同的GPU上。因此，tensor列表中的输入tensor需要是GPUtensor。而且，tensor列表中的每个tensor需要驻留在不同的GPU上。


 调用后，“tensor_list”中的所有“tensor”在所有进程中将按位相同。


 支持复杂tensor。


 目前仅支持 nccl 和 gloo 后端tensor应该只能是 GPU tensor


 参数 
* **tensor_list** ( *List
* *[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*]
* ) – 集合的输入和输出tensor列表。该函数就地运行，要求每个tensor都是不同 GPU 上的 GPU tensor。您还需要确保调用此函数的所有分布式进程的 len(tensor_list) 是相同的。
* **op
* 
* ( *可选
* ) – `torch.distributed.ReduceOp` 枚举中的值之一。指定用于逐元素缩减的操作。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果 `None` ，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 此操作是否应该是异步操作


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 火炬分布式。


 减少_multigpu


 ( *tensor_list
* , *dst
* , *op=<RedOpType.SUM: 0>
* , *group=None
* , *async_op=False
* , *dst_tensor=0
* ) [[来源]] (_modules/torch/distributed/distributed_c10d.html#reduce_multigpu)[¶](#torch.distributed.reduce_multigpu"此定义的永久链接")


 减少所有机器上多个 GPU 上的tensor数据。每个tensor“tensor_list”应驻留在单独的 GPU 上


 只有等级为“dst”的进程上“tensor_list[dst_tensor]”的 GPU 才会收到最终结果。


 目前仅支持 nccl 后端tensor应该只能是 GPU tensor


 参数 
* **tensor_list** ( *List
* *[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*]
* ) – 集合的输入和输出 GPU tensor。该函数就地运行。您还需要确保调用此函数的所有分布式进程的 len(tensor_list) 相同。
* **dst** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 目标排名
* **op** ( *可选
* ) – `torch.distributed 中的值之一.ReduceOp` 枚举。指定用于逐元素缩减的操作。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作
* **dst_tensor** ( [*int*](https://docs.python.org/3/library/function.html#int "(in Python v3.12)")*,
* *可选
* ) – `tensor_list` 中的目标tensor排名


 退货


 异步工作句柄，如果 async_op 设置为 True.None，否则


 火炬分布式。


 所有_gather_multigpu


 ( *输出_tensor_列表
* , *输入_tensor_列表
* , *组



 =
 


 无
* , *异步_op



 =
 


 False
* ) [[source]](_modules/torch/distributed/distributed_c10d.html#all_gather_multigpu)[¶](#torch.distributed.all_gather_multigpu "此定义的永久链接")


 将整个组中的tensor收集到列表中。“tensor_list”中的每个tensor应驻留在单独的 GPU 上


 目前仅支持 nccl 后端tensor应该只能是 GPU tensor


 支持复杂tensor。


 参数 
* **output_tensor_lists** ( *List
* *[
* *List
* *[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*]
* *]
* ) –


 输出列表。它应该在每个 GPU 上包含正确大小的tensor，用于集体的输出，例如`output_tensor_lists[i]` 包含驻留在 `input_tensor_list[i]` 的 GPU 上的所有收集结果。


 请注意，“output_tensor_lists”的每个元素的大小为“world_size * len(input_tensor_list)”，因为该函数会收集组中每个 GPU 的结果。为了解释 `output_tensor_lists[i]` 的每个元素，请注意等级为 k 的 `input_tensor_list[j]` 将出现在 `output_tensor_lists[i][k * world_size 
+ j]`


 另请注意 `len(output_tensor_lists)` 以及 `output_tensor_lists` 中每个元素的大小(每个元素是一个列表，因此 `len(output_tensor_lists[i])` 需要对于调用此函数的所有分布式进程都是相同的。
* **input_tensor_list** ( *List
* *[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*] 
* ) – 要从当前进程广播的tensor列表(在不同的 GPU 上)。请注意，对于调用此函数的所有分布式进程，`len(input_tensor_list)` 需要相同。
* **group** ( 
* ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分


 火炬分布式。


 减少_分散_multigpu


 ( *output_tensor_list
* , *input_tensor_lists
* , *op=<RedOpType.SUM: 0>
* , *group=None
* , *async_op=False
* ) [[source]](_modules /torch/distributed/distributed_c10d.html#reduce_scatter_multigpu)[¶](#torch.distributed.reduce_scatter_multigpu "此定义的永久链接")


 减少tensor列表并将其分散到整个组中。目前仅支持 nccl 后端。


 `output_tensor_list` 中的每个tensor都应该驻留在单独的 GPU 上，就像 `input_tensor_lists` 中的每个tensor列表一样。


 参数 
* **output_tensor_list** ( *List
* *[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*]
* ) –


 输出tensor(在不同的 GPU 上)以接收运算结果。


 请注意，对于调用此函数的所有分布式进程，`len(output_tensor_list)` 需要相同。
* **input_tensor_lists** ( *List
* *[
* *List
* [
* [*tensor*](tensors.html#torch.Tensor "torch.Tensor")*]
* *]
* ) –


 输入列表。它应该在每个 GPU 上包含正确大小的tensor，用于集体的输入，例如`input_tensor_lists[i]` 包含驻留在 `output_tensor_list[i]` 的 GPU 上的reduce_scatter 输入。


 请注意，“input_tensor_lists”的每个元素的大小为“world_size * len(output_tensor_list)”，因为这些函数分散了组中每个 GPU 的结果。为了解释 `input_tensor_lists[i]` 的每个元素，请注意，秩为 k 的 `output_tensor_list[j]` 接收来自 `input_tensor_lists[i][k * world] 的归约分散结果_size 
+ j]`


 另请注意 `len(input_tensor_lists)` ，以及 `input_tensor_lists` 中每个元素的大小(每个元素是一个列表，因此 `len(input_tensor_lists[i])` )需要对于调用此函数的所有分布式进程都是相同的。
* **group** ( *ProcessGroup
* *,
* *可选
* ) – 要处理的进程组。如果没有，将使用默认进程组。
* **async_op** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)")*,
* *可选
* ) – 此操作是否应该是异步操作。


 退货


 异步工作句柄，如果 async_op 设置为 True.None，如果不是 async_op 或者不属于组的一部分。


## 第三方后端 [¶](#third-party-backends "此标题的永久链接")


 除了内置的 GLOO/MPI/NCCL 后端外，PyTorch 分布式还通过运行时注册机制支持第三方后端。有关如何通过 C++ 扩展开发第三方后端的参考，请参阅[教程 
- 自定义 C++ 和 CUDA 扩展](https://pytorch.org/tutorials/advanced/cpp_extension.html) 和 `test/cpp_extensions/cpp_c10d_extension.cpp` 。第三方后端的能力由其自身的实现决定。


 新的后端源自`c10d::ProcessGroup`，并通过[`torch.distributed.Backend.register_backend()`](#torch.distributed.Backend.register_backend "torch.distributed.Backend"注册后端名称和实例化接口.register_backend") 导入时。


 当手动导入此后端并使用相应的后端名称调用 [`torch.distributed.init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group") 时，`torch.distributed` 包在新后端上运行。


!!! warning "警告"

     第三方后端的支持是实验性的，可能会发生变化。


## 启动实用程序 [¶](#launch-utility "永久链接到此标题")


 torch.distributed 包还在 torch.distributed.launch 中提供了启动实用程序。该帮助程序实用程序可用于为每个节点启动多个进程以进行分布式训练。


“torch.distributed.launch”是一个在每个训练节点上生成多个分布式训练进程的模块。


!!! warning "警告"

     该模块将被弃用，取而代之的是 [torchrun](elastic/run.html#launcher-api) 。


 该实用程序可用于单节点分布式训练，其中每个节点将生成一个或多个进程。该实用程序可用于 CPU 训练或 GPU 训练。如果该实用程序用于 GPU 训练，则每个分布式进程将在单个 GPU 上运行。这样可以很好地提高单节点训练性能。它还可以用于多节点分布式训练，通过在每个节点上生成多个进程来很好地改进多节点分布式训练性能。这对于具有直接 GPU 支持的多个 Infiniband 接口的系统尤其有利，因为所有其中可用于聚合通信带宽。


 在单节点分布式训练或多节点分布式训练的情况下，该实用程序将为每个节点启动给定数量的进程(“--nproc-per-node”)。如果用于 GPU 训练，这个数字需要小于当前系统上的 GPU 数量 ( `nproc_per_node` )，并且每个进程都将在单个 GPU 上运行，从 *GPU 0 到 GPU (nproc\ _per_node 
- 1)
* 。


**如何使用该模块：**


1、单节点多进程分布式训练


```
python -m torch.distributed.launch --nproc-per-node=NUM_GPUS_YOU_HAVE
           YOUR_TRAINING_SCRIPT.py (--arg1 --arg2 --arg3 and all other
           arguments of your training script)

```


2.多节点多进程分布式训练：(例如两个节点)


 节点1：*(IP：192.168.1.1，并且有空闲端口：1234)*



```
python -m torch.distributed.launch --nproc-per-node=NUM_GPUS_YOU_HAVE
           --nnodes=2 --node-rank=0 --master-addr="192.168.1.1"
           --master-port=1234 YOUR_TRAINING_SCRIPT.py (--arg1 --arg2 --arg3
           and all other arguments of your training script)

```


 节点2：


```
python -m torch.distributed.launch --nproc-per-node=NUM_GPUS_YOU_HAVE
           --nnodes=2 --node-rank=1 --master-addr="192.168.1.1"
           --master-port=1234 YOUR_TRAINING_SCRIPT.py (--arg1 --arg2 --arg3
           and all other arguments of your training script)

```


3. 要查找该模块提供的可选参数：


```
python -m torch.distributed.launch --help

```


**重要通知：**


 1. 此实用程序和多进程分布式(单节点或多节点)GPU 训练目前仅使用 NCCL 分布式后端才能实现最佳性能。因此，NCCL 后端是推荐用于 GPU 训练的后端。


 2. 在你的训练程序中，你必须解析命令行参数： `--local-rank=LOCAL_PROCESS_RANK` ，该参数将由本模块提供。如果你的训练程序使用 GPU，你应该确保你的代码仅在LOCAL_PROCESS_RANK的GPU设备上运行。这可以通过以下方式完成：


 解析 local_rank Parameters


```
>>> import argparse
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument("--local-rank", type=int)
>>> args = parser.parse_args()

```


 使用以下任一方法将您的设备设置为本地排名


```
>>> torch.cuda.set_device(args.local_rank)  # before your code runs

```




 or
 


```
>>> with torch.cuda.device(args.local_rank):
>>>    # your code to run
>>>    ...

```


 3. 在你的训练程序中，你应该在开始时调用以下函数来启动分布式后端。强烈建议 `init_method=env://` 。其他 init 方法(例如 `tcp://` )也可以工作，但 `env://` 是该模块官方支持的方法。


```
>>> torch.distributed.init_process_group(backend='YOUR BACKEND',
>>>                                      init_method='env://')

```


 4. 在您​​的训练计划中，您可以使用常规分布式函数或使用 [`torch.nn.parallel.DistributedDataParallel()`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 模块。如果您的训练程序使用 GPU 进行训练，并且您希望使用 [`torch.nn.parallel.DistributedDataParallel()`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 模块，这里是如何配置它。


```
>>> model = torch.nn.parallel.DistributedDataParallel(model,
>>>                                                   device_ids=[args.local_rank],
>>>                                                   output_device=args.local_rank)

```


 请确保“device_ids”参数设置为您的代码将在其上运行的唯一 GPU 设备 ID。这通常是进程的本地等级。换句话说，“device_ids”需要是“[args.local_rank]”，并且“output_device”需要是“args.local_rank”才能使用此实用程序


 5. 另一种通过环境变量“LOCAL_RANK”将“local_rank”传递给子进程的方法。当您使用 `--use-env=True` 启动脚本时，会启用此行为。您必须调整上面的子进程示例，将 `args.local_rank` 替换为 `os.environ['LOCAL_RANK']` ；当您指定此标志时，启动器将不会传递“--local-rank”。


!!! warning "警告"

    `local_rank` 不是全局唯一的：它仅是机器上每个进程唯一的。因此，不要用它来决定是否应该写入网络文件系统等。请参阅 <https://github.com/pytorch/pytorch/issues/12042
> 的示例，了解如果不正确执行此操作，事情可能会出错。


## 生成实用程序 [¶](#spawn-utility "此标题的固定链接")


 [Multiprocessing 包 
- torch.multiprocessing](multiprocessing.html#multiprocessing-doc) 包还在 [`torch.multiprocessing.spawn()`](multiprocessing.html#torch.multiprocessing.spawn "torch" 中提供了一个 `spawn` 函数.multiprocessing.spawn") 。该辅助函数可用于生成多个进程。它的工作原理是传入您想要运行的函数并生成 N 个进程来运行它。这也可以用于多进程分布式训练。


 有关如何使用的参考，请参考[PyTorch示例-ImageNet实现](https://github.com/pytorch/examples/tree/master/imagenet)


 请注意，此功能需要 Python 3.4 或更高版本。


## 调试 `torch.distributed` 应用程序 [¶](#debugging-torch-distributed-applications "永久链接到此标题")


 由于难以理解挂起、崩溃或跨级别行为不一致，调试分布式应用程序可能具有挑战性。 “torch.distributed”提供了一套工具来帮助以自助方式调试培训应用程序：


### 监控屏障 [¶](#monitored-barrier "此标题的永久链接")


 从 v1.10 开始，[`torch.distributed.monitored_barrier()`](#torch.distributed.monitored_barrier "torch.distributed.monitored_barrier") 作为 [`torch.distributed.barrier()`] 的替代方案存在(#torch.distributed.barrier "torch.distributed.barrier") 失败，并提供有关崩溃时哪个等级可能出现故障的有用信息，即并非所有等级都调用 [`torch.distributed.monitored_barrier()`](#torch.distributed.monitored_barrier "torch.distributed.monitored_barrier") 在提供的超时时间内。 [`torch.distributed.monitored_barrier()`](#torch.distributed.monitored_barrier "torch.distributed.monitored_barrier") 在类似于确认的过程中使用 `send` /`recv` 通信原语实现主机侧屏障，允许等级 0 报告哪个等级未能及时确认障碍。作为示例，请考虑以下函数，其中排名 1 无法调用 [`torch.distributed.monitored_barrier()`](#torch.distributed.monitored_barrier "torch.distributed.monitored_barrier") (实际上，这可能是由于应用程序错误或挂在以前的集合中)：


```
import os
from datetime import timedelta

import torch
import torch.distributed as dist
import torch.multiprocessing as mp


def worker(rank): dist.init_process_group("nccl",rank=rank, world_size=2) # 受监控的屏障需要 gloo 进程组来执行主机端同步。 group_gloo = dist.new_group(backend="gloo") 如果排名不在 [1] 中： dist.monitored_barrier(group=group_gloo, timeout=timedelta(seconds=2))


if __name__ == "__main__":
    os.environ["MASTER_ADDR"] = "localhost"
    os.environ["MASTER_PORT"] = "29501"
    mp.spawn(worker, nprocs=2, args=())

```


 在等级 0 上生成以下错误消息，允许用户确定哪个等级可能有问题并进一步调查：


```
RuntimeError: Rank 1 failed to pass monitoredBarrier in 2000 ms
 Original exception:
[gloo/transport/tcp/pair.cc:598] Connection closed by peer [2401:db00:eef0:1100:3560:0:1c05:25d]:8594

```


### `TORCH_DISTRIBUTED_DEBUG`[¶](#torch-distributed-debug "此标题的永久链接")


 使用“TORCH_CPP_LOG_LEVEL=INFO”，环境变量“TORCH_DISTRIBUTED_DEBUG”可用于触发其他有用的日志记录和集体同步检查，以确保所有等级正确同步。根据所需的调试级别，可以将“TORCH_DISTRIBUTED_DEBUG”设置为“OFF”(默认)、“INFO”或“DETAIL”。请注意，最详细的选项“DETAIL”可能会影响应用程序性能，因此仅应在调试问题时使用。


 当使用 [`torch.nn.parallel.DistributedDataParallel()`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn 训练模型时，设置 TORCH_DISTRIBUTED_DEBUG=INFO` 将导致额外的调试日志记录。初始化parallel.DistributedDataParallel“torch.nn.parallel.DistributedDataParallel”)，并且“TORCH_DISTRIBUTED_DEBUG=DETAIL”将另外记录选定迭代次数的运行时性能统计信息。这些运行时统计数据包括前向时间、后向时间、梯度通信时间等数据。作为示例，给出以下应用：


```
import os

import torch
import torch.distributed as dist
import torch.multiprocessing as mp


类 TwoLinLayerNet(torch.nn.Module): def __init__(self): super().__init__() self.a = torch.nn.Linear(10, 10 , 偏差=False) self.b = torch.nn.Linear(10, 1, 偏差=False) def 前向(self, x): a = self.a(x) b = self.b(x) return (a , b)


defworker(rank): dist.init_process_group("nccl",rank=rank, world_size=2) torch.cuda.set_device(rank) print("init model") model = TwoLinLayerNet().cuda() print("init ddp") ddp_model = torch.nn.parallel.DistributedDataParallel(model, device_ids=[rank]) inp = torch.randn(10, 10).cuda() print(" train") for _ in range(20): 输出 = ddp_model(inp) 损失 = 输出[0] 
+ 输出[1] loss.sum().backward()


if __name__ == "__main__":
    os.environ["MASTER_ADDR"] = "localhost"
    os.environ["MASTER_PORT"] = "29501"
    os.environ["TORCH_CPP_LOG_LEVEL"]="INFO"
    os.environ[
        "TORCH_DISTRIBUTED_DEBUG"
    ] = "DETAIL"  # set to DETAIL for runtime logging.
    mp.spawn(worker, nprocs=2, args=())

```


 初始化时会呈现以下日志：


```
I0607 16:10:35.739390 515217 logger.cpp:173] [Rank 0]: DDP Initialized with:
broadcast_buffers: 1
bucket_cap_bytes: 26214400
find_unused_parameters: 0
gradient_as_bucket_view: 0
is_multi_device_module: 0
iteration: 0
num_parameter_tensors: 2
output_device: 0
rank: 0
total_parameter_size_bytes: 440
world_size: 2
backend_name: nccl
bucket_sizes: 440
cuda_visible_devices: N/A
device_ids: 0
dtypes: float
master_addr: localhost
master_port: 29501
module_name: TwoLinLayerNet
nccl_async_error_handling: N/A
nccl_blocking_wait: N/A
nccl_debug: WARN
nccl_ib_timeout: N/A
nccl_nthreads: N/A
nccl_socket_ifname: N/A
torch_distributed_debug: INFO

```


 以下日志在运行时呈现(当设置“TORCH_DISTRIBUTED_DEBUG=DETAIL”时)：


```
I0607 16:18:58.085681 544067 logger.cpp:344] [Rank 1 / 2] Training TwoLinLayerNet unused_parameter_size=0
 Avg forward compute time: 40838608
 Avg backward compute time: 5983335
Avg backward comm. time: 4326421
 Avg backward comm/comp overlap time: 4207652
I0607 16:18:58.085693 544066 logger.cpp:344] [Rank 0 / 2] Training TwoLinLayerNet unused_parameter_size=0
 Avg forward compute time: 42850427
 Avg backward compute time: 3885553
Avg backward comm. time: 2357981
 Avg backward comm/comp overlap time: 2234674

```


 此外，`TORCH_DISTRIBUTED_DEBUG=INFO` 增强了 [`torch.nn.parallel.DistributedDataParallel()`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel " 中的崩溃日志记录torch.nn.parallel.DistributedDataParallel") 由于模型中未使用参数。目前，`find_unused_parameters=True` 必须传入 [`torch.nn.parallel.DistributedDataParallel()`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 初始化，如果有在前向传递中可能未使用的参数，并且从 v1.10 开始，所有模型输出都需要在损失计算中使用，如 [`torch.nn.parallel.DistributedDataParallel( )`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 不支持向后传递中未使用的参数。这些约束对于较大的模型来说尤其具有挑战性，因此当因错误而崩溃时，[`torch.nn.parallel.DistributedDataParallel()`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 将记录所有未使用的参数的完全限定名称。例如，在上面的应用中，如果我们将“loss”修改为“loss = output[1]”，则“TwoLinLayerNet.a”在向后传递中不会收到梯度，从而导致“DDP”失败。崩溃时，用户会收到有关未使用的参数的信息，这对于手动查找大型模型可能具有挑战性：


```
RuntimeError: Expected to have finished reduction in the prior iteration before starting a new one. This error indicates that your module has parameters that were not used in producing loss. You can enable unused parameter detection by passing
 the keyword argument `find_unused_parameters=True` to `torch.nn.parallel.DistributedDataParallel`, and by
making sure all `forward` function outputs participate in calculating loss.
If you already have done the above, then the distributed data parallel module wasn't able to locate the output tensors in the return value of your module's `forward` function. Please include the loss function and the structure of the return va
lue of `forward` of your module when reporting this issue (e.g. list, dict, iterable).
Parameters which did not receive grad for rank 0: a.weight
Parameter indices which did not receive grad for rank 0: 0

```


 设置 TORCH_DISTRIBUTED_DEBUG=DETAIL 将会触发对用户直接或间接发出的每个集体调用的额外一致性和同步检查(例如 DDP `allreduce` )。这是通过创建一个包装进程组来完成的，该进程组包装由 [`torch.distributed.init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group") 和 [`torch.distributed.new_group()`](#torch.distributed.new_group "torch.distributed.new_group") API。因此，这些 API 将返回一个包装器进程组，该进程组可以像常规进程组一样使用，但在将集合分派到底层进程组之前执行一致性检查。目前，这些检查包括 [`torch.distributed.monitored_barrier()`](#torch.distributed.monitored_barrier "torch.distributed.monitored_barrier") ，确保所有队伍完成其未完成的集体调用并报告卡住的队伍。接下来，通过确保所有集合函数匹配并以一致的tensor形状调用来检查集合本身的一致性。如果不是这种情况，应用程序崩溃时会包含详细的错误报告，而不是挂起或无信息的错误消息。例如，考虑以下函数，其输入形状与 [`torch.distributed.all_reduce()`](#torch.distributed.all_reduce "torch.distributed.all_reduce") 不匹配：


```
import torch
import torch.distributed as dist
import torch.multiprocessing as mp


defworker(rank): dist.init_process_group("nccl",rank=rank,world_size=2) torch.cuda.set_device(rank) tensor = torch.randn(10 如果rank == 0否则 20).cuda() dist.all_reduce(tensor) torch.cuda.synchronize(device=rank)


if __name__ == "__main__":
    os.environ["MASTER_ADDR"] = "localhost"
    os.environ["MASTER_PORT"] = "29501"
    os.environ["TORCH_CPP_LOG_LEVEL"]="INFO"
    os.environ["TORCH_DISTRIBUTED_DEBUG"] = "DETAIL"
    mp.spawn(worker, nprocs=2, args=())

```


 使用“NCCL”后端，此类应用程序可能会导致挂起，这在重要场景中可能对根本原因造成挑战。如果用户启用“TORCH_DISTRIBUTED_DEBUG=DETAIL”并重新运行应用程序，以下错误消息将揭示根本原因：


```
work = default_pg.allreduce([tensor], opts)
RuntimeError: Error when verifying shape tensors for collective ALLREDUCE on rank 0. This likely indicates that input shapes into the collective are mismatched across ranks. Got shapes:  10
[ torch.LongTensor{1} ]

```


!!! note "笔记"

    为了在运行时对调试级别进行细粒度控制，可以使用函数 `torch.distributed.set_debug_level()` 、 `torch.distributed.set_debug_level_from_env()` 和 `torch.distributed也可以使用.get_debug_level()`。


 此外，TORCH_DISTRIBUTED_DEBUG=DETAIL 可以与 TORCH_SHOW_CPP_STACKTRACES=1 结合使用，以在检测到集体不同步时记录整个调用堆栈。这些集体去同步检查将适用于使用由 [`torch.distributed.init_process_group()`](#torch.distributed.init_process_group "torch.distributed.init_process_group 创建的进程组支持的 `c10d` 集体调用的所有应用程序") 和 [`torch.distributed.new_group()`](#torch.distributed.new_group "torch.distributed.new_group") API。


## 日志记录 [¶](#logging "此标题的永久链接")


 除了通过 [`torch.distributed.monitored_barrier()`](#torch.distributed.monitored_barrier "torch.distributed.monitored_barrier") 和 `TORCH_DISTRIBUTED_DEBUG` 提供显式调试支持之外，`TORCH_DISTRIBUTED_DEBUG` 的底层 C++ 库torch.distributed` 还输出各个级别的日志消息。这些消息有助于了解分布式训练作业的执行状态以及解决网络连接故障等问题。以下矩阵显示了如何通过“TORCH_CPP_LOG_LEVEL”和“TORCH_DISTRIBUTED_DEBUG”环境变量的组合来调整日志级别。


| 	`TORCH_CPP_LOG_LEVEL`	 | 	`TORCH_DISTRIBUTED_DEBUG`	 | 	 Effective Log Level	  |
| --- | --- | --- |
| 	`ERROR`	 | 	 ignored	  | 	 Error	  |
| 	`WARNING`	 | 	 ignored	  | 	 Warning	  |
| 	`INFO`	 | 	 ignored	  | 	 Info	  |
| 	`INFO`	 | 	`INFO`	 | 	 Debug	  |
| 	`INFO`	 | 	`DETAIL`	 | 	 Trace (a.k.a. All)	  |


 Distributed 有一个从 RuntimeError 派生的自定义异常类型，称为 torch.distributed.DistBackendError 。当发生后端特定错误时会引发此异常。例如，如果使用 NCCL 后端，并且用户尝试使用 NCCL 库不可用的 GPU。


*班级*


 火炬分布式。


 DistBackendError [¶](#torch.distributed.DistBackendError "此定义的永久链接")


 分布式发生后端错误时引发异常


!!! warning "警告"

     DistBackendError 异常类型是一项实验性功能，可能会发生变化。