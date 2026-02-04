# torch.utils.checkpoint [¶](#torch-utils-checkpoint "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/checkpoint>
>
> 原始地址：<https://pytorch.org/docs/stable/checkpoint.html>




!!! note "笔记"

    检查点是通过在向后过程中为每个设置检查点的段重新运行前向传递段来实现的。这可能会导致像 RNG 状态这样的持久状态比没有检查点时更先进。默认情况下，检查点包括处理 RNG 状态的逻辑，以便使用 RNG 的检查点传递(例如通过 dropout)与非检查点传递相比具有确定性输出。根据检查点操作的运行时间，存储和恢复 RNG 状态的逻辑可能会导致适度的性能影响。如果不需要与非检查点传递相比的确定性输出，请将“preserve_rng_state=False”提供给“checkpoint”或“checkpoint_sequential”，以在每个检查点期间省略存储和恢复 RNG 状态。


 存储逻辑保存并恢复 CPU 和另一个设备类型的 RNG 状态(通过 `_infer_device_type` 从 Tensor 参数推断设备类型，不包括 CPUtensors)到 `run_fn` 。如果有多个设备，则仅保存单一设备类型的设备的设备状态，其余设备将被忽略。因此，如果任何检查点函数涉及随机性，则可能会导致不正确的梯度。 (注意，如果检测到的设备中有 CUDA 设备，则优先考虑；否则，将选择第一个遇到的设备。)如果没有 CPU-tensors，则默认设备类型状态(默认值为 cuda ，可以设置为其他设备(通过 `DefaultDeviceType` )将被保存和恢复。但是，逻辑无法预测用户是否会将tensor移动到 `run_fn` 本身内的新设备。因此，如果您在 `run_fn` 中将tensor移动到新设备(“新”意味着不属于[当前设备 
+ tensor参数设备]集合)，则永远无法保证与非检查点传递相比的确定性输出。


 torch.utils.checkpoint。


 检查站


 ( *function
* , **args
* , *use_reentrant=None
* , *context_fn=<function noop_context_fn>
* , *确定性_check='default'
* , *debug=False
* , ***kwargs
* ) [[source]](_modules/torch/utils/checkpoint.html#checkpoint)[¶](#torch.utils.checkpoint.checkpoint "此定义的永久链接")


 检查模型或模型的一部分


 激活检查点是一种用计算换取内存的技术。检查点区域中的前向计算不会保留向后所需的tensor，直到在向后传递期间用于梯度计算，而是忽略保存向后的tensor并在向后传递期间重新计算它们。激活检查点可以应用于模型的任何部分。


 当前有两种可用的检查点实现，由“use_reentrant”参数确定。建议您使用 `use_reentrant=False` 。请参阅下面的注释来讨论它们的差异。


!!! warning "警告"

     如果后向传递期间的“函数”调用与前向传递不同，例如，由于全局变量，则检查点版本可能不等效，可能导致引发错误或导致默默地错误梯度。


!!! warning "警告"

     如果您使用 `use_reentrant=True` 变体(这是当前默认值)，请参阅下面的注释以了解重要注意事项和潜在限制。


!!! note "笔记"

    检查点的可重入变体 ( `use_reentrant=True` ) 和检查点的不可重入变体 ( `use_reentrant=False` ) 在以下方面有所不同：



* 一旦重新计算了所有需要的中间激活，不可重入检查点就会停止重新计算。该功能默认启用，但可以通过 set_checkpoint_early_stop()` 禁用。可重入检查点在向后传递期间始终重新计算整个“函数”。
* 可重入变体在前向传递期间不会记录自动分级图，因为它在 [`torch.no_grad()`](generated/torch.no_grad.html#torch.no_grad "torch.no_grad") 下运行前向传递。不可重入版本确实记录了 autograd 图，允许在检查点区域内的图上向后执行。
* 可重入检查点仅支持 [`torch.autograd.backward()`](generated/torch.autograd.backward.html# torch.autograd.backward "torch.autograd.backward") 用于向后传递的 API，不带输入参数，而不可重入版本支持执行向后传递的所有方式。
* 至少一个输入和输出必须具有 `requires_grad =True` 对于可重入变体。如果不满足此条件，模型的检查点部分将不会有梯度。不可重入版本没有此要求。
* 可重入版本不将嵌套结构(例如自定义对象、列表、字典等)中的tensor视为参与 autograd，而不可重入版本则这样做。
* 可重入检查点会考虑不支持与计算图中分离tensor的检查点区域，而不可重入版本则支持。对于可重入变体，如果检查点段包含使用 detach() 或 [`torch.no_grad()`](generated/torch.no_grad.html#torch.no_grad "torch.no_grad") 分离的tensor，则后向pass 会引发错误。这是因为“检查点”使所有输出都需要梯度，当tensor被定义为模型中没有梯度时，这会导致问题。为了避免这种情况，请将tensor分离到“checkpoint”函数之外。


 参数 
* **函数** – 描述在模型或部分模型的前向传递中运行的内容。它还应该知道如何处理作为元组传递的输入。例如，在 LSTM 中，如果用户传递 `(activation, hide)` ，则 `function` 应正确使用第一个输入作为 `activation`，第二个输入作为 `hidden`
* **preserve_rng_state** ( [
* bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 在每个检查点期间省略存储和恢复 RNG 状态.默认值：`True`
* **使用_reentrant** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") *,
* *可选
* ) – 使用需要可重入自动分级的检查点实现。如果指定了 `use_reentrant=False`，则 `checkpoint` 将使用不需要可重入自动分级的实现。这允许“checkpoint”支持附加功能，例如按预期与“torch.autograd.grad”一起工作，并支持输入到检查点函数中的关键字参数。请注意，PyTorch 的未来版本将默认为 `use_reentrant=False` 。默认值： `True`
* **context_fn** ( *Callable
* *,
* *可选
* ) – 返回两个上下文管理器元组的可调用对象。该函数及其重新计算将分别在第一个和第二个上下文管理器下运行。仅当 `use_reentrant=False` 时才支持此参数。
* **确定性_check** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* *可选
* ) – 指定要执行的确定性检查的字符串。默认情况下，它设置为“default”，它将重新计算的tensor与保存的tensor的形状、数据类型和设备进行比较。要关闭此检查，请指定“none”。目前，这是唯一受支持的两个值。如果您想查看更多确定性检查，请提出问题。仅当 `use_reentrant=False` 时才支持此参数，如果 `use_reentrant=True` ，则始终禁用确定性检查。
* **debug** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `True` ，错误消息还将包括在原始正向计算期间运行的运算符的跟踪以及重新计算。仅当 `use_reentrant=False` 时才支持此参数。
* **args** – 包含“函数”输入的元组


 退货


 在 `*args` 上运行 `function` 的输出


 torch.utils.checkpoint。


 检查点_顺序


 (*函数*、*段*、*输入*、*使用_可重入



 =
 


 真的
* ， ***


 kwargs
* ) [[source]](_modules/torch/utils/checkpoint.html#checkpoint_sequential)[¶](#torch.utils.checkpoint.checkpoint_sequential "此定义的永久链接")


 用于检查顺序模型的辅助函数。


 顺序模型按顺序(顺序)执行一系列模块/函数。因此，我们可以将这样的模型划分为各个部分，并对每个部分进行检查点。除了最后一个段之外的所有段都不会存储中间激活。每个检查点段的输入将被保存，以便在向后传递中重新运行该段。


!!! warning "警告"

     如果您使用 use_reentrant=True 变体(这是默认值)，请参阅 :func:`~torch.utils.checkpoint.checkpoint` 了解此变体的重要注意事项和限制。建议您使用“use_reentrant=False”。


 参数 
* **函数** – [`torch.nn.Sequential`](generated/torch.nn.Sequential.html#torch.nn.Sequential "torch.nn.Sequential") 或模块或函数列表(包括模型)按顺序运行。
* **segments** – 在模型中创建的块数
* **input** – 输入到 `functions`
* **preserve_rng_state** ( [ *bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 在每个过程中省略存储和恢复 RNG 状态checkpoint.Default: `True`
* **使用_reentrant** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)" )*,
* *可选
* ) – 使用需要可重入自动分级的检查点实现。如果指定了 `use_reentrant=False`，则 `checkpoint` 将使用不需要可重入自动分级的实现。这允许“checkpoint”支持附加功能，例如按预期与“torch.autograd.grad”一起工作，并支持输入到检查点函数中的关键字参数。默认值：“True”


 退货


 在“*inputs”上顺序运行“functions”的输出


 例子


```
>>> model = nn.Sequential(...)
>>> input_var = checkpoint_sequential(model, chunks, input_var)

```