# 分布式检查点 
- torch.distributed.checkpoint [¶](#distributed-checkpoint-torch-distributed-checkpoint "永久链接到此标题")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/distributed.checkpoint>
>
> 原始地址：<https://pytorch.org/docs/stable/distributed.checkpoint.html>


 分布式检查点 (DCP) 支持并行加载和保存多个等级的模型。它处理加载时重新分片，从而可以在一个集群拓扑中保存并加载到另一个集群拓扑中。


 DCP 在几个重要方面与 torch.save 和 torch.load 不同：



* 它在每个检查点生成多个文件，每个等级至少生成一个文件。 
* 它就地运行，这意味着模型应首先分配其数据，然后 DCP 使用该存储。


 加载和保存检查点的入口点如下：


 火炬.分布式.检查点。


 加载_state_dict


 ( *state_dict
* , *storage_reader
* , *process_group



 =
 


 无
* , *协调员_rank



 =
 


 0
* , *无_dist



 =
 


 错误*、*计划者



 =
 


 无
* ) [[source]](_modules/torch/distributed/checkpoint/state_dict_loader.html#load_state_dict)[¶](#torch.distributed.checkpoint.load_state_dict "此定义的永久链接")


 以 SPMD 风格加载分布式 `state_dict`。


 每个等级将尝试读取满足请求的 state_dict 所需的最少数据量。加载“ShardedTensor”实例时，每个等级仅读取其本地分片的数据。


!!! warning "警告"

     在调用此函数之前，“state_dict”中的所有tensor必须在其目标设备上分配。


 所有非tensor数据都使用 torch.load() 加载并在 state_dict 上进行修改。


!!! warning "警告"

     用户必须在根模块上调用 load_state_dict 以确保 loadpos 处理和非tensor数据正确传播。


 参数 
* **state_dict** ( *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)" )*,
* *Any
* *]
* ) – 要加载的 state_dict。请注意，此状态字典将就地更新。
* **storage_reader** ( [*StorageReader*](#torch.distributed.checkpoint.StorageReader "torch.distributed.checkpoint.StorageReader") ) – StorageReader 用于从以下位置加载数据.
* **process_group** ( *ProcessGroup
* ) – 用于跨等级同步的 ProcessGroup。
* **coordinator_rank** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 用于协调检查点的排名。默认情况下使用rank0。
* **no_dist** ( [*bool*]( https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` ，分布式检查点将不会以 SPMD 方式保存。 (默认值：`False`)


 退货


 None.
 


 Return type


 None
 


 例子




```
>>> my_model = MyModule()
>>> optimizer = Adagrad(my_model.parameters())
>>> model_state_dict = my_model.state_dict()
>>> fs_storage_reader = torch.distributed.checkpoint.FileSystemReader("/checkpoint/1")

```



```
>>> torch.distributed.checkpoint.load_state_dict(
>>>     state_dict=model_state_dict,
>>>     storage_reader=fs_storage_reader,
>>> )

```



```
>>> # module.load_state_dict() function might have customized steps
>>> # to flush the state_dict, must call it to
>>> # ensure correct behavior.
>>> my_model.load_state_dict(model_state_dict)

```




!!! note "笔记"

    load_state_dict 使用集体来协调跨队列的读取。对于基于 NCCL 的进程组，对象的内部tensor表示必须在通信发生之前移动到 GPU 设备。在这种情况下，使用的设备由 `torch.cuda 给出.current_device()` 并且用户有责任确保通过 `torch.cuda.set_device()` 进行设置，以便每个等级都有一个单独的 GPU。


 火炬.分布式.检查点。


 保存_state_dict


 ( *state_dict
* , *storage_writer
* , *process_group



 =
 


 无
* , *协调员_rank



 =
 


 0
* , *无_dist



 =
 


 错误*、*计划者



 =
 


 无
* ) [[source]](_modules/torch/distributed/checkpoint/state_dict_saver.html#save_state_dict)[¶](#torch.distributed.checkpoint.save_state_dict "此定义的永久链接")


 以 SPMD 样式保存分布式模型。


 此函数与 torch.save() 不同，因为它通过让每个等级仅保存其本地分片来处理 ShardedTensor。


!!! warning "警告"

     对于保存的状态_dicts，不保证跨 PyTorch 版本的向后兼容性。


!!! warning "警告"

     如果使用 process_group 参数，请确保只有其rankscall save_state_dict 并且 state_dict 中的所有数据都属于它。


!!! note "笔记"

    为 FSDP 的 ShardingStrategy.HYBRID_SHARD 保存检查点时，只有一个 shard_group 需要调用 save_state_dict 并传入相应的进程组。


!!! note "笔记"

    此函数可用于保存状态_dict，而无需通过传递“no_dist=True”来初始化进程组。


 参数 
* **state_dict** ( *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)" )*,
* *Any
* *]
* ) – 要保存的状态_dict。
* **storage_writer** ( [*StorageWriter*](#torch.distributed.checkpoint.StorageWriter "torch.distributed.checkpoint.StorageWriter ") ) – 用于执行写入的 StorageWrite 实例。
* **process_group** ( *ProcessGroup
* ) – 用于跨等级同步的 ProcessGroup。
* **coordinator_rank** ( [*int*] (https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 用于协调检查点的排名。默认情况下使用rank0。
* **否_dist** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` ，分布式检查点将不以 SPMD 方式保存。 (默认值：`False`)


 退货


 已保存检查点的元数据对象。


 Return type


 元数据


 例子


```
>>> my_model = MyModule()

```



```
>>> model_state_dict = my_model.state_dict()

```



```
>>> fs_storage_writer = torch.distributed.checkpoint.FileSystemWriter("/checkpoint/1")
>>> torch.distributed.checkpoint.save_state_dict(
>>>     state_dict=model_state_dict,
>>>     storage_writer=fs_storage_writer,
>>> )

```


!!! note "笔记"

    save_state_dict 使用集体来协调跨队列的写入。对于基于 NCCL 的进程组，对象的内部tensor表示必须在通信发生之前移动到 GPU 设备。在这种情况下，使用的设备由 `torch.cuda 给出.current_device()` 并且用户有责任确保通过 `torch.cuda.set_device()` 进行设置，以便每个等级都有一个单独的 GPU。


 此示例展示了如何使用 Pytorch 分布式检查点来保存 FSDP 模型。


 以下类型定义检查点期间使用的 IO 接口：


*班级*


 火炬.分布式.检查点。


 StorageReader [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageReader)[¶](#torch.distributed.checkpoint.StorageReader "此定义的永久链接")


 `load_state_dict` 用于从存储读取的接口。


 一个 StorageReader 实例在分布式检查点中既充当协调者又充当追随者。作为初始化的一部分，每个实例都会被告知其角色。


 子类应该预期 `load_state_dict` 的以下调用序列：


1.(所有级别)读取_metadata()2。 (所有等级)设置_up_storage_reader()3。 (所有级别)准备_local_plan()4。 (协调员)准备_global_plan()5。 (所有等级)read_data()


*抽象的*


 准备_全局_计划


 ( *plans
* ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageReader.prepare_global_plan)[¶](#torch.distributed.checkpoint.StorageReader.prepare_global_plan "此定义的永久链接")


 对存储装载进行集中规划。


 该方法仅在协调器实例上调用。


 虽然此方法可以生成完全不同的计划，但首选方法是将存储特定数据存储在 LoadPlan::storage_data 中。


 Parameters


**计划** ( [*List*](https://docs.python.org/3/library/typing.html#typing.List "(在 Python v3.12 中)")*[
* [*LoadPlan
* ](#torch.distributed.checkpoint.LoadPlan "torch.distributed.checkpoint.planner.LoadPlan")*]
* ) – `LoadPlan` 实例列表，每个等级一个。


 退货


 存储全局规划后转换后的`LoadPlan`列表


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*LoadPlan*](#torch.distributed.checkpoint.LoadPlan“torch.distributed.checkpoint.planner.LoadPlan”)]


*抽象的*


 准备_本地_计划


 ( *plan
* ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageReader.prepare_local_plan)[¶](#torch.distributed.checkpoint.StorageReader.prepare_local_plan "此定义的永久链接")


 执行特定于存储的本地规划。


 虽然此方法可以生成完全不同的计划，但推荐的方法是将存储特定数据存储在 LoadPlan::storage_data 中。


 Parameters


**plan** ( [*LoadPlan*](#torch.distributed.checkpoint.LoadPlan "torch.distributed.checkpoint.LoadPlan") ) – 来自正在使用的 `LoadPlan` 的本地计划。


 退货


 存储本地规划后转换后的“LoadPlan”


 Return type


[*LoadPlan*](#torch.distributed.checkpoint.LoadPlan "torch.distributed.checkpoint.planner.LoadPlan")


*抽象的*


 读取_数据


 ( *plan
* , *planner
* ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageReader.read_data)[¶](#torch.distributed.checkpoint.StorageReader.read_data "此定义的永久链接")


 使用“planner”读取“plan”中的所有项目来解析数据。


 子类应该调用“LoadPlanner::load_bytes”将 BytesIO 对象反序列化到正确的位置。


 子类应该调用“LoadPlanner::resolve_tensor”来访问应将数据加载到的tensor。


 存储层有责任正确安排所需的任何跨设备副本。


 参数 
* **plan** ( [*LoadPlan*](#torch.distributed.checkpoint.LoadPlan "torch.distributed.checkpoint.LoadPlan") ) – 要执行的本地计划
* **planner** ( [*LoadPlanner *](#torch.distributed.checkpoint.LoadPlanner "torch.distributed.checkpoint.LoadPlanner") ) – 用于解析项目的规划器对象。


 退货


 一旦所有读取完成，未来就完成了。


 Return type


[*Future*](futures.html#torch.futures.Future "torch.jit.Future") [无]


*抽象的*


 读取_元数据


 ( ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageReader.read_metadata)[¶](#torch.distributed.checkpoint.StorageReader.read_metadata "此定义的永久链接")


 读取检查点元数据。


 退货


 与正在加载的检查点关联的元数据对象。


 Return type


*元数据*


*抽象的*


 设置_up_storage_reader


 ( *metadata
* , *is_coordinator
* ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageReader.set_up_storage_reader)[¶](#torch.distributed.checkpoint.StorageReader.set_up_storage_reader "永久链接这个定义")


 初始化该实例。


 参数 
* **metadata** ( *Metadata
* ) – 要使用的元数据模式。
* **is_coordinator** ( [*bool*](https://docs.python.org/3/library/functions. html#bool "(in Python v3.12)") ) – 此实例是否负责协调检查点。


*班级*


 火炬.分布式.检查点。


 StorageWriter [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageWriter)[¶](#torch.distributed.checkpoint.StorageWriter "此定义的永久链接")


 `save_state_dict` 用于写入存储的接口。


 一个 StorageWriter 实例在分布式检查点中既充当协调者又充当追随者。作为初始化的一部分，每个实例都会被告知其角色。


 子类应该遵循以下调用顺序。


1.(所有等级)设置_up_storage_writer()2. (所有级别)准备_local_plan()3。 (协调员)准备_global_plan()4。 (所有等级)写入_data()5。 (协调员)完成()


*抽象的*


 结束


 ( *metadata
* , *results
* ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageWriter.finish)[¶](#torch.distributed.checkpoint.StorageWriter.finish "此定义的永久链接")


 写入元数据并将当前检查点标记为成功。


 用于序列化元数据的实际格式/模式是实现细节。唯一的要求是它可以恢复到同一个对象图中。


 参数 
* **元数据** ( *Metadata
* ) – 新检查点的元数据
* **结果** ( [*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12 中)")*[
* [*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12 中) ")*[
* *WriteResult
* *]
* *]
* ) – 所有级别的 WriteResult 列表。


 退货


 None
 


 Return type


 None
 


*抽象的*


 准备_全局_计划


 ( *plans
* ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageWriter.prepare_global_plan)[¶](#torch.distributed.checkpoint.StorageWriter.prepare_global_plan "此定义的永久链接")


 对存储进行集中规划。


 该方法仅在协调器实例上调用。


 虽然此方法可以生成完全不同的计划，但首选方法是将存储特定数据存储在 SavePlan::storage_data 中。


 Parameters


**计划** ( [*List*](https://docs.python.org/3/library/typing.html#typing.List "(in Python v3.12)")*[
* [*SavePlan
* ](#torch.distributed.checkpoint.SavePlan "torch.distributed.checkpoint.planner.SavePlan")*]
* ) – `SavePlan` 实例的列表，每个等级一个。


 退货


 存储全局规划后转换后的`SavePlan`列表


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*SavePlan*](#torch.distributed.checkpoint.SavePlan“torch.distributed.checkpoint.planner.SavePlan”)]


*抽象的*


 准备_本地_计划


 ( *plan
* ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageWriter.prepare_local_plan)[¶](#torch.distributed.checkpoint.StorageWriter.prepare_local_plan "此定义的永久链接")


 执行特定于存储的本地规划。


 虽然此方法可以生成完全不同的计划，但推荐的方法是将存储特定数据存储在 SavePlan::storage_data 中。


 Parameters


**plan** ( [*SavePlan*](#torch.distributed.checkpoint.SavePlan "torch.distributed.checkpoint.SavePlan") ) – 来自正在使用的 `SavePlanner` 的本地计划。


 退货


 存储本地规划后转换后的“SavePlan”


 Return type


[*保存计划*](#torch.distributed.checkpoint.SavePlan "torch.distributed.checkpoint.planner.SavePlan")


*抽象的*


 设置_up_storage_writer


 ( *is_coordinator
* ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageWriter.set_up_storage_writer)[¶](#torch.distributed.checkpoint.StorageWriter.set_up_storage_writer "此定义的永久链接")


 初始化该实例。


 Parameters


**is_coordinator** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 此实例是否负责用于协调检查点。


*抽象的*


 写入_数据


 ( *plan
* , *planner
* ) [[source]](_modules/torch/distributed/checkpoint/storage.html#StorageWriter.write_data)[¶](#torch.distributed.checkpoint.StorageWriter.write_data "此定义的永久链接")


 使用“planner”写入“plan”中的所有项目来解析数据。


 子类应该对计划中的每个项目调用 `SavePlanner::resolve_data` 来访问要写入的底层对象。


 子类应该延迟调用resolve_data，因为它可以分配内存。对于tensor，请做出以下假设：



* 它们可能位于任何设备上，包括与 `WriteItem::tensor_data` 上的设备不匹配
* 它们可能是视图或不连续。仅需要保存投影。


 参数 
* **plan** ( [*SavePlan*](#torch.distributed.checkpoint.SavePlan "torch.distributed.checkpoint.SavePlan") ) – 要执行的保存计划。
* **planner** ( [*SavePlanner *](#torch.distributed.checkpoint.SavePlanner "torch.distributed.checkpoint.SavePlanner") ) – 用于将项目解析为数据的 Planner 对象。


 退货


 完成 WriteResult 列表的 future


 Return type


[*Future*](futures.html#torch.futures.Future "torch.jit.Future") [ [*List*](https://docs.python.org/3/library/typing.html#typing.列表 "(Python v3.12)") [ *WriteResult
* ]]


 以下类型定义了检查点期间使用的规划器接口：


*班级*


 火炬.分布式.检查点。


 LoadPlanner [[source]](_modules/torch/distributed/checkpoint/planner.html#LoadPlanner)[¶](#torch.distributed.checkpoint.LoadPlanner "此定义的永久链接")


 定义 load_state_dict 用来规划加载过程的协议的抽象类。


 LoadPlanner 是有状态的对象，可用于自定义整个加载过程。


 LoadPlanner 充当 state_dict 的访问代理，因此对其所做的任何转换对于整个过程都是可见的。


 规划器子类在 load_state_dict 期间可以预期以下调用顺序：


1. set_up_planner 
- 召集所有队伍。


 发出开始加载检查点的信号。2。 create_local_plan 
- 号召所有队伍。


 处理 state_dict 并生成一个 LoadPlan，该 LoadPlan 将被发送用于全局规划。3。 create_global_plan 
- 仅在协调员级别上调用。


 从所有等级中获取 LoadPlan 并做出任何全局决策。4。 load_bytes 
- 在每个等级上调用多次


 state_dict.5 中的每个非tensor值都会调用一次。 solve_tensor 和 commit_tensor 
- 在每个等级上调用多次


 它们针对 state_dict 中的每个tensor值成对调用。


 建议用户扩展DefaultLoadPlanner而不是直接扩展此接口，因为大多数更改都可以通过单个方法的更改来表达。


 有两种常见的扩展模式：


 重写状态_dict。这是扩展加载过程的最简单方法，因为它不需要理解 LoadPlan 工作原理的本质。当负载发生时，我们需要保留对原始状态_dict的引用，因此我们需要能够就地执行它


```
>>> class RenamePlanner(DefaultLoadPlanner):
>>>     def set_up_planner(self, state_dict, metadata, is_coordinator):
>>>         self.original_state_dict = state_dict
>>>         super().set_up_planner(self, {"foo_" + k: v for k, v in state_dict.items()}, is_coordinator)
>>>
>>>     def load_bytes(self, read_item, value):
>>>         # Remove the "foo_" prefix
>>>         self.original_state_dict[read_item.dest_index.fqn[4:]] = torch.load(value)

```


 修改resolve_tensor和commit_tensor来处理加载时间转换。


```
>>> class MetaModelMaterialize(DefaultSavePlanner):
>>>     def resolve_tensor(self, read_item):
>>>         tensor = super().resolve_tensor(read_item)
>>>         return torch.empty_like(tensor, device="cpu")
>>>
>>>     def commit_tensor(self, read_item, tensor):
>>>         self.state_dict[read_item.dest_index.fqn] = tensor

```


*抽象的*


 提交_tensor


 ( *read_item
* , *tensor
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#LoadPlanner.commit_tensor)[¶](#torch.distributed.checkpoint.LoadPlanner.commit_tensor "永久链接这个定义")


 一旦 StorageReader 完成将数据加载到 `tensor` 中，就会调用此方法。


 提供的tensor与调用 `resolve_tensor` 返回的tensor相同。仅当 LoadPlanner 需要在将 `tensor` 复制回 state_dict 中的tensor之前对其进行后处理时，才需要此方法。


 tensor的内容将遵循其设备同步模型。


*抽象的*


 创建_全局_计划


 ( *global_plan
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#LoadPlanner.create_global_plan)[¶](#torch.distributed.checkpoint.LoadPlanner.create_global_plan "此定义的永久链接")


 计算每个等级的全局负载计划和返回计划。


 。注意：这仅在协调员级别上调用


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*LoadPlan*](#torch.distributed.checkpoint.LoadPlan“torch.distributed.checkpoint.planner.LoadPlan”)]


*抽象的*


 创建_本地_计划


 ( ) [[source]](_modules/torch/distributed/checkpoint/planner.html#LoadPlanner.create_local_plan)[¶](#torch.distributed.checkpoint.LoadPlanner.create_local_plan "此定义的永久链接")


 根据 set_up_planner 提供的 state_dict 和元数据创建 LoadPlan。


 。注意：每个级别都会调用此方法。


 Return type


[*LoadPlan*](#torch.distributed.checkpoint.LoadPlan "torch.distributed.checkpoint.planner.LoadPlan")


*抽象的*


 完成_计划


 ( *central_plan
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#LoadPlanner.finish_plan)[¶](#torch.distributed.checkpoint.LoadPlanner.finish_plan "此定义的永久链接")


 接受协调器的计划并返回最终的 LoadPlan。


 Return type


[*LoadPlan*](#torch.distributed.checkpoint.LoadPlan "torch.distributed.checkpoint.planner.LoadPlan")


*抽象的*


 加载_bytes


 ( *read_item
* , *value
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#LoadPlanner.load_bytes)[¶](#torch.distributed.checkpoint.LoadPlanner.load_bytes "永久链接这个定义")


 加载由 `read_item` 和 ``value` 描述的项目。


 该方法预计会就地修改底层状态_dict。


 “value”的内容由用于生成正在加载的检查点的 SavePlanner 定义。


*抽象的*


 解析_tensor


 ( *read_item
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#LoadPlanner.resolve_tensor)[¶](#torch.distributed.checkpoint.LoadPlanner.resolve_tensor "此定义的永久链接")


 返回由 `read_item` 描述的tensor，供 StorageReader 用于加载 read_item 。


 tensor应该与底层 state_dict 上的一个别名，因为 StorageReader 将替换其内容。如果出于某种原因，这是不可能的，规划器可以使用 `commit_tensor` 方法将数据复制回 state\ 中的数据_dict。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


*抽象的*


 设置_up_计划者


 ( *state_dict
* , *metadata
* , *is_coordinator
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#LoadPlanner.set_up_planner)[¶](#torch.distributed.checkpoint. LoadPlanner.set_up_planner"此定义的永久链接")


 初始化此实例以将数据加载到 `state_dict`


 。注意：每个级别都会调用此方法。


*班级*


 火炬.分布式.检查点。


 装载计划


 ( *项目



 :
 


 List
 


 [ [torch.distributed.checkpoint.planner.ReadItem](#torch.distributed.checkpoint.ReadItem "torch.distributed.checkpoint.planner.ReadItem")


 ]
* , *存储_数据



 :
 


 Any
 


 =
 


 无
* , *规划器_data



 :
 


 Any
 


 =
 


 无
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#LoadPlan)[¶](#torch.distributed.checkpoint.LoadPlan "此定义的永久链接")


*班级*


 火炬.分布式.检查点。


 读取项目


 ( *类型



 :
 


 torch.distributed.checkpoint.planner.LoadItemType
* , *dest_index



 :
 


 torch.distributed.checkpoint.metadata.MetadataIndex
* , *dest_offsets



 :
 


 torch.Size
* , *存储_index



 :
 


 torch.distributed.checkpoint.metadata.MetadataIndex
* , *storage_offsets



 :
 


 火炬.尺寸*，*长度



 :
 


 torch.Size
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#ReadItem)[¶](#torch.distributed.checkpoint.ReadItem "此定义的永久链接")


*班级*


 火炬.分布式.检查点。


 SavePlanner [[source]](_modules/torch/distributed/checkpoint/planner.html#SavePlanner)[¶](#torch.distributed.checkpoint.SavePlanner "此定义的永久链接")


 定义 save_state_dict 用来规划保存过程的协议的抽象类。


 SavePlanners 是有状态的对象，可用于自定义整个保存过程。


 SavePlanner 充当 state_dict 的访问代理，因此对其所做的任何转换都将对整个过程可见。


 规划器子类在 save_state_dict 期间可以预期以下调用顺序：


1. set_up_planner 
- 召集所有队伍。


 发出检查点保存开始的信号。2。 create_local_plan 
- 号召所有队伍。


 处理 state_dict 并生成一个 SavePlan，该 SavePlan 将被发送用于全局规划。3。 create_global_plan 
- 仅在协调员级别上调用。


 从所有级别中获取 SavePlan 并做出任何全局决策。4。 finish_plan 
- 号召所有队伍。


 这使每个级别都有机会适应全球规划决策。5。 solve_data 
- 在每个等级上调用多次


 在 state_dict 上查找要写入的存储层的值。


 建议用户扩展 DefaultSavePlanner 而不是直接扩展此接口，因为大多数更改都可以通过单个方法的更改来表达。


 有 3 种常见的扩展模式：


 重写状态_dict。这是扩展保存过程的最简单方法，因为它不需要理解 SavePlan 工作原理的复杂性：


```
>>> class RenamePlanner(DefaultSavePlanner):
>>>     def set_up_planner(self, state_dict, is_coordinator):
>>>         # prefix all keys with `foo_``
>>>         super().set_up_planner({"foo_" + k: v for k, v in state_dict.items()}, is_coordinator)

```


 同时修改本地计划和查找。当精确控制数据的持久化方式时，这非常有用


```
>>> class FP16Planner(DefaultSavePlanner):
>>>     def create_local_plan(self):
>>>         plan = super().create_local_plan()
>>>         for p in plan:
>>>             if p.tensor_data is not None:
>>>                 p.tensor_data.properties.dtype = torch.float16
>>>         return plan
>>>
>>>     def resolve_data(self, write_item):
>>>         item = super().resolve_data(write_item)
>>>         return item if write_item.type == WriteItemType.BYTE_IO else item.to(torch.float16)

```


 使用全局规划步骤来做出每个级别无法单独做出的中央决策


```
>>> from itertools import islice
>>> from dataclasses import replace
>>> class DDPLoadBalancingPlanner(DefaultSavePlanner):
>>>     # This uses the default local plan behavior of having all non-sharded writes in rank 0
>>>     # This sample doesn't handle ShardedTensors
>>>     def create_global_plan(self, all_plans):
>>>         def chunk(it, size):
>>>             it = iter(it)
>>>         return list(iter(lambda: tuple(islice(it, size)), ()))
>>>         all_plans = [
>>>             replace(plan, items=items) for plan, items in
>>>                 zip(all_plans, chunk(all_plans[0].items, len(all_plans)))
>>>         ]
>>>         return super().create_global_plan(all_plans)

```


 最后，一些规划器需要在检查点中保存额外的元数据，这是通过让每个等级在本地规划中贡献其数据项并由全局规划器聚合它们来完成的：


```
>>> class SaveExtraDataPlanner(DefaultSavePlanner):
>>>     def create_local_plan(self) -> SavePlan:
>>>         plan = super().create_local_plan()
>>>         return replace(plan, planner_data="per-rank-data")
>>>
>>>     def create_global_plan(self, all_plans: List[SavePlan]) -> Tuple[List[SavePlan], Metadata]:
>>>         global_plan, metadata = super().create_global_plan(all_plans)
>>>         merged_data = [p.planner_data for p in global_plan]
>>>         metadata = replace(metadata, planner_data=merged_data)
>>>         return global_plan, metadata

```


*抽象的*


 创建_全局_计划


 ( *all_plans
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#SavePlanner.create_global_plan)[¶](#torch.distributed.checkpoint.SavePlanner.create_global_plan "此定义的永久链接")


 计算全局检查点计划并返回每个等级的局部计划。


 这仅在协调员级别上调用。


 Return type


[*Tuple*](https://docs.python.org/3/library/typing.html#typing.Tuple "(在 Python v3.12 中)") [ [*List*](https://docs. python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*SavePlan*](#torch.distributed.checkpoint.SavePlan "torch.distributed.checkpoint.planner.SavePlan ") ], *元数据
* ]


*抽象的*


 创建_本地_计划


 ( ) [[source]](_modules/torch/distributed/checkpoint/planner.html#SavePlanner.create_local_plan)[¶](#torch.distributed.checkpoint.SavePlanner.create_local_plan "此定义的永久链接")


 计算当前Rank的保存计划。这将被聚合并传递给create_global_plan。Planner特定数据可以通过SavePlan::planner_data传递。


 这是对所有阶层的呼吁。


 Return type


[*保存计划*](#torch.distributed.checkpoint.SavePlan "torch.distributed.checkpoint.planner.SavePlan")


*抽象的*


 完成_计划


 ( *new_plan
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#SavePlanner.finish_plan)[¶](#torch.distributed.checkpoint.SavePlanner.finish_plan "此定义的永久链接")


 合并 create_local_plan 创建的计划和 create_global_plan 的结果。


 这是对所有阶层的呼吁。


 Return type


[*保存计划*](#torch.distributed.checkpoint.SavePlan "torch.distributed.checkpoint.planner.SavePlan")


*抽象的*


 解析_data


 ( *write_item
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#SavePlanner.resolve_data)[¶](#torch.distributed.checkpoint.SavePlanner.resolve_data "此定义的永久链接")


 在“state_dict”中查找与“write_item”关联的对象，并在存储层使用它之前应用任何转换(例如序列化)。


 对每个排名多次调用，最终 SavePlan 中的每个 WriteItem 至少调用一次。


 该方法应该是幂等的并且节省线程。 StorageWriter 实现可以根据需要频繁地调用它。


 任何分配内存的转换都应该在调用他的方法时延迟完成，以减少检查点所需的峰值内存。


 返回tensor时，它们可以是任何设备或格式，也可以是视图。存储层有责任弄清楚如何保存它们。


 Return type


[*Union*](https://docs.python.org/3/library/typing.html#typing.Union "(Python v3.12)") [ [*Tensor*](tensors.html#torch.tensor“torch.Tensor”)，*BytesIO*]


*抽象的*


 设置_up_计划者


 ( *state_dict
* , *is_coordinator
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#SavePlanner.set_up_planner)[¶](#torch.distributed.checkpoint.SavePlanner.set_up_planner "此定义的永久链接")


 初始化此规划器以保存 `state_dict` 。


 实现应该保存这些值，因为它们不会在保存过程中稍后提供。


 这是对所有阶层的呼吁。


*班级*


 火炬.分布式.检查点。


 保存计划


 ( *项目



 :
 


 List
 


 [ [torch.distributed.checkpoint.planner.WriteItem](#torch.distributed.checkpoint.WriteItem "torch.distributed.checkpoint.planner.WriteItem")


 ]
* , *存储_数据



 :
 


 Any
 


 =
 


 无
* , *规划器_data



 :
 


 Any
 


 =
 


 无
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#SavePlan)[¶](#torch.distributed.checkpoint.SavePlan "此定义的永久链接")


*班级*


 火炬.分布式.检查点。


 写项目


 ( *指数



 :
 


 torch.distributed.checkpoint.metadata.MetadataIndex
* , *类型



 :
 


 torch.distributed.checkpoint.planner.WriteItemType
* , *tensor_data



 :
 


 Union
 


 [
 


 torch.distributed.checkpoint.planner.TensorWriteData


 ,
 


 无类型


 ]
 



 =
 


 无
* ) [[source]](_modules/torch/distributed/checkpoint/planner.html#WriteItem)[¶](#torch.distributed.checkpoint.WriteItem "此定义的永久链接")


 我们提供基于文件系统的存储层：


*班级*


 火炬.分布式.检查点。


 文件系统读取器


 ( *path
* ) [[source]](_modules/torch/distributed/checkpoint/filesystem.html#FileSystemReader)[¶](#torch.distributed.checkpoint.FileSystemReader "此定义的永久链接")


*班级*


 火炬.分布式.检查点。


 文件系统写入器


 ( *路径
* , *单个_文件_每个_rank



 =
 


 True
* , *sync_files



 =
 


 True
* , *线程_count



 =
 


 1
* , *每个_线程_copy_ahead



 =
 


 10000000
* ) [[source]](_modules/torch/distributed/checkpoint/filesystem.html#FileSystemWriter)[¶](#torch.distributed.checkpoint.FileSystemWriter "此定义的永久链接")


 使用文件 IO 的 StorageWriter 基本实现。


 该实现做出了以下假设和简化：



* 检查点路径是一个空的或不存在的目录。
* 文件创建是原子的


 检查点由每个写入请求一个文件以及一个包含序列化元数据的.metadata 文件组成。


 我们提供了 LoadPlanner 和 SavePlanner 的默认实现，可以处理所有 torch.distributed 结构，例如 FSDP、DDP、ShardedTensor 和 DistributedTensor。


*班级*


 火炬.分布式.检查点。


 默认保存计划器


 ( *展平_state_dict



 =
 


 True
* , *展平_sharded_tensor



 =
 


 True
* , *dedup_replicated_tensors



 =
 


 True
* ) [[source]](_modules/torch/distributed/checkpoint/default_planner.html#DefaultSavePlanner)[¶](#torch.distributed.checkpoint.DefaultSavePlanner "此定义的永久链接")


 查找_object


 ( *index
* ) [[source]](_modules/torch/distributed/checkpoint/default_planner.html#DefaultSavePlanner.lookup_object)[¶](#torch.distributed.checkpoint.DefaultSavePlanner.lookup_object "此定义的永久链接")


 这是规划器界面的扩展，可以轻松扩展默认规划器


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)


 变换_object


 ( *write_item
* , *object
* ) [[source]](_modules/torch/distributed/checkpoint/default_planner.html#DefaultSavePlanner.transform_object)[¶](#torch.distributed.checkpoint.DefaultSavePlanner.transform_object "永久链接这个定义")


 这是规划器界面的扩展，可以轻松扩展默认规划器


*班级*


 火炬.分布式.检查点。


 默认负载规划器


 ( *展平_state_dict



 =
 


 True
* , *展平_sharded_tensor



 =
 


 True
* ) [[source]](_modules/torch/distributed/checkpoint/default_planner.html#DefaultLoadPlanner)[¶](#torch.distributed.checkpoint.DefaultLoadPlanner "此定义的永久链接")


 DefaultLoadPlanner 在 LoadPlanner 之上添加了多个功能。


 特别是它添加了以下内容：


 flatten_state_dict：使用嵌套字典处理 state_dictsflatten_sharded_tensors：对于 2D 并行模式下的 FSDP


 查找_tensor


 ( *index
* ) [[source]](_modules/torch/distributed/checkpoint/default_planner.html#DefaultLoadPlanner.lookup_tensor)[¶](#torch.distributed.checkpoint.DefaultLoadPlanner.lookup_tensor "此定义的永久链接")


 这是规划器界面的扩展，可以轻松扩展默认规划器


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 变换_tensor


 ( *read_item
* , *tensor
* ) [[source]](_modules/torch/distributed/checkpoint/default_planner.html#DefaultLoadPlanner.transform_tensor)[¶](#torch.distributed.checkpoint.DefaultLoadPlanner.transform_tensor "永久链接这个定义")


 这是规划器界面的扩展，可以轻松扩展默认规划器