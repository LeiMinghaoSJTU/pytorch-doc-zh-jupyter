# torch.masked [¶](#torch-masked "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/masked>
>
> 原始地址：<https://pytorch.org/docs/stable/masked.html>


## 简介 [¶](#introduction "此标题的永久链接")


### 动机 [¶](#motivation "此标题的永久链接")


!!! warning "警告"

     屏蔽tensor的 PyTorch API 处于原型阶段，将来可能会或可能不会改变。


 MaskedTensor 作为 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 的扩展，为用户提供了以下功能：



* 使用任何屏蔽语义(例如可变长度tensor、nan* 运算符等)
* 区分 0 和 NaN 梯度
* 各种稀疏应用(请参阅下面的教程)


 “指定”和“未指定”在 PyTorch 中由来已久，没有正式的语义，当然也没有一致性；事实上，MaskedTensor 的诞生是由于普通的 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 类无法正确解决的问题的累积。因此，MaskedTensor 的主要目标是成为 PyTorch 中“指定”和“未指定”值的真相来源，它们是一等公民而不是事后的想法。反过来，这应该进一步解锁[稀疏性](https： //pytorch.org/docs/stable/sparse.html)的潜力，使操作员更安全、更一致，并为用户和开发人员提供更流畅、更直观的体验。


### 什么是 MaskedTensor？ [¶](#what-is-a-maskedtensor"此标题的永久链接")


 MaskedTensor 是一个tensor子类，由 1) 输入(数据)和 2) 掩码组成。掩码告诉我们应该包含或忽略输入中的哪些条目。


 举例来说，假设我们想要屏蔽掉所有等于 0 的值(用灰色表示)并取最大值：


[![_images/tensor_comparison.jpg](_images/tensor_comparison.jpg)](_images/tensor_comparison.jpg) 顶部是普通tensor示例，而底部是 MaskedTensor，其中所有 0 都被屏蔽。这显然会产生不同的结果取决于我们是否有掩码，但这种灵活的结构允许用户在计算过程中系统地忽略他们想要的任何元素。


 我们已经编写了许多现有教程来帮助用户入门，例如：



* [概述 
- 新用户开始的地方，讨论如何使用 MaskedTensors 以及它们为什么有用](https://pytorch.org/tutorials/prototype/maskedtensor_overview)
* [Sparsity 
- MaskedTensor 支持稀疏 COO 和 CSR 数据和 mask Tensors](https://pytorch.org/tutorials/prototype/maskedtensor_sparsity)
* [Adagrad 稀疏语义 
- MaskedTensor 如何简化稀疏语义和实现的实际示例](https://pytorch.org/tutorials/prototype /maskedtensor_adagrad)
* [高级语义 
- 讨论为什么做出某些决定(例如，要求掩码匹配二进制/归约操作)、与 NumPy 的 MaskedArray 的差异以及归约语义](https://pytorch.org/tutorials/prototype/maskedtensor_advanced_semantics)


## 支持的运算符 [¶](#supported-operators "固定链接到此标题")


### 一元运算符 [¶](#unary-operators "此标题的永久链接")


 一元运算符是仅包含单个输入的运算符。将它们应用于 MaskedTensors 相对简单：如果数据在给定索引处被屏蔽，我们将应用该运算符，否则我们将继续屏蔽数据。


 可用的一元运算符有：


|  |  |
| --- | --- |
| [`abs`](generated/torch.abs.html#torch.abs "torch.abs") |计算 `input` 中每个元素的绝对值。 |
| [`绝对`](generated/torch.absolute.html#torch.absolute "torch.absolute") | [`torch.abs()`](generated/torch.abs.html#torch.abs "torch.abs") |
| 的别名


[`acos`](generated/torch.acos.html#torch.acos "torch.acos") |计算 `input` 中每个元素的反余弦。 |
| [`arccos`](generated/torch.arccos.html#torch.arccos "torch.arccos") | [`torch.acos()`](generated/torch.acos.html#torch.acos "torch.acos") 的别名。 |
| [`acosh`](generated/torch.acosh.html#torch.acosh“torch.acosh”)|返回具有 `input` 元素的反双曲余弦的新tensor。 |
| [`arccosh`](generated/torch.arccosh.html#torch.arccosh“torch.arccosh”)| [`torch.acosh()`](generated/torch.acosh.html#torch.acosh "torch.acosh") 的别名。 |
| [`角度`](generated/torch.angle.html#torch.angle "torch.angle") |计算给定“输入”tensor的元素角度(以弧度为单位)。 |
| [`asin`](generated/torch.asin.html#torch.asin "torch.asin") |返回一个新的tensor，其值为 `input` 元素的反正弦。 |
| [`arcsin`](generated/torch.arcsin.html#torch.arcsin "torch.arcsin") | [`torch.asin()`](generated/torch.asin.html#torch.asin "torch.asin") 的别名。 |
| [`asinh`](generated/torch.asinh.html#torch.asinh "torch.asinh") |返回一个具有 `input` 元素的反双曲正弦的新tensor。 |
| [`arcsinh`](generated/torch.arcsinh.html#torch.arcsinh "torch.arcsinh") | [`torch.asinh()`](generated/torch.asinh.html#torch.asinh "torch.asinh") 的别名。 |
| [`atan`](generated/torch.atan.html#torch.atan "torch.atan") |返回一个新的tensor，其值为 `input` 元素的反正切值。 |
| [`arctan`](generated/torch.arctan.html#torch.arctan "torch.arctan") | [`torch.atan()`](generated/torch.atan.html#torch.atan "torch.atan") 的别名。 |
| [`atanh`](generated/torch.atanh.html#torch.atanh "torch.atanh") |返回一个带有 `input` 元素的反双曲正切的新tensor。 |
| [`arctanh`](generated/torch.arctanh.html#torch.arctanh "torch.arctanh") | [`torch.atanh()`](generated/torch.atanh.html#torch.atanh "torch.atanh") 的别名。 |
| [`bitwise_not`](generated/torch.bitwise_not.html#torch.bitwise_not "torch.bitwise_not") |计算给定输入tensor的按位 NOT。 |
| [`ceil`](generated/torch.ceil.html#torch.ceil "torch.ceil") |返回一个新的tensor，其中包含 `input` 元素的 ceil，即大于或等于每个元素的最小整数。 |
| [`clamp`](generated/torch.clamp.html#torch.clamp "torch.clamp") |将 `input` 中的所有元素限制在范围 [ [`min`](generated/torch.min.html#torch.min "torch.min") , [`max`](generated/torch.max.html#torch.max "火炬.max") ]. |
| [`clip`](generated/torch.clip.html#torch.clip“torch.clip”)| [`torch.clamp()`](generated/torch.clamp.html#torch.clamp "torch.clamp") 的别名。 |
| [`conj_physical`](generated/torch.conj_physical.html#torch.conj_physical "torch.conj_physical") |计算给定“输入”tensor的逐元素共轭。 |
| [`cos`](generated/torch.cos.html#torch.cos "torch.cos") |返回一个新的tensor，其值为 `input` 元素的余弦值。 |
| [`cosh`](generated/torch.cosh.html#torch.cosh "torch.cosh") |返回具有 `input` 元素的双曲余弦的新tensor。 |
| [`deg2rad`](generated/torch.deg2rad.html#torch.deg2rad "torch.deg2rad") |返回一个新的tensor，其中“input”的每个元素都从角度(以度为单位)转换为弧度。 |
| [`digamma`](generated/torch.digamma.html#torch.digamma“torch.digamma”)| [`torch.special.digamma()`](special.html#torch.special.digamma "torch.special.digamma") 的别名。 |
| [`erf`](generated/torch.erf.html#torch.erf "torch.erf") | [`torch.special.erf()`](special.html#torch.special.erf "torch.special.erf") 的别名。 |
| [`erfc`](generated/torch.erfc.html#torch.erfc "torch.erfc") | [`torch.special.erfc()`](special.html#torch.special.erfc "torch.special.erfc") 的别名。 |
| [`erfinv`](generated/torch.erfinv.html#torch.erfinv "torch.erfinv") | [`torch.special.erfinv()`](special.html#torch.special.erfinv "torch.special.erfinv") 的别名。 |
| [`exp`](generated/torch.exp.html#torch.exp "torch.exp") |返回一个新tensor，其具有输入tensor“input”元素的指数。 |
| [`exp2`](generated/torch.exp2.html#torch.exp2 "torch.exp2") | [`torch.special.exp2()`](special.html#torch.special.exp2 "torch.special.exp2") 的别名。 |
| [`expm1`](generated/torch.expm1.html#torch.expm1 "torch.expm1") | [`torch.special.expm1()`](special.html#torch.special.expm1 "torch.special.expm1") 的别名。 |
| [`修复`](generated/torch.fix.html#torch.fix "torch.fix") | [`torch.trunc()`](generated/torch.trunc.html#torch.trunc "torch.trunc") 的别名 |
| [`floor`](generated/torch.floor.html#torch.floor "torch.floor") |返回一个新的tensor，其元素为“input”的下限，即小于或等于每个元素的最大整数。 |
| [`frac`](generated/torch.frac.html#torch.frac "torch.frac") |计算 `input` 中每个元素的小数部分。 |
| [`lgamma`](generated/torch.lgamma.html#torch.lgamma "torch.lgamma") |计算 `input` 上 gamma 函数绝对值的自然对数。 |
| [`log`](generated/torch.log.html#torch.log "torch.log") |返回具有 `input` 元素的自然对数的新tensor。 |
| [`log10`](generated/torch.log10.html#torch.log10 "torch.log10") |返回一个新的tensor，其对数以 `input` 的元素为底 10。 |
| [`log1p`](generated/torch.log1p.html#torch.log1p "torch.log1p") |返回自然对数为 (1 
+ `input` ) 的新tensor。 |
| [`log2`](generated/torch.log2.html#torch.log2 "torch.log2") |返回一个新的tensor，其对数以 `input` 的元素为底 2。 |
| [`logit`](generated/torch.logit.html#torch.logit "torch.logit") | [`torch.special.logit()`](special.html#torch.special.logit "torch.special.logit") 的别名。 |
| [`i0`](generated/torch.i0.html#torch.i0 "torch.i0") | [`torch.special.i0()`](special.html#torch.special.i0 "torch.special.i0") 的别名。 |
| [`isnan`](generated/torch.isnan.html#torch.isnan "torch.isnan") |返回一个新的tensor，其中布尔元素表示“input”的每个元素是否为 NaN。 |
| [`nan_to_num`](generated/torch.nan_to_num.html#torch.nan_to_num "torch.nan_to_num") |将“input”中的“NaN”、正无穷大和负无穷值分别替换为“nan”、“posinf”和“neginf”指定的值。 |
| [`neg`](generated/torch.neg.html#torch.neg "torch.neg") |返回一个新tensor，其值为 `input` 元素的负数。 |
| [`负数`](generated/torch.负数.html#torch.负数“火炬.负数”) | [`torch.neg()`](generated/torch.neg.html#torch.neg "torch.neg") |
| 的别名


[`积极`](generated/torch. Positive.html#torch. Positive "torch. Positive") |返回`输入`。 |
| [`pow`](generated/torch.pow.html#torch.pow "torch.pow") |使用“exponent”获取“input”中每个元素的幂，并返回带有结果的tensor。 |
| [`rad2deg`](generated/torch.rad2deg.html#torch.rad2deg "torch.rad2deg") |返回一个新的tensor，其中“input”的每个元素都从弧度角度转换为度数。 |
| [`倒数`](generated/torch.reciprocal.html#torch.reciprocal "torch.reciprocal") |返回一个新的tensor，其值为 `input` 元素的倒数 |
| [`round`](generated/torch.round.html#torch.round "torch.round") |将“input”的元素舍入为最接近的整数。 |
| [`rsqrt`](generated/torch.rsqrt.html#torch.rsqrt "torch.rsqrt") |返回一个新的tensor，其值为 `input` 每个元素的平方根的倒数。 |
| [`sigmoid`](generated/torch.sigmoid.html#torch.sigmoid“torch.sigmoid”) | [`torch.special.expit()`](special.html#torch.special.expit "torch.special.expit") 的别名。 |
| [`sign`](generated/torch.sign.html#torch.sign "torch.sign") |返回一个带有 `input` 元素符号的新tensor。 |
| [`sgn`](generated/torch.sgn.html#torch.sgn "torch.sgn") |该函数是 torch.sign() 对复杂tensor的扩展。 |
| [`signbit`](generated/torch.signbit.html#torch.signbit "torch.signbit") |测试“input”的每个元素是否设置了符号位。 |
| [`sin`](generated/torch.sin.html#torch.sin "torch.sin") |返回一个新的tensor，其值为 `input` 元素的正弦值。 |
| [`sinc`](generated/torch.sinc.html#torch.sinc "torch.sinc") | [`torch.special.sinc()`](special.html#torch.special.sinc "torch.special.sinc") 的别名。 |
| [`sinh`](generated/torch.sinh.html#torch.sinh "torch.sinh") |返回具有 `input` 元素的双曲正弦的新tensor。 |
| [`sqrt`](generated/torch.sqrt.html#torch.sqrt "torch.sqrt") |返回一个新的tensor，其值为 `input` 元素的平方根。 |
| [`square`](generated/torch.square.html#torch.square "torch.square") |返回一个新的tensor，其值为 `input` 元素的平方。 |
| [`tan`](generated/torch.tan.html#torch.tan "torch.tan") |返回一个新的tensor，其值为 `input` 元素的正切值。 |
| [`tanh`](generated/torch.tanh.html#torch.tanh "torch.tanh") |返回具有 `input` 元素的双曲正切的新tensor。 |
| [`trunc`](generated/torch.trunc.html#torch.trunc "torch.trunc") |返回一个新的tensor，其中包含 `input` 元素的截断整数值。 |


 可用的就地一元运算符是上面所有的**除了**：


|  |  |
| --- | --- |
| [`角度`](generated/torch.angle.html#torch.angle "torch.angle") |计算给定“输入”tensor的元素角度(以弧度为单位)。 |
| [`积极`](generated/torch. Positive.html#torch. Positive "torch. Positive") |返回`输入`。 |
| [`signbit`](generated/torch.signbit.html#torch.signbit "torch.signbit") |测试“input”的每个元素是否设置了符号位。 |
| [`isnan`](generated/torch.isnan.html#torch.isnan "torch.isnan") |返回一个新的tensor，其中布尔元素表示“input”的每个元素是否为 NaN。 |


### 二元运算符 [¶](#binary-operators "此标题的永久链接")


 正如您在教程中所看到的，“MaskedTensor”还实现了二元运算，但需要注意的是两个 MaskedTensor 中的掩码必须匹配，否则会引发错误。正如错误中所指出的，如果您需要对特定运算符的支持，或者建议了它们应该如何表现的语义，请在 GitHub 上打开一个问题。目前，我们决定采用最保守的实现方式，以确保用户确切地知道正在发生的事情，并通过屏蔽语义有意做出他们的决定。


 可用的二元运算符有：


|  |  |
| --- | --- |
| [`add`](generated/torch.add.html#torch.add "torch.add") |将按 `alpha` 缩放的 `other` 添加到 `input` 中。 |
| [`atan2`](generated/torch.atan2.html#torch.atan2 "torch.atan2") |元素方面的反正切


 输入我


 /
 


 其他我


 	ext{输入}_{i} /	ext{其他}_{i}




 input
 


 i
 


 ​
 


 /
 



 other
 


 i
 


 ​
 


 考虑到象限。 |
| [`arctan2`](generated/torch.arctan2.html#torch.arctan2 "torch.arctan2") | [`torch.atan2()`](generated/torch.atan2.html#torch.atan2 "torch.atan2") 的别名。 |
| [`bitwise_and`](generated/torch.bitwise_and.html#torch.bitwise_and "torch.bitwise_and") |计算 `input` 和 `other` 的按位与。 |
| [`按位_or`](generated/torch.bitwise_or.html#torch.bitwise_or "torch.bitwise_or") |计算 `input` 和 `other` 的按位或。 |
| [`按位_xor`](generated/torch.bitwise_xor.html#torch.bitwise_xor "torch.bitwise_xor") |计算 `input` 和 `other` 的按位异或。 |
| [`bitwise_left_shift`](generated/torch.bitwise_left_shift.html#torch.bitwise_left_shift "torch.bitwise_left_shift") |通过“其他”位计算“输入”的左算术移位。 |
| [`bitwise_right_shift`](generated/torch.bitwise_right_shift.html#torch.bitwise_right_shift "torch.bitwise_right_shift") |通过“其他”位计算“输入”的右算术移位。 |
| [`div`](generated/torch.div.html#torch.div "torch.div") |将输入 `input` 的每个元素除以 `other` 的相应元素。 |
| [`divide`](generated/torch.divide.html#torch.divide "torch.divide") | [`torch.div()`](generated/torch.div.html#torch.div "torch.div") 的别名。 |
| [`floor_divide`](generated/torch.floor_divide.html#torch.floor_divide "torch.floor_divide") | |
| [`fmod`](generated/torch.fmod.html#torch.fmod "torch.fmod") |按条目应用 C++ 的 [std::fmod](https://en.cppreference.com/w/cpp/numeric/math/fmod)。 |
| [`logaddexp`](generated/torch.logaddexp.html#torch.logaddexp "torch.logaddexp") |输入的幂总和的对数。 |
| [`logaddexp2`](generated/torch.logaddexp2.html#torch.logaddexp2 "torch.logaddexp2") |输入以 2 为底的指数总和的对数。 |
| [`mul`](generated/torch.mul.html#torch.mul "torch.mul") |将 `input` 乘以 `other` 。 |
| [`multiply`](generated/torch.multiply.html#torch.multiply "torch.multiply") | [`torch.mul()`](generated/torch.mul.html#torch.mul "torch.mul") 的别名。 |
| [`nextafter`](generated/torch.nextafter.html#torch.nextafter "torch.nextafter") |按元素将 `input` 之后的下一个浮点值返回到 `other` 。 |
| [`remainder`](generated/torch.remainder.html#torch.remainder "torch.remainder") |按条目计算 [Python 的模运算](https://docs.python.org/3/reference/expressions.html#binary-arithmetic-operations)。 |
| [`sub`](generated/torch.sub.html#torch.sub "torch.sub") |从 `input` 中减去按 `alpha` 缩放的 `other` 。 |
| [`减法`](generated/torch.subtract.html#torch.subtract "torch.subtract") | [`torch.sub()`](generated/torch.sub.html#torch.sub "torch.sub") 的别名。 |
| [`true_divide`](generated/torch.true_divide.html#torch.true_divide "torch.true_divide") | [`torch.div()`](generated/torch.div.html#torch.div "torch.div") 的别名，带有 `rounding_mode=None` 。 |
| [`eq`](generated/torch.eq.html#torch.eq "torch.eq") |计算元素级相等 |
| [`ne`](generated/torch.ne.html#torch.ne "torch.ne") |计算


 输入≠其他


 	ext{输入} 
eq 	ext{其他}



 input
 




 
 



 =
 



 other
 


 元素方面。 |
| [`le`](generated/torch.le.html#torch.le "torch.le") |计算


 输入≤其他


 	ext{输入} \leq 	ext{其他}



 input
 




 ≤
 


 other
 


 元素方面。 |
| [`ge`](generated/torch.ge.html#torch.ge "torch.ge") |计算


 输入≥其他


 	ext{输入} \geq 	ext{其他}



 input
 




 ≥
 


 other
 


 元素方面。 |
| [`greater`](generated/torch.greater.html#torch.greater "torch.greater") | [`torch.gt()`](generated/torch.gt.html#torch.gt "torch.gt") 的别名。 |
| [`greater_equal`](generated/torch.greater_equal.html#torch.greater_equal "torch.greater_equal") | [`torch.ge()`](generated/torch.ge.html#torch.ge "torch.ge") 的别名。 |
| [`gt`](generated/torch.gt.html#torch.gt "torch.gt") |计算


 输入 
> 其他


 	ext{输入} 
> 	ext{其他}



 input
 




 >
 


 other
 


 元素方面。 |
| [`less_equal`](generated/torch.less_equal.html#torch.less_equal "torch.less_equal") | [`torch.le()`](generated/torch.le.html#torch.le "torch.le") 的别名。 |
| [`lt`](generated/torch.lt.html#torch.lt "torch.lt") |计算


 输入 < 其他


 	ext{输入} < 	ext{其他}



 input
 




 <
 


 other
 


 元素方面。 |
| [`less`](generated/torch.less.html#torch.less "torch.less") | [`torch.lt()`](generated/torch.lt.html#torch.lt "torch.lt") 的别名。 |
| [`最大值`](generated/torch.maximum.html#torch.maximum "torch.maximum") |计算 `input` 和 `other` 的逐元素最大值。 |
| [`最小值`](generated/torch.minimum.html#torch.minimum "torch.minimum") |计算 `input` 和 `other` 的元素最小值。 |
| [`fmax`](generated/torch.fmax.html#torch.fmax "torch.fmax") |计算 `input` 和 `other` 的逐元素最大值。 |
| [`fmin`](generated/torch.fmin.html#torch.fmin "torch.fmin") |计算 `input` 和 `other` 的元素最小值。 |
| [`不_等于`](generated/torch.not_equal.html#torch.not_equal "torch.not_equal") | [`torch.ne()`](generated/torch.ne.html#torch.ne "torch.ne") 的别名。 |


 可用的就地二元运算符除**之外都是以上：


|  |  |
| --- | --- |
| [`logaddexp`](generated/torch.logaddexp.html#torch.logaddexp "torch.logaddexp") |输入的幂总和的对数。 |
| [`logaddexp2`](generated/torch.logaddexp2.html#torch.logaddexp2 "torch.logaddexp2") |输入以 2 为底的指数总和的对数。 |
| [`equal`](generated/torch.equal.html#torch.equal "torch.equal") |如果两个tensor具有相同的大小和元素，则为“True”，否则为“False”。 |
| [`fmin`](generated/torch.fmin.html#torch.fmin "torch.fmin") |计算 `input` 和 `other` 的元素最小值。 |
| [`最小值`](generated/torch.minimum.html#torch.minimum "torch.minimum") |计算 `input` 和 `other` 的元素最小值。 |
| [`fmax`](generated/torch.fmax.html#torch.fmax "torch.fmax") |计算 `input` 和 `other` 的逐元素最大值。 |


### Reductions [¶](#reductions "此标题的永久链接")


 可以进行以下减少(通过 autograd 支持)。有关更多信息，[概述](https://pytorch.org/tutorials/prototype/maskedtensor_overview.html) 教程详细介绍了一些缩减示例，而[高级语义](https://pytorch.org/tutorials/prototype/maskedtensor_advanced_semantics.html)教程对我们如何决定某些简化语义进行了一些更深入的讨论。


|  |  |
| --- | --- |
| [`sum`](generated/torch.sum.html#torch.sum "torch.sum") |返回“输入”tensor中所有元素的总和。 |
| [`mean`](generated/torch.mean.html#torch.mean "torch.mean") |返回“输入”tensor中所有元素的平均值。 |
| [`amin`](generated/torch.amin.html#torch.amin "torch.amin") |返回给定维度“dim”中“输入”tensor的每个切片的最小值。 |
| [`amax`](generated/torch.amax.html#torch.amax "torch.amax") |返回给定维度“dim”中“输入”tensor的每个切片的最大值。 |
| [`argmin`](generated/torch.argmin.html#torch.argmin "torch.argmin") |返回展平tensor或沿维度 |
| 的最小值的索引


[`argmax`](generated/torch.argmax.html#torch.argmax "torch.argmax") |返回“输入”tensor中所有元素的最大值的索引。 |
| [`prod`](generated/torch.prod.html#torch.prod "torch.prod") |返回“输入”tensor中所有元素的乘积。 |
| [`全部`](generated/torch.all.html#torch.all "torch.all") |测试“input”中的所有元素的计算结果是否为 True 。 |
| [`norm`](generated/torch.norm.html#torch.norm "torch.norm") |返回给定tensor的矩阵范数或向量范数。 |
| [`var`](generated/torch.var.html#torch.var "torch.var") |计算“dim”指定维度上的方差。 |
| [`std`](generated/torch.std.html#torch.std "torch.std") |计算“dim”指定尺寸的标准偏差。 |


### 查看和选择函数 [¶](#view-and-select-functions "永久链接到此标题")


 我们还提供了许多查看和选择功能；直观上，这些运算符将应用于数据和掩码，然后将结果包装在“MaskedTensor”中。举个简单的例子，考虑 [`select()`](generated/torch.select.html#torch.select "torch.select") ：


```
>>> data = torch.arange(12, dtype=torch.float).reshape(3, 4)
>>> data
tensor([[ 0., 1., 2., 3.],
 [ 4., 5., 6., 7.],
 [ 8., 9., 10., 11.]])
>>> mask = torch.tensor([[True, False, False, True], [False, True, False, False], [True, True, True, True]])
>>> mt = masked_tensor(data, mask)
>>> data.select(0, 1)
tensor([4., 5., 6., 7.])
>>> mask.select(0, 1)
tensor([False, True, False, False])
>>> mt.select(0, 1)
MaskedTensor(
 [ --, 5.0000, --, --]
)

```


 目前支持以下操作：


|  |  |
| --- | --- |
| [`atleast_1d`](generated/torch.atleast_1d.html#torch.atleast_1d "torch.atleast_1d") |返回每个零维度输入tensor的一维视图。 |
| [`broadcast_tensors`](generated/torch.broadcast_tensors.html#torch.broadcast_tensors "torch.broadcast_tensors") |根据 [广播语义](notes/broadcasting.html#broadcasting-semantics) 广播给定的tensor。 |
| [`broadcast_to`](generated/torch.broadcast_to.html#torch.broadcast_to "torch.broadcast_to") |将“input”广播到形状“shape”。 |
| [`cat`](generated/torch.cat.html#torch.cat "torch.cat") |连接给定维度中给定的“seq”tensor序列。 |
| [`chunk`](generated/torch.chunk.html#torch.chunk "torch.chunk") |尝试将tensor拆分为指定数量的块。 |
| [`column_stack`](generated/torch.column_stack.html#torch.column_stack "torch.column_stack") |通过水平堆叠 `tensors` 中的tensor来创建一个新的tensor。 |
| [`dsplit`](generated/torch.dsplit.html#torch.dsplit "torch.dsplit") |根据“indices_or_sections”将具有三个或更多维度的tensor“input”按深度拆分为多个tensor。 |
| [`展平`](generated/torch.展平.html#torch.展平“火炬.展平”) |通过将“输入”重塑为一维tensor来展平“输入”。 |
| [`hsplit`](generated/torch.hsplit.html#torch.hsplit "torch.hsplit") |根据 `indices_or_sections` 将具有一维或多维的tensor `input` 水平拆分为多个tensor。 |
| [`hstack`](generated/torch.hstack.html#torch.hstack "torch.hstack") |按水平顺序堆叠tensor(按列)。 |
| [`kron`](generated/torch.kron.html#torch.kron "torch.kron") |计算克罗内克积，表示为



 ⊗
 


 有时


 ⊗
 


 ，“输入”和“其他”。 |
| [`meshgrid`](generated/torch.meshgrid.html#torch.meshgrid "torch.meshgrid") |创建由 attr :tensors 中的 1D 输入指定的坐标网格。 |
| [`narrow`](generated/torch.narrow.html#torch.narrow "torch.narrow") |返回一个新tensor，它是“输入”tensor的缩小版本。 |
| [`ravel`](generated/torch.ravel.html#torch.ravel "torch.ravel") |返回连续的展平tensor。 |
| [`选择`](generated/torch.select.html#torch.select "torch.select") |沿给定索引处的选定维度对“输入”tensor进行切片。 |
| [`split`](generated/torch.split.html#torch.split "torch.split") |将tensor分割成块。 |
| [`t`](generated/torch.t.html#torch.t "torch.t") |期望“输入”<= 2-D tensor并转置维度 0 和 1。 |
| [`transpose`](generated/torch.transpose.html#torch.transpose "torch.transpose") |返回一个tensor，它是 `input` 的转置版本。 |
| [`vsplit`](generated/torch.vsplit.html#torch.vsplit "torch.vsplit") |根据 `indices_or_sections` 将具有二维或更多维度的tensor `input` 垂直拆分为多个tensor。 |
| [`vstack`](generated/torch.vstack.html#torch.vstack "torch.vstack") |按顺序垂直(行)堆叠tensor。 |
| [`Tensor.expand`](generated/torch.Tensor.expand.html#torch.Tensor.expand "torch.Tensor.expand") |返回“self”tensor的新视图，其中单例维度扩展到更大的尺寸。 |
| [`Tensor.expand_as`](generated/torch.Tensor.expand_as.html#torch.Tensor.expand_as "torch.Tensor.expand_as") |将此tensor扩展为与“other”相同的大小。 |
| [`Tensor.reshape`](generated/torch.Tensor.reshape.html#torch.Tensor.reshape“torch.Tensor.reshape”)|返回一个tensor，其数据和元素数量与“self”相同，但具有指定的形状。 |
| [`Tensor.reshape_as`](generated/torch.Tensor.reshape_as.html#torch.Tensor.reshape_as "torch.Tensor.reshape_as") |返回此tensor，其形状与“other”相同。 |
| [`Tensor.view`](generated/torch.Tensor.view.html#torch.Tensor.view "torch.Tensor.view") |返回一个新tensor，其数据与“self”tensor相同，但“shape”不同。 |