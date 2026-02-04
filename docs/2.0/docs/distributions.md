# 概率分布 
- torch.distributions [¶](#module-torch.distributions "此标题的固定链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/distributions>
>
> 原始地址：<https://pytorch.org/docs/stable/distributions.html>


 “distributions”包包含可参数化的概率分布和采样函数。这允许构建随机计算图和随机梯度估计器以进行优化。该包通常遵循 [TensorFlow Distributions](https://arxiv.org/abs/1711.10604) 包的设计。


 不可能直接通过随机样本进行反向传播。然而，有两种主要方法来创建可以反向传播的代理函数。这些是得分函数估计器/似然比估计器/REINFORCE 和路径导数估计器。 REINFORCE 通常被视为强化学习中策略梯度方法的基础，而路径导数估计器通常出现在变分自编码器的重新参数化技巧中。而评分函数只需要样本的值


 f(x)


 f(x)
 


 f(x)


 ，路径导数需要导数




 f
 

 ′
 


 (  X  )


 f'(x)
 



 f
 




 ′
 


 (  X  )


 。下一节将在强化学习示例中讨论这两者。有关更多详细信息，请参阅[使用随机计算图的梯度估计](https://arxiv.org/abs/1506.05254)。


## 评分函数 [¶](#score-function "此标题的永久链接")


 当概率密度函数相对于其参数可微时，我们只需要`sample()`和`log_prob()`来实现REINFORCE：


 Δθ=αr


 ∂ log ⁡ p ( a ∣


 π
 

 θ
 


 ( ) )



 ∂
 

 θ
 


 \Delta	heta = lpha r rac{\partial\log p(a|\pi^	heta(s))}{\partial	heta}


 Δ
 

 θ
 



 =
 




 α
 

 r
 



 ∂
 

 θ
 




 ∂
 



 lo
 
 g
 


 p ( ∣


 π
 



 θ
 


 ( )))




 ​
 



 where
 



 θ
 


 θ


 θ
 


 是参数，



 α
 


 \α


 α
 


 是学习率，



 r
 


 r
 


 r
 


 是奖励并且


 p ( ∣


 π
 

 θ
 


 ( ) )


 p(a|\pi^	heta(s))


 p ( ∣


 π
 



 θ
 


 ( )))


 是采取行动的概率



 a
 


 a
 


 a
 


 处于状态



 s
 


 s
 


 s
 


 给定的政策




 π
 

 θ
 


 \pi^\θ



 π
 



 θ
 


.
 


 在实践中，我们将从网络的输出中采样一个动作，在环境中应用该动作，然后使用“log_prob”构建等效的损失函数。请注意，我们使用负数是因为优化器使用梯度下降，而上面的规则假设梯度上升。对于分类策略，实施 REINFORCE 的代码如下：


```
probs = policy_network(state)
# Note that this is equivalent to what used to be called multinomial
m = Categorical(probs)
action = m.sample()
next_state, reward = env.step(action)
loss = -m.log_prob(action) * reward
loss.backward()

```


## 路径导数 [¶](#pathwise-derivative "此标题的永久链接")


 实现这些随机/策略梯度的另一种方法是使用“rsample()”方法中的重新参数化技巧，其中可以通过无参数随机变量的参数化确定性函数来构造参数化随机变量。因此，重新参数化的样本变得可微。实现路径导数的代码如下：


```
params = policy_network(state)
m = Normal(*params)
# Any distribution with .has_rsample == True could work based on the application
action = m.rsample()
next_state, reward = env.step(action)  # Assuming that reward is differentiable
loss = -reward
loss.backward()

```


## 分布 [¶](#distribution "此标题的永久链接")


*班级*


 torch.distributions.distribution。


 分配


 ( *batch_shape



 =
 


 torch.Size([])
* , *event_shape



 =
 


 torch.Size([])
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution)[¶](#torch.distributions.distribution.Distribution "此定义的永久链接")


 基础：[`object`](https://docs.python.org/3/library/functions.html#object "(in Python v3.12)")


 Distribution 是概率分布的抽象基类。


*财产*


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*[¶](#torch.distributions.distribution.Distribution.arg_constraints"此定义的永久链接")


 返回从参数名称到此分布的每个参数应满足的 [`Constraint`](#torch.distributions.constraints.Constraint "torch.distributions.constraints.Constraint") 对象的字典。不是tensor的参数不需要出现在这个字典中。


*财产*


 批次_形状 *:


 Size*[¶](#torch.distributions.distribution.Distribution.batch_shape"此定义的永久链接")


 返回对参数进行批处理的形状。


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution.cdf)[¶](#torch.distributions.distribution.Distribution.cdf "此定义的永久链接")


 返回在 value 处计算的累积密度/质量函数。


 Parameters


**值** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) –


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 熵


 ( ) [[source]](_modules/torch/distributions/distribution.html#Distribution.entropy)[¶](#torch.distributions.distribution.Distribution.entropy "此定义的永久链接")


 返回分布熵，按batch_shape 进行批处理。


 退货


 形状batch_shape 的tensor。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 枚举_支持


 ( *扩张



 =
 


 True
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution.enumerate_support)[¶](#torch.distributions.distribution.Distribution.enumerate_support "此定义的永久链接")


 返回包含离散分布支持的所有值的tensor。结果将在维度 0 上进行枚举，因此结果的形状将为 (cardinality,) 
+ batch_shape 
+ event_shape (其中 event_shape = () 对于单变量分布)。


 请注意，这枚举了锁步 [[0, 0], [1, 1], …] 中的所有批处理tensor。当 Expand=False 时，枚举沿着 dim 0 发生，但剩余的批量维度是单一维度， [[0], [1],... 。


 要迭代完整的笛卡尔积，请使用 itertools.product(m.enumerate_support()) 。


 Parameters


**expand** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否扩展对批次的支持变暗以匹配分布的batch_shape。


 退货


 tensor在维度 0 上迭代。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


*财产*


 事件_形状 *:


 Size*[¶](#torch.distributions.distribution.Distribution.event_shape"此定义的永久链接")


 返回单个样本的形状(不进行批处理)。


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution.expand)[¶](#torch.distributions.distribution.Distribution.expand "此定义的永久链接")


 返回一个新的分布实例(或填充派生类提供的现有实例)，其中批量维度扩展为 batch_shape 。此方法对分布的参数调用 [`expand`](generated/torch.Tensor.expand.html#torch.Tensor.expand "torch.Tensor.expand")。因此，这不会为扩展的分发实例分配新内存。此外，当首次创建实例时，这不会重复 __init__.py 中的任何参数检查或参数广播。


 参数 
* **batch_shape** ( *torch.Size
* ) – 所需的扩展大小。
* **_instance** – 需要覆盖.expand 的子类提供的新实例。


 退货


 新的分发实例的批次尺寸扩展为batch_size。




 icdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution.icdf)[¶](#torch.distributions.distribution.Distribution.icdf "此定义的永久链接")


 返回在 value 处计算的逆累积密度/质量函数。


 Parameters


**值** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) –


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution.log_prob)[¶](#torch.distributions.distribution.Distribution.log_prob "此定义的永久链接")


 返回在 value 处计算的概率密度/质量函数的对数。


 Parameters


**值** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) –


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


*财产*


 意思是 *：


[Tensor](tensors.html#torch.Tensor "torch.Tensor")*[¶](#torch.distributions.distribution.Distribution.mean "此定义的永久链接")


 返回分布的平均值。


*财产*


 模式 *：


[Tensor](tensors.html#torch.Tensor "torch.Tensor")*[¶](#torch.distributions.distribution.Distribution.mode "此定义的永久链接")


 返回分布模式。


 困惑


 ( ) [[source]](_modules/torch/distributions/distribution.html#Distribution.perplexity)[¶](#torch.distributions.distribution.Distribution.perplexity "此定义的永久链接")


 返回分布的困惑度，按batch_shape 进行批量处理。


 退货


 形状batch_shape 的tensor。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution.rsample)[¶](#torch.distributions.distribution.Distribution.rsample "此定义的永久链接")


 如果分布参数是批处理的，则生成样本形状重新参数化样本或样本样本一批重新参数化样本。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution.sample)[¶](#torch.distributions.distribution.Distribution.sample "此定义的永久链接")


 如果分布参数是批处理的，则生成样本形状的样本或样本形状的批次样本。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 样本_n


 ( *n
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution.sample_n)[¶](#torch.distributions.distribution.Distribution.sample_n "此定义的永久链接")


 如果分布参数是批处理的，则生成 n 个样本或 n 批样本。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


*静止的*


 设置_默认_验证_args


 ( *value
* ) [[source]](_modules/torch/distributions/distribution.html#Distribution.set_default_validate_args)[¶](#torch.distributions.distribution.Distribution.set_default_validate_args "此定义的永久链接")


 设置是否启用或禁用验证。


 默认行为模仿 Python 的 `assert` 语句：默认情况下验证处于打开状态，但如果 Python 在优化模式下运行(通过 `python -O` )，则验证被禁用。验证可能会很昂贵，因此您可能希望在模型运行后将其禁用。


 Parameters


**value** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否启用验证。


*财产*


 标准设备*：


[Tensor](tensors.html#torch.Tensor "torch.Tensor")*[¶](#torch.distributions.distribution.Distribution.stddev "此定义的永久链接")


 返回分布的标准差。


*财产*


 支持 *：


[可选](https://docs.python.org/3/library/typing.html#typing.Optional“(在 Python v3.12 中)”)


 [ [任何](https://docs.python.org/3/library/typing.html#typing.Any“(Python v3.12)”)


 ]*[¶](#torch.distributions.distribution.Distribution.support"此定义的永久链接")


 返回一个代表此发行版支持的 [`Constraint`](#torch.distributions.constraints.Constraint "torch.distributions.constraints.Constraint") 对象。


*财产*


 方差*：


[Tensor](tensors.html#torch.Tensor "torch.Tensor")*[¶](#torch.distributions.distribution.Distribution.variance "此定义的永久链接")


 返回分布的方差。


## ExponentialFamily [¶](#exponentialfamily "此标题的永久链接")


*班级*


 torch.distributions.exp_family。


 指数族


 ( *batch_shape



 =
 


 torch.Size([])
* , *event_shape



 =
 


 torch.Size([])
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/exp_family.html#ExponentialFamily)[¶](#torch.distributions.exp_family.ExponentialFamily "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 ExponentialFamily 是属于指数族的概率分布的抽象基类，其概率质量/密度函数的形式定义如下


 p
 

 F
 


 ( x ; θ ) = exp ⁡ ( ⟨ t ( x ) , θ ⟩ − F ( θ ) 
+ k ( x ) )


 p_{F}(x; 	heta) = \exp(\langle t(x), 	heta
angle 
- F(	heta) 
+ k(x))



 p
 




 F
 


 ​
 


 (  X  ;



 θ
 

 )
 



 =
 


 exp (⟨ t ( x ) ,



 θ
 

 ⟩
 



 −
 


 F(i)



 +
 


 k(x))



 where
 



 θ
 


 θ


 θ
 


 表示自然参数，


 t(x)


 t(x)
 


 t(x)


 表示充分统计量，


 F(i)


 F(θ)


 F(i)


 是给定族的对数归一化函数，


 k(x)


 k(x)
 


 k(x)


 是载体测量。




!!! note "笔记"

    该类是 Distribution 类和属于指数族的分布之间的中介，主要用于检查.entropy() 和分析 KLdivergence 方法的正确性。我们使用此类来使用 AD 框架和 Bregman 散度来计算熵和 KL 散度(由 Frank Nielsen 和 Richard Nock，Entropies and Cross-entropies of Exponential Families 提供)。


 熵


 ( ) [[source]](_modules/torch/distributions/exp_family.html#ExponentialFamily.entropy)[¶](#torch.distributions.exp_family.ExponentialFamily.entropy "此定义的永久链接")


 使用对数归一化器的布雷格曼散度计算熵的方法。


## 伯努利 [¶](#bernoulli"此标题的永久链接")


*班级*


 torch.distributions.bernoulli。


 伯努利


 (*问题



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/bernoulli.html#Bernoulli)[¶](#torch.distributions.bernoulli.Bernoulli "此定义的永久链接")


 基础： [`ExponentialFamily`](#torch.distributions.exp_family.ExponentialFamily "torch.distributions.exp_family.ExponentialFamily")


 创建由 [`probs`](#torch.distributions.bernoulli.Bernoulli.probs "torch.distributions.bernoulli.Bernoulli.probs") 或 [`logits`](#torch.distributions.bernoulli.Bernoulli.probs") 参数化的伯努利分布。 logits“torch.distributions.bernoulli.Bernoulli.logits”)(但不是两者)。


 样本是二进制的(0 或 1)。他们以 p 的概率取值 1，以 1 
- p 的概率取 0。


 例子：


```
>>> m = Bernoulli(torch.tensor([0.3]))
>>> m.sample()  # 30% chance 1; 70% chance 0
tensor([ 0.])

```


 参数 
* **probs** ( *Number
* *,
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 采样 1
* **logits** ( *Number
* *,
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 采样 1 的对数几率


 arg_constraints *=


 {'logits': Real(), 'probs': Interval(lower_bound=0.0, upper_bound=1.0)}*[¶](#torch.distributions.bernoulli.Bernoulli.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/bernoulli.html#Bernoulli.entropy)[¶](#torch.distributions.bernoulli.Bernoulli.entropy "此定义的永久链接")


 枚举_支持


 ( *扩张



 =
 


 True
* ) [[source]](_modules/torch/distributions/bernoulli.html#Bernoulli.enumerate_support)[¶](#torch.distributions.bernoulli.Bernoulli.enumerate_support "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/bernoulli.html#Bernoulli.expand)[¶](#torch.distributions.bernoulli.Bernoulli.expand "此定义的永久链接")


 有_枚举_支持 *=


 True*[¶](#torch.distributions.bernoulli.Bernoulli.has_enumerate_support "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/bernoulli.html#Bernoulli.log_prob)[¶](#torch.distributions.bernoulli.Bernoulli.log_prob "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.bernoulli.Bernoulli.logits "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.bernoulli.Bernoulli.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.bernoulli.Bernoulli.mode "此定义的永久链接")


*财产*


 param_shape [¶](#torch.distributions.bernoulli.Bernoulli.param_shape "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.bernoulli.Bernoulli.probs "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/bernoulli.html#Bernoulli.sample)[¶](#torch.distributions.bernoulli.Bernoulli.sample "此定义的永久链接")


 支持*=


 Boolean()*[¶](#torch.distributions.bernoulli.Bernoulli.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.bernoulli.Bernoulli.variance "此定义的永久链接")


## Beta [¶](#beta "此标题的永久链接")


*班级*


 torch.distributions.beta。



 Beta
 


 (*浓度1*，*浓度0*，*验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/beta.html#Beta)[¶](#torch.distributions.beta.Beta "此定义的永久链接")


 基础： [`ExponentialFamily`](#torch.distributions.exp_family.ExponentialFamily "torch.distributions.exp_family.ExponentialFamily")


 Beta 分布由 [`concentration1`](#torch.distributions.beta.Beta.concentration1 "torch.distributions.beta.Beta.concentration1") 和 [`concentration0`](#torch.distributions.beta.Beta.concentration0 " 参数化torch.distributions.beta.Beta.concentration0") 。


 例子：


```
>>> m = Beta(torch.tensor([0.5]), torch.tensor([0.5]))
>>> m.sample()  # Beta distributed with concentration concentration1 and concentration0
tensor([ 0.1046])

```


 参数 
* **浓度1** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的第一个浓度参数(通常称为 alpha)
* **浓度0** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 第二个浓度参数分布(通常称为 beta)


 arg_constraints *=


 {'concentration0': GreaterThan(lower_bound=0.0), 'concentration1': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.beta.Beta.arg_constraints "此定义的永久链接")


*财产*


 浓度0 [¶](#torch.distributions.beta.Beta.concentration0"此定义的永久链接")


*财产*


 浓度1 [¶](#torch.distributions.beta.Beta.concentration1"此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/beta.html#Beta.entropy)[¶](#torch.distributions.beta.Beta.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/beta.html#Beta.expand)[¶](#torch.distributions.beta.Beta.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.beta.Beta.has_rsample "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/beta.html#Beta.log_prob)[¶](#torch.distributions.beta.Beta.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.beta.Beta.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.beta.Beta.mode"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 ()
* ) [[source]](_modules/torch/distributions/beta.html#Beta.rsample)[¶](#torch.distributions.beta.Beta.rsample "此定义的永久链接")


 支持*=


 间隔(lower_bound=0.0, upper_bound=1.0)*[¶](#torch.distributions.beta.Beta.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.beta.Beta.variance "此定义的永久链接")


## 二项式 [¶](#binomial "此标题的永久链接")


*班级*


 torch.distributions.binomial。


 二项式


 ( *总数



 =
 


 1
* , 
* 问题



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/binomial.html#Binomial)[¶](#torch.distributions.binomial.Binomial "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建由 `total_count` 和 [`probs`](#torch.distributions.binomial.Binomial.probs "torch.distributions.binomial.Binomial.probs") 或 [`logits`](#torch. distributions.binomial.Binomial.logits“torch.distributions.binomial.Binomial.logits”)(但不是两者)。 `total_count` 必须可以使用 [`probs`](#torch.distributions.binomial.Binomial.probs "torch.distributions.binomial.Binomial.probs") /[`logits`](#torch.distributions.binomial. Binomial.logits "torch.distributions.binomial.Binomial.logits") 。


 例子：


```
>>> m = Binomial(100, torch.tensor([0 , .2, .8, 1]))
>>> x = m.sample()
tensor([ 0., 22., 71., 100.])

>>> m = Binomial(torch.tensor([[5.], [10.]]), torch.tensor([0.5, 0.8]))
>>> x = m.sample()
tensor([[ 4., 5.],
 [ 7., 6.]])

```


 参数 
* **total_count** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*或
* [
* Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 伯努利试验次数
* **probs** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 事件概率
* **logits** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 事件日志赔率


 arg_constraints *=


 {'logits': Real(), 'probs': 间隔(lower_bound=0.0, upper_bound=1.0), 'total_count': IntegerGreaterThan(lower_bound=0)}*[¶](#torch.distributions.binomial.Binomial.arg_constraints"此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/binomial.html#Binomial.entropy)[¶](#torch.distributions.binomial.Binomial.entropy "此定义的永久链接")


 枚举_支持


 ( *扩张



 =
 


 True
* ) [[source]](_modules/torch/distributions/binomial.html#Binomial.enumerate_support)[¶](#torch.distributions.binomial.Binomial.enumerate_support "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/binomial.html#Binomial.expand)[¶](#torch.distributions.binomial.Binomial.expand "此定义的永久链接")


 有_枚举_支持 *=


 True*[¶](#torch.distributions.binomial.Binomial.has_enumerate_support "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/binomial.html#Binomial.log_prob)[¶](#torch.distributions.binomial.Binomial.log_prob "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.binomial.Binomial.logits "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.binomial.Binomial.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.binomial.Binomial.mode "此定义的永久链接")


*财产*


 param_shape [¶](#torch.distributions.binomial.Binomial.param_shape "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.binomial.Binomial.probs"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/binomial.html#Binomial.sample)[¶](#torch.distributions.binomial.Binomial.sample "此定义的永久链接")


*财产*


 支持[¶](#torch.distributions.binomial.Binomial.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.binomial.Binomial.variance "此定义的永久链接")


## 分类 [¶](#categorical"此标题的永久链接")


*班级*


 torch.distributions.categorical。


 分类的


 (*问题



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/categorical.html#Categorical)[¶](#torch.distributions.categorical.Categorical"此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建由 [`probs`](#torch.distributions.categorical.Categorical.probs "torch.distributions.categorical.Categorical.probs") 或 [`logits`](#torch.distributions.categorical.Categorical) 参数化的分类分布.logits“torch.distributions.categorical.Categorical.logits”)(但不是两者)。




!!! note "笔记"

    它相当于 [`torch.multinomial()`](generated/torch.multinomial.html#torch.multinomial "torch.multinomial") 样本的分布。


 样本是来自的整数


 { 0 , … , K − 1 }


 \{0,\l点,K-1\}


 { 0 ,



 …
 



 ,
 



 K
 



 −
 




 1
 

 }
 


 其中 K 是 `probs.size(-1)` 。


 如果 probs 是长度为 K 的一维，则每个元素都是在该索引处对类进行采样的相对概率。


 如果 probs 是 N 维，则前 N-1 维被视为一批相对概率向量。




!!! note "笔记"

    probs 参数必须是非负的、有限的并且具有非零和，并且它将沿着最后一个维度标准化为和为 1。 [`probs`](#torch.distributions.categorical.Categorical.probs "torch.distributions.categorical.Categorical.probs") 将返回此归一化值。logits 参数将被解释为非归一化对数概率，因此可以是任何实数。它同样会被归一化，以便沿最后一个维度得到的概率总和为 1。 [`logits`](#torch.distributions.categorical.Categorical.logits "torch.distributions.categorical.Categorical.logits") 将返回此标准化值。


 另请参阅：[`torch.multinomial()`](generated/torch.multinomial.html#torch.multinomial "torch.multinomial")


 例子：


```
>>> m = Categorical(torch.tensor([ 0.25, 0.25, 0.25, 0.25 ]))
>>> m.sample()  # equal probability of 0, 1, 2, 3
tensor(3)

```


 参数 
* **probs** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 事件概率
* **logits** ( [*Tensor*](tensors.html#torch.tensor "torch.Tensor") ) – 事件日志概率(未归一化)


 arg_constraints *=


 {'logits': IndependentConstraint(Real(), 1), 'probs': Simplex()}*[¶](#torch.distributions.categorical.Categorical.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/categorical.html#Categorical.entropy)[¶](#torch.distributions.categorical.Categorical.entropy "此定义的永久链接")


 枚举_支持


 ( *扩张



 =
 


 True
* ) [[source]](_modules/torch/distributions/categorical.html#Categorical.enumerate_support)[¶](#torch.distributions.categorical.Categorical.enumerate_support "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/categorical.html#Categorical.expand)[¶](#torch.distributions.categorical.Categorical.expand "此定义的永久链接")


 有_枚举_支持 *=


 True*[¶](#torch.distributions.categorical.Categorical.has_enumerate_support "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/categorical.html#Categorical.log_prob)[¶](#torch.distributions.categorical.Categorical.log_prob "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.categorical.Categorical.logits"此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.categorical.Categorical.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.categorical.Categorical.mode"此定义的永久链接")


*财产*


 param_shape [¶](#torch.distributions.categorical.Categorical.param_shape "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.categorical.Categorical.probs"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/categorical.html#Categorical.sample)[¶](#torch.distributions.categorical.Categorical.sample "此定义的永久链接")


*财产*


 支持[¶](#torch.distributions.categorical.Categorical.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.categorical.Categorical.variance"此定义的永久链接")


## 柯西 [¶](#cauchy "此标题的永久链接")


*班级*


 torch.distributions.cauchy。


 柯西


 ( *loc
* 、 *scale
* 、 *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/cauchy.html#Cauchy)[¶](#torch.distributions.cauchy.Cauchy"此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 来自柯西(洛伦兹)分布的样本。均值为 0 的独立正态分布随机变量的比率分布遵循柯西分布。


 例子：


```
>>> m = Cauchy(torch.tensor([0.0]), torch.tensor([1.0]))
>>> m.sample()  # sample from a Cauchy distribution with loc=0 and scale=1
tensor([ 2.3214])

```


 参数 
* **loc** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的众数或中位数。
* **scale** ( [*float*](https://docs.python.org/3/library) /functions.html#float "(in Python v3.12)")*or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 半最大值的半宽度。


 arg_constraints *=


 {'loc': Real(), 'scale': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.cauchy.Cauchy.arg_constraints "此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/cauchy.html#Cauchy.cdf)[¶](#torch.distributions.cauchy.Cauchy.cdf "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/cauchy.html#Cauchy.entropy)[¶](#torch.distributions.cauchy.Cauchy.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/cauchy.html#Cauchy.expand)[¶](#torch.distributions.cauchy.Cauchy.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.cauchy.Cauchy.has_rsample "此定义的永久链接")


 icdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/cauchy.html#Cauchy.icdf)[¶](#torch.distributions.cauchy.Cauchy.icdf "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/cauchy.html#Cauchy.log_prob)[¶](#torch.distributions.cauchy.Cauchy.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.cauchy.Cauchy.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.cauchy.Cauchy.mode"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/cauchy.html#Cauchy.rsample)[¶](#torch.distributions.cauchy.Cauchy.rsample "此定义的永久链接")


 支持*=


 Real()*[¶](#torch.distributions.cauchy.Cauchy.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.cauchy.Cauchy.variance "此定义的永久链接")


## Chi2 [¶](#chi2 "此标题的固定链接")


*班级*


 torch.distributions.chi2。



 Chi2
 


 ( *df
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/chi2.html#Chi2)[¶](#torch.distributions.chi2.Chi2 "此定义的永久链接")


 基础： [`Gamma`](#torch.distributions.gamma.Gamma "torch.distributions.gamma.Gamma")


 创建由形状参数 [`df`](#torch.distributions.chi2.Chi2.df "torch.distributions.chi2.Chi2.df") 参数化的卡方分布。这完全等同于 `Gamma(alpha=0.5) *df, beta=0.5)`


 例子：


```
>>> m = Chi2(torch.tensor([1.0]))
>>> m.sample()  # Chi2 distributed with shape df=1
tensor([ 0.1046])

```


 Parameters


**df** ( [*float*](https://docs.python.org/3/library/functions.html#float "(在 Python v3.12 中)")*或
* [*Tensor*]( tensors.html#torch.Tensor "torch.Tensor") ) – 分布的形状参数


 arg_constraints *=


 {'df': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.chi2.Chi2.arg_constraints "此定义的永久链接")


*财产*


 df [¶](#torch.distributions.chi2.Chi2.df"此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/chi2.html#Chi2.expand)[¶](#torch.distributions.chi2.Chi2.expand "此定义的永久链接")


## ContinuousBernoulli [¶](#continuousbernoulli"此标题的永久链接")


*班级*


 torch.distributions.Continous_bernoulli。


 连续伯努利


 (*问题



 =
 


 无
* , *logits



 =
 


 无
* , *限制



 =
 


 (0.499, 0.501)
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/continuous_bernoulli.html#ContinouslyBernoulli)[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli "此定义的永久链接")


 基础： [`ExponentialFamily`](#torch.distributions.exp_family.ExponentialFamily "torch.distributions.exp_family.ExponentialFamily")


 创建由 [`probs`](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.probs "torch.distributions.continuous_bernoulli.ContinouslyBernoulli.probs") 或 [`logits`](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli) 参数化的连续伯努利分布.logits“torch.distributions.continuous_bernoulli.ContinouslyBernoulli.logits”)(但不是两者)。


 该分布在 [0, 1] 中得到支持，并通过“probs”(in(0,1))或“logits”(实值)进行参数化。请注意，与伯努利不同，“probs”不对应于概率，“logits”不对应于对数赔率，但由于与伯努利相似，因此使用相同的名称。有关更多详细信息，请参阅[1]。


 例子：


```
>>> m = ContinuousBernoulli(torch.tensor([0.3]))
>>> m.sample()
tensor([ 0.2538])

```


 参数 
* **probs** ( *Number
* *,
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – (0,1) 值参数
* **logits** ( 
* Number
* *,
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – sigmoid 与 'probs' 匹配的实值参数


 [1] 连续伯努利：修复变分自动编码器中普遍存在的错误，Loaiza-Ganem G 和 Cunningham JP，NeurIPS 2019。<https://arxiv.org/abs/1907.06845>


 arg_constraints *=


 {'logits': Real(), 'probs': Interval(lower_bound=0.0, upper_bound=1.0)}*[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.arg_constraints "此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/continuous_bernoulli.html#ContinouslyBernoulli.cdf)[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.cdf "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/continuous_bernoulli.html#ContinouslyBernoulli.entropy)[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/continuous_bernoulli.html#ContinouslyBernoulli.expand)[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.has_rsample "此定义的永久链接")


 icdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/continuous_bernoulli.html#ContinouslyBernoulli.icdf)[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.icdf "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/continuous_bernoulli.html#ContinouslyBernoulli.log_prob)[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.log_prob "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.logits "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.mean"此定义的永久链接")


*财产*


 param_shape [¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.param_shape "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.probs"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/continuous_bernoulli.html#ContinouslyBernoulli.rsample)[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.rsample "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/continuous_bernoulli.html#ContinouslyBernoulli.sample)[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.sample "此定义的永久链接")


*财产*


 stddev [¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.stddev"此定义的永久链接")


 支持*=


 间隔(lower_bound=0.0, upper_bound=1.0)*[¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.continuous_bernoulli.ContinouslyBernoulli.variance "此定义的永久链接")


## 狄利克雷 [¶](#dirichlet "此标题的永久链接")


*班级*


 torch.distributions.dirichlet。


 狄利克雷


 ( *浓度
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/dirichlet.html#Dirichlet)[¶](#torch.distributions.dirichlet.Dirichlet "此定义的永久链接")


 基础： [`ExponentialFamily`](#torch.distributions.exp_family.ExponentialFamily "torch.distributions.exp_family.ExponentialFamily")


 创建由浓度 `concentration` 参数化的狄利克雷分布。


 例子：


```
>>> m = Dirichlet(torch.tensor([0.5, 0.5]))
>>> m.sample()  # Dirichlet distributed with concentration [0.5, 0.5]
tensor([ 0.1046, 0.8954])

```


 Parameters


**浓度** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的浓度参数(通常称为 alpha)


 arg_constraints *=


 {'concentration': IndependentConstraint(GreaterThan(lower_bound=0.0), 1)}*[¶](#torch.distributions.dirichlet.Dirichlet.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/dirichlet.html#Dirichlet.entropy)[¶](#torch.distributions.dirichlet.Dirichlet.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/dirichlet.html#Dirichlet.expand)[¶](#torch.distributions.dirichlet.Dirichlet.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.dirichlet.Dirichlet.has_rsample "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/dirichlet.html#Dirichlet.log_prob)[¶](#torch.distributions.dirichlet.Dirichlet.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.dirichlet.Dirichlet.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.dirichlet.Dirichlet.mode "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 ()
* ) [[source]](_modules/torch/distributions/dirichlet.html#Dirichlet.rsample)[¶](#torch.distributions.dirichlet.Dirichlet.rsample "此定义的永久链接")


 支持*=


 Simplex()*[¶](#torch.distributions.dirichlet.Dirichlet.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.dirichlet.Dirichlet.variance "此定义的永久链接")


## 指数 [¶](#exponential "此标题的永久链接")


*班级*


 torch.distributions.exponential。


 指数


 ( *rate
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/exponential.html#Exponential)[¶](#torch.distributions.exponential.Exponential "此定义的永久链接")


 基础： [`ExponentialFamily`](#torch.distributions.exp_family.ExponentialFamily "torch.distributions.exp_family.ExponentialFamily")


 创建由 `rate` 参数化的指数分布。


 例子：


```
>>> m = Exponential(torch.tensor([1.0]))
>>> m.sample()  # Exponential distributed with rate=1
tensor([ 0.1046])

```


 Parameters


**rate** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor*]( tensors.html#torch.Tensor "torch.Tensor") ) – 比率 = 1 /分布范围


 arg_constraints *=


 {'rate': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.exponential.Exponential.arg_constraints "此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/exponential.html#Exponential.cdf)[¶](#torch.distributions.exponential.Exponential.cdf "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/exponential.html#Exponential.entropy)[¶](#torch.distributions.exponential.Exponential.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/exponential.html#Exponential.expand)[¶](#torch.distributions.exponential.Exponential.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.exponential.Exponential.has_rsample "此定义的永久链接")


 icdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/exponential.html#Exponential.icdf)[¶](#torch.distributions.exponential.Exponential.icdf "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/exponential.html#Exponential.log_prob)[¶](#torch.distributions.exponential.Exponential.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.exponential.Exponential.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.exponential.Exponential.mode"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/exponential.html#Exponential.rsample)[¶](#torch.distributions.exponential.Exponential.rsample "此定义的永久链接")


*财产*


 stddev [¶](#torch.distributions.exponential.Exponential.stddev"此定义的永久链接")


 支持*=


 GreaterThanEq(lower_bound=0.0)*[¶](#torch.distributions.exponential.Exponential.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.exponential.Exponential.variance"此定义的永久链接")


## FisherSnedecor [¶](#fishersnedecor "此标题的永久链接")


*班级*


 torch.distributions.fishersnedecor。


 费舍尔斯内装饰公司


 ( *df1
* 、 *df2
* 、 *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/fishersnedecor.html#FisherSnedecor)[¶](#torch.distributions.fishersnedecor.FisherSnedecor "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建由 `df1` 和 `df2` 参数化的 Fisher-Snedecor 分布。


 例子：


```
>>> m = FisherSnedecor(torch.tensor([1.0]), torch.tensor([2.0]))
>>> m.sample()  # Fisher-Snedecor-distributed with df1=1 and df2=2
tensor([ 0.2453])

```


 参数 
* **df1** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 自由度参数 1
* **df2** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 自由度参数 2


 arg_constraints *=


 {'df1': GreaterThan(lower_bound=0.0), 'df2': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.fishersnedecor.FisherSnedecor.arg_constraints "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/fishersnedecor.html#FisherSnedecor.expand)[¶](#torch.distributions.fishersnedecor.FisherSnedecor.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.fishersnedecor.FisherSnedecor.has_rsample "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/fishersnedecor.html#FisherSnedecor.log_prob)[¶](#torch.distributions.fishersnedecor.FisherSnedecor.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.fishersnedecor.FisherSnedecor.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.fishersnedecor.FisherSnedecor.mode"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/fishersnedecor.html#FisherSnedecor.rsample)[¶](#torch.distributions.fishersnedecor.FisherSnedecor.rsample "此定义的永久链接")


 支持*=


 GreaterThan(lower_bound=0.0)*[¶](#torch.distributions.fishersnedecor.FisherSnedecor.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.fishersnedecor.FisherSnedecor.variance "此定义的永久链接")


## Gamma [¶](#gamma "此标题的永久链接")


*班级*


 torch.distributions.gamma。



 Gamma
 


 (*浓度*、*速率*、*验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/gamma.html#Gamma)[¶](#torch.distributions.gamma.Gamma "此定义的永久链接")


 基础： [`ExponentialFamily`](#torch.distributions.exp_family.ExponentialFamily "torch.distributions.exp_family.ExponentialFamily")


 创建由形状“浓度”和“速率”参数化的伽玛分布。


 例子：


```
>>> m = Gamma(torch.tensor([1.0]), torch.tensor([1.0]))
>>> m.sample()  # Gamma distributed with concentration=1 and rate=1
tensor([ 0.1046])

```


 参数 
* **浓度** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的形状参数(通常称为 alpha)
* **rate** ( [*float*](https://docs.python. org/3/library/functions.html#float "(Python v3.12)")*或
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 速率 = 1 /比例分布(通常称为 beta)


 arg_constraints *=


 {'concentration': GreaterThan(lower_bound=0.0), 'rate': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.gamma.Gamma.arg_constraints "此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/gamma.html#Gamma.cdf)[¶](#torch.distributions.gamma.Gamma.cdf "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/gamma.html#Gamma.entropy)[¶](#torch.distributions.gamma.Gamma.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/gamma.html#Gamma.expand)[¶](#torch.distributions.gamma.Gamma.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.gamma.Gamma.has_rsample "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/gamma.html#Gamma.log_prob)[¶](#torch.distributions.gamma.Gamma.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.gamma.Gamma.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.gamma.Gamma.mode "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/gamma.html#Gamma.rsample)[¶](#torch.distributions.gamma.Gamma.rsample "此定义的永久链接")


 支持*=


 GreaterThanEq(lower_bound=0.0)*[¶](#torch.distributions.gamma.Gamma.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.gamma.Gamma.variance "此定义的永久链接")


## 几何 [¶](#geometric "此标题的固定链接")


*班级*


 torch.distributions.geometric.


 几何的


 (*问题



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/geometric.html#Geometric)[¶](#torch.distributions.geometric.Geometric "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建由 [`probs`](#torch.distributions.geometric.Geometric.probs "torch.distributions.geometric.Geometric.probs") 参数化的几何分布，其中 [`probs`](#torch.distributions.geometric.Geometric.probs "torch.distributions.geometric.Geometric.probs") 是伯努利试验成功的概率。它表示在


 k+1


 k + 1
 


 k
 



 +
 




 1
 


 第一次伯努利试验



 k
 


 k
 


 k
 


 在看到成功之前，尝试失败了。


 样本是非负整数 [0,


 信息⁡


 \inf
 


 in
 
 f
 


 ).
 


 例子：


```
>>> m = Geometric(torch.tensor([0.3]))
>>> m.sample()  # underlying Bernoulli has 30% chance 1; 70% chance 0
tensor([ 2.])

```


 参数 
* **probs** ( *Number
* *,
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 采样 1 的概率。必须在 (0, 1]
* **logits** ( *Number
* *,
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) 范围内 – 采样 1 的对数几率。


 arg_constraints *=


 {'logits': Real(), 'probs': Interval(lower_bound=0.0, upper_bound=1.0)}*[¶](#torch.distributions.geometric.Geometric.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/geometric.html#Geometric.entropy)[¶](#torch.distributions.geometric.Geometric.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/geometric.html#Geometric.expand)[¶](#torch.distributions.geometric.Geometric.expand "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/geometric.html#Geometric.log_prob)[¶](#torch.distributions.geometric.Geometric.log_prob "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.geometric.Geometric.logits "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.geometric.Geometric.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.geometric.Geometric.mode "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.geometric.Geometric.probs"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/geometric.html#Geometric.sample)[¶](#torch.distributions.geometric.Geometric.sample "此定义的永久链接")


 支持*=


 IntegerGreaterThan(lower_bound=0)*[¶](#torch.distributions.geometric.Geometric.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.geometric.Geometric.variance "此定义的永久链接")


## Gumbel [¶](#gumbel "此标题的永久链接")


*班级*


 torch.distributions.gumbel。


 甘贝尔


 ( *loc
* 、 *scale
* 、 *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/gumbel.html#Gumbel)[¶](#torch.distributions.gumbel.Gumbel "此定义的永久链接")


 基础： [`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution")


 来自 Gumbel 分布的样本。


 例子：


```
>>> m = Gumbel(torch.tensor([1.0]), torch.tensor([2.0]))
>>> m.sample()  # sample from Gumbel distribution with loc=1, scale=2
tensor([ 1.0124])

```


 参数 
* **loc** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的位置参数
* **scale** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的尺度参数


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {'loc': Real(), 'scale': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.gumbel.Gumbel.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/gumbel.html#Gumbel.entropy)[¶](#torch.distributions.gumbel.Gumbel.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/gumbel.html#Gumbel.expand)[¶](#torch.distributions.gumbel.Gumbel.expand "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/gumbel.html#Gumbel.log_prob)[¶](#torch.distributions.gumbel.Gumbel.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.gumbel.Gumbel.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.gumbel.Gumbel.mode"此定义的永久链接")


*财产*


 stddev [¶](#torch.distributions.gumbel.Gumbel.stddev"此定义的永久链接")


 支持*=


 Real()*[¶](#torch.distributions.gumbel.Gumbel.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.gumbel.Gumbel.variance "此定义的永久链接")


## HalfCauchy [¶](#halfcauchy"此标题的固定链接")


*班级*


 torch.distributions.half_cauchy。


 哈夫柯西


 ( *scale
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/half_cauchy.html#HalfCauchy)[¶](#torch.distributions.half_cauchy.HalfCauchy "此定义的永久链接")


 基础： [`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution")


 创建按尺度参数化的半柯西分布，其中：


```
X ~ Cauchy(0, scale)
Y = |X| ~ HalfCauchy(scale)

```


 例子：


```
>>> m = HalfCauchy(torch.tensor([1.0]))
>>> m.sample()  # half-cauchy distributed with scale=1
tensor([ 2.3214])

```


 Parameters


**scale** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor*]( tensors.html#torch.Tensor "torch.Tensor") ) – 完整柯西分布的尺度


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {'scale': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.half_cauchy.HalfCauchy.arg_constraints"此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/half_cauchy.html#HalfCauchy.cdf)[¶](#torch.distributions.half_cauchy.HalfCauchy.cdf "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/half_cauchy.html#HalfCauchy.entropy)[¶](#torch.distributions.half_cauchy.HalfCauchy.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/half_cauchy.html#HalfCauchy.expand)[¶](#torch.distributions.half_cauchy.HalfCauchy.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.half_cauchy.HalfCauchy.has_rsample "此定义的永久链接")


 icdf
 


 ( *prob
* ) [[source]](_modules/torch/distributions/half_cauchy.html#HalfCauchy.icdf)[¶](#torch.distributions.half_cauchy.HalfCauchy.icdf "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/half_cauchy.html#HalfCauchy.log_prob)[¶](#torch.distributions.half_cauchy.HalfCauchy.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.half_cauchy.HalfCauchy.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.half_cauchy.HalfCauchy.mode"此定义的永久链接")


*财产*


 scale [¶](#torch.distributions.half_cauchy.HalfCauchy.scale"此定义的永久链接")


 支持*=


 GreaterThanEq(lower_bound=0.0)*[¶](#torch.distributions.half_cauchy.HalfCauchy.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.half_cauchy.HalfCauchy.variance"此定义的永久链接")


## HalfNormal [¶](#halfnormal "此标题的永久链接")


*班级*


 torch.distributions.half_normal。


 半正常


 ( *scale
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/half_normal.html#HalfNormal)[¶](#torch.distributions.half_normal.HalfNormal "此定义的永久链接")


 基础： [`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution")


 创建按比例参数化的半正态分布，其中：


```
X ~ Normal(0, scale)
Y = |X| ~ HalfNormal(scale)

```


 例子：


```
>>> m = HalfNormal(torch.tensor([1.0]))
>>> m.sample()  # half-normal distributed with scale=1
tensor([ 0.1046])

```


 Parameters


**scale** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor*]( tensors.html#torch.Tensor "torch.Tensor") ) – 完整正态分布的尺度


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {'scale': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.half_normal.HalfNormal.arg_constraints"此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/half_normal.html#HalfNormal.cdf)[¶](#torch.distributions.half_normal.HalfNormal.cdf "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/half_normal.html#HalfNormal.entropy)[¶](#torch.distributions.half_normal.HalfNormal.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/half_normal.html#HalfNormal.expand)[¶](#torch.distributions.half_normal.HalfNormal.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.half_normal.HalfNormal.has_rsample "此定义的永久链接")


 icdf
 


 ( *prob
* ) [[source]](_modules/torch/distributions/half_normal.html#HalfNormal.icdf)[¶](#torch.distributions.half_normal.HalfNormal.icdf "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/half_normal.html#HalfNormal.log_prob)[¶](#torch.distributions.half_normal.HalfNormal.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.half_normal.HalfNormal.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.half_normal.HalfNormal.mode"此定义的永久链接")


*财产*


 比例[¶](#torch.distributions.half_normal.HalfNormal.scale"此定义的永久链接")


 支持*=


 GreaterThanEq(lower_bound=0.0)*[¶](#torch.distributions.half_normal.HalfNormal.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.half_normal.HalfNormal.variance "此定义的永久链接")


## 独立[¶](#独立"此标题的永久链接")


*班级*


 torch.distributions.independent。


 独立的


 ( *base_distribution
* , *重新解释_batch_ndims
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/independent.html#Independent)[¶](#torch.distributions.independent.Independent "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 将分布的某些批量暗淡重新解释为事件暗淡。


 这主要用于更改 [`log_prob()`](#torch.distributions.independent.Independent.log_prob "torch.distributions.independent.Independent.log_prob") 结果的形状。例如，要创建与多元正态分布形状相同的对角正态分布(因此它们可以互换)，您可以：


```
>>> from torch.distributions.multivariate_normal import MultivariateNormal
>>> from torch.distributions.normal import Normal
>>> loc = torch.zeros(3)
>>> scale = torch.ones(3)
>>> mvn = MultivariateNormal(loc, scale_tril=torch.diag(scale))
>>> [mvn.batch_shape, mvn.event_shape]
[torch.Size([]), torch.Size([3])]
>>> normal = Normal(loc, scale)
>>> [normal.batch_shape, normal.event_shape]
[torch.Size([3]), torch.Size([])]
>>> diagn = Independent(normal, 1)
>>> [diagn.batch_shape, diagn.event_shape]
[torch.Size([]), torch.Size([3])]

```


 参数 
* **base_distribution** ( [*torch.distributions.distribution.Distribution*](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution") ) – 基本分布
* **重新解释_batch _ndims** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 解释为事件的批量 Dims 数量变暗


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {}*[¶](#torch.distributions.independent.Independent.arg_constraints"此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/independent.html#Independent.entropy)[¶](#torch.distributions.independent.Independent.entropy "此定义的永久链接")


 枚举_支持


 ( *扩张



 =
 


 True
* ) [[source]](_modules/torch/distributions/independent.html#Independent.enumerate_support)[¶](#torch.distributions.independent.Independent.enumerate_support "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/independent.html#Independent.expand)[¶](#torch.distributions.independent.Independent.expand "此定义的永久链接")


*财产*


 has_enumerate_support [¶](#torch.distributions.independent.Independent.has_enumerate_support "此定义的永久链接")


*财产*


 has_rsample [¶](#torch.distributions.independent.Independent.has_rsample "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/independent.html#Independent.log_prob)[¶](#torch.distributions.independent.Independent.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.independent.Independent.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.independent.Independent.mode"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/independent.html#Independent.rsample)[¶](#torch.distributions.independent.Independent.rsample "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/independent.html#Independent.sample)[¶](#torch.distributions.independent.Independent.sample "此定义的永久链接")


*财产*


 支持[¶](#torch.distributions.independent.Independent.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.independent.Independent.variance "此定义的永久链接")


## Kumaraswamy [¶](#kumaraswamy"此标题的永久链接")


*班级*


 torch.distributions.kumaraswamy。


 库马拉斯瓦米


 (*浓度1*，*浓度0*，*验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/kumaraswamy.html#Kumaraswamy)[¶](#torch.distributions.kumaraswamy.Kumaraswamy"此定义的永久链接")


 基础： [`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution")


 来自 Kumaraswamy 分布的样本。


 例子：


```
>>> m = Kumaraswamy(torch.tensor([1.0]), torch.tensor([1.0]))
>>> m.sample()  # sample from a Kumaraswamy distribution with concentration alpha=1 and beta=1
tensor([ 0.1729])

```


 参数 
* **浓度1** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的第一个浓度参数(通常称为 alpha)
* **浓度0** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 第二个浓度参数分布(通常称为 beta)


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {'concentration0': GreaterThan(lower_bound=0.0), 'concentration1': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.kumaraswamy.Kumaraswamy.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/kumaraswamy.html#Kumaraswamy.entropy)[¶](#torch.distributions.kumaraswamy.Kumaraswamy.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/kumaraswamy.html#Kumaraswamy.expand)[¶](#torch.distributions.kumaraswamy.Kumaraswamy.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.kumaraswamy.Kumaraswamy.has_rsample "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.kumaraswamy.Kumaraswamy.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.kumaraswamy.Kumaraswamy.mode"此定义的永久链接")


 支持*=


 间隔(lower_bound=0.0, upper_bound=1.0)*[¶](#torch.distributions.kumaraswamy.Kumaraswamy.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.kumaraswamy.Kumaraswamy.variance"此定义的永久链接")


## LKJCholesky [¶](#lkjcholesky "此标题的永久链接")


*班级*


 torch.distributions.lkj_cholesky。


 LKJ乔尔斯基


 (*暗淡*，*浓度



 =
 


 1.0
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/lkj_cholesky.html#LKJCholesky)[¶](#torch.distributions.lkj_cholesky.LKJCholesky "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 相关矩阵较低 Cholesky 因子的 LKJ 分布。该分布由“浓度”参数控制



 η
 


 \eta
 


 η
 


 制作相关矩阵的概率



 M
 


 M
 


 M
 


 由 Cholesky 因子生成，与


 让 ⁡ ( M


 )
 


 h−1


 \det(M)^{\eta 
- 1}


 给(M


 )
 


 h−1


 。因此，当“浓度 == 1”时，我们在相关矩阵的 Choleskyfactors 上有一个均匀分布：


```
L ~ LKJCholesky(dim, concentration)
X = L @ L' ~ LKJCorr(dim, concentration)

```


 请注意，该分布对相关矩阵的 Cholesky 因子进行采样，而不是对相关矩阵本身进行采样，因此与 [1] 中 LKJCorr 分布的推导略有不同。对于采样，这使用 [1] 第 3 节中的洋葱方法。


 例子：


```
>>> l = LKJCholesky(3, 0.5)
>>> l.sample()  # l @ l.T is a sample of a correlation 3x3 matrix
tensor([[ 1.0000, 0.0000, 0.0000],
 [ 0.3516, 0.9361, 0.0000],
 [-0.1899, 0.4748, 0.8593]])

```


 参数 
* **维度** ( *dim
* ) – 矩阵的维度
* **浓度** ( [*float*](https://docs.python.org/3/library/functions.html#float " (在 Python v3.12 中)")*或
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的浓度/形状参数(通常称为 eta)


**参考**


 [1] 基于藤蔓和扩展洋葱法生成随机相关矩阵(2009)，Daniel Lewandowski，Dorota Kurowicka，Harry Joe。多元分析杂志。 100.10.1016/j.jmva.2009.04.008


 arg_constraints *=


 {'concentration': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.lkj_cholesky.LKJCholesky.arg_constraints"此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/lkj_cholesky.html#LKJCholesky.expand)[¶](#torch.distributions.lkj_cholesky.LKJCholesky.expand "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/lkj_cholesky.html#LKJCholesky.log_prob)[¶](#torch.distributions.lkj_cholesky.LKJCholesky.log_prob "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/lkj_cholesky.html#LKJCholesky.sample)[¶](#torch.distributions.lkj_cholesky.LKJCholesky.sample "此定义的永久链接")


 支持*=


 CorrCholesky()*[¶](#torch.distributions.lkj_cholesky.LKJCholesky.support "此定义的永久链接")


## 拉普拉斯 [¶](#laplace "此标题的永久链接")


*班级*


 torch.distributions.laplace。


 拉普拉斯


 ( *loc
* 、 *scale
* 、 *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/laplace.html#Laplace)[¶](#torch.distributions.laplace.Laplace"此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建由 `loc` 和 `scale` 参数化的拉普拉斯分布。


 例子：


```
>>> m = Laplace(torch.tensor([0.0]), torch.tensor([1.0]))
>>> m.sample()  # Laplace distributed with loc=0, scale=1
tensor([ 0.1046])

```


 参数 
* **loc** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 分布平均值
* **scale** ( [*float*](https://docs.python.org/3/library/functions. html#float "(in Python v3.12)")*or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 分布规模


 arg_constraints *=


 {'loc': Real(), 'scale': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.laplace.Laplace.arg_constraints "此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/laplace.html#Laplace.cdf)[¶](#torch.distributions.laplace.Laplace.cdf "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/laplace.html#Laplace.entropy)[¶](#torch.distributions.laplace.Laplace.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/laplace.html#Laplace.expand)[¶](#torch.distributions.laplace.Laplace.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.laplace.Laplace.has_rsample "此定义的永久链接")


 icdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/laplace.html#Laplace.icdf)[¶](#torch.distributions.laplace.Laplace.icdf "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/laplace.html#Laplace.log_prob)[¶](#torch.distributions.laplace.Laplace.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.laplace.Laplace.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.laplace.Laplace.mode "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/laplace.html#Laplace.rsample)[¶](#torch.distributions.laplace.Laplace.rsample "此定义的永久链接")


*财产*


 stddev [¶](#torch.distributions.laplace.Laplace.stddev"此定义的永久链接")


 支持*=


 Real()*[¶](#torch.distributions.laplace.Laplace.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.laplace.Laplace.variance "此定义的永久链接")


## LogNormal [¶](#lognormal "此标题的永久链接")


*班级*


 torch.distributions.log_normal。


 对数正态分布


 ( *loc
* 、 *scale
* 、 *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/log_normal.html#LogNormal)[¶](#torch.distributions.log_normal.LogNormal "此定义的永久链接")


 基础： [`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution")


 创建由 [`loc`](#torch.distributions.log_normal.LogNormal.loc "torch.distributions.log_normal.LogNormal.loc") 和 [`scale`](#torch.distributions.log_normal) 参数化的对数正态分布。 LogNormal.scale "torch.distributions.log_normal.LogNormal.scale") 其中：


```
X ~ Normal(loc, scale)
Y = exp(X) ~ LogNormal(loc, scale)

```


 例子：


```
>>> m = LogNormal(torch.tensor([0.0]), torch.tensor([1.0]))
>>> m.sample()  # log-normal distributed with mean=0 and stddev=1
tensor([ 0.1046])

```


 参数 
* **loc** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 分布对数平均值
* **scale** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 分布对数的标准差


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {'loc': Real(), 'scale': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.log_normal.LogNormal.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/log_normal.html#LogNormal.entropy)[¶](#torch.distributions.log_normal.LogNormal.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/log_normal.html#LogNormal.expand)[¶](#torch.distributions.log_normal.LogNormal.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.log_normal.LogNormal.has_rsample "此定义的永久链接")


*财产*


 loc [¶](#torch.distributions.log_normal.LogNormal.loc"此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.log_normal.LogNormal.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.log_normal.LogNormal.mode"此定义的永久链接")


*财产*


 scale [¶](#torch.distributions.log_normal.LogNormal.scale"此定义的永久链接")


 支持*=


 GreaterThan(lower_bound=0.0)*[¶](#torch.distributions.log_normal.LogNormal.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.log_normal.LogNormal.variance "此定义的永久链接")


## LowRankMultivariateNormal [¶](#lowrankmultivariatenormal "此标题的永久链接")


*班级*


 torch.distributions.lowrank_multivariate_normal。


 低秩多元正态


 ( *loc
* 、 *cov_factor
* 、 *cov_diag
* 、 *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/lowrank_multivariate_normal.html#LowRankMultivariateNormal)[¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建一个多元正态分布，其协方差矩阵具有由 `cov_factor` 和 `cov_diag` 参数化的低秩形式：


```
covariance_matrix = cov_factor @ cov_factor.T + cov_diag

```


 例子


```
>>> m = LowRankMultivariateNormal(torch.zeros(2), torch.tensor([[1.], [0.]]), torch.ones(2))
>>> m.sample()  # normally distributed with mean=`[0,0]`, cov_factor=`[[1],[0]]`, cov_diag=`[1,1]`
tensor([-0.2102, -0.5429])

```


 参数 
* **loc** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 形状为 batch_shape 
+ event_shape
* **cov_factor** 的分布平均值( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 形状为 batch_shape 
+ event_shape 
+ (rank,)
* **cov 的协方差矩阵低阶形式的因子部分_diag** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 形状为 batch_shape 
+ event_shape 的协方差矩阵低阶形式的对角部分



!!! note "笔记"

    当 cov_factor.shape[1] << cov_factor.shape[0] 时，由于 [Woodbury 矩阵恒等式](https://en.wikipedia.org/wiki/Woodbury_matrix_identity)和[矩阵行列式引理](https://en.wikipedia.org/wiki/Matrix_determinant_lemma)。借助这些公式，我们只需要计算小尺寸“电容”矩阵的行列式和逆矩阵：


```
capacitance = I + cov_factor.T @ inv(cov_diag) @ cov_factor

```


 arg_constraints *=


 {'cov_diag': IndependentConstraint(GreaterThan(lower_bound=0.0), 1), 'cov_factor': IndependentConstraint(Real(), 2), 'loc': IndependentConstraint(Real(), 1)}
* [¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.arg_constraints"此定义的永久链接")


*财产*


 covariance_matrix [¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.covariance_matrix "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/lowrank_multivariate_normal.html#LowRankMultivariateNormal.entropy)[¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/lowrank_multivariate_normal.html#LowRankMultivariateNormal.expand)[¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.has_rsample "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/lowrank_multivariate_normal.html#LowRankMultivariateNormal.log_prob)[¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.mean"此定义的永久链接")


*财产*


 模式 [¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.mode "此定义的永久链接")


*财产*


 precision_matrix [¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal. precision_matrix "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/lowrank_multivariate_normal.html#LowRankMultivariateNormal.rsample)[¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.rsample "此定义的永久链接")


*财产*


 scale_tril [¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.scale_tril "此定义的永久链接")


 支持*=


 IndependentConstraint(Real(), 1)*[¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.lowrank_multivariate_normal.LowRankMultivariateNormal.variance "此定义的永久链接")


## MixtureSameFamily [¶](#mixturesamefamily "此标题的永久链接")


*班级*


 torch.distributions.mixture_same_family。


 混合物同一个家庭


 ( *mixture_distribution
* , *component_distribution
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/mixture_same_family.html#MixtureSameFamily)[¶](#torch.distributions.mixture_same_family.MixtureSameFamily "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 MixtureSameFamily 分布实现(批次)混合分布，其中所有组件都来自同一分布类型的不同参数化。它由分类“选择分布”(针对 k 分量)和分量分布进行参数化，即具有最右边批次形状(等于 [k] )的分布，该分布对每个(批次)分量进行索引。


 例子：


```
>>> # Construct Gaussian Mixture Model in 1D consisting of 5 equally
>>> # weighted normal distributions
>>> mix = D.Categorical(torch.ones(5,))
>>> comp = D.Normal(torch.randn(5,), torch.rand(5,))
>>> gmm = MixtureSameFamily(mix, comp)

>>> # Construct Gaussian Mixture Model in 2D consisting of 5 equally
>>> # weighted bivariate normal distributions
>>> mix = D.Categorical(torch.ones(5,))
>>> comp = D.Independent(D.Normal(
...          torch.randn(5,2), torch.rand(5,2)), 1)
>>> gmm = MixtureSameFamily(mix, comp)

>>> # Construct a batch of 3 Gaussian Mixture Models in 2D each
>>> # consisting of 5 random weighted bivariate normal distributions
>>> mix = D.Categorical(torch.rand(3,5))
>>> comp = D.Independent(D.Normal(
...         torch.randn(3,5,2), torch.rand(3,5,2)), 1)
>>> gmm = MixtureSameFamily(mix, comp)

```


 参数 
* **mixture_distribution** – torch.distributions.Categorical -likeinstance。管理选择组件的概率。类别数必须与组件_distribution最右边的批次维度匹配。必须具有标量batch_shape 或batch_shape 匹配组件_distribution.batch_shape[:-1]
* **组件_distribution** – torch.distributions.Distribution 类似实例。最右边的批量维度索引组件。


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {}*[¶](#torch.distributions.mixture_same_family.MixtureSameFamily.arg_constraints"此定义的永久链接")


 cdf
 


 ( *x
* ) [[source]](_modules/torch/distributions/mixture_same_family.html#MixtureSameFamily.cdf)[¶](#torch.distributions.mixture_same_family.MixtureSameFamily.cdf "此定义的永久链接")


*财产*


 component_distribution [¶](#torch.distributions.mixture_same_family.MixtureSameFamily.component_distribution"此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/mixture_same_family.html#MixtureSameFamily.expand)[¶](#torch.distributions.mixture_same_family.MixtureSameFamily.expand "此定义的永久链接")


 有_r样本 *=


 False*[¶](#torch.distributions.mixture_same_family.MixtureSameFamily.has_rsample"此定义的永久链接")


 日志_prob


 ( *x
* ) [[source]](_modules/torch/distributions/mixture_same_family.html#MixtureSameFamily.log_prob)[¶](#torch.distributions.mixture_same_family.MixtureSameFamily.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.mixture_same_family.MixtureSameFamily.mean"此定义的永久链接")


*财产*


 mix_distribution [¶](#torch.distributions.mixture_same_family.MixtureSameFamily.mixture_distribution "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/mixture_same_family.html#MixtureSameFamily.sample)[¶](#torch.distributions.mixture_same_family.MixtureSameFamily.sample "此定义的永久链接")


*财产*


 支持[¶](#torch.distributions.mixture_same_family.MixtureSameFamily.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.mixture_same_family.MixtureSameFamily.variance "此定义的永久链接")


## 多项式 [¶](#multinomial "此标题的永久链接")


*班级*


 torch.distributions.multinomial。


 多项式


 ( *总数



 =
 


 1
* , 
* 问题



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/multinomial.html#Multinomial)[¶](#torch.distributions.multinomial.Multinomial "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建由 [`total_count`](#torch.distributions.multinomial.Multinomial.total_count "torch.distributions.multinomial.Multinomial.total_count") 和 [`probs`](#torch.distributions.multinomial. Multinomial.probs "torch.distributions.multinomial.Multinomial.probs") 或 [`logits`](#torch.distributions.multinomial.Multinomial.logits "torch.distributions.multinomial.Multinomial.logits") (但不是两者)。 [`probs`](#torch.distributions.multinomial.Multinomial.probs "torch.distributions.multinomial.Multinomial.probs") 的最内部维度对类别进行索引。所有其他维度的批次索引。


 请注意，如果仅 [`log_prob()`](#torch.distributions.multinomial.Multinomial.log_prob "torch.distributions.multinomial.Multinomial.log_prob") 被调用(参见下面的示例)




!!! note "笔记"

    probs 参数必须是非负的、有限的并且具有非零和，并且它将沿着最后一个维度标准化为和为 1。 [`probs`](#torch.distributions.multinomial.Multinomial.probs "torch.distributions.multinomial.Multinomial.probs") 将返回此归一化值。logits 参数将被解释为非归一化对数概率，因此可以是任何实数。它同样会被归一化，以便沿最后一个维度得到的概率总和为 1。 [`logits`](#torch.distributions.multinomial.Multinomial.logits "torch.distributions.multinomial.Multinomial.logits") 将返回此标准化值。



* [`sample()`](#torch.distributions.multinomial.Multinomial.sample "torch.distributions.multinomial.Multinomial.sample") 需要所有参数和样本的单个共享总数_count。
* [`log_prob( )`](#torch.distributions.multinomial.Multinomial.log_prob "torch.distributions.multinomial.Multinomial.log_prob") 允许每个参数和样本有不同的总计数。


 例子：


```
>>> m = Multinomial(100, torch.tensor([ 1., 1., 1., 1.]))
>>> x = m.sample()  # equal probability of 0, 1, 2, 3
tensor([ 21., 24., 30., 25.])

>>> Multinomial(probs=torch.tensor([1., 1., 1., 1.])).log_prob(x)
tensor([-4.1338])

```


 参数 
* **total_count** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 试验次数
* **probs** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 事件概率
* **logits** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 事件日志概率(未归一化)


 arg_constraints *=


 {'logits': IndependentConstraint(Real(), 1), 'probs': Simplex()}*[¶](#torch.distributions.multinomial.Multinomial.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/multinomial.html#Multinomial.entropy)[¶](#torch.distributions.multinomial.Multinomial.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/multinomial.html#Multinomial.expand)[¶](#torch.distributions.multinomial.Multinomial.expand "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/multinomial.html#Multinomial.log_prob)[¶](#torch.distributions.multinomial.Multinomial.log_prob "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.multinomial.Multinomial.logits "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.multinomial.Multinomial.mean"此定义的永久链接")


*财产*


 param_shape [¶](#torch.distributions.multinomial.Multinomial.param_shape "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.multinomial.Multinomial.probs"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/multinomial.html#Multinomial.sample)[¶](#torch.distributions.multinomial.Multinomial.sample "此定义的永久链接")


*财产*


 支持[¶](#torch.distributions.multinomial.Multinomial.support"此定义的永久链接")


 总数 *：


[int](https://docs.python.org/3/library/functions.html#int"(Python v3.12)")*[¶](#torch.distributions.multinomial.Multinomial.total_count"永久链接到这个定义")


*财产*


 方差 [¶](#torch.distributions.multinomial.Multinomial.variance "此定义的永久链接")


## MultivariateNormal [¶](#multivariatenormal "此标题的永久链接")


*班级*


 torch.distributions.multivariate_normal。


 多元正态


 ( *loc
* , *协方差_matrix



 =
 


 无
* , *精度_矩阵



 =
 


 无
* , *scale_tril



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/multivariate_normal.html#MultivariateNormal)[¶](#torch.distributions.multivariate_normal.MultivariateNormal "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建由均值向量和协方差矩阵参数化的多元正态(也称为高斯)分布。


 多元正态分布可以用正定协方差矩阵来参数化



 Σ
 


 \mathbf{\西格玛}


 Σ
 


 或正定精度矩阵




 Σ
 


 −
 

 1
 


 \mathbf{\Sigma}^{-1}



 Σ
 




 −
 

 1
 


 或下三角矩阵



 L
 


 \mathbf{L}


 L
 


 具有正值对角线条目，这样


 Σ = L


 L
 

 ⊤
 


 \mathbf{\Sigma} = \mathbf{L}\mathbf{L}^	op


 Σ
 



 =
 




 L
 


 L
 



 ⊤
 


 。该三角矩阵可以通过例如获得协方差的 Cholesky 分解。


 例子


```
>>> m = MultivariateNormal(torch.zeros(2), torch.eye(2))
>>> m.sample()  # normally distributed with mean=`[0,0]` and covariance_matrix=`I`
tensor([-0.2102, -0.5429])

```


 参数 
* **loc** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的平均值
* **协方差_matrix** ( [*Tensor*](tensors. html#torch.Tensor "torch.Tensor") ) – 正定协方差矩阵
* **精度_matrix** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 正-定精度矩阵
* **scale_tril** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 协方差的下三角因子，对角线为正值



!!! note "笔记"

    仅 [`covariance_matrix`](#torch.distributions.multivariate_normal.MultivariateNormal.covariance_matrix "torch.distributions.multivariate_normal.MultivariateNormal.covariance_matrix") 或 [` precision_matrix`](#torch.distributions.multivariate_normal.MultivariateNormal) 之一. precision_matrix "torch.distributions.multivariate_normal.MultivariateNormal. precision_matrix") 或 [`scale_tril`](#torch.distributions.multivariate_normal.MultivariateNormal.scale_tril "torch.distributions.multivariate_normal.MultivariateNormal.scale_tril") 可以指定。


 使用 [`scale_tril`](#torch.distributions.multivariate_normal.MultivariateNormal.scale_tril "torch.distributions.multivariate_normal.MultivariateNormal.scale_tril") 会更高效：内部所有计算都基于 [`scale_tril`](# torch.distributions.multivariate_normal.MultivariateNormal.scale_tril "torch.distributions.multivariate_normal.MultivariateNormal.scale_tril") 。如果 [`covariance_matrix`](#torch.distributions.multivariate_normal.MultivariateNormal.covariance_matrix "torch.distributions.multivariate_normal.MultivariateNormal.covariance_matrix") 或 [` precision_matrix`](#torch.distributions.multivariate_normal.MultivariateNormal. precision_matrix相反，传递“torch.distributions.multivariate_normal.MultivariateNormal. precision_matrix”)，它仅用于使用 Cholesky 分解计算相应的下三角矩阵。


 arg_constraints *=


 {'协方差_matrix': PositiveDefinite(), 'loc': IndependentConstraint(Real(), 1), '精度_matrix': PositiveDefinite(), 'scale_tril': LowerCholesky()}*[¶](# torch.distributions.multivariate_normal.MultivariateNormal.arg_constraints"此定义的永久链接")


*财产*


 covariance_matrix [¶](#torch.distributions.multivariate_normal.MultivariateNormal.covariance_matrix "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/multivariate_normal.html#MultivariateNormal.entropy)[¶](#torch.distributions.multivariate_normal.MultivariateNormal.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/multivariate_normal.html#MultivariateNormal.expand)[¶](#torch.distributions.multivariate_normal.MultivariateNormal.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.multivariate_normal.MultivariateNormal.has_rsample "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/multivariate_normal.html#MultivariateNormal.log_prob)[¶](#torch.distributions.multivariate_normal.MultivariateNormal.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.multivariate_normal.MultivariateNormal.mean"此定义的永久链接")


*财产*


 模式 [¶](#torch.distributions.multivariate_normal.MultivariateNormal.mode "此定义的永久链接")


*财产*


 precision_matrix [¶](#torch.distributions.multivariate_normal.MultivariateNormal. precision_matrix "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/multivariate_normal.html#MultivariateNormal.rsample)[¶](#torch.distributions.multivariate_normal.MultivariateNormal.rsample "此定义的永久链接")


*财产*


 scale_tril [¶](#torch.distributions.multivariate_normal.MultivariateNormal.scale_tril "此定义的永久链接")


 支持*=


 IndependentConstraint(Real(), 1)*[¶](#torch.distributions.multivariate_normal.MultivariateNormal.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.multivariate_normal.MultivariateNormal.variance "此定义的永久链接")


## NegativeBinomial [¶](# Negativebinomial "此标题的固定链接")


*班级*


 torch.distributions.负_二项式。


 负二项式


 (*总数_count*，*概率



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/male_binomial.html#NegativeBinomial)[¶](#torch.distributions.male_binomial.NegativeBinomial "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建负二项分布，即在实现“total_count”失败之前成功的独立且相同的伯努利试验数量的分布。每个伯努利试验的成功概率为 [`probs`](#torch.distributions.male_binomial.NegativeBinomial.probs "torch.distributions.male_binomial.NegativeBinomial.probs") 。


 参数 
* **total_count** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [
* Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 要停止的负伯努利试验的非负数，尽管分布对于实值计数仍然有效
* **probs** ( [*Tensor*] (tensors.html#torch.Tensor "torch.Tensor") ) – 半开区间 [0, 1)
* **logits** ( [*Tensor*](tensors.html#torch.Tensor) 中成功的事件概率"torch.Tensor") ) – 成功概率的事件日志赔率


 arg_constraints *=


 {'logits': Real(), 'probs': HalfOpenInterval(lower_bound=0.0, upper_bound=1.0), 'total_count': GreaterThanEq(lower_bound=0)}*[¶](#torch.distributions.negative_binomial.NegativeBinomial.arg_constraints"此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/male_binomial.html#NegativeBinomial.expand)[¶](#torch.distributions.male_binomial.NegativeBinomial.expand "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/negative_binomial.html#NegativeBinomial.log_prob)[¶](#torch.distributions. Negative_binomial.NegativeBinomial.log_prob "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.negative_binomial.NegativeBinomial.logits "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.negative_binomial.NegativeBinomial.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.negative_binomial.NegativeBinomial.mode"此定义的永久链接")


*财产*


 param_shape [¶](#torch.distributions.negative_binomial.NegativeBinomial.param_shape "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.negative_binomial.NegativeBinomial.probs"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/negative_binomial.html#NegativeBinomial.sample)[¶](#torch.distributions.negative_binomial.NegativeBinomial.sample "此定义的永久链接")


 支持*=


 IntegerGreaterThan(lower_bound=0)*[¶](#torch.distributions. Negative_binomial.NegativeBinomial.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.negative_binomial.NegativeBinomial.variance "此定义的永久链接")


## 正常 [¶](#normal "此标题的固定链接")


*班级*


 火炬.分布.正常。


 普通的


 ( *loc
* 、 *scale
* 、 *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/normal.html#Normal)[¶](#torch.distributions.normal.Normal "此定义的永久链接")


 基础： [`ExponentialFamily`](#torch.distributions.exp_family.ExponentialFamily "torch.distributions.exp_family.ExponentialFamily")


 创建由 `loc` 和 `scale` 参数化的正态(也称为高斯)分布。


 例子：


```
>>> m = Normal(torch.tensor([0.0]), torch.tensor([1.0]))
>>> m.sample()  # normally distributed with loc=0 and scale=1
tensor([ 0.1046])

```


 参数 
* **loc** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 分布平均值(通常称为 mu)
* **scale** ( [*float*](https://docs.python.org /3/library/functions.html#float "(in Python v3.12)")*or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的标准差(通常称为西格玛)


 arg_constraints *=


 {'loc': Real(), 'scale': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.normal.Normal.arg_constraints "此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/normal.html#Normal.cdf)[¶](#torch.distributions.normal.Normal.cdf "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/normal.html#Normal.entropy)[¶](#torch.distributions.normal.Normal.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/normal.html#Normal.expand)[¶](#torch.distributions.normal.Normal.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.normal.Normal.has_rsample "此定义的永久链接")


 icdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/normal.html#Normal.icdf)[¶](#torch.distributions.normal.Normal.icdf "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/normal.html#Normal.log_prob)[¶](#torch.distributions.normal.Normal.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.normal.Normal.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.normal.Normal.mode"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/normal.html#Normal.rsample)[¶](#torch.distributions.normal.Normal.rsample "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/normal.html#Normal.sample)[¶](#torch.distributions.normal.Normal.sample "此定义的永久链接")


*财产*


 stddev [¶](#torch.distributions.normal.Normal.stddev"此定义的永久链接")


 支持*=


 Real()*[¶](#torch.distributions.normal.Normal.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.normal.Normal.variance "此定义的永久链接")


## OneHotCategorical [¶](#onehotcategorical"此标题的永久链接")


*班级*


 torch.distributions.one_hot_categorical。


 热门类别


 (*问题



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/one_hot_categorical.html#OneHotCategorical)[¶](#torch.distributions.one_hot_categorical.OneHotCategorical "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建一个由 [`probs`](#torch.distributions.one_hot_categorical.OneHotCategorical.probs "torch.distributions.one_hot_categorical.OneHotCategorical.probs") 或 [`logits`](#torch.distributions.one_hot_categorical) 参数化的单热分类分布.OneHotCategorical.logits "torch.distributions.one_hot_categorical.OneHotCategorical.logits").


 样本是大小为“probs.size(-1)”的一次性编码向量。




!!! note "笔记"

    probs 参数必须是非负的、有限的并且具有非零和，并且它将沿着最后一个维度标准化为和为 1。 [`probs`](#torch.distributions.one_hot_categorical.OneHotCategorical.probs "torch.distributions.one_hot_categorical.OneHotCategorical.probs") 将返回此归一化值。logits 参数将被解释为非归一化对数概率，因此可以是任何实数。它同样会被归一化，以便沿最后一个维度得到的概率总和为 1。 [`logits`](#torch.distributions.one_hot_categorical.OneHotCategorical.logits "torch.distributions.one_hot_categorical.OneHotCategorical.logits") 将返回此标准化值。


 另请参阅：`torch.distributions.Categorical()`，了解 [`probs`](#torch.distributions.one_hot_categorical.OneHotCategorical.probs "torch.distributions.one_hot_categorical.OneHotCategorical.probs") 和 [`logits`]( #torch.distributions.one_hot_categorical.OneHotCategorical.logits "torch.distributions.one_hot_categorical.OneHotCategorical.logits") 。


 例子：


```
>>> m = OneHotCategorical(torch.tensor([ 0.25, 0.25, 0.25, 0.25 ]))
>>> m.sample()  # equal probability of 0, 1, 2, 3
tensor([ 0., 0., 0., 1.])

```


 参数 
* **probs** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 事件概率
* **logits** ( [*Tensor*](tensors.html#torch.tensor "torch.Tensor") ) – 事件日志概率(未归一化)


 arg_constraints *=


 {'logits': IndependentConstraint(Real(), 1), 'probs': Simplex()}*[¶](#torch.distributions.one_hot_categorical.OneHotCategorical.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/one_hot_categorical.html#OneHotCategorical.entropy)[¶](#torch.distributions.one_hot_categorical.OneHotCategorical.entropy "此定义的永久链接")


 枚举_支持


 ( *扩张



 =
 


 True
* ) [[source]](_modules/torch/distributions/one_hot_categorical.html#OneHotCategorical.enumerate_support)[¶](#torch.distributions.one_hot_categorical.OneHotCategorical.enumerate_support "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/one_hot_categorical.html#OneHotCategorical.expand)[¶](#torch.distributions.one_hot_categorical.OneHotCategorical.expand "此定义的永久链接")


 有_枚举_支持 *=


 True*[¶](#torch.distributions.one_hot_categorical.OneHotCategorical.has_enumerate_support "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/one_hot_categorical.html#OneHotCategorical.log_prob)[¶](#torch.distributions.one_hot_categorical.OneHotCategorical.log_prob "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.one_hot_categorical.OneHotCategorical.logits"此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.one_hot_categorical.OneHotCategorical.mean"此定义的永久链接")


*财产*


 模式 [¶](#torch.distributions.one_hot_categorical.OneHotCategorical.mode"此定义的永久链接")


*财产*


 param_shape [¶](#torch.distributions.one_hot_categorical.OneHotCategorical.param_shape "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.one_hot_categorical.OneHotCategorical.probs"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/one_hot_categorical.html#OneHotCategorical.sample)[¶](#torch.distributions.one_hot_categorical.OneHotCategorical.sample "此定义的永久链接")


 支持*=


 OneHot()*[¶](#torch.distributions.one_hot_categorical.OneHotCategorical.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.one_hot_categorical.OneHotCategorical.variance "此定义的永久链接")


## Pareto [¶](#pareto "此标题的永久链接")


*班级*


 torch.distributions.pareto。


 帕累托


 ( *scale
* 、 *alpha
* 、 *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/pareto.html#Pareto)[¶](#torch.distributions.pareto.Pareto "此定义的永久链接")


 基础： [`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution")


 来自帕累托 1 型分布的样本。


 例子：


```
>>> m = Pareto(torch.tensor([1.0]), torch.tensor([1.0]))
>>> m.sample()  # sample from a Pareto distribution with scale=1 and alpha=1
tensor([ 1.5623])

```


 参数 
* **scale** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的尺度参数
* **alpha** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 分布的形状参数


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {'alpha': GreaterThan(lower_bound=0.0), 'scale': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.pareto.Pareto.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/pareto.html#Pareto.entropy)[¶](#torch.distributions.pareto.Pareto.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/pareto.html#Pareto.expand)[¶](#torch.distributions.pareto.Pareto.expand "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.pareto.Pareto.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.pareto.Pareto.mode "此定义的永久链接")


*财产*


 支持[¶](#torch.distributions.pareto.Pareto.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.pareto.Pareto.variance "此定义的永久链接")


## 泊松 [¶](#poisson "此标题的固定链接")


*班级*


 torch.distributions.poisson。


 泊松


 ( *rate
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/poisson.html#Poisson)[¶](#torch.distributions.poisson.Poisson "此定义的永久链接")


 基础： [`ExponentialFamily`](#torch.distributions.exp_family.ExponentialFamily "torch.distributions.exp_family.ExponentialFamily")


 创建由速率参数“rate”参数化的泊松分布。


 样本是非负整数，pmf 为


 速度


 k
 




 e
 


 −
 


 速度


 k
 

 !
 


 \mathrm{速率}^k rac{e^{-\mathrm{速率}}}{k!}




 rate
 




 k
 




 k
 

 !
 


 e
 




 −
 


 rate
 




 ​
 


 例子：


```
>>> m = Poisson(torch.tensor([4]))
>>> m.sample()
tensor([ 3.])

```


 Parameters


**rate** ( *Number
* *,
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 速率参数


 arg_constraints *=


 {'rate': GreaterThanEq(lower_bound=0.0)}*[¶](#torch.distributions.poisson.Poisson.arg_constraints "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/poisson.html#Poisson.expand)[¶](#torch.distributions.poisson.Poisson.expand "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/poisson.html#Poisson.log_prob)[¶](#torch.distributions.poisson.Poisson.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.poisson.Poisson.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.poisson.Poisson.mode "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/poisson.html#Poisson.sample)[¶](#torch.distributions.poisson.Poisson.sample "此定义的永久链接")


 支持*=


 IntegerGreaterThan(lower_bound=0)*[¶](#torch.distributions.poisson.Poisson.support "此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.poisson.Poisson.variance "此定义的永久链接")


## RelaxedBernoulli [¶](#relaxedbernoulli "此标题的永久链接")


*班级*


 torch.distributions.relaxed_bernoulli。


 放松伯努利


 (*温度*，*问题



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/relaxed_bernoulli.html#RelaxedBernoulli)[¶](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli "此定义的永久链接")


 基础： [`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution")


 创建一个 RelaxedBernoulli 分布，参数化为 [`temporch`](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli.Temperature "torch.distributions.relaxed_bernoulli.RelaxedBernoulli.Temperature") 和 [`probs`](#torch.distributions.relaxed_bernoulli).RelaxedBernoulli.probs "torch.distributions.relaxed_bernoulli.RelaxedBernoulli.probs") 或 [`logits`](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli.logits "torch.distributions.relaxed_bernoulli.RelaxedBernoulli.logits") (但不是两者) 。这是伯努利分布的宽松版本，因此值位于 (0, 1) 中，并且具有可重新参数化的样本。


 例子：


```
>>> m = RelaxedBernoulli(torch.tensor([2.2]),
...                      torch.tensor([0.1, 0.2, 0.3, 0.99]))
>>> m.sample()
tensor([ 0.2951, 0.3442, 0.8918, 0.9021])

```


 参数 
* **温度** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 弛豫温度
* **probs** ( *Number
* *,
* [*Tensor*]( tensors.html#torch.Tensor "torch.Tensor") ) – 采样概率 1
* **logits** ( *Number
* *,
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor ") ) – 采样 1 的对数几率


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {'logits': Real(), 'probs': Interval(lower_bound=0.0, upper_bound=1.0)}*[¶](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli.arg_constraints "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/relaxed_bernoulli.html#RelaxedBernoulli.expand)[¶](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli.has_rsample "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli.logits "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli.probs"此定义的永久链接")


 支持*=


 间隔(lower_bound=0.0, upper_bound=1.0)*[¶](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli.support "此定义的永久链接")


*财产*


 温度 [¶](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli.温度"此定义的永久链接")


## LogitRelaxedBernoulli [¶](#logitrelaxedbernoulli "此标题的永久链接")


*班级*


 torch.distributions.relaxed_bernoulli。


 Logit松弛伯努利


 (*温度*，*问题



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/relaxed_bernoulli.html#LogitRelaxedBernoulli)[¶](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建由 [`probs`](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.probs "torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.probs") 或 [`logits`](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli) 参数化的 LogitRelaxedBernoulli 分布。 logits "torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.logits")(但不是两者)，这是 RelaxedBernoullidistribution 的 logit。


 样本是 (0, 1) 中值的对数。有关更多详细信息，请参阅[1]。


 参数 
* **温度** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 弛豫温度
* **probs** ( *Number
* *,
* [*Tensor*]( tensors.html#torch.Tensor "torch.Tensor") ) – 采样概率 1
* **logits** ( *Number
* *,
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor ") ) – 采样 1 的对数几率


 [1] 具体分布：离散随机变量的连续松弛(Maddison 等人，2017)


 [2] 使用 Gumbel-Softmax 进行分类重参数化(Jang 等人，2017)


 arg_constraints *=


 {'logits': Real(), 'probs': Interval(lower_bound=0.0, upper_bound=1.0)}*[¶](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.arg_constraints "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/relaxed_bernoulli.html#LogitRelaxedBernoulli.expand)[¶](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.expand "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/relaxed_bernoulli.html#LogitRelaxedBernoulli.log_prob)[¶](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.log_prob "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.logits "此定义的永久链接")


*财产*


 param_shape [¶](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.param_shape "此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.probs"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/relaxed_bernoulli.html#LogitRelaxedBernoulli.rsample)[¶](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.rsample "此定义的永久链接")


 支持*=


 Real()*[¶](#torch.distributions.relaxed_bernoulli.LogitRelaxedBernoulli.support "此定义的永久链接")


## RelaxedOneHotCategorical [¶](#relaxedonehotcategorical"此标题的永久链接")


*班级*


 torch.distributions.relaxed_categorical。


 RelaxedOneHot类别


 (*温度*，*问题



 =
 


 无
* , *logits



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/relaxed_categorical.html#RelaxedOneHotCategorical)[¶](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical "此定义的永久链接")


 基础： [`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution")


 创建一个由 [`temporary`](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.Temperature "torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.temporary") 和 [`probs`](#torch.distributions.relaxed_categorical) 参数化的 RelaxedOneHotCategorical 分布。 RelaxedOneHotCategorical.probs "torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.probs") 或 [`logits`](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.logits "torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.logits") 。这是一个宽松版本属于“OneHotCategorical”分布，因此样本是单纯形的，并且是可重新参数化的。


 例子：


```
>>> m = RelaxedOneHotCategorical(torch.tensor([2.2]),
...                              torch.tensor([0.1, 0.2, 0.3, 0.4]))
>>> m.sample()
tensor([ 0.1294, 0.2324, 0.3859, 0.2523])

```


 参数 
* **温度** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 弛豫温度
* **probs** ( [*Tensor*](tensors.html#torch.tensor "torch.Tensor") ) – 事件概率
* **logits** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 每个事件的非标准化对数概率


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {'logits': IndependentConstraint(Real(), 1), 'probs': Simplex()}*[¶](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.arg_constraints "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/relaxed_categorical.html#RelaxedOneHotCategorical.expand)[¶](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.has_rsample "此定义的永久链接")


*财产*


 logits [¶](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.logits"此定义的永久链接")


*财产*


 probs [¶](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.probs"此定义的永久链接")


 支持*=


 Simplex()*[¶](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.support"此定义的永久链接")


*财产*


 温度 [¶](#torch.distributions.relaxed_categorical.RelaxedOneHotCategorical.Temperature"此定义的永久链接")


## StudentT [¶](#studentt "此标题的固定链接")


*班级*


 torch.distributions.studentT.


 学生


 ( *df
* , *loc



 =
 


 0.0
* , *比例



 =
 


 1.0
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/studentT.html#StudentT)[¶](#torch.distributions.studentT.StudentT "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 创建由自由度 `df` 、均值 `loc` 和尺度 `scale` 参数化的学生 t 分布。


 例子：


```
>>> m = StudentT(torch.tensor([2.0]))
>>> m.sample()  # Student's t-distributed with degrees of freedom=2
tensor([ 0.1046])

```


 参数 
* **df** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 自由度
* **loc** ( [*float*](https://docs.python.org/3/library/functions.html #float "(Python v3.12)")*或
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 分布平均值
* **scale** ( [*float *](https://docs.python.org/3/library/functions.html#float "(Python v3.12)")*或
* [*Tensor*](tensors.html#torch.Tensor "火炬.Tensor") ) – 分布范围


 arg_constraints *=


 {'df': GreaterThan(lower_bound=0.0), 'loc': Real(), 'scale': GreaterThan(lower_bound=0.0)}*[¶](#torch.distributions.studentT.StudentT.arg_constraints "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/studentT.html#StudentT.entropy)[¶](#torch.distributions.studentT.StudentT.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/studentT.html#StudentT.expand)[¶](#torch.distributions.studentT.StudentT.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.studentT.StudentT.has_rsample "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/studentT.html#StudentT.log_prob)[¶](#torch.distributions.studentT.StudentT.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.studentT.StudentT.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.studentT.StudentT.mode"此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/studentT.html#StudentT.rsample)[¶](#torch.distributions.studentT.StudentT.rsample "此定义的永久链接")


 支持*=


 Real()*[¶](#torch.distributions.studentT.StudentT.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.studentT.StudentT.variance "此定义的永久链接")


## TransformedDistribution [¶](#transformeddistribution "此标题的永久链接")


*班级*


 torch.distributions.transformed_distribution。


 转换分布


 ( *base_distribution
* , *transforms
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/transformed_distribution.html#TransformedDistribution)[¶](#torch.distributions.transformed_distribution.TransformedDistribution "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 Distribution 类的扩展，它将一系列 Transforms 应用于基本分布。令 f 为所应用变换的组合：


```
X ~ BaseDistribution
Y = f(X) ~ TransformedDistribution(BaseDistribution, f)
log p(Y) = log p(X) + log |det (dX/dY)|

```


 请注意，[`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution") 的`.event_shape`是其基本分布及其变换的最大形状，因为transformedscan引入了相关性事件之间。


 使用 TransformedDistribution 的示例如下：


```
# Building a Logistic Distribution
# X ~ Uniform(0, 1)
# f = a + b * logit(X)
# Y ~ f(X) ~ Logistic(a, b)
base_distribution = Uniform(0, 1)
transforms = [SigmoidTransform().inv, AffineTransform(loc=a, scale=b)]
logistic = TransformedDistribution(base_distribution, transforms)

```


 有关更多示例，请查看 [`Gumbel`](#torch.distributions.gumbel.Gumbel "torch.distributions.gumbel.Gumbel") 、 [`HalfCauchy`](#torch.distributions.half_cauchy.HalfCauchy " 的实现torch.distributions.half_cauchy.HalfCauchy") , [`HalfNormal`](#torch.distributions.half_normal.HalfNormal "torch.distributions.half_normal.HalfNormal") , [`LogNormal`](#torch.distributions.log_normal.LogNormal " torch.distributions.log_normal.LogNormal") , [`Pareto`](#torch.distributions.pareto.Pareto "torch.distributions.pareto.Pareto") , [`Weibull`](#torch.distributions.weibull.Weibull " torch.distributions.weibull.Weibull") , [`RelaxedBernoulli`](#torch.distributions.relaxed_bernoulli.RelaxedBernoulli "torch.distributions.relaxed_bernoulli.RelaxedBernoulli") 和 [`RelaxedOneHotCategorical`](#torch.distributions.relaxed_categorical.RelaxedOne热门分类“ torch.distributions.relaxed_categorical.RelaxedOneHotCategorical")


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {}*[¶](#torch.distributions.transformed_distribution.TransformedDistribution.arg_constraints"此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/transformed_distribution.html#TransformedDistribution.cdf)[¶](#torch.distributions.transformed_distribution.TransformedDistribution.cdf "此定义的永久链接")


 通过反转变换并计算基本分布的分数来计算累积分布函数。


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/transformed_distribution.html#TransformedDistribution.expand)[¶](#torch.distributions.transformed_distribution.TransformedDistribution.expand "此定义的永久链接")


*财产*


 has_rsample [¶](#torch.distributions.transformed_distribution.TransformedDistribution.has_rsample "此定义的永久链接")


 icdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/transformed_distribution.html#TransformedDistribution.icdf)[¶](#torch.distributions.transformed_distribution.TransformedDistribution.icdf "此定义的永久链接")


 使用变换计算逆累积分布函数并计算基本分布的分数。


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/transformed_distribution.html#TransformedDistribution.log_prob)[¶](#torch.distributions.transformed_distribution.TransformedDistribution.log_prob "此定义的永久链接")


 通过反转变换并使用基本分布的分数和雅可比对数来计算分数，对样本进行评分。


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/transformed_distribution.html#TransformedDistribution.rsample)[¶](#torch.distributions.transformed_distribution.TransformedDistribution.rsample "此定义的永久链接")


 如果分布参数是批处理的，则生成样本形状重新参数化样本或样本样本形状的重新参数化样本批次。首先从基本分布中采样，并对列表中的每个变换应用 Transform() 。


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/transformed_distribution.html#TransformedDistribution.sample)[¶](#torch.distributions.transformed_distribution.TransformedDistribution.sample "此定义的永久链接")


 如果分布参数是批处理的，则生成样本形状的样本或样本形状的批次样本。首先从基分布中采样，并对列表中的每个变换应用 Transform() 。


*财产*


 支持[¶](#torch.distributions.transformed_distribution.TransformedDistribution.support"此定义的永久链接")


## Uniform [¶](#uniform "此标题的永久链接")


*班级*


 火炬.distributions.uniform。


 制服


 ( *低
* , *高
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/uniform.html#Uniform)[¶](#torch.distributions.uniform.Uniform "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 从半开区间“[low, high)”生成均匀分布的随机样本。


 例子：


```
>>> m = Uniform(torch.tensor([0.0]), torch.tensor([5.0]))
>>> m.sample()  # uniformly distributed in the range [0.0, 5.0)
tensor([ 2.3418])

```


 参数 
* **low** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 较低范围(含)。
* **高** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 上限(不包括)。


 arg_constraints *=


 {'high': Dependent(), 'low': Dependent()}*[¶](#torch.distributions.uniform.Uniform.arg_constraints "此定义的永久链接")


 cdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/uniform.html#Uniform.cdf)[¶](#torch.distributions.uniform.Uniform.cdf "此定义的永久链接")


 熵


 ( ) [[source]](_modules/torch/distributions/uniform.html#Uniform.entropy)[¶](#torch.distributions.uniform.Uniform.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/uniform.html#Uniform.expand)[¶](#torch.distributions.uniform.Uniform.expand "此定义的永久链接")


 有_r样本 *=


 True*[¶](#torch.distributions.uniform.Uniform.has_rsample "此定义的永久链接")


 icdf
 


 ( *value
* ) [[source]](_modules/torch/distributions/uniform.html#Uniform.icdf)[¶](#torch.distributions.uniform.Uniform.icdf "此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/uniform.html#Uniform.log_prob)[¶](#torch.distributions.uniform.Uniform.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.uniform.Uniform.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.uniform.Uniform.mode "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/uniform.html#Uniform.rsample)[¶](#torch.distributions.uniform.Uniform.rsample "此定义的永久链接")


*财产*


 stddev [¶](#torch.distributions.uniform.Uniform.stddev"此定义的永久链接")


*财产*


 支持[¶](#torch.distributions.uniform.Uniform.support"此定义的永久链接")


*财产*


 方差 [¶](#torch.distributions.uniform.Uniform.variance "此定义的永久链接")


## VonMises [¶](#vonmises "此标题的永久链接")


*班级*


 torch.distributions.von_mises。


 冯·米塞斯


 ( *loc
* , *浓度
* , *validate_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/von_mises.html#VonMises)[¶](#torch.distributions.von_mises.VonMises "此定义的永久链接")


 基础：[`Distribution`](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution")


 圆形冯米塞斯分布。


 该实现使用极坐标。 `loc` 和 `value` arg 可以是任何实数(以促进无约束优化)，但被解释为以 2 pi 为模的角度。


 例子：：




```
>>> m = VonMises(torch.tensor([1.0]), torch.tensor([1.0]))
>>> m.sample()  # von Mises distributed with loc=1 and concentration=1
tensor([1.9777])

```


 Parameters 
* **loc**  ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – an angle in radians.
* **concentration**  ( [*torch.Tensor*] (TENSOR.HTML＃TORCH.TENSOR“ TORCH.TESOR”) 
- 集中参数


 arg_constraints *=


 {'concentration': GreaterThan(lower_bound=0.0), 'loc': Real()}*[¶](#torch.distributions.von_mises.VonMises.arg_constraints "此定义的永久链接")


 扩张


 ( *batch_shape
* ) [[source]](_modules/torch/distributions/von_mises.html#VonMises.expand)[¶](#torch.distributions.von_mises.VonMises.expand "此定义的永久链接")


 有_r样本 *=


 False*[¶](#torch.distributions.von_mises.VonMises.has_rsample"此定义的永久链接")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/von_mises.html#VonMises.log_prob)[¶](#torch.distributions.von_mises.VonMises.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.von_mises.VonMises.mean"此定义的永久链接")


 提供的平均值是圆形。


*财产*


 模式[¶](＃torch.distributions.von_mises.vonmises.mode" permalink到此定义")


 样本


 ( *样本_形状



 =
 


 torch.Size([])
* ) [[source]](_modules/torch/distributions/von_mises.html#VonMises.sample)[¶](#torch.distributions.von_mises.VonMises.sample "此定义的永久链接")


 von Mises 分布的采样算法基于以下论文：Best, D. J., and Nicholas I. Fisher.“Efficientsimulation of the von Mises distribution”。应用统计学(1979)：152-157。


 支持*=


 Real()*[¶](#torch.distributions.von_mises.VonMises.support"此定义的永久链接")


*财产*


 方差[¶​​](＃torch.distributions.von_mises.vonmises.voniancer.variance“ permalink to to此定义”)


 提供的方差是圆形方差。


## weibull [¶](＃weibull"固定链接到此标题")


*班级*


 TORCH.Distributions.Weibull。


 威布尔


 ( *规模
* , *浓度
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/weibull.html#Weibull)[¶](#torch.distributions.weibull.Weibull "此定义的永久链接")


 基础： [`TransformedDistribution`](#torch.distributions.transformed_distribution.TransformedDistribution "torch.distributions.transformed_distribution.TransformedDistribution")


 来自两参数威布尔分布的样品。


 例子


```
>>> m = Weibull(torch.tensor([1.0]), torch.tensor([1.0]))
>>> m.sample()  # sample from a Weibull distribution with scale=1, concentration=1
tensor([ 0.4784])

```


 参数 
* **scale** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](TENSORS.HTML＃TORCH.TENSOR“ TORCH.TENSOR”)) 
- 分布的比例参数(lambda)。**浓度**([*float*] library/functions.html＃float“(在python v3.12)”)*或*[*tensor*](tensors.html＃torch.tensor“ torch.tensor”)) 
- 分布的浓度参数(k/shape) 。


 arg_constraints *:


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[约束](#torch.distributions.constraints.Constraint“torch.distributions.constraints.Constraint”)


 ]*
*=
 


 {'浓度'：esleterthan(soulth \ _bound = 0.0)，'scale'：orearthan(soulter \ _bound = 0.0)}*[¶](＃torch.distributions.weibull.weibull.weibull.Arg_contraints" permalints" permalink to th这个定义")


 熵


 ( ) [[source]](_modules/torch/distributions/weibull.html#Weibull.entropy)[¶](#torch.distributions.weibull.Weibull.entropy "此定义的永久链接")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/weibull.html#Weibull.expand)[¶](#torch.distributions.weibull.Weibull.expand "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.weibull.Weibull.mean"此定义的永久链接")


*财产*


 mode [¶](#torch.distributions.weibull.Weibull.mode "此定义的永久链接")


 支持*=


 esseythan(下\ _bound = 0.0)*[¶](＃torch.distributions.weibull.weibull.support" permalink to to此定义")


*财产*


 方差[¶​​](＃torch.distributions.weibull.weibull.wariance“ permalink to th这个定义”)


## wishart [¶](＃wishart"固定到此标题")


*班级*


 torch.distributions.wishart。


 威沙特


 ( *df
* , *协方差_matrix



 =
 


 无
* , *精度_矩阵



 =
 


 无
* , *scale_tril



 =
 


 无
* , *验证_args



 =
 


 无
* ) [[source]](_modules/torch/distributions/wishart.html#Wishart)[¶](#torch.distributions.wishart.Wishart"此定义的永久链接")


 基础： [`ExponentialFamily`](#torch.distributions.exp_family.ExponentialFamily "torch.distributions.exp_family.ExponentialFamily")


 创建通过对称正定矩阵参数化的WishArt分布



 Σ
 


 \西格玛


 Σ
 


 ，或其 Cholesky 分解


 Σ = L


 L
 

 ⊤
 


 \mathbf{\Sigma} = \mathbf{L}\mathbf{L}^	op


 Σ
 



 =
 




 L
 


 L
 



 ⊤
 


 例子


```
>>> m = Wishart(torch.Tensor([2]), covariance_matrix=torch.eye(2))
>>> m.sample()  # Wishart distributed with mean=`df * I` and
>>>             # variance(x_ij)=`df` for i != j and variance(x_ij)=`2 * df` for i == j

```


 参数 
* **df** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* [*Tensor
* ](tensors.html#torch.Tensor "torch.Tensor") ) – 大于(方阵维数)的实值参数 
- 1
* **covariance_matrix** ( [*Tensor*](tensors.html #torch.Tensor "torch.Tensor") ) – 正定协方差矩阵
* **精度_matrix** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 正定精度矩阵
* **scale_tril** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 协方差的下三角因子，对角线为正值



!!! note "笔记"

    仅 [`covariance_matrix`](#torch.distributions.wishart.Wishart.covariance_matrix "torch.distributions.wishart.Wishart.covariance_matrix") 或 [` precision_matrix`](#torch.distributions.wishart.Wishart) 之一. precision_matrix "torch.distributions.wishart.Wishart. precision_matrix") 或 [`scale_tril`](#torch.distributions.wishart.Wishart.scale_tril "torch.distributions.wishart.Wishart.scale_tril") 可以指定。使用[`scale_tril`](#torch.distributions.wishart.Wishart.scale_tril "torch.distributions.wishart.Wishart.scale_tril") 将更加高效：内部所有计算均基于 [`scale_tril`](#torch.distributions.wishart.Wishart.scale_tril "火炬.distributions.wishart.Wishart.scale_tril").如果 [`covariance_matrix`](#torch.distributions.wishart.Wishart.covariance_matrix "torch.distributions.wishart.Wishart.covariance_matrix") 或 [` precision_matrix`](#torch.distributions.wishart.Wishart. precision_matrix相反，传递“torch.distributions.wishart.Wishart. precision_matrix”)，它仅用于使用 Cholesky 分解计算相应的下三角矩阵。“torch.distributions.LKJCholesky”是受限的 Wishart 分布。[1]


**参考**


 [1] Wang, Z.、Wu, Y. 和 Chu, H.，2018。关于 LKJ 分布和受限 Wishart 分布的等价性。[2] Sawyer, S.，2007。Wishart 分布和逆 Wishart 采样。 [3] Anderson, T. W., 2003。多元统计分析简介(第 3 版)。[4] Odell, P. L. 和 Feiveson, A. H., 1966。生成样本协方差矩阵的数值过程。 JASA，61(313)：199-203。[5] Ku，Y.-C。 & Bloomfield, P., 2010。生成具有 OX 分数自由度的随机 Wishart 矩阵。


 arg_constraints *=


 {'协方差_matrix'：PositiveDefinite()，'df'：GreaterThan(lower_bound=0)，'精度_matrix'：PositiveDefinite()，'scale_tril'：LowerCholesky()}*[¶](# torch.distributions.wishart.Wishart.arg_constraints"此定义的永久链接")


*财产*


 covariance_matrix [¶](#torch.distributions.wishart.Wishart.covariance_matrix "此定义的永久链接")


 熵


 ()[[source]](_模块/火炬/distributions/wishart.html＃wishart.entropy)[¶](＃torch.distributions.wishart.wishart.wishart.wishart.entropy" permalink to this Windact")


 扩张


 ( *batch_shape
* , *_instance



 =
 


 无
* ) [[source]](_modules/torch/distributions/wishart.html#Wishart.expand)[¶](#torch.distributions.wishart.Wishart.expand "此定义的永久链接")


 有_r样本 *=


 true*[¶](＃torch.distributions.wishart.wishart.has_rsample" permalink to to此定义")


 日志_prob


 ( *value
* ) [[source]](_modules/torch/distributions/wishart.html#Wishart.log_prob)[¶](#torch.distributions.wishart.Wishart.log_prob "此定义的永久链接")


*财产*


 意思是[¶](#torch.distributions.wishart.Wishart.mean"此定义的永久链接")


*财产*


 模式[¶](＃torch.distributions.wishart.wishart.mode" permalink to此定义")


*财产*


 precision_matrix [¶](#torch.distributions.wishart.Wishart. precision_matrix "此定义的永久链接")


 样本


 ( *样本_形状



 =
 


 torch.size([]) *， 
* max \ _try \ _ correction



 =
 


 无
* ) [[source]](_modules/torch/distributions/wishart.html#Wishart.rsample)[¶](#torch.distributions.wishart.Wishart.rsample "此定义的永久链接")


!!! warning "警告"

     在某些情况下，基于Bartlett分解的采样算法可能会返回奇异矩阵样本。默认情况下会执行多次尝试纠正奇异样本，但最终可能会返回奇异矩阵样本。奇异样本可能会在.log_prob() 中返回 -inf 值。在这些情况下，用户应验证样本并修复 df 的值或相应地调整.rsample 中参数的 max_try_ Correction 值。


*财产*


 比例\ _tril [¶](＃torch.distributions.wishart.wishart.scale_tril" permalink to to此定义")


 支持*=


 pititiveDefinite()*[¶](＃torch.distributions.wishart.wishart.support" permalink to thit this Undication")


*财产*


 方差 [¶](#torch.distributions.wishart.Wishart.variance"此定义的永久链接")


## KL Divergence [¶](＃Module-Torch.distributions.kl"固定链接到此标题")


 torch.distributions.kl。


 kl \ _divergence


 ( *p
* , *q
* ) [[source]](_modules/torch/distributions/kl.html#kl_divergence)[¶](#torch.distributions.kl.kl_divergence "此定义的永久链接")


 计算 Kullback-Leibler 散度


 KL ( p ∥ q )


 KL(p \| q)


 KL ( p ∥ q )


 两个分布之间。


 K L ( p ∥ q ) = ∫ p ( x ) log ⁡


 p(x)


 q(x)


 d
 

 x
 


 KL(p \| q) = \int p(x) \lograc {p(x)} {q(x)} \,dx


 KL ( p ∥ q )



 =
 




 ∫
 


 p(x)



 lo
 
 g
 


 q(x)


 p(x)




 ​
 



 d
 

 x
 


 参数 
* **p** ( [*Distribution*](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution") ) – 一个 `Distribution` 对象。
* **q** ( [*Distribution *](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution") ) – `Distribution` 对象。


 退货


 形状为batch_shape的一批KL散度。


 Return type


[tensor](tensors.html#torch.Tensor "torch.Tensor")


 提高


[**NotImplementedError**](https://docs.python.org/3/library/exceptions.html#NotImplementedError "(in Python v3.12)") – 如果分发类型尚未通过 [`register 注册_kl()`](#torch.distributions.kl.register_kl "torch.distributions.kl.register_kl") 。


 目前，KL 散度针对以下分布对实现： 
* `Bernoulli` 和 `Bernoulli`
* `Bernoulli` 和 `Poisson`
* `Beta` 和 `Beta`
* `Beta` 和 `ContinouslyBernoulli`
* `Beta` 和 `Exponential `
* `Beta` 和 `Gamma`
* `Beta` 和 `Normal`
* `Beta` 和 `Pareto`
* `Beta` 和 `Uniform`
* `二项式` 和 `二项式`
* `分类` 和 `分类`
* “柯西”和“柯西”
* “连续伯努利”和“连续伯努利”
* “连续伯努利”和“指数”
* “连续伯努利”和“正态”
* “连续伯努利”和“帕累托”
* “连续伯努利”和“均匀”
* “狄利克雷” ` 和 `Dirichlet`
* `指数` 和 `Beta`
* `指数` 和 `连续伯努利`
* `指数` 和`指数`
* `指数` 和 `Gamma`
* `指数` 和 `Gumbel`
* `指数` 和“正态”
* “指数”和“帕累托”
* “指数”和“均匀”
* “指数族”和“指数族”
* “伽玛”和“贝塔”
* “伽玛”和“连续伯努利”
* “伽玛”和“指数” `
* `Gamma` 和 `Gamma`
* `Gamma` 和 `Gumbel`
* `Gamma` 和 `Normal`
* `Gamma` 和 `Pareto`
* `Gamma` 和 `Uniform`
* `几何`和`几何`
* “Gumbel”和“Beta”
* “Gumbel”和“连续伯努利”
* “Gumbel”和“指数”
* “Gumbel”和“Gamma”
* “Gumbel”和“Gumbel”
* “Gumbel”和“普通”
* “Gumbel” ` 和 `Pareto`
* `Gumbel` 和 `Uniform`
* `HalfNormal` 和 `HalfNormal`
* `独立`和`独立`
* `拉普拉斯`和`Beta`*`拉普拉斯`和`连续伯努利`*`拉普拉斯`和“指数”
* “拉普拉斯”和“伽玛”
* “拉普拉斯”和“拉普拉斯”
* “拉普拉斯”和“正态”
* “拉普拉斯”和“帕累托”
* “拉普拉斯”和“均匀”
* “LowRankMultivariateNormal”和“LowRankMultivariateNormal” `
* `LowRankMultivariateNormal` 和 `MultivariateNormal`
* `MultivariateNormal` 和 `LowRankMultivariateNormal`
* `MultivariateNormal` 和 `MultivariateNormal`
* `Normal` 和 `Beta`
* `Normal` 和 `ContinouslyBernoulli`
* `Normal` 和 `Exponential`
* `Normal` 和 `Gamma`
* `Normal` 和 `Gumbel`
* `Normal` 和 `Laplace`
* `Normal` 和 `Normal`
* `Normal` 和 `Pareto`
* `Normal` 和 `Uniform`
* `OneHotCategorical ` 和 `OneHotCategorical`
* `Pareto` 和 `Beta`
* `Pareto` 和 `ContinouslyBernoulli`
* `Pareto` 和 `Exponential`
* `Pareto` 和 `Gamma`
* `Pareto` 和 `Normal`
* `Pareto` 和`帕累托`
* `帕累托`和`均匀分布`
* `泊松`和`伯努利`
* `泊松`和`二项式`
* `泊松`和`泊松`
* `TransformedDistribution`和`TransformedDistribution`
* `均匀`和`Beta `
* `均匀`和`连续伯努利`
* `均匀`和`指数`
* `均匀`和`Gamma`*`均匀`和`Gumbel`*`均匀`和`正态`*`均匀`和`帕累托`
* “制服”和“制服”


 torch.distributions.kl。


 注册_kl


 ( *type_p
* , *type_q
* ) [[source]](_modules/torch/distributions/kl.html#register_kl)[¶](#torch.distributions.kl.register_kl "此定义的永久链接")


 装饰器用 [`kl_divergence()`](#torch.distributions.kl.kl_divergence "torch.distributions.kl.kl_divergence") 注册成对函数。用法：


```
@register_kl(Normal, Normal)
def kl_normal_normal(p, q):
    # insert implementation here

```


 查找返回按子类排序的最具体的(类型，类型)匹配。如果匹配不明确，则会引发运行时警告。例如解决模棱两可的情况：


```
@register_kl(BaseP, DerivedQ)
def kl_version1(p, q): ...
@register_kl(DerivedP, BaseQ)
def kl_version2(p, q): ...

```


 您应该注册第三个最具体的实现，例如：


```
register_kl(DerivedP, DerivedQ)(kl_version1)  # Break the tie.

```


 参数 
* **type_p** ( [*type*](https://docs.python.org/3/library/functions.html#type "(in Python v3.12)") ) – 的子类`Distribution`.
* **type_q** ( [*type*](https://docs.python.org/3/library/functions.html#type "(in Python v3.12)") ) – `Distribution` 的子类。


## 转换 [¶](#module-torch.distributions.transforms "此标题的永久链接")


*班级*


 torch.distributions.transforms。


 绝对变换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#AbsTransform)[¶](#torch.distributions.transforms.AbsTransform"此定义的永久链接")


 通过映射进行变换


 y = ∣ x ∣


 y = |x|


 y
 



 =
 


 ∣ x ∣




.
 


*班级*


 torch.distributions.transforms。


 仿射变换


 ( *loc
* 、 *scale
* 、 *event_dim



 =
 


 0
* , *缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#AffineTransform)[¶](#torch.distributions.transforms.AffineTransform"此定义的永久链接")


 通过逐点仿射映射进行变换


 y = loc 
+ 比例 × x


 y = 	ext{loc} 
+ 	ext{scale} 	imes x


 y
 



 =
 


 loc
 




 +
 


 scale
 




 ×
 




 x
 




.
 


 参数 
* **loc** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*或
* [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 位置参数。
* **scale** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*或
* [ *float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 缩放参数。
* **event_dim** ( [
* int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – event_shape 的可选大小。对于单变量随机变量，该值应为零；对于向量的分布，该值应为 1；对于矩阵的分布，该值应为 2，等等。


*班级*


 torch.distributions.transforms。


 猫变换


 (*tseq*，*暗淡



 =
 


 0
* , *长度



 =
 


 无
* , *缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#CatTransform)[¶](#torch.distributions.transforms.CatTransform"此定义的永久链接")


 变换函子，以与 [`torch.cat()`](generated/torch.cat.html#torch.猫“torch.cat”)。


 例子：


```
x0 = torch.cat([torch.range(1, 10), torch.range(1, 10)], dim=0)
x = torch.cat([x0, x0], dim=0)
t0 = CatTransform([ExpTransform(), identity_transform], dim=0, lengths=[10, 10])
t = CatTransform([t0, t0], dim=0, lengths=[20, 20])
y = t(x)

```


*班级*


 torch.distributions.transforms。


 组合转换


 ( *零件
* , *缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#ComposeTransform)[¶](#torch.distributions.transforms.ComposeTransform"此定义的永久链接")


 将多个转换组成一个链。组成的转换负责缓存。


 参数 
* **parts** ([`Transform`](#torch.distributions.transforms.Transform "torch.distributions.transforms.Transform") 的列表) – 要组合的变换列表。
* **cache_size
* 
* ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 缓存大小。如果为零，则不进行缓存。如果有，则缓存最新的单个值。仅支持 0 和 1。


*班级*


 torch.distributions.transforms。


 奇Cholesky变换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#CorrCholeskyTransform)[¶](#torch.distributions.transforms.CorrCholeskyTransform "此定义的永久链接")


 变换无约束实向量



 x
 


 x
 


 x
 


 与长度


 D*(D−1)/2


 D*(D-1)/2


 D
 



 ∗
 




 (
 

 D
 



 −
 


 1)/2


 代入 D 维相关矩阵的 Cholesky 因子。该 Cholesky 因子是一个下三角矩阵，每行具有正对角线和单位欧几里德范数。变换处理如下：



> 
> 
> 1. 首先我们将 x 按行顺序转换为下三角矩阵。
> 2. 对于每一行
> 
> 
> 
> 
> 
> 
> 
> 
> X
> 
> 
> i
> 
> 
> 
> 
> X_i
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> X
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> 
> ​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 的下三角部分，我们应用 
> *signed
* 
> version of
> class
> [`StickBreakingTransform`](#torch.distributions.transforms.StickBreakingTransform "torch.distributions.transforms.StickBreakingTransform")
> 进行转换
> 
> 
> 
> 
> 
> 
> 
> 
> X
> 
> 
> i
> 
> 
> 
> 
> X_i
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> X
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> >​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 使用以下步骤转换为>单位欧氏长度向量：
> 
- 缩放为区间
> 
> 
> 
> 
> 
> 
> 
> (
> 
> 
> −
> 
> 
> 1
> 
> 
> ,
> 
> 
> 1
> 
> 
> )
> 
> 
> 
> (-1, 1)
> 
> 
> 
> 
> 
> 
> 
> 
> 
> (
> 
> 
> −
> 
> 
> 1
> 
> 
> ,
> 
> 
> 
> 
> 1
> 
> 
> )
> 
> 
> 
> 
> 
> 域：
> 
> 
> 
> 
> 
> 
> 
> 
> r
> 
> 
> i
> 
> 
> 
> =
> 
> 
> tanh
> 
> 
> ⁡
> 
> 
> (
> 
> 
> 
> X
> 
> 
> i
> 
> 
> 
> )
> 
> 
> 
> r_i = 	anh(X_i)
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> r
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> 
> ​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> =
> 
> 
> 
> 
> 
> 
> 
> 
> tanh
> 
> 
> (
> 
> 
> 
> X
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> 
> ​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> )
> 
> 
> 
> 
> 
>.
> 
- 转换为无符号域：
> 
> 
> 
> 
> 
> 
> 
> 
> z
> 
> 
> i
> 
> 
> 
> =
> 
> 
> 
> r
> 
> 
> i
> 
> 
> 2
> 
> 
> 
> 
> z_i = r_i^2
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> z
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i 
> 
> 
> 
> 
> >​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> =
> 
> 
> 
> 
> 
> 
> 
> 
> 
> r
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> 
> 
> 
> 
> 2
> 
> 
> 
> 
> 
> ​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
>.
> 
- 适用
> 
> 
> 
> 
> 
> 
> 
> s
> 
> 
> i
> 
> 
> 
> =
> 
> 
> S
> 
> 
> t
> 
> 
> i
> 
> 
> c
> 
> 
> k
> 
> 
> B
> 
> 
> r
> 
> 
> e
> 
> 
> a
> 
> 
> k
> 
> 
> i
> 
> 
> n 
> 
> 
> g
> 
> 
> T
> 
> 
> r
> 
> 
> a
> 
> 
> n
> 
> 
> s
> 
> 
> f
> 
> 
> o
> 
> 
> r
> 
> 
> m
> 
> 
> (
> 
> 
> 
> z
> 
> 
> i
> 
> 
> 
> )
> 
> 
> 
> s_i = StickBreakingTransform(z_i)
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> s
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> 
> ​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> =
> 
> 
> 
> 
> 
> 
> 
> 
> St
> 
> 
> i
> 
> 
> c
> 
> 
> k
> 
> 
> B
> 
> 
> re
> 
> 
> 类似
> 
> 
> g
> 
> 
> T
> 
> 
> r
> 
> 
> an
> 
> 
> s
> 
> 
> f
> 
> 
> 或
> 
> 
> m
> 
> 
> (
> 
> 
> 
> z
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> 
> ​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> )
> 
> 
> 
> 
> 
>.
> 
- 转换回签名域：
> 
> 
> 
> 
> 
> 
> 
> 
> y
> 
> 
> i
> 
> 
> 
> =
> 
> 
> s
> 
> 
> i
> 
> 
> g
> 
> 
> n
> 
> 
> (
> 
> 
> 
> r
> 
> 
> i
> 
> 
> 
> )
> 
> 
> *
> 
> 
> 
> 
> s
> 
> 
> i
> 
> 
> 
> 
> 
> y_i = 符号(r_i) * \sqrt{s_i}
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> y
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> =
> 
> 
> 
> 
> 
> 
> 
> 
> s
> 
> 
> i
> 
> 
> g
> 
> 
> n
> 
> 
> (
> 
> 
> 
> r
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> 
> ​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> )
> 
> 
> 
> 
> ＊
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> s
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> i
> 
> 
> 
> 
> >​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> >​
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
>.
> 
> 
> >


*班级*


 torch.distributions.transforms。


 累积分布变换


 ( *分布
* , *缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#CumulativeDistributionTransform)[¶](#torch.distributions.transforms.CumulativeDistributionTransform"此定义的永久链接")


 通过概率分布的累积分布函数进行变换。


 Parameters


**distribution** ( [*Distribution*](#torch.distributions.distribution.Distribution "torch.distributions.distribution.Distribution") ) – 其累积分布函数用于转换的分布。


 例子：


```
# Construct a Gaussian copula from a multivariate normal.
base_dist = MultivariateNormal(
    loc=torch.zeros(2),
    scale_tril=LKJCholesky(2).sample(),
)
transform = CumulativeDistributionTransform(Normal(0, 1))
copula = TransformedDistribution(base_dist, [transform])

```


*班级*


 torch.distributions.transforms。


 表达式变换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#ExpTransform)[¶](#torch.distributions.transforms.ExpTransform"此定义的永久链接")


 通过映射进行变换


 y = exp ⁡ ( x )


 y = \exp(x)


 y
 



 =
 


 指数 ( x )




.
 


*班级*


 torch.distributions.transforms。


 独立变换


 ( *base_transform
* , *重新解释_batch_ndims
* , *cache_size



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#IndependentTransform)[¶](#torch.distributions.transforms.IndependentTransform"此定义的永久链接")


 包装另一个转换以将“reinterpreted_batch_ndims”视为依赖的许多额外的最右侧维度。这对前向或后向变换没有影响，但会总结出 `reinterpreted_batch_ndims` 
- `log_abs_det_jacobian()` 中最右边的许多维度。


 参数 
* **base_transform** ( [`Transform`](#torch.distributions.transforms.Transform "torch.distributions.transforms.Transform") ) – 基础变换。
* **重新解释_batch_ndims** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 视为依赖的额外最右边维度的数量。


*班级*


 torch.distributions.transforms。


 下乔列斯基变换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#LowerCholeskyTransform)[¶](#torch.distributions.transforms.LowerCholeskyTransform "此定义的永久链接")


 从无约束矩阵转换为具有非负对角线项的下三角矩阵。


 这对于根据 Cholesky 分解来参数化正定矩阵非常有用。


*班级*


 torch.distributions.transforms。


 正定变换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#PositiveDefiniteTransform)[¶](#torch.distributions.transforms.PositiveDefiniteTransform"此定义的永久链接")


 从无约束矩阵变换为正定矩阵。


*班级*


 torch.distributions.transforms。


 电源转换


 ( *指数
* , *缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#PowerTransform)[¶](#torch.distributions.transforms.PowerTransform"此定义的永久链接")


 通过映射进行变换



 y
 

 =
 


 x 指数


 y = x^{	ext{指数}}


 y
 



 =
 


 x
 


 指数




.
 


*班级*


 torch.distributions.transforms。


 重塑变换


 ( *in_shape
* , *out_shape
* , *cache_size



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#ReshapeTransform)[¶](#torch.distributions.transforms.ReshapeTransform"此定义的永久链接")


 单位雅可比变换可重塑tensor的最右边部分。


 请注意，“in_shape”和“out_shape”必须具有相同数量的元素，就像 [`torch.Tensor.reshape()`](generated/torch.Tensor.reshape.html#torch.Tensor.reshape “火炬.tensor.重塑”)。


 参数 
* **in_shape** ( *torch.Size
* ) – 输入事件形状。
* **out_shape** ( *torch.Size
* ) – 输出事件形状。


*班级*


 torch.distributions.transforms。


 Sigmoid变换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#SigmoidTransform)[¶](#torch.distributions.transforms.SigmoidTransform"此定义的永久链接")


 通过映射进行变换



 y
 

 =
 


 1
 


 1 
+ exp ⁡ ( − x )


 y = rac{1}{1 
+ \exp(-x)}


 y
 



 =
 




 1
 

 +
 


 经验值


 (−x)



 1
 


 ​
 




 and
 


 x = 逻辑 ( y )


 x = 	ext{logit}(y)


 x
 



 =
 


 logit
 


 ( )




.
 


*班级*


 torch.distributions.transforms。


 Softplus变换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#SoftplusTransform)[¶](#torch.distributions.transforms.SoftplusTransform"此定义的永久链接")


 通过映射进行变换


 Softplus ( x ) = log ⁡ ( 1 
+ exp ⁡ ( x ) )


 	ext{Softplus}(x) = \log(1 
+ \exp(x))


 软加


 (  X  )



 =
 




 lo
 
 g
 


 (
 

 1
 



 +
 


 指数 ( x ))


.当以下情况时，实现恢复为线性函数：


 x 
> 20


 x 
> 20


 x
 



 >
 




 20
 




.
 


*班级*


 torch.distributions.transforms。


 Tanh变换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#TanhTransform)[¶](#torch.distributions.transforms.TanhTransform"此定义的永久链接")


 通过映射进行变换


 y = 可疑 ⁡ ( x )


 y = tanh(x)


 y
 



 =
 


 腥味 ( x )




.
 


 它相当于 `` ComposeTransform([AffineTransform(0., 2.), SigmoidTransform(), AffineTransform(-1., 2.)]) `` 但这可能在数值上不稳定，因此建议使用 TanhTransform反而。


 请注意，当涉及 NaN/Inf 值时，应使用 cache_size=1。


*班级*


 torch.distributions.transforms。


 Softmax变换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#SoftmaxTransform)[¶](#torch.distributions.transforms.SoftmaxTransform"此定义的永久链接")


 从无约束空间变换到单纯形过孔


 y = exp ⁡ ( x )


 y = \exp(x)


 y
 



 =
 


 指数 ( x )


 然后正常化。


 这不是双射的，不能用于 HMC。然而，这主要是按坐标进行的(最终归一化除外)，因此适用于按坐标优化算法。


*班级*


 torch.distributions.transforms。


 栈变换


 (*tseq*，*暗淡



 =
 


 0
* , *缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#StackTransform)[¶](#torch.distributions.transforms.StackTransform"此定义的永久链接")


 变换函子，以与 [`torch.stack()`](generated/torch.stack.html#torch.stack "torch.stack") 兼容的方式将一系列变换 tseq 组件按分量应用于昏暗的每个子矩阵。


 例子：


```
x = torch.stack([torch.range(1, 10), torch.range(1, 10)], dim=1)
t = StackTransform([ExpTransform(), identity_transform], dim=1)
y = t(x)

```


*班级*


 torch.distributions.transforms。


 破棍变形


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#StickBreakingTransform)[¶](#torch.distributions.transforms.StickBreakingTransform "此定义的永久链接")


 通过破棍过程从不受约束的空间转变为一个附加维度的单纯形。


 该变换是狄利克雷分布的破棍构造中的迭代 sigmoid 变换：第一个 logit 通过 sigmoid 变换为第一个概率和其他所有概率，然后该过程递归。


 这是双射的并且适合在 HMC 中使用；然而，它将坐标混合在一起，不太适合优化。


*班级*


 torch.distributions.transforms。


 转换


 (*缓存_大小



 =
 


 0
* ) [[source]](_modules/torch/distributions/transforms.html#Transform)[¶](#torch.distributions.transforms.Transform"此定义的永久链接")


 具有可计算 logdet 雅可比矩阵的可逆变换的抽象类。它们主要用于 `torch.distributions.TransformedDistribution` 。


 缓存对于其逆运算成本昂贵或数值不稳定的变换很有用。请注意，必须小心记忆的值，因为自动梯度图可能会颠倒。例如，以下内容在有或没有缓存的情况下都可以工作：


```
y = t(x)
t.log_abs_det_jacobian(x, y).backward()  # x will receive gradients.

```


 然而，由于依赖关系反转，缓存时会出现以下错误：


```
y = t(x)
z = t.inv(y)
grad(z.sum(), [y])  # error because z is x

```


 派生类应该实现 `_call()` 或 `_inverse()` 之一或两者。设置 bijective=True 的派生类还应该实现 [`log_abs_det_jacobian()`](#torch.distributions.transforms.Transform.log_abs_det_jacobian "torch.distributions.transforms.Transform.log_abs_det_jacobian") 。


 Parameters


**cache_size** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 缓存的大小。如果为零，则不进行缓存。如果有，则缓存最新的单个值。仅支持 0 和 1。


 变量 
* **domain** ( [`Constraint`](#torch.distributions.constraints.Constraint "torch.distributions.constraints.Constraint") ) – 表示此转换的有效输入的约束。
* **codomain** ( [`Constraint`](#torch.distributions.constraints.Constraint "torch.distributions.constraints.Constraint") ) – 表示此变换的有效输出的约束，它们是逆变换的输入。
* **双射** ( [
* bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 此变换是否是双射的。变换“t”是双射的，当且仅当对于域中的每个“x”和域中的每个“y”，“t.inv(t(x)) == x”和“t(t.inv(y)) == y”共域。非双射变换至少应保持较弱的伪逆属性 `t(t.inv(t(x)) == t(x)` 和 `t.inv(t(t.inv(y))) == t.inv(y)`.
* **sign** ( [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)")
* or
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 对于双射单变量变换，这应该是 +1 或 -1，具体取决于变换是单调递增还是单调递减。


*财产*


 inv [¶](#torch.distributions.transforms.Transform.inv "此定义的永久链接")


 返回此变换的逆 [`Transform`](#torch.distributions.transforms.Transform "torch.distributions.transforms.Transform")。这应该满足 `t.inv.inv is t` 。


*财产*


 符号[¶](#torch.distributions.transforms.Transform.sign"此定义的永久链接")


 如果适用，返回雅可比行列式的符号。一般来说，这仅对双射变换有意义。


 log_abs_det_jacobian


 ( *x
* , *y
* ) [[source]](_modules/torch/distributions/transforms.html#Transform.log_abs_det_jacobian)[¶](#torch.distributions.transforms.Transform.log_abs_det_jacobian "此定义的永久链接")


 计算 log det jacobian log |dy/dx|给定输入和输出。


 向前_形状


 ( *shape
* ) [[source]](_modules/torch/distributions/transforms.html#Transform.forward_shape)[¶](#torch.distributions.transforms.Transform.forward_shape "此定义的永久链接")


 给定输入形状，推断前向计算的形状。默认为保留形状。


 逆_形状


 ( *shape
* ) [[source]](_modules/torch/distributions/transforms.html#Transform.inverse_shape)[¶](#torch.distributions.transforms.Transform.inverse_shape "此定义的永久链接")


 给定输出形状，推断逆计算的形状。默认为保留形状。


## 约束 [¶](#module-torch.distributions.constraints "此标题的永久链接")


 实施以下约束：



* `constraints.boolean`
* `constraints.cat`
* `constraints.corr_cholesky`
* `constraints.dependent`
* `constraints.greater_than(lower_bound)`
* `constraints.greater_than_eq(lower _bound)`
* `constraints.independent(约束，重新解释_batch_ndims)`
* `constraints.integer_interval(下_界，上_界)`
* `constraints.interval(下_界，上_界)`
* `constraints.interval(下_界，上_界) `
* `constraints.less_than(upper_bound)`
* `constraints.lower_cholesky`
* `constraints.lower_triangle`
* `constraints.multinomial`
* `constraints.nonnegative`
* `constraints.nonnegative_integer` 
* `constraints.one_hot`
* `constraints.positive_integer`
* `constraints.positive`
* `constraints.positive_semidefinite`
* `constraints.positive_definite`
* `constraints.real_vector`
* `约束。真实`
* `constraints.simplex`
* `constraints.对称`
* `constraints.stack`
* `constraints.square`
* `constraints.对称`
* `constraints.unit_interval`


*班级*


 torch.distributions.constraints。


 约束 [[source]](_modules/torch/distributions/constraints.html#Constraint)[¶](#torch.distributions.constraints.Constraint "此定义的永久链接")


 约束的抽象基类。


 约束对象表示变量有效的区域，例如可以在其中优化变量。


 变量 
* **is_discrete** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否受限空间是离散的。默认为 False。
* **event_dim** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)" ) ) – 共同定义事件的最右侧维度的数量。 [`check()`](#torch.distributions.constraints.Constraint.check "torch.distributions.constraints.Constraint.check") 方法将在计算有效性时删除这么多维度。


 check
 


 ( *value
* ) [[source]](_modules/torch/distributions/constraints.html#Constraint.check)[¶](#torch.distributions.constraints.Constraint.check "此定义的永久链接")


 返回“sample_shape 
+ batch_shape”的字节tensor，指示 value 中的每个事件是否满足此约束。


 torch.distributions.constraints。


 cat [¶](#torch.distributions.constraints.cat "此定义的永久链接")


 `_Cat` 的别名


 torch.distributions.constraints。


 dependent_property [¶](#torch.distributions.constraints.dependent_property "此定义的永久链接")


 `_DependentProperty` 的别名


 torch.distributions.constraints。


 大于_than [¶](#torch.distributions.constraints.greater_than"此定义的永久链接")


 `_GreaterThan` 的别名


 torch.distributions.constraints。


 大于_than_eq [¶](#torch.distributions.constraints.greater_than_eq "此定义的永久链接")


 `_GreaterThanEq` 的别名


 torch.distributions.constraints。


 独立 [¶](#torch.distributions.constraints.independent"此定义的永久链接")


 `_IndependentConstraint` 的别名


 torch.distributions.constraints。


 integer_interval [¶](#torch.distributions.constraints.integer_interval "此定义的永久链接")


 `_IntegerInterval` 的别名


 torch.distributions.constraints。


 间隔 [¶](#torch.distributions.constraints.interval "此定义的永久链接")


 `_Interval` 的别名


 torch.distributions.constraints。


 half_open_interval [¶](#torch.distributions.constraints.half_open_interval "此定义的永久链接")


 别名或 `_HalfOpenInterval`


 torch.distributions.constraints。


 less_than [¶](#torch.distributions.constraints.less_than "此定义的永久链接")


 `_LessThan` 的别名


 torch.distributions.constraints。


 多项式 [¶](#torch.distributions.constraints.multinomial "此定义的永久链接")


 `_Multinomial` 的别名


 torch.distributions.constraints。


 stack [¶](#torch.distributions.constraints.stack"此定义的永久链接")


 `_Stack` 的别名


## 约束注册表 [¶](#module-torch.distributions.constraint_registry "此标题的永久链接")


 PyTorch 提供了两个链接 [`Constraint`](#torch.distributions.constraints.Constraint "torch.distributions.ConstraintRegistry") 的全局 ConstraintRegistry 对象。 constraints.Constraint") 对象到 [`Transform`](#torch.distributions.transforms.Transform "torch.distributions.transforms.Transform") 对象。这些对象都有输入约束和返回变换，但它们对双射性有不同的保证。


1. `biject_to(constraint)` 从 `constraints.real` 查找一个双射 [`Transform`](#torch.distributions.transforms.Transform "torch.distributions.transforms.Transform") 到给定的 `constraint` 。返回的变换保证具有 `.bijective = True` 并且应该实现 `.log_abs_det_jacobian()`.2。 `transform_to(constraint)` 从 `constraints.real` 查找一个不一定是双射的 [`Transform`](#torch.distributions.transforms.Transform "torch.distributions.transforms.Transform") 到给定的 `constraint `。返回的转换不保证实现 `.log_abs_det_jacobian()` 。


 `transform_to()` 注册表对于对概率分布的约束参数执行无约束优化很有用，这些参数由每个分布的 `.arg_constraints` 字典表示。这些变换通常会过度参数化空间以避免旋转；因此它们更适合像 Adam 这样的坐标优化算法：


```
loc = torch.zeros(100, requires_grad=True)
unconstrained = torch.zeros(100, requires_grad=True)
scale = transform_to(Normal.arg_constraints['scale'])(unconstrained)
loss = -Normal(loc, scale).log_prob(data).sum()

```


 “biject_to()”注册表对于哈密顿蒙特卡洛很有用，其中来自受约束“.support”的概率分布的样本在不受约束的空间中传播，并且算法通常是旋转不变的。：


```
dist = Exponential(rate)
unconstrained = torch.zeros(100, requires_grad=True)
sample = biject_to(dist.support)(unconstrained)
potential_energy = -dist.log_prob(sample).sum()

```


!!! note "笔记"

    `transform_to` 和 `biject_to` 不同的一个例子是 `constraints.simplex` ： `transform_to(constraints.simplex)` 返回一个 [`SoftmaxTransform`](#torch.distributions.transforms.SoftmaxTransform "torch".distributions.transforms.SoftmaxTransform") 简单地对其输入求幂并标准化；这是一种廉价且主要是坐标操作，适用于 SVI 等算法。相比之下， `biject_to(constraints.simplex)` 返回一个 [`StickBreakingTransform`](#torch.distributions.transforms.StickBreakingTransform "torch.distributions.transforms.StickBreakingTransform")，它将其输入向下投射到一维空间;这是一种成本更高、数值稳定性较差的变换，但对于 HMC 等算法来说是必需的。


 `biject_to` 和 `transform_to` 对象可以通过用户定义的约束进行扩展，并使用它们的 `.register()` 方法作为单例约束上的函数进行转换：


```
transform_to.register(my_constraint, my_transform)

```


 或者作为参数化约束的装饰器：


```
@transform_to.register(MyConstraintClass)
def my_factory(constraint):
    assert isinstance(constraint, MyConstraintClass)
    return MyTransform(constraint.param1, constraint.param2)

```


 您可以通过创建新的 [`ConstraintRegistry`](#torch.distributions.constraint_registry.ConstraintRegistry "torch.distributions.constraint_registry.ConstraintRegistry") 对象来创建自己的注册表。


*班级*


 torch.distributions.constraint_registry。


 ConstraintRegistry [[source]](_modules/torch/distributions/constraint_registry.html#ConstraintRegistry)[¶](#torch.distributions.constraint_registry.ConstraintRegistry"此定义的永久链接")


 将约束链接到转换的注册表。


 登记


 (*约束*，*工厂



 =
 


 无
* ) [[source]](_modules/torch/distributions/constraint_registry.html#ConstraintRegistry.register)[¶](#torch.distributions.constraint_registry.ConstraintRegistry.register "此定义的永久链接")


 在此注册表中注册一个 [`Constraint`](#torch.distributions.constraints.Constraint "torch.distributions.constraints.Constraint") 子类。用法：


```
@my_registry.register(MyConstraintClass)
def construct_transform(constraint):
    assert isinstance(constraint, MyConstraint)
    return MyTransform(constraint.arg_constraints)

```


 参数 
* **constraint** ([`Constraint`](#torch.distributions.constraints.Constraint "torch.distributions.constraints.Constraint") 的子类) – [`Constraint`](#torch.distributions.Constraint) 的子类。 constraint "torch.distributions.constraints.Constraint") ，或所需类的单例对象。
* **factory** ( *Callable
* ) – 输入约束对象并返回 [`Transform`](# torch.distributions.transforms.Transform“torch.distributions.transforms.Transform”)对象。