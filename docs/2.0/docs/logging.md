# torch._logging [¶](#torch-logging "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/logging>
>
> 原始地址：<https://pytorch.org/docs/stable/logging.html>


 PyTorch 有一个可配置的日志系统，可以为不同的组件提供不同的日志级别设置。例如，一个组件的日志消息可以完全禁用，而另一个组件的日志消息可以设置为最大详细程度。


!!! warning "警告"

     此功能是一个原型，将来可能会出现兼容性重大变化。


!!! warning "警告"

     此功能尚未扩展为控制 PyTorch 中所有组件的日志消息。


 配置日志系统有两种方法：通过环境变量`TORCH_LOGS`或python API torch._logging.set_logs。


|  |  |
| --- | --- |
| [`set_logs`](generated/torch._logging.set_logs.html#torch._logging.set_logs "torch._logging.set_logs") |设置各个组件的日志级别并切换各个日志工件类型。 |


 环境变量“TORCH_LOGS”是一个以逗号分隔的“[+-]<组件>”对列表，其中“<组件>”是下面指定的组件。 “+”前缀将降低组件的日志级别，显示更多的日志消息，而“-”前缀将提高组件的日志级别并显示更少的日志消息。默认设置是未在“TORCH_LOGS”中指定组件时的行为。除了组件之外，还有工件。工件是与显示或不显示的组件关联的特定调试信息，因此在工件前添加“+”或“-”前缀将是无操作。由于它们与组件关联，因此启用该组件通常也会启用该工件，除非将该工件指定为 off_by_default 。此选项在 _registrations.py 中指定，用于那些垃圾邮件过多的工件，只有在显式启用时才应显示。以下组件和工件可通过 `TORCH_LOGS` 环境变量进行配置(请参阅 torch._logging.set_logs 来了解python API)：


 成分：



`all`


 特殊组件，配置所有组件的默认日志级别。默认值：`logging.WARN`


`发电机`


 TorchDynamo 组件的日志级别。默认值：`logging.WARN`


`aot`


 AOTAutograd 组件的日志级别。默认值：`logging.WARN`


`电感器`


 TorchInductor 组件的日志级别。默认值：`logging.WARN`


`你的.custom.module`


 任意未注册模块的日志级别。提供完全限定名称，模块将被启用。默认值：`logging.WARN`


 文物：


`字节码`


 是否从 TorchDynamo 发出原始和生成的字节码。默认：`False`


`aot_graphs`


 是否发出 AOTAutograd 生成的图表。默认值：“假”


`aot_joint_graph`


 是否发出 AOTAutograd 生成的联合前向-后向图。默认值：“假”


`ddp_graphs`


 是否发出 DDPOptimizer 生成的图表。默认值：“假”


`图`


 是否以表格格式发出 TorchDynamo 捕获的图表。默认值：`False`


`图_代码`


 是否发出 TorchDynamo 捕获的图的 python 源。默认: `False`


`图_breaks`


 在 TorchDynamo 跟踪期间遇到唯一的图形中断时是否发出消息。默认值：“假”


‘守卫’


 是否发出 TorchDynamo 为每个编译函数生成的防护。默认值：“假”


`重新编译`


 每次 TorchDynamo 重新编译函数时是否发出防护失败原因和消息。默认值：“假”


`输出_代码`


 是否发出 TorchInductor 输出代码。默认值：“假”


`时间表`


 是否发出 TorchInductor 时间表。默认值：“假”


 例子：


`TORCH_LOGS="+dynamo,aot"` 将把 TorchDynamo 的日志级别设置为 `logging.DEBUG`，将 AOT 设置为 `logging.INFO`


`TORCH_LOGS="-dynamo,+inductor"` 将把 TorchDynamo 的日志级别设置为 `logging.ERROR`，将 TorchInductor 的日志级别设置为 `logging.DEBUG`


`TORCH_LOGS="aot_graphs"` 将启用 `aot_graphs` 工件


`TORCH_LOGS="+dynamo,schedule"` 将启用将 TorchDynamo 的日志级别设置为 `logging.DEBUG` 并启用 `schedule` 工件


`TORCH_LOGS="+some.random.module,schedule"` 会将 some.random.module 的日志级别设置为 `logging.DEBUG` 并启用 `schedule` 工件