# tensor并行 
- torch.distributed.tensor.parallel [¶](#tensor-parallelism-torch-distributed-tensor-parallel "永久链接到此标题")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/distributed.tensor.parallel>
>
> 原始地址：<https://pytorch.org/docs/stable/distributed.tensor.parallel.html>


 Tensor Parallelism(TP) 建立在 PyTorch DistributedTensor ( [DTensor](https://github.com/pytorch/pytorch/blob/main/torch/distributed/_tensor/README.md) 之上，并提供多种并行风格：Rowwise、Colwise 和 Pairwise 并行。


!!! warning "警告"

     tensor并行 API 是实验性的，可能会发生变化。


 使用tensor并行化并行化“nn.Module”的入口点是：


 torch.distributed.tensor.parallel。


 并行化模块


 (*模块*，*设备_mesh*，*并行化_plan*，*tp_mesh_dim



 =
 


 0
* ) [[source]](_modules/torch/distributed/tensor/parallel/api.html#parallelize_module)[¶](#torch.distributed.tensor.parallel.parallelize_module "此定义的永久链接")


 在 PyTorch 中应用tensor并行性 (TP) 的 API。我们基于并行化计划并行化模块或子模块。 Parallelize_plan 包含 `ParallelStyle` ，它指示用户希望模块或子模块如何并行化。


 用户还可以为每个模块指定不同的并行样式完全限定名称(FQN)。API 通过接受 n 维设备 _mes 来原生支持 2D 并行性，用户只需要指定我们执行tensor并行性的维度。


 参数 
* **module** ( `nn.Module` ) – 要并行化的模块。
* **device_mesh** ( `DeviceMesh` ) – 描述 DTensor 设备的网格拓扑的对象。
* **parallelize\ _plan** (Union[ `ParallelStyle` , Dict[str, `ParallelStyle` ]]) – 用于并行化模块的计划。它可以是一个 `ParallelStyle` 对象，其中包含我们如何为tensor并行准备输入/输出，也可以是模块 FQN 及其相应的 `ParallelStyle` 对象的指令。
* **tp_mesh_dim** ( [*int
* ](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 我们执行tensor并行的 `device_mesh` 的维度。


 退货


 并行化的“nn.Module”对象。


 Return type


[*模块*](generated/torch.nn.Module.html#torch.nn.Module“torch.nn.modules.module.Module”)


 例子：：




```
>>> from torch.distributed.tensor.parallel import parallelize_module, PairwiseParallel
>>>
>>> # Define the module.
>>> m = Model(...)
>>> m = parallelize_module(m, PairwiseParallel())
>>>

```


!!! warning "警告"

    “PairwiseParallel”目前有一些限制。如果您需要更细的粒度，则需要传入模块 FQN 和并行样式的字典。


 tensor并行支持以下并行样式：


*班级*


 torch.distributed.tensor.parallel.style。


 行并行


 ( *_prepare_input=<function make_input_shard_1d_last_dim>
* , *_prepare_output=<function make_output_tensor>
* ) [[source]](_modules/torch/distributed /tensor/parallel/style.html#RowwiseParallel)[¶](#torch.distributed.tensor.parallel.style.RowwiseParallel "此定义的永久链接")


 对模块的行进行分区。我们假设输入是分片的`DTensor`，输出是 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 。


*班级*


 torch.distributed.tensor.parallel.style。


 Colwise并行


 ( *_prepare_input=<函数 make_input_replicate_1d>
* , *_prepare_output=<函数 make_sharded_output_tensor>
* ) [[source]](_modules/torch/distributed/tensor /parallel/style.html#ColwiseParallel)[¶](#torch.distributed.tensor.parallel.style.ColwiseParallel "此定义的永久链接")


 对tensor或模块的列进行分区。我们假设输入是复制的“DTensor”，输出是分片的 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 。


*班级*


 torch.distributed.tensor.parallel.style。


 成对并联


 ( *_准备_输入



 =
 


 无
* , *_prepare_output



 =
 


 无
* ) [[source]](_modules/torch/distributed/tensor/parallel/style.html#PairwiseParallel)[¶](#torch.distributed.tensor.parallel.style.PairwiseParallel "此定义的永久链接")


 PairwiseParallel 将 colwise 和 rowwise 样式连接为固定对，就像 Megatron-LM ( <https://arxiv.org/abs/1909.08053
> ) 正在做的那样。我们假设输入和输出都需要是复制 DTensors。


!!! warning "警告"

     PairwiseParallel 目前还不能很好地支持 `nn.MultiheadAttention` 、 `nn.Transformer` 。一种解决方法是将“ColwiseParallel”和“RowwiseParallel”应用于变压器的组件。目前我们建议仅对偶数层 MLP 使用“PairwiseParallel”。


!!! warning "警告"

     序列并行性仍处于实验阶段，尚未进行评估。


*班级*


 torch.distributed.tensor.parallel.style。


 SequenceParallel [[source]](_modules/torch/distributed/tensor/parallel/style.html#SequenceParallel)[¶](#torch.distributed.tensor.parallel.style.SequenceParallel"此定义的永久链接")


 SequenceParallel 将 colwise 和 rowwise 样式连接为固定对，并与序列并行连接在一起，就像 Megatron-LM Sequence parallel( <https://arxiv.org/pdf/2205.05198.pdf
> ) 所做的那样。我们假设输入和输出都需要分片D tensor。


!!! warning "警告"

     SequenceParallel 目前还不能很好地支持 `nn.MultiheadAttention` 、 `nn.Transformer` 。一种解决方法是将“ColwiseParallel”和“RowwiseParallel”应用于变压器的组件。目前我们建议仅对偶数层 MLP 使用“SequenceParallel”。


 由于 Tensor Parallelism 是建立在 DTensor 之上的，因此我们需要使用 DTensor 指定模块的输入和输出位置，以便它可以按预期与前后模块进行交互。以下是用于输入/输出准备的函数：


 torch.distributed.tensor.parallel.style。


 制作_输入_复制_1d


 ( *输入
* , *设备_mesh



 =
 


 无
* ) [[source]](_modules/torch/distributed/tensor/parallel/style.html#make_input_replicate_1d)[¶](#torch.distributed.tensor.parallel.style.make_input_replicate_1d "此定义的永久链接")


 通过一维设备网格复制输入tensor。该函数将在ParallelStyle 中使用。


 参数 
* **input** (Union[ [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") , `DTensor` ]) – 该输入tensor将在一维`上复制DeviceMesh`.
* **device_mesh** ( `DeviceMesh` ，可选) – 将复制 `input` 的一维设备网格。如果没有传递 `DeviceMesh` 并且 `input` 是一个 `DTensor` ，将使用`input.device_mesh`。如果`DeviceMesh`不是一维的，则会抛出异常。默认：`None`


 退货


 通过“device_mesh”复制的“DTensor”。


 Return type


*Dtensor*


 torch.distributed.tensor.parallel.style。


 make_input_reshard_replicate


 ( *input
* , *device_mesh
* ) [[source]](_modules/torch/distributed/tensor/parallel/style.html#make_input_reshard_replicate)[¶](#torch.distributed.tensor.parallel.style.make_input_reshard_replicate "此定义的永久链接")


 从不同等级的tensor构建分片 DTensor，然后转换为复制 DTensor。


 参数 
* **input** ( [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") ) – 每个等级上的输入tensor，由维度“0”上的全局 DTensorsharded 组成1-D `DeviceMesh`，然后将分片的 DTensor 转换为复制 DTensor。
* **device_mesh** ( `DeviceMesh` ，可选) – 将分片 `input` 的 1-D 设备网格。如果 ` DeviceMesh` 不是一维的，会抛出异常。默认值：`None`


 退货


 在“device_mesh”上的维度“0”上分片的“DTensor”


 然后转换为复制。


 Return type


*Dtensor*


 torch.distributed.tensor.parallel.style。


 制作_input_shard_1d


 ( *输入
* , *设备_mesh



 =
 


 无*，*暗淡



 =
 


 0
* ) [[source]](_modules/torch/distributed/tensor/parallel/style.html#make_input_shard_1d)[¶](#torch.distributed.tensor.parallel.style.make_input_shard_1d "此定义的永久链接")


 一维设备网格上“dim”上的分片输入tensor。该函数将在ParallelStyle 中使用。


 参数 
* **input** (Union[ [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") , `DTensor` ]) – 单个tensor将在维度 `dim` 上进行分片1-D `DeviceMesh`.
* **device_mesh** ( `DeviceMesh` ，可选) – 一维设备网格，其中 `input` 将被分片。如果没有传递 `DeviceMesh` 并且 `input` 是`DTensor` , input.device_mesh 将被使用。如果 `DeviceMesh` 不是一维的，则会抛出异常。默认: `None`
* **dim** ( [*int*](https:///docs.python.org/3/library/functions.html#int "(in Python v3.12)")*,
* *可选
* ) – `input` tensor的分片维度。默认: 0


 退货


 在“device_mesh”上的“dim”维度上分片的“DTensor”。


 Return type


*Dtensor*


 torch.distributed.tensor.parallel.style。


 make_input_shard_1d_last_dim


 ( *输入
* , *设备_mesh



 =
 


 无
* ) [[source]](_modules/torch/distributed/tensor/parallel/style.html#make_input_shard_1d_last_dim)[¶](#torch.distributed.tensor.parallel.style.make_input_shard_1d_last_dim "此定义的永久链接")


 `make_input_shard_1d` 的包装函数，其中 `dim` = -1。


 参数 
* **input** (Union[ [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") , `DTensor` ]) – 这个单个tensor将在 1 的最后一个维度上进行分片-D `DeviceMesh`.
* **device_mesh** ( `DeviceMesh` ，可选) – 一维设备网格，其中 `input` 将被分片。如果没有传递 `DeviceMesh` 并且 `input` 是一个 ` DTensor` , input.device_mesh 将被使用。如果 `DeviceMesh` 不是一维的，将会抛出异常。默认: `None`


 退货


 在“device_mesh”的最后一个维度上分片的“DTensor”。


 Return type


*Dtensor*


 torch.distributed.tensor.parallel.style。


 make_output_replicate_1d


 ( *输出
* , *设备_mesh



 =
 


 无
* ) [[source]](_modules/torch/distributed/tensor/parallel/style.html#make_output_replicate_1d)[¶](#torch.distributed.tensor.parallel.style.make_output_replicate_1d "此定义的永久链接")


 将输出 DTensor 转换为复制的 DTensor。这将在 ParallelStyle 中使用。


 参数 
* **output** ( `DTensor` ) – 要转换的模块的输出。
* **device_mesh** ( `DeviceMesh` ，可选) – 复制输出所需的对象，并且它需要是一个 1D ` device_mesh`，如果传入非一维的`device_mesh`，我们将抛出异常。如果没有传入`device_mesh`，我们将重用输出中的那个。默认：`None`


 退货


 复制了“DTensor”对象。


 Return type


*Dtensor*


 torch.distributed.tensor.parallel.style。


 制作_output_reshard_tensor


 ( *输出
* , *设备_mesh



 =
 


 无
* ) [[source]](_modules/torch/distributed/tensor/parallel/style.html#make_output_reshard_tensor)[¶](#torch.distributed.tensor.parallel.style.make_output_reshard_tensor "此定义的永久链接")


 将输出 DTensor 转换为分片 DTensor 并返回局部tensor。


 参数 
* **output** ( `DTensor` ) – 要转换的模块的输出。
* **device_mesh** ( `DeviceMesh` ，可选) – 分片输出所需的对象，并且它需要是一个 1D ` device_mesh`，如果传入非一维的`device_mesh`，我们将抛出异常。如果没有传入`device_mesh`，我们将重用输出中的那个。默认：`None`


 退货


 从输出 DTensor 转换而来的 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 对象。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 torch.distributed.tensor.parallel.style。


 make_output_shard_1d


 ( *输出
* , *设备_mesh



 =
 


 无*，*暗淡



 =
 


 0
* ) [[source]](_modules/torch/distributed/tensor/parallel/style.html#make_output_shard_1d)[¶](#torch.distributed.tensor.parallel.style.make_output_shard_1d "此定义的永久链接")


 将输出 DTensor 转换为分片 DTensor。这将在 ParallelStyle 中使用。


 参数 
* **output** ( `DTensor` ) – 要转换的模块的输出。
* **device_mesh** ( `DeviceMesh` ，可选) – 分片输出所需的对象，并且它需要是一个 1D ` device_mesh`，如果传入非一维的`device_mesh`，我们将抛出异常。如果没有传入`device_mesh`，我们将重用输出中的那个。默认：`None`
* ** dim** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 对输出进行分片 dim。默认值：0


 退货


 在给定的维度上分片的“DTensor”对象。


 Return type


*Dtensor*


 torch.distributed.tensor.parallel.style。


 制作_输出_tensor


 ( *输出
* , *设备_mesh



 =
 


 无
* ) [[source]](_modules/torch/distributed/tensor/parallel/style.html#make_output_tensor)[¶](#torch.distributed.tensor.parallel.style.make_output_tensor "此定义的永久链接")


 首先将输出 DTensor 转换为复制的 DTensor，然后将其转换为 Tensor。


 参数 
* **output** ( `DTensor` ) – 要转换的模块的输出。
* **device_mesh** ( `DeviceMesh` , 可选) – 复制输出所需的对象，并且必须是 1D `device_mesh`，如果传入非一维 `device_mesh`，我们将抛出异常。如果没有传入 `device_mesh`，我们将重用输出中的那个。默认：`None`


 退货


 从输出 DTensor 转换而来的 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 对象。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 目前，存在一些限制，使得“MultiheadAttention”模块很难开箱即用地用于tensor并行，因此我们建议用户对每个参数尝试“ColwiseParallel”和“RowwiseParallel”。由于我们正在“MultiheadAttention”模块的 head dim 上进行并行化，因此现在可能需要进行一些代码更改。


 我们还支持2D并行，将tensor并行与数据并行结合起来。要与`FullyShardedDataParallel`集成，用户只需要显式调用以下API：


 torch.distributed.tensor.parallel.fsdp。


 启用_2d_with_fsdp


 ( ) [[source]](_modules/torch/distributed/tensor/parallel/fsdp.html#enable_2d_with_fsdp)[¶](#torch.distributed.tensor.parallel.fsdp.enable_2d_with_fsdp "此定义的永久链接")


 API 注册tensor并行性 (TP) 所需的扩展，以便与 FullShardedDataParallel (FSDP) 配合使用。我们首先基于并行化计划并行化一个模块或子模块内的参数，并让 FSDPreshard 分布式参数的局部tensor，本质上是一个 DTensor。


 退货


 bool 表示扩展注册是否成功。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 要与“DistributedDataParallel”集成，用户只需显式调用以下API：


 torch.distributed.tensor.parallel.ddp。


 pre_dp_module_transform


 ( *module
* ) [[source]](_modules/torch/distributed/tensor/parallel/ddp.html#pre_dp_module_transform)[¶](#torch.distributed.tensor.parallel.ddp.pre_dp_module_transform "此定义的永久链接")


 使用 DDP 时，在 PyTorch 中启用tensor并行性 (TP) 和数据并行性 (DP) 之间的可组合性。在使用数据并行 API 包装之前，我们需要将 DTensor 参数转换为本地tensor。然后我们注册两个钩子，一个用于将本地tensor转换回 DTensorpreforward，另一个用于在 Forward 之后将 DTensor 转换回tensor。通过这种方式积分，我们避免了 DDP 对 DTensor 参数的任何特殊处理，并将 DTensor 的梯度传播回 DP，例如DDP 的梯度桶。


 目前，此 API 仅适用于“DistributedDataParallel”。稍后它将支持其他 DP 方法，例如 FSDP。


 Parameters


**module** ( `nn.Module` ) – 已应用 TP 的模块。


 例子：：




```
>>> from torch.distributed.tensor.parallel import parallelize_module, PairwiseParallel
>>> from torch.nn.parallel import DistributedDataParallel as DDP
>>> from torch.distributed.tensor.parallel.ddp import pre_dp_module_transform
>>>
>>> # Define the module.
>>> m = module(...)
>>> parallelize_module(m, PairwiseParallel())
>>> m = pre_dp_module_transform(m)
>>> m = DDP(m)
>>>

```