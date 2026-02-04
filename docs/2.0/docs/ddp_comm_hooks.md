# DDP 通信挂钩 [¶](#ddp-communication-hooks "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/ddp_comm_hooks>
>
> 原始地址：<https://pytorch.org/docs/stable/ddp_comm_hooks.html>


 DDP 通信钩子是一个通用接口，用于通过覆盖 [DistributedDataParallel](https://pytorch.org/docs/stable/generated/torch.nn.parallel.DistributedDataParallel.html#torch) 中的普通 allreduce 来控制如何在工作人员之间进行梯度通信。 nn.parallel.DistributedDataParallel.)。提供了一些内置的通信钩子，用户可以轻松应用这些钩子中的任何一个来优化通信。此外，钩子接口还可以支持用户定义的通信策略，以实现更高级的用例。


## 如何使用通信钩子？ [¶](#how-to-use-a-communication-hook"此标题的永久链接")


 要使用通信钩子，用户只需让 DDP 模型在训练循环之前注册钩子，如下所示。


[`torch.nn.parallel.DistributedDataParallel.register_comm_hook()`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel.register_comm_hook "torch.nn.parallel.DistributedDataParallel.注册通讯钩子”)


## 通信钩子的作用是什么？ [¶](#what-does-a-communication-hook-operate-on"此标题的永久链接")


 通信钩子提供了一种灵活的方法来进行 allreduce 梯度。因此，它主要对 allreduce 之前每个副本上的梯度进行操作，将其分桶以增加通信和计算之间的重叠。特别是，[`torch.distributed.GradBucket`](# torch.distributed.GradBucket "torch.distributed.GradBucket") 表示要全部归约的梯度tensor桶。


*班级*


 火炬分布式。


 GradBucket [¶](#torch.distributed.GradBucket "此定义的永久链接")


 该类主要将一个扁平化的梯度tensor(由[`buffer()`](#torch.distributed.GradBucket.buffer "torch.distributed.GradBucket.buffer")返回)传递给DDP通信钩子。这个tensor可以进一步分解为此存储桶中的每个参数tensor的列表(由 `get_per_parameter_tensors()` 返回)以应用分层操作。


 torch.distributed.GradBucket。



 index
 


 ( *自己



 :
 


[火炬._C._distributed_c10d.GradBucket](#torch.distributed.GradBucket“火炬._C._distributed_c10d.GradBucket”)*)


 → [int](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)")


[¶](#torch.distributed.GradBucket.index"此定义的永久链接")


!!! warning "警告"

     由于存储桶是在第一次迭代后重建的，因此不应依赖训练开始时的索引。


 退货


 存储几个连续层的梯度的桶的索引。所有梯度都被分桶化。


 torch.distributed.GradBucket。


 缓冲


 ( *自己



 :
 


[火炬._C._distributed_c10d.GradBucket](#torch.distributed.GradBucket“火炬._C._distributed_c10d.GradBucket”)*)


 → [torch.Tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.distributed.GradBucket.buffer"此定义的永久链接")


 退货


 扁平化的一维“torch.Tensor”缓冲区，可以进一步分解为该存储桶内的每个参数tensor的列表。


 torch.distributed.GradBucket。


 渐变


 ( *自己



 :
 


[火炬._C._distributed_c10d.GradBucket](#torch.distributed.GradBucket“火炬._C._distributed_c10d.GradBucket”)*)


 →
 


 List
 


 [ [torch.Tensor](tensors.html#torch.Tensor "torch.Tensor")


 ]
 


[¶](#torch.distributed.GradBucket.gradients"此定义的永久链接")


 退货


 `torch.Tensor` 列表。列表中的每个tensor对应一个梯度。


 torch.distributed.GradBucket。


 是_最后一个


 ( *自己



 :
 


[火炬._C._distributed_c10d.GradBucket](#torch.distributed.GradBucket“火炬._C._distributed_c10d.GradBucket”)*)


 → [bool](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)")


[¶](#torch.distributed.GradBucket.is_last "此定义的永久链接")


 退货


 该桶是否是迭代中最后一个进行 allreduce 的桶。这也意味着该桶对应于前向传递中的前几层。


 torch.distributed.GradBucket。


 设置_缓冲区


 ( *自己



 :
 


[torch._C._distributed_c10d.GradBucket](#torch.distributed.GradBucket "torch._C._distributed_c10d.GradBucket")
* , *缓冲区



 :
 


[torch.Tensor](tensors.html#torch.Tensor "torch.Tensor")
* )


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.distributed.GradBucket.set_buffer"此定义的永久链接")


 将桶中的tensor替换为输入tensor缓冲区。


 torch.distributed.GradBucket。


 Parameters


 ( *自己



 :
 


[火炬._C._distributed_c10d.GradBucket](#torch.distributed.GradBucket“火炬._C._distributed_c10d.GradBucket”)*)


 →
 


 List
 


 [ [torch.Tensor](tensors.html#torch.Tensor "torch.Tensor")


 ]
 


[¶](#torch.distributed.GradBucket.parameters"此定义的永久链接")


 退货


 `torch.Tensor` 列表。列表中的每个tensor对应一个模型参数。


## 默认通信挂钩 [¶](#default-communication-hooks "此标题的永久链接")


 默认通信钩子是简单的**无状态**钩子，因此 `register_comm_hook` 中的输入状态要么是进程组，要么是 `None` 。输入 `bucket` 是一个 [`torch.distributed.GradBucket`]( #torch.distributed.GradBucket“torch.distributed.GradBucket”)对象。


 torch.distributed.algorithms.ddp_comm_hooks.default_hooks。


 allreduce_hook


 ( *process_group
* , *bucket
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/default_hooks.html#allreduce_hook)[¶](#torch.distributed.algorithms.ddp_comm_hooks.default_hooks.allreduce_hook "此定义的永久链接")


 此 DDP 通信挂钩仅使用“GradBucket”tensor调用“allreduce”。一旦梯度tensor在所有工作线程中聚合，它的“then”回调就会取平均值并返回结果。如果用户注册了此钩子，则 DDP 结果预计与未注册钩子的情况相同。因此，这不会改变 DDP 的行为，用户可以将此用作参考或修改此钩子以记录有用信息或任何其他信息目的，同时不影响 DDP 行为。


 例子：：




```
>>> ddp_model.register_comm_hook(process_group, allreduce_hook)

```


 Return type


[*Future*](futures.html#torch.futures.Future "torch.jit.Future") [ [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ]


 torch.distributed.algorithms.ddp_comm_hooks.default_hooks。


 fp16_compress_hook


 ( *process_group
* , *bucket
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/default_hooks.html#fp16_compress_hook)[¶](#torch.distributed.algorithms.ddp_comm_hooks.default_hooks.fp16_compress_hook "此定义的永久链接")


 这个 DDP 通信钩子实现了一种简单的梯度压缩方法，将“GradBucket”tensor转换为半精度浮点格式(“torch.float16”)，然后将其除以进程组大小。这一切都减少了那些“float16”梯度tensor。一旦压缩的梯度tensor全部归约，链式回调“decompress”会将其转换回输入数据类型(例如“float32”)。


 例子：：




```
>>> ddp_model.register_comm_hook(process_group, fp16_compress_hook)

```


 Return type


[*Future*](futures.html#torch.futures.Future "torch.jit.Future") [ [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ]


 torch.distributed.algorithms.ddp_comm_hooks.default_hooks。


 bf16_compress_hook


 ( *process_group
* , *bucket
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/default_hooks.html#bf16_compress_hook)[¶](#torch.distributed.algorithms.ddp_comm_hooks.default_hooks.bf16_compress_hook "此定义的永久链接")


 警告：此 API 是实验性的，需要 NCCL 版本高于 2.9.6。


 这个 DDP 通信钩子实现了一种简单的梯度压缩方法，将 `GradBucket` tensor转换为半精度 [Brain 浮点格式](https://en.wikipedia.org/wiki/Bfloat16_floating-point_format) (`torch.bfloat16`) 和然后除以进程组大小。这一切都会减少那些“bfloat16”梯度tensor。一旦压缩的梯度tensor全部归约，链式回调“decompress”会将其转换回输入数据类型(例如“float32”)。


 例子：：




```
>>> ddp_model.register_comm_hook(process_group, bf16_compress_hook)

```


 Return type


[*Future*](futures.html#torch.futures.Future "torch.jit.Future") [ [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ]


 此外，还提供了一个通信钩子包装器来支持 [`fp16_compress_hook()`](#torch.distributed.algorithms.ddp_comm_hooks.default_hooks.fp16_compress_hook "torch.distributed.algorithms.ddp_comm_hooks.default_hooks.fp16_compress_hook") 或 [ `bf16_compress_hook()`](#torch.distributed.algorithms.ddp_comm_hooks.default_hooks.bf16_compress_hook "torch.distributed.algorithms.ddp_comm_hooks.default_hooks.bf16_compress_hook") 作为包装器，可以与其他通信挂钩组合。


 torch.distributed.algorithms.ddp_comm_hooks.default_hooks。


 fp16_compress_wrapper


 ( *hook
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/default_hooks.html#fp16_compress_wrapper)[¶](#torch.distributed.algorithms.ddp_comm_hooks.default_hooks.fp16_compress_wrapper "此定义的永久链接")


 该包装器将给定 DDP 通信钩子的输入梯度tensor转换为半精度浮点格式(“torch.float16”)，并将给定钩子的结果tensor转换回输入数据类型，例如“float32”。


 因此， `fp16_compress_hook` 相当于 `fp16_compress_wrapper(allreduce_hook)` 。


 例子：：




```
>>> state = PowerSGDState(process_group=process_group, matrix_approximation_rank=1, start_powerSGD_iter=10)
>>> ddp_model.register_comm_hook(state, fp16_compress_wrapper(powerSGD_hook))

```


 Return type


[*Callable*](https://docs.python.org/3/library/typing.html#typing.Callable "(在 Python v3.12 中)") [[ [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)") , [*GradBucket*](#torch.distributed.GradBucket "torch._C._distributed_c10d.GradBucket") ] , [*Future*](futures.html#torch.futures.Future "torch.jit.Future") [ [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ]]


 torch.distributed.algorithms.ddp_comm_hooks.default_hooks。


 bf16_compress_wrapper


 ( *hook
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/default_hooks.html#bf16_compress_wrapper)[¶](#torch.distributed.algorithms.ddp_comm_hooks.default_hooks.bf16_compress_wrapper "此定义的永久链接")


 警告：此 API 是实验性的，需要 NCCL 版本高于 2.9.6。


 该包装器将给定 DDP 通信挂钩的输入梯度tensor转换为半精度 Brain 浮点格式 <https://en.wikipedia.org/wiki/Bfloat16_floating-point_format
> `_ (``torch. bfloat16` )，并将给定钩子的结果tensor转换回输入数据类型，例如 `float32` 。


 因此， `bf16_compress_hook` 相当于 `bf16_compress_wrapper(allreduce_hook)` 。


 例子：：




```
>>> state = PowerSGDState(process_group=process_group, matrix_approximation_rank=1, start_powerSGD_iter=10)
>>> ddp_model.register_comm_hook(state, bf16_compress_wrapper(powerSGD_hook))

```


 Return type


[*Callable*](https://docs.python.org/3/library/typing.html#typing.Callable "(在 Python v3.12 中)") [[ [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)") , [*GradBucket*](#torch.distributed.GradBucket "torch._C._distributed_c10d.GradBucket") ] , [*Future*](futures.html#torch.futures.Future "torch.jit.Future") [ [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ]]


## PowerSGD 通信挂钩 [¶](#powersgd-communication-hook "此标题的永久链接")


 PowerSGD([Vogels et al., NeurIPS 2019](https://arxiv.org/abs/1905.13727))是一种梯度压缩算法，可以提供非常高的压缩率并加速带宽限制的分布式训练。该算法需要维护一些超参数和内部状态。因此，PowerSGD通信钩子是一个**有状态**钩子，用户需要提供如下定义的状态对象。


### PowerSGD 状态 [¶](#powersgd-state "此标题的永久链接")


*班级*


 torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook。


 电源SGD状态


 ( *process_group
* , *matrix_approximation_rank



 =
 


 1
* , *启动_powerSGD_iter



 =
 


 1000
* , *最小压缩率



 =
 


 2
* , *使用_错误_反馈



 =
 


 真*，*温暖_start



 =
 


 True
* , *正交化_epsilon



 =
 


 0
* , *随机_种子



 =
 


 0
* , *压缩_统计_记录_频率



 =
 


 10000
* , *batch_tensors_with_same_shape



 =
 


 False
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/powerSGD_hook.html#PowerSGDState)[¶](#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.PowerSGDState "此定义的永久链接")


 存储算法的超参数和训练期间所有梯度的内部状态。特别是，“matrix_approximation_rank”和“start_powerSGD_iter”是用户应调整的主要超参数。为了性能，我们建议保持二进制超参数 `use_error_feedback` 和 `warm_start` 开启。


1. `matrix_approximation_rank` 控制压缩的低秩tensor的大小，它决定了压缩率。等级越低，压缩越强。



> 
> 
> 
> 1.1.如果
> `matrix_approximation_rank`
> 太低，完整的模型质量将需要更多的训练步骤才能达到或永远不会达到并产生精度损失。
> 
> 
> 
> 
> 1.2。 
> `matrix_approximation_rank`
> 的增加会大幅增加压缩的计算成本，并且超过某个 
> `matrix_approximation_rank`
> 阈值后精度可能不会进一步提高。
> 
> 
> 
> >


 要调整 `matrix_approximation_rank` ，我们建议从 1 开始并增加 2 倍(如指数网格搜索，1,2,4，...)，直到达到满意的精度。通常仅使用较小的值 1-4。对于某些NLP任务(如原论文附录D所示)，该值已增加至32。


2. `start_powerSGD_iter` 将 PowerSGD 压缩推迟到步骤 `start_powerSGD_iter` ，并且普通 allreduce 在步骤 `start_powerSGD_iter` 之前运行。这种**vanilla allreduce 
+ PowerSGD**的混合方案可以有效提高精度，即使使用相对较小的`matrix_approximation_rank`。这是因为，训练阶段的开始通常对不准确的梯度非常敏感，过早压缩梯度可能会使训练很快采取次优轨迹，从而对精度造成不可恢复的影响。


 要调整 `start_powerSGD_iter` ，我们建议从总训练步骤的 10% 开始，然后增加它直到达到满意的精度。如果训练中有预热阶段，`start_powerSGD_iter`通常应不少于预热步骤数。


3. `min_compression_rate` 是压缩某一层时所需的最小压缩率。由于压缩会产生计算开销，只有在能够充分节省带宽的情况下才值得压缩tensor，其中 `(num_rows 
+ num_cols) * matrix_approximation_rank * min_compression_rate < 行数 * 列数` 。如果不能满足指定的压缩率阈值，则tensor将直接allreduce，而不进行压缩。


 PowerSGD 压缩开始后，每次“compression_stats_logging_Frequency”迭代都会记录压缩统计信息。


4. `orthogonalization_epsilon` 可以是一个非常小的值(例如，1e-8)，在正交化步骤中添加到每个归一化矩阵列，以防止在任何列全为 0 时出现除零错误。如果这种情况已经可以避免(例如，通过批量归一化)，则建议将 epsilon 设置为 0 以确保准确度。5。 `batch_tensors_with_same_shape` 控制是否在批量操作中压缩和解压缩具有相同形状的tensor，以实现更高的并行度。请注意，您还应该增加桶大小(即 DDP 构造函数中的 `bucket_cap_mb` arg)以使更多相同形状的tensor出现在同一个桶中，但这可能会减少计算和通信之间的重叠，并增加由于堆叠相同形状的tensor而导致的内存占用。如果压缩/解压缩计算是瓶颈，则设置为“True”。


!!! warning "警告"

     如果启用了错误反馈或预热，则 DDP 中允许的 `start_powerSGD_iter` 最小值为 2。这是因为在 DDP 中存在另一个内部优化，该优化会在迭代 1 时重建存储桶，这可能会与任何在重建过程之前记忆的tensor。


### PowerSGD 挂钩 [¶](#powersgd-hooks "此标题的永久链接")


!!! warning "警告"

     PowerSGD 通常需要与模型梯度大小相同的额外内存来实现误差反馈，这可以补偿有偏差的压缩通信并提高准确性。


!!! warning "警告"

     PowerSGD hooks可能与[Apex自动混合精度包](https://github.com/NVIDIA/apex)冲突。请使用PyTorch[原生自动混合精度包](https://pytorch.org/docs/stable/amp.html) 代替。


 torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook。


 powerSGD_hook


 ( *state
* , *bucket
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/powerSGD_hook.html#powerSGD_hook)[¶](#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.powerSGD_hook "永久链接这个定义")


 该 DDP 通信钩子实现了[论文](https://arxiv.org/abs/1905.13727)中描述的 PowerSGD 梯度压缩算法。一旦梯度tensor在所有工作线程中聚合，该钩子就会应用压缩，如下所示：


1. 将输入的展平一维梯度tensor视为每参数tensor的列表，并将所有tensor分为两组：



> 
> 
> 
> 1.1 在allreduce之前应该压缩的tensor，因为压缩可以节省足够的带宽。
> 
> 
> 
> 
> 1.2 其余tensor将直接allreduce而不压缩，包括所有向量tensor(对于偏差).
> 
> 
> 
> >2.处理未压缩的tensor：



> 
> 
> 
> 2.1.为那些未压缩的tensor分配连续的内存，并将所有未压缩的tensor作为一个批次进行allreduce，不进行压缩；
> 
> 
> 
> 
> 2.2.将单个未压缩tensor从连续内存复制回输入tensor。
> 
> 
> 
> >3.处理应通过 PowerSGD 压缩进行压缩的tensor：



> 
> 
> 
> 3.1.对于每个tensor M，创建两个低秩tensor P 和 Q 用于分解 M，
> 使得 M = PQ^T，其中 Q 从标准正态分布初始化并正交化；
> 
> 
> 
> 
> 3.2。计算Ps中的每个P，等于MQ；
> 
> 
> 
> 
> 3.3。批量减少Ps；
> 
> 
> 
> 
> 3.4.正交化 Ps 中的每个 P；
> 
> 
> 
> 
> 3.5。计算Qs中的每个Q，约等于M^TP；
> 
> 
> 
> 
> 3.6。批量减少Qs；
> 
> 
> 
> 
> 3.7。计算所有压缩tensor中的每个M，大约等于PQ^T。
> 
> 
> 
> >


 请注意，此通信挂钩对第一个 `state.start_powerSGD_iter` 迭代强制执行普通 allreduce。这不仅使用户能够更好地控制加速和准确性之间的权衡，而且还有助于抽象出内部优化的一些复杂性。 DDP 供未来通信钩子开发人员使用。


 参数 
* **state** ( [*PowerSGDState*](#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.PowerSGDState "torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.PowerSGDState") ) – 用于配置压缩率和支持错误反馈、热启动等。调整压缩配置，主要需要调整 `matrix_approximation_rank` 、 `start_powerSGD_iter` 和 `min_compression_rate` 。
* **bucket** ( *dist.GradBucket
* ) – 存储一维扁平梯度tensor的存储桶，该tensor对多个每个变量tensor进行批处理。请注意，由于 DDP comm hook 仅支持单进程单设备模式，因此该存储桶中仅存储一个tensor。


 退货


 通信的未来处理程序，它更新适当的梯度。


 Return type


[*Future*](futures.html#torch.futures.Future "torch.jit.Future") [ [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ]


 例子：：




```
>>> state = PowerSGDState(process_group=process_group, matrix_approximation_rank=1,
 start_powerSGD_iter=10, min_compression_rate=0.5)
>>> ddp_model.register_comm_hook(state, powerSGD_hook)

```


 torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook。


 批处理_powerSGD_hook


 ( *state
* , *bucket
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/powerSGD_hook.html#batched_powerSGD_hook)[¶](#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.batched_powerSGD_hook "永久链接这个定义")


 这个DDP通信钩子实现了[论文](https://arxiv.org/abs/1905.13727)中描述的简化的PowerSGD梯度压缩算法。这个变体不会逐层压缩梯度，而是压缩批处理的扁平化输入tensor因此，它比 [`powerSGD_hook()`](#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.powerSGD_hook "torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.powerSGD_hook") **快** ，但通常会导致**低得多的精度**，除非 `matrix_approximation_rank` 为 1。


!!! warning "警告"

     这里增加 `matrix_approximation_rank` 不一定会提高精度，因为在没有列/行对齐的情况下对每个参数tensor进行批处理可能会破坏低秩结构。因此，用户应始终考虑 [`powerSGD_hook()`] (#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.powerSGD_hook "torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.powerSGD_hook") 首先，并且只有当 `matrix_approximation_rank` 可以达到满意的精度时才考虑此变体1.


 一旦梯度tensor在所有工作线程中聚合，该钩子就会应用压缩，如下所示：


1. 将输入的展平一维梯度tensor视为具有 0 填充的方形tensor M；2.创建两个低秩tensor P 和 Q 用于分解 M，使得 M = PQ^T，其中 Q 从标准正态分布初始化并正交化；3．计算P，它等于MQ；4．全部减少P；5．正交化P；6。计算 Q，约等于 M^TP；7。全部减少Q；8．计算 M，约等于 PQ^T.9。将输入tensor截断为原始长度。


 请注意，此通信挂钩对第一个 `state.start_powerSGD_iter` 迭代强制执行普通 allreduce。这不仅使用户能够更好地控制加速和准确性之间的权衡，而且还有助于抽象出内部优化的一些复杂性。 DDP 供未来通信钩子开发人员使用。


 参数 
* **state** ( [*PowerSGDState*](#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.PowerSGDState "torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.PowerSGDState") ) – 用于配置压缩率和支持错误反馈、热启动等。调整压缩配置，主要需要调整 `matrix_approximation_rank` 和 `start_powerSGD_iter`.
* **bucket** ( *dist.GradBucket
* ) – Bucket存储一个一维展平梯度tensor，该tensor对多个每个变量tensor进行批处理。请注意，由于 DDP comm hook 仅支持单进程单设备模式，因此该存储桶中仅存储一个tensor。


 退货


 通信的未来处理程序，它更新适当的梯度。


 Return type


[*Future*](futures.html#torch.futures.Future "torch.jit.Future") [ [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ]


 例子：：




```
>>> state = PowerSGDState(process_group=process_group, matrix_approximation_rank=1)
>>> ddp_model.register_comm_hook(state, batched_powerSGD_hook)

```


## 调试通信挂钩 [¶](#debugging-communication-hooks "此标题的固定链接")


 顾名思义，调试通信挂钩**仅**用于调试和性能优化目的。


!!! warning "警告"

     调试通信钩子不一定会输出正确的结果。


 torch.distributed.algorithms.ddp_comm_hooks.debugging_hooks。


 noop_hook


 ( *_
* , *bucket
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/debugging_hooks.html#noop_hook)[¶](#torch.distributed.algorithms.ddp_comm_hooks.debugging_hooks.noop_hook "永久链接到这个定义")


 这个 DDP 通信钩子返回一个包装输入的 future，因此它是一个不会产生任何通信开销的 noop。


 这个钩子应该**只**用于allreduce优化的余量分析，而不是正常的梯度同步。例如，如果注册这个钩子后只能观察到不到10%的训练时间加速，这通常意味着：对于这种情况，allreduce 不是性能瓶颈。如果 GPU 跟踪无法轻松检索，或者跟踪分析因 allreduce 和计算之间的重叠或跨等级不同步等因素而变得复杂，则此类检测可能特别有用。


 例子：：




```
>>> ddp_model.register_comm_hook(None, noop_hook)

```


 Return type


[*Future*](futures.html#torch.futures.Future "torch.jit.Future") [ [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ]


## 通信挂钩的检查点 [¶](#checkpointing-of-communication-hooks "此标题的永久链接")


 有状态通信钩子可以保存为模型检查点的一部分，以便训练器重新启动。要使钩子可序列化，应定义`__setstate__`和`__getstate__`。


!!! warning "警告"

    `__getstate__` 应该从返回的字典中排除不可序列化的属性。


!!! warning "警告"

    `__setstate__` 应正确初始化不可序列化的属性，从提供的 `state` 中排除。


[`PowerSGDState`](#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.PowerSGDState "torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.PowerSGDState") 有 `__setstate__` 和 `__getstate\ __`已实现，可以作为参考。


*班级*


 torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook。


 PowerSGDState [[来源]](_modules/torch/distributed/algorithms/ddp_comm_hooks/powerSGD_hook.html#PowerSGDState)


 __getstate__


 ( ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/powerSGD_hook.html#PowerSGDState.__getstate__)[¶](#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.PowerSGDState.__getstate__ "此定义的永久链接" )


 返回一个“Dict[str, Any]”，它将被腌制并保存。 `process_group` 不可序列化，并且从返回状态中排除。


 __设置状态__


 ( *state
* ) [[source]](_modules/torch/distributed/algorithms/ddp_comm_hooks/powerSGD_hook.html#PowerSGDState.__setstate__)[¶](#torch.distributed.algorithms.ddp_comm_hooks.powerSGD_hook.PowerSGDState.__setstate__ "永久链接这个定义")


 获取提供的“state”并检索“PowerSGDState”。 `process_group` 设置为默认值。


 以下是保存和重新加载 PowerSGD 状态和挂钩的简单端到端示例。


```
import os
import sys
import tempfile
import torch
import torch.distributed as dist
import torch.nn as nn
import torch.optim as optim
import torch.multiprocessing as mp

from torch.nn.parallel import DistributedDataParallel
from torch.distributed.algorithms.ddp_comm_hooks import powerSGD_hook as powerSGD

class SimpleModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(24,24)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(24,12)

    def forward(self, x):
        return self.fc2(self.relu(self.fc1(x)))

def setup(rank, world_size):
    os.environ['MASTER_ADDR'] = 'localhost'
    os.environ['MASTER_PORT'] = '12355'

    # initialize the process group
    dist.init_process_group("nccl", rank=rank, world_size=world_size)

def cleanup():
    dist.destroy_process_group()

def run_demo(demo_fn, world_size):
    mp.spawn(
        demo_fn,
        args=(world_size,),
        nprocs=world_size,
        join=True)

def demo_serialization(rank, world_size):
    setup(rank, world_size)

    CHECKPOINT = tempfile.gettempdir() + "/checkpoint.pt"

    model = SimpleModel().to(rank)
    ddp_model = DistributedDataParallel(model, device_ids=[rank])

    powersgd_hook = powerSGD.powerSGD_hook
    powersgd_state = powerSGD.PowerSGDState(process_group=None)

    optimizer = optim.SGD(ddp_model.parameters(), lr=0.001)
    ddp_model.register_comm_hook(powersgd_state, powersgd_hook)

    state = {
        'state_dict': ddp_model.state_dict(),
        'comm_hook': powersgd_hook,
        'comm_hook_state': powersgd_state}

    if rank == 0:
        torch.save(state, CHECKPOINT)

    dist.barrier()
    map_location = {'cuda:%d' % 0: 'cuda:%d' % rank}
    checkpoint = torch.load(CHECKPOINT, map_location=map_location)

    new_ddp_model = DistributedDataParallel(SimpleModel().to(rank), device_ids=[rank])
    new_ddp_model.load_state_dict(checkpoint['state_dict'])
    powersgd_hook = checkpoint['comm_hook']
    powersgd_state = checkpoint['comm_hook_state']

    new_ddp_model.register_comm_hook(powersgd_state, powersgd_hook)

    if rank == 0:
        os.remove(CHECKPOINT)

    cleanup()

if __name__ == "__main__":
    n_gpus = torch.cuda.device_count()
    assert n_gpus >= 2, f"Requires at least 2 GPUs to run, but got {n_gpus}"
    world_size = n_gpus
    run_demo(demo_serialization, world_size)

```


## 致谢 [¶](#acknowledgements "此标题的永久链接")


 非常感谢 PowerSGD 论文作者 **Thijs Vogels** 对 PowerSGD 通信钩子的代码审查，以及[比较实验](https://observablehq.com/@tvogels/powersgd-benchmark)，这表明性能PowerSGD 通信钩子的实现与原始[论文](https://arxiv.org/abs/1905.13727)中的实现相同。