# torch.utils.mobile_optimizer [¶](#torch-utils-mobile-optimizer "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/mobile_optimizer>
>
> 原始地址：<https://pytorch.org/docs/stable/mobile_optimizer.html>


!!! warning "警告"

     此 API 处于测试阶段，可能会在不久的将来发生变化。


 Torch mobile 支持 `torch.utils.mobile_optimizer.optimize_for_mobile` 实用程序以 eval 模式运行模块的优化传递列表。该方法采用以下参数：torch.jit.ScriptModule 对象、块列表优化set、保留的方法列表和后端。


 对于 CPU 后端，默认情况下，如果优化块列表为 None 或空，则 `optimize_for_mobile` 将运行以下优化： 
* **Conv2D 
+ BatchNorm fusion** (块列表选项 mobile_optimizer.MobileOptimizerType.CONV_BN\ _FUSION )：此优化过程将此模块及其所有子模块的“forward”方法中的“Conv2d-BatchNorm2d”折叠为“Conv2d”。 `Conv2d` 的权重和偏差相应更新。
* **插入和折叠预打包操作**(阻止列表选项 mobile_optimizer.MobileOptimizerType.INSERT_FOLD_PREPACK_OPS )：此优化过程重写图形以替换 2D卷积和线性运算及其预先打包的对应项。预打包操作是有状态操作，因为它们需要创建一些状态，例如权重预打包，并在操作执行期间使用此状态，即预打包权重。 XNNPACK 就是这样的后端之一，它提供预打包的操作，其内核针对移动平台(例如 ARM CPU)进行了优化。权重的预打包可以实现高效的内存访问，从而加快内核执行速度。目前，“optimize_for_mobile”通道重写了图表，将“Conv2D/Linear”替换为 1) 为 XNNPACK conv2d/线性操作预打包权重的操作和 2) 将预包装权重和激活作为输入的操作并生成输出激活。由于 1 只需要执行一次，因此我们折叠权重预打包，以便在模型加载时只执行一次。 `optimize_for_mobile` 的这一遍执行 1 和 2，然后折叠，即删除预打包操作的权重。
* **ReLU/Hardtanh 融合**：XNNPACK 操作支持钳位融合。也就是说，输出激活的钳位是作为内核的一部分完成的，包括 2D 卷积和线性运算内核。因此，夹紧是免费的。因此，任何可以表示为钳位操作的操作，例如“ReLU”或“hardtanh”，都可以与 XNNPACK 中先前的“Conv2D”或“线性”操作融合。此通道通过查找上一个通道写入的 XNNPACK `Conv2D/线性` 操作之后的 `ReLU/hardtanh` 操作来重写图表，并将它们融合在一起。
* **丢弃删除**(阻止列表选项 mobile_optimizer.MobileOptimizerType.REMOVE _DROPOUT )：当训练为 false 时，此优化过程会从此模块中删除 `dropout` 和 `dropout_` 节点。
* **Conv 打包参数提升**(阻止列表选项 mobile_optimizer.MobileOptimizerType.HOIST_CONV_PACKED\ _PARAMS )：此优化过程将卷积打包参数移动到根模块，以便可以删除卷积结构。这会减小模型大小而不影响数值。
* **Add/ReLU fusion** (阻止列表选项 mobile_optimizer.MobileOptimizerType.FUSE_ADD_RELU )：此过程查找遵循 `add` 操作和融合的 `relu` 操作的实例将它们放入单个 `add_relu` 中。


 对于 Vulkan 后端，默认情况下，如果优化阻止列表为 None 或空，则 `optimize_for_mobile` 将运行以下优化： 
* **自动 GPU 传输**(阻止列表选项 mobile_optimizer.MobileOptimizerType.VULKAN_AUTOMATIC_GPU _TRANSFER )：此优化过程重写图形，以便将输入和输出数据移入和移出 GPU 成为模型的一部分。


`optimize_for_mobile` 还将调用 freeze_module pass，它只保留 `forward` 方法。如果您还有其他需要保留的方法，请将它们添加到保留的方法列表中并传递到该方法中。


 torch.utils.mobile_optimizer。


 针对移动设备进行优化


 ( *script_module
* , *optimization_blocklist



 =
 


 无*，*保留_方法



 =
 


 无
* , *后端



 =
 


 'CPU'
* ) [[source]](_modules/torch/utils/mobile_optimizer.html#optimize_for_mobile)[¶](#torch.utils.mobile_optimizer.optimize_for_mobile "此定义的永久链接")


 参数 
* **script_module** ( [*ScriptModule*](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit._script.ScriptModule") ) – torch 脚本模块的实例ScriptModule 的类型。
* **optimization_blocklist** ( [*Optional*](https://docs.python.org/3/library/typing.html#typing.Optional "(in Python v3.12)") *[
* [*Set*](https://docs.python.org/3/library/typing.html#typing.Set "(在 Python v3.12 中)")*[
* *_MobileOptimizerType
* *]
* *]
* ) – 类型为 MobileOptimizerType 的集合。当set不通过时，优化方法将运行所有优化器pass；否则，optimizermethod 将运行优化_blocklist 中未包含的优化过程。
* **preserved_methods** ( [*Optional*](https://docs.python.org/3/library/typing.html# Typing.Optional "(Python v3.12 中)")*[
* [*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12 中) )")*]
* ) – 调用 freeze_module pass 时需要保留的方法列表
* **后端** ( [*str*](https://docs.python.org/3/library) /stdtypes.html#str "(Python v3.12)") ) – 用于运行结果模型的设备类型('CPU'(默认)、'Vulkan' 或 'Metal')。


 退货


 新优化的 Torch 脚本模块


 Return type


*递归脚本模块*