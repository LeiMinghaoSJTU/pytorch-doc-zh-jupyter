# 分布式优化器 [¶](#distributed-optimizers "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/distributed.optim>
>
> 原始地址：<https://pytorch.org/docs/stable/distributed.optim.html>


!!! warning "警告"

     使用 CUDA tensor时，当前不支持分布式优化器


[`torch.distributed.optim`](#module-torch.distributed.optim "torch.distributed.optim") 公开 DistributedOptimizer，它采用远程参数列表 (`RRef`) 并在参数所在的工作器上本地运行优化器居住。分布式优化器可以使用任何本地优化器[基类](optim.html#optimizer-algorithms) 在每个工作线程上应用梯度。


*班级*


 火炬.分布式.优化。


 分布式优化器


 ( *optimizer_class
* , *params_rref
* , **


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/distributed/optim/optimizer.html#DistributedOptimizer)[¶](#torch.distributed.optim.DistributedOptimizer "此定义的永久链接")


 DistributedOptimizer 对分散在工作人员中的参数进行远程引用，并在本地为每个参数应用给定的优化器。


 此类使用 [`get_gradients()`](rpc.html#torch.distributed.autograd.get_gradients "torch.distributed.autograd.get_gradients") 来检索特定参数的梯度。


 对 [`step()`](#torch.distributed.optim.DistributedOptimizer.step "torch.distributed.optim.DistributedOptimizer.step") 的并发调用，无论是来自相同还是不同的客户端，都将在每个工作线程上序列化 – 因为每个工人的优化器一次只能处理一组梯度。但是，不能保证一次会为一个客户端执行完整的前向-后向优化器序列。这意味着所应用的梯度可能与在给定工作线程上执行的最新前向传播不对应。此外，工人之间的订购也没有保证。


 DistributedOptimizer 默认情况下会创建启用 TorchScript 的本地优化器，以便在多线程训练(例如 DistributedModel Parallel)的情况下，优化器更新不会被 Python GlobalInterpreter Lock (GIL) 阻止。目前大多数优化器都启用了此功能。您还可以按照 PyTorch 教程中的[秘诀](https://github.com/pytorch/tutorials/pull/1465) 为您自己的自定义优化器启用 TorchScript 支持。


 参数 
* **optimizer_class** ( [*optim.Optimizer*](optim.html#torch.optim.Optimizer "torch.optim.Optimizer") ) – 在每个工作线程上实例化的优化器类。
* **params _rref** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(在 Python v3.12 中)")*[
* *RRef
* *]
* ) – 要优化的本地或远程参数的 RRef 列表。
* **args** – 要传递给每个工作线程上的优化器构造函数的参数。
* **kwargs** – 要传递给每个工作线程上的优化器构造函数的参数。


 例子：：




```
>>> import torch.distributed.autograd as dist_autograd
>>> import torch.distributed.rpc as rpc
>>> from torch import optim
>>> from torch.distributed.optim import DistributedOptimizer
>>>
>>> with dist_autograd.context() as context_id:
>>>   # Forward pass.
>>>   rref1 = rpc.remote("worker1", torch.add, args=(torch.ones(2), 3))
>>>   rref2 = rpc.remote("worker1", torch.add, args=(torch.ones(2), 1))
>>>   loss = rref1.to_here() + rref2.to_here()
>>>
>>>   # Backward pass.
>>>   dist_autograd.backward(context_id, [loss.sum()])
>>>
>>>   # Optimizer.
>>>   dist_optim = DistributedOptimizer(
>>>      optim.SGD,
>>>      [rref1, rref2],
>>>      lr=0.05,
>>>   )
>>>   dist_optim.step(context_id)

```



 step
 


 ( *context_id
* ) [[source]](_modules/torch/distributed/optim/optimizer.html#DistributedOptimizer.step)[¶](#torch.distributed.optim.DistributedOptimizer.step "此定义的永久链接")


 执行单个优化步骤。


 这将在每个包含参数的工作线程上调用 [`torch.optim.Optimizer.step()`](generated/torch.optim.Optimizer.step.html#torch.optim.Optimizer.step "torch.optim.Optimizer.step")进行优化，并将阻塞直到所有工人返回。提供的 `context_id` 将用于检索相应的 [`context`](rpc.html#torch.distributed.autograd.context "torch.distributed.autograd.context")，其中包含应应用于参数的渐变。


 Parameters


**context_id** – 我们应该运行优化器步骤的 autograd 上下文 id。


*班级*


 火炬.分布式.优化。


 PostLocalSGD优化器


 ( *optim
* , *averager
* ) [[source]](_modules/torch/distributed/optim/post_localSGD_optimizer.html#PostLocalSGDOptimizer)[¶](#torch.distributed.optim.PostLocalSGDOptimizer "此定义的永久链接")


 包装任意 [`torch.optim.Optimizer`](optim.html#torch.optim.Optimizer "torch.optim.Optimizer") 并运行 [post-local SGD](https://arxiv.org/abs/1808.07217 )，该优化器在每一步都运行本地优化器。在预热阶段之后，它会在应用本地优化器后定期对参数进行平均。


 参数 
* **optim** ( [*Optimizer*](optim.html#torch.optim.Optimizer "torch.optim.optimizer.Optimizer") ) – 本地优化器。
* **averager** ( *ModelAverager
* ) – 用于运行 post-localSGD 算法的模型平均器实例。


 例子：


```
>>> import torch
>>> import torch.distributed as dist
>>> import torch.distributed.algorithms.model_averaging.averagers as averagers
>>> import torch.nn as nn
>>> from torch.distributed.optim import PostLocalSGDOptimizer
>>> from torch.distributed.algorithms.ddp_comm_hooks.post_localSGD_hook import (
>>>   PostLocalSGDState,
>>>   post_localSGD_hook,
>>> )
>>>
>>> model = nn.parallel.DistributedDataParallel(
>>>    module, device_ids=[rank], output_device=rank
>>> )
>>>
>>> # Register a post-localSGD communication hook.
>>> state = PostLocalSGDState(process_group=None, subgroup=None, start_localSGD_iter=100)
>>> model.register_comm_hook(state, post_localSGD_hook)
>>>
>>> # Create a post-localSGD optimizer that wraps a local optimizer.
>>> # Note that ``warmup_steps`` used in ``PostLocalSGDOptimizer`` must be the same as
>>> # ``start_localSGD_iter`` used in ``PostLocalSGDState``.
>>> local_optim = torch.optim.SGD(params=model.parameters(), lr=0.01)
>>> opt = PostLocalSGDOptimizer(
>>>     optim=local_optim,
>>>     averager=averagers.PeriodicModelAverager(period=4, warmup_steps=100)
>>> )
>>>
>>> # In the first 100 steps, DDP runs global gradient averaging at every step.
>>> # After 100 steps, DDP runs gradient averaging within each subgroup (intra-node by default),
>>> # and post-localSGD optimizer runs global model averaging every 4 steps after applying the local optimizer.
>>> for step in range(0, 200):
>>>    opt.zero_grad()
>>>    loss = loss_fn(output, labels)
>>>    loss.backward()
>>>    opt.step()

```


 加载_state_dict


 ( *state_dict
* ) [[source]](_modules/torch/distributed/optim/post_localSGD_optimizer.html#PostLocalSGDOptimizer.load_state_dict)[¶](#torch.distributed.optim.PostLocalSGDOptimizer.load_state_dict "此定义的永久链接")


 这与 [`torch.optim.Optimizer`](optim.html#torch.optim.Optimizer "torch.optim.Optimizer")[`load_state_dict()`](#torch.distributed.optim.PostLocalSGDOptimizer.load_state_dict "torch.distributed.optim.PostLocalSGDOptimizer.load_state_dict") ，而且还将模型平均器的步长值恢复到提供的 `state_dict` 中保存的值。


 如果 `state_dict` 中没有 `"step"` 条目，它将发出警告并将模型平均器的步长初始化为 0。


 状态_dict


 ( ) [[source]](_modules/torch/distributed/optim/post_localSGD_optimizer.html#PostLocalSGDOptimizer.state_dict)[¶](#torch.distributed.optim.PostLocalSGDOptimizer.state_dict "此定义的永久链接")


 这与 [`torch.optim.Optimizer`](optim.html#torch.optim.Optimizer "torch.optim.Optimizer")[`state_dict()`](#torch.distributed.optim.PostLocalSGDOptimizer.state_dict "torch.distributed.optim.PostLocalSGDOptimizer.state_dict") ，但添加了一个额外的条目来记录模型平均器到检查点的步骤，以确保重新加载不会再次导致不必要的预热。


 step
 


 ( ) [[source]](_modules/torch/distributed/optim/post_localSGD_optimizer.html#PostLocalSGDOptimizer.step)[¶](#torch.distributed.optim.PostLocalSGDOptimizer.step "此定义的永久链接")


 执行单个优化步骤(参数更新)。


*班级*


 火炬.分布式.优化。


 零冗余优化器


 ( *params
* , *optimizer_class
* , *process_group



 =
 


 无
* , *parameters_as_bucket_view



 =
 


 假
* , *与_ddp 重叠_



 =
 


 错误的
* ， ***


 defaults
* ) [[source]](_modules/torch/distributed/optim/zero_redundancy_optimizer.html#ZeroRedundancyOptimizer)[¶](#torch.distributed.optim.ZeroRedundancyOptimizer "此定义的永久链接")


 此类包装任意 [`optim.Optimizer`](optim.html#torch.optim.Optimizer "torch.optim.Optimizer") 并将其状态按照 [ZeRO](https://arxiv) 描述的组中的等级进行分片.org/abs/1910.02054)。每个等级中的本地优化器实例仅负责更新大约“1/world_size”参数，因此只需要保留“1/world_size”优化器状态。参数在本地更新后，每个Rank将其参数广播给所有其他对等点，以保持所有模型副本处于相同状态。 `ZeroRedundancyOptimizer` 可以与 [`torch.nn.parallel.DistributedDataParallel`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 结合使用以减少每列峰值内存消耗。


“ZeroRedundancyOptimizer”使用排序贪心算法在每个等级打包多个参数。每个参数都属于一个单独的等级，并且不会在等级之间划分。分区是任意的，可能与参数注册或使用顺序不匹配。


 Parameters


**params** ( `Iterable` ) – [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") sor [`dict`](https://docs. python.org/3/library/stdtypes.html#dict "(in Python v3.12)") s 给出所有参数，这些参数将跨等级分片。


 关键字参数 
* **optimizer_class** ( `torch.nn.Optimizer` ) – 本地优化器的类。
* **process_group** ( `ProcessGroup` ，可选) – `torch.distributed``ProcessGroup` (默认值：由 [`torch.distributed.init_process_group()`](distributed.html#torch.distributed.init_process_group "torch.distributed.init_process_group") 初始化的 `dist.group.WORLD` )。
* **参数_as_bucket_view** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `True` ，参数被打包到桶中以加速通信，并且 `param.data` 字段指向不同偏移量的桶视图；如果“False”，则每个单独的参数单独通信，并且每个“params.data”保持完整(默认值：“False”)。
* **overlap_with_ddp** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(Python v3.12)")*,
* *可选
* ) – 如果 `True` , [`step()`](#torch.distributed.optim.ZeroRedundancyOptimizer.step "torch.distributed.optim.ZeroRedundancyOptimizer.step") 与 `DistributedDataParallel` 的梯度同步重叠；这需要(1)“optimizer_class”参数的功能优化器或具有等效功能的功能优化器，以及(2)注册由“ddp_zero_hook.py”中的函数之一构造的 DDP 通信挂钩；参数被打包到与 `DistributedDataParallel` 中的存储桶匹配，这意味着 `parameters_as_bucket_view` 参数被忽略。如果 `False` ， [`step()`](#torch.distributed.optim.ZeroRedundancyOptimizer.step "torch. Distribution.optim.ZeroRedundancyOptimizer.step") 在向后传递之后不相交地运行(按照正常)。(默认值： `False` )
* ****defaults** – 任何尾随参数，这些参数将转发到本地优化器。


 例子：


```
>>> import torch.nn as nn
>>> from torch.distributed.optim import ZeroRedundancyOptimizer
>>> from torch.nn.parallel import DistributedDataParallel as DDP
>>> model = nn.Sequential(*[nn.Linear(2000, 2000).to(rank) for _ in range(20)])
>>> ddp = DDP(model, device_ids=[rank])
>>> opt = ZeroRedundancyOptimizer(
>>>     ddp.parameters(),
>>>     optimizer_class=torch.optim.Adam,
>>>     lr=0.01
>>> )
>>> ddp(inputs).sum().backward()
>>> opt.step()

```


!!! warning "警告"

     目前，“ZeroRedundancyOptimizer”要求所有传入的参数都是相同的密集类型。


!!! warning "警告"

     如果您传递 `overlap_with_ddp=True` ，请注意以下事项： 鉴于将 `DistributedDataParallel` 与 [`ZeroRedundancyOptimizer`](#torch.distributed.optim.ZeroRedundancyOptimizer "torch.distributed.optim.ZeroRedundancyOptimizer" 重叠的方式) 目前已实现，前两次或三次训练迭代不会在优化器步骤中执行参数更新，具体取决于是否 `static_graph=False` 或 `static_graph=True` 。这是因为它需要有关“DistributedDataParallel”使用的梯度分桶策略的信息，如果“static_graph=False”，则直到第二次前向传递才最终确定，或者如果“static_graph=True”，直到第三次前向传递才最终确定。要对此进行调整，一种选择是预先添加虚拟输入。


!!! warning "警告"

     ZeroRedundancyOptimizer 是实验性的，可能会发生变化。


 添加_param_group


 ( *param_group
* ) [[source]](_modules/torch/distributed/optim/zero_redundancy_optimizer.html#ZeroRedundancyOptimizer.add_param_group)[¶](#torch.distributed.optim.ZeroRedundancyOptimizer.add_param_group "此定义的永久链接")


 将参数组添加到“Optimizer”的“param_groups”中。


 这在微调预训练网络时非常有用，因为 freezelayers 可以进行训练并在训练过程中添加到“优化器”中。


 Parameters


**param_group** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 指定要使用的参数优化和特定于组的优化选项。


!!! warning "警告"

     此方法处理更新所有分区上的分片，但需要在所有等级上调用。在排名的子集上调用此方法将导致训练挂起，因为通信原语是根据托管参数调用的，并且期望所有排名都参与同一组参数。


 巩固_state_dict


 (
 
*to
 



 =
 


 0
* ) [[source]](_modules/torch/distributed/optim/zero_redundancy_optimizer.html#ZeroRedundancyOptimizer.consolidate_state_dict)[¶](#torch.distributed.optim.ZeroRedundancyOptimizer.consolidate_state_dict "此定义的永久链接")


 在目标排名上合并一个“state_dict”列表(每个排名一个)。


 Parameters


**to** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 接收优化器状态的排名(默认值：0)。


 提高


[**RuntimeError**](https://docs.python.org/3/library/exceptions.html#RuntimeError "(in Python v3.12)") – 如果 `overlap_with_ddp=True` 并且这个方法在此 [`ZeroRedundancyOptimizer`](#torch.distributed.optim.ZeroRedundancyOptimizer "torch.distributed.optim.ZeroRedundancyOptimizer") 实例完全初始化之前调用，一旦重建 `DistributedDataParallel` 梯度桶就会发生这种情况。


!!! warning "警告"

     这需要向所有队伍呼吁。


 加入_hook


 (***


 kwargs
* ) [[source]](_modules/torch/distributed/optim/zero_redundancy_optimizer.html#ZeroRedundancyOptimizer.join_hook)[¶](#torch.distributed.optim.ZeroRedundancyOptimizer.join_hook "此定义的永久链接")


 返回 ZeRO 连接钩子，它可以通过在优化器步骤中隐藏集体通信来进行不均匀输入的训练。


 在调用此钩子之前必须正确设置渐变。


 Parameters


**kwargs** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 一个 [`dict`]( https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") 包含任何关键字参数以在运行时修改连接钩子的行为；共享相同连接上下文管理器的所有“Joinable”实例都会转发相同的“kwargs”值。


 该钩子不支持任何关键字参数；即“kwargs”未使用。


 加载_state_dict


 ( *state_dict
* ) [[source]](_modules/torch/distributed/optim/zero_redundancy_optimizer.html#ZeroRedundancyOptimizer.load_state_dict)[¶](#torch.distributed.optim.ZeroRedundancyOptimizer.load_state_dict "此定义的永久链接")


 从输入 `state_dict` 加载与给定排名相关的状态，根据需要更新本地优化器。


 Parameters


**state_dict** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 优化器状态；应该是从调用 [`state_dict()`](#torch.distributed.optim.ZeroRedundancyOptimizer.state_dict "torch.distributed.optim.ZeroRedundancyOptimizer.state_dict") 返回的对象。


 提高


[**RuntimeError**](https://docs.python.org/3/library/exceptions.html#RuntimeError "(in Python v3.12)") – 如果 `overlap_with_ddp=True` 并且这个方法在此 [`ZeroRedundancyOptimizer`](#torch.distributed.optim.ZeroRedundancyOptimizer "torch.distributed.optim.ZeroRedundancyOptimizer") 实例完全初始化之前调用，一旦重建 `DistributedDataParallel` 梯度桶就会发生这种情况。


 状态_dict


 ( ) [[source]](_modules/torch/distributed/optim/zero_redundancy_optimizer.html#ZeroRedundancyOptimizer.state_dict)[¶](#torch.distributed.optim.ZeroRedundancyOptimizer.state_dict "此定义的永久链接")


 返回此等级已知的最后一个全局优化器状态。


 提高


[**RuntimeError**](https://docs.python.org/3/library/exceptions.html#RuntimeError "(in Python v3.12)") – 如果 `overlap_with_ddp=True` 并且这个在此 [`ZeroRedundancyOptimizer`](#torch.distributed.optim.ZeroRedundancyOptimizer "torch.distributed.optim.ZeroRedundancyOptimizer") 实例完全初始化之前调用方法，一旦重建 `DistributedDataParallel` 梯度桶就会发生这种情况；或者如果调用此方法时没有事先调用 [`consolidate_state_dict()`](#torch.distributed.optim.ZeroRedundancyOptimizer.consolidate_state_dict "torch.distributed.optim.ZeroRedundancyOptimizer.consolidate_state_dict") 。


 Return type


[*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(Python v3.12)") , [*Any*](https://docs.python.org/3/library/typing.html#typing.Any " (在 Python v3.12 中)") ]




 step
 


 (*关闭



 =
 


 没有任何
* ， ***


 kwargs
* ) [[source]](_modules/torch/distributed/optim/zero_redundancy_optimizer.html#ZeroRedundancyOptimizer.step)[¶](#torch.distributed.optim.ZeroRedundancyOptimizer.step "此定义的永久链接")


 执行单个优化器步骤并同步所有等级的参数。


 Parameters


**closure** ( *Callable
* ) – 重新评估模型并返回损失的闭包；对于大多数优化器来说是可选的。


 退货


 可选损失取决于底层本地优化器。


 Return type


[*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中)") [ [float](https://docs.python. org/3/library/functions.html#float "(在 Python v3.12 中)") ]