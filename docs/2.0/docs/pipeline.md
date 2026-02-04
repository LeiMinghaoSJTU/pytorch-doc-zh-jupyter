# 管道并行性 [¶](#pipeline-parallelism "永久链接到此标题")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/pipeline>
>
> 原始地址：<https://pytorch.org/docs/stable/pipeline.html>


 管道并行最初是在 [Gpipe](https://arxiv.org/abs/1811.06965) 论文中引入的，是一种在多个 GPU 上训练大型模型的有效技术。


!!! warning "警告"

     管道并行性是实验性的，可能会发生变化。


## 使用多个 GPU 的模型并行性 [¶](#model-parallelism-using-multiple-gpus"此标题的永久链接")


 通常，对于不适合单个 GPU 的大型模型，会采用模型并行性，将模型的某些部分放置在不同的 GPU 上。不过，如果对于顺序模型天真地这样做，则训练过程会因 GPU 利用率不足而受到影响，因为只有一个 GPU GPU一次处于活动状态如下图所示：


![_images/no_pipe.png](_images/no_pipe.png)


 该图表示放置在 4 个不同 GPU(纵轴)上的 4 个层的模型。横轴代表训练该模型的时间，表明一次仅使用 1 个 GPU([图片来源](https://arxiv.org/abs/1811.06965))。 [¶](#id2"此图像的永久链接")


## 流水线执行 [¶](#pipelined-execution "永久链接到此标题")


 为了缓解这个问题，管道并行性将输入小批量拆分为多个微批量，并在多个 GPU 上以管道方式执行这些微批量。下图概述了这一点：


![_images/pipe.png](_images/pipe.png)


 该图表示放置在 4 个不同 GPU(纵轴)上的 4 个层的模型。横轴表示通过时间训练该模型，证明 GPU 的利用效率要高得多。但是，仍然存在一个气泡(如图所示)，其中某些 GPU 未得到利用。( [图片来源](https://arxiv. org/abs/1811.06965))。 [¶](#id3"此图像的永久链接")


## PyTorch 中的管道 API [¶](#pipe-apis-in-pytorch "永久链接到此标题")
- -


*班级*


 torch.distributed.pipeline.sync。



 Pipe
 


 ( *模块
* , *块



 =
 


 1
* , *检查点



 =
 


 '除了_last'*，*延迟_batch_norm



 =
 


 False
* ) [[source]](_modules/torch/distributed/pipeline/sync/pipe.html#Pipe)[¶](#torch.distributed.pipeline.sync.Pipe "此定义的永久链接")


 包装任意 [`nn.Sequential`](generated/torch.nn.Sequential.html#torch.nn.Sequential "torch.nn.Sequential") 模块以使用同步管道并行性进行训练。如果模块需要大量内存并且不适合单个 GPU，则管道并行性是一种用于训练的有用技术。


 该实现基于 [torchgpipe](https://arxiv.org/abs/2004.09910) 论文。


 Pipe 将管道并行性与检查点相结合，以减少训练所需的峰值内存，同时最大限度地减少设备利用率不足。


 您应该将所有模块放在适当的设备上，并将它们包装到一个 [`nn.Sequential`](generated/torch.nn.Sequential.html#torch.nn.Sequential "torch.nn.Sequential") 模块中，定义所需的顺序执行。如果模块不包含任何参数/缓冲区，则假定该模块应在 CPU 上执行，并且在执行之前将模块的适当输入tensor移至 CPU。此行为可以被“WithDevice”包装器覆盖，该包装器可用于显式指定模块应在哪个设备上运行。


 参数 
* **module** ( [`nn.Sequential`](generated/torch.nn.Sequential.html#torch.nn.Sequential "torch.nn.Sequential") ) – 使用流水线并行化的顺序模块。序列中的每个模块都必须在单个设备上具有其所有参数。序列中的每个模块必须是 nn.Module 或 [`nn.Sequential`](generated/torch.nn.Sequential.html#torch.nn.Sequential "torch.nn.Sequential") (以组合多个顺序模块单个设备)
* **块** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 块的数量微批次(默认值： `1` )
* **检查点** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中) ") ) – 何时启用检查点， `'always'` 、 `' except_last'` 或 `'never'` 之一(默认值： `' except_last'` )。 “never”完全禁用检查点，“ except_last”启用除最后一个之外的所有微批次的检查点，“always”启用所有微批次的检查点。
* **deferred_batch_norm** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否使用延迟 `BatchNorm` 移动统计(默认值： [`False`](https://docs.python.org/3/library/constants.html#False "(在 Python v3.12 中)") )。如果设置为 [`True`](https://docs.python.org/3/library/constants.html#True "(in Python v3.12)") ，我们会跟踪多个微批次的统计信息以更新正在运行的每小批量的统计数据。


 引发 
* [**TypeError**](https://docs.python.org/3/library/exceptions.html#TypeError "(in Python v3.12)") – 该模块不是 [`nn.Sequential `](generated/torch.nn.Sequential.html#torch.nn.Sequential "torch.nn.Sequential").
* [**ValueError**](https://docs.python.org/3/library/exceptions.html#ValueError "(in Python v3.12)") – 无效参数


 例子：：


 跨 GPU 0 和 1 的两个 FC 层的管道。


```
>>> # Need to initialize RPC framework first.
>>> os.environ['MASTER_ADDR'] = 'localhost'
>>> os.environ['MASTER_PORT'] = '29500'
>>> torch.distributed.rpc.init_rpc('worker', rank=0, world_size=1)
>>>
>>> # Build pipe.
>>> fc1 = nn.Linear(16, 8).cuda(0)
>>> fc2 = nn.Linear(8, 4).cuda(1)
>>> model = nn.Sequential(fc1, fc2)
>>> model = Pipe(model, chunks=8)
>>> input = torch.rand(16, 16).cuda(0)
>>> output_rref = model(input)

```




!!! note "笔记"

    您可以使用 [`torch.nn.parallel.DistributedDataParallel`](generated/torch. nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 仅当 checkpoint 参数为 [`Pipe`](#torch.distributed.pipeline.sync.Pipe "torch.distributed" 时.pipeline.sync.Pipe") 是 `'never'` 。


!!! note "笔记"

    [`Pipe`](#torch.distributed.pipeline.sync.Pipe "torch.distributed.pipeline.sync.Pipe") 目前仅支持节点内流水线，但将来会扩展以支持节点间流水线。 forward 函数返回一个“RRef”，以允许将来进行节点间管道传输，其中输出可能位于远程主机上。对于节点内管道，您可以使用“local_value()”在本地检索输出。


!!! warning "警告"

    [`Pipe`](#torch.distributed.pipeline.sync.Pipe "torch.distributed.pipeline.sync.Pipe") 是实验性的，可能会发生变化。


 向前


 (
 
*\*
 


 输入
* ) [[source]](_modules/torch/distributed/pipeline/sync/pipe.html#Pipe.forward)[¶](#torch.distributed.pipeline.sync.Pipe.forward "此定义的永久链接")


 通过管道处理单个输入小批量并返回指向输出的“RRef”。 [`Pipe`](#torch.distributed.pipeline.sync.Pipe "torch.distributed.pipeline.sync.Pipe") 是一个相当透明的模块包装器。它不会修改底层模块的输入和输出签名。但有类型限制。输入和输出必须至少包含一个tensor。此限制也适用于分区边界。


 输入序列作为“*inputs”送入管道的第一阶段。因此，该函数的位置参数应与管道第一阶段的位置参数匹配。相同的条件适用于管道的一个阶段的输出，该输出是下一阶段的输入。


 根据用于初始化 [`Pipe`](#torch.distributed.pipeline.sync.Pipe "torch.distributed.pipeline.sync.Pipe") 的 `chunks` 参数，输入tensor被分成多个微批次。假设批量大小是tensor的第一维，如果批量大小小于“块”，则微批量的数量等于批量大小。


 只有tensor被分成多个微批次，非tensor输入只是按原样复制到每个微批次中。对于管道最后阶段的非tensor输出，它们被聚合为“List”并返回给用户。例如，如果您有 2 个微批次返回整数 5，则用户将收到 [5, 5] 的合并输出


 所有输入tensor都需要与管道的第一个分区位于同一设备上。


 如果tensor使用“NoChunk”包装器进行包装，则tensor不会跨微批次进行分割，并且会按与非tensor类似的方式进行复制。


 Parameters


**输入** – 输入小批量


 退货


“RRef”到小批量的输出


 提高


[**TypeError**](https://docs.python.org/3/library/exceptions.html#TypeError "(in Python v3.12)") – 输入不包含至少一个tensor


 Return type


*R参考*


### 跳过连接 [¶](#skip-connections "永久链接到此标题")


 某些模型，如 [ResNeXt](https://pytorch.org/hub/pytorch_vision_resnext/) 不是完全顺序的，并且在层之间具有跳跃连接。天真地实现为管道并行性的一部分意味着我们需要通过多个复制某些层的输出。 GPU 直到我们最终到达跳跃连接层所在的 GPU。为了避免这种复制开销，我们提供了下面的 API 来在模型的不同层中存储和弹出tensor。


 torch.distributed.pipeline.sync.skip.skippable。


 可跳过


 (*藏匿



 =
 


 ()
* ， *流行音乐



 =
 


 ()
* ) [[source]](_modules/torch/distributed/pipeline/sync/skip/skippable.html#skippable)[¶](#torch.distributed.pipeline.sync.skip.skippable.skippable "永久链接到此定义")


 用于定义带有skipconnections的 [`nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") 的装饰器。装饰模块称为“可跳过”。即使模块没有被 [`Pipe`](#torch.distributed.pipeline.sync.Pipe "torch.distributed.pipeline.sync.Pipe") 包装，此功能也可以完美运行。


 每个跳跃tensor都由其名称管理。在操作跳过tensor之前，可跳过的模块必须通过 stash 和/或 pop 参数静态声明跳过tensor的名称。跳过具有预先声明名称的tensor可以通过 `yield stash(name, tensor)` 来隐藏或通过 `tensor = yield pop(name)` 弹出。


 这是一个三层的示例。名为“1to3”的跳跃tensor分别在第一层和最后一层被隐藏和弹出：


```
@skippable(stash=['1to3'])
class Layer1(nn.Module):
    def forward(self, input):
        yield stash('1to3', input)
        return f1(input)

class Layer2(nn.Module):
    def forward(self, input):
        return f2(input)

@skippable(pop=['1to3'])
class Layer3(nn.Module):
    def forward(self, input):
        skip_1to3 = yield pop('1to3')
        return f3(input) + skip_1to3

model = nn.Sequential(Layer1(), Layer2(), Layer3())

```


 一个可跳过的模块可以存储或弹出多个跳过tensor：


```
@skippable(stash=['alice', 'bob'], pop=['carol'])
class StashStashPop(nn.Module):
    def forward(self, input):
        yield stash('alice', f_alice(input))
        yield stash('bob', f_bob(input))
        carol = yield pop('carol')
        return input + carol

```


 每个跳过tensor必须与一对 stash 和 pop 关联。 [`Pipe`](#torch.distributed.pipeline.sync.Pipe "torch.distributed.pipeline.sync.Pipe") 在包装模块时自动检查此限制。您还可以通过 [`verify_skippables()`](#torch.distributed.pipeline.sync.skip.skippable.verify_skippables "torch.distributed.pipeline.sync.skip.skippable.verify_skippables") 来检查限制，而无需 [`Pipe `](#torch.distributed.pipeline.sync.Pipe "torch.distributed.pipeline.sync.Pipe") 。


 Return type


[*Callable*](https://docs.python.org/3/library/typing.html#typing.Callable "(在 Python v3.12 中)") [[ [*Type*](https://docs.python.org/3/library/typing.html#typing.Type "(Python v3.12)") [ [*Module*](generated/torch.nn.Module.html#torch.nn.Module "火炬.nn.modules.module.Module") ]], [*Type*](https://docs.python.org/3/library/typing.html#typing.Type "(在 Python v3.12 中)") [ *可跳过
* ]]


*班级*


 torch.distributed.pipeline.sync.skip.skippable。



 stash
 


 ( *name
* , *tensor
* ) [[source]](_modules/torch/distributed/pipeline/sync/skip/skippable.html#stash)[¶](#torch.distributed.pipeline.sync.skip.skippable. stash"此定义的永久链接")


 存储跳过tensor的命令。


```
def forward(self, input):
    yield stash('name', input)
    return f(input)

```


 参数 
* **name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 跳过tensor的名称
* **input** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor")*或
* *None
* ) – 传递到跳过连接的tensor


*班级*


 torch.distributed.pipeline.sync.skip.skippable。



 pop
 


 ( *name
* ) [[source]](_modules/torch/distributed/pipeline/sync/skip/skippable.html#pop)[¶](#torch.distributed.pipeline.sync.skip.skippable.pop "永久链接这个定义")


 弹出跳过tensor的命令。


```
def forward(self, input):
    skip = yield pop('name')
    return f(input) + skip

```


 Parameters


**name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 跳过tensor的名称


 退货


 先前由同名的另一层存储的跳过tensor


 Return type


 None
 


 torch.distributed.pipeline.sync.skip.skippable。


 验证_skipables


 ( *module
* ) [[source]](_modules/torch/distributed/pipeline/sync/skip/skippable.html#verify_skippables)[¶](#torch.distributed.pipeline.sync.skip.skippable.verify_skippables "永久链接这个定义")


 验证底层可跳过模块是否满足完整性。


 每个跳过tensor必须只有一对 stash 和 pop 。如果存在一对或多对不匹配，它将引发 [`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError "(in Python v3.12)") 并包含详细消息。


 以下是一些失败案例。 [`verify_skippables()`](#torch.distributed.pipeline.sync.skip.skippable.verify_skippables "torch.distributed.pipeline.sync.skip.skippable.verify_skippables") 将报告以下情况的失败：


```
# Layer1 stashes "1to3".
# Layer3 pops "1to3".

nn.Sequential(Layer1(), Layer2())
# └──── ?

nn.Sequential(Layer2(), Layer3())
# ? ────┘

nn.Sequential(Layer1(), Layer2(), Layer3(), Layer3())
# └───────────────────┘ ^^^^^^

nn.Sequential(Layer1(), Layer1(), Layer2(), Layer3())
# ^^^^^^ └───────────────────┘

```


 要对多个跳过tensor使用相同的名称，它们必须由不同的命名空间隔离。请参阅“isolate()”。


 提高


[**TypeError**](https://docs.python.org/3/library/exceptions.html#TypeError "(in Python v3.12)") – 一对或多对 stash 和 pop 不匹配。


## 教程 [¶](#tutorials "此标题的永久链接")


 以下教程很好地概述了如何使用 [`Pipe`](#torch.distributed.pipeline.sync.Pipe "torch.distributed.pipeline.sync.Pipe") API 通过其余组件训练模型PyTorch 提供：



* [使用管道并行性训练 Transformer 模型](https://pytorch.org/tutorials/intermediate/pipeline_tutorial.html)
* [使用分布式数据并行性和管道并行性训练 Transformer 模型](https://pytorch.org/tutorials/高级/ddp_pipeline.html)


## 致谢 [¶](#acknowledgements "此标题的永久链接")


 管道并行的实现基于 [fairscale 的管道实现](https://github.com/facebookresearch/fairscale/tree/main/fairscale/nn/pipe) 和 [torchgpipe](https://github.com/kakaobrain) /火炬管)。我们要感谢两个团队为将管道并行性引入 PyTorch 所做的贡献和指导。