# 量化 [¶](#module-torch.ao.quantization "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/quantization>
>
> 原始地址：<https://pytorch.org/docs/stable/quantization.html>


!!! warning "警告"

     量化处于测试阶段，可能会发生变化。


## 量化简介 [¶](#introduction-to-quantization "此标题的固定链接")


 量化是指以低于浮点精度的位宽执行计算和存储tensor的技术。量化模型以降低的精度而不是全精度(浮点)值对tensor执行部分或全部运算。这允许更紧凑的模型表示以及在许多硬件平台上使用高性能矢量化操作。与典型的 FP32 模型相比，PyTorch 支持 INT8 量化，允许模型大小减少 4 倍，内存带宽要求减少 4 倍。与 FP32 计算相比，对 INT8 计算的硬件支持通常快 2 到 4 倍。量化主要是一种加速推理的技术，量化算子仅支持前向传递。


 PyTorch 支持多种量化深度学习模型的方法。大多数情况下，模型在 FP32 中训练，然后模型转换为 INT8。此外，PyTorch 还支持量化感知训练，它使用假量化模块对前向和后向传递中的量化误差进行建模。注意，整个计算都是按浮点进行的。在量化感知训练结束时，PyTorch 提供转换函数，将训练后的模型转换为较低精度。


 在较低级别，PyTorch 提供了一种表示量化tensor并使用它们执行操作的方法。它们可用于直接构建以较低精度执行全部或部分计算的模型。提供了更高级别的 API，其中包含将 FP32 模型转换为较低精度的典型工作流程，并且精度损失最小。


## 量化 API 摘要 [¶](#quantization-api-summary "此标题的永久链接")


 PyTorch 提供两种不同的量化模式：Eager 模式量化和 FX Graph 模式量化。


 Eager 模式量化是测试版功能。用户需要进行融合并手动指定量化和反量化发生的位置，而且它仅支持模块而不支持函数。


 FX 图形模式量化是 PyTorch 中的一个新的自动量化框架，目前它是一个原型功能。它通过添加对函数的支持和自动化量化过程来改进 Eager 模式量化，尽管人们可能需要重构模型以使模型与 FX 图形模式量化兼容(通过“torch.fx”进行符号跟踪)。请注意，FX 图形模式量化预计不适用于任意模型，因为该模型可能无法符号追踪，我们会将其集成到 torchvision 等域库中，并且用户将能够使用 FX 量化与支持的域库中的模型类似的模型图模式量化。对于任意模型，我们将提供一般指南，但要真正使其发挥作用，用户可能需要熟悉“torch.fx”，特别是如何使模型以符号方式可追溯。


 鼓励量化的新用户首先尝试 FX 图形模式量化，如果不起作用，用户可以尝试遵循[使用 FX 图形模式量化](https://pytorch.org/tutorials/prototype/fx_graph_mode_quant_guide.html)或退回到急切模式量化。


 下表比较了 Eager 模式量化和 FX Graph 模式量化之间的差异：


|  |  |  |
| --- | --- | --- |
|  | 	 Eager Mode	Quantization	  | 	 FX Graph	Mode	Quantization	  |
| 	 Release	Status	  | 	 beta	  | 	 prototype	  |
| 	 Operator	Fusion	  | 	 Manual	  | 	 Automatic	  |
| 	 Quant/DeQuant	Placement	  | 	 Manual	  | 	 Automatic	  |
| 	 Quantizing	Modules	  | 	 Supported	  | 	 Supported	  |
| 	 Quantizing	Functionals/Torch	Ops	  | 	 Manual	  | 	 Automatic	  |
| 	 Support for	Customization	  | 	 Limited Support	  | 	 Fully	Supported	  |
| 	 Quantization Mode	Support	  | 	 Post Training	Quantization:	Static, Dynamic,	Weight Only	 		 Quantization Aware	Training:	Static	  | 	 Post Training	Quantization:	Static, Dynamic,	Weight Only	 		 Quantization Aware	Training:	Static	  |
| 	 Input/Output	Model Type	  | 	`torch.nn.Module`	 | 	`torch.nn.Module`	 (May need some	refactors to make	the model	compatible with FX	Graph Mode	Quantization)	  |


 支持三种类型的量化：


1. 动态量化(使用激活读取/存储的浮点进行量化的权重并进行量化以进行计算)2。静态量化(权重量化、激活量化、训练后需要校准)3。静态量化感知训练(权重量化、激活量化、训练期间建模的量化数值)


 请参阅我们的[PyTorch 量化简介](https://pytorch.org/blog/introduction-to-quantization-on-pytorch/) 博客文章，以更全面地概述这些量化类型之间的权衡。


 动态和静态量化之间的运算符覆盖范围有所不同，如下表所示。请注意，对于 FX 量化，还支持相应的泛函。


|  |  |  |
| --- | --- | --- |
|  | 	 Static	Quantization	  | 	 Dynamic	Quantization	  |
| 		 nn.Linear	 		 nn.Conv1d/2d/3d	 	 | 		 Y	 		 Y	 	 | 		 Y	 		 N	 	 |
| 		 nn.LSTM


 nn.GRU | Y(通过自定义模块)N |是


 是|
| nn.RNNCell nn.GRUCell nn.LSTMCell | N N N |是是是 |
| nn.EmbeddingBag | nn.EmbeddingBag | Y(激活在 fp32 中)|是|
| nn.嵌入 |是 |否|
| nn.MultiheadAttention | nn.MultiheadAttention | Y(通过自定义模块)|不支持 |
|激活 |广泛支持|不变，计算仍为 fp32 |


### Eager 模式量化 [¶](#eager-mode-quantization "此标题的永久链接")


 有关量化流程的一般介绍，包括不同类型的量化，请查看 [通用量化流程](#general-quantization-flow) 。


#### 训练后动态量化 [¶](#post-training-dynamic-quantization "此标题的永久链接")


 这是最简单的量化应用形式，其中权重提前量化，但激活在推理过程中动态量化。这用于模型执行时间主要由从内存加载权重而不是计算矩阵乘法的情况。对于小批量的 LSTM 和 Transformer 类型模型来说确实如此。


 图表：


```
# original model
# all tensors and computations are in floating point
previous_layer_fp32 -- linear_fp32 -- activation_fp32 -- next_layer_fp32
                 /
linear_weight_fp32

# dynamically quantized model
# linear and LSTM weights are in int8
previous_layer_fp32 -- linear_int8_w_fp32_inp -- activation_fp32 -- next_layer_fp32
                     /
   linear_weight_int8

```


 PTDQ API 示例：


```
import torch

# define a floating point model
class M(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.fc = torch.nn.Linear(4, 4)

    def forward(self, x):
        x = self.fc(x)
        return x

# create a model instance
model_fp32 = M()
# create a quantized model instance
model_int8 = torch.ao.quantization.quantize_dynamic(
    model_fp32,  # the original model
    {torch.nn.Linear},  # a set of layers to dynamically quantize
    dtype=torch.qint8)  # the target dtype for quantized weights

# run the model
input_fp32 = torch.randn(4, 4, 4, 4)
res = model_int8(input_fp32)

```


 要了解有关动态量化的更多信息，请参阅我们的[动态量化教程](https://pytorch.org/tutorials/recipes/recipes/dynamic_quantization.html)。


#### 训练后静态量化 [¶](#post-training-static-quantization "永久链接到此标题")


 训练后静态量化(PTQ static)量化模型的权重和激活。它尽可能将激活融合到前面的层中。它需要使用代表性数据集进行校准，以确定激活的最佳量化参数。当内存带宽和计算节省都很重要且 CNN 是非典型用例时，通常会使用训练后静态量化。


 在应用训练后静态量化之前，我们可能需要修改模型。请参阅[Eager 模式静态量化的模型准备](#model-preparation-for-eager-mode-static-quantization) 。


 图表：


```
# original model
# all tensors and computations are in floating point
previous_layer_fp32 -- linear_fp32 -- activation_fp32 -- next_layer_fp32
                    /
    linear_weight_fp32

# statically quantized model
# weights and activations are in int8
previous_layer_int8 -- linear_with_activation_int8 -- next_layer_int8
                    /
  linear_weight_int8

```


 PTSQ API 示例：


```
import torch

# define a floating point model where some layers could be statically quantized
class M(torch.nn.Module):
    def __init__(self):
        super().__init__()
        # QuantStub converts tensors from floating point to quantized
        self.quant = torch.ao.quantization.QuantStub()
        self.conv = torch.nn.Conv2d(1, 1, 1)
        self.relu = torch.nn.ReLU()
        # DeQuantStub converts tensors from quantized to floating point
        self.dequant = torch.ao.quantization.DeQuantStub()

    def forward(self, x):
        # manually specify where tensors will be converted from floating
        # point to quantized in the quantized model
        x = self.quant(x)
        x = self.conv(x)
        x = self.relu(x)
        # manually specify where tensors will be converted from quantized
        # to floating point in the quantized model
        x = self.dequant(x)
        return x

# create a model instance
model_fp32 = M()

# model must be set to eval mode for static quantization logic to work
model_fp32.eval()

# attach a global qconfig, which contains information about what kind
# of observers to attach. Use 'x86' for server inference and 'qnnpack'
# for mobile inference. Other quantization configurations such as selecting
# symmetric or asymmetric quantization and MinMax or L2Norm calibration techniques
# can be specified here.
# Note: the old 'fbgemm' is still available but 'x86' is the recommended default
# for server inference.
# model_fp32.qconfig = torch.ao.quantization.get_default_qconfig('fbgemm')
model_fp32.qconfig = torch.ao.quantization.get_default_qconfig('x86')

# Fuse the activations to preceding layers, where applicable.
# This needs to be done manually depending on the model architecture.
# Common fusions include `conv + relu` and `conv + batchnorm + relu`
model_fp32_fused = torch.ao.quantization.fuse_modules(model_fp32, [['conv', 'relu']])

# Prepare the model for static quantization. This inserts observers in
# the model that will observe activation tensors during calibration.
model_fp32_prepared = torch.ao.quantization.prepare(model_fp32_fused)

# calibrate the prepared model to determine quantization parameters for activations
# in a real world setting, the calibration would be done with a representative dataset
input_fp32 = torch.randn(4, 1, 4, 4)
model_fp32_prepared(input_fp32)

# Convert the observed model to a quantized model. This does several things:
# quantizes the weights, computes and stores the scale and bias value to be
# used with each activation tensor, and replaces key operators with quantized
# implementations.
model_int8 = torch.ao.quantization.convert(model_fp32_prepared)

# run the model, relevant calculations will happen in int8
res = model_int8(input_fp32)

```


 要了解有关静态量化的更多信息，请参阅[静态量化教程](https://pytorch.org/tutorials/advanced/static_quantization_tutorial.html)。


#### 静态量化的量化感知训练 [¶](#quantization-aware-training-for-static-quantization"此标题的永久链接")


 量化感知训练 (QAT) 对训练过程中的量化效果进行建模，与其他量化方法相比，具有更高的准确性。我们可以对静态、动态或仅权值量化进行 QAT。在训练期间，所有计算均以浮点形式完成，并使用 fake_quant 模块通过钳位和舍入来模拟量化效果，以模拟 INT8 的效果。模型转换后，权重和激活被量化，并且激活尽可能融合到前一层中。它通常与 CNN 一起使用，并且与静态量化相比具有更高的精度。


 在应用训练后静态量化之前，我们可能需要修改模型。请参阅[Eager 模式静态量化的模型准备](#model-preparation-for-eager-mode-static-quantization) 。


 图表：


```
# original model
# all tensors and computations are in floating point
previous_layer_fp32 -- linear_fp32 -- activation_fp32 -- next_layer_fp32
                      /
    linear_weight_fp32

# model with fake_quants for modeling quantization numerics during training
previous_layer_fp32 -- fq -- linear_fp32 -- activation_fp32 -- fq -- next_layer_fp32
                           /
   linear_weight_fp32 -- fq

# quantized model
# weights and activations are in int8
previous_layer_int8 -- linear_with_activation_int8 -- next_layer_int8
                     /
   linear_weight_int8

```


 QAT API 示例：


```
import torch

# define a floating point model where some layers could benefit from QAT
class M(torch.nn.Module):
    def __init__(self):
        super().__init__()
        # QuantStub converts tensors from floating point to quantized
        self.quant = torch.ao.quantization.QuantStub()
        self.conv = torch.nn.Conv2d(1, 1, 1)
        self.bn = torch.nn.BatchNorm2d(1)
        self.relu = torch.nn.ReLU()
        # DeQuantStub converts tensors from quantized to floating point
        self.dequant = torch.ao.quantization.DeQuantStub()

    def forward(self, x):
        x = self.quant(x)
        x = self.conv(x)
        x = self.bn(x)
        x = self.relu(x)
        x = self.dequant(x)
        return x

# create a model instance
model_fp32 = M()

# model must be set to eval for fusion to work
model_fp32.eval()

# attach a global qconfig, which contains information about what kind
# of observers to attach. Use 'x86' for server inference and 'qnnpack'
# for mobile inference. Other quantization configurations such as selecting
# symmetric or asymmetric quantization and MinMax or L2Norm calibration techniques
# can be specified here.
# Note: the old 'fbgemm' is still available but 'x86' is the recommended default
# for server inference.
# model_fp32.qconfig = torch.ao.quantization.get_default_qconfig('fbgemm')
model_fp32.qconfig = torch.ao.quantization.get_default_qat_qconfig('x86')

# fuse the activations to preceding layers, where applicable
# this needs to be done manually depending on the model architecture
model_fp32_fused = torch.ao.quantization.fuse_modules(model_fp32,
    [['conv', 'bn', 'relu']])

# Prepare the model for QAT. This inserts observers and fake_quants in
# the model needs to be set to train for QAT logic to work
# the model that will observe weight and activation tensors during calibration.
model_fp32_prepared = torch.ao.quantization.prepare_qat(model_fp32_fused.train())

# run the training loop (not shown)
training_loop(model_fp32_prepared)

# Convert the observed model to a quantized model. This does several things:
# quantizes the weights, computes and stores the scale and bias value to be
# used with each activation tensor, fuses modules where appropriate,
# and replaces key operators with quantized implementations.
model_fp32_prepared.eval()
model_int8 = torch.ao.quantization.convert(model_fp32_prepared)

# run the model, relevant calculations will happen in int8
res = model_int8(input_fp32)

```


 要了解有关量化感知训练的更多信息，请参阅 [QATtutorial](https://pytorch.org/tutorials/advanced/static_quantization_tutorial.html)。


#### Eager 模式静态量化的模型准备 [¶](#model-preparation-for-eager-mode-static-quantization"永久链接到此标题")


 目前有必要在 Eager 模式量化之前对模型定义进行一些修改。这是因为当前量化是在逐个模块的基础上进行的。具体来说，对于所有量化技术，用户需要：


1. 将任何需要输出重新量化(因此具有附加参数)的操作从泛函转换为模块形式(例如，使用 `torch.nn.ReLU` 而不是 `torch.nn.function.relu` )。2.通过在子模块上分配“.qconfig”属性或指定“qconfig_mapping”来指定模型的哪些部分需要量化。例如，设置“model.conv1.qconfig = None”意味着“model.conv”图层不会被量化，并且设置 `model.linear1.qconfig = custom_qconfig` 意味着 `model.linear1` 的量化设置将使用 `custom_qconfig` 而不是全局 qconfig。


 对于量化激活的静态量化技术，用户还需要执行以下操作：


1. 指定激活的量化和反量化位置。这是使用 [`QuantStub`](generated/torch.ao.quantization.QuantStub.html#torch.ao.quantization.QuantStub "torch.ao.quantization.QuantStub") 和 [`DeQuantStub`](generated/torch. ao.quantization.DeQuantStub.html#torch.ao.quantization.DeQuantStub "torch.ao.quantization.DeQuantStub") 模块.2.使用 [`FloatFunctional`](generated/torch.ao.nn.quantized.FloatFunctional.html#torch.ao.nn.quantized.FloatFunctional "torch.ao.nn.quantized.FloatFunctional") 包装需要特殊处理的tensor操作量化为模块。例如“add”和“cat”等操作，它们需要特殊处理来确定输出量化参数。3．熔断模块：将操作/模块组合成单个模块以获得更高的精度和性能。这是使用 [`fuse_modules()`](generated/torch.ao.quantization.fuse_modules.html#torch.ao.quantization.fuse_modules "torch.ao.quantization.fuse_modules") API 完成的，该 API 接受列表要融合的模块数。我们目前支持以下融合：[Conv, Relu]、[Conv, BatchNorm]、[Conv, BatchNorm, Relu]、[Linear, Relu]


###(原型)FX 图形模式量化 [¶](#prototype-fx-graph-mode-quantization"此标题的固定链接")


 训练后量化有多种量化类型(仅权重、动态和静态)，配置是通过 qconfig_mapping (prepare_fx 函数的参数)完成的。


 FXPTQ API 示例：


```
import torch
from torch.ao.quantization import (
  get_default_qconfig_mapping,
  get_default_qat_qconfig_mapping,
  QConfigMapping,
)
import torch.ao.quantization.quantize_fx as quantize_fx
import copy

model_fp = UserModel()

#
# post training dynamic/weight_only quantization
#

# we need to deepcopy if we still want to keep model_fp unchanged after quantization since quantization apis change the input model
model_to_quantize = copy.deepcopy(model_fp)
model_to_quantize.eval()
qconfig_mapping = QConfigMapping().set_global(torch.ao.quantization.default_dynamic_qconfig)
# a tuple of one or more example inputs are needed to trace the model
example_inputs = (input_fp32)
# prepare
model_prepared = quantize_fx.prepare_fx(model_to_quantize, qconfig_mapping, example_inputs)
# no calibration needed when we only have dynamic/weight_only quantization
# quantize
model_quantized = quantize_fx.convert_fx(model_prepared)

#
# post training static quantization
#

model_to_quantize = copy.deepcopy(model_fp)
qconfig_mapping = get_default_qconfig_mapping("qnnpack")
model_to_quantize.eval()
# prepare
model_prepared = quantize_fx.prepare_fx(model_to_quantize, qconfig_mapping, example_inputs)
# calibrate (not shown)
# quantize
model_quantized = quantize_fx.convert_fx(model_prepared)

#
# quantization aware training for static quantization
#

model_to_quantize = copy.deepcopy(model_fp)
qconfig_mapping = get_default_qat_qconfig_mapping("qnnpack")
model_to_quantize.train()
# prepare
model_prepared = quantize_fx.prepare_qat_fx(model_to_quantize, qconfig_mapping, example_inputs)
# training loop (not shown)
# quantize
model_quantized = quantize_fx.convert_fx(model_prepared)

#
# fusion
#
model_to_quantize = copy.deepcopy(model_fp)
model_fused = quantize_fx.fuse_fx(model_to_quantize)

```


 有关 FX 图形模式量化的更多信息，请参阅以下教程：



* [FX 图形模式量化使用用户指南](https://pytorch.org/tutorials/prototype/fx_graph_mode_quant_guide.html)
* [FX 图形模式训练后静态量化](https://pytorch.org/tutorials/prototype /fx_graph_mode_ptq_static.html)
* [FX 图形模式训练后动态量化](https://pytorch.org/tutorials/prototype/fx_graph_mode_ptq_dynamic.html)


## 量化堆栈 [¶](#quantization-stack "此标题的永久链接")


 量化是将浮点模型转换为量化模型的过程。因此，在高层次上，量化堆栈可以分为两部分：1)。量化模型的构建块或抽象 2)。将浮点模型转换为量化模型的量化流程的构建块或抽象


### 量化模型 [¶](#quantized-model "此标题的固定链接")


#### 量化tensor [¶](#quantized-tensor "此标题的固定链接")


 为了在 PyTorch 中进行量化，我们需要能够用tensor表示量化数据。量化tensor允许存储量化数据(表示为 int8/uint8/int32)以及尺度和零点等量化参数。量化tensor除了允许以量化格式序列化数据之外，还允许许多有用的操作，使量化算术变得容易。


 PyTorch 支持每tensor和每通道的对称和非对称量化。每个tensor意味着tensor内的所有值都使用相同的量化参数以相同的方式量化。每个通道意味着对于每个维度(通常是tensor的通道维度)，tensor中的值使用不同的量化参数进行量化。这可以减少将tensor转换为量化值时的错误，因为异常值只会影响其所在的通道，而不是整个tensor。


 映射是通过使用转换浮点tensor来执行的


[![_images/math-quantizer-equation.png](_images/math-quantizer-equation.png)](_images/math-quantizer-equation.png) 请注意，我们确保浮点数中的零表示为无量化后的误差，从而确保填充等操作不会导致额外的量化误差。


 以下是量化tensor的几个关键属性：



* QScheme (torch.qscheme)：一个枚举，指定我们量化tensor的方式



+ torch.per_tensor_affine 
+ torch.per_tensor_symmetry 
+ torch.per_channel_affine 
+ torch.per_channel_symmetry
* dtype (torch.dtype)：量化tensor的数据类型



+ torch.quint8 
+ torch.qint8 
+ torch.qint32 
+ torch.float16
* 量化参数(根据 QScheme 变化)：所选量化方式的参数



+ torch.per_tensor_affine 的量化参数为 
- scale (float) 
- 零点 (int) 
+ torch.per_channel_affine 的量化参数为 
- per_channel_scales (float 列表) -每个通道零点(整数列表)
- 轴(整数)


#### 量化和反量化 [¶](#quantize-and-dequantize "此标题的永久链接")


 模型的输入和输出都是浮点tensor，但量化模型中的激活是量化的，因此我们需要运算符在浮点和量化tensor之间进行转换。



* 量化(浮点 -
> 量化)



+ torch.quantize_per_tensor(x,scale,zero_point,dtype) 
+ torch.quantize_per_channel(x,scales,zero_points,axis,dtype) 
+ torch.quantize_per_tensor_dynamic (x, dtype, reduce_range) 
+ to(torch.float16)
* 反量化(量化 -
> float)



+ quantized_tensor.dequantize() 
- 在 torch.float16 tensor上调用 dequantize 会将tensor转换回 torch.float 
+ torch.dequantize(x)


#### 量化运算符/模块 [¶](#quantized-operators-modules "此标题的永久链接")



* 量化运算符是以量化tensor为输入并输出量化tensor的运算符。 
* 量化模块是执行量化运算的 PyTorch 模块。它们通常是为线性和卷积等加权运算定义的。


#### 量化引擎 [¶](#quantized-engine "此标题的永久链接")


 当执行量化模型时，qengine (torch.backends.quantized.engine) 指定使用哪个后端来执行。重要的是要确保qengine在量化激活和权重的取值范围方面与量化模型兼容。


### 量化流程 [¶](#quantization-flow "此标题的永久链接")


#### Observer 和 FakeQuantize [¶](#observer-and-fakequantize "永久链接到此标题")



* 观察者是 PyTorch 模块，用于：



+ 收集tensor统计数据，例如通过观察者的tensor的最小值和最大值 
+ 并根据收集的tensor统计数据计算量化参数
* FakeQuantize 是 PyTorch 模块，用于：



+ 模拟网络中tensor的量化(执行量化/反量化) 
+ 它可以根据观察者收集的统计数据计算量化参数，也可以学习量化参数


#### QConfig [¶](#qconfig "此标题的永久链接")



* QConfig 是 Observer 或 FakeQuantize Module 类的命名元组，可以使用 qscheme、dtype 等进行配置。它用于配置如何观察操作员



+ 运算符/模块的量化配置 
- 不同类型的 Observer/FakeQuantize 
- dtype 
- qscheme 
- quant_min/quant_max：可用于模拟较低精度的tensor +​​ 目前支持激活和权重的配置 
+ 我们插入输入/权重/基于为给定操作员或模块配置的 qconfig 的输出观察者


#### 通用量化流程 [¶](#general-quantization-flow "固定链接到此标题")


 一般来说，流程如下



* 准备



+ 根据用户指定的 qconfig
* 校准/训练插入 Observer/FakeQuantize 模块(取决于训练后量化或量化感知训练)



+ 允许观察者收集统计数据或 FakeQuantize 模块学习量化参数
* 转换



+ 将校准/训练模型转换为量化模型


 量化有不同的模式，它们可以分为两种方式：


 就我们应用量化流程的位置而言，我们有：


1.训练后量化(训练后应用量化，根据样本校准数据计算量化参数)2。量化感知训练(在训练过程中模拟量化，以便使用训练数据与模型一起学习量化参数)


 就我们如何量化运算符而言，我们可以：



* 仅权重量化(仅权重静态量化)
* 动态量化(权重静态量化，激活动态量化)
* 静态量化(权重和激活均静态量化)


 我们可以在同一量化流程中混合不同的量化运算符方式。例如，我们可以进行具有静态和动态量化运算符的训练后量化。


## 量化支持矩阵 [¶](#quantization-support-matrix "此标题的固定链接")


### 量化模式支持 [¶](#quantization-mode-support "固定链接到此标题")


|  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
|  | 	 Quantization	Mode	  | 	 Dataset	Requirement	  | 	 Works Best For	  | 	 Accuracy	  | 	 Notes	  |
| 	 Post Training Quantization	  | 	 Dynamic/Weight Only Quantization	  | 	 activation	dynamically	quantized (fp16,	int8) or not	quantized, weight	statically quantized	(fp16, int8, in4)	  | 	 None	  | 	 LSTM, MLP,	Embedding,	Transformer	  | 	 good	  | 	 Easy to use,	close to static	quantization when	performance is	compute or memory	bound due to	weights	  |
| 	 Static Quantization	  | 	 activation and	weights statically	quantized (int8)	  | 	 calibration	dataset	  | 	 CNN	  | 	 good	  | 	 Provides best	perf, may have	big impact on	accuracy, good	for hardwares	that only support	int8 computation	  |
| 	 Quantization Aware Training	  | 	 Dynamic Quantization	  | 	 activation and	weight are fake	quantized	  | 	 fine-tuning	dataset	  | 	 MLP, Embedding	  | 	 best	  | 	 Limited support	for now	  |
| 	 Static Quantization	  | 	 activation and	weight are fake	quantized	  | 	 fine-tuning	dataset	  | 	 CNN, MLP,	Embedding	  | 	 best	  | 	 Typically used	when static	quantization	leads to bad	accuracy, and	used to close the	accuracy gap	  |


 请参阅我们的[Pytorch 量化简介](https://pytorch.org/blog/introduction-to-quantization-on-pytorch/) 博客文章，以更全面地概述这些量化类型之间的权衡。


### 量化流支持 [¶](#quantization-flow-support "固定链接到此标题")


 PyTorch 提供两种量化模式：Eager 模式量化和 FX Graph 模式量化。


 Eager 模式量化是测试版功能。用户需要进行融合并手动指定量化和反量化发生的位置，而且它仅支持模块而不支持函数。


 FX 图形模式量化是 PyTorch 中的一个自动量化框架，目前它是一个原型功能。它通过添加对函数的支持和自动化量化过程来改进 Eager 模式量化，尽管人们可能需要重构模型以使模型与 FX 图形模式量化兼容(通过“torch.fx”进行符号跟踪)。请注意，FX 图形模式量化预计不适用于任意模型，因为该模型可能无法符号追踪，我们会将其集成到 torchvision 等域库中，并且用户将能够使用 FX 量化与支持的域库中的模型类似的模型图模式量化。对于任意模型，我们将提供一般指南，但要真正使其发挥作用，用户可能需要熟悉“torch.fx”，特别是如何使模型以符号方式可追溯。


 鼓励量化的新用户首先尝试 FX 图形模式量化，如果不起作用，用户可以尝试遵循[使用 FX 图形模式量化](https://pytorch.org/tutorials/prototype/fx_graph_mode_quant_guide.html)或退回到急切模式量化。


 下表比较了 Eager 模式量化和 FX Graph 模式量化之间的差异：


|  |  |  |
| --- | --- | --- |
|  | 	 Eager Mode	Quantization	  | 	 FX Graph	Mode	Quantization	  |
| 	 Release	Status	  | 	 beta	  | 	 prototype	  |
| 	 Operator	Fusion	  | 	 Manual	  | 	 Automatic	  |
| 	 Quant/DeQuant	Placement	  | 	 Manual	  | 	 Automatic	  |
| 	 Quantizing	Modules	  | 	 Supported	  | 	 Supported	  |
| 	 Quantizing	Functionals/Torch	Ops	  | 	 Manual	  | 	 Automatic	  |
| 	 Support for	Customization	  | 	 Limited Support	  | 	 Fully	Supported	  |
| 	 Quantization Mode	Support	  | 	 Post Training	Quantization:	Static, Dynamic,	Weight Only	 		 Quantization Aware	Training:	Static	  | 	 Post Training	Quantization:	Static, Dynamic,	Weight Only	 		 Quantization Aware	Training:	Static	  |
| 	 Input/Output	Model Type	  | 	`torch.nn.Module`	 | 	`torch.nn.Module`	 (May need some	refactors to make	the model	compatible with FX	Graph Mode	Quantization)	  |


### 后端/硬件支持 [¶](#backend-hardware-support "永久链接到此标题")


|  |  |  |  |  |
| --- | --- | --- | --- | --- |
| 	 Hardware	  | 	 Kernel Library	  | 	 Eager Mode	Quantization	  | 	 FX Graph	Mode	Quantization	  | 	 Quantization	Mode Support	  |
| 	 server CPU	  | 	 fbgemm/onednn	  | 	 Supported	  | 	 All	Supported	  |
| 	 mobile CPU	  | 	 qnnpack/xnnpack	  |
| 	 server GPU	  | 	 TensorRT (early	prototype)	  | 	 Not support	this it	requires a	graph	  | 	 Supported	  | 	 Static	Quantization	  |


 如今，PyTorch 支持以下后端来高效运行量化运算符：



* 支持 AVX2 或更高版本的 x86 CPU(没有 AVX2，某些操作的实现效率低下)，通过 [fbgemm](https://github.com/pytorch/FBGEMM) 和 [onednn](https://github.com) 优化的 x86 /oneapi-src/oneDNN)(请参阅 [RFC](https://github.com/pytorch/pytorch/issues/83888) 中的详细信息)
* ARM CPU(通常在移动/嵌入式设备中找到)，通过 [qnnpack] (https://github.com/pytorch/pytorch/tree/main/aten/src/ATen/native/quantized/cpu/qnnpack)*(早期原型)通过 [TensorRT](https://developer) 支持 NVidia GPU.nvidia.com/tensorrt)通过 fx2trt(将开源)


#### 本机 CPU 后端注意事项 [¶](#note-for-native-cpu-backends "永久链接到此标题")


 我们使用相同的本机 pytorch 量化运算符公开 x86 和 qnnpack，因此我们需要额外的标志来区分它们。 x86 和 qnnpack 的相应实现是根据 PyTorch 构建模式自动选择的，尽管用户可以选择通过将 torch.backends.quantization.engine 设置为 x86 或 qnnpack 来覆盖它。


 在准备量化模型时，需要确保qconfig和用于量化计算的引擎与执行模型的后端相匹配。 qconfig 控制量化过程中使用的观察者的类型。 qengine 控制在打包线性和卷积函数和模块的权重时是否使用 x86 或 qnnpack 特定的打包函数。例如：


 x86 的默认设置：


```
# set the qconfig for PTQ
# Note: the old 'fbgemm' is still available but 'x86' is the recommended default on x86 CPUs
qconfig = torch.ao.quantization.get_default_qconfig('x86')
# or, set the qconfig for QAT
qconfig = torch.ao.quantization.get_default_qat_qconfig('x86')
# set the qengine to control weight packing
torch.backends.quantized.engine = 'x86'

```


 qnnpack 的默认设置：


```
# set the qconfig for PTQ
qconfig = torch.ao.quantization.get_default_qconfig('qnnpack')
# or, set the qconfig for QAT
qconfig = torch.ao.quantization.get_default_qat_qconfig('qnnpack')
# set the qengine to control weight packing
torch.backends.quantized.engine = 'qnnpack'

```


### 操作员支持 [¶](#operator-support "永久链接到此标题")


 动态和静态量化之间的运算符覆盖范围有所不同，如下表所示。请注意，对于 FX 图形模式量化，还支持相应的泛函。


|  |  |  |
| --- | --- | --- |
|  | 	 Static	Quantization	  | 	 Dynamic	Quantization	  |
| 		 nn.Linear	 		 nn.Conv1d/2d/3d	 	 | 		 Y	 		 Y	 	 | 		 Y	 		 N	 	 |
| 		 nn.LSTM	 		 nn.GRU	 	 | 		 N	 		 N	 	 | 		 Y	 		 Y	 	 |
| 		 nn.RNNCell	 		 nn.GRUCell	 		 nn.LSTMCell	 	 | 		 N	 		 N	 		 N	 	 | 		 Y	 		 Y	 		 Y	 	 |
| 	 nn.EmbeddingBag	  | 	 Y (activations	are in fp32)	  | 	 Y	  |
| 	 nn.Embedding	  | 	 Y	  | 	 N	  |
| 	 nn.MultiheadAttention	  | 	 Not Supported	  | 	 Not supported	  |
| 	 Activations	  | 	 Broadly supported	  | 	 Un-changed,	computations	stay in fp32	  |


 注意：这将很快用本机后端 _config_dict 生成的一些信息进行更新。


## 量化 API 参考 [¶](#quantization-api-reference "此标题的永久链接")


 [量化 API 参考](quantization-support.html) 包含量化 API 的文档，例如量化通道、量化tensor运算以及支持的量化模块和函数。


## 量化后端配置 [¶](#quantization-backend-configuration "此标题的固定链接")


 [量化后端配置](quantization-backend-configuration.html) 包含有关如何为各种后端配置量化工作流程的文档。


## 量化精度调试 [¶](#quantization-accuracy-debugging "此标题的固定链接")


 [量化精度调试](quantization-accuracy-debugging.html) 包含有关如何调试量化精度的文档。


## 量化自定义 [¶](#quantization-customizations "此标题的永久链接")


 虽然提供了观察者基于观察到的tensor数据选择比例因子和偏差的默认实现，但开发人员可以提供自己的量化函数。量化可以选择性地应用于模型的不同部分，或者针对模型的不同部分进行不同的配置。


 我们还为 **conv1d()** 、 **conv2d()** 、 **conv3d()** 和 **linear()** 提供每通道量化支持。


 量化工作流程通过在模型的模块层次结构中添加(例如，将观察者添加为“.observer”子模块)或替换(例如，将“nn.Conv2d”转换为“nn.quantized.Conv2d”)子模块来工作。这意味着该模型在整个过程中保持常规的基于“nn.Module”的实例，因此可以与 PyTorch API 的其余部分一起使用。


### 量化自定义模块 API [¶](#quantization-custom-module-api "此标题的永久链接")


 Eager 模式和 FX 图形模式量化 API 都为用户提供了一个钩子，用于指定以自定义方式量化的模块，并使用用户定义的观察和量化逻辑。用户需要指定：


1.源fp32模块的Python类型(模型中存在)2。被观察模块的Python类型(由用户提供)。该模块需要定义一个 from_float 函数，该函数定义如何从原始 fp32 模块创建观察到的模块。3。量化模块的Python类型(由用户提供)。该模块需要定义一个 from_observed 函数，该函数定义如何从观察到的模块创建量化模块。4。描述上述 (1)、(2)、(3) 的配置，传递给量化 API。


 然后框架将执行以下操作：


1. 在准备模块交换期间，它将使用 (2).2 中类的 from_float 函数将 (1) 中指定类型的每个模块转换为 (2) 中指定的类型。在转换模块交换期间，它将使用 (3) 中类的 from_observed 函数将 (2) 中指定类型的每个模块转换为 (3) 中指定的类型。


 目前，要求 ObservedCustomModule 将具有单个 Tensor 输出，并且框架(而不是用户)将在该输出上添加观察者。观察者将作为自定义模块实例的属性存储在activation_post_process 键下。未来可能会放宽这些限制。


 自定义 API 示例：


```
import torch
import torch.ao.nn.quantized as nnq
from torch.ao.quantization import QConfigMapping
import torch.ao.quantization.quantize_fx

# original fp32 module to replace
class CustomModule(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = torch.nn.Linear(3, 3)

    def forward(self, x):
        return self.linear(x)

# custom observed module, provided by user
class ObservedCustomModule(torch.nn.Module):
    def __init__(self, linear):
        super().__init__()
        self.linear = linear

    def forward(self, x):
        return self.linear(x)

    @classmethod
    def from_float(cls, float_module):
        assert hasattr(float_module, 'qconfig')
        observed = cls(float_module.linear)
        observed.qconfig = float_module.qconfig
        return observed

# custom quantized module, provided by user
class StaticQuantCustomModule(torch.nn.Module):
    def __init__(self, linear):
        super().__init__()
        self.linear = linear

    def forward(self, x):
        return self.linear(x)

    @classmethod
    def from_observed(cls, observed_module):
        assert hasattr(observed_module, 'qconfig')
        assert hasattr(observed_module, 'activation_post_process')
        observed_module.linear.activation_post_process =             observed_module.activation_post_process
        quantized = cls(nnq.Linear.from_float(observed_module.linear))
        return quantized

#
# example API call (Eager mode quantization)
#

m = torch.nn.Sequential(CustomModule()).eval()
prepare_custom_config_dict = {
    "float_to_observed_custom_module_class": {
        CustomModule: ObservedCustomModule
    }
}
convert_custom_config_dict = {
    "observed_to_quantized_custom_module_class": {
        ObservedCustomModule: StaticQuantCustomModule
    }
}
m.qconfig = torch.ao.quantization.default_qconfig
mp = torch.ao.quantization.prepare(
    m, prepare_custom_config_dict=prepare_custom_config_dict)
# calibration (not shown)
mq = torch.ao.quantization.convert(
    mp, convert_custom_config_dict=convert_custom_config_dict)
#
# example API call (FX graph mode quantization)
#
m = torch.nn.Sequential(CustomModule()).eval()
qconfig_mapping = QConfigMapping().set_global(torch.ao.quantization.default_qconfig)
prepare_custom_config_dict = {
    "float_to_observed_custom_module_class": {
        "static": {
            CustomModule: ObservedCustomModule,
        }
    }
}
convert_custom_config_dict = {
    "observed_to_quantized_custom_module_class": {
        "static": {
            ObservedCustomModule: StaticQuantCustomModule,
        }
    }
}
mp = torch.ao.quantization.quantize_fx.prepare_fx(
    m, qconfig_mapping, torch.randn(3,3), prepare_custom_config=prepare_custom_config_dict)
# calibration (not shown)
mq = torch.ao.quantization.quantize_fx.convert_fx(
    mp, convert_custom_config=convert_custom_config_dict)

```


## 最佳实践 [¶](#best-practices "此标题的永久链接")


 1. 如果您使用的是 `x86` 后端，我们需要使用 7 位而不是 8 位。确保减小 `quant\_min` 、 `quant\_max` 的范围，例如，如果 `dtype` 是 `torch.quint8` ，请确保将自定义 `quant_min` 设置为 `0`并且 `quant_max` 为 `127` ( `255` /`2` )如果 `dtype` 为 `torch.qint8` ，请确保将自定义 `quant_min` 设置为 `-64` ( ` -128` /`2` ) 和 `quant_max` 为 `63` ( `127` /`2` )，如果您调用 torch.ao.quantization.get_default_qconfig(backend ) 或 torch.ao.quantization.get_default_qat_qconfig(backend) 函数获取 `x86` 或 `qnnpack` 后端的默认 `qconfig`


 2. 如果选择 `onednn` 后端，则默认 qconfig 映射 `torch.ao.quantization.get_default_qconfig_mapping('onednn')` 和默认 qconfig `torch.ao 中将使用 8 位激活。 quantization.get_default_qconfig('onednn')` 。建议在支持矢量神经网络指令 (VNNI) 的 CPU 上使用。否则，将激活观察者的“reduce_range”设置为 True，以便在没有 VNNI 支持的 CPU 上获得更好的精度。


## 常见问题 [¶](#frequently-asked-questions "此标题的永久链接")


1. 如何在GPU上进行量化推理？：


 我们还没有官方 GPU 支持，但这是一个正在积极开发的领域，您可以在[此处](https://github.com/pytorch/pytorch/issues/87395)2找到更多信息。我在哪里可以获得量化模型的 ONNX 支持？：


 当您遇到 ONNX 问题时，您可以在 [GitHub 
- onnx/onnx](https://github.com/onnx/onnx) 中提出问题，或者联系此列表中的人员：[PyTorch 治理 |维护者 | ONNX 导出器](https://pytorch.org/docs/stable/community/persons_of_interest.html#onnx-exporter)3.如何使用 LSTM 进行量化？：


 我们的自定义模块 api 在 eager 模式和 fx graph 模式量化中支持 LSTM。示例可以在Eager模式中找到： [pytorch/test_quantized_op.py TestQuantizedOps.test_custom_module_lstm](https://github.com/pytorch/pytorch/blob/9b88dcf248e717ca6c3f8c5e11f600825547a561/test/quantization/core/test_quantized_op.py#L2782) FX 图形模式： [pytorch/test_quantize_fx.py TestQuantizeFx.test_static_lstm](https://github.com/pytorch/pytorch/blob/9b88dcf248e717ca6c3f8c5e11f600825547a561/test/quantization/fx /test_quantize_fx.py#L4116)


## 常见错误 [¶](#common-errors "此标题的永久链接")


### 将非量化tensor传递到量化内核中 [¶](#passing-a-non-quantized-tensor-into-a-quantized-kernel "Permalink to this header")


 如果您看到类似以下内容的错误：


```
RuntimeError: Could not run 'quantized::some_operator' with arguments from the 'CPU' backend...

```


 这意味着您正在尝试将非量化tensor传递给量化内核。常见的解决方法是使用“torch.ao.quantization.QuantStub”来量化tensor。这需要在 Eager 模式量化中手动完成。一个 e2e 示例：


```
class M(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.quant = torch.ao.quantization.QuantStub()
        self.conv = torch.nn.Conv2d(1, 1, 1)

    def forward(self, x):
        # during the convert step, this will be replaced with a
        # `quantize_per_tensor` call
        x = self.quant(x)
        x = self.conv(x)
        return x

```


### 将量化tensor传递到非量化内核中 [¶](#passing-a-quantized-tensor-into-a-non-quantized-kernel "Permalink to this header")


 如果您看到类似以下内容的错误：


```
RuntimeError: Could not run 'aten::thnn_conv2d_forward' with arguments from the 'QuantizedCPU' backend.

```


 这意味着您正在尝试将量化tensor传递给非量化内核。常见的解决方法是使用“torch.ao.quantization.DeQuantStub”来对tensor进行反量化。这需要在 Eager 模式量化中手动完成。一个 e2e 示例：


```
class M(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.quant = torch.ao.quantization.QuantStub()
        self.conv1 = torch.nn.Conv2d(1, 1, 1)
        # this module will not be quantized (see `qconfig = None` logic below)
        self.conv2 = torch.nn.Conv2d(1, 1, 1)
        self.dequant = torch.ao.quantization.DeQuantStub()

    def forward(self, x):
        # during the convert step, this will be replaced with a
        # `quantize_per_tensor` call
        x = self.quant(x)
        x = self.conv1(x)
        # during the convert step, this will be replaced with a
        # `dequantize` call
        x = self.dequant(x)
        x = self.conv2(x)
        return x

m = M()
m.qconfig = some_qconfig
# turn off quantization for conv2
m.conv2.qconfig = None

```


### 保存和加载量化模型 [¶](# saving-and-loading-quantized-models "固定链接到此标题")


 在量化模型上调用“torch.load”时，如果您看到如下错误：


```
AttributeError: 'LinearPackedParams' object has no attribute '_modules'

```


 这是因为不支持使用“torch.save”和“torch.load”直接保存和加载量化模型。要保存/加载量化模型，可以使用以下方式：


1. 保存/加载量化模型状态_dict


 一个例子：


```
class M(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(5, 5)
        self.relu = nn.ReLU()

    def forward(self, x):
        x = self.linear(x)
        x = self.relu(x)
        return x

m = M().eval()
prepare_orig = prepare_fx(m, {'' : default_qconfig})
prepare_orig(torch.rand(5, 5))
quantized_orig = convert_fx(prepare_orig)

# Save/load using state_dict
b = io.BytesIO()
torch.save(quantized_orig.state_dict(), b)

m2 = M().eval()
prepared = prepare_fx(m2, {'' : default_qconfig})
quantized = convert_fx(prepared)
b.seek(0)
quantized.load_state_dict(torch.load(b))

```


2. 使用“torch.jit.save”和“torch.jit.load”保存/加载脚本量化模型


 一个例子：


```
# Note: using the same model M from previous example
m = M().eval()
prepare_orig = prepare_fx(m, {'' : default_qconfig})
prepare_orig(torch.rand(5, 5))
quantized_orig = convert_fx(prepare_orig)

# save/load using scripted model
scripted = torch.jit.script(quantized_orig)
b = io.BytesIO()
torch.jit.save(scripted, b)
b.seek(0)
scripted_quantized = torch.jit.load(b)

```


### 使用 FX 图形模式量化时的符号跟踪错误 [¶](#symbolic-trace-error-when-using-fx-graph-mode-quantization "永久链接到此标题")


 符号可追溯性是 [(Prototype) FX Graph Mode Quantization](#prototype-fx-graph-mode-quantization) 的要求，因此，如果您传递的 PyTorch 模型不可符号追溯至 torch.ao.quantization.prepare_fx或 torch.ao.quantization.prepare_qat_fx ，我们可能会看到如下错误：


```
torch.fx.proxy.TraceError: symbolically traced variables cannot be used as inputs to control flow

```


 请查看 [符号跟踪的局限性](https://docs-preview.pytorch.org/76223/fx.html#limitations-of-symbolic-tracing) 并使用 
- [使用 FX 图形模式量化的用户指南](https://pytorch.org/tutorials/prototype/fx_graph_mode_quant_guide.html) 来解决该问题。