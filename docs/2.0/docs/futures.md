# torch.futures [¶](#torch-futures "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/futures>
>
> 原始地址：<https://pytorch.org/docs/stable/futures.html>


 该包提供了一个 [`Future`](#torch.futures.Future "torch.futures.Future") 类型，它封装了异步执行和一组实用函数来简化 [`Future`](#torch.futures.Future 上的操作“torch.futures.Future”)对象。目前，[`Future`](#torch.futures.Future "torch.futures.Future") 类型主要由[分布式 RPC 框架](rpc.html#distributed-rpc-framework) 使用。


*班级*


 火炬期货。


 未来


 ( *** ， *设备



 =
 


 无
* ) [¶](#torch.futures.Future "此定义的永久链接")


 围绕“torch._C.Future”的包装，封装了可​​调用的异步执行，例如[`rpc_async()`](rpc.html#torch.distributed.rpc.rpc_async "torch.distributed.rpc.rpc_async") 。它还公开了一组 API 来添加回调函数和设置结果。


!!! warning "警告"

     GPU 支持是测试版功能，可能会发生变化。


 添加_done_callback


 ( *callback
* ) [[source]](_modules/torch/futures.html#Future.add_done_callback)[¶](#torch.futures.Future.add_done_callback "此定义的永久链接")


 将给定的回调函数附加到此“Future”，该函数将在“Future”完成时运行。可以将多个回调添加到同一个 `Future` 中，但无法保证它们的执行顺序。回调必须采用一个参数，即对此 `Future` 的引用。回调函数可以使用 [`value()`](#torch.futures.Future.value "torch.futures.Future.value") 方法来获取值。请注意，如果此“Future”已经完成，则给定的回调将内联运行。


 我们建议您使用 [`then()`](#torch.futures.Future.then "torch.futures.Future.then") 方法，因为它提供了一种在回调完成后进行同步的方法。如果您的回调不返回任何内容，`add_done_callback` 可能会更便宜。但是 [`then()`](#torch.futures.Future.then "torch.futures.Future.then") 和 `add_done_callback` 在底层都使用相同的回调注册 API。


 对于 GPU tensor，此方法的行为方式与 [`then()`](#torch.futures.Future.then "torch.futures.Future.then") 相同。


 Parameters


**callback** ( `Future` ) – 一个接受一个参数的 `Callable` ，它是对此 `Future` 的引用。



!!! note "笔记"

    请注意，如果回调函数抛出异常，无论是通过原始 future 完成异常并调用“fut.wait()”，还是通过回调中的其他代码，都必须仔细处理错误处理。例如，如果此回调稍后完成其他 future，则这些 future 不会被标记为已完成并出现错误，并且用户负责独立处理这些 future 的完成/等待。


 例子：：




```
>>> def callback(fut):
...     print("This will run after the future has finished.")
...     print(fut.wait())
>>> fut = torch.futures.Future()
>>> fut.add_done_callback(callback)
>>> fut.set_result(5)
This will run after the future has finished.
5

```


 done
 


 ( ) [[source]](_modules/torch/futures.html#Future.done)[¶](#torch.futures.Future.done "此定义的永久链接")


 如果“Future”完成，则返回“True”。如果“Future”有结果或异常，则“Future”完成。


 如果该值包含驻留在 GPU 上的tensor，即使填充这些tensor的异步内核尚未在设备上完成运行，“Future.done()”也会返回“True”，因为在这个阶段结果已经可用，前提是执行适当的同步(请参阅 [`wait()`](#torch.futures.Future.wait "torch.futures.Future.wait") )。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 设置_异常


 ( *结果
* ) [[source]](_modules/torch/futures.html#Future.set_exception)[¶](#torch.futures.Future.set_exception "此定义的永久链接")


 为此“Future”设置一个例外，这将将此“Future”标记为已完成并出现错误，并触发所有附加的回调。请注意，当在此 `Future` 上调用 wait()/value() 时，此处设置的异常将内联引发。


 Parameters


**结果** ( [*BaseException*](https://docs.python.org/3/library/exceptions.html#BaseException "(in Python v3.12)") ) – 这个 `Future` 的异常。


 例子：：




```
>>> fut = torch.futures.Future()
>>> fut.set_exception(ValueError("foo"))
>>> fut.wait()
Traceback (most recent call last):
...
ValueError: foo

```


 设置_结果


 ( *result
* ) [[source]](_modules/torch/futures.html#Future.set_result)[¶](#torch.futures.Future.set_result "此定义的永久链接")


 设置此“Future”的结果，这将将此“Future”标记为已完成并触发所有附加的回调。请注意，“Future”不能被标记为已完成两次。


 如果结果包含驻留在 GPU 上的tensor，则即使填充这些tensor的异步内核尚未在设备上完成运行，也可以调用此方法，前提是在此方法时将这些内核排队的流设置为当前流叫做。简而言之，在启动这些内核后立即调用此方法是安全的，无需任何额外的同步，只要中间不更改流即可。此方法将记录所有相关当前流上的事件，并使用它们来确保此“Future”的所有消费者的正确调度。


 Parameters


**result** ( [*object*](https://docs.python.org/3/library/functions.html#object "(in Python v3.12)") ) – 这个 `Future 的结果对象`。


 例子：：




```
>>> import threading
>>> import time
>>> def slow_set_future(fut, value):
...     time.sleep(0.5)
...     fut.set_result(value)
>>> fut = torch.futures.Future()
>>> t = threading.Thread(
...     target=slow_set_future,
...     args=(fut, torch.ones(2) * 3)
... )
>>> t.start()
>>> print(fut.wait())
tensor([3., 3.])
>>> t.join()

```


 then
 


 ( *callback
* ) [[source]](_modules/torch/futures.html#Future.then)[¶](#torch.futures.Future.then "此定义的永久链接")


 将给定的回调函数附加到此“Future”，该函数将在“Future”完成时运行。可以将多个回调添加到同一个 `Future` 中，但无法保证它们的执行顺序(要强制执行特定顺序，请考虑链接： `fut.then(cb1).then(cb2)` )。回调必须采用一个参数，即对此 `Future` 的引用。回调函数可以使用 [`value()`](#torch.futures.Future.value "torch.futures.Future.value") 方法来获取值。请注意，如果此“Future”已经完成，则给定的回调将立即内联运行。


 如果“Future”的值包含驻留在 GPU 上的tensor，则可能会在填充这些tensor的异步内核尚未在设备上完成执行时调用回调。但是，回调将通过设置为当前(从全局池中获取)的一些专用流来调用，这些流将与那些内核同步。因此，回调对这些tensor执行的任何操作都将在内核完成后安排在设备上。换句话说，只要回调不切换流，它就可以安全地操作结果，而无需任何额外的同步。这类似于 [`wait()`](#torch.futures.Future.wait "torch.futures.Future.wait") 的非阻塞行为。


 同样，如果回调返回一个包含驻留在 GPU 上的tensor的值，那么即使生成这些tensor的内核仍在设备上运行，只要回调在执行期间没有更改流，它也可以这样做。如果想要更改流，则必须小心地将它们与原始流(即调用回调时当前的流)重新同步。


 Parameters


**callback** ( `Callable` ) – 一个以 `Future` 作为唯一参数的 `Callable`。


 退货


 一个新的“Future”对象，它保存“callback”的返回值，并在给定的“callback”完成时被标记为已完成。


 Return type


[*Future*](#torch.futures.Future "torch.jit.Future") [ *S
* ]



!!! note "笔记"

    请注意，如果回调函数抛出异常，无论是通过原始 future 完成异常并调用 `fut.wait()` ，还是通过回调中的其他代码，`then` 返回的 future 将被适当地标记为遇到的错误。但是，如果此回调稍后完成其他 future，这些 future 不会被标记为已完成并出现错误，并且用户负责独立处理这些 future 的完成/等待。


 例子：：




```
>>> def callback(fut):
...     print(f"RPC return value is {fut.wait()}.")
>>> fut = torch.futures.Future()
>>> # The inserted callback will print the return value when
>>> # receiving the response from "worker1"
>>> cb_fut = fut.then(callback)
>>> chain_cb_fut = cb_fut.then(
...     lambda x : print(f"Chained cb done. {x.wait()}")
... )
>>> fut.set_result(5)
RPC return value is 5.
Chained cb done. None

```


 value
 


 ( ) [[source]](_modules/torch/futures.html#Future.value)[¶](#torch.futures.Future.value "此定义的永久链接")


 获得已经完成的未来的价值。


 仅应在调用 [`wait()`](#torch.futures.Future.wait "torch.futures.Future.wait") 完成后调用此方法，或者在传递给 [`then() 的回调函数内调用`](#torch.futures.Future.then "torch.futures.Future.then") 。在其他情况下，这个“Future”可能尚未保存值，并且调用“value()”可能会失败。


 如果该值包含驻留在 GPU 上的tensor，则此方法将“不”执行任何额外的同步。这应该预先通过调用 [`wait()`](#torch.futures.Future.wait "torch.futures.Future.wait") 单独完成(除了回调内，它已经由[`then()`](#torch.futures.Future.then "torch.futures.Future.then") )。


 退货


 这个 `Future` 所持有的价值。如果创建值的函数(回调或 RPC)抛出错误，则此“value()”方法也会抛出错误。


 Return type


*T* 


 wait
 


 ( ) [[source]](_modules/torch/futures.html#Future.wait)[¶](#torch.futures.Future.wait "此定义的永久链接")


 阻塞直到这个 `Future` 的值准备好。


 如果该值包含驻留在 GPU 上的tensor，则与可能异步填充这些tensor的内核(在设备上执行)执行额外的同步。这种同步是非阻塞的，这意味着“wait()”将在当前流中插入必要的指令，以确保在这些流上排队的进一步操作将在异步内核之后正确调度，但是一旦完成，“wait()”将返回，即使那些内核仍在运行。只要不更改流，访问和使用值时就不需要进一步同步。


 退货


 这个 `Future` 所持有的价值。如果创建值的函数(回调或 RPC)抛出错误，则此“wait”方法也会抛出错误。


 Return type


*T* 


 火炬期货。


 收集所有


 ( *futures
* ) [[source]](_modules/torch/futures.html#collect_all)[¶](#torch.futures.collect_all "此定义的永久链接")


 将提供的 [`Future`](#torch.futures.Future "torch.futures.Future") 对象收集到已完成的单个组合 [`Future`](#torch.futures.Future "torch.futures.Future") 中当所有子期货完成时。


 Parameters


**futures** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)") ) – [`Future` 的列表](#torch.futures.Future "torch.futures.Future") 对象。


 退货


 将 [`Future`](#torch.futures.Future "torch.futures.Future") 对象返回到传入的 Future 列表中。


 Return type


[*Future*](#torch.futures.Future "torch.jit.Future") [ [*List*](https://docs.python.org/3/library/typing.html#typing.List "(在 Python v3.12)") [ [*Future*](#torch.futures.Future "torch.jit.Future") ]]


 例子：：




```
>>> fut0 = torch.futures.Future()
>>> fut1 = torch.futures.Future()
>>> fut = torch.futures.collect_all([fut0, fut1])
>>> fut0.set_result(0)
>>> fut1.set_result(1)
>>> fut_list = fut.wait()
>>> print(f"fut0 result = {fut_list[0].wait()}")
fut0 result = 0
>>> print(f"fut1 result = {fut_list[1].wait()}")
fut1 result = 1

```


 火炬期货。


 等待_all


 ( *futures
* ) [[source]](_modules/torch/futures.html#wait_all)[¶](#torch.futures.wait_all "此定义的永久链接")


 等待所有提供的 future 完成，并返回已完成值的列表。如果任何一个 future 遇到错误，该方法将提前退出并报告错误，而不等待其他 future 完成。


 Parameters


**futures** ( [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)") ) – [`Future` 的列表](#torch.futures.Future "torch.futures.Future") 对象。


 退货


 已完成的 [`Future`](#torch.futures.Future "torch.futures.Future") 结果列表。如果任何 [`Future`](#torch.futures.Future "torch.futures.Future") 上的 `wait` 抛出，此方法将抛出错误。


 Return type


[*列表*](https://docs.python.org/3/library/typing.html#typing.List“(在Python v3.12中)”)