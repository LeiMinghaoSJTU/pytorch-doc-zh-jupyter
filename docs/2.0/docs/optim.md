# torch.optim [¶](#module-torch.optim "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/optim>
>
> 原始地址：<https://pytorch.org/docs/stable/optim.html>


[`torch.optim`](#module-torch.optim "torch.optim") 是一个实现各种优化算法的包。已经支持了最常用的方法，并且接口足够通用，因此也可以实现更复杂的方法轻松融入未来。


## 如何使用优化器 [¶](#how-to-use-an-optimizer "永久链接到此标题")


 要使用 torch.optim ，您必须构造一个优化器对象，该对象将保存当前状态并根据计算的梯度更新参数。


### 构建它 [¶](#constructing-it "此标题的永久链接")


 要构造一个 [`Optimizer`](#torch.optim.Optimizer "torch.optim.Optimizer")，您必须给它一个包含要优化的参数(全部应该是 `Variable` )的迭代。然后，您可以指定优化器特定的选项，例如学习率、权重衰减等。


 例子：


```
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
optimizer = optim.Adam([var1, var2], lr=0.0001)

```


### 每个参数选项 [¶](#per-parameter-options "永久链接到此标题")


[`Optimizer`](#torch.optim.Optimizer "torch.optim.Optimizer") 还支持指定每个参数选项。为此，不要传递 `Variable` 的迭代，而是传递 [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(在 Python v3.1 中) 的迭代。 12)") 秒。它们中的每一个都将定义一个单独的参数组，并且应该包含一个“params”键，其中包含属于它的参数列表。其他键应与优化器接受的关键字参数匹配，并将用作该组的优化选项。




!!! note "笔记"

    您仍然可以将选项作为关键字参数传递。它们将在未覆盖它们的组中用作默认值。当您只想更改单个选项，同时保持参数组之间的所有其他选项一致时，这非常有用。


 例如，当想要指定每一层的学习率时，这非常有用：


```
optim.SGD([
                {'params': model.base.parameters()},
                {'params': model.classifier.parameters(), 'lr': 1e-3}
            ], lr=1e-2, momentum=0.9)

```


 这意味着 model.base 的参数将使用默认学习率 1e-2，model.classifier 的参数将使用学习率 1e-3 和动量 0.9 ` 将用于所有参数。


### 采取优化步骤 [¶](#take-an-optimization-step "永久链接到此标题")


 所有优化器都实现 [`step()`](generated/torch.optim.Optimizer.step.html#torch.optim.Optimizer.step "torch.optim.Optimizer.step") 方法，用于更新参数。它可以通过两种方式使用：


#### `optimizer.step()`[¶](#optimizer-step "永久链接到此标题")


 这是大多数优化器支持的简化版本。一旦使用例如计算梯度，就可以调用该函数`向后()` 。


 例子：


```
for input, target in dataset:
    optimizer.zero_grad()
    output = model(input)
    loss = loss_fn(output, target)
    loss.backward()
    optimizer.step()

```


#### `optimizer.step(closure)`[¶](#optimizer-step-closure "此标题的永久链接")


 一些优化算法(例如共轭梯度和 LBFGS)需要多次重新评估函数，因此您必须传入一个闭包以允许它们重新计算您的模型。闭包应该清除梯度、计算损失并返回它。


 例子：


```
for input, target in dataset:
    def closure():
        optimizer.zero_grad()
        output = model(input)
        loss = loss_fn(output, target)
        loss.backward()
        return loss
    optimizer.step(closure)

```


## 基类 [¶](#base-class "此标题的永久链接")


*班级*


 火炬优化。


 优化器


 ( *params
* , *defaults
* ) [[source]](_modules/torch/optim/optimizer.html#Optimizer)[¶](#torch.optim.Optimizer "此定义的永久链接")


 所有优化器的基类。


!!! warning "警告"

     参数需要指定为具有在运行之间一致的确定性顺序的集合。不满足这些属性的对象的示例是字典值的集合和迭代器。


 参数 
* **params** ( *iterable
* ) – [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") s 或 [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(在 Python v3.12 中)") s。指定应优化哪些tensor。
* **defaults** ( [*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(in Python v3.12)" )*[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(in Python v3.12)")*]
* ) – (dict)：包含 optimizationoptions 默认值的字典(当参数组没有指定它们)。


|  |  |
| --- | --- |
| 	[`Optimizer.add_param_group`](generated/torch.optim.Optimizer.add_param_group.html#torch.optim.Optimizer.add_param_group "torch.optim.Optimizer.add_param_group")	 | 	 Add a param group to the	 [`Optimizer`](#torch.optim.Optimizer "torch.optim.Optimizer")	 s	 	 param_groups	 	.	  |
| 	[`Optimizer.load_state_dict`](generated/torch.optim.Optimizer.load_state_dict.html#torch.optim.Optimizer.load_state_dict "torch.optim.Optimizer.load_state_dict")	 | 	 Loads the optimizer state.	  |
| 	[`Optimizer.state_dict`](generated/torch.optim.Optimizer.state_dict.html#torch.optim.Optimizer.state_dict "torch.optim.Optimizer.state_dict")	 | 	 Returns the state of the optimizer as a	 [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)")	.	  |
| 	[`Optimizer.step`](generated/torch.optim.Optimizer.step.html#torch.optim.Optimizer.step "torch.optim.Optimizer.step")	 | 	 Performs a single optimization step (parameter update).	  |
| 	[`Optimizer.zero_grad`](generated/torch.optim.Optimizer.zero_grad.html#torch.optim.Optimizer.zero_grad "torch.optim.Optimizer.zero_grad")	 | 	 Resets the gradients of all optimized	 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor")	 s.	  |


## 算法 [¶](#algorithms "此标题的永久链接")


|  |  |
| --- | --- |
| [`Adadelta`](generated/torch.optim.Adadelta.html#torch.optim.Adadelta "torch.optim.Adadelta") |实现 Adadelta 算法。 |
| [`Adagrad`](generated/torch.optim.Adagrad.html#torch.optim.Adagrad“torch.optim.Adagrad”) |实现 Adagrad 算法。 |
| [`Adam`](generated/torch.optim.Adam.html#torch.optim.Adam "torch.optim.Adam") |实现 Adam 算法。 |
| [`AdamW`](generated/torch.optim.AdamW.html#torch.optim.AdamW "torch.optim.AdamW") |实现 AdamW 算法。 |
| [`SparseAdam`](generated/torch.optim.SparseAdam.html#torch.optim.SparseAdam "torch.optim.SparseAdam") | SparseAdam 实现了适合稀疏梯度的 Adam 算法的屏蔽版本。 |
| [`Adamax`](generated/torch.optim.Adamax.html#torch.optim.Adamax "torch.optim.Adamax") |实现 Adamax 算法(Adam 基于无穷范数的变体)。 |
| [`ASGD`](generated/torch.optim.ASGD.html#torch.optim.ASGD“torch.optim.ASGD”)|实现平均随机梯度下降。 |
| [`LBFGS`](generated/torch.optim.LBFGS.html#torch.optim.LBFGS“torch.optim.LBFGS”)|实现 L-BFGS 算法，深受 [minFunc](https://www.cs.ubc.ca/~schmidtm/Software/minFunc.html) 的启发。 |
| [`NAdam'](generated/torch.optim.NAdam.html#torch.optim.NAdam“torch.optim.NAdam”)|实现 NAdam 算法。 |
| [`RAdam`](generated/torch.optim.RAdam.html#torch.optim.RAdam "torch.optim.RAdam") |实现RAdam算法。 |
| [`RMSprop`](generated/torch.optim.RMSprop.html#torch.optim.RMSprop“torch.optim.RMSprop”)|实现 RMSprop 算法。 |
| [`Rprop`](generated/torch.optim.Rprop.html#torch.optim.Rprop“torch.optim.Rprop”) |实现弹性反向传播算法。 |
| [`SGD`](generated/torch.optim.SGD.html#torch.optim.SGD“torch.optim.SGD”)|实现随机梯度下降(可以选择使用动量)。 |


 我们的许多算法都有针对性能、可读性和/或通用性进行优化的各种实现，因此，如果用户没有指定特定的实现，我们会尝试默认为当前设备通常最快的实现。


 我们有 3 个主要的实现类别：for-loop、foreach(多tensor)和fused。最直接的实现是对具有大量计算的参数进行 for 循环。 For 循环通常比我们的 foreach 实现慢，后者将参数组合成多tensor并同时运行大量计算，从而节省了许多连续的内核调用。我们的一些优化器具有更快的融合实现，它将大块计算融合到一个内核中。我们可以将 foreach 实现视为水平融合，将融合实现视为在此基础上垂直融合。


 一般来说，这 3 种实现的性能顺序是 fused 
> foreach 
> for-loop。因此，在适用的情况下，我们默认使用 foreach 而不是 for-loop。适用意味着 foreach 实现可用，用户没有指定任何特定于实现的 kwargs(例如，融合、foreach、可微分)，并且所有tensor都是原生的且位于 CUDA 上。请注意，虽然 fused 应该比 foreach 更快，但实现较新，我们希望在到处切换开关之前给它们更多的烘烤时间。欢迎您尝试一下！


 下表显示了每种算法的可用和默认实现：


| 	 Algorithm	  | 	 Default	  | 	 Has foreach?	  | 	 Has fused?	  |
| --- | --- | --- | --- |
| [`Adadelta`](generated/torch.optim.Adadelta.html#torch.optim.Adadelta "torch.optim.Adadelta") | foreach |是的 |没有 |
| [`Adagrad`](generated/torch.optim.Adagrad.html#torch.optim.Adagrad“torch.optim.Adagrad”) | foreach |是的 |没有 |
| [`Adam`](generated/torch.optim.Adam.html#torch.optim.Adam "torch.optim.Adam") | foreach |是的 |是的|
| [`AdamW`](generated/torch.optim.AdamW.html#torch.optim.AdamW "torch.optim.AdamW") | foreach |是的 |是的|
| [`SparseAdam`](generated/torch.optim.SparseAdam.html#torch.optim.SparseAdam "torch.optim.SparseAdam") | for 循环 |没有|没有 |
| [`Adamax`](generated/torch.optim.Adamax.html#torch.optim.Adamax "torch.optim.Adamax") | foreach |是的 |没有 |
| [`ASGD`](generated/torch.optim.ASGD.html#torch.optim.ASGD“torch.optim.ASGD”)| foreach |是的 |没有 |
| [`LBFGS`](generated/torch.optim.LBFGS.html#torch.optim.LBFGS“torch.optim.LBFGS”)| for 循环 |没有|没有 |
| [`NAdam`](generated/torch.optim.NAdam.html#torch.optim.NAdam "torch.optim.NAdam") | foreach |是的 |没有 |
| [`RAdam`](generated/torch.optim.RAdam.html#torch.optim.RAdam "torch.optim.RAdam") | foreach |是的 |没有 |
| [`RMSprop`](generated/torch.optim.RMSprop.html#torch.optim.RMSprop“torch.optim.RMSprop”)| foreach |是的 |没有 |
| [`Rprop`](generated/torch.optim.Rprop.html#torch.optim.Rprop“torch.optim.Rprop”) | foreach |是的 |没有 |
| [`SGD`](generated/torch.optim.SGD.html#torch.optim.SGD“torch.optim.SGD”)| foreach |是的 |没有|


## 如何调整学习率[¶](#how-to-adjust-learning-rate "永久链接到此标题")


`torch.optim.lr_scheduler` 提供了几种根据时期数调整学习率的方法。 [`torch.optim.lr_scheduler.ReduceLROnPlateau`](generated/torch.optim.lr_scheduler.ReduceLROnPlateau.html#torch.optim.lr_scheduler.ReduceLROnPlateau "torch.optim.lr_scheduler.ReduceLROnPlateau") 允许基于一些验证测量。


 学习率调度应在优化器更新后应用；例如，您应该这样编写代码：


 例子：


```
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
scheduler = ExponentialLR(optimizer, gamma=0.9)

for epoch in range(20):
    for input, target in dataset:
        optimizer.zero_grad()
        output = model(input)
        loss = loss_fn(output, target)
        loss.backward()
        optimizer.step()
    scheduler.step()

```


 大多数学习率调度程序可以称为背靠背(也称为链接调度程序)。结果是每个调度器相继应用前一个调度器获得的学习率。


 例子：


```
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
scheduler1 = ExponentialLR(optimizer, gamma=0.9)
scheduler2 = MultiStepLR(optimizer, milestones=[30,80], gamma=0.1)

for epoch in range(20):
    for input, target in dataset:
        optimizer.zero_grad()
        output = model(input)
        loss = loss_fn(output, target)
        loss.backward()
        optimizer.step()
    scheduler1.step()
    scheduler2.step()

```


 在文档的许多地方，我们将使用以下模板来引用调度程序算法。


```
>>> scheduler = ...
>>> for epoch in range(100):
>>>     train(...)
>>>     validate(...)
>>>     scheduler.step()

```


!!! warning "警告"

     在 PyTorch 1.1.0 之前，学习率调度器预计会在优化器更新之前调用； 1.1.0 以破坏 BC 的方式改变了这种行为。如果您在优化器更新(调用“optimizer.step()”)之前使用学习率调度程序(调用“scheduler.step()”)，这将跳过学习率调度的第一个值。如果您无法重现结果升级到 PyTorch 1.1.0 后，请检查您是否在错误的时间调用了 `scheduler.step()`。


|  |  |
| --- | --- |
| 	[`lr_scheduler.LambdaLR`](generated/torch.optim.lr_scheduler.LambdaLR.html#torch.optim.lr_scheduler.LambdaLR "torch.optim.lr_scheduler.LambdaLR")	 | 	 Sets the learning rate of each parameter group to the initial lr times a given function.	  |
| 	[`lr_scheduler.MultiplicativeLR`](generated/torch.optim.lr_scheduler.MultiplicativeLR.html#torch.optim.lr_scheduler.MultiplicativeLR "torch.optim.lr_scheduler.MultiplicativeLR")	 | 	 Multiply the learning rate of each parameter group by the factor given in the specified function.	  |
| 	[`lr_scheduler.StepLR`](generated/torch.optim.lr_scheduler.StepLR.html#torch.optim.lr_scheduler.StepLR "torch.optim.lr_scheduler.StepLR")	 | 	 Decays the learning rate of each parameter group by gamma every step_size epochs.	  |
| 	[`lr_scheduler.MultiStepLR`](generated/torch.optim.lr_scheduler.MultiStepLR.html#torch.optim.lr_scheduler.MultiStepLR "torch.optim.lr_scheduler.MultiStepLR")	 | 	 Decays the learning rate of each parameter group by gamma once the number of epoch reaches one of the milestones.	  |
| 	[`lr_scheduler.ConstantLR`](generated/torch.optim.lr_scheduler.ConstantLR.html#torch.optim.lr_scheduler.ConstantLR "torch.optim.lr_scheduler.ConstantLR")	 | 	 Decays the learning rate of each parameter group by a small constant factor until the number of epoch reaches a pre-defined milestone: total_iters.	  |
| 	[`lr_scheduler.LinearLR`](generated/torch.optim.lr_scheduler.LinearLR.html#torch.optim.lr_scheduler.LinearLR "torch.optim.lr_scheduler.LinearLR")	 | 	 Decays the learning rate of each parameter group by linearly changing small multiplicative factor until the number of epoch reaches a pre-defined milestone: total_iters.	  |
| 	[`lr_scheduler.ExponentialLR`](generated/torch.optim.lr_scheduler.ExponentialLR.html#torch.optim.lr_scheduler.ExponentialLR "torch.optim.lr_scheduler.ExponentialLR")	 | 	 Decays the learning rate of each parameter group by gamma every epoch.	  |
| 	[`lr_scheduler.PolynomialLR`](generated/torch.optim.lr_scheduler.PolynomialLR.html#torch.optim.lr_scheduler.PolynomialLR "torch.optim.lr_scheduler.PolynomialLR")	 | 	 Decays the learning rate of each parameter group using a polynomial function in the given total_iters.	  |
| 	[`lr_scheduler.CosineAnnealingLR`](generated/torch.optim.lr_scheduler.CosineAnnealingLR.html#torch.optim.lr_scheduler.CosineAnnealingLR "torch.optim.lr_scheduler.CosineAnnealingLR")	 | 	 Set the learning rate of each parameter group using a cosine annealing schedule, where




 η
 


 最大限度


 eta_{最大值}



 η
 




 ma
 

 x
 


 ​
 


 设置为初始 lr 并且




 T
 


 你的


 T_{cur}



 T
 


 你的


 ​
 


 是自 SGDR 中上次重新启动以来的历元数：|
| [`lr_scheduler.ChainedScheduler`](generated/torch.optim.lr_scheduler.ChainedScheduler.html#torch.optim.lr_scheduler.ChainedScheduler "torch.optim.lr_scheduler.ChainedScheduler") |学习率调度程序的链列表。 |
| [`lr_scheduler.SequentialLR`](generated/torch.optim.lr_scheduler.SequentialLR.html#torch.optim.lr_scheduler.SequentialLR“torch.optim.lr_scheduler.SequentialLR”)|接收预计在优化过程中按顺序调用的调度程序列表和里程碑点，提供精确的时间间隔以反映在给定时期应调用哪个调度程序。 |
| [`lr_scheduler.ReduceLROnPlateau`](generated/torch.optim.lr_scheduler.ReduceLROnPlateau.html#torch.optim.lr_scheduler.ReduceLROnPlateau "torch.optim.lr_scheduler.ReduceLROnPlateau") |当指标停止改进时降低学习率。 |
| [`lr_scheduler.CyclicLR`](generated/torch.optim.lr_scheduler.CyclicLR.html#torch.optim.lr_scheduler.CyclicLR“torch.optim.lr_scheduler.CyclicLR”)|根据循环学习率策略(CLR)设置每个参数组的学习率。 |
| [`lr_scheduler.OneCycleLR`](generated/torch.optim.lr_scheduler.OneCycleLR.html#torch.optim.lr_scheduler.OneCycleLR "torch.optim.lr_scheduler.OneCycleLR") |按照1cycle学习率策略设置各参数组的学习率。 |
| [`lr_scheduler.CosineAnnealingWarmRestarts`](generated/torch.optim.lr_scheduler.CosineAnnealingWarmRestarts.html#torch.optim.lr_scheduler.CosineAnnealingWarmRestarts“torch.optim.lr_scheduler.CosineAnnealingWarmRestarts”)|使用余弦退火计划设置每个参数组的学习率，其中




 η
 


 最大限度


 eta_{最大值}



 η
 




 ma
 

 x
 


 ​
 


 被设置为初始lr，




 T
 


 你的


 T_{cur}



 T
 


 你的


 ​
 


 是自上次重新启动以来的纪元数，




 T
 

 i
 


 T_{i}



 T
 




 i
 


 ​
 


 是 SGDR 中两次热重启之间的纪元数：


## 权重平均(SWA 和 EMA)[¶](#weight-averaging-swa-and-ema "此标题的固定链接")


`torch.optim.swa_utils` 实现随机权重平均 (SWA) 和指数移动平均 (EMA)。特别是，`torch.optim.swa_utils.AvergedModel`类实现了SWA和EMA模型，`torch.optim.swa_utils.SWALR`实现了SWA学习率调度器和`torch.optim.swa_utils.update _bn()` 是一个实用函数，用于在训练结束时更新 SWA/EMA 批量归一化统计数据。


 SWA 已在 [Averaging Weights Leads to Wider Optima and Better Generalization](https://arxiv.org/abs/1803.05407) 中提出。


 EMA 是一种众所周知的技术，可通过减少所需的权重更新次数来减少训练时间。它是 [Polyak averaging](https://paperswithcode.com/method/polyak-averaging) 的变体，但在迭代中使用指数权重而不是相等权重。


### 构建平均模型 [¶](#constructing-averagged-models "此标题的永久链接")


 AveragedModel 类用于计算 SWA 或 EMA 模型的权重。


 您可以通过运行以下命令创建 SWA 平均模型：


```
>>> averaged_model = AveragedModel(model)

```


 EMA 模型是通过指定 `multi_avg_fn` 参数构建的，如下所示：


```
>>> decay = 0.999
>>> averaged_model = AveragedModel(model, multi_avg_fn=get_ema_multi_avg_fn(decay))

```


 衰减是一个介于 0 和 1 之间的参数，用于控制平均参数衰减的速度。如果未提供给 `get_ema_multi_avg_fn` ，则默认值为 0.999。


`get_ema_multi_avg_fn` 返回一个函数，该函数将以下 EMA 方程应用于权重：


 W
 


 t+1


 EMA
 


 =
 

 α
 


 EMA


 
+ ( 1 − a )


 WT模型


 W^	extrm{EMA}_{t+1} = lpha W^	extrm{EMA}_{t} 
+ (1 
- lpha) W^	extrm{模型}_t



 W
 


 t+1



 EMA
 


 ​
 




 =
 




 α
 


 W
 




 t
 



 EMA
 


 ​
 




 +
 




 (
 

 1
 



 −
 




 α
 

 )
 


 W
 



 t
 


 model
 


 ​
 


 其中 alpha 是 EMA 衰减。


 这里的模型 model 可以是任意的 [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") 对象。 `averaging_model` 将跟踪 `model` 参数的运行平均值。要更新这些平均值，您应该在 optimizationr.step() 之后使用 update_parameters() 函数：


```
>>> averaged_model.update_parameters(model)

```


 对于 SWA 和 EMA，此调用通常在优化器“step()”之后立即完成。对于 SWA，通常在训练开始时跳过一些步骤。


### 自定义平均策略 [¶](#custom-averaging-strategies "永久链接到此标题")


 默认情况下，“torch.optim.swa_utils.AvergedModel”计算您提供的参数的运行相等平均值，但您也可以使用带有“avg_fn”或“multi_avg_fn”参数的自定义平均函数：



* `avg_fn` 允许定义对每个参数元组(平均参数、模型参数)进行操作的函数，并应返回新的平均参数。
* `multi_avg_fn` 允许定义对参数列表元组进行更有效的操作，(平均参数列表，模型参数列表)，同时，例如使用 `torch._foreach*` 函数。该函数必须就地更新平均参数。


 在以下示例中，“ema_model”使用“avg_fn”参数计算指数移动平均值：


```
>>> ema_avg = lambda averaged_model_parameter, model_parameter, num_averaged:>>>         0.9 * averaged_model_parameter + 0.1 * model_parameter
>>> ema_model = torch.optim.swa_utils.AveragedModel(model, avg_fn=ema_avg)

```


 在以下示例中，“ema_model”使用更高效的“multi_avg_fn”参数计算指数移动平均值：


```
>>> ema_model = AveragedModel(model, multi_avg_fn=get_ema_multi_avg_fn(0.9))

```


### SWA 学习率计划 [¶](#swa-learning-rate-schedules "永久链接到此标题")


 通常，在 SWA 中，学习率被设置为一个较高的恒定值。 `SWALR` 是一个学习率调度器，它将学习率退火到固定值，然后保持恒定。例如，以下代码创建一个调度程序，在每个参数组内的 5 个时期内将学习率从初始值线性退火到 0.05：


```
>>> swa_scheduler = torch.optim.swa_utils.SWALR(optimizer, >>>         anneal_strategy="linear", anneal_epochs=5, swa_lr=0.05)

```


 您还可以通过设置 `anneal_strategy="cos"` 使用余弦退火到固定值而不是线性退火。


### 处理批量标准化 [¶](#take-care-of-batch-normalization "永久链接到此标题")


update_bn() 是一个实用函数，允许在训练结束时计算给定数据加载器 loader 上的 SWA 模型的批标准化统计数据：


```
>>> torch.optim.swa_utils.update_bn(loader, swa_model)

```


`update_bn()` 将 `swa_model` 应用到数据加载器中的每个元素，并计算模型中每个批量归一化层的激活统计信息。


!!! warning "警告"

    `update_bn()` 假设数据加载器 `loader` 中的每个批次都是tensor或列表常量，其中第一个元素是网络 `swa_model` 应该应用到的tensor。如果您的数据加载器有不同的结构，您可以通过在数据集的每个元素上使用“swa_model”进行前向传递来更新“swa_model”的批量归一化统计信息。


### 将它们放在一起：SWA [¶](#putting-it-all-together-swa "永久链接到此标题")


 在下面的示例中，`swa_model` 是累积权重平均值的 SWA 模型。我们训练模型总共 300 个 epoch，然后切换到 SWA 学习率计划并开始收集参数的 SWA 平均值纪元 160：


```
>>> loader, optimizer, model, loss_fn = ...
>>> swa_model = torch.optim.swa_utils.AveragedModel(model)
>>> scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=300)
>>> swa_start = 160
>>> swa_scheduler = SWALR(optimizer, swa_lr=0.05)
>>>
>>> for epoch in range(300):
>>>       for input, target in loader:
>>>           optimizer.zero_grad()
>>>           loss_fn(model(input), target).backward()
>>>           optimizer.step()
>>>       if epoch > swa_start:
>>>           swa_model.update_parameters(model)
>>>           swa_scheduler.step()
>>>       else:
>>>           scheduler.step()
>>>
>>> # Update bn statistics for the swa_model at the end
>>> torch.optim.swa_utils.update_bn(loader, swa_model)
>>> # Use swa_model to make predictions on test data
>>> preds = swa_model(test_input)

```


### 将它们放在一起：EMA [¶](#putting-it-all-together-ema "永久链接到此标题")


 在下面的示例中，“ema_model”是 EMA 模型，它累积权重的指数衰减平均值，衰减率为 0.999。我们总共训练模型 300 个周期，并立即开始收集 EMA 平均值。


```
>>> loader, optimizer, model, loss_fn = ...
>>> ema_model = torch.optim.swa_utils.AveragedModel(model, >>>             multi_avg_fn=torch.optim.swa_utils.get_ema_multi_avg_fn(0.999))
>>>
>>> for epoch in range(300):
>>>       for input, target in loader:
>>>           optimizer.zero_grad()
>>>           loss_fn(model(input), target).backward()
>>>           optimizer.step()
>>>           ema_model.update_parameters(model)
>>>
>>> # Update bn statistics for the ema_model at the end
>>> torch.optim.swa_utils.update_bn(loader, ema_model)
>>> # Use ema_model to make predictions on test data
>>> preds = ema_model(test_input)

```