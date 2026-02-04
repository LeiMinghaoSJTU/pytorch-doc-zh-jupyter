# torch.monitor [¶](#torch-monitor "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/monitor>
>
> 原始地址：<https://pytorch.org/docs/stable/monitor.html>


!!! warning "警告"

     该模块是一个原型版本，其接口和功能可能会在未来的 PyTorch 版本中发生变化，恕不另行通知。


`torch.monitor` 提供了一个用于记录 PyTorch 事件和计数器的接口。


 stat 接口设计用于跟踪高级指标，这些指标定期注销以用于监控系统性能。由于统计数据以特定的窗口大小聚合，因此您可以从关键循环中记录它们，而对性能的影响最小。


 对于更不常见的事件或值，例如丢失、准确性、使用情况跟踪，可以直接使用事件接口。


 可以注册事件处理程序来处理事件并将它们传递到外部事件接收器。


## API 参考 [¶](#module-torch.monitor "此标题的永久链接")


*班级*


 火炬.监视器。


 聚合 [¶](#torch.monitor.Aggregation"此定义的永久链接")



> 
> 
> 
> 这些是可用于累积统计数据的聚合类型。
> 
> 
> 
> >


 成员：



> 
> 
> 
> 
> VALUE :
> 
> 
> 
> VALUE 返回要添加的最后一个值。
> 
> 
> 
> 
> 
> MEAN :
> 
> 
> 
> MEAN 计算所有添加值的算术平均值。
> 
> 
> 
> 
> 
> COUNT :
> 
> 
> 
> COUNT 返回相加值的总数。
> 
> 
> 
> 
> 
> SUM :
> 
> 
> 
> SUM 返回相加值的总和。
> 
> 
> 
> 
> 
> MAX :
> 
> 
> 
> MAX 返回添加值的最大值。
> 
> 
> 
> 
> 
> MIN :
> 
> 
> 
> MIN 返回添加值的最小值。
> 
> 
> 
> 
> 
> >


*财产*


 name [¶](#torch.monitor.Aggregation.name "此定义的永久链接")


*班级*


 火炬.监视器。


 Stat [¶](#torch.monitor.Stat"此定义的永久链接")


 Stat 用于以超固定间隔的高性能方式计算摘要统计数据。 Stat 在每个“window_size”持续时间将统计信息记录为事件。当窗口关闭时，统计信息将通过事件处理程序记录为“torch.monitor.Stat”事件。


`window_size` 应设置为相对较高的值，以避免记录大量事件。例如：60 年代。 Stat 使用毫秒精度。


 如果设置了“max_samples”，则一旦发生“max_samples”添加，统计信息将通过丢弃添加调用来限制每个窗口的样本数量。如果未设置，则将包含窗口期间的所有“add”调用。这是一个可选字段，当样本数量可能变化时，可以使聚合在窗口之间更直接地进行比较。


 当统计数据被破坏时，即使窗口尚未过去，它也会记录所有剩余的数据。


 __在里面__


 ( *自己



 :
 


[火炬._C._monitor.Stat](#torch.monitor.Stat "火炬._C._monitor.Stat")
* , *名称



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")
* , *聚合



 :
 


 List
 


 [ [火炬._C._monitor.Aggregation](#torch.monitor.Aggregation "火炬._C._monitor.Aggregation")


 ]
* , *窗口_大小



 :
 


[datetime.timedelta](https://docs.python.org/3/library/datetime.html#datetime.timedelta "(在 Python v3.12 中)")
* , *max_samples



 :
 


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 =
 


 9223372036854775807*)


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.monitor.Stat.__init__ "此定义的永久链接")


 构造 `Stat` 。


 add
 


 ( *自己



 :
 


[火炬._C._monitor.Stat](#torch.monitor.Stat "火炬._C._monitor.Stat")
* , *v



 :
 


[float](https://docs.python.org/3/library/functions.html#float "(在 Python v3.12 中)")
* )


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.monitor.Stat.add"此定义的永久链接")


 根据配置的统计数据类型和聚合将值添加到要聚合的统计数据。


*财产*


 count [¶](#torch.monitor.Stat.count "此定义的永久链接")


 当前已收集的数据点数量。记录事件后重置。


 get
 


 ( *自己



 :
 


[火炬._C._monitor.Stat](#torch.monitor.Stat "火炬._C._monitor.Stat")
* )


 →
 


 Dict
 


 [ [火炬._C._monitor.Aggregation](#torch.monitor.Aggregation "火炬._C._monitor.Aggregation")


 ,
 


[float](https://docs.python.org/3/library/functions.html#float“(在Python v3.12中)”)


 ]
 


[¶](#torch.monitor.Stat.get"此定义的永久链接")


 返回统计数据的当前值，主要用于测试目的。如果已记录统计数据并且未添加其他值，则该值将为零。


*财产*


 name [¶](#torch.monitor.Stat.name "此定义的永久链接")


 创建期间设置的统计数据的名称。


*班级*


 火炬.监视器。


 data_value_t [¶](#torch.monitor.data_value_t "此定义的永久链接")


 data_value_t 是 `str` 、 `float` 、 `int` 、 `bool` 之一。


*班级*


 火炬.监视器。


 事件[¶](#torch.monitor.Event"此定义的永久链接")


 事件表示要记录的特定类型事件。这可以表示高级数据点，例如每个时期的损失或准确性，或更低级的聚合，例如通过该库提供的统计数据。


 同一类型的所有事件应具有相同的名称，以便下游处理程序可以正确处理它们。


 __在里面__


 ( *自己



 :
 


[torch._C._monitor.Event](#torch.monitor.Event "torch._C._monitor.Event")
* , *名称



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")
* , *时间戳



 :
 


[datetime.datetime](https://docs.python.org/3/library/datetime.html#datetime.datetime "(在 Python v3.12 中)")
* , *data



 :
 


 Dict
 


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[data_value_t](#torch.monitor.data_value_t "torch.monitor.data_value_t")


 ]*

 )
 


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.monitor.Event.__init__ "此定义的永久链接")


 构造 `Event` 。


*财产*


 data [¶](#torch.monitor.Event.data "此定义的永久链接")


 `Event` 中包含的结构化数据。


*财产*


 name [¶](#torch.monitor.Event.name "此定义的永久链接")


 `Event` 的名称。


*财产*


 时间戳 [¶](#torch.monitor.Event.timestamp "此定义的永久链接")


 “Event”发生时的时间戳。


*班级*


 火炬.监视器。


 EventHandlerHandle [¶](#torch.monitor.EventHandlerHandle "此定义的永久链接")


 EventHandlerHandle 是 `register_event_handler` 返回的包装类型，用于通过 `unregister_event_handler` 取消注册处理程序。不能直接初始化。


 火炬.监视器。


 记录_事件


 ( *事件



 :
 


[火炬._C._monitor.Event](#torch.monitor.Event "火炬._C._monitor.Event")
* )


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.monitor.log_event "此定义的永久链接")


 log_event 将指定的事件记录到所有已注册的事件处理程序。由事件处理程序将事件记录到相应的事件接收器。


 如果没有注册事件处理程序，则此方法是无操作的。


 火炬.监视器。


 注册_事件_处理程序


 ( *打回来



 :
 


 可调用


 [
 


 [ [火炬._C._monitor.Event](#torch.monitor.Event "火炬._C._monitor.Event")


 ]
 



 ,
 


[无](https://docs.python.org/3/library/constants.html#None“(Python v3.12)”)


 ]*

 )
 


 → [torch._C._monitor.EventHandlerHandle](#torch.monitor.EventHandlerHandle "torch._C._monitor.EventHandlerHandle")


[¶](#torch.monitor.register_event_handler "此定义的永久链接")


 register_event_handler 注册一个回调，每当通过 `log_event` 记录事件时调用。这些处理程序应避免阻塞主线程，因为这可能会干扰它们在“log_event”调用期间运行时的训练。


 火炬.监视器。


 取消注册_event_handler


 (*处理程序



 :
 


[torch._C._monitor.EventHandlerHandle](#torch.monitor.EventHandlerHandle "torch._C._monitor.EventHandlerHandle")
* )


 → [无](https://docs.python.org/3/library/constants.html#None "(Python v3.12)")


[¶](#torch.monitor.unregister_event_handler"此定义的永久链接")


 unregister_event_handler 取消注册调用 `register_event_handler` 之后返回的 `EventHandlerHandle` 。返回后，事件处理程序将不再接收事件。


*班级*


 火炬.监视器。


 TensorboardEventHandler


 ( *writer
* ) [[source]](_modules/torch/monitor.html#TensorboardEventHandler)[¶](#torch.monitor.TensorboardEventHandler "此定义的永久链接")


 TensorboardEventHandler 是一个事件处理程序，它将已知事件写入提供的 SummaryWriter。


 目前仅支持记录为标量的“torch.monitor.Stat”事件。


 例子


```
>>> from torch.utils.tensorboard import SummaryWriter
>>> from torch.monitor import TensorboardEventHandler, register_event_handler
>>> writer = SummaryWriter("log_dir")
>>> register_event_handler(TensorboardEventHandler(writer))

```


 __在里面__


 ( *writer
* ) [[source]](_modules/torch/monitor.html#TensorboardEventHandler.__init__)[¶](#torch.monitor.TensorboardEventHandler.__init__ "此定义的永久链接")


 构造 `TensorboardEventHandler` 。