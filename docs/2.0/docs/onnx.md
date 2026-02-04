# torch.onnx [¶](#torch-onnx "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/onnx>
>
> 原始地址：<https://pytorch.org/docs/stable/onnx.html>


## 概述 [¶](#overview "此标题的永久链接")


[开放神经网络交换 (ONNX)](https://onnx.ai/) 是一种用于表示机器学习模型的开放标准格式。 `torch.onnx` 模块从原生 PyTorch [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") 模型捕获计算图并转换将其转换为 [ONNX 图表](https://github.com/onnx/onnx/blob/main/docs/IR.md) 。


 导出的模型可以由任何支持 ONNX 的运行时使用(https://onnx.ai/supported-tools.html#deployModel)，包括 Microsoft 的 [ONNX 运行时](https://www.onnxruntime.ai )。


**您可以使用两种类型的 ONNX 导出器 API，如下所列：**


## 基于 TorchDynamo 的 ONNX 导出器 [¶](#torchdynamo-based-onnx-exporter "永久链接到此标题")


*基于 TorchDynamo 的 ONNX 导出器是 PyTorch 2.0 及更高版本的最新(和测试版)导出器*


 TorchDynamo 引擎用于连接 Python 的框架评估 API 并将其字节码动态重写为 FX 图表。然后对生成的 FX 图进行完善，然后最终将其转换为 ONNX 图。


 这种方法的主要优点是使用字节码分析捕获 [FX 图](https://pytorch.org/docs/stable/fx.html)，保留模型的动态特性，而不是使用传统的静态跟踪技术。


[了解有关基于 TorchDynamo 的 ONNX 导出器的更多信息](onnx_dynamo.html)


## 基于 TorchScript 的 ONNX 导出器 [¶](#torchscript-based-onnx-exporter "此标题的永久链接") 


*基于 TorchScript 的 ONNX 导出器自 PyTorch 1.2.0 起可用*


[TorchScript](https://pytorch.org/docs/stable/jit.html) 用于跟踪(通过 [`torch.jit.trace()`](generated/torch.jit.trace.html#torch. jit.trace "torch.jit.trace") ) 模型并捕获静态计算图。


 因此，生成的图表有一些限制：



* 它不记录任何控制流，如 if 语句或循环；
* 不处理“training”和“eval”模式之间的细微差别；
* 不真正处理动态输入


 为了尝试支持静态跟踪限制，导出器还支持 TorchScript 脚本(通过 [`torch.jit.script()`](generated/torch.jit.script.html#torch.jit.script "torch.jit. script") )，例如，它增加了对数据相关控制流的支持。然而，TorchScript 本身是 Python 语言的子集，因此并不支持 Python 中的所有功能，例如就地操作。


[了解有关基于 TorchScript 的 ONNX 导出器的更多信息](onnx_torchscript.html)


## 贡献/开发 [¶](#contributing-developmenting "永久链接到此标题")


 ONNX 导出器是一个社区项目，我们欢迎贡献。我们遵循 [PyTorch 贡献指南](https://github.com/pytorch/pytorch/blob/main/CONTRIBUTING.md)，但您可能也有兴趣阅读我们的[开发维基](https://github.com/pytorch/blob/main/CONTRIBUTING.md)。 com/pytorch/pytorch/wiki/PyTorch-ONNX-exporter) 。