# torch.utils.data [¶](#module-torch.utils.data "此标题的永久链接") =

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/data>
>
> 原始地址：<https://pytorch.org/docs/stable/data.html>


 PyTorch 数据加载实用程序的核心是 [`torch.utils.data.DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 类。它表示数据集上的 Python 可迭代，支持



* [地图样式和可迭代样式数据集](#dataset-types) ,
* [自定义数据加载顺序](#data-loading-order-and-sampler) ,
* [自动批处理](#loading-batched-and -non-batched-data) ,
* [单进程和多进程数据加载](#single-and-multi-process-data-loading) ,
* [自动内存固定](#memory-pinning) 。


 这些选项由 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的构造函数参数配置，其签名为：


```
DataLoader(dataset, batch_size=1, shuffle=False, sampler=None,
           batch_sampler=None, num_workers=0, collate_fn=None,
           pin_memory=False, drop_last=False, timeout=0,
           worker_init_fn=None, *, prefetch_factor=2,
           persistent_workers=False)

```


 以下部分详细描述了这些选项的效果和用法。


## 数据集类型 [¶](#dataset-types "此标题的永久链接")


 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 构造函数最重要的参数是 `dataset` ，它指示要从中加载数据的数据集对象。 PyTorch 支持两种不同类型的数据集：



* [地图样式数据集](#map-style-datasets) ,
* [可迭代样式数据集](#iterable-style-datasets) 。


### 地图样式数据集 [¶](#map-style-datasets "此标题的永久链接")


 地图样式数据集是实现“__getitem__()”和“__len__()”协议的数据集，并表示来自(可能是非整数)索引/的地图数据样本的键。


 例如，这样的数据集，当使用“dataset[idx]”访问时，可以从磁盘上的文件夹中读取“idx”图像及其相应的标签。


 有关更多详细信息，请参阅[`Dataset`](#torch.utils.data.Dataset "torch.utils.data.Dataset")。


### 可迭代式数据集 [¶](#iterable-style-datasets "永久链接到此标题")


 可迭代样式数据集是 [`IterableDataset`](#torch.utils.data.IterableDataset "torch.utils.data.IterableDataset") 子类的实例，它实现了 `__iter__() ` 协议，表示可迭代的数据样本。这种类型的数据集特别适合随机读取成本昂贵甚至不可能的情况，以及批量大小取决于获取的数据的情况。


 例如，这样的数据集，当调用“iter(dataset)”时​​，可以返回从数据库、远程服务器甚至实时生成的日志读取的数据流。


 有关更多详细信息，请参阅 [`IterableDataset`](#torch.utils.data.IterableDataset "torch.utils.data.IterableDataset")。




!!! note "笔记"

    将 [`IterableDataset`](#torch.utils.data.IterableDataset "torch.utils.data.IterableDataset") 与 [多进程数据加载](#multi-process-data-loading) 一起使用时。相同的数据集对象在每个工作进程上复制，因此必须对副本进行不同的配置以避免重复数据。请参阅 [`IterableDataset`](#torch.utils.data.IterableDataset "torch.utils.data.IterableDataset") 文档了解如何实现此目的。


## 数据加载顺序和 [`Sampler`](#torch.utils.data.Sampler "torch.utils.data.Sampler")[¶](#data-loading-order-and-sampler "永久链接到此标题")
- 


 对于 [iterable-style-datasets](#iterable-style-datasets) ，数据加载顺序完全由用户定义的 iterable 控制。这允许更轻松地实现块读取和动态批量大小(例如，通过每次生成批量样本)。


 本节的其余部分涉及 [地图样式数据集](#map-style-datasets) 的情况。 [`torch.utils.data.Sampler`](#torch.utils.data.Sampler "torch.utils.data.Sampler") 类用于指定数据加载中使用的索引/键的序列。它们表示可迭代对象超过数据集的索引。例如，在随机梯度下降 (SGD) 的常见情况下，[`Sampler`](#torch.utils.data.Sampler "torch.utils.data.Sampler") 可以随机排列一系列索引并以时间，或者为小批量 SGD 产生少量。


 将根据 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的 `shuffle` 参数自动构建顺序或随机采样器。或者，用户可以使用 ` Sampler` 参数指定一个自定义 [`Sampler`](#torch.utils.data.Sampler "torch.utils.data.Sampler") 对象，该对象每次都会生成要获取的下一个索引/键。


 一次生成批量索引列表的自定义 [`Sampler`](#torch.utils.data.Sampler "torch.utils.data.Sampler") 可以作为 `batch_sampler` 参数传递。也可以通过“batch_size”和“drop_last”参数启用。有关详细信息，请参阅[下一节](#loading-batched-and-non-batched-data)。




!!! note "笔记"

    “sampler”和“batch_sampler”都不与可迭代式数据集兼容，因为此类数据集没有键或索引的概念。


## 加载批处理和非批处理数据 [¶](#loading-batched-and-non-batched-data "永久链接到此标题")


[`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 支持通过参数 `batch_size` 、 `drop_last` 、 `batch_sampler 自动将单独获取的数据样本整理到批次中` 和 `collat​​e_fn` (具有默认函数)。


### 自动批处理(默认)[¶](#automatic-batching-default "永久链接到此标题")


 这是最常见的情况，对应于获取小批量数据并将它们整理成批量样本，即包含其中一个维度为批量维度(通常是第一个维度)的tensor。


 当“batch_size”(默认为“1”)不是“None”时，数据加载器将生成批量样本而不是单个样本。 `batch_size` 和 `drop_last` 参数用于指定数据加载器如何获取批量的数据集键。对于地图样式的数据集，用户也可以指定“batch_sampler”，它一次生成一个键列表。




!!! note "笔记"

    `batch_size` 和 `drop_last` 参数本质上用于从 `sampler` 构造一个 `batch_sampler` 。对于地图样式数据集，“sampler”要么由用户提供，要么基于“shuffle”参数构造。对于可迭代式数据集，“采样器”是一个虚拟的无限采样器。有关采样器的更多详细信息，请参阅[本节](#data-loading-order-and-sampler)。


!!! note "笔记"

    当使用 [multi-processing](#multi-process-data-loading) 从 [iterable-style datasets](#iterable-style-datasets) 获取数据时， `drop_last` 参数会删除每个数据集的最后一个非完整批次工作人员的数据集副本。


 使用采样器中的索引获取样本列表后，作为“collat​​e_fn”参数传递的函数用于将样本列表整理成批次。


 在这种情况下，从地图样式数据集加载大致相当于：


```
for indices in batch_sampler:
    yield collate_fn([dataset[i] for i in indices])

```


 从可迭代式数据集加载大致相当于：


```
dataset_iter = iter(dataset)
for indices in batch_sampler:
    yield collate_fn([next(dataset_iter) for _ in indices])

```


 自定义“collat​​e_fn”可用于自定义排序规则，例如，将顺序数据填充到批次的最大长度。有关“collat​​e_fn”的更多信息，请参阅[本节](#dataloader-collat​​e-fn)。


### 禁用自动批处理 [¶](#disable-automatic-batching "永久链接到此标题")


 在某些情况下，用户可能希望在数据集代码中手动处理批处理，或者只是加载单个样本。例如，直接加载批量数据(例如，从数据库批量读取或读取连续的内存块)可能会更便宜，或者批量大小取决于数据，或者程序设计为处理单个样本。在这些情况下，最好不要使用自动批处理(其中使用“collat​​e_fn”来整理样本)，而是让数据加载器直接返回“dataset”对象的每个成员。


 当“batch_size”和“batch_sampler”均为“None”(“batch_sampler”的默认值已经是“None”)时，自动批处理将被禁用。从“dataset”获得的每个样本都使用作为“collat​​e_fn”参数传递的函数进行处理。


**当自动批处理被禁用**时，默认的 `collat​​e_fn` 只是将 NumPy 数组转换为 PyTorch tensor，并保持其他所有内容不变。


 在这种情况下，从地图样式数据集加载大致相当于：


```
for index in sampler:
    yield collate_fn(dataset[index])

```


 从可迭代式数据集加载大致相当于：


```
for data in iter(dataset):
    yield collate_fn(data)

```


 有关“collat​​e_fn”的更多信息，请参阅[本节](#dataloader-collat​​e-fn)。


### 使用 `collat​​e_fn`[¶](#working-with-collat​​e-fn "永久链接到此标题")


 启用或禁用自动批处理时，“collat​​e_fn”的使用略有不同。


**当自动批处理被禁用**时，会对每个单独的数据样本调用“collat​​e_fn”，并从数据加载迭代器产生输出。在这种情况下，默认的“collat​​e_fn”只是将 NumPyarray 转换为 PyTorch tensor。


**当启用自动批处理**时，每次都会使用数据样本列表调用“collat​​e_fn”。预计会将输入样本整理成一批，以便从数据加载器迭代器中生成。本节的其余部分描述默认 `collat​​e_fn` ( [`default_collat​​e()`](#torch.utils.data.default_collat​​e "torch.utils.data.default_collat​​e") )的行为。


 例如，如果每个数据样本由一个 3 通道图像和一个整体类标签组成，即数据集的每个元素返回一个元组 `(image, class_index)` ，则默认的 `collat​​e_fn` 会整理这样的列表元组转换为批处理图像tensor和批处理类标签tensor的单个元组。特别是，默认的“collat​​e_fn”具有以下属性：



* 它总是在前面添加一个新维度作为批量维度。 
* 它自动将 NumPy 数组和 Python 数值转换为 PyTorch Tensors。 
* 它保留数据结构，例如，如果每个样本是一个字典，它会输出一个具有相同键集的字典但将tensor批量化为值(如果值无法转换为tensor则列出)。对于 `list` 、 `tuple` 、 `namedtuple` 等也是如此。


 用户可以使用自定义的“collat​​e_fn”来实现自定义批处理，例如，沿第一个维度以外的维度进行排序、填充不同长度的序列或添加对自定义数据类型的支持。


 如果您遇到 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的输出的维度或类型与您的期望不同的情况，您可能需要检查您的`整理_fn` 。


## 单进程和多进程数据加载 [¶](#single-and-multi-process-data-loading "永久链接到此标题")


 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 默认使用单进程数据加载。


 在 Python 进程中，[全局解释器锁 (GIL)]​​(https://wiki.python.org/moin/GlobalInterpreterLock) 会阻止跨线程真正完全并行化 Python 代码。为了避免数据加载时阻塞计算代码，PyTorch 提供了一个简单的切换来执行多进程数据加载，只需将参数“num_workers”设置为正整数即可。


### 单进程数据加载(默认)[¶](#single-process-data-loading-default "永久链接到此标题")


 在此模式下，数据获取是在初始化 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的同一进程中完成的。因此，数据加载可能会阻塞计算。然而，当用于在进程之间共享数据的资源(例如共享内存、文件描述符)有限时，或者当整个数据集很小并且可以完全加载到内存中时，这种模式可能是首选。此外，单进程加载通常会显示更易读的错误跟踪，因此对于调试很有用。


### 多进程数据加载 [¶](#multi-process-data-loading "永久链接到此标题")


 将参数“num_workers”设置为正整数将打开具有指定数量的加载器工作进程的多进程数据加载。


!!! warning "警告"

     经过几次迭代后，加载器工作进程将消耗与父进程相同数量的 CPU 内存，用于父进程中从工作进程访问的所有 Python 对象。如果数据集包含大量数据(例如，您在数据集构建时加载非常大的文件名列表)和/或您正在使用大量工作人员(总体内存使用量是“工作人员数量*父级的大小”)，这可能会出现问题过程`)。最简单的解决方法是用非引用计数表示替换 Python 对象，例如 Pandas、Numpy 或 PyArrow 对象。请查看 [issue #13246](https://github.com/pytorch/pytorch/issues/13246#issuecomment-905703662)，了解有关发生这种情况的原因的更多详细信息以及如何解决这些问题的示例代码。


 在此模式下，每次创建 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的迭代器时(例如，当您调用 `enumerate(dataloader)` 时)，创建了“num_workers”工作进程。此时，`dataset`、`collat​​e_fn` 和 `worker_init_fn` 被传递给每个worker，它们用于初始化和获取数据。这意味着数据集访问及其内部 IO、转换(包括“collat​​e_fn”)在工作进程中运行。


[`torch.utils.data.get_worker_info()`](#torch.utils.data.get_worker_info "torch.utils.data.get_worker_info") 返回工作进程中的各种有用信息(包括工作进程 ID、数据集副本、初始种子等)，并在主进程中返回“None”。用户可以在数据集代码和/或“worker_init_fn”中使用此函数来单独配置每个数据集副本，并确定代码是否在工作进程中运行。例如，这对于分割数据集特别有帮助。


 对于地图样式数据集，主进程使用“sampler”生成索引并将其发送给工作人员。因此，任何洗牌随机化都是在主进程中完成的，该进程通过分配要加载的索引来指导加载。


 对于可迭代样式的数据集，由于每个工作进程都会获取“dataset”对象的副本，因此简单的多进程加载通常会导致重复的数据。使用 [`torch.utils.data.get_worker_info()`](#torch.utils.data.get_worker_info "torch.utils.data.get_worker_info") 和/或 `worker_init_fn` ，用户可以独立配置每个副本。 (有关如何实现此目的，请参阅 [`IterableDataset`](#torch.utils.data.IterableDataset "torch.utils.data.IterableDataset") 文档。)出于类似的原因，在多进程加载中，`drop_last` 参数删除每个工作线程的可迭代式数据集副本的最后一个非完整批次。


 一旦到达迭代结束，或者当迭代器被垃圾回收时，工作线程就会被关闭。


!!! warning "警告"

     通常不建议在多进程加载中返回 CUDA tensor，因为在多处理中使用 CUDA 和共享 CUDA tensor有很多微妙之处(请参阅 [多处理中的 CUDA](notes/multiprocessing.html#multiprocessing-cuda-note) )。相反，我们建议使用[自动内存固定](#memory-pinning)(即设置 `pin_memory=True` )，这样可以将数据快速传输到支持 CUDA 的 GPU。


#### 特定于平台的行为 [¶](#platform-specific-behaviors "永久链接到此标题")


 由于工作程序依赖于 Python [`multiprocessing`](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing "(in Python v3.12)") ，因此与 Windows 相比，工作程序启动行为有所不同到 Unix。



* 在 Unix 上， `fork()` 是默认的 [`multiprocessing`](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing "(in Python v3.12)") 启动使用 `fork()` ，子进程通常可以通过克隆的地址空间直接访问 `dataset` 和 Python 参数函数。*在 Windows 或 MacOS 上，`spawn()` 是默认的 [`multiprocessing`](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing "(in Python v3.12)") start 方法。使用 `spawn()` ，启动另一个解释器来运行您的主脚本，如下通过内部工作函数通过 [`pickle`](https://docs.python.org/3/library/pickle.html#module-pickle "(在 Python v3.12)") 序列化中。


 这种单独的序列化意味着您应该采取两个步骤来确保在使用多进程数据加载时与 Windows 兼容：



* 将大部分主脚本代码包装在 `if __name__ == '__main__':` 块中，以确保它不会再次运行(最有可能生成错误)当每个工作进程启动时。您可以将数据集和 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 实例创建逻辑放置在此处，因为它不需要在工作线程中重新执行。*确保任何自定义 `collat​​e_fn` 、 `worker_init_fn` 或 `dataset` 代码都声明为顶级定义，位于 `__main__` 检查之外。这确保它们在工作进程中可用。(这是必需的，因为函数仅作为引用而不是“字节码”进行腌制。)


#### 多进程数据加载中的随机性 [¶](#randomness-in-multi-process-data-loading "永久链接到此标题")


 默认情况下，每个worker都会将其PyTorch种子设置为“base_seed +worker_id”，其中“base_seed”是由主进程使用其RNG(从而强制消耗RNG状态)或指定的long生成的`发电机`。然而，其他库的种子可能会在初始化工作人员时重复，导致每个工作人员返回相同的随机数。 (请参阅常见问题解答中的[本节](notes/faq.html#dataloader-workers-random-seed)。)。


 在 `worker_init_fn` 中，您可以使用 [`torch.utils.data.get_worker_info().seed`](#torch.utils.data.get_worker_info "torch.utils.data.get_worker_info") 或 [`torch.initial_seed()`](generated/torch.initial_seed.html#torch.initial_seed "torch.initial_seed") ，并在数据加载之前使用它为其他库提供种子。


## 内存固定 [¶](#memory-pinning "此标题的永久链接")


 当主机到 GPU 的副本源自固定(页面锁定)内存时，速度要快得多。有关通常何时以及如何使用固定内存的更多详细信息，请参阅[使用固定内存缓冲区](notes/cuda.html#cuda-memory-pinning)。


 对于数据加载，将 `pin_memory=True` 传递给 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 将自动将获取的 dataTensors 放入固定内存中，从而可以更快地将数据传输到支持 CUDA 的 GPU。


 默认内存固定逻辑仅识别tensor以及包含tensor的映射和迭代。默认情况下，如果固定逻辑看到一个自定义类型的批次(如果您有返回自定义批次类型的“collat​​e_fn”，则会发生这种情况)，或者如果批次的每个元素都是自定义类型，则固定逻辑将不会识别它们，它将返回该批次(或那些元素)而不固定内存。要为自定义批次或数据类型启用内存固定，请在自定义类型上定义“pin_memory()”方法。


 请参阅下面的示例。


 例子：


```
class SimpleCustomBatch:
    def __init__(self, data):
        transposed_data = list(zip(*data))
        self.inp = torch.stack(transposed_data[0], 0)
        self.tgt = torch.stack(transposed_data[1], 0)

    # custom memory pinning method on custom type
    def pin_memory(self):
        self.inp = self.inp.pin_memory()
        self.tgt = self.tgt.pin_memory()
        return self

def collate_wrapper(batch):
    return SimpleCustomBatch(batch)

inps = torch.arange(10 * 5, dtype=torch.float32).view(10, 5)
tgts = torch.arange(10 * 5, dtype=torch.float32).view(10, 5)
dataset = TensorDataset(inps, tgts)

loader = DataLoader(dataset, batch_size=2, collate_fn=collate_wrapper,
                    pin_memory=True)

for batch_ndx, sample in enumerate(loader):
    print(sample.inp.is_pinned())
    print(sample.tgt.is_pinned())

```


*班级*


 火炬.utils.数据。


 数据加载器


 ( *数据集
* , *batch_size



 =
 


 1
* , *随机播放



 =
 


 无
* , *采样器



 =
 


 无
* , *batch_sampler



 =
 


 无
* , *num_workers



 =
 


 0
* , *整理_fn



 =
 


 无
* , *pin_memory



 =
 


 假
* , *drop_last



 =
 


 假*，*超时



 =
 


 0
* , *worker_init_fn



 =
 


 无
* , *多处理_context



 =
 


 无
* , *生成器



 =
 


 无
* 、 *** 、 *预取_factor



 =
 


 无
* , *持久_workers



 =
 


 False
* , *pin_memory_device



 =
 


 ''
* ) [[source]](_modules/torch/utils/data/dataloader.html#DataLoader)[¶](#torch.utils.data.DataLoader "此定义的永久链接")


 数据加载器。组合数据集和采样器，并在给定数据集上提供可迭代。


 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 支持地图样式和可迭代样式数据集，具有单进程或多进程加载、自定义加载顺序和可选的自动批处理(排序规则)和内存固定。


 有关更多详细信息，请参阅 [`torch.utils.data`](#module-torch.utils.data "torch.utils.data") 文档页面。


 参数 
* **dataset** ( [*Dataset*](#torch.utils.data.Dataset "torch.utils.data.Dataset") ) – 从中加载数据的数据集。
* **batch_size** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*,
* *可选
* ) – 每批有多少个样本加载(默认：`1`)。
* **随机播放**([*bool*](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”) )*,
* *可选
* ) – 设置为 `True` 以使数据在每个时期重新洗牌(默认值：`False` ).
* **sampler** ( [*Sampler*](#torch.utils.data.Sampler "torch.utils.data.Sampler")*或
* *Iterable
* *,
* *可选
* ) – 定义从数据集中抽取样本的策略。可以是任何实现了“__len__”的“Iterable”。如果指定，则不得指定 `shuffle`。
* **batch_sampler** ( [*Sampler*](#torch.utils.data.Sampler "torch.utils.data.Sampler")*或
* *Iterable
* *,
* *可选
* ) – 类似于 `sampler` ，但一次返回一批索引。与 `batch_size` 、 `shuffle` 、 `sampler` 和 `drop_last` 互斥。
* **num_workers** ( [*int*](https://docs.python.org/3 /library/functions.html#int "(in Python v3.12)")*,
* *可选
* ) – 用于数据加载的子进程数。 `0` 表示数据将在主进程中加载​​。(默认值: `0` )
* **collat​​e_fn** ( *Callable
* *,
* *可选
* ) – 合并样本列表以形成 amini 
- 一批tensor。使用从 amap 样式数据集批量加载时使用。
* **pin_memory** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中).12)")*,
* *可选
* ) – 如果 `True` ，数据加载器将在返回之前将 Tensors 复制到设备/CUDA 固定内存中。如果您的数据元素是自定义类型，或者您的“collat​​e_fn”返回自定义类型的批次，请参阅下面的示例。
* **drop_last** ( [*bool*](https://docs. python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 设置为 `True` 以删除最后一个不完整的批次(如果数据集大小不可整除)按批量大小。如果“False”并且数据集的大小不能被批次大小整除，则最后一个批次将会更小。 (默认值：`False`)
* **超时**(*数字
* *,
* *可选*) – 如果为正，则为从工作人员收集批次的超时值。应始终为非负数。 (默认： `0` )
* **worker_init_fn** ( *Callable
* *,
* *可选
* ) – 如果不是 `None` ，这将在每个worker子进程上使用worker id(一个 int )被调用`[0, num_workers 
- 1]` ) 作为输入，在播种之后和数据加载之前。 (默认值：`None`)
* **multiprocessing_context** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)" )*或
* *multiprocessing.context.BaseContext
* *,
* *可选
* ) – 如果为 `None` ，则默认 [多处理上下文](https://docs.python.org/3/library/multiprocessing.html#contexts将使用操作系统的 -and-start-methods)。 (默认值：`无`)
* **生成器** ( [*torch.Generator*](generated/torch.Generator.html#torch.Generator "torch.Generator")*,
* *可选
* ) – 如果不是 ` None`，该 RNG 将被 RandomSampler 用于生成随机索引和多重处理，以便为工作人员生成“base_seed”。 (默认值：`None`)
* **预取_factor** ( [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)" )*,
* *可选
* *,
* *仅限关键字参数
* ) – 每个工作人员提前加载的批次数。 “2”表示所有工作人员总共预取 2 个 * num_workers 批次。 (默认值取决于 num_workers 的设置值。如果 num_workers=0 默认为 `None` 。否则，如果 `num_workers 
> 0` 默认为 `2` ).
* **持久\ _workers** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(Python v3.12)")*,
* *可选
* ) – 如果 `True ` ，数据加载器在数据集被消耗一次后不会关闭工作进程。这允许保持工作数据集实例处于活动状态。 (默认: `False` )
* **pin_memory_device** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中) )")*,
* *可选
* ) – 如果 `pin_memory` 为 `True` 则`pin_memory` 的设备。


!!! warning "警告"

     如果使用“spawn”启动方法，“worker_init_fn”不能是不可拾取的对象，例如 lambda 函数。有关 PyTorch 中多重处理的更多详细信息，请参阅[多重​​处理最佳实践](notes/multiprocessing.html#multiprocessing-best-practices)。


!!! warning "警告"

    `len(dataloader)` 启发式基于所使用的采样器的长度。当 `dataset` 是一个 [`IterableDataset`](#torch.utils.data.IterableDataset "torch.utils.data.IterableDataset") 时，它会改为返回基于 `len(dataset) /batch_size` 的估计值，并根据 `drop_last` 进行适当舍入，无论多进程加载配置如何。这代表了 PyTorch 可以做出的最佳猜测，因为 PyTorch 信任用户“数据集”代码正确处理多进程加载以避免重复数据。


 但是，如果分片导致多个工作人员的最后一批不完整，则此估计仍然可能不准确，因为(1)原本完整的批次可能会被分成多个批次，并且(2)当“drop”时，可能会丢弃超过一批的样本。 _last` 已设置。不幸的是，PyTorch 一般无法检测到此类情况。


 有关这两种类型的数据集以及 [`IterableDataset`](#torch.utils.data.IterableDataset "torch.utils.data.IterableDataset") 如何与 [Multi 交互] 的更多详细信息，请参阅[数据集类型](#dataset-types) -处理数据加载](#multi-process-data-loading) 。


!!! warning "警告"

     请参阅 [再现性](notes/randomness.html#reproducibility) 和 [我的数据加载器工作程序返回相同的随机数](notes/faq.html#dataloader-workers-random-seed) 和 [多进程数据加载中的随机性](#data-loading-randomness) 随机种子相关问题的注释。


*班级*


 火炬.utils.数据。


 数据集


 (
 
*\*
 


 Parameters
* , ***


 kwds
* ) [[source]](_modules/torch/utils/data/dataset.html#Dataset)[¶](#torch.utils.data.Dataset "此定义的永久链接")


 表示 [`Dataset`](#torch.utils.data.Dataset "torch.utils.data.Dataset") 的抽象类。


 所有表示从键到数据样本的映射的数据集都应该对其进行子分类。所有子类都应该覆盖 `__getitem__()` ，支持获取给定键的数据样本。子类还可以选择覆盖 `__len__()` ，预计它将返回数据集的大小 [`Sampler`](#torch.utils.data.Sampler "torch.utils.data.Sampler") 实现和 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的默认选项。子类还可以选择实现 `__getitems__()` ，以加速批量样本加载。该方法接受批次样本索引列表并返回样本列表。




!!! note "笔记"

    [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 默认情况下构造一个生成整数索引的索引采样器。为了使其与具有非整数索引/键的地图样式数据集一起使用，必须提供自定义采样器。


*班级*


 火炬.utils.数据。


 可迭代数据集


 (
 
*\*
 


 Parameters
* , ***


 kwds
* ) [[source]](_modules/torch/utils/data/dataset.html#IterableDataset)[¶](#torch.utils.data.IterableDataset "此定义的永久链接")


 可迭代的数据集。


 所有表示数据样本可迭代的数据集都应该对其进行子类化。当数据来自流时，这种形式的数据集特别有用。


 所有子类都应覆盖 `__iter__()` ，这将返回此数据集中样本的迭代器。


 当子类与 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 一起使用时，数据集中的每个项目都将从 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 迭代器。当 num_workers 
> 0 时，每个工作进程将拥有数据集对象的不同副本，因此通常需要独立配置每个副本以避免从工作进程返回重复的数据。 [`get_worker_info()`](#torch.utils.data.get_worker_info "torch.utils.data.get_worker_info") 在工作进程中调用时，返回有关工作人员的信息。它可以在数据集的 `__iter__()` 方法或 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的 `worker 中使用_init_fn` 选项来修改每个副本的行为。


 示例 1：在 `__iter__()` 中的所有工作人员之间分配工作负载：


```
>>> class MyIterableDataset(torch.utils.data.IterableDataset):
...     def __init__(self, start, end):
...         super(MyIterableDataset).__init__()
...         assert end > start, "this example code only works with end >= start"
...         self.start = start
...         self.end = end
...
...     def __iter__(self):
...         worker_info = torch.utils.data.get_worker_info()
...         if worker_info is None:  # single-process data loading, return the full iterator
...             iter_start = self.start
...             iter_end = self.end
...         else:  # in a worker process
...             # split workload
...             per_worker = int(math.ceil((self.end - self.start) / float(worker_info.num_workers)))
...             worker_id = worker_info.id
...             iter_start = self.start + worker_id * per_worker
...             iter_end = min(iter_start + per_worker, self.end)
...         return iter(range(iter_start, iter_end))
...
>>> # should give same set of data as range(3, 7), i.e., [3, 4, 5, 6].
>>> ds = MyIterableDataset(start=3, end=7)

>>> # Single-process loading
>>> print(list(torch.utils.data.DataLoader(ds, num_workers=0)))
[tensor([3]), tensor([4]), tensor([5]), tensor([6])]

>>> # Mult-process loading with two worker processes
>>> # Worker 0 fetched [3, 4]. Worker 1 fetched [5, 6].
>>> print(list(torch.utils.data.DataLoader(ds, num_workers=2)))
[tensor([3]), tensor([5]), tensor([4]), tensor([6])]

>>> # With even more workers
>>> print(list(torch.utils.data.DataLoader(ds, num_workers=12)))
[tensor([3]), tensor([5]), tensor([4]), tensor([6])]

```


 示例 2：使用 `worker_init_fn` 将工作负载分配给所有工作人员：


```
>>> class MyIterableDataset(torch.utils.data.IterableDataset):
...     def __init__(self, start, end):
...         super(MyIterableDataset).__init__()
...         assert end > start, "this example code only works with end >= start"
...         self.start = start
...         self.end = end
...
...     def __iter__(self):
...         return iter(range(self.start, self.end))
...
>>> # should give same set of data as range(3, 7), i.e., [3, 4, 5, 6].
>>> ds = MyIterableDataset(start=3, end=7)

>>> # Single-process loading
>>> print(list(torch.utils.data.DataLoader(ds, num_workers=0)))
[3, 4, 5, 6]
>>>
>>> # Directly doing multi-process loading yields duplicate data
>>> print(list(torch.utils.data.DataLoader(ds, num_workers=2)))
[3, 3, 4, 4, 5, 5, 6, 6]

>>> # Define a `worker_init_fn` that configures each dataset copy differently
>>> def worker_init_fn(worker_id):
...     worker_info = torch.utils.data.get_worker_info()
...     dataset = worker_info.dataset  # the dataset copy in this worker process
...     overall_start = dataset.start
...     overall_end = dataset.end
...     # configure the dataset to only process the split workload
...     per_worker = int(math.ceil((overall_end - overall_start) / float(worker_info.num_workers)))
...     worker_id = worker_info.id
...     dataset.start = overall_start + worker_id * per_worker
...     dataset.end = min(dataset.start + per_worker, overall_end)
...

>>> # Mult-process loading with the custom `worker_init_fn`
>>> # Worker 0 fetched [3, 4]. Worker 1 fetched [5, 6].
>>> print(list(torch.utils.data.DataLoader(ds, num_workers=2, worker_init_fn=worker_init_fn)))
[3, 5, 4, 6]

>>> # With even more workers
>>> print(list(torch.utils.data.DataLoader(ds, num_workers=12, worker_init_fn=worker_init_fn)))
[3, 4, 5, 6]

```


*班级*


 火炬.utils.数据。


 tensor数据集


 (
 
*\*
 


 tensor
* ) [[source]](_modules/torch/utils/data/dataset.html#TensorDataset)[¶](#torch.utils.data.TensorDataset "此定义的永久链接")


 数据集包装tensor。


 每个样本将通过沿第一维索引tensor来检索。


 Parameters


***tensors** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 与第一维大小相同的tensor。


*班级*


 火炬.utils.数据。


 堆栈数据集


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/utils/data/dataset.html#StackDataset)[¶](#torch.utils.data.StackDataset "此定义的永久链接")


 数据集作为多个数据集的堆叠。


 此类对于组装以数据集形式给出的复杂输入数据的不同部分非常有用。


 例子


```
>>> images = ImageDataset()
>>> texts = TextDataset()
>>> tuple_stack = StackDataset(images, texts)
>>> tuple_stack[0] == (images[0], texts[0])
>>> dict_stack = StackDataset(image=images, text=texts)
>>> dict_stack[0] == {'image': images[0], 'text': texts[0]}

```


 参数 
* ***args** ( [*Dataset*](#torch.utils.data.Dataset "torch.utils.data.Dataset") ) – 以元组形式返回的用于堆叠的数据集。
* **** kwargs** ( [*Dataset*](#torch.utils.data.Dataset "torch.utils.data.Dataset") ) – 用于堆叠的数据集以字典形式返回。


*班级*


 火炬.utils.数据。


 连接数据集


 ( *datasets
* ) [[source]](_modules/torch/utils/data/dataset.html#ConcatDataset)[¶](#torch.utils.data.ConcatDataset "此定义的永久链接")


 数据集作为多个数据集的串联。


 此类对于组装不同的现有数据集很有用。


 Parameters


**datasets** ( *sequence
* ) – 要连接的数据集列表


*班级*


 火炬.utils.数据。


 链数据集


 ( *datasets
* ) [[source]](_modules/torch/utils/data/dataset.html#ChainDataset)[¶](#torch.utils.data.ChainDataset "此定义的永久链接")


 用于链接多个 [`IterableDataset`](#torch.utils.data.IterableDataset "torch.utils.data.IterableDataset") 的数据集。


 此类对于组装不同的现有数据集流非常有用。链接操作是即时完成的，因此将大规模数据集与此类连接起来会非常高效。


 Parameters


**datasets** ( *iterable
* *of
* [*IterableDataset*](#torch.utils.data.IterableDataset "torch.utils.data.IterableDataset") ) – 要链接在一起的数据集


*班级*


 火炬.utils.数据。


 子集


 ( *dataset
* , *indices
* ) [[source]](_modules/torch/utils/data/dataset.html#Subset)[¶](#torch.utils.data.Subset "此定义的永久链接")


 指定索引处的数据集的子集。


 参数 
* **dataset** ( [*Dataset*](#torch.utils.data.Dataset "torch.utils.data.Dataset") ) – 整个数据集
* **索引** ( *sequence
* ) – 索引在为子集选择的整个集合中


 torch.utils.data._utils.collat​​e。


 整理


 ( *batch
* , *** , *collat​​e_fn_map



 =
 


 无
* ) [[source]](_modules/torch/utils/data/_utils/collat​​e.html#collat​​e)[¶](#torch.utils.data._utils.collat​​e.collat​​e "此定义的永久链接")


 通用整理功能，处理每个批次内元素的集合类型，并打开函数注册表来处理特定元素类型。 default_collat​​e_fn_map 为tensor、numpy 数组、数字和字符串提供默认的整理函数。


 参数 
* **batch** – 要整理的单个批次
* **collat​​e_fn_map** ( [*可选*](https://docs.python.org/3/library/typing.html#typing.可选 "(Python v3.12 中)")*[
* [*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12 中) ")*[
* [*Union*](https://docs.python.org/3/library/typing.html#typing.Union "(在 Python v3.12 中)")*[
* [*Type*] (https://docs.python.org/3/library/typing.html#typing.Type“(在Python v3.12中)”)*,
* [*Tuple*](https://docs.python.org /3/library/typing.html#typing.Tuple "(Python v3.12)")*[
* [*Type*](https://docs.python.org/3/library/typing.html#typing.Type "(Python v3.12)")*,
* *...
* *]
* *]
* *,
* [*Callable*](https://docs.python.org/3/library/typing.html#typing.Callable "(in Python v3.12)")*]
* *]
* ) – 从元素类型到相应排序函数的可选字典映射。如果该字典中不存在该元素类型，则使用此函数如果元素类型是键的子类，将按插入顺序遍历字典的每个键以调用相应的整理函数。


 例子


```
>>> # Extend this function to handle batch of tensors
>>> def collate_tensor_fn(batch, *, collate_fn_map):
...     return torch.stack(batch, 0)
>>> def custom_collate(batch):
...     collate_map = {torch.Tensor: collate_tensor_fn}
...     return collate(batch, collate_fn_map=collate_map)
>>> # Extend `default_collate` by in-place modifying `default_collate_fn_map`
>>> default_collate_fn_map.update({torch.Tensor: collate_tensor_fn})

```


!!! note "笔记"

    每个整理函数都需要一个批处理的位置参数和一个整理函数字典的关键字参数，如 collat​​e_fn_map 。


 火炬.utils.数据。


 默认_整理


 ( *batch
* ) [[source]](_modules/torch/utils/data/_utils/collat​​e.html#default_collat​​e)[¶](#torch.utils.data.default_collat​​e "此定义的永久链接")


 函数接收一批数据并将批次内的元素放入具有附加外部维度(批次大小)的tensor中。确切的输出类型可以是 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") ，一个 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 的序列") ， [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 的集合，或保持不变，具体取决于输入类型。当批量_size时，这用作排序规则的默认函数或batch_sampler 在 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 中定义。


 以下是一般输入类型(基于批处理中元素的类型)到输出类型的映射：


>> 
> 
* [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor")
> ->> [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 
> (添加外部维度批量大小)
> 
* NumPy 数组 ->> [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor")
> 
* float
> 
> ->> [`torch. Tensor`](tensors.html#torch.Tensor "torch.Tensor")
> 
* int
> 
> ->> [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor")
> 
* str
> 
> ->> 
> str>> (不变)
> 
* 字节
> 
> ->> 
> 字节>> (不变)
> 
* 映射[K, V_i]
> 
> ->> 
> 映射[K, 默认_collat​​e( [V_1, V_2, …])]
> 
* NamedTuple[V1_i, V2_i, …]
> 
> ->> 
> NamedTuple[default_collat​​e([V1_1, V1_2, …] ),
> 默认_collat​​e([V2_1, V2_2, …]), …]
> 
* 序列[V1_i, V2_i, …]
> 
> ->> 
> 序列[默认_collat​​e([V1 _1, V1_2, …]),
> 默认_collat​​e([V2_1, V2_2, …]), …]
> 
> 
> >


 Parameters


**批次** – 要整理的单个批次


 例子


```
>>> # Example with a batch of `int`s:
>>> default_collate([0, 1, 2, 3])
tensor([0, 1, 2, 3])
>>> # Example with a batch of `str`s:
>>> default_collate(['a', 'b', 'c'])
['a', 'b', 'c']
>>> # Example with `Map` inside the batch:
>>> default_collate([{'A': 0, 'B': 1}, {'A': 100, 'B': 100}])
{'A': tensor([ 0, 100]), 'B': tensor([ 1, 100])}
>>> # Example with `NamedTuple` inside the batch:
>>> Point = namedtuple('Point', ['x', 'y'])
>>> default_collate([Point(0, 0), Point(1, 1)])
Point(x=tensor([0, 1]), y=tensor([0, 1]))
>>> # Example with `Tuple` inside the batch:
>>> default_collate([(0, 1), (2, 3)])
[tensor([0, 2]), tensor([1, 3])]
>>> # Example with `List` inside the batch:
>>> default_collate([[0, 1], [2, 3]])
[tensor([0, 2]), tensor([1, 3])]
>>> # Two options to extend `default_collate` to handle specific type
>>> # Option 1: Write custom collate function and invoke `default_collate`
>>> def custom_collate(batch):
...     elem = batch[0]
...     if isinstance(elem, CustomType):  # Some custom condition
...         return ...
...     else:  # Fall back to `default_collate`
...         return default_collate(batch)
>>> # Option 2: In-place modify `default_collate_fn_map`
>>> def collate_customtype_fn(batch, *, collate_fn_map=None):
...     return ...
>>> default_collate_fn_map.update(CustoType, collate_customtype_fn)
>>> default_collate(batch)  # Handle `CustomType` automatically

```


 火炬.utils.数据。


 默认_convert


 ( *data
* ) [[source]](_modules/torch/utils/data/_utils/collat​​e.html#default_convert)[¶](#torch.utils.data.default_convert "此定义的永久链接")


 将每个 NumPy 数组元素转换为 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 的函数。如果输入是 Sequence 、 Collection 或 Mapping ，它会尝试将内部的每个元素转换为 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 。如果输入不是 NumPy当 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.数据加载器”)。


 一般输入类型到输出类型的映射类似于 [`default_collat​​e()`](#torch.utils.data.default_collat​​e "torch.utils.data.default_collat​​e") 。请参阅那里的描述以了解更多详细信息。


 Parameters


**数据** – 要转换的单个数据点


 例子


```
>>> # Example with `int`
>>> default_convert(0)
0
>>> # Example with NumPy array
>>> default_convert(np.array([0, 1]))
tensor([0, 1])
>>> # Example with NamedTuple
>>> Point = namedtuple('Point', ['x', 'y'])
>>> default_convert(Point(0, 0))
Point(x=0, y=0)
>>> default_convert(Point(np.array(0), np.array(0)))
Point(x=tensor(0), y=tensor(0))
>>> # Example with List
>>> default_convert([np.array([0, 1]), np.array([2, 3])])
[tensor([0, 1]), tensor([2, 3])]

```


 火炬.utils.数据。


 获取_worker_info


 ( ) [[source]](_modules/torch/utils/data/_utils/worker.html#get_worker_info)[¶](#torch.utils.data.get_worker_info "此定义的永久链接")


 返回有关当前 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 迭代器工作进程的信息。


 当在工作线程中调用时，这会返回一个保证具有以下属性的对象：



* `id` ：当前worker id。
* `num_workers` ：worker 总数。
* `seed` ：当前worker 的随机种子集。该值由主进程RNG和worker id确定。有关更多详细信息，请参阅 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的文档。
* `dataset` ：**此**进程中数据集对象的副本。请注意，这将是与主进程中的对象不同的进程中的不同对象。


 当在主进程中调用时，返回 None 。




!!! note "笔记"

    当在传递给 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的 `worker_init_fn` 中使用时，此方法可用于以不同方式设置每个工作进程例如，使用“worker_id”将“dataset”对象配置为仅读取分片数据集的特定部分，或使用“seed”为数据集代码中使用的其他库提供种子。


 Return type


[*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中)") [ *WorkerInfo
* ]


 火炬.utils.数据。


 随机_分割


 ( *dataset
* , *lengths
* , *generator=<torch._C.Generator 对象>
* ) [[source]](_modules/torch/utils/data/dataset.html#random_split)[¶](#torch. utils.data.random_split"此定义的永久链接")


 将数据集随机拆分为给定长度的不重叠的新数据集。


 如果给出总和为 1 的分数列表，则对于提供的每个分数，长度将自动计算为floor(frac * len(dataset))。


 计算完长度后，如果有余数，则将 1 个计数以循环方式分配给长度，直到没有余数为止。


 可以选择修复生成器以获得可重现的结果，例如：


 例子


```
>>> generator1 = torch.Generator().manual_seed(42)
>>> generator2 = torch.Generator().manual_seed(42)
>>> random_split(range(10), [3, 7], generator=generator1)
>>> random_split(range(30), [0.3, 0.3, 0.4], generator=generator2)

```


 参数 
* **dataset** ( [*Dataset*](#torch.utils.data.Dataset "torch.utils.data.Dataset") ) – 要分割的数据集
* **长度** ( *sequence
* ) –要生成的分割的长度或分数
* **generator** ( [*Generator*](generated/torch.Generator.html#torch.Generator "torch.Generator") ) – 用于随机排列的生成器。


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*Subset*](#torch.utils.data.子集“torch.utils.data.dataset.Subset”)[*T*]]


*班级*


 火炬.utils.数据。


 采样器


 ( *数据源



 =
 


 无
* ) [[source]](_modules/torch/utils/data/sampler.html#Sampler)[¶](#torch.utils.data.Sampler "此定义的永久链接")


 所有采样器的基类。


 每个 Sampler 子类都必须提供一个 `__iter__()` 方法，用于迭代数据集元素的索引或索引列表(批次)，以及一个 `__len__( )` 方法返回返回的迭代器的长度。


 Parameters


**data_source** ( [*Dataset*](#torch.utils.data.Dataset "torch.utils.data.Dataset") ) – 该参数未使用，将在 2.2.0 中删除。您可以仍然有使用它的自定义实现。


 例子


```
>>> class AccedingSequenceLengthSampler(Sampler[int]):
>>>     def __init__(self, data: List[str]) -> None:
>>>         self.data = data
>>>
>>>     def __len__(self) -> int:
>>>         return len(self.data)
>>>
>>>     def __iter__(self) -> Iterator[int]:
>>>         sizes = torch.tensor([len(x) for x in self.data])
>>>         yield from torch.argsort(sizes).tolist()
>>>
>>> class AccedingSequenceLengthBatchSampler(Sampler[List[int]]):
>>>     def __init__(self, data: List[str], batch_size: int) -> None:
>>>         self.data = data
>>>         self.batch_size = batch_size
>>>
>>>     def __len__(self) -> int:
>>>         return (len(self.data) + self.batch_size - 1) // self.batch_size
>>>
>>>     def __iter__(self) -> Iterator[List[int]]:
>>>         sizes = torch.tensor([len(x) for x in self.data])
>>>         for batch in torch.chunk(torch.argsort(sizes), len(self)):
>>>             yield batch.tolist()

```


!!! note "笔记"

    [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 并不严格要求 `__len__()` 方法，但在任何计算中都需要该方法涉及 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 的长度。


*班级*


 火炬.utils.数据。


 顺序采样器


 ( *data_source
* ) [[source]](_modules/torch/utils/data/sampler.html#SequentialSampler)[¶](#torch.utils.data.SequentialSampler "此定义的永久链接")


 按顺序对元素进行采样，且顺序始终相同。


 Parameters


**data_source** ( [*Dataset*](#torch.utils.data.Dataset "torch.utils.data.Dataset") ) – 从中采样的数据集


*班级*


 火炬.utils.数据。


 随机采样器


 ( *data_source
* , *替换



 =
 


 假
* , *num_samples



 =
 


 无
* , *生成器



 =
 


 无
* ) [[source]](_modules/torch/utils/data/sampler.html#RandomSampler)[¶](#torch.utils.data.RandomSampler "此定义的永久链接")


 随机采样元素。如果没有替换，则从打乱的数据集中采样。如果有替换，则用户可以指定“num_samples”进行绘制。


 参数 
* **data_source** ( [*Dataset*](#torch.utils.data.Dataset "torch.utils.data.Dataset") ) – 样本数据集
* **替换** ( [*bool *](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` ，default=` 则按需抽取样本并进行替换`False``
* **num_samples** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) –要绘制的样本数量，默认=`len(dataset)`。
* **generator** ( [*Generator*](generated/torch.Generator.html#torch.Generator "torch.Generator") ) – 使用的生成器采样。


*班级*


 火炬.utils.数据。


 子集随机采样器


 ( *索引
* , *生成器



 =
 


 无
* ) [[source]](_modules/torch/utils/data/sampler.html#SubsetRandomSampler)[¶](#torch.utils.data.SubsetRandomSampler "此定义的永久链接")


 从给定的索引列表中随机采样元素，而不进行替换。


 参数 
* **indices** ( *sequence
* ) – 索引序列
* **generator** ( [*Generator*](generated/torch.Generator.html#torch.Generator "torch.Generator") ) – 生成器用于采样。


*班级*


 火炬.utils.数据。


 加权随机采样器


 ( *权重
* , *num_samples
* , *替换



 =
 


 真*，*生成器



 =
 


 无
* ) [[source]](_modules/torch/utils/data/sampler.html#WeightedRandomSampler)[¶](#torch.utils.data.WeightedRandomSampler "此定义的永久链接")


 以给定的概率(权重)从“[0,..,len(weights)-1]”中采样元素。


 参数 
* **weights** ( *sequence
* ) – 权重序列，不一定求和为一个
* **num_samples** ( [*int*](https://docs.python.org/3) /library/functions.html#int "(in Python v3.12)") ) – 要绘制的样本数量
* **替换** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` ，则以替换方式绘制样本。如果不是，则以不替换方式绘制样本，这意味着当为一行绘制样本索引时，无法再次绘制该行。
* **generator** ( [*Generator*](generated/torch.Generator.html#torch.Generator "torch.Generator") ) – 采样中使用的生成器。


 例子


```
>>> list(WeightedRandomSampler([0.1, 0.9, 0.4, 0.7, 3.0, 0.6], 5, replacement=True))
[4, 4, 1, 4, 5]
>>> list(WeightedRandomSampler([0.9, 0.4, 0.05, 0.2, 0.3, 0.1], 5, replacement=False))
[0, 1, 4, 3, 2]

```


*班级*


 火炬.utils.数据。


 批量采样器


 ( *sampler
* , *batch_size
* , *drop_last
* ) [[source]](_modules/torch/utils/data/sampler.html#BatchSampler)[¶](#torch.utils.data.BatchSampler "此定义的永久链接")


 包装另一个采样器以生成小批量索引。


 参数 
* **sampler** ( [*Sampler*](#torch.utils.data.Sampler "torch.utils.data.Sampler")*或
* *Iterable
* ) – 基础采样器。可以是任何可迭代对象
* **batch_size** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 小批量的大小。
* **drop_last** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)" ) ) – 如果 `True` ，则采样器将删除最后一个批次，如果其大小小于 `batch_size`


 例子


```
>>> list(BatchSampler(SequentialSampler(range(10)), batch_size=3, drop_last=False))
[[0, 1, 2], [3, 4, 5], [6, 7, 8], [9]]
>>> list(BatchSampler(SequentialSampler(range(10)), batch_size=3, drop_last=True))
[[0, 1, 2], [3, 4, 5], [6, 7, 8]]

```


*班级*


 torch.utils.data.distributed。


 分布式采样器


 ( *数据集
* , *num_replicas



 =
 


 无
* , *等级



 =
 


 无
* , *随机播放



 =
 


 真*，*种子



 =
 


 0
* , *删除_last



 =
 


 False
* ) [[source]](_modules/torch/utils/data/distributed.html#DistributedSampler)[¶](#torch.utils.data.distributed.DistributedSampler "此定义的永久链接")


 将数据加载限制为数据集子集的采样器。


 它与 [`torch.nn.parallel.DistributedDataParallel`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel") 结合使用特别有用。在这种情况下，每个进程都可以将“DistributedSampler”实例作为 [`DataLoader`](#torch.utils.data.DataLoader "torch.utils.data.DataLoader") 采样器传递，并加载原始数据集的子集专属于它。




!!! note "笔记"

    假设数据集具有恒定大小，并且它的任何实例总是以相同的顺序返回相同的元素。


 参数 
* **dataset** – 用于采样的数据集。
* **num_replicas** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*,
* *可选
* ) – 参与分布式训练的进程数。默认情况下，`world_size` 是从当前分布式组中检索的。
* **rank** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*,
* *可选
* ) – `num_replicas` 中当前进程的排名。默认情况下，从当前分布式组中检索`rank`。
* **shuffle** ( [*bool *](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `True` (默认)，采样器将洗牌索引。
* **种子** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*,
* *可选
* ) – 如果 `shuffle=True` 则用于对采样器进行洗牌的随机种子。该数字在分布式组中的所有进程中应该是相同的。默认值：`0`.
* **drop_last** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") *,
* *可选
* ) – 如果 `True` ，则采样器将删除数据的尾部，使其可在副本数量上均匀划分。如果为 False ，采样器将添加额外的索引以使数据在副本之间均匀划分。默认值：`False`。


!!! warning "警告"

     在分布式模式下，为了使洗牌在多个纪元之间正​​常工作，需要在每个纪元创建“DataLoader”迭代器之前**在每个纪元开始时调用“set_epoch()”方法。否则，将始终使用相同的顺序。


 例子：


```
>>> sampler = DistributedSampler(dataset) if is_distributed else None
>>> loader = DataLoader(dataset, shuffle=(sampler is None),
...                     sampler=sampler)
>>> for epoch in range(start_epoch, n_epochs):
...     if is_distributed:
...         sampler.set_epoch(epoch)
...     train(loader)

```