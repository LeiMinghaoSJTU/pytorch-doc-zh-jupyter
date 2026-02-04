# 分布式 RPC 框架 [¶](#distributed-rpc-framework "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/rpc>
>
> 原始地址：<https://pytorch.org/docs/stable/rpc.html>


 分布式 RPC 框架通过一组原语提供多机模型训练机制以允许远程通信，并提供更高级别的 API 来自动区分跨多台机器的模型。


!!! warning "警告"

     RPC包中的API是稳定的。有多个正在进行的工作项目可以提高性能和错误处理，这些工作项目将在未来的版本中发布。


!!! warning "警告"

     CUDA 支持是在 PyTorch 1.9 中引入的，并且仍然是一个 **beta** 功能。并非 RPC 包的所有功能都与 CUDA 支持兼容，因此不鼓励使用它们。这些不受支持的功能包括：RRefs、JIT 兼容性、dist autograd 和 dist 优化器以及分析。这些缺点将在未来的版本中得到解决。


!!! note "笔记"

    请参阅[PyTorch分布式概述](https://pytorch.org/tutorials/beginner/dist_overview.html)，简要介绍与分布式训练相关的所有功能。


## 基础知识 [¶](#basics "此标题的永久链接")


 分布式 RPC 框架可以轻松地远程运行函数，支持引用远程对象而无需复制真实数据，并提供自动分级和优化器 API 来透明地向后运行并跨 RPC 边界更新参数。这些功能可分为四组 API。


1. **远程过程调用 (RPC)** 支持使用给定参数在指定目标工作线程上运行函数并获取返回值或创建对返回值的引用。主要有 3 个 RPC API： [`rpc_sync()`](#torch.distributed.rpc.rpc_sync "torch.distributed.rpc.rpc_sync")(同步)、[`rpc_async()`](# torch.distributed.rpc.rpc_async "torch.distributed.rpc.rpc_async") (异步)和 [`remote()`](#torch.distributed.rpc.remote "torch.distributed.rpc.remote") (异步)并返回对远程返回值的引用)。如果用户代码在没有返回值的情况下无法继续，请使用同步 API。否则，使用异步 API 获取 future，并在调用者需要返回值时等待 future。当需要远程创建某些内容但不需要将其获取给调用者时，[`remote()`](#torch.distributed.rpc.remote "torch.distributed.rpc.remote") API 非常有用。想象一下驱动程序进程正在设置参数服务器和训练器的情况。驱动程序可以在参数服务器上创建嵌入表，然后与训练器共享对嵌入表的引用，但驱动程序本身永远不会在本地使用嵌入表。在这种情况下，[`rpc_sync()`](#torch.distributed.rpc.rpc_sync "torch.distributed.rpc.rpc_sync") 和 [`rpc_async()`](#torch.distributed.rpc。 rpc_async "torch.distributed.rpc.rpc_async") 不再合适，因为它们总是暗示返回值将立即或将来返回给调用者。2. **远程引用 (RRef)** 用作本地或远程对象的分布式共享指针。它可以与其他工作人员共享，并且引用计数将被透明地处理。每个 RRef 只有一个所有者，并且该对象仅存在于该所有者中。持有 RRef 的非所有者工作人员可以通过显式请求从所有者那里获取对象的副本。当工作人员需要访问某些数据对象，但其本身既不是创建者([`remote()`](#torch.distributed.rpc.remote "torch.distributed.rpc.remote") 的调用者)或对象的所有者。我们将在下面讨论的分布式优化器就是此类用例的一个示例。3． **分布式 Autograd** 将前向传递中涉及的所有工作人员上的本地 Autograd 引擎缝合在一起，并在后向传递期间自动联系它们以计算梯度。如果在进行分布式模型并行训练、参数服务器训练等时，前向传递需要跨越多台机器，则这尤其有用。有了此功能，用户代码不再需要担心如何跨 RPC 边界发送梯度以及以何种顺序发送梯度是否应该启动本地 autograd 引擎，如果前向传递中存在嵌套且相互依赖的 RPC 调用，这可能会变得非常复杂。4。 **Distributed Optimizer** 的构造函数采用 [`Optimizer()`](optim.html#torch.optim.Optimizer "torch.optim.Optimizer") (例如 [`SGD()`](generated/torch.optim.SGD.html#torch.optim.SGD "torch.optim.SGD") , [`Adagrad()`](generated/torch.optim.Adagrad.html#torch.optim.Adagrad "torch.optim.Adagrad ") 等)和参数 RRef 列表，在每个不同的 RRef 所有者上创建一个 [`Optimizer()`](optim.html#torch.optim.Optimizer "torch.optim.Optimizer") 实例，并相应地更新参数当运行 `step()` 时。当您分配前向和后向传递时，参数和梯度将分散在多个工作人员中，因此需要对每个相关工作人员都有一个优化器。分布式优化器将所有这些局部优化器包装成一个，并提供简洁的构造函数和“step()”API。


## RPC [¶](#rpc "此标题的永久链接")


 在使用 RPC 和分布式 autograd 原语之前，必须进行初始化。为了初始化 RPC 框架，我们需要使用 [`init_rpc()`](#torch.distributed.rpc.init_rpc "torch.distributed.rpc.init_rpc") 来初始化 RPCframework、RRef 框架和分布式 autograd。


 torch.distributed.rpc。


 初始化_rpc


 ( *名称
* , *后端



 =
 


 无
* , *排名



 =
 


 -1
* , *世界_大小



 =
 


 无
* , *rpc_backend_options



 =
 


 无
* ) [[source]](_modules/torch/distributed/rpc.html#init_rpc)[¶](#torch.distributed.rpc.init_rpc "此定义的永久链接")


 初始化 RPC 原语，例如本地 RPC 代理和分布式 autograd，这会立即使当前进程准备好发送和接收 RPC。


 参数 
* **name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 全局唯一名称这个节点。 (例如，`Trainer3`、`ParameterServer2`、`Master`、`Worker1`)名称只能包含数字、字母、下划线、冒号和/或破折号，并且必须短于 128 个字符。
* **后端** ( [*BackendType*](#torch.distributed.rpc.BackendType "torch.distributed.rpc.BackendType")*,
* *可选
* ) – RPC 后端实现的类型。支持的值为 `BackendType.TENSORPIPE` (默认值)。有关详细信息，请参阅 [Backends](#rpc-backends)。
* **rank** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 该节点的全局唯一 id/rank。
* **world_size** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 组中的工作人员数量。
* **rpc_backend_options** ( [*RpcBackendOptions
* ](#torch.distributed.rpc.RpcBackendOptions "torch.distributed.rpc.RpcBackendOptions")*,
* *可选
* ) – 传递给 RpcAgent 构造函数的选项。它必须是 [`RpcBackendOptions`](#torch.distributed.rpc.RpcBackendOptions "torch.distributed.rpc.RpcBackendOptions") 的特定于代理的子类，并且包含特定于代理的初始化配置。默认情况下，对于所有代理，它将默认超时设置为 60 秒，并与使用 `init_method = "env://"` 初始化的底层进程组执行会合，这意味着环境变量 `MASTER_ADDR` 和 `MASTER_PORT ` 需要正确设置。请参阅 [后端](#rpc-backends) 了解更多信息并查找可用的选项。


 以下 API 允许用户远程执行函数以及创建对远程数据对象的引用 (RRef)。在这些 API 中，当传递“Tensor”作为参数或返回值时，目标工作线程将尝试创建具有相同元(即形状、步幅等)的“Tensor”。我们故意禁止传输 CUDA tensor，因为如果源工作器和目标工作器上的设备列表不匹配，则可能会崩溃。在这种情况下，应用程序始终可以显式地将输入tensor移动到调用方的 CPU，并在必要时将其移动到被调用方所需的设备。


!!! warning "警告"

     RPC 中的 TorchScript 支持是一个原型功能，可能会发生变化。从v1.5.0开始，“torch.distributed.rpc”支持调用TorchScript函数作为RPC目标函数，这将有助于提高被调用方的并行性，因为执行TorchScript函数不需要GIL。


 torch.distributed.rpc。


 rpc_sync


 (*到*，*func*，*args



 =
 


 无*，*kwargs



 =
 


 无
* , *超时



 =
 


 -1.0
* ) [[source]](_modules/torch/distributed/rpc/api.html#rpc_sync)[¶](#torch.distributed.rpc.rpc_sync "此定义的永久链接")


 进行阻塞 RPC 调用以在工作进程“to”上运行函数“func”。 RPC 消息的发送和接收与 Python 代码的执行并行。该方法是线程安全的。


 参数 
* **to** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*或
* [*WorkerInfo
* ](#torch.distributed.rpc.WorkerInfo "torch.distributed.rpc.WorkerInfo")*或
* [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12)") ) – 目标工作人员的名称/等级/`WorkerInfo`。
* **func** ( *Callable
* ) – 可调用函数，例如 Python 可调用函数、内置运算符(例如 [`add( )`](generated/torch.add.html#torch.add "torch.add") ) 和带注释的TorchScript 函数。
* **args** ( [*tuple*](https://docs.python.org/3 /library/stdtypes.html#tuple "(in Python v3.12)") ) – `func` 调用的参数元组。
* **kwargs** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 是 `func` 调用的关键字参数的字典。
* **timeout** ( [*float*]( https://docs.python.org/3/library/functions.html#float "(Python v3.12)")*,
* *可选
* ) – 用于此 RPC 的超时(以秒为单位)。如果 RPC 在这段时间内没有完成，则会引发一个异常，表明它已超时。值 0 表示无限超时，即永远不会引发超时错误。如果未提供，则使用初始化期间设置的默认值或使用 `_set_rpc_timeout` 设置的默认值。


 退货


 返回使用 `args` 和 `kwargs` 运行 `func` 的结果。


 例子：：


 确保两个工作进程上的“MASTER_ADDR”和“MASTER_PORT”均已正确设置。请参阅 [`init_process_group()`](distributed.html#torch.distributed.init_process_group "torch.distributed.init_process_group") API 了解更多详细信息。例如，


 导出 MASTER_ADDR=localhostexport MASTER_PORT=5678


 然后在两个不同的进程中运行以下代码：


```
>>> # On worker 0:
>>> import torch
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker0", rank=0, world_size=2)
>>> ret = rpc.rpc_sync("worker1", torch.add, args=(torch.ones(2), 3))
>>> rpc.shutdown()

```



```
>>> # On worker 1:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker1", rank=1, world_size=2)
>>> rpc.shutdown()

```


 下面是使用 RPC 运行 TorchScript 函数的示例。


```
>>> # On both workers:
>>> @torch.jit.script
>>> def my_script_add(t1, t2):
>>>    return torch.add(t1, t2)

```



```
>>> # On worker 0:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker0", rank=0, world_size=2)
>>> ret = rpc.rpc_sync("worker1", my_script_add, args=(torch.ones(2), 3))
>>> rpc.shutdown()

```



```
>>> # On worker 1:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker1", rank=1, world_size=2)
>>> rpc.shutdown()

```


 torch.distributed.rpc。


 rpc_async


 (*到*，*func*，*args



 =
 


 无*，*kwargs



 =
 


 无
* , *超时



 =
 


 -1.0
* ) [[source]](_modules/torch/distributed/rpc/api.html#rpc_async)[¶](#torch.distributed.rpc.rpc_async "此定义的永久链接")


 进行非阻塞 RPC 调用以在工作进程“to”上运行函数“func”。 RPC 消息的发送和接收与 Python 代码的执行并行。该方法是线程安全的。此方法将立即返回一个可以等待的 [`Future`](futures.html#torch.futures.Future "torch.futures.Future")。


 参数 
* **to** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*或
* [*WorkerInfo
* ](#torch.distributed.rpc.WorkerInfo "torch.distributed.rpc.WorkerInfo")*或
* [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12)") ) – 目标工作人员的名称/等级/`WorkerInfo`。
* **func** ( *Callable
* ) – 可调用函数，例如 Python 可调用函数、内置运算符(例如 [`add( )`](generated/torch.add.html#torch.add "torch.add") ) 和带注释的TorchScript 函数。
* **args** ( [*tuple*](https://docs.python.org/3 /library/stdtypes.html#tuple "(in Python v3.12)") ) – `func` 调用的参数元组。
* **kwargs** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 是 `func` 调用的关键字参数的字典。
* **timeout** ( [*float*]( https://docs.python.org/3/library/functions.html#float "(Python v3.12)")*,
* *可选
* ) – 用于此 RPC 的超时(以秒为单位)。如果 RPC 在这段时间内没有完成，则会引发一个异常，表明它已超时。值 0 表示无限超时，即永远不会引发超时错误。如果未提供，则使用初始化期间设置的默认值或使用 `_set_rpc_timeout` 设置的默认值。


 退货


 返回一个可以等待的 [`Future`](futures.html#torch.futures.Future "torch.futures.Future") 对象。完成后，可以从 [`Future`](futures.html#torch.futures.Future "torch.futures.Future") 对象中检索 `args` 和 `kwargs` 上的 `func` 的返回值。


!!! warning "警告"

     不支持使用 GPU tensor作为“func”的参数或返回值，因为我们不支持通过线路发送 GPU tensor。在将 GPU tensor用作参数或返回 `func` 值之前，您需要将 GPU tensor显式复制到 CPU。


!!! warning "警告"

     `rpc_async` API 不会复制参数tensor的存储，直到通过线路发送它们，这可以由不同的线程完成，具体取决于 RPC 后端类型。调用者应确保这些tensor的内容保持完整，直到返回的 [`Future`](futures.html#torch.futures.Future "torch.futures.Future") 完成。


 例子：：


 确保两个工作进程上的“MASTER_ADDR”和“MASTER_PORT”均已正确设置。请参阅 [`init_process_group()`](distributed.html#torch.distributed.init_process_group "torch.distributed.init_process_group") API 了解更多详细信息。例如，


 导出 MASTER_ADDR=localhostexport MASTER_PORT=5678


 然后在两个不同的进程中运行以下代码：


```
>>> # On worker 0:
>>> import torch
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker0", rank=0, world_size=2)
>>> fut1 = rpc.rpc_async("worker1", torch.add, args=(torch.ones(2), 3))
>>> fut2 = rpc.rpc_async("worker1", min, args=(1, 2))
>>> result = fut1.wait() + fut2.wait()
>>> rpc.shutdown()

```



```
>>> # On worker 1:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker1", rank=1, world_size=2)
>>> rpc.shutdown()

```


 下面是使用 RPC 运行 TorchScript 函数的示例。


```
>>> # On both workers:
>>> @torch.jit.script
>>> def my_script_add(t1, t2):
>>>    return torch.add(t1, t2)

```



```
>>> # On worker 0:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker0", rank=0, world_size=2)
>>> fut = rpc.rpc_async("worker1", my_script_add, args=(torch.ones(2), 3))
>>> ret = fut.wait()
>>> rpc.shutdown()

```



```
>>> # On worker 1:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker1", rank=1, world_size=2)
>>> rpc.shutdown()

```


 torch.distributed.rpc。


 偏僻的


 (*到*，*func*，*args



 =
 


 无*，*kwargs



 =
 


 无
* , *超时



 =
 


 -1.0
* ) [[source]](_modules/torch/distributed/rpc/api.html#remote)[¶](#torch.distributed.rpc.remote "此定义的永久链接")


 远程调用在worker `to`上运行`func`并立即返回一个`RRef`结果值。Worker `to`将是返回的`RRef`的所有者，调用`remote`的worker是一个用户。所有者管理其“RRef”的全局引用计数，并且仅当全局不存在对其的活动引用时，所有者“RRef”才会被破坏。


 参数 
* **to** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*或
* [*WorkerInfo
* ](#torch.distributed.rpc.WorkerInfo "torch.distributed.rpc.WorkerInfo")*或
* [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12)") ) – 目标工作人员的名称/等级/`WorkerInfo`。
* **func** ( *Callable
* ) – 可调用函数，例如 Python 可调用函数、内置运算符(例如 [`add( )`](generated/torch.add.html#torch.add "torch.add") ) 和带注释的TorchScript 函数。
* **args** ( [*tuple*](https://docs.python.org/3 /library/stdtypes.html#tuple "(in Python v3.12)") ) – `func` 调用的参数元组。
* **kwargs** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 是 `func` 调用的关键字参数的字典。
* **timeout** ( [*float*]( https://docs.python.org/3/library/functions.html#float "(Python v3.12)")*,
* *可选
* ) – 此远程调用的超时时间(以秒为单位)。如果在此超时内未在此工作线程上成功处理在工作进程“to”上创建的“RRef”，则下次尝试使用 RRef(例如“to_here()”)时，将引发超时，指示这次失败。值 0 表示无限超时，即永远不会引发超时错误。如果未提供，则使用初始化期间或使用 `_set_rpc_timeout` 设置的默认值。


 退货


 结果值的用户“RRef”实例。使用阻塞 API `torch.distributed.rpc.RRef.to_here()` 在本地检索结果值。


!!! warning "警告"

     “远程” API 不会复制参数tensor的存储，直到通过线路发送它们，这可以通过不同的线程来完成，具体取决于 RPC 后端类型。调用者应确保这些tensor的内容保持完整，直到返回的 RRef 得到所有者确认，可以使用“torch.distributed.rpc.RRef.confirmed_by_owner()” API 进行检查。


!!! warning "警告"

     “远程” API 超时等错误将尽力处理。这意味着当“remote”发起的远程调用失败时，例如出现超时错误，我们会采取尽力而为的方法来处理错误。这意味着错误会在异步的基础上处理并设置在生成的 RRef 上。如果应用程序在此处理之前尚未使用 RRef(例如“to_here”或 fork 调用)，则将来使用“RRef”将相应地引发错误。但是，用户应用程序可能会在处理错误之前使用“RRef”。在这种情况下，可能不会引发错误，因为错误尚未得到处理。


 例子：


```
Make sure that ``MASTER_ADDR`` and ``MASTER_PORT`` are set properly
on both workers. Refer to :meth:`~torch.distributed.init_process_group`
API for more details. For example,

export MASTER_ADDR=localhost
export MASTER_PORT=5678

Then run the following code in two different processes:

>>> # On worker 0:
>>> import torch
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker0", rank=0, world_size=2)
>>> rref1 = rpc.remote("worker1", torch.add, args=(torch.ones(2), 3))
>>> rref2 = rpc.remote("worker1", torch.add, args=(torch.ones(2), 1))
>>> x = rref1.to_here() + rref2.to_here()
>>> rpc.shutdown()

>>> # On worker 1:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker1", rank=1, world_size=2)
>>> rpc.shutdown()

Below is an example of running a TorchScript function using RPC.

>>> # On both workers:
>>> @torch.jit.script
>>> def my_script_add(t1, t2):
>>>    return torch.add(t1, t2)

>>> # On worker 0:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker0", rank=0, world_size=2)
>>> rref = rpc.remote("worker1", my_script_add, args=(torch.ones(2), 3))
>>> rref.to_here()
>>> rpc.shutdown()

>>> # On worker 1:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker1", rank=1, world_size=2)
>>> rpc.shutdown()

```


 torch.distributed.rpc。


 获取_worker_info


 ( *工人_姓名



 =
 


 无
* ) [[source]](_modules/torch/distributed/rpc/api.html#get_worker_info)[¶](#torch.distributed.rpc.get_worker_info "此定义的永久链接")


 获取给定工作人员名称的 [`WorkerInfo`](#torch.distributed.rpc.WorkerInfo "torch.distributed.rpc.WorkerInfo")。使用此 [`WorkerInfo`](#torch.distributed.rpc.WorkerInfo "torch. distribution.rpc.WorkerInfo") 以避免在每次调用时传递昂贵的字符串。


 Parameters


**worker_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – a 的字符串名称工人。如果为 None ，则返回当前工作人员的 id。 (默认“无”)


 退货


给定 `worker_name` 或 [`WorkerInfo`](#torch.distributed.rpc.WorkerInfo "torch" 的 [`WorkerInfo`](#torch.distributed.rpc.WorkerInfo "torch.distributed.rpc.WorkerInfo") 实例如果 `worker_name` 为 `None` ，则为当前工作线程的.distributed.rpc.WorkerInfo") 。


 torch.distributed.rpc。


 关闭


 (*优雅



 =
 


 真*，*超时



 =
 


 0
* ) [[source]](_modules/torch/distributed/rpc/api.html#shutdown)[¶](#torch.distributed.rpc.shutdown "此定义的永久链接")


 关闭 RPC 代理，然后销毁 RPC 代理。这会阻止本地代理接受未完成的请求，并通过终止所有 RPC 线程来关闭 RPC 框架。如果 `graceful=True` ，这将阻塞，直到所有本地和远程 RPC 进程到达此方法并等待所有未完成的工作完成。否则，如果 `graceful=False` ，则这是本地关闭，并且它不会等待其他 RPC 进程到达此方法。


!!! warning "警告"

     对于 [`Future`](futures.html#torch.futures.Future "torch.futures.Future") 由 [`rpc_async()`](#torch.distributed.rpc.rpc_async "torch.distributed. rpc.rpc_async") ， `future.wait()` 不应在 `shutdown()` 之后调用。


 Parameters


**graceful** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否进行正常关闭或不是。如果为 True，这将 1) 等待，直到没有“UserRRefs”的待处理系统消息并将其删除； 2) 阻塞，直到所有本地和远程 RPC 进程都到达此方法，并等待所有未完成的工作完成。


 例子：：


 确保两个工作进程上的“MASTER_ADDR”和“MASTER_PORT”均已正确设置。请参阅 [`init_process_group()`](distributed.html#torch.distributed.init_process_group "torch.distributed.init_process_group") API 了解更多详细信息。例如，


 导出 MASTER_ADDR=localhostexport MASTER_PORT=5678


 然后在两个不同的进程中运行以下代码：


```
>>> # On worker 0:
>>> import torch
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker0", rank=0, world_size=2)
>>> # do some work
>>> result = rpc.rpc_sync("worker1", torch.add, args=(torch.ones(1), 1))
>>> # ready to shutdown
>>> rpc.shutdown()

```



```
>>> # On worker 1:
>>> import torch.distributed.rpc as rpc
>>> rpc.init_rpc("worker1", rank=1, world_size=2)
>>> # wait for worker 0 to finish work, and then shutdown.
>>> rpc.shutdown()

```


*班级*


 torch.distributed.rpc。


 WorkerInfo [¶](#torch.distributed.rpc.WorkerInfo "此定义的永久链接")


 封装系统中worker信息的结构体。包含worker的名称和ID。这个类并不意味着直接构造，而是可以通过 [`get_worker_info()`](#torch.distributed.rpc.get_worker_info "torch.distributed.rpc.get_worker_info") 检索实例，结果可以是传递给诸如 [`rpc_sync()`](#torch.distributed.rpc.rpc_sync "torch.distributed.rpc.rpc_sync") 、 [`rpc_async()`](#torch.distributed. rpc.rpc_async "torch.distributed.rpc.rpc_async") 、 [`remote()`](#torch.distributed.rpc.remote "torch.distributed.rpc.remote") 以避免每次调用时复制字符串。


*财产*


 id [¶](#torch.distributed.rpc.WorkerInfo.id "此定义的永久链接")


 用于识别工人的全局唯一 ID。


*财产*


 name [¶](#torch.distributed.rpc.WorkerInfo.name "此定义的永久链接")


 工人的姓名。


 RPC 包还提供了装饰器，允许应用程序指定如何在被调用方处理给定函数。


 torch.distributed.rpc.functions。


 异步_执行


 ( *fn
* ) [[source]](_modules/torch/distributed/rpc/functions.html#async_execution)[¶](#torch.distributed.rpc.functions.async_execution "此定义的永久链接")


 函数的装饰器，指示函数的返回值保证是一个 [`Future`](futures.html#torch.futures.Future "torch.futures.Future") 对象，并且该函数可以在 RPC 被调用者上异步运行。更具体地说，被调用者提取被包装函数返回的 [`Future`](futures.html#torch.futures.Future "torch.futures.Future") 并将后续处理步骤安装为 [`Future`](futures.html#torch.futures.Future "torch.futures.Future").完成后，安装的回调将从 [`Future`](futures.html#torch.futures.Future "torch.futures.Future") 读取值，并将该值作为 RPC 响应发送回来。这也意味着返回的 [`Future`](futures.html#torch.futures.Future "torch.futures.Future") 仅存在于被调用方，并且永远不会通过 RPC 发送。当包装函数( `fn` )的执行由于包含 [`rpc_async()`](#torch.distributed.rpc.rpc_async "torch.distributed.rpc. rpc_async") 或等待其他信号。




!!! note "笔记"

    要启用异步执行，应用程序必须将此装饰器返回的函数对象传递给 RPC API。如果 RPC 检测到此装饰器安装的属性，它就知道此函数返回一个“Future”对象，并将相应地处理该对象。但是，这并不意味着在定义函数时此装饰器必须是最外层的。例如，当与“@staticmethod”或“@classmethod”结合使用时，“@rpc.functions.async_execution”需要是内部装饰器，以允许目标函数被识别为静态函数或类函数。该目标函数仍然可以异步执行，因为在访问时，静态或类方法会保留由 `@rpc.functions.async_execution` 安装的属性。


 例子：：


 返回的 [`Future`](futures.html#torch.futures.Future "torch.futures.Future") 对象可以来自 [`rpc_async()`](#torch.distributed.rpc.rpc_async "torch. Distribution.rpc.rpc_async") 、 [`then()`](futures.html#torch.futures.Future.then "torch.futures.Future.then") 或 [`Future`](futures.html#torch.futures.Future“torch.futures.Future”)构造函数。下面的示例显示直接使用 [`then()`](futures.html#torch.futures.Future) 返回的 [`Future`](futures.html#torch.futures.Future "torch.futures.Future")。然后“torch.futures.Future.then”)。


```
>>> from torch.distributed import rpc
>>>
>>> # omitting setup and shutdown RPC
>>>
>>> # On all workers
>>> @rpc.functions.async_execution
>>> def async_add_chained(to, x, y, z):
>>>     # This function runs on "worker1" and returns immediately when
>>>     # the callback is installed through the `then(cb)` API. In the
>>>     # mean time, the `rpc_async` to "worker2" can run concurrently.
>>>     # When the return value of that `rpc_async` arrives at
>>>     # "worker1", "worker1" will run the lambda function accordingly
>>>     # and set the value for the previously returned `Future`, which
>>>     # will then trigger RPC to send the result back to "worker0".
>>>     return rpc.rpc_async(to, torch.add, args=(x, y)).then(
>>>         lambda fut: fut.wait() + z
>>>     )
>>>
>>> # On worker0
>>> ret = rpc.rpc_sync(
>>>     "worker1",
>>>     async_add_chained,
>>>     args=("worker2", torch.ones(2), 1, 1)
>>> )
>>> print(ret)  # prints tensor([3., 3.])

```


 与 TorchScript 装饰器结合使用时，该装饰器必须是最外面的装饰器。


```
>>> from torch import Tensor
>>> from torch.futures import Future
>>> from torch.distributed import rpc
>>>
>>> # omitting setup and shutdown RPC
>>>
>>> # On all workers
>>> @torch.jit.script
>>> def script_add(x: Tensor, y: Tensor) -> Tensor:
>>>     return x + y
>>>
>>> @rpc.functions.async_execution
>>> @torch.jit.script
>>> def async_add(to: str, x: Tensor, y: Tensor) -> Future[Tensor]:
>>>     return rpc.rpc_async(to, script_add, (x, y))
>>>
>>> # On worker0
>>> ret = rpc.rpc_sync(
>>>     "worker1",
>>>     async_add,
>>>     args=("worker2", torch.ones(2), 1)
>>> )
>>> print(ret)  # prints tensor([2., 2.])

```


 当与静态或类方法结合使用时，该装饰器必须是内部装饰器。


```
>>> from torch.distributed import rpc
>>>
>>> # omitting setup and shutdown RPC
>>>
>>> # On all workers
>>> class AsyncExecutionClass:
>>>
>>>     @staticmethod
>>>     @rpc.functions.async_execution
>>>     def static_async_add(to, x, y, z):
>>>         return rpc.rpc_async(to, torch.add, args=(x, y)).then(
>>>             lambda fut: fut.wait() + z
>>>         )
>>>
>>>     @classmethod
>>>     @rpc.functions.async_execution
>>>     def class_async_add(cls, to, x, y, z):
>>>         ret_fut = torch.futures.Future()
>>>         rpc.rpc_async(to, torch.add, args=(x, y)).then(
>>>             lambda fut: ret_fut.set_result(fut.wait() + z)
>>>         )
>>>         return ret_fut
>>>
>>>     @rpc.functions.async_execution
>>>     def bound_async_add(self, to, x, y, z):
>>>         return rpc.rpc_async(to, torch.add, args=(x, y)).then(
>>>             lambda fut: fut.wait() + z
>>>         )
>>>
>>> # On worker0
>>> ret = rpc.rpc_sync(
>>>     "worker1",
>>>     AsyncExecutionClass.static_async_add,
>>>     args=("worker2", torch.ones(2), 1, 2)
>>> )
>>> print(ret)  # prints tensor([4., 4.])
>>>
>>> ret = rpc.rpc_sync(
>>>     "worker1",
>>>     AsyncExecutionClass.class_async_add,
>>>     args=("worker2", torch.ones(2), 1, 2)
>>> )
>>> print(ret)  # prints tensor([4., 4.])

```


 该装饰器还可以与 RRef 助手一起使用，即. `torch.distributed.rpc.RRef.rpc_sync()` 、 `torch.distributed.rpc.RRef.rpc_async()` 和 `torch.distributed.rpc.RRef.remote()` 。


```
>>> from torch.distributed import rpc
>>>
>>> # reuse the AsyncExecutionClass class above
>>> rref = rpc.remote("worker1", AsyncExecutionClass)
>>> ret = rref.rpc_sync().static_async_add("worker2", torch.ones(2), 1, 2)
>>> print(ret)  # prints tensor([4., 4.])
>>>
>>> rref = rpc.remote("worker1", AsyncExecutionClass)
>>> ret = rref.rpc_async().static_async_add("worker2", torch.ones(2), 1, 2).wait()
>>> print(ret)  # prints tensor([4., 4.])
>>>
>>> rref = rpc.remote("worker1", AsyncExecutionClass)
>>> ret = rref.remote().static_async_add("worker2", torch.ones(2), 1, 2).to_here()
>>> print(ret)  # prints tensor([4., 4.])

```


### 后端 [¶](#backends "此标题的永久链接")


 RPC模块可以利用不同的后端来执行节点之间的通信。可以在 [`init_rpc()`](#torch.distributed.rpc.init_rpc "torch.distributed.rpc.init_rpc") 函数中通过传递 [`BackendType` 的特定值来指定要使用的后端](#torch.distributed.rpc.BackendType "torch.distributed.rpc.BackendType") 枚举。无论使用什么后端，RPC API 的其余部分都不会改变。每个后端还定义了自己的 [`RpcBackendOptions`](#torch.distributed.rpc.RpcBackendOptions "torch.distributed.rpc.RpcBackendOptions") 类的子类，该类的实例也可以传递给 [`init_rpc()` ](#torch.distributed.rpc.init_rpc "torch.distributed.rpc.init_rpc") 配置后端的行为。


*班级*


 torch.distributed.rpc。


 后端类型


 ( *value
* ) [¶](#torch.distributed.rpc.BackendType "此定义的永久链接")


 可用后端的枚举类。


 PyTorch 附带内置的“BackendType.TENSORPIPE”后端。可以使用“register_backend()”函数注册其他后端。


*班级*


 torch.distributed.rpc。


 RpcBackendOptions [¶](#torch.distributed.rpc.RpcBackendOptions "此定义的永久链接")


 封装传递到 RPC 后端的选项的抽象结构。可以将此类的实例传入 [`init_rpc()`](#torch.distributed.rpc.init_rpc "torch.distributed.rpc.init_rpc") 以便使用特定配置初始化 RPC，例如 RPC要使用的超时和“init_method”。


*财产*


 init_method [¶](#torch.distributed.rpc.RpcBackendOptions.init_method "此定义的永久链接")


 指定如何初始化进程组的 URL。默认为 `env://`


*财产*


 rpc_timeout [¶](#torch.distributed.rpc.RpcBackendOptions.rpc_timeout "此定义的永久链接")


 指示用于所有 RPC 的超时的浮点数。如果 RPC 在此时间范围内未完成，它将完成并出现异常，表明它已超时。


#### TensorPipe 后端 [¶](#tensorpipe-backend "此标题的永久链接")


 默认情况下的 TensorPipe 代理利用 [TensorPipe 库](https://github.com/pytorch/tensorpipe)，它提供了专门适合机器学习的本机点对点通信原语，从根本上解决了一些限制格洛。与 Gloo 相比，它的优点是异步，允许大量传输同时发生，每个传输都以自己的速度进行，而不会相互阻塞。它只会在需要时按需打开一对节点之间的管道，并且当一个节点发生故障时，仅其事件管道将被关闭，而所有其他节点将继续正常工作。此外，它能够支持多种不同的传输(当然还有 TCP，还有共享内存、NVLink、InfiniBand 等)，并且可以自动检测它们的可用性并协商每个管道使用的最佳传输。


 TensorPipe 后端已在 PyTorch v1.6 中引入，并且正在积极开发中。目前，它仅支持 CPU tensor，GPU 支持即将推出。它配备了基于 TCP 的传输，就像 Gloo 一样。它还能够在多个套接字和线程上自动对大tensor进行分块和复用，以实现非常高的带宽。代理将能够自行选择最佳运输方式，无需干预。


 例子：


```
>>> import os
>>> from torch.distributed import rpc
>>> os.environ['MASTER_ADDR'] = 'localhost'
>>> os.environ['MASTER_PORT'] = '29500'
>>>
>>> rpc.init_rpc(
>>>     "worker1",
>>>     rank=0,
>>>     world_size=2,
>>>     rpc_backend_options=rpc.TensorPipeRpcBackendOptions(
>>>         num_worker_threads=8,
>>>         rpc_timeout=20 # 20 second timeout
>>>     )
>>> )
>>>
>>> # omitting init_rpc invocation on worker2

```


*班级*


 torch.distributed.rpc。


 TensorPipeRpc后端选项


 ( *** , *num_worker_threads



 =
 


 16
* , *rpc_超时



 =
 


 60.0
* , *初始化_方法



 =
 


 'env://'
* , *device_maps



 =
 


 无
* , *设备



 =
 


 无
* , *_transports



 =
 


 无
* , *_channels



 =
 


 无
* ) [[source]](_modules/torch/distributed/rpc/options.html#TensorPipeRpcBackendOptions)[¶](#torch.distributed.rpc.TensorPipeRpcBackendOptions "此定义的永久链接")


 `TensorPipeAgent` 的后端选项，派生自 [`RpcBackendOptions`](#torch.distributed.rpc.RpcBackendOptions "torch.distributed.rpc.RpcBackendOptions") 。


 参数 
* **num_worker_threads** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*,
* *可选
* ) – `TensorPipeAgent` 用于执行请求的线程池中的线程数(默认值：16)。
* **rpc_timeout** ( [*float*](https://docs.python.org /3/library/functions.html#float "(Python v3.12)")*,
* *可选
* ) – RPC 请求的默认超时(以秒为单位)(默认值：60 秒)。如果 RPC 在此时间范围内未完成，则会引发异常。调用者可以在 [`rpc_sync()`](#torch.distributed.rpc.rpc_sync "torch.distributed.rpc.rpc_sync") 和 [`rpc_async()`](#torch.如有必要，distributed.rpc.rpc_async "torch.distributed.rpc.rpc_async")。
* **init_method** ( [*str*](https://docs.python.org/3/library/stdtypes.html #str "(Python v3.12)")*,
* *可选
* ) – 用于初始化用于集合的分布式存储的 URL。它采用 [`init_process_group()`](distributed.html#torch.distributed.init_process_group "torch.distributed.init_process_group") 的相同参数接受的任何值(默认值： `env://` )。
* **device_maps** ( *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")
* ,
* *Dict
* *]
* *,
* *可选
* ) – 从该工作线程到被调用者的设备布局映射。键是被调用者工作人员名称和值字典(`int`、`str` 或 `torch.device` 的 `Dict` )，该字典将此工作人员的设备映射到被调用者工作人员的设备。(默认值：`None` )
* ** devices** (List[int, str, or `torch.device` ], 可选) – RPC 代理使用的所有本地CUDA 设备。默认情况下，它将被初始化为来自其自己的“device_maps”的所有本地设备以及来自其对等方的“device_maps”的相应设备。当处理 CUDA RPC 请求时，代理将正确同步此“列表”中所有设备的 CUDA 流。


*财产*


 device_maps [¶](#torch.distributed.rpc.TensorPipeRpcBackendOptions.device_maps "此定义的永久链接")


 设备地图位置。


*财产*


 devices [¶](#torch.distributed.rpc.TensorPipeRpcBackendOptions.devices"此定义的永久链接")


 本地代理使用的所有设备。


*财产*


 init_method [¶](#torch.distributed.rpc.TensorPipeRpcBackendOptions.init_method "此定义的永久链接")


 指定如何初始化进程组的 URL。默认为 `env://`


*财产*


 num_worker_threads [¶](#torch.distributed.rpc.TensorPipeRpcBackendOptions.num_worker_threads "此定义的永久链接")


 `TensorPipeAgent` 用于执行请求的线程池中的线程数。


*财产*


 rpc_timeout [¶](#torch.distributed.rpc.TensorPipeRpcBackendOptions.rpc_timeout "此定义的永久链接")


 指示用于所有 RPC 的超时的浮点数。如果 RPC 在此时间范围内未完成，它将完成并出现异常，表明它已超时。


 设置_设备_地图


 ( *to
* , *device_map
* ) [[source]](_modules/torch/distributed/rpc/options.html#TensorPipeRpcBackendOptions.set_device_map)[¶](#torch.distributed.rpc.TensorPipeRpcBackendOptions.set_device_map "永久链接这个定义")


 设置每个 RPC 调用者和被调用者对之间的设备映射。可以多次调用此函数以增量添加设备放置配置。


 参数 
* **to** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 被调用方名称。
* 
* *device_map** ( *Dict
* *of
* [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*, 
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*, 或
* [*torch.device*](tensor_attributes. html#torch.device "torch.device") ) – 从该工作线程到被调用者的设备放置映射。该地图必须是可反转的。


 例子


```
>>> # both workers
>>> def add(x, y):
>>>     print(x)  # tensor([1., 1.], device='cuda:1')
>>>     return x + y, (x + y).to(2)
>>>
>>> # on worker 0
>>> options = TensorPipeRpcBackendOptions(
>>>     num_worker_threads=8,
>>>     device_maps={"worker1": {0: 1}}
>>>     # maps worker0's cuda:0 to worker1's cuda:1
>>> )
>>> options.set_device_map("worker1", {1: 2})
>>> # maps worker0's cuda:1 to worker1's cuda:2
>>>
>>> rpc.init_rpc(
>>>     "worker0",
>>>     rank=0,
>>>     world_size=2,
>>>     backend=rpc.BackendType.TENSORPIPE,
>>>     rpc_backend_options=options
>>> )
>>>
>>> x = torch.ones(2)
>>> rets = rpc.rpc_sync("worker1", add, args=(x.to(0), 1))
>>> # The first argument will be moved to cuda:1 on worker1. When
>>> # sending the return value back, it will follow the invert of
>>> # the device map, and hence will be moved back to cuda:0 and
>>> # cuda:1 on worker0
>>> print(rets[0])  # tensor([2., 2.], device='cuda:0')
>>> print(rets[1])  # tensor([2., 2.], device='cuda:1')

```


 设置设备


 ( *devices
* ) [[source]](_modules/torch/distributed/rpc/options.html#TensorPipeRpcBackendOptions.set_devices)[¶](#torch.distributed.rpc.TensorPipeRpcBackendOptions.set_devices "此定义的永久链接")


 设置 TensorPipe RPC 代理使用的本地设备。当处理 CUDA RPC 请求时，TensorPipe RPC 代理将正确同步此“List”中所有设备的 CUDA 流。


 Parameters


**设备** ( *List
* *of
* [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*, 或
* [*torch.device*](tensor_attributes.html #torch.device "torch.device") ) – TensorPipe RPC 代理使用的本地设备。




!!! note "笔记"

    RPC 框架不会自动重试任何 [`rpc_sync()`](#torch.distributed.rpc.rpc_sync "torch.distributed.rpc.rpc_sync") 、 [`rpc_async()`](#torch. distribution.rpc.rpc_async "torch.distributed.rpc.rpc_async") 和 [`remote()`](#torch.distributed.rpc.remote "torch.distributed.rpc.remote") 调用。原因是 RPC 框架无法确定操作是否幂等以及重试是否安全。因此，应用程序有责任处理失败并在必要时重试。 RPC 通信基于 TCP，因此可能由于网络故障或间歇性网络连接问题而发生故障。在这种情况下，应用程序需要通过合理的退避来适当地重试，以确保网络不会因激进的重试而不堪重负。


## RRef [¶](#rref "此标题的永久链接")


!!! warning "警告"

     使用 CUDA tensor时当前不支持 RRef


 “RRef”(远程引用)是对远程工作人员上某种类型“T”(例如“Tensor”)的值的引用。此句柄使引用的远程值在所有者上保持活动状态，但并不意味着该值将来会传输到本地工作人员。 RRefs 可以通过保存对其他工作器上存在的 [nn.Modules](https://pytorch.org/docs/stable/nn.html#torch.nn.Module) 的引用并调用适当的函数来用于多机训练在训练期间检索或修改其参数。有关更多详细信息，请参阅[远程引用协议](rpc/rref.html#remote-reference-protocol)。


*班级*


 torch.distributed.rpc。


 吡啶参考文献


 ( *RRef
* ) [¶](#torch.distributed.rpc.PyRRef "此定义的永久链接")


 封装对远程工作人员上某种类型值的引用的类。该句柄将使引用的远程值在工作人员上保持活动状态。当 1) 应用程序代码和本地 RRef 上下文中都没有对它的引用，或者 2) 应用程序调用了正常关闭时，“UserRRef”将被删除。对已删除的 RRef 调用方法会导致未定义的行为。 RRef 实现仅提供尽力而为的错误检测，应用程序不应在 `rpc.shutdown()` 之后使用 `UserRRefs` 。


!!! warning "警告"

     RRef 只能通过 RPC 模块进行序列化和反序列化。无需 RPC 即可序列化和反序列化 RRef(例如，Pythonpickle、torch [`save()`](generated/torch.save.html#torch.save "torch.save") /[`load()`](generated/torch.load.html#torch.load "torch.load") ,JIT [`save()`](generated/torch.jit.save.html#torch.jit.save "torch.jit.save") /[`load()`](generated/torch.jit.load.html#torch.jit.load "torch.jit.load") 等)会导致错误。


 参数 
* **value** ( [*object*](https://docs.python.org/3/library/functions.html#object "(in Python v3.12)") ) – 要包装的值通过此 RRef.
* **type_hint** ( *Type
* *,
* *可选
* ) – 应将 Python 类型传递给 `TorchScript` 编译器作为 `value` 的类型提示。


 例子：：


 为了简单起见，以下示例跳过 RPC 初始化和关闭代码。有关这些详细信息，请参阅 RPC 文档。


1.使用rpc.remote创建RRef


```
>>> import torch
>>> import torch.distributed.rpc as rpc
>>> rref = rpc.remote("worker1", torch.add, args=(torch.ones(2), 3))
>>> # get a copy of value from the RRef
>>> x = rref.to_here()

```


2. 从本地对象创建 RRef


```
>>> import torch
>>> from torch.distributed.rpc import RRef
>>> x = torch.zeros(2, 2)
>>> rref = RRef(x)

```


3. 与其他工作人员共享 RRef


```
>>> # On both worker0 and worker1:
>>> def f(rref):
>>>   return rref.to_here() + 1

```



```
>>> # On worker0:
>>> import torch
>>> import torch.distributed.rpc as rpc
>>> from torch.distributed.rpc import RRef
>>> rref = RRef(torch.zeros(2, 2))
>>> # the following RPC shares the rref with worker1, reference
>>> # count is automatically updated.
>>> rpc.rpc_sync("worker1", f, args=(rref,))

```


 落后


 ( *自己



 :
 


[火炬._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef "火炬._C._distributed_rpc.PyRRef")
* , *dist_autograd_ctx_id



 :
 


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 =
 


 -1
* , *保留_graph



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 =
 


 错误的
* )


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.distributed.rpc.PyRRef.backward"此定义的永久链接")



> 
> 
> 
> 使用 RRef 作为向后传递的根来运行向后传递。如果提供了`dist_autograd_ctx_id`>，>我们使用提供的
> ctx_id 从 RRef 的所有者开始执行分布式向后传递。在这种情况下，
> [`get_gradients()`](#torch.distributed.autograd.get_gradients "torch.distributed.autograd.get_gradients")
> 应该
> 用于检索梯度。如果
> `dist_autograd_ctx_id`
> is
> `None`
> ，则假设这是一个局部自动梯度图>并且我们只执行局部向后传递。在本地情况下，
> 调用此 API 的节点必须是 RRef 的所有者。
> RRef 的值预计为标量 Tensor。
> 
> 
> 
> >


 参数 
* **dist_autograd_ctx_id** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")
* ,
* *可选
* ) – 我们应该检索梯度的分布式自动梯度上下文 ID (默认值: -1).
* **retain_graph** ( [*bool*](https://docs.python.org/3 /library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `False` ，用于计算梯度的图将被释放。请注意，在几乎所有情况下，都不需要将此选项设置为“True”，并且通常可以通过更有效的方式解决。通常，您需要将其设置为“True”以向后运行多次(默认值：False)。


 例子：：




```
>>> import torch.distributed.autograd as dist_autograd
>>> with dist_autograd.context() as context_id:
>>>     rref.backward(context_id)

```


 由所有者确认


 ( *自己



 :
 


[火炬._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef“火炬._C._distributed_rpc.PyRRef”)*)


 → [bool](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)")


[¶](#torch.distributed.rpc.PyRRef.confirmed_by_owner"此定义的永久链接")


 返回此“RRef”是否已被所有者确认。 `OwnerRRef` 总是返回 true，而 `UserRRef` 仅当所有者知道这个 `UserRRef` 时才返回 true。


 是_所有者


 ( *自己



 :
 


[火炬._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef“火炬._C._distributed_rpc.PyRRef”)*)


 → [bool](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)")


[¶](#torch.distributed.rpc.PyRRef.is_owner"此定义的永久链接")


 返回当前节点是否是此“RRef”的所有者。


 本地_值


 ( *自己



 :
 


[火炬._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef“火炬._C._distributed_rpc.PyRRef”)*)


 → [对象](https://docs.python.org/3/library/functions.html#object "(在 Python v3.12 中)")


[¶](#torch.distributed.rpc.PyRRef.local_value"此定义的永久链接")


 如果当前节点是所有者，则返回对本地值的引用。否则，抛出异常。


 owner
 


 ( *自己



 :
 


[火炬._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef“火炬._C._distributed_rpc.PyRRef”)*)


 → [torch._C._distributed_rpc.WorkerInfo](#torch.distributed.rpc.WorkerInfo "torch._C._distributed_rpc.WorkerInfo")


[¶](#torch.distributed.rpc.PyRRef.owner"此定义的永久链接")


 返回拥有此“RRef”的节点的工作人员信息。


 所有者_name


 ( *自己



 :
 


[火炬._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef“火炬._C._distributed_rpc.PyRRef”)*)


 → [str](https://docs.python.org/3/library/stdtypes.html#str“(Python v3.12)”)


[¶](#torch.distributed.rpc.PyRRef.owner_name"此定义的永久链接")


 返回拥有此“RRef”的节点的工作人员名称。


 偏僻的


 ( *自己



 :
 


[torch._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef "torch._C._distributed_rpc.PyRRef")
* , *超时



 :
 


[float](https://docs.python.org/3/library/functions.html#float“(在Python v3.12中)”)


 =
 


 -1.0
* )


 → [对象](https://docs.python.org/3/library/functions.html#object "(在 Python v3.12 中)")


[¶](#torch.distributed.rpc.PyRRef.remote"此定义的永久链接")


 创建一个辅助代理，以使用 RRef 的所有者作为目标来轻松启动“远程”，以便在此 RRef 引用的对象上运行函数。更具体地说， `rref.remote().func_name(*args, **kwargs)` 与以下内容相同：


```
>>> def run(rref, func_name, args, kwargs):
>>>   return getattr(rref.local_value(), func_name)(*args, **kwargs)
>>>
>>> rpc.remote(rref.owner(), run, args=(rref, func_name, args, kwargs))

```


 Parameters


**超时** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*,
* *可选
* ) – 超时对于 `rref.remote()` 。如果在超时内未成功完成此“RRef”的创建，则下次尝试使用 RRef(例如“to_here”)时，将引发超时。如果未提供，将使用默认的 RPC 超时。请参阅“rpc.remote()”以了解“RRef”的特定超时语义。


 例子：：




```
>>> from torch.distributed import rpc
>>> rref = rpc.remote("worker1", torch.add, args=(torch.zeros(2, 2), 1))
>>> rref.remote().size().to_here()  # returns torch.Size([2, 2])
>>> rref.remote().view(1, 4).to_here()  # returns tensor([[1., 1., 1., 1.]])

```


 rpc_async


 ( *自己



 :
 


[torch._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef "torch._C._distributed_rpc.PyRRef")
* , *超时



 :
 


[float](https://docs.python.org/3/library/functions.html#float“(在Python v3.12中)”)


 =
 


 -1.0
* )


 → [对象](https://docs.python.org/3/library/functions.html#object "(在 Python v3.12 中)")


[¶](#torch.distributed.rpc.PyRRef.rpc_async"此定义的永久链接")


 创建一个帮助器代理，以使用 RRef 的所有者作为目标来轻松启动“rpc_async”，以便在此 RRef 引用的对象上运行函数。更具体地说， `rref.rpc_async().func_name(*args, **kwargs)` 与以下内容相同：


```
>>> def run(rref, func_name, args, kwargs):
>>>   return getattr(rref.local_value(), func_name)(*args, **kwargs)
>>>
>>> rpc.rpc_async(rref.owner(), run, args=(rref, func_name, args, kwargs))

```


 Parameters


**超时** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*,
* *可选
* ) – 超时对于 `rref.rpc_async()` 。如果调用未在此时间范围内完成，则会引发异常。如果未提供此参数，则将使用默认的 RPC 超时。


 例子：：




```
>>> from torch.distributed import rpc
>>> rref = rpc.remote("worker1", torch.add, args=(torch.zeros(2, 2), 1))
>>> rref.rpc_async().size().wait()  # returns torch.Size([2, 2])
>>> rref.rpc_async().view(1, 4).wait()  # returns tensor([[1., 1., 1., 1.]])

```


 rpc_sync


 ( *自己



 :
 


[torch._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef "torch._C._distributed_rpc.PyRRef")
* , *超时



 :
 


[float](https://docs.python.org/3/library/functions.html#float“(在Python v3.12中)”)


 =
 


 -1.0
* )


 → [对象](https://docs.python.org/3/library/functions.html#object "(在 Python v3.12 中)")


[¶](#torch.distributed.rpc.PyRRef.rpc_sync"此定义的永久链接")


 创建一个帮助器代理，以使用 RRef 的所有者作为目标来轻松启动“rpc_sync”，以便在此 RRef 引用的对象上运行函数。更具体地说， `rref.rpc_sync().func_name(*args, **kwargs)` 与以下内容相同：


```
>>> def run(rref, func_name, args, kwargs):
>>>   return getattr(rref.local_value(), func_name)(*args, **kwargs)
>>>
>>> rpc.rpc_sync(rref.owner(), run, args=(rref, func_name, args, kwargs))

```


 Parameters


**超时** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*,
* *可选
* ) – 超时对于 `rref.rpc_sync()` 。如果调用未在此时间范围内完成，则会引发异常。如果未提供此参数，则将使用默认的 RPC 超时。


 例子：：




```
>>> from torch.distributed import rpc
>>> rref = rpc.remote("worker1", torch.add, args=(torch.zeros(2, 2), 1))
>>> rref.rpc_sync().size()  # returns torch.Size([2, 2])
>>> rref.rpc_sync().view(1, 4)  # returns tensor([[1., 1., 1., 1.]])

```


 到这里


 ( *自己



 :
 


[torch._C._distributed_rpc.PyRRef](#torch.distributed.rpc.PyRRef "torch._C._distributed_rpc.PyRRef")
* , *超时



 :
 


[float](https://docs.python.org/3/library/functions.html#float“(在Python v3.12中)”)


 =
 


 -1.0
* )


 → [对象](https://docs.python.org/3/library/functions.html#object "(在 Python v3.12 中)")


[¶](#torch.distributed.rpc.PyRRef.to_here"此定义的永久链接")


 将 RRef 的值从所有者复制到本地节点并返回它的阻塞调用。如果当前节点是所有者，则返回对本地值的引用。


 Parameters


**超时** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*,
* *可选
* ) – 超时对于 `to_here` 。如果调用未在此时间范围内完成，则会引发异常。如果未提供此参数，则将使用默认的 RPC 超时(60 秒)。


 有关 RRef 的更多信息



* [远程引用协议](rpc/rref.html) 
+ [背景](rpc/rref.html#background) 
+ [假设](rpc/rref.html#asstitutions) 
+ [RRef 生命周期](rpc/rref.html #rref-lifetime) 
- [设计推理](rpc/rref.html#design-reasoning) 
- [实现](rpc/rref.html#implementation) 
+ [协议场景](rpc/rref.html#protocol-scenarios) 
- [用户与所有者共享 RRef 作为返回值](rpc/rref.html#user-share-rref-with-owner-as-return-value) 
- [用户与所有者共享 RRef 作为参数](rpc/rref.html #user-share-rref-with-owner-as-argument) 
- [所有者与用户共享 RRef](rpc/rref.html#owner-share-rref-with-user) 
- [用户与用户共享 RRef](rpc /rref.html#user-share-rref-with-user)


## RemoteModule [¶](#remotemodule "此标题的永久链接")


!!! warning "警告"

     使用 CUDA tensor时当前不支持 RemoteModule


`RemoteModule` 是在不同进程上远程创建 nn.Module 的简单方法。实际的模块驻留在远程主机上，但本地主机拥有该模块的句柄，并以类似于常规 nn.Module 的方式调用该模块。但是，该调用会引发对远程端的 RPC 调用，并且如果需要，可以通过支持的其他 API 异步执行通过远程模块。


*班级*


 torch.distributed.nn.api.remote_module。


 远程模块


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/distributed/nn/api/remote_module.html#RemoteModule)[¶](#torch.distributed.nn.api.remote_module.RemoteModule "此定义的永久链接")



> 
> 
> 
> RemoteModule 实例只能在 RPC 初始化后创建。
> 它在指定的远程节点上创建用户指定的模块。
> 它的行为类似于常规的
> `nn.Module`>，除了 
> `forward`>方法>在远程节点上执行。>它负责自动梯度记录，以确保向后传播传播>梯度回到相应的远程模块。
> 
> 
> 
> >它生成两个方法>`forward_async`>和>` forward`
> 基于 
> `module_cls`
> 的 
> `forward`
> 方法的签名。
> `forward_async`
> 异步运行并返回 Future。 
> `forward_async`
> 和
> `forward`
> 的参数与 
> `module_cls`
> 返回的 module
> 的 
> `forward`
> 方法相同。
> 
> 
> 
> 
> 例如， if
> `module_cls`
> 返回一个 
> `nn.Linear`
> 的实例，
> 具有 
> `forward`
> 方法签名：
> `def
> 
> >forward(input:
> 
> 
> Tensor)
> 
> 
> 
- >> 
> 
> Tensor:`
> ,
> 生成的
> `RemoteModule`
> 将有 2 个方法，其签名为：
> 
> 
> 
> 
> 
> `def
> 
> >forward(input:
> 
> 
> Tensor)
> 
> 
> -
> 
> 
> 
> tensor:`
> 
> 
> `def
> 
> 
> 前向_async(输入:
> 
> 
> tensor)
> 
> 
> ->> 
> 
> 未来[tensor]:`
> 
> 
> >


 参数 
* **remote_device** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 设备上我们要放置此模块的目标工作程序。格式应为“<workername>/<device>”，其中设备字段可以解析为 torch.device 类型。例如，“trainer0/cpu”、“trainer0” ，“ps0/cuda:0”。此外，设备字段可以是可选的，默认值为“cpu”。
* **module_cls** ( [*nn.Module*](generated/torch.nn. Module.html#torch.nn.Module "torch.nn.Module") ) –


 要远程创建的模块的类。例如，


```
>>> class MyModule(nn.Module):
>>>     def forward(input):
>>>         return input + 1
>>>
>>> module_cls = MyModule

```
* **args** 
 (
 *Sequence* 
*,* 
*optional* 
 ) – args to be passed to
 `module_cls`
 .
* **kwargs** 
 (
 *Dict* 
*,* 
*optional* 
 ) – kwargs to be passed to
 `module_cls`
 .


 退货


 一个远程模块实例，它包装由用户提供的“module_cls”创建的“Module”，它具有阻塞“forward”方法和异步“forward_async”方法，该方法返回用户上“forward”调用的未来 -在远程端提供模块。


 例子：：


 在两个不同的进程中运行以下代码：


```
>>> # On worker 0:
>>> import torch
>>> import torch.distributed.rpc as rpc
>>> from torch import nn, Tensor
>>> from torch.distributed.nn.api.remote_module import RemoteModule
>>>
>>> rpc.init_rpc("worker0", rank=0, world_size=2)
>>> remote_linear_module = RemoteModule(
>>>     "worker1/cpu", nn.Linear, args=(20, 30),
>>> )
>>> input = torch.randn(128, 20)
>>> ret_fut = remote_linear_module.forward_async(input)
>>> ret = ret_fut.wait()
>>> rpc.shutdown()

```



```
>>> # On worker 1:
>>> import torch
>>> import torch.distributed.rpc as rpc
>>>
>>> rpc.init_rpc("worker1", rank=1, world_size=2)
>>> rpc.shutdown()

```


 此外，可以在这个[教程]( https://pytorch.org/tutorials/advanced/rpc_ddp_tutorial.html)。


 获取_模块_rref


 ( ) [¶](#torch.distributed.nn.api.remote_module.RemoteModule.get_module_rref "此定义的永久链接")


 返回指向远程模块的“RRef”(“RRef[nn.Module]”)。


 Return type


*RRef
* [ [*模块*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.modules.module.Module") ]


 远程_参数


 (*递归



 =
 


 True
* ) [¶](#torch.distributed.nn.api.remote_module.RemoteModule.remote_parameters "此定义的永久链接")


 返回指向远程模块参数的“RRef”列表。这通常可以与 [`DistributedOptimizer`](distributed.optim.html#torch.distributed.optim.DistributedOptimizer "torch.distributed.optim.DistributedOptimizer") 结合使用。


 Parameters


**recurse** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果为 True，则返回参数远程模块和远程模块的所有子模块。否则，仅返回远程模块直接成员的参数。


 退货


 远程模块参数的“RRef”列表(“List[RRef[nn.Parameter]]”)。


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ *RRef
* [ [*Parameter*](generated/torch.nn.parameter.Parameter.html#torch.nn.parameter.Parameter "torch.nn.parameter.Parameter") ]]


## 分布式 Autograd 框架 [¶](#distributed-autograd-framework "此标题的永久链接")


!!! warning "警告"

     使用 CUDA tensor时，当前不支持分布式 autograd


 该模块提供了一个基于RPC的分布式autograd框架，可用于模型并行训练等应用。简而言之，应用程序可以通过 RPC 发送和接收梯度记录tensor。在前向传递中，我们记录何时通过 RPC 发送梯度记录tensor，在后向传递中，我们使用此信息来使用 RPC 执行分布式后向传递。有关更多详细信息，请参阅[分布式 Autograd 设计](rpc/distributed_autograd.html#distributed-autograd-design) 。


 torch.distributed.autograd。


 落后


 (*上下文_id



 :
 


[int](https://docs.python.org/3/library/functions.html#int "(Python v3.12)")
* , *roots



 :
 


 List
 


 [ [tensor](tensors.html#torch.Tensor "torch.Tensor")


 ]
* , *保留_graph



 =
 


 错误的
* )


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.distributed.autograd.backward"此定义的永久链接")


 使用提供的根开始分布式向后传递。目前，它实现了 [FAST 模式算法](rpc/distributed_autograd.html#fast-mode-algorithm)，该算法假设在同一分布式 autograd 上下文中跨工作线程发送的所有 RPC 消息将在向后传递期间成为 autograd 图的一部分。


 我们使用提供的根来发现 autograd 图并计算适当的依赖关系。此方法会阻塞，直到整个 autograd 计算完成。


 我们在每个节点上适当的 [`torch.distributed.autograd.context`](#torch.distributed.autograd.context "torch.distributed.autograd.context") 中累积梯度。根据 [`torch.distributed.autograd.backward()`](#torch.distributed.autograd.backward "torch.distributed.autograd.backward" 时传入的 `context_id` 查找要使用的 autogradcontext “) 叫做。如果没有与给定 ID 对应的有效 autograd 上下文，我们会抛出错误。您可以使用 [`get_gradients()`](#torch.distributed.autograd.get_gradients "torch.distributed.autograd.get_gradients") API 检索累积的梯度。


 参数 
* **context_id** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – autograd 上下文我们应该检索梯度的 id。
* **roots** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)" ) ) – 代表自动梯度计算的根的tensor。所有tensor都应该是标量。
* **retain_graph** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)" )*,
* *可选
* ) – 如果为 False，则用于计算梯度的图形将被释放。请注意，几乎在所有情况下，都不需要将此选项设置为 True，并且通常可以通过更有效的方式解决。通常，您需要将其设置为 True 才能向后运行多次。


 例子：：




```
>>> import torch.distributed.autograd as dist_autograd
>>> with dist_autograd.context() as context_id:
>>>     pred = model.forward()
>>>     loss = loss_func(pred, loss)
>>>     dist_autograd.backward(context_id, loss)

```


*班级*


 torch.distributed.autograd。


 context [[source]](_modules/torch/distributed/autograd.html#context)[¶](#torch.distributed.autograd.context "此定义的永久链接")


 使用分布式自动分级时用于包装前向和后向传递的上下文对象。在“with”语句中生成的“context_id”需要唯一地标识所有工作人员的分布式后向传递。每个工作线程都存储与此 `context_id` 关联的元数据，这是正确执行分布式 autograd 过程所必需的。


 例子：：




```
>>> import torch.distributed.autograd as dist_autograd
>>> with dist_autograd.context() as context_id:
>>>     t1 = torch.rand((3, 3), requires_grad=True)
>>>     t2 = torch.rand((3, 3), requires_grad=True)
>>>     loss = rpc.rpc_sync("worker1", torch.add, args=(t1, t2)).sum()
>>>     dist_autograd.backward(context_id, [loss])

```


 torch.distributed.autograd。


 获取_梯度


 (*上下文_id



 :
 


[int](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)")
* )


 →
 


 Dict
 


 [ [tensor](tensors.html#torch.Tensor "torch.Tensor")


 ,
 


[tensor](tensors.html#torch.Tensor "torch.Tensor")


 ]
 


[¶](#torch.distributed.autograd.get_gradients"此定义的永久链接")


 检索从 Tensor 到对应于给定“context_id”所提供的上下文中累积的 Tensor 的适当梯度的映射，作为分布式自动梯度向后传递的一部分。


 Parameters


**context_id** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – autograd 上下文 id我们应该检索梯度。


 退货


 一个映射，其中键是tensor，值是该tensor的关联梯度。


 例子：：




```
>>> import torch.distributed.autograd as dist_autograd
>>> with dist_autograd.context() as context_id:
>>>     t1 = torch.rand((3, 3), requires_grad=True)
>>>     t2 = torch.rand((3, 3), requires_grad=True)
>>>     loss = t1 + t2
>>>     dist_autograd.backward(context_id, [loss.sum()])
>>>     grads = dist_autograd.get_gradients(context_id)
>>>     print(grads[t1])
>>>     print(grads[t2])

```


 有关 RPC Autograd 的更多信息



* [分布式 Autograd 设计](rpc/distributed_autograd.html) 
+ [背景](rpc/distributed_autograd.html#background) 
+ [前向传播过程中的 Autograd 记录](rpc/distributed_autograd.html#autograd-recording-during-the-前向传递) 
+ [分布式 Autograd 上下文](rpc/distributed_autograd.html#distributed-autograd-context) 
+ [分布式后向传递](rpc/distributed_autograd.html#distributed-backward-pass) 
- [计算依赖项](rpc/distribution_autograd.html#computing-dependencies) 
- [FAST 模式算法](rpc/distributed_autograd.html#fast-mode-algorithm) 
- [SMART 模式算法](rpc/distributed_autograd.html#smart-mode-algorithm) 
+ [分布式优化器](rpc/distributed_autograd.html#distributed-optimizer) 
+ [简单的端到端示例](rpc/distributed_autograd.html#simple-end-to-end-example)


## 分布式优化器 [¶](#distributed-optimizer "此标题的固定链接")


 有关分布式优化器的文档，请参阅 [torch.distributed.optim](https://pytorch.org/docs/main/distributed.optim.html) 页面。


## 设计说明[¶](#design-notes"此标题的永久链接")


 分布式 autograd 设计说明涵盖了基于 RPC 的分布式 autograd 框架的设计，该框架对于模型并行训练等应用非常有用。



* [分布式Autograd设计](rpc/distributed_autograd.html#distributed-autograd-design)


 RRef 设计说明涵盖了 [RRef](#rref)(远程引用)协议的设计，该协议用于通过框架引用远程工作人员的值。



* [远程引用协议](rpc/rref.html#remote-reference-protocol)


## 教程 [¶](#tutorials "此标题的永久链接")


 RPC 教程向用户介绍 RPC 框架，提供几个使用 [torch.distributed.rpc](#distributed-rpc-framework) API 的示例应用程序，并演示如何使用 [分析器](https://pytorch.org/docs/stable/autograd.html#profiler) 来分析基于 RPC 的工作负载。



* [分布式 RPC 框架入门](https://pytorch.org/tutorials/intermediate/rpc_tutorial.html)
* [使用分布式 RPC 框架实现参数服务器](https://pytorch.org/tutorials/intermediate/rpc_param_server_tutorial.html)
* [将分布式 DataParallel 与分布式 RPC 框架相结合](https://pytorch.org/tutorials/advanced/rpc_ddp_tutorial.html)(也涵盖 **RemoteModule**)
* [分析基于 RPC 的工作负载]( https://pytorch.org/tutorials/recipes/distributed_rpc_profiling.html)
* [实现批量RPC处理](https://pytorch.org/tutorials/intermediate/rpc_async_execution.html)
* [分布式管道并行](https://pytorch.org/tutorials/intermediate/dist_pipeline_parallel_tutorial.html)