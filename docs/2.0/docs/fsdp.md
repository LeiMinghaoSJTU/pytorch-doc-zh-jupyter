# FullShardedDataParallel [¶](#module-torch.distributed.fsdp "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/fsdp>
>
> 原始地址：<https://pytorch.org/docs/stable/fsdp.html>


*班级*


 torch.distributed.fsdp。


 完全分片数据并行


 ( *模块
* , *进程_group



 =
 


 无
* , *分片_strategy



 =
 


 无
* , *cpu_offload



 =
 


 无
* , *auto_wrap_policy



 =
 


 无
* , *向后_prefetch



 =
 


 BackwardPrefetch.BACKWARD_PRE
* , *混合_精度



 =
 


 无
* , *忽略_modules



 =
 


 无
* , *param_init_fn



 =
 


 无
* , *设备_id



 =
 


 无
* , *sync_module_states



 =
 


 False
* , *forward_prefetch



 =
 


 False
* , *限制_all_gathers



 =
 


 True
* , *使用_orig_params



 =
 


 False
* , *忽略_states



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel)[¶](#torch.distributed.fsdp.FullyShardedDataParallel "此定义的永久链接")


 用于跨数据并行工作器分片模块参数的包装器。这是受到 [Xu 等人](https://arxiv.org/abs/2004.13336) 以及来自 [DeepSpeed](https://www.deepspeed.ai/) 的 ZeRO Stage 3 的启发。FullyShardedDataParallel 通常会缩短到 FSDP。


 例子：


```
>>> import torch
>>> from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
>>> torch.cuda.set_device(device_id)
>>> sharded_module = FSDP(my_module)
>>> optim = torch.optim.Adam(sharded_module.parameters(), lr=0.0001)
>>> x = sharded_module(x, y=3, z=torch.Tensor([1]))
>>> loss = x.sum()
>>> loss.backward()
>>> optim.step()

```


!!! warning "警告"

     优化器必须在使用 FSDP 包装模块*之后进行初始化，因为 FSDP 将以可能不保留原始参数变量的方式对模块的参数进行分片和转换。因此，先前初始化的优化器可能具有对参数的过时引用。


!!! warning "警告"

     如果目标 CUDA 设备具有 ID `dev_id` ，则 (1) `module` 应已放置在该设备上，(2) 应使用 `torch.cuda.set_device(dev_id)` 设置设备，或 (3) `dev_id` 应传递到 `device_id` 构造函数参数中。此 FSDP 实例的计算设备将是该目标设备。对于(1)和(3)，FSDP初始化总是发生在GPU上。对于(2)，FSDP初始化发生在`module`的当前设备上，可能是CPU。


!!! warning "警告"

     当使用 CPU 卸载时，FSDP 目前不支持 `no_sync()` 之外的梯度累积。尝试这样做会产生不正确的结果，因为 FSDP 将使用新减少的梯度而不是累积任何现有梯度。


!!! warning "警告"

     构造后更改原始参数变量名称将导致未定义的行为。


!!! warning "警告"

     传入 `sync_module_states=True` 标志要求 `module` 位于 GPU 上，或者使用 `device_id` 参数来指定 FSDP 将在 FSDP 构造函数中将 `module` 移动到的 CUDA 设备。这是因为 `sync_module_states=True` 需要 GPU 通信。


!!! warning "警告"

     从 PyTorch 1.12 开始，FSDP 仅提供对共享参数的有限支持(例如，将一个“Linear”层的权重设置为另一层的权重)。特别是，共享参数的模块必须包装为同一 FSDP 单元的一部分。如果您的用例需要增强的共享参数支持，请 ping <https://github.com/pytorch/pytorch/issues/77724>


!!! warning "警告"

     FSDP 对冻结参数有一些限制(即设置 `param.requires_grad=False` )。对于 use_orig_params=False ，每个 FSDP 实例必须管理全部冻结或全部非冻结的参数。对于 use_orig_params=True ，FSDP 支持混合冻结和非冻结，但我们建议不要这样做，因为这样梯度内存使用量将高于预期(即相当于不冻结这些参数)。这意味着理想情况下，冻结参数应隔离到它们自己的“nn.Module”中，并用 FSDP 单独包装。


!!! note "笔记"

    不支持尝试运行 FSDP 实例中包含的子模块的前向传递，这将导致错误。这是因为子模块的参数将被分片，但它本身不是 FSDP 实例，因此它的前向传递不会适当地全部收集完整参数。当尝试仅运行编码器-解码器模型的编码器时，可能会发生这种情况，并且编码器未包装在其自己的 FSDP 实例中。要解决此问题，请将子模块包装在其自己的 FSDP 单元中。


!!! note "笔记"

    FSDP 将输入tensor通过“forward”方法移至 GPU 计算设备，因此用户无需手动将它们从 CPU 移出。


!!! warning "警告"

     自修改以来，用户不应在不使用 [`summon_full_params()`](#torch.distributed.fsdp.FullyShardedDataParallel.summon_full_params "torch.distributed.fsdp.FullyShardedDataParallel.summon_full_params") 上下文的情况下修改向前和向后之间的参数可能不会坚持下去。此外，对于“use_orig_params=False”，在向前和向后之间访问原始参数可能会引发非法内存访问。


!!! warning "警告"

     对于 `use_orig_params=True` ， `ShardingStrategy.SHARD_GRAD_OP` 会暴露未分片的参数，而不是分片的参数，因为它不会释放未分片的参数，这与 `ShardingStrategy.FULL_SHARD` 不同。需要注意的是，由于梯度总是分片或“None”，因此“ShardingStrategy.SHARD_GRAD_OP”不会在转发后使用未分片的参数公开分片梯度。如果你想检查梯度，请尝试 [`summon_full_params()`](#torch.distributed.fsdp.FullyShardedDataParallel.summon_full_params "torch.distributed.fsdp.FullyShardedDataParallel.summon_full_params") 和 `with_grads=True `。


!!! warning "警告"

     出于与 autograd 相关的原因，FSDP 在前向和后向计算期间将托管模块的参数替换为“torch.Tensor”视图。如果模块的前向依赖于保存的参数引用，而不是每次迭代重新获取引用，那么它将看不到 FSDP 新创建的视图，并且 autograd 将无法正常工作。


!!! note "笔记"

    使用“limit_all_gathers=True”，您可能会在 FSDPpre-forward 中看到一个间隙，其中 CPU 线程没有发出任何内核。这是有意为之的，并且显示了速率限制器的效果。以这种方式同步 CPU 线程可以防止为后续的所有收集过度分配内存，并且实际上不应延迟 GPU 内核的执行。


!!! note "笔记"

    当使用 sharding_strategy=ShardingStrategy.HYBRID_SHARD 且分片进程组为节点内且复制进程组为节点间时，设置 NCCL_CROSS_NIC=1 有助于提高复制的 all-reduce 时间某些集群设置的进程组。


 参数 
* **module** ( [*nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 这是要使用 FSDP 包装的模块.
* **process_group** ( *可选
* *[
* *联合
* *[
* *ProcessGroup
* *,
* *元组
* *[
* *ProcessGroup
* *,
* *ProcessGroup
* *]
* *]
* 
* ]
* ) – 这是对模型进行分片的进程组，因此用于 FSDP 的全收集和减少分散集体通信。如果为 None ，则 FSDP 使用默认进程组。对于诸如“ShardingStrategy.HYBRID_SHARD”之类的混合分片策略，用户可以传入进程组的元组，分别表示要分片和复制的组。如果为“None”，则 FSDP 为用户构建进程组以在节点内进行分片并在节点间进行复制。(默认：“None”)
* **sharding_strategy** ( *可选
* *[
* [*ShardingStrategy*] (#torch.distributed.fsdp.ShardingStrategy "torch.distributed.fsdp.ShardingStrategy")*]
* ) – 配置分片策略，可能会权衡内存节省和通信开销。有关详细信息，请参阅[`ShardingStrategy`](#torch.distributed.fsdp.ShardingStrategy "torch.distributed.fsdp.ShardingStrategy")。 (默认：`FULL_SHARD`)
* **cpu_offload** (*可选
* *[
* [*CPUOffload*](#torch.distributed.fsdp.CPUOffload "torch.distributed.fsdp.CPUOffload")*] 
* ) – 这配置 CPU 卸载。如果将其设置为“None”，则不会发生 CPU 卸载。有关详细信息，请参阅 [`CPUOffload`](#torch.distributed.fsdp.CPUOffload "torch.distributed.fsdp.CPUOffload")。(默认：`None` )
* **auto_wrap_policy** ( *可选
* 
* [
* *联合
* *[
* *可调用
* [
* *[
* [*nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module")*, 
* [*bool*](https://docs.python.org/3/library/functions.html#bool "(Python v3.12)")*,
* [*int*](https://docs.python.org/3/library/functions.html#int "(Python v3.12)")*]
* *,
* [*bool*](https://docs.python.org/3/library/function.html#bool "(在 Python v3.12 中)")*]
* *,
* *ModuleWrapPolicy
* *]
* *]
* ) –


 这指定了将 FSDP 应用于“module”的子模块的策略，这是通信和计算重叠所需的，从而影响性能。如果 `None` ，则 FSDP 仅适用于 `module` ，用户应手动将 FSDP 应用到父模块本身(自下而上进行)。为了方便起见，它直接接受“ModuleWrapPolicy”，它允许用户指定要包装的模块类(例如变压器块)。否则，这应该是一个可调用的，它接受三个参数 `module: nn.Module` 、 `recurse: bool` 和 `nonwrapped_numel: int` 并且应该返回一个 `bool` 指定传入的 `module` 是否应该如果 `recurse=False` 则应用 FSDP，或者如果 `recurse=True` 则遍历应继续进入模块的子树。用户可以向可调用函数添加额外的参数。 `torch.distributed.fsdp.wrap.py` 中的 `size_based_auto_wrap_policy` 给出了一个可调用示例，如果子树中的参数超过 100M numel，则将 FSDP 应用于模块。我们建议在应用 FSDP 后打印模型并根据需要进行调整。


 例子：


```
>>> def custom_auto_wrap_policy(
>>>     module: nn.Module,
>>>     recurse: bool,
>>>     nonwrapped_numel: int,
>>>     # Additional custom arguments
>>>     min_num_params: int = int(1e8),
>>> ) -> bool:
>>>     return nonwrapped_numel >= min_num_params
>>> # Configure a custom `min_num_params`
>>> my_auto_wrap_policy = functools.partial(custom_auto_wrap_policy, min_num_params=int(1e5))

```
* **backward_prefetch** 
 (
 *Optional* 
** 
[*BackwardPrefetch*
*]* 
 ) – This configures explicit backward prefetching of all-gathers. If
 `None`
 , then FSDP does not backward prefetch, and there is no
communication and computation overlap in the backward pass. See
 `BackwardPrefetch`
 for details. (Default:
 `BACKWARD_PRE`
 )
* **mixed_precision** 
 (
 *Optional* 
** 
[*MixedPrecision*
*]* 
 ) – This configures native mixed precision for FSDP. If this is set to
 `None`
 , then no mixed precision is used. Otherwise, parameter,
buffer, and gradient reduction dtypes can be set. See
 `MixedPrecision`
 for details. (Default:
 `None`
 )
* **ignored_modules** 
 (
 *Optional* 
** 
*Iterable* 
*[* 
[*torch.nn.Module*
*]* 
*]* 
 ) – Modules whose
own parameters and child modules’ parameters and buffers are
ignored by this instance. None of the modules directly in
 `ignored_modules`
 should be
 `FullyShardedDataParallel`
 instances, and any child modules that are already-constructed
 `FullyShardedDataParallel`
 instances will not be ignored if
they are nested under this instance. This argument may be used to
avoid sharding specific parameters at module granularity when using an
 `auto_wrap_policy`
 or if parameters’ sharding is not managed by
FSDP. (Default:
 `None`
 )
* **param_init_fn** 
 (
 *Optional* 
** 
*Callable* 
*[* 
*[* 
[*nn.Module*
*]* 
*,* 
*None* 
*]* 
*]* 
 ) –
 


 “Callable[torch.nn.Module] -
> None”指定当前元设备上的模块应如何初始化到实际设备上。从 v1.12 开始，FSDP 通过“is_meta”检测元设备上具有参数或缓冲区的模块，并且如果指定则应用“param_init_fn”，否则调用“nn.Module.reset_parameters()”。对于这两种情况，实现应该“仅”初始化模块的参数/缓冲区，而不是其子模块的参数/缓冲区。这是为了避免重新初始化。此外，FSDP 还支持通过 torchdistX 的 ( <https://github.com/pytorch/torchdistX
> ) `deferred_init()` API 进行延迟初始化，其中延迟模块通过调用 `param_init_fn` (如果指定)或否则，torchdistX 的默认 `materialize_module()` 。如果指定了“param_init_fn”，则它将应用于所有元设备模块，这意味着它可能应该针对模块类型。 FSDP 在参数扁平化和分片之前调用初始化函数。


 例子：


```
>>> module = MyModule(device="meta")
>>> def my_init_fn(module: nn.Module):
>>>     # E.g. initialize depending on the module type
>>>     ...
>>> fsdp_model = FSDP(module, param_init_fn=my_init_fn, auto_wrap_policy=size_based_auto_wrap_policy)
>>> print(next(fsdp_model.parameters()).device) # current CUDA device
>>> # With torchdistX
>>> module = deferred_init.deferred_init(MyModule, device="cuda")
>>> # Will initialize via deferred_init.materialize_module().
>>> fsdp_model = FSDP(module, auto_wrap_policy=size_based_auto_wrap_policy)

```
* **device_id** 
 (
 *Optional* 
** 
*Union* 
*[* 
[*int*")
*,* 
*torch.device*
*]* 
*]* 
 ) – An
 `int`
 or
 `torch.device`
 giving the CUDA device on which FSDP
initialization takes place, including the module initialization
if needed and the parameter sharding. This should be specified to
improve initialization speed if
 `module`
 is on CPU. If the
default CUDA device was set (e.g. via
 `torch.cuda.set_device`
 ),
then the user may pass
 `torch.cuda.current_device`
 to this.
(Default:
 `None`
 )
* **sync_module_states** 
 (
 *bool*")
 ) – If
 `True`
 , then each FSDP module will
broadcast module parameters and buffers from rank 0 to ensure that
they are replicated across ranks (adding communication overhead to
this constructor). This can help load
 `state_dict`
 checkpoints
via
 `load_state_dict`
 in a memory efficient way. See
 `FullStateDictConfig`
 for an example of this. (Default:
 `False`
 )
* **forward_prefetch** 
 (
 *bool*")
 ) – If
 `True`
 , then FSDP
 *explicitly* 
 prefetches
the next forward-pass all-gather before the current forward
computation. This is only useful for CPU-bound workloads, in which
case issuing the next all-gather earlier may improve overlap. This
should only be used for static-graph models since the prefetching
follows the first iteration’s execution order. (Default:
 `False`
 )
* **limit_all_gathers** 
 (
 *bool*")
 ) – If
 `True`
 , then FSDP explicitly
synchronizes the CPU thread to ensure GPU memory usage from only
 *two* 
 consecutive FSDP instances (the current instance running
computation and the next instance whose all-gather is prefetched).
If
 `False`
 , then FSDP allows the CPU thread to issue all-gathers
without any extra synchronization. (Default:
 `True`
 ) We often
refer to this feature as the “rate limiter”. This flag should only
be set to
 `False`
 for specific CPU-bound workloads with low
memory pressure in which case the CPU thread can aggressively issue
all kernels without concern for the GPU memory usage.
* **use_orig_params** 
 (
 *bool*")
 ) – Setting this to
 `True`
 has FSDP use
 `module`
 ‘s original parameters. FSDP exposes those original
parameters to the user via
 `nn.Module.named_parameters()`
 instead of FSDP’s internal
 `FlatParameter`
 s. This means
that the optimizer step runs on the original parameters, enabling
per-original-parameter hyperparameters. FSDP preserves the original
parameter variables and manipulates their data between unsharded
and sharded forms, where they are always views into the underlying
unsharded or sharded
 `FlatParameter`
 , respectively. With the
current algorithm, the sharded form is always 1D, losing the
original tensor structure. An original parameter may have all,
some, or none of its data present for a given rank. In the none
case, its data will be like a size-0 empty tensor. Users should not
author programs relying on what data is present for a given
original parameter in its sharded form.
 `True`
 is required to
use
 `torch.compile()`
 . Setting this to
 `False`
 exposes FSDP’s
internal
 `FlatParameter`
 s to the user via
 `nn.Module.named_parameters()`
 . (Default:
 `False`
 )
* **ignored_states** 
 (
 *Optional* 
*[* 
*Iterable* 
*[* 
*torch.nn.Parameter* 
*]* 
*]* 
*,* 
*Optional* 
** 
*Iterable* 
*[* 
[*torch.nn.Module*
*]* 
*]* 
 ) – Ignored parameters or modules that will not be managed by this FSDP
instance, meaning that the parameters are not sharded and their
gradients are not reduced across ranks. This argument unifies with
the existing
 `ignored_modules`
 argument, and we may deprecate
 `ignored_modules`
 soon. For backward compatibility, we keep both
 `ignored_states`
 and
 
 ignored_modules`
 
 , but FSDP only allows one
of them to be specified as not
 `None`
 .


 apply
 


 ( *fn
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.apply)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.apply "此定义的永久链接")


 将 `fn` 递归地应用于每个子模块(由 `.children()` 返回)以及 self。典型用途包括初始化模型的参数(另请参阅 [torch.nn.init](nn.init.html#nn-init-doc) )。


 与 torch.nn.Module.apply 相比，此版本在应用 fn 之前还收集了完整的参数。不应在另一个“summon_full_params”上下文中调用它。


 Parameters


**fn** ( `Module` -
> None) – 应用于每个子模块的函数


 退货


 self
 


 Return type


[模块](generated/torch.nn.Module.html#torch.nn.Module“torch.nn.Module”)


 剪辑_grad_norm_


 ( *max_norm
* , *norm_type



 =
 


 2.0
* ) [[source]](_modules/torch/distributed/fsdp/completely_sharded_data_parallel.html#FullyShardedDataParallel.clip_grad_norm_)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.clip_grad_norm_"此定义的永久链接")


 剪切所有参数的梯度范数。范数是计算作为单个向量的整体参数的梯度，并且梯度被就地修改。


 参数 
* **max_norm** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [
* int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 梯度的最大范数
* **norm_type** ( [ *float*](https://docs.python.org/3/library/functions.html#float "(在 Python v3.12 中)")*或
* [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 所使用的 p-范数的类型。可以是无穷范数的“inf”。


 退货


 参数的总范数(视为单个向量)。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")




!!! note "笔记"

    如果每个 FSDP 实例都使用“NO_SHARD”，这意味着没有梯度跨等级分片，那么您可以直接使用 [`torch.nn.utils.clip_grad_norm_()`](generated/torch.nn. utils.clip_grad_norm_.html#torch.nn.utils.clip_grad_norm_“torch.nn.utils.clip_grad_norm_”)。


!!! note "笔记"

    如果至少某些 FSDP 实例使用分片策略(即除 `NO_SHARD` 之外的策略)，那么您应该使用此方法而不是 [`torch.nn.utils.clip_grad_norm_()`](generated /torch.nn.utils.clip_grad_norm_.html#torch.nn.utils.clip_grad_norm_ "torch.nn.utils.clip_grad_norm_") 因为此方法处理梯度跨等级分片的事实。


!!! note "笔记"

    返回的总范数将具有 PyTorch 类型提升语义定义的所有参数/梯度中的“最大”dtype。例如，如果*所有*参数/梯度使用低精度数据类型，则返回的范数的数据类型将是该低精度数据类型，但如果至少存在一个使用 FP32 的参数/梯度，则返回的范数的数据类型将为 FP32。


!!! warning "警告"

     由于它使用集体通信，因此需要在所有级别上进行调用。


*静止的*


 展平_sharded_optim_state_dict


 ( *sharded_optim_state_dict
* , *model
* , *optim
* ) [[source]](_modules/torch/distributed/fsdp/full_sharded_data_parallel.html#FullyShardedDataParallel.flatten_sharded_optim_state_dict)[¶](#torch.distributed. fsdp.FullyShardedDataParallel.flatten_sharded_optim_state_dict"此定义的永久链接")


 该 API 类似于 [`shard_full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict") 。唯一的区别是输入 `sharded_optim_state_dict` 应从 [`sharded_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.sharded_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.sharded_optim_state_dict") 。因此，每个等级都会有 all-gather 调用来收集“ShardedTensor”。


 参数 
* **sharded_optim_state_dict** ( *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3 中).12)")*,
* *Any
* *]
* ) – 对应于未展平参数并保存分片优化器状态的优化器状态字典。
* **model** ( [*torch.nn.Module*](generated/torch. nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 请参阅 [`shard_full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict " torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict").
* **optim** ( [*torch.optim.Optimizer*](optim.html#torch.optim.Optimizer "torch.optim.Optimizer") ) – 优化器`模型` '参数。


 退货


 请参阅 [`shard_full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict") 。


 Return type


[*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(Python v3.12)") , [*Any*](https://docs.python.org/3/library/typing.html#typing.Any " (在 Python v3.12 中)") ]


 向前


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.forward)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.forward "此定义的永久链接")


 运行包装模块的前向传递，插入 FSDP 特定的前向和后向分片逻辑。


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)


*静止的*


 fsdp_modules


 ( *模块
* , *root_only



 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/completely_sharded_data_parallel.html#FullyShardedDataParallel.fsdp_modules)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.fsdp_modules "此定义的永久链接")


 返回所有嵌套的 FSDP 实例，可能包括 `module` 本身，并且如果 `root_only=True` 则仅包括 FSDP 根模块。


 参数 
* **module** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 根模块，可能或可能不是 `FSDP` 模块。
* **root_only** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中) ") ) – 是否仅返回 FSDP 根模块。(默认值： `False` )


 退货


 嵌套在输入 `module` 中的 FSDP 模块。


 Return type


 列表[ [FullyShardedDataParallel](#torch.distributed.fsdp.FullyShardedDataParallel "torch.distributed.fsdp.FullyShardedDataParallel") ]


*静止的*


 完整_optim_state_dict


 (*模型*，*优化*，*优化_输入



 =
 


 无
* , *rank0_only



 =
 


 真*，*组



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.full_optim_state_dict)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict "此定义的永久链接")


 合并排名 0 上的完整优化器状态，​​并按照约定将其返回为 [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") [`torch.optim.Optimizer.state_dict()`](generated/torch.optim.Optimizer.state_dict.html#torch.optim.Optimizer.state_dict "torch.optim.Optimizer.state_dict") ，即带有键`"state"` 和 `"param_groups"` 。 “model”中包含的“FSDP”模块中的扁平化参数被映射回其未扁平化参数。


!!! warning "警告"

     由于它使用集体通信，因此需要在所有级别上进行调用。但是，如果 `rank0_only=True` ，则状态字典仅填充在排名 0 上，所有其他排名返回一个空的 [`dict`](https://docs.python.org/3/library/stdtypes.html #dict "(Python v3.12)") 。


!!! warning "警告"

     与 torch.optim.Optimizer.state_dict() 不同，此方法使用完整的参数名称作为键而不是参数 ID。


!!! note "笔记"

    就像在 [`torch.optim.Optimizer.state_dict()`](generated/torch.optim.Optimizer.state_dict.html#torch.optim.Optimizer.state_dict "torch.optim.Optimizer.state_dict") 中一样，tensor包含在优化器状态下字典不会被克隆，因此可能会出现别名意外。对于最佳实践，请考虑立即保存返回的优化器状态字典，例如使用“torch.save()”。


 参数 
* **model** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 根模块(可能或可能不是参数被传递到优化器“optim”的 [`FullyShardedDataParallel`](#torch.distributed.fsdp.FullyShardedDataParallel "torch.distributed.fsdp.FullyShardedDataParallel") 实例)。
* **optim** ( [*torch. optim.Optimizer*](optim.html#torch.optim.Optimizer "torch.optim.Optimizer") ) – `model` 'sparameters 的优化器。
* **optim_input** ( *可选
* *[
* *Union 
* *[
* *List
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)" )*,
* *Any
* *]
* *]
* *,
* *Iterable
* *[
* *torch.nn.Parameter
* *]
* *]
* *]
* ) – 传递到优化器 `optim` 的输入代表任一参数组的 [`list`](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)") 或参数的可迭代；如果 `None` ，那么这个方法假设输入是`model.parameters()`。该参数已被弃用，不再需要传递它。 (默认值：`None`)
* **rank0_only** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)" ) ) – 如果 `True` ，则仅在等级 0 上保存填充的 [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ;如果“False”，则将其保存在所有等级上。 (默认值：`True`)
* **group** (*dist.ProcessGroup*) – 模型的进程组，如果使用默认进程组，则为`None`。 (默认值：`无`)


 退货


 一个 [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") 包含 `model` 的原始未展平参数的优化器状态和包括遵循 [`torch.optim.Optimizer.state_dict()`](generate/torch.optim.Optimizer.state_dict.html#torch.optim.Optimizer.state_dict) 约定的键“state”和“param_groups” “torch.optim.Optimizer.state_dict”)。如果 `rank0_only=True` ，则非零排名返回一个空的 [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") 。


 Return type


 Dict[ [str](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") , Any]


*静止的*


 获取_state_dict_type


 ( *module
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.get_state_dict_type)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.get_state_dict_type "此定义的永久链接")


 获取以 `module` 为根的 FSDP 模块的 state_dict_type 和相应配置。目标模块不必是 FSDP 模块。


 退货


 `StateDictSettings` 包含当前设置的 state_dict_type 和 state_dict /optim_state_dict 配置。


 如果 StateDictSettings 不同** –
* **FSDP 子模块不同，则引发 
* **AssertionError`。** –


 Return type


[*StateDictSettings*](#torch.distributed.fsdp.StateDictSettings "torch.distributed.fsdp.api.StateDictSettings")


*财产*


 模块 *：


[模块](generated/torch.nn.Module.html#torch.nn.Module"torch.nn.modules.module.Module")*[¶](#torch.distributed.fsdp.FullyShardedDataParallel.module"永久链接到此定义")


 返回包装的模块(如“DistributedDataParallel”)。


 命名_buffers


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.named_buffers)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.named_buffers "此定义的永久链接")


 覆盖 [`named_buffers()`](#torch.distributed.fsdp.FullyShardedDataParallel.named_buffers "torch.distributed.fsdp.FullyShardedDataParallel.named_buffers") 以拦截缓冲区名称并删除所有出现的 FSDP 特定扁平化缓冲区前缀。 [`summon_full_params()`](#torch.distributed.fsdp.FullyShardedDataParallel.summon_full_params "torch.distributed.fsdp.FullyShardedDataParallel.summon_full_params") 上下文管理器。


 Return type


[*Iterator*](https://docs.python.org/3/library/typing.html#typing.Iterator "(in Python v3.12)") [ [*Tuple*](https://docs. python.org/3/library/typing.html#typing.Tuple "(Python v3.12)") [ [str](https://docs.python.org/3/library/stdtypes.html#str " (在 Python v3.12 中)") , [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ]]


 命名_参数


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.named_pa​​rameters)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.named_pa​​rameters "此定义的永久链接")


 覆盖 [`named_parameters()`](#torch.distributed.fsdp.FullyShardedDataParallel.named_pa​​rameters "torch.distributed.fsdp.FullyShardedDataParallel.named_pa​​rameters") 以拦截参数名称并删除所有出现的 FSDP 特定扁平参数前缀。 [`summon_full_params()`](#torch.distributed.fsdp.FullyShardedDataParallel.summon_full_params "torch.distributed.fsdp.FullyShardedDataParallel.summon_full_params") 上下文管理器。


 Return type


[*Iterator*](https://docs.python.org/3/library/typing.html#typing.Iterator "(in Python v3.12)") [ [*Tuple*](https://docs. python.org/3/library/typing.html#typing.Tuple "(Python v3.12)") [ [str](https://docs.python.org/3/library/stdtypes.html#str " (在Python v3.12中)") , [*参数*](generated/torch.nn.parameter.Parameter.html#torch.nn.parameter.Parameter "torch.nn.parameter.Parameter") ]]


 没有_同步


 ( ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.no_sync)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.no_sync "此定义的永久链接")


 用于禁用跨 FSDP 实例的梯度同步的上下文管理器。在此上下文中，梯度将累积在模块变量中，稍后将在退出上下文后的第一次前向-后向传递中进行同步。这只能在根 FSDP 实例上使用，并将递归地应用于所有子 FSDP 实例。




!!! note "笔记"

    这可能会导致更高的内存使用量，因为 FSDP 将累积完整的模型梯度(而不是梯度分片)直到最终同步。


!!! note "笔记"

    当与 CPU 卸载一起使用时，梯度在上下文管理器内部时不会被卸载到 CPU。相反，它们只会在最终同步后立即被卸载。


 Return type


[*生成器*](https://docs.python.org/3/library/typing.html#typing.Generator“(在Python v3.12中)”)


*静止的*


 优化_state_dict


 (*模型*，*优化*，*优化_状态_dict



 =
 


 无
* , *组



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.optim_state_dict)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.optim_state_dict "此定义的永久链接")


 将 FSDP 分片的“model”的“optim”的 state_dict 转换为三种类型之一：1) 完整优化器状态_dict，2) 分片优化器状态_dict，3) 本地优化器状态_dict。


 对于完整优化器状态_dict，所有状态都是未展平的且未分片。仅Rank0和CPU只能通过[`state_dict_type()`](#torch.distributed.fsdp.FullyShardedDataParallel.state_dict_type "torch.distributed.fsdp.FullyShardedDataParallel.state_dict_type") 以避免 OOM。


 对于分片优化器 state_dict，所有状态均未展平，但 sharded.CPU 只能通过 [`state_dict_type()`](#torch.distributed.fsdp.FullyShardedDataParallel.state_dict_type "torch.distributed.fsdp.FullyShardedDataParallel.state_dict_type") 以进一步节省内存。


 对于本地状态_dict，不会执行任何转换。但状态将从 nn.Tensor 转换为 ShardedTensor 以表示其分片性质(尚不支持)。


 例子：


```
>>> from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
>>> from torch.distributed.fsdp import StateDictType
>>> from torch.distributed.fsdp import FullStateDictConfig
>>> from torch.distributed.fsdp import FullOptimStateDictConfig
>>> # Save a checkpoint
>>> model, optim = ...
>>> FSDP.set_state_dict_type(
>>>     model,
>>>     StateDictType.FULL_STATE_DICT,
>>>     FullStateDictConfig(rank0_only=False),
>>>     FullOptimStateDictConfig(rank0_only=False),
>>> )
>>> state_dict = model.state_dict()
>>> optim_state_dict = FSDP.optim_state_dict(model, optim)
>>> save_a_checkpoint(state_dict, optim_state_dict)
>>> # Load a checkpoint
>>> model, optim = ...
>>> state_dict, optim_state_dict = load_a_checkpoint()
>>> FSDP.set_state_dict_type(
>>>     model,
>>>     StateDictType.FULL_STATE_DICT,
>>>     FullStateDictConfig(rank0_only=False),
>>>     FullOptimStateDictConfig(rank0_only=False),
>>> )
>>> model.load_state_dict(state_dict)
>>> optim_state_dict = FSDP.optim_state_dict_to_load(
>>>     optim_state_dict, model, optim
>>> )
>>> optim.load_state_dict(optim_state_dict)

```


 参数 
* **model** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 根模块(可能或可能不是参数被传递到优化器“optim”的 [`FullyShardedDataParallel`](#torch.distributed.fsdp.FullyShardedDataParallel "torch.distributed.fsdp.FullyShardedDataParallel") 实例)。
* **optim** ( [*torch. optim.Optimizer*](optim.html#torch.optim.Optimizer "torch.optim.Optimizer") ) – `model` 参数的优化器。
* **optim_state_dict** ( *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *Any
* *]
* ) – 目标优化器状态_dict 进行转换。如果值为 None，则将使用 optim.state_dict()。 (默认值： `None` )
* **group** ( *dist.ProcessGroup
* ) – 模型的进程组，参数在其中进行分片；如果使用默认进程组，则为“None”。 (默认值：`无`)


 退货


 包含 `model` 优化器状态的 [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") 。优化器状态的分片基于 `state_dict_type` 。


 Return type


 Dict[ [str](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") , Any]


*静止的*


 优化_state_dict_to_load


 ( *model
* , *optim
* , *optim_state_dict
* , *is_named_optimizer



 =
 


 False
* , *直接加载_



 =
 


 假*，*组



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.optim_state_dict_to_load)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.optim_state_dict_to_load "此定义的永久链接")


 给定一个通过 [`optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.optim_state_dict") 转换的 `optim_state_dict` ，将其转换到可以加载到“optim”的扁平化的optimizerstate_dict，它是“model”的优化器。 `model` 必须通过FullyShardedDataParallel 进行分片。


```
>>> from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
>>> from torch.distributed.fsdp import StateDictType
>>> from torch.distributed.fsdp import FullStateDictConfig
>>> from torch.distributed.fsdp import FullOptimStateDictConfig
>>> # Save a checkpoint
>>> model, optim = ...
>>> FSDP.set_state_dict_type(
>>>     model,
>>>     StateDictType.FULL_STATE_DICT,
>>>     FullStateDictConfig(rank0_only=False),
>>>     FullOptimStateDictConfig(rank0_only=False),
>>> )
>>> state_dict = model.state_dict()
>>> original_osd = optim.state_dict()
>>> optim_state_dict = FSDP.optim_state_dict(
>>>     model,
>>>     optim,
>>>     optim_state_dict=original_osd
>>> )
>>> save_a_checkpoint(state_dict, optim_state_dict)
>>> # Load a checkpoint
>>> model, optim = ...
>>> state_dict, optim_state_dict = load_a_checkpoint()
>>> FSDP.set_state_dict_type(
>>>     model,
>>>     StateDictType.FULL_STATE_DICT,
>>>     FullStateDictConfig(rank0_only=False),
>>>     FullOptimStateDictConfig(rank0_only=False),
>>> )
>>> model.load_state_dict(state_dict)
>>> optim_state_dict = FSDP.optim_state_dict_to_load(
>>>     optim_state_dict, model, optim
>>> )
>>> optim.load_state_dict(optim_state_dict)

```


 参数 
* **model** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 根模块(可能或可能不是参数被传递到优化器“optim”的 [`FullyShardedDataParallel`](#torch.distributed.fsdp.FullyShardedDataParallel "torch.distributed.fsdp.FullyShardedDataParallel") 实例)。
* **optim** ( [*torch. optim.Optimizer*](optim.html#torch.optim.Optimizer "torch.optim.Optimizer") ) – `model` 参数的优化器。
* **optim_state_dict** ( *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *Any
* *]
* ) – 优化器指出被加载。
* **is_named_optimizer** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 此优化器是 NamedOptimizer 还是 KeyedOptimizer。仅当“optim”是 TorchRec 的 KeyedOptimizer 或 torch.distributed 的 NamedOptimizer 时才设置为 True。
* **load_directly** ( [*bool*](https://docs.python.org/3/library/functions.html #bool "(in Python v3.12)") ) – 如果设置为 True，此 API 还将在返回结果之前调用 optim.load_state_dict(result)。否则，用户有责任调用 `optim.load_state_dict(result)。 load_state_dict()` (默认值： `False` )
* **group** ( *dist.ProcessGroup
* ) – 参数被分片的模型进程组，如果使用默认进程组则为“None”。 (默认值：`无`)


 Return type


[*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(Python v3.12)") , [*Any*](https://docs.python.org/3/library/typing.html#typing.Any " (在 Python v3.12 中)") ]


 注册_comm_hook


 ( *state
* , *hook
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.register_comm_hook)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.register_comm_hook "此定义的永久链接")


 注册一个通信钩子，这是一项增强功能，为用户提供灵活的钩子，用户可以指定 FSDP 如何跨多个工作线程聚合梯度。此钩子可用于实现多种算法，例如 [GossipGrad](https://arxiv.org/abs/1803.05880 )和梯度压缩，其中涉及使用 [`FullyShardedDataParallel`](#torch.distributed.fsdp.FullyShardedDataParallel "torch.distributed.fsdp.FullyShardedDataParallel") 进行训练时参数同步的不同通信策略。


!!! warning "警告"

     FSDP 通信挂钩应在运行初始前向传递之前注册，并且仅注册一次。


 参数 
* **state** ( [*object*](https://docs.python.org/3/library/functions.html#object "(in Python v3.12)") ) –


 传递给钩子以维护训练过程中的任何状态信息。示例包括梯度压缩中的错误反馈、[GossipGrad](https://arxiv.org/abs/1803.05880)中与下一个进行通信的peer等。它是本地的由每个worker存储并由worker上的所有梯度tensor共享。
* **hook** ( *Callable
* ) – 可调用，具有以下签名之一：1) `hook: Callable[torch.Tensor] -
> None ` ：该函数接受一个 Python tensor，它表示与此 FSDP 单元正在包装的模型相对应的所有变量的完整、扁平、未分片的梯度(未被其他 FSDP 子单元包装)。然后它执行所有必要的处理并返回 `None` ;2) `hook: Callable[torch.Tensor, torch.Tensor] -
> None` ：该函数接受两个 Python tensor，第一个表示相对于对应于的所有变量的完整、扁平、未分片的梯度该 FSDP 单元正在包装的模型(未被其他 FSDP 子单元包装)。后者表示一个预先确定大小的tensor，用于存储缩减后的分片梯度块。在这两种情况下，callable 都会执行所有必要的处理并返回“None”。具有签名 1 的 Callable 预计会处理 NO_SHAARD 情况下的梯度通信。Callables签名 2 预计可以处理分片情况下的梯度通信。


*静止的*


 重新加密_optim_state_dict


 ( *optim_state_dict
* 、 *optim_state_key_type
* 、 *model
* 、 *optim_input



 =
 


 无*，*最好



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.rekey_optim_state_dict)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.rekey_optim_state_dict "此定义的永久链接")


 重新设置优化器状态字典 `optim_state_dict` 的密钥以使用密钥类型 `optim_state_key_type` 。这可用于实现具有 FSDP 实例的模型和不具有 FSDP 实例的模型的优化器状态指令之间的兼容性。


 重新键入 FSDP 完整优化器状态字典(即来自 [`full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict") )使用参数 ID 并可加载到非包装模型：


```
>>> wrapped_model, wrapped_optim = ...
>>> full_osd = FSDP.full_optim_state_dict(wrapped_model, wrapped_optim)
>>> nonwrapped_model, nonwrapped_optim = ...
>>> rekeyed_osd = FSDP.rekey_optim_state_dict(full_osd, OptimStateKeyType.PARAM_ID, nonwrapped_model)
>>> nonwrapped_optim.load_state_dict(rekeyed_osd)

```


 要将普通优化器状态字典从非包装模型重新设置为可加载到包装模型：


```
>>> nonwrapped_model, nonwrapped_optim = ...
>>> osd = nonwrapped_optim.state_dict()
>>> rekeyed_osd = FSDP.rekey_optim_state_dict(osd, OptimStateKeyType.PARAM_NAME, nonwrapped_model)
>>> wrapped_model, wrapped_optim = ...
>>> sharded_osd = FSDP.shard_full_optim_state_dict(rekeyed_osd, wrapped_model)
>>> wrapped_optim.load_state_dict(sharded_osd)

```


 退货


 使用 `optim_state_key_type` 指定的参数键重新设置优化器状态字典的键。


 Return type


 Dict[ [str](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") , Any]


*静止的*


 分散_full_optim_state_dict


 ( *full_optim_state_dict
* , *model
* , *optim_input



 =
 


 无*，*最好



 =
 


 无
* , *组



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/fully_sharded_data_parallel.html#FullyShardedDataParallel.scatter_full_optim_state_dict)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.scatter_full_optim_state_dict"此定义的永久链接")


 将完整的优化器状态字典从等级 0 分散到所有其他等级，返回每个等级上的分片优化器状态字典。返回值与 [`shard_full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict") 相同，并且在rank0上，第一个参数应该是 [`full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict") 的返回值。


 例子：


```
>>> from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
>>> model, optim = ...
>>> full_osd = FSDP.full_optim_state_dict(model, optim)  # only non-empty on rank 0
>>> # Define new model with possibly different world size
>>> new_model, new_optim, new_group = ...
>>> sharded_osd = FSDP.scatter_full_optim_state_dict(full_osd, new_model, group=new_group)
>>> new_optim.load_state_dict(sharded_osd)

```


!!! note "笔记"

    [`shard_full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict") 和 [`scatter_full_optim_state\ _dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.scatter_full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.scatter_full_optim_state_dict") 可用于获取要加载的分片优化器状态字典。假设完整优化器状态字典驻留在CPU内存中，前者要求每个Rank在CPU内存中拥有完整的字典，其中每个Rank单独对字典进行分片而无需任何通信，而后者只需要Rank 0在CPU内存中拥有完整的字典，其中等级 0 将每个分片移动到 GPU 内存(对于 NCCL)并将其传达到适当的等级。因此，前者具有较高的总 CPU 内存成本，而后者具有较高的通信成本。


 参数 
* **full_optim_state_dict** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html# str "(in Python v3.12)")*,
* *Any
* *]
* *]
* ) – 对应于未展平参数的优化器状态字典，并在等级 0 时保持完整的非分片优化器状态；该参数在非零等级上被忽略。
* **model** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – Root模块(可能是也可能不是 [`FullyShardedDataParallel`](#torch.distributed.fsdp.FullyShardedDataParallel "torch.distributed.fsdp.FullyShardedDataParallel") 实例)，其参数对应于 `full_optim_state_dict 中的优化器状态`.
* **optim_input** ( *可选
* *[
* *联合
* *[
* *列表
* *[
* *字典
* *[
* [*str*](https://docs.python.org /3/library/stdtypes.html#str "(Python v3.12)")*,
* *任意
* *]
* *]
* *,
* *可迭代
* *[
* *torch.nn.Parameter
* *] 
* *]
* *]
* ) – 传递到优化器的输入表示 [`list`](https://docs.python.org/3/library/stdtypes.html#list "(在 Python v3.12 中) ") 参数组或可迭代参数；如果 `None` ，则此方法假定输入为 `model.parameters()` 。该参数已被弃用，不再需要传递它。 (默认：`无`)
* **optim** ( *可选
* *[
* [*torch.optim.Optimizer*](optim.html#torch.optim.Optimizer "torch.optim.Optimizer")*]
* ) – 将加载此方法返回的状态字典的优化器。这是优于 `optim_input` 的首选参数。 (默认值：`None`)
* **group** (*dist.ProcessGroup*) – 模型的进程组，如果使用默认进程组，则为`None`。 (默认值：`无`)


 退货


 完整的优化器状态字典现在重新映射到扁平化参数而不是未扁平化参数，并且仅限于仅包含该排名的优化器状态部分。


 Return type


 Dict[ [str](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") , Any]


*静止的*


 设置_state_dict_type


 ( *模块
* , *state_dict_type
* , *state_dict_config



 =
 


 无
* , *optim_state_dict_config



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.set_state_dict_type)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.set_state_dict_type "此定义的永久链接")


 设置目标模块的所有后代 FSDP 模块的 `state_dict_type` 和相应的(可选)配置。目标模块不必是 FSDP 模块。如果目标模块是 FSDP 模块，则其“state_dict_type”也会更改。




!!! note "笔记"

    仅应为顶级(根)模块调用此 API。


!!! note "笔记"

    在根 FSDP 模块被另​​一个“nn.Module”包装的情况下，该 API 使用户能够透明地使用传统的“state_dict” API 来获取模型检查点。例如，以下内容将确保在所有非 FSDP 实例上调用“state_dict”，同时分派到 FSDP 的分片_state_dict 实现：


 例子：


```
>>> model = DDP(FSDP(...))
>>> FSDP.set_state_dict_type(
>>>     model,
>>>     StateDictType.SHARDED_STATE_DICT,
>>>     state_dict_config = ShardedStateDictConfig(offload_to_cpu=True),
>>>     optim_state_dict_config = OptimStateDictConfig(offload_to_cpu=True),
>>> )
>>> param_state_dict = model.state_dict()
>>> optim_state_dict = FSDP.optim_state_dict(model, optim)

```


 参数 
* **module** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 根模块。
* **状态_dict_type** ( *StateDictType
* ) – 要设置的所需 `state_dict_type`。
* **state_dict_config** ( *可选
* *[
* [*StateDictConfig*](#torch. distribution.fsdp.StateDictConfig "torch.distributed.fsdp.StateDictConfig")*]
* ) – 目标 `state_dict_type` 的配置。


 退货


 StateDictSettings 包括先前的 state_dict 类型和模块的配置。


 Return type


[*StateDictSettings*](#torch.distributed.fsdp.StateDictSettings "torch.distributed.fsdp.api.StateDictSettings")


*静止的*


 分片_full_optim_state_dict


 ( *full_optim_state_dict
* , *model
* , *optim_input



 =
 


 无*，*最好



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.shard_full_optim_state_dict)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict "此定义的永久链接")


 通过将状态重新映射到扁平化参数而不是未扁平化参数并仅限于优化器状态的该等级部分，对完整优化器状态字典“full_optim_state_dict”进行分片。第一个参数应该是 [`full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict") 的返回值。


 例子：


```
>>> from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
>>> model, optim = ...
>>> full_osd = FSDP.full_optim_state_dict(model, optim)
>>> torch.save(full_osd, PATH)
>>> # Define new model with possibly different world size
>>> new_model, new_optim = ...
>>> full_osd = torch.load(PATH)
>>> sharded_osd = FSDP.shard_full_optim_state_dict(full_osd, new_model)
>>> new_optim.load_state_dict(sharded_osd)

```


!!! note "笔记"

    [`shard_full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.shard_full_optim_state_dict") 和 [`scatter_full_optim_state\ _dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.scatter_full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.scatter_full_optim_state_dict") 可用于获取要加载的分片优化器状态字典。假设完整优化器状态字典驻留在CPU内存中，前者要求每个Rank在CPU内存中拥有完整的字典，其中每个Rank单独对字典进行分片而无需任何通信，而后者只需要Rank 0在CPU内存中拥有完整的字典，其中等级 0 将每个分片移动到 GPU 内存(对于 NCCL)并将其传达到适当的等级。因此，前者具有较高的总 CPU 内存成本，而后者具有较高的通信成本。


 参数 
* **full_optim_state_dict** ( *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3 中).12)")*,
* *Any
* *]
* ) – 优化器状态字典对应于未展平的参数并保存完整的非分片优化器状态。
* **模型** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 根模块(可能是也可能不是 [`FullyShardedDataParallel`](#torch.distributed.fsdp.FullyShardedDataParallel "torch.distributed.fsdp.FullyShardedDataParallel") 实例)，其参数对应于 `full_optim_state_dict` 中的优化器状态。
* **optim_input** ( *可选
* *[
* *Union
* *[
* *List 
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *任何
* *]
* *]
* *,
* *Iterable
* *[
* *torch.nn.Parameter
* *]
* *]
* *]
* ) – 传递到优化器的输入表示 [`list`](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)") 参数组或可迭代参数；如果 `None` ，则此方法假定输入为 `model.parameters()`.该参数已被弃用，不再需要传递它。 (默认：`无`)
* **optim** ( *可选
* *[
* [*torch.optim.Optimizer*](optim.html#torch.optim.Optimizer "torch.optim.Optimizer")*]
* ) – 将加载此方法返回的状态字典的优化器。这是优于 `optim_input` 的首选参数。 (默认值：`无`)


 退货


 完整的优化器状态字典现在重新映射到扁平化参数而不是未扁平化参数，并且仅限于仅包含该排名的优化器状态部分。


 Return type


 Dict[ [str](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") , Any]


*静止的*


 分片_optim_state_dict


 (*模型*，*优化*，*组



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.sharded_optim_state_dict)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.sharded_optim_state_dict "此定义的永久链接")


 该 API 类似于 [`full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict")，但此 API 块均为非零维度状态为“ShardedTensor”以节省内存。仅当模型“state_dict”是通过上下文管理器“with state_dict_type(SHAARDED_STATE_DICT):”派生时才应使用此 API。


 详细使用方法请参考 [`full_optim_state_dict()`](#torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict "torch.distributed.fsdp.FullyShardedDataParallel.full_optim_state_dict") 。


!!! warning "警告"

     返回的状态字典包含 `ShardedTensor` ，不能直接被常规的 `optim.load_state_dict` 使用。


 Return type


[*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(Python v3.12)") , [*Any*](https://docs.python.org/3/library/typing.html#typing.Any " (在 Python v3.12 中)") ]


*静止的*


 状态_dict_type


 ( *模块
* , *state_dict_type
* , *state_dict_config



 =
 


 无
* , *optim_state_dict_config



 =
 


 无
* ) [[source]](_modules/torch/distributed/fsdp/fully_sharded_data_parallel.html#FullyShardedDataParallel.state_dict_type)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.state_dict_type "此定义的永久链接")


 一个上下文管理器，用于设置目标模块的所有后代 FSDP 模块的“state_dict_type”。此上下文管理器具有与 [`set_state_dict_type()`](#torch.distributed.fsdp.FullyShardedDataParallel.set_state_dict_type "torch.distributed.fsdp.FullyShardedDataParallel.set_state_dict_type") 相同的功能。有关详细信息，请阅读 [`set_state_dict_type()`](#torch.distributed.fsdp.FullyShardedDataParallel.set_state_dict_type "torch.distributed.fsdp.FullyShardedDataParallel.set_state_dict_type") 文档。


 例子：


```
>>> model = DDP(FSDP(...))
>>> with FSDP.state_dict_type(
>>>     model,
>>>     StateDictType.SHARDED_STATE_DICT,
>>> ):
>>>     checkpoint = model.state_dict()

```


 参数 
* **module** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 根模块。
* **状态_dict_type** ( *StateDictType
* ) – 要设置的所需 `state_dict_type`。
* **state_dict_config** ( *可选
* *[
* [*StateDictConfig*](#torch. distribution.fsdp.StateDictConfig "torch.distributed.fsdp.StateDictConfig")*]
* ) – 目标 `state_dict_type` 的模型 `state_dict` 配置。
* **optim_state_dict_config
* 
* ( *Optional
* *[
* [*OptimStateDictConfig*](#torch.distributed.fsdp.OptimStateDictConfig "torch.distributed.fsdp.OptimStateDictConfig")*]
* ) – 目标 `state` 的优化器 `state_dict` 配置_dict_type` 。


 Return type


[*生成器*](https://docs.python.org/3/library/typing.html#typing.Generator“(在Python v3.12中)”)


*静止的*


 召唤_full_params


 (*模块*，*递归



 =
 


 真*，*回写



 =
 


 真
* , *rank0_only



 =
 


 False
* , *卸载_to_cpu



 =
 


 假
* , *与_grads



 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/filled_sharded_data_parallel.html#FullyShardedDataParallel.summon_full_params)[¶](#torch.distributed.fsdp.FullyShardedDataParallel.summon_full_params "此定义的永久链接")


 用于公开 FSDP 实例的完整参数的上下文管理器。在模型向前/向后获取参数以进行额外处理或检查时非常有用。它可以采用非 FSDP 模块，并将根据“recurse”参数调用所有包含的 FSDP 模块及其子模块的完整参数。




!!! note "笔记"

    这可用于内部 FSDP。


!!! note "笔记"

    这不能在前向或后向传递中使用。前进和后退也不能从这种背景下开始。


!!! note "笔记"

    contextmanager退出后参数将恢复到本地分片，存储行为与forward相同。


!!! note "笔记"

    可以修改完整参数，但在上下文管理器退出后，只有与本地参数分片相对应的部分会保留(除非 `writeback=False` ，在这种情况下更改将被丢弃)。在 FSDP 不对参数进行分片的情况下，目前仅当 `world_size == 1` 或 `NO_SHARD` 配置时，无论 `writeback` 如何，修改都会被持久化。


!!! note "笔记"

    此方法适用于本身不是 FSDP 但可能包含多个独立 FSDP 单元的模块。在这种情况下，给定的参数将适用于所有包含的 FSDP 单位。


!!! warning "警告"

     请注意，当前不支持“rank0_only=True”与“writeback=True”结合使用，并且会引发错误。这是因为上下文中不同等级的模型参数形状会有所不同，并且在退出上下文时写入它们可能会导致不同等级之间的不一致。


!!! warning "警告"

     请注意，“offload_to_cpu”和“rank0_only=False”将导致驻留在同一机器上的 GPU 的完整参数被冗余复制到 CPU 内存，这可能会带来 CPU OOM 的风险。建议使用 `offload_to_cpu` 和 `rank0_only=True` 。


 参数 
* **recurse** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 递归调用嵌套FSDP实例的所有参数(默认值：True)。
* **writeback** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(Python 中) v3.12)")*,
* *可选
* ) – 如果 `False` ，则在上下文管理器退出后丢弃对参数的修改；禁用此功能可能会稍微更有效(默认值：True)
* **rank0_only** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `True` ，完整参数仅在全局等级 0 上具体化。这意味着在上下文中，只有等级 0 将具有完整参数，其他等级将具有分片参数。请注意，不支持将“rank0_only=True”设置为“writeback=True”，因为模型参数形状在上下文中的各个rank中会有所不同，并且在退出上下文时写入它们可能会导致各个rank之间的不一致。
* **卸载_to_cpu** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果为“True”，则完整参数将卸载到 CPU。请注意，当前仅在参数分片时才会发生此卸载(仅在 world_size = 1 或“NO_SHARD”配置的情况下不是这种情况)。建议使用“offload_to_cpu”和“rank0_only=True”，以避免将模型参数的冗余副本卸载到同一 CPU 内存。
* **with_grads** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `True` ，梯度也不会与参数分片。目前，仅当将 `use_orig_params=True` 传递给 FSDP 构造函数并将 `offload_to_cpu=False` 传递给此方法时才支持此功能。(默认值： `False` )


 Return type


[*生成器*](https://docs.python.org/3/library/typing.html#typing.Generator“(在Python v3.12中)”)


*班级*


 torch.distributed.fsdp。


 向后预取


 ( *value
* ) [[source]](_modules/torch/distributed/fsdp/api.html#BackwardPrefetch)[¶](#torch.distributed.fsdp.BackwardPrefetch "此定义的永久链接")


 这配置了显式向后预取，通过在向后传递中启用通信和计算重叠来提高吞吐量，但代价是稍微增加内存使用量。



* `BACKWARD_PRE` ：这会实现最多的重叠，但会增加最多的内存使用量。这会在当前参数集的梯度计算之前*预取下一组参数。这与 *nextall-gather
* 和 *当前梯度计算
* 重叠，并且在峰值时，它在内存中保存当前参数集、下一组参数和当前梯度集。
* `BACKWARD_POST` ：这可以减少重叠但需要较少的内存使用。这会在当前参数集的梯度计算之后预取下一组参数。这与*当前的reduce-scatter*和*下一个梯度计算*重叠，并且它在为下一组参数分配内存之前释放当前的参数集，仅在峰值时将下一组参数和当前的梯度集保存在内存中。
* FSDP 的 `backward_prefetch` 参数接受 `None` ，这会完全禁用向后预取。这没有重叠，也不会增加内存使用量。一般来说，我们不建议使用此设置，因为它可能会显着降低吞吐量。


 对于更多技术背景：对于使用 NCCL 后端的单个进程组，任何集合体，即使从不同的流发出，也会争用相同的每设备 NCCL 流，这意味着发出集合体的相对顺序对于重叠很重要。两个向后预取值对应不同的发布顺序。


*班级*


 torch.distributed.fsdp。


 分片策略


 ( *value
* ) [[source]](_modules/torch/distributed/fsdp/api.html#ShardingStrategy)[¶](#torch.distributed.fsdp.ShardingStrategy "此定义的永久链接")


 这指定了 [`FullyShardedDataParallel`](#torch.distributed.fsdp.FullyShardedDataParallel "torch.distributed.fsdp.FullyShardedDataParallel") 用于分布式训练的分片策略。



* `FULL_SHARD` ：参数、梯度和优化器状态被分片。对于参数，该策略在前向计算之前取消分片(通过全收集)，在前向计算之后重新分片，在后向计算之前取消分片，在后向计算之后重新分片。对于梯度，它在反向计算后同步并分片它们(通过reduce-scatter)。分片优化器状态按等级在本地更新。
* `SHARD_GRAD_OP` ：梯度和优化器状态在计算期间进行分片，此外，参数在计算之外进行分片。对于参数，该策略在前向计算之前取消分片，在前向计算之后不重新分片，仅在后向计算之后重新分片。分片优化器状态按等级本地更新。在 `no_sync()` 内部，参数在向后计算后不会重新分片。
* `NO_SHAARD` ：参数、梯度和优化器状态不会分片，而是跨等级复制，类似于 PyTorch 的 `DistributedDataParallel` API。对于梯度，该策略在向后计算后同步它们(通过 all-reduce)。未分片的优化器状态按等级在本地更新。
* `HYBRID_SHARD` ：在节点内应用 `FULL_SHARD`，并跨节点复制参数。这会导致通信量减少，因为昂贵的全收集和减少分散仅在节点内完成，这对于中型模型来说性能更高。 
* `_HYBRID_SHARD_ZERO2` ：应用 `SHARD_GRAD_OP`在节点内，并跨节点复制参数。这类似于“HYBRID_SHARD”，不同之处在于这可能提供更高的吞吐量，因为在前向传递后未释放未分片的参数，从而将所有收集保存在预向后中。


*班级*


 torch.distributed.fsdp。


 混合精度


 ( *param_dtype=None
* , *reduce_dtype=None
* , *buffer_dtype=None
* , *keep_low_ precision_grads=False
* , *cast_forward_inputs=False
* , *cast\ _root_forward_inputs=True
* , *_module_classes_to_ignore=(<class 'torch.nn.modules.batchnorm._BatchNorm'>
* , *)
* ) [[source]](_modules/torch /distributed/fsdp/api.html#MixedPrecision)[¶](#torch.distributed.fsdp.MixedPrecision "此定义的永久链接")


 这将配置 FSDP 原生混合精度训练。


 变量 
* **param_dtype** ( *可选
* *[
* [*torch.dtype*](tensor_attributes.html#torch.dtype "torch.dtype")*]
* ) – 指定转发期间模型参数的 dtype和后向，以及用于前向和后向计算的数据类型。除了向前和向后之外，*分片*参数保持全精度(例如，对于优化器步骤)，并且对于模型检查点，参数始终以全精度保存。 (默认： `None` )
* **reduce_dtype** ( *可选
* *[
* [*torch.dtype*](tensor_attributes.html#torch.dtype "torch.dtype")*]
* ) – 这指定梯度减少的 dtype(即减少分散或全部减少)。如果这是 `None` 但 `param_dtype` 不是 `None` ，那么它采用 `param_dtype` 值，仍然以低精度运行梯度下降。这允许与 `param_dtype` 不同，例如强制梯度缩减以全精度运行。 (默认： `None` )
* **buffer_dtype** ( *可选
* *[
* [*torch.dtype*](tensor_attributes.html#torch.dtype "torch.dtype")*]
* ) – 这指定缓冲区的数据类型。 FSDP 不会对缓冲区进行分片。相反，FSDP 在第一次前向传递中将它们转换为“buffer_dtype”，然后将它们保留在该 dtype 中。对于模型检查点，除了 `LOCAL_STATE_DICT` 之外，缓冲区均以全精度保存。 (默认：`None`)
* **keep_low_ precision_grads** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)") ) – 如果 `False` ，则 FSDP 在向后传递后将梯度向上转换为全精度，为优化器步骤做准备。如果为 True ，则 FSDP 将梯度保留在用于梯度缩减的数据类型中，如果使用支持低精度运行的自定义优化器，可以节省内存。(默认值：False )
* **cast_forward_inputs** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` ，则此 FSDP 模块将向前传递参数并kwargs 为 `param_dtype` 。这是为了确保参数和输入数据类型匹配正向计算，正如许多操作所要求的那样。当仅将混合精度应用于某些但不是全部 FSDP 模块时，可能需要将其设置为“True”，在这种情况下，混合精度 FSDP 子模块需要重新转换其输入。(默认值：“False”)
* **cast_root\ _forward_inputs** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` ，则根 FSDP 模块将其前向参数和 kwargs 转换为 `param_dtype` ，覆盖 `cast_forward_inputs` 的值。对于非根 FSDP 模块，这不会执行任何操作。 (默认：`True`)
* **_module_classes_to_ignore** ( *序列
* *[
* *类型
* *[
* [*torch.nn.modules.module.Module*](generated/火炬.nn.Module.html#torch.nn.Module "torch.nn.modules.module.Module")*]
* *]
* ) – (Sequence[Type[nn.Module]])：指定要忽略的模块类使用“auto_wrap_policy”时的混合精度：这些类的模块将单独应用 FSDP，并禁用混合精度(这意味着最终的 FSDP 构造将偏离指定的策略)。如果未指定“auto_wrap_policy”，则不会执行任何操作。此 API 是实验性的，可能会发生变化。(默认值：`(_BatchNorm,)` )



!!! note "笔记"

    此 API 是实验性的，可能会发生变化。


!!! note "笔记"

    仅浮点tensor被转换为其指定的数据类型。


!!! note "笔记"

    在`summon_full_params`中，参数被强制为全精度，但缓冲区不是。


!!! note "笔记"

    即使它们的输入是低精度(如“float16”或“bfloat16”)，层范数和批量范数也会在“float32”中累积。禁用这些范数模块的 FSDP 混合精度仅意味着仿射参数保留在“float32”中。然而，这会导致这些标准模块的分离全聚集和减少分散，这可能效率低下，因此如果工作量允许，用户应该更愿意仍然对这些模块应用混合精度。


!!! note "笔记"

    默认情况下，如果用户传递带有任何 `_BatchNorm` 模块的模型并指定 `auto_wrap_policy` ，则批量标准化模块将单独应用 FSDP，并禁用混合精度。请参阅“_module_classes_to_ignore”参数。


!!! note "笔记"

    `MixedPrecision` 默认有 `cast_root_forward_inputs=True` 和 `cast_forward_inputs=False`。对于根 FSDP 实例，其 `cast_root_forward_inputs` 优先于其 `cast_forward_inputs` 。对于非根 FSDP 实例，其 `cast_root_forward_inputs` 值将被忽略。对于每个 FSDP 实例具有相同“MixedPrecision”配置并且只需要在模型前向传递开始时将输入强制转换为“param_dtype”的典型情况，默认设置就足够了。


!!! note "笔记"

    对于具有不同 `MixedPrecision` 配置的嵌套 FSDP 实例，我们建议在每个实例的前向之前设置单独的 `cast_forward_inputs` 值以配置是否配置转换输入。在这种情况下，由于转换发生在每个 FSDP 实例转发之前，因此父 FSDP 实例应在其 FSDP 子模块之前运行其非 FSDP 子模块，以避免由于不同的“MixedPrecision”配置而更改激活数据类型。


 例子：


```
>>> model = nn.Sequential(nn.Linear(3, 3), nn.Linear(3, 3))
>>> model[1] = FSDP(
>>>     model[1],
>>>     mixed_precision=MixedPrecision(param_dtype=torch.float16, cast_forward_inputs=True),
>>> )
>>> model = FSDP(
>>>     model,
>>>     mixed_precision=MixedPrecision(param_dtype=torch.bfloat16, cast_forward_inputs=True),
>>> )

```


 上面显示了一个工作示例。另一方面，如果将 model[1] 替换为 model[0] ，意味着使用不同 MixedPrecision 的子模块首先向前运行，则 model[1] 将错误地看到 float16 激活`bfloat16` 的。


*班级*


 torch.distributed.fsdp。


 CPU卸载


 (*卸载_params



 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/api.html#CPUOffload)[¶](#torch.distributed.fsdp.CPUOffload "此定义的永久链接")


 这将配置 CPU 卸载。


 变量


**offload_params** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 指定是否卸载当不参与计算时将参数传递给CPU。如果为 True ，那么这也会将梯度卸载到 CPU，这意味着优化器步骤在 CPU 上运行。


*班级*


 torch.distributed.fsdp。


 状态字典配置


 (*卸载_到_cpu



 =
 


 False
* , *使用_dtensor



 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/api.html#StateDictConfig)[¶](#torch.distributed.fsdp.StateDictConfig "此定义的永久链接")


`StateDictConfig` 是所有 `state_dict` 配置类的基类。用户应该实例化一个子类(例如 `FullStateDictConfig` )，以便配置 FSDP 支持的相应 `state_dict` 类型的设置。


 变量 
* **offload_to_cpu** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – If `True` ，则 FSDP 将状态字典值卸载到 CPU，如果是 `False` ，则 FSDP 将它们保留在 GPU 上。(默认值：`False` )
* **use_dtensor** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果为 `True` ，则 FSDP 将状态字典值保存为 `DTensor` (如果该值已分片)，如果为 False ，则 FSDP 将它们保存为 ShardedTensor 。 (默认值：`False`)


*班级*


 torch.distributed.fsdp。


 完整状态字典配置


 (*卸载_到_cpu



 =
 


 False
* , *使用_dtensor



 =
 


 假
* , *rank0_only



 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/api.html#FullStateDictConfig)[¶](#torch.distributed.fsdp.FullStateDictConfig "此定义的永久链接")


`FullStateDictConfig` 是一个配置类，旨在与 `StateDictType.FULL_STATE_DICT` 一起使用。我们建议在保存完整状态字典时启用“offload_to_cpu=True”和“rank0_only=True”，以分别节省 GPU 内存和 CPU 内存。该配置类旨在通过 `state_dict_type()` 上下文管理器使用，如下所示：


```
>>> from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
>>> fsdp = FSDP(model, auto_wrap_policy=...)
>>> cfg = FullStateDictConfig(offload_to_cpu=True, rank0_only=True)
>>> with FSDP.state_dict_type(fsdp, StateDictType.FULL_STATE_DICT, cfg):
>>>     state = fsdp.state_dict()
>>>     # `state` will be empty on non rank 0 and contain CPU tensors on rank 0.
>>> # To reload checkpoint for inference, finetuning, transfer learning, etc:
>>> model = model_fn() # Initialize model on CPU in preparation for wrapping with FSDP
>>> if dist.get_rank() == 0:
>>>     # Load checkpoint only on rank 0 to avoid memory redundancy
>>>     state_dict = torch.load("my_checkpoint.pt")
>>>     model.load_state_dict(state_dict)
>>> # All ranks initialize FSDP module as usual. `sync_module_states` argument
>>> # communicates loaded checkpoint states from rank 0 to rest of the world.
>>> fsdp = FSDP(model, device_id=torch.cuda.current_device(), auto_wrap_policy=..., sync_module_states=True)
>>> # After this point, all ranks have FSDP model with loaded checkpoint.

```


 变量


**rank0_only** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` ，那么只有等级 0 保存完整的状态字典，非零等级保存一个空字典。如果为 False ，则所有等级都会保存完整的状态字典。 (默认值：`False`)


*班级*


 torch.distributed.fsdp。


 ShardedStateDictConfig


 (*卸载_到_cpu



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 =
 


 False
* , *使用_dtensor



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/api.html#ShardedStateDictConfig)[¶](#torch.distributed.fsdp.ShardedStateDictConfig "此定义的永久链接")


*班级*


 torch.distributed.fsdp。


 本地状态字典配置


 (*卸载_到_cpu



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 =
 


 False
* , *使用_dtensor



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/api.html#LocalStateDictConfig)[¶](#torch.distributed.fsdp.LocalStateDictConfig "此定义的永久链接")


*班级*


 torch.distributed.fsdp。


 OptimStateDictConfig


 (*卸载_到_cpu



 =
 


 True
* , *使用_dtensor



 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/api.html#OptimStateDictConfig)[¶](#torch.distributed.fsdp.OptimStateDictConfig "此定义的永久链接")


`OptimStateDictConfig` 是所有 `optim_state_dict` 配置类的基类。用户应该实例化一个子类(例如 `FullOptimStateDictConfig` )，以便配置 FSDP 支持的相应 `optim_state_dict` 类型的设置。


 变量 
* **offload_to_cpu** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – If如果为“True”，则 FSDP 将状态字典的tensor值卸载到 CPU，如果为“False”，则 FSDP 将它们保留在原始设备上(除非启用了参数 CPU 卸载，否则为 GPU)。 (默认：`True`)
* **使用_dtensor** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)" ) ) – 如果为“True”，则 FSDP 将状态字典值保存为“DTensor”(如果该值已分片)；如果为“False”，则 FSDP 将其保存为“ShardedTensor”。 (默认值：`False`)


*班级*


 torch.distributed.fsdp。


 FullOptimStateDictConfig


 (*卸载_到_cpu



 =
 


 True
* , *使用_dtensor



 =
 


 假
* , *rank0_only



 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/api.html#FullOptimStateDictConfig)[¶](#torch.distributed.fsdp.FullOptimStateDictConfig "此定义的永久链接")


 变量


**rank0_only** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` ，那么只有等级 0 保存完整的状态字典，非零等级保存一个空字典。如果为 False ，则所有等级都会保存完整的状态字典。 (默认值：`False`)


*班级*


 torch.distributed.fsdp。


 ShardedOptimStateDictConfig


 (*卸载_到_cpu



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 =
 


 True
* , *使用_dtensor



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/api.html#ShardedOptimStateDictConfig)[¶](#torch.distributed.fsdp.ShardedOptimStateDictConfig "此定义的永久链接")


*班级*


 torch.distributed.fsdp。


 LocalOptimStateDictConfig


 (*卸载_到_cpu



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 =
 


 False
* , *使用_dtensor



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 =
 


 False
* ) [[source]](_modules/torch/distributed/fsdp/api.html#LocalOptimStateDictConfig)[¶](#torch.distributed.fsdp.LocalOptimStateDictConfig "此定义的永久链接")


*班级*


 torch.distributed.fsdp。


 状态字典设置


 ( *state_dict_type



 :
 


 torch.distributed.fsdp.api.StateDictType
* , *state_dict_config



 :
 


[torch.distributed.fsdp.api.StateDictConfig](#torch.distributed.fsdp.StateDictConfig "torch.distributed.fsdp.api.StateDictConfig")
* , *optim_state_dict_config



 :
 


[torch.distributed.fsdp.api.OptimStateDictConfig](#torch.distributed.fsdp.OptimStateDictConfig "torch.distributed.fsdp.api.OptimStateDictConfig")
* ) [[source]](_modules/torch/distributed/fsdp/api. html#StateDictSettings)[¶](#torch.distributed.fsdp.StateDictSettings"此定义的永久链接")