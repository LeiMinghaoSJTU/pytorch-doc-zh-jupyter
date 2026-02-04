# 通用连接上下文管理器 [¶](#generic-join-context-manager "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/distributed.algorithms.join>
>
> 原始地址：<https://pytorch.org/docs/stable/distributed.algorithms.join.html>


 通用连接上下文管理器有助于对不均匀输入进行分布式训练。本页概述了相关类的 API：“Join”、“Joinable”和“JoinHook”。有关教程，请参阅[使用连接上下文管理器进行不均匀输入的分布式训练](https://pytorch.org/tutorials/advanced/generic_join.html)。


*班级*


 火炬分布式算法。



 Join
 


 ( *可连接
* , *启用



 =
 


 True
* , *在提前终止时抛出_



 =
 


 错误的
* ， ***


 kwargs
* ) [[source]](_modules/torch/distributed/algorithms/join.html#Join)[¶](#torch.distributed.algorithms.Join "此定义的永久链接")


 此类定义了通用连接上下文管理器，它允许在进程连接后调用自定义挂钩。这些钩子应该遮蔽非连接进程的集体通信，以防止挂起和错误并确保算法的正确性。有关钩子定义的详细信息，请参阅 [`JoinHook`](#torch.distributed.algorithms.JoinHook "torch.distributed.algorithms.JoinHook")。


!!! warning "警告"

     上下文管理器要求每个参与的 [`Joinable`](#torch.distributed.algorithms.Joinable "torch.distributed.algorithms.Joinable") 调用方法 [`notify_join_context()`](#torch.distributed.算法.Join.notify_join_context“torch.distributed.algorithms.Join.notify_join_context”)在其自己的每迭代集体通信之前确保正确性。


!!! warning "警告"

     上下文管理器要求 [`JoinHook`](#torch.distributed.algorithms.JoinHook "torch.distributed.algorithms.JoinHook") 对象中的所有 `process_group` 属性都相同。如果有多个 [`JoinHook`](#torch.distributed.algorithms.JoinHook "torch.distributed.algorithms.JoinHook") 对象，则使用第一个的`device`。进程组和设备信息用于检查未加入的进程，并在启用了“ throw_on_early_termination”时通知进程抛出异常，这两者都使用 all-reduce。


 参数 
* **joinables** ( *List
* *[
* [*Joinable*](#torch.distributed.algorithms.Joinable "torch.distributed.algorithms.Joinable")*]
* ) – 参与的列表 [` Joinable`](#torch.distributed.algorithms.Joinable "torch.distributed.algorithms.Joinable") s;它们的钩子按照给定的顺序迭代。
* **enable** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)" ) ) – 启用不均匀输入检测的标志；设置为“False”会禁用上下文管理器的功能，并且仅当用户知道输入不会不均匀时才应设置(默认值：“True”)。
* ** throw_on_early_termination** ( [*bool*] (https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 控制是否在检测到不均匀输入时抛出异常的标志(默认值： `False` ) 。


 例子：


```
>>> import os
>>> import torch
>>> import torch.distributed as dist
>>> import torch.multiprocessing as mp
>>> import torch.nn.parallel.DistributedDataParallel as DDP
>>> import torch.distributed.optim.ZeroRedundancyOptimizer as ZeRO
>>> from torch.distributed.algorithms.join import Join
>>>
>>> # On each spawned worker
>>> def worker(rank):
>>>     dist.init_process_group("nccl", rank=rank, world_size=2)
>>>     model = DDP(torch.nn.Linear(1, 1).to(rank), device_ids=[rank])
>>>     optim = ZeRO(model.parameters(), torch.optim.Adam, lr=0.01)
>>>     # Rank 1 gets one more input than rank 0
>>>     inputs = [torch.tensor([1.]).to(rank) for _ in range(10 + rank)]
>>>     with Join([model, optim]):
>>>         for input in inputs:
>>>             loss = model(input).sum()
>>>             loss.backward()
>>>             optim.step()
>>>     # All ranks reach here without hanging/erroring

```


*静止的*


 通知_join_context


 ( *joinable
* ) [[source]](_modules/torch/distributed/algorithms/join.html#Join.notify_join_context)[¶](#torch.distributed.algorithms.Join.notify_join_context "此定义的永久链接")


 通知加入上下文管理器调用进程尚未加入；然后，如果 `throw_on_early_termination=True` ，检查是否检测到不均匀的输入(即一个进程是否已经加入)，如果是，则抛出异常。


 在每次迭代集体通信之前，应该从 Joinable 对象调用此方法。例如，应该在“DistributedDataParallel”中的前向传递开始时调用它。


 只有传递到 contextmanager 的第一个 [`Joinable`](#torch.distributed.algorithms.Joinable "torch.distributed.algorithms.Joinable") 对象在此方法中执行集体通信，对于其他对象，此方法是空的。


 Parameters


**joinable** ( [*Joinable*](#torch.distributed.algorithms.Joinable "torch.distributed.algorithms.Joinable") ) – [`Joinable`](#torch.distributed.algorithms.Joinable "torch. Distributed.algorithms.Joinable") 对象调用此方法。


 退货


 all-reduce 的异步工作句柄意味着如果“joinable”是第一个传递到上下文管理器的进程尚未加入，则通知上下文管理器；否则“无”。


*班级*


 火炬分布式算法。


 Joinable [[source]](_modules/torch/distributed/algorithms/join.html#Joinable)[¶](#torch.distributed.algorithms.Joinable"此定义的永久链接")


 这定义了可连接类的抽象基类。可连接类(继承自 [`Joinable`](#torch.distributed.algorithms.Joinable "torch.distributed.algorithms.Joinable") )应该实现 [`join_hook()`](#torch.distributed.algorithms. Joinable.join_hook "torch.distributed.algorithms.Joinable.join_hook") ，除了 [`JoinHook`](#torch.distributed.algorithms.JoinHook "torch.distributed.algorithms.JoinHook") 实例之外，还返回 [` join_device()`](#torch.distributed.algorithms.Joinable.join_device "torch.distributed.algorithms.Joinable.join_device") 和 [`join_process_group()`](#torch.distributed.algorithms. Joinable.join_process_group "torch.distributed.algorithms.Joinable.join_process_group") 分别返回设备和进程组信息。


*抽象的


 财产*


 加入_设备 *:


[device](tensor_attributes.html#torch.device "torch.device")*[¶](#torch.distributed.algorithms.Joinable.join_device "此定义的永久链接")


 返回执行加入上下文管理器实现本身所需的集体通信的设备。


*抽象的*


 加入_hook


 (***


 kwargs
* ) [[source]](_modules/torch/distributed/algorithms/join.html#Joinable.join_hook)[¶](#torch.distributed.algorithms.Joinable.join_hook "此定义的永久链接")


 返回给定的 [`Joinable`](#torch.distributed.algorithms.Joinable "torch.distributed.algorithms") 的 [`JoinHook`](#torch.distributed.algorithms.JoinHook "torch.distributed.algorithms.JoinHook") 实例.可加入”)。


 Parameters


**kwargs** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 一个 [`dict`]( https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") 包含任何关键字参数以在运行时修改连接钩子的行为；所有共享相同 join contextmanager 的 [`Joinable`](#torch.distributed.algorithms.Joinable "torch.distributed.algorithms.Joinable") 实例都会转发相同的 `kwargs` 值。


 Return type


[*JoinHook*](#torch.distributed.algorithms.JoinHook“torch.distributed.algorithms.join.JoinHook”)


*抽象的


 财产*


 加入_process_group *:


[任何](https://docs.python.org/3/library/typing.html#typing.Any"(Python v3.12)")*[¶](#torch.distributed.algorithms.Joinable.join_process_group "此定义的永久链接")


 返回连接上下文管理器本身所需的集体通信的进程组。


*班级*


 火炬分布式算法。


 JoinHook [[source]](_modules/torch/distributed/algorithms/join.html#JoinHook)[¶](#torch.distributed.algorithms.JoinHook"此定义的永久链接")


 这定义了一个连接钩子，它在连接上下文管理器中提供了两个入口点：一个主钩子，当存在未连接的进程时重复调用它；以及一个后钩子，当所有进程加入时调用它。


 要为通用连接上下文管理器实现连接钩子，请定义一个继承自 [`JoinHook`](#torch.distributed.algorithms.JoinHook "torch.distributed.algorithms.JoinHook") 的类并覆盖 `main_hook()`和适当的`post_hook()`。


 主_钩子


 ( ) [[source]](_modules/torch/distributed/algorithms/join.html#JoinHook.main_hook)[¶](#torch.distributed.algorithms.JoinHook.main_hook "此定义的永久链接")


 当在一次训练迭代中(即在一个前向传递、后向传递和优化器步骤中)存在一个非连接过程来影子集体通信时，会重复调用此钩子。


 后_钩子


 ( *is_last_joiner
* ) [[source]](_modules/torch/distributed/algorithms/join.html#JoinHook.post_hook)[¶](#torch.distributed.algorithms.JoinHook.post_hook "此定义的永久链接")


 该钩子在所有进程加入后被调用。它传递了一个额外的“bool”参数“is_last_joiner”，它指示该排名是否是最后加入的排名之一。


 Parameters


**is_last_joiner** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – `True`如果该级别是最后加入的之一；否则为“假”。