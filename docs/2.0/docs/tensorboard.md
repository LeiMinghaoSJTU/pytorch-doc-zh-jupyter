# torch.utils.tensorboard [¶](#module-torch.utils.tensorboard "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/tensorboard>
>
> 原始地址：<https://pytorch.org/docs/stable/tensorboard.html>


 在进一步讨论之前，有关 TensorBoard 的更多详细信息可以在 <https://www.tensorflow.org/tensorboard/
> 找到


 安装 TensorBoard 后，这些实用程序可让您将 PyTorch 模型和指标记录到 TensorBoard UI 中的可视化目录中。PyTorch 模型和tensor以及 Caffe2 网络和 blob 均支持标量、图像、直方图、图表和嵌入可视化。


 SummaryWriter 类是记录数据以供 TensorBoard 使用和可视化的主要入口。例如：


```
import torch
import torchvision
from torch.utils.tensorboard import SummaryWriter
from torchvision import datasets, transforms

# Writer will output to ./runs/ directory by default
writer = SummaryWriter()

transform = transforms.Compose([transforms.ToTensor(), transforms.Normalize((0.5,), (0.5,))])
trainset = datasets.MNIST('mnist_train', train=True, download=True, transform=transform)
trainloader = torch.utils.data.DataLoader(trainset, batch_size=64, shuffle=True)
model = torchvision.models.resnet50(False)
# Have ResNet model take in grayscale rather than RGB
model.conv1 = torch.nn.Conv2d(1, 64, kernel_size=7, stride=2, padding=3, bias=False)
images, labels = next(iter(trainloader))

grid = torchvision.utils.make_grid(images)
writer.add_image('images', grid, 0)
writer.add_graph(model, images)
writer.close()

```


 然后可以使用 TensorBoard 进行可视化，它应该可以通过以下方式安装和运行：


```
pip install tensorboard
tensorboard --logdir=runs

```


 一项实验可以记录大量信息。为了避免 UI 混乱并获得更好的结果聚类，我们可以通过分层命名来对图进行分组。例如，在 TensorBoard 界面中，“Loss/train”和“Loss/test”将被分组在一起，而“Accuracy/train”和“Accuracy/test”将被单独分组。


```
from torch.utils.tensorboard import SummaryWriter
import numpy as np

writer = SummaryWriter()

for n_iter in range(100):
    writer.add_scalar('Loss/train', np.random.random(), n_iter)
    writer.add_scalar('Loss/test', np.random.random(), n_iter)
    writer.add_scalar('Accuracy/train', np.random.random(), n_iter)
    writer.add_scalar('Accuracy/test', np.random.random(), n_iter)

```


 预期结果：


[![_images/hier_tags.png](_images/hier_tags.png)](_images/hier_tags.png)


*班级*


 torch.utils.tensorboard.writer。


 摘要作者


 ( *log_dir



 =
 


 无
* , *评论



 =
 


 ''
* , *清除_step



 =
 


 无
* , *max_queue



 =
 


 10
* , *刷新_秒



 =
 


 120
* , *文件名_后缀



 =
 


 ''
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter)[¶](#torch.utils.tensorboard.writer.SummaryWriter "此定义的永久链接")


 将条目直接写入 log_dir 中的事件文件以供 TensorBoard 使用。


 SummaryWriter 类提供了一个高级 API，用于在给定目录中创建事件文件并向其中添加摘要和事件。该类异步更新文件内容。这允许训练程序调用方法直接从训练循环将数据添加到文件中，而不会减慢训练速度。


 __在里面__


 ( *log_dir



 =
 


 无
* , *评论



 =
 


 ''
* , *清除_step



 =
 


 无
* , *max_queue



 =
 


 10
* , *刷新_秒



 =
 


 120
* , *文件名_后缀



 =
 


 ''
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.__init__)[¶](#torch.utils.tensorboard.writer.SummaryWriter.__init__ "此定义的永久链接")


 创建一个 SummaryWriter，将事件和摘要写入事件文件。


 参数 
* **log_dir** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 保存目录位置。默认为runs/**CURRENT_DATETIME_HOSTNAME** ，每次运行后都会更改。使用分层文件夹结构可以轻松比较运行之间的情况。例如传入 'runs/exp1'、'runs/exp2' 等，以便每个新实验进行比较。
* **comment** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 注释 log_dir 后缀附加到默认的 `log_dir` 。如果指定了 `log_dir`，则此参数无效。
* **purge_step** ( [*int*](https://docs.python.org/3/library/functions.html#int " (在 Python v3.12 中)") ) – 当日志记录在步骤中崩溃时


 T+X


 T+X
 


 T
 



 +
 




 X
 


 并在步骤重新启动



 T
 


 T
 


 T
 


 ，任何全局_step大于或等于的事件



 T
 


 T
 


 T
 


 将从 TensorBoard 中清除并隐藏。请注意，崩溃和恢复的实验应具有相同的 `log_dir` 。
* **max_queue** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 在“add”调用之一强制刷新到磁盘之前待处理事件和摘要的队列大小。默认为 10 项。
* **刷新_secs** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 刷新的频率(以秒为单位)将待处理事件和摘要保存到磁盘。默认为每两分钟一次。
* **文件名_后缀** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 添加到 log_dir 目录中的所有事件文件名的后缀。有关文件名构造 intensorboard.summary.writer.event_file_writer.EventFileWriter 的更多详细信息。


 例子：


```
from torch.utils.tensorboard import SummaryWriter

# create a summary writer with automatically generated folder name.
writer = SummaryWriter()
# folder location: runs/May04_22-14-54_s-MacBook-Pro.local/

# create a summary writer using the specified folder name.
writer = SummaryWriter("my_experiment")
# folder location: my_experiment

# create a summary writer with comment appended.
writer = SummaryWriter(comment="LR_0.1_BATCH_16")
# folder location: runs/May04_22-14-54_s-MacBook-Pro.localLR_0.1_BATCH_16/

```


 添加_标量


 ( *标签
* , *标量_值
* , *全局_步长



 =
 


 无
* , *墙上时间



 =
 


 无
* , *新_风格



 =
 


 假
* , *双精度



 =
 


 False
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_scalar)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_scalar "此定义的永久链接")


 将标量数据添加到摘要中。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* **标量_value** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)")*或
* *string/blobname
* ) – 要保存的值
* **global_step** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) –要记录的全局步值
* **walltime** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 可选用事件纪元后的秒数覆盖默认 walltime (time.time())
* **new_style** ( *boolean
* ) – 是否使用新样式(tensor字段)或旧样式(简单_value 字段)。新样式可以加快数据加载速度。


 例子：


```
from torch.utils.tensorboard import SummaryWriter
writer = SummaryWriter()
x = range(100)
for i in x:
    writer.add_scalar('y=2x', i * 2, i)
writer.close()

```


 预期结果：


[![_images/add_scalar.png](_images/add_scalar.png)](_images/add_scalar.png)


 添加_标量


 ( *main_tag
* , *tag_scalar_dict
* , *global_step



 =
 


 无
* , *墙上时间



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_scalars)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_scalars "此定义的永久链接")


 添加许多标量数据到摘要中。


 参数 
* **main_tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 父名称对于标签
* **tag_scalar_dict** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 存储标签和相应值的键值对
* **global_step** ( [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3 中).12)") ) – 要记录的全局步值
* **walltime** ( [*float*](https://docs.python.org/3/library/functions.html#float "(在 Python v3.1 中) 12)") ) – 可选覆盖事件纪元后的默认 walltime (time.time()) 秒


 例子：


```
from torch.utils.tensorboard import SummaryWriter
writer = SummaryWriter()
r = 5
for i in range(100):
    writer.add_scalars('run_14h', {'xsinx':i*np.sin(i/r),
                                    'xcosx':i*np.cos(i/r),
                                    'tanx': np.tan(i/r)}, i)
writer.close()
# This call adds three values to the same scalar plot with the tag
# 'run_14h' in TensorBoard's scalar section.

```


 预期结果：


[![_images/add_scalars.png](_images/add_scalars.png)](_images/add_scalars.png)


 添加_直方图


 ( *标签
* , *值
* , *全局_step



 =
 


 无*，*垃圾箱



 =
 


 'tensor流'*，*walltime



 =
 


 无
* , *max_bins



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_histogram)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_histogram "此定义的永久链接")


 将直方图添加到摘要中。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* **值** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* [*numpy.ndarray*](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray "(in NumPy v1.26)")*, 或
* *string/blobname
* ) – 构建直方图的值
* **global_step** ( [*int*] (https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 记录的全局步值
* **bins** ( [*str*]( https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)") ) – {'tensorflow','auto', 'fd', …} 之一。这决定了垃圾箱的制作方式。您可以在以下位置找到其他选项： <https://docs.scipy.org/doc/numpy/reference/generated/numpy.histogram.html>
* **walltime** ( [*float*](https://docs. python.org/3/library/functions.html#float "(in Python v3.12)") ) – 可选覆盖事件纪元后的默认 walltime (time.time()) 秒


 例子：


```
from torch.utils.tensorboard import SummaryWriter
import numpy as np
writer = SummaryWriter()
for i in range(10):
    x = np.random.random(1000)
    writer.add_histogram('distribution centers', x + i, i)
writer.close()

```


 预期结果：


[![_images/add_histogram.png](_images/add_histogram.png)](_images/add_histogram.png)


 添加图片


 ( *tag
* , *img_tensor
* , *global_step



 =
 


 无
* , *墙上时间



 =
 


 无
* , *数据格式



 =
 


 'CHW'
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_image)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_image "此定义的永久链接")


 将图像数据添加到摘要中。


 请注意，这需要“pillow”包。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* ** img_tensor** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* [*numpy.ndarray*](https://numpy.org/doc/stable/参考/generated/numpy.ndarray.html#numpy.ndarray“(在NumPy v1.26中)”)*，或
* *string/blobname*) – 图像数据
* **global_step** ( [*int*] (https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 记录的全局步值
* **walltime** ( [*float*]( https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 可选覆盖事件纪元后的默认walltime (time.time())秒
* ** dataformats** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 形式CHW、HWC的图像数据格式规范、HW、WH 等


 形状：


 img_tensor：默认为


 (3、高、宽)


 (3、高、宽)


 ( 3 ,



 H
 

 ,
 



 W
 

 )
 


 。您可以使用 torchvision.utils.make_grid()` 将一批tensor转换为 3xHxW 格式或调用 `add_images` 让我们完成这项工作。Tensor 与


 ( 1 , 高 , 宽 )


 (1、高、宽)


 ( 1 ,



 H
 

 ,
 



 W
 

 )
 




 ,
 


 (高、宽)


 (高、宽)


 (  H  ，



 W
 

 )
 




 ,
 


 ( 高 , 宽 , 3 )


 (高、宽、3)


 (  H  ，



 W
 

 ,
 



 3
 

 )
 


 只要传递相应的“dataformats”参数也适用，例如“CHW”、“HWC”、“HW”。


 例子：


```
from torch.utils.tensorboard import SummaryWriter
import numpy as np
img = np.zeros((3, 100, 100))
img[0] = np.arange(0, 10000).reshape(100, 100) / 10000
img[1] = 1 - np.arange(0, 10000).reshape(100, 100) / 10000

img_HWC = np.zeros((100, 100, 3))
img_HWC[:, :, 0] = np.arange(0, 10000).reshape(100, 100) / 10000
img_HWC[:, :, 1] = 1 - np.arange(0, 10000).reshape(100, 100) / 10000

writer = SummaryWriter()
writer.add_image('my_image', img, 0)

# If you have non-default dimension setting, set the dataformats argument.
writer.add_image('my_image_HWC', img_HWC, 0, dataformats='HWC')
writer.close()

```


 预期结果：


[![_images/add_image.png](_images/add_image.png)](_images/add_image.png)


 添加_图像


 ( *tag
* , *img_tensor
* , *global_step



 =
 


 无
* , *墙上时间



 =
 


 无
* , *数据格式



 =
 


 'NCHW'
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_images)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_images "此定义的永久链接")


 将批量图像数据添加到摘要中。


 请注意，这需要“pillow”包。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* ** img_tensor** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* [*numpy.ndarray*](https://numpy.org/doc/stable/参考/generated/numpy.ndarray.html#numpy.ndarray“(在NumPy v1.26中)”)*，或
* *string/blobname*) – 图像数据
* **global_step** ( [*int*] (https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 记录的全局步值
* **walltime** ( [*float*]( https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 可选覆盖事件纪元后的默认walltime (time.time())秒
* ** dataformats** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – NCHW、NHWC 形式的图像数据格式规范、CHW、HWC、HW、WH 等


 形状：


 img_tensor：默认为


 ( 北 , 3 , 高 , 宽 )


 (北、3、高、宽)


 (N,



 3
 

 ,
 



 H
 

 ,
 



 W
 

 )
 


 。如果指定了“dataformats”，则将接受其他形状。例如NCHW 或 NHWC。


 例子：


```
from torch.utils.tensorboard import SummaryWriter
import numpy as np

img_batch = np.zeros((16, 3, 100, 100))
for i in range(16):
    img_batch[i, 0] = np.arange(0, 10000).reshape(100, 100) / 10000 / 16 * i
    img_batch[i, 1] = (1 - np.arange(0, 10000).reshape(100, 100) / 10000) / 16 * i

writer = SummaryWriter()
writer.add_images('my_image_batch', img_batch, 0)
writer.close()

```


 预期结果：


[![_images/add_images.png](_images/add_images.png)](_images/add_images.png)


 添加_图


 ( *标签
* , *图
* , *全局_step



 =
 


 无
* , *关闭



 =
 


 真实*，*墙上时间



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_figure)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_figure"此定义的永久链接")


 将 matplotlib 图渲染为图像并将其添加到摘要中。


 请注意，这需要“matplotlib”包。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* **图** ( *matplotlib.pyplot.figure
* ) – 图形或图形列表
* **global_step** ( [*int*](https://docs.python.org/3/library/functions. html#int "(in Python v3.12)") ) – 要记录的全局步值
* **close** ( [*bool*](https://docs.python.org/3/library/functions.html #bool "(Python v3.12)") ) – 自动关闭图窗的标志
* **walltime** ( [*float*](https://docs.python.org/3/library/functions.html #float "(in Python v3.12)") ) – 可选覆盖事件纪元后的默认 walltime (time.time()) 秒


 添加_视频


 ( *tag
* , *vid_tensor
* , *global_step



 =
 


 无
* , *fps



 =
 


 4
* , *墙上时间



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_video)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_video "此定义的永久链接")


 将视频数据添加到摘要中。


 请注意，这需要“moviepy”包。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* ** vid_tensor** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 视频数据
* **global_step** ( [*int*](https://docs.python.org/3/library/functions.html#int "(Python v3.12)") ) – 记录的全局步值
* **fps** ( [*float*](https://docs.python.org/3/library/functions.html#float "(Python v3.12)")*或
* [*int*](https://docs.python.org/3/library/functions.html #int "(Python v3.12)") ) – 每秒帧数
* **walltime** ( [*float*](https://docs.python.org/3/library/functions.html#float " (在 Python v3.12 中)") ) – 可选覆盖事件纪元后的默认 walltime (time.time()) 秒


 形状：


 在_tensor：


 (北、中、高、宽)


 (北、东、中、高、西)


 (N,



 T
 

 ,
 



 C
 

 ,
 



 H
 

 ,
 



 W
 

 )
 


 。对于 uint8 类型，值应位于 [0, 255] 中；对于 float 类型，值应位于 [0, 1] 中。


 添加_音频


 ( *tag
* , *snd_tensor
* , *global_step



 =
 


 无
* , *样本_率



 =
 


 44100
* , *挂墙时间



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_audio)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_audio "此定义的永久链接")


 将音频数据添加到摘要中。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* ** snd_tensor** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 声音数据
* **global_step** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 要记录的全局步值
* **sample_rate** ( [*int*](https:///docs.python.org/3/library/functions.html#int "(Python v3.12)") ) – 采样率，单位 Hz
* **walltime** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 可选覆盖事件纪元后的默认 walltime (time.time()) 秒


 形状：


 snd_tensor：


 ( 1 , 左 )


 (1，L)


 ( 1 ,



 L
 

 )
 


 。这些值应介于 [-1, 1] 之间。


 添加文字


 ( *tag
* , *text_string
* , *global_step



 =
 


 无
* , *墙上时间



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_text)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_text "此定义的永久链接")


 将文本数据添加到摘要中。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* ** text_string** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要保存的字符串
* **全局_step** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 要记录的全局步值
* ** walltime** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 可选覆盖默认 walltime (time.time( )) 事件纪元后的秒数


 例子：


```
writer.add_text('lstm', 'This is an lstm', 0)
writer.add_text('rnn', 'This is an rnn', 10)

```


 添加_graph


 ( *模型
* , *输入_模型_模型



 =
 


 无*，*详细



 =
 


 假
* , *使用_strict_trace



 =
 


 True
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_graph)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_graph "此定义的永久链接")


 将图表数据添加到摘要中。


 参数 
* **model** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 要绘制的模型。
* **输入_to_model** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor")*或
* [*list*](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)")*of
* [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 一个变量或变量元组be fed.
* **verbose** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否打印控制台中的图形结构。
* **use_strict_trace** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)" ) ) – 是否将关键字参数 strict 传递给 torch.jit.trace 。当您希望跟踪器记录可变容器类型(列表、字典)时，传递 False


 添加_嵌入


 ( *mat
* , *元数据



 =
 


 无
* , *标签_img



 =
 


 无
* , *全局_step



 =
 


 无
* , *标签



 =
 


 '默认'
* , *元数据_header



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_embedding)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_embedding "此定义的永久链接")


 将嵌入投影仪数据添加到摘要中。


 参数 
* **mat** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor")*或
* [*numpy.ndarray*](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray "(in NumPy v1.26)") ) – 一个矩阵，每行都是数据点的特征向量
* **元数据** ( [*list *](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)") ) – 标签列表，每个元素将转换为字符串
* **标签_img** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 图像对应于每个数据点
* **global_step** ( [*int*](https ://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 要记录的全局步值
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 嵌入的名称


 形状：


 mat:
 


 (N,D)


 (N、D)


 (N,



 D
 

 )
 


 ，其中 N 是数据数量，D 是特征维度


 标签_img:


 (北、中、高、宽)


 (北、中、高、西)


 (N,



 C
 

 ,
 



 H
 

 ,
 



 W
 

 )
 


 例子：


```
import keyword
import torch
meta = []
while len(meta)<100:
    meta = meta+keyword.kwlist # get some strings
meta = meta[:100]

for i, v in enumerate(meta):
    meta[i] = v+str(i)

label_img = torch.rand(100, 3, 10, 32)
for i in range(100):
    label_img[i]*=i/100.0

writer.add_embedding(torch.randn(100, 5), metadata=meta, label_img=label_img)
writer.add_embedding(torch.randn(100, 5), label_img=label_img)
writer.add_embedding(torch.randn(100, 5), metadata=meta)

```


 添加_pr_曲线


 ( *标签
* , *标签
* , *预测
* , *全局_step



 =
 


 无
* , *num_thresholds



 =
 


 127
* , *权重



 =
 


 无
* , *墙上时间



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_pr_curve)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_pr_curve "此定义的永久链接")


 添加精确召回曲线。绘制精确召回曲线可以让您了解模型在不同阈值设置下的性能。通过此函数，您可以为每个目标提供真实标签 (T/F) 和预测置信度(通常是模型的输出)。 TensorBoard UI 将让您以交互方式选择阈值。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* **标签** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* [*numpy.ndarray*](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray "(in NumPy v1.26)")*, 或
* *string/blobname
* ) – 真实数据。每个元素的二进制标签。
* **预测** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* [*numpy.ndarray*](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray "(in NumPy v1.26)")*, or
* *string/blobname
* ) – 元素被分类为 true 的概率.值应位于 [0, 1]
* **global_step** ( [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3.1 中) 12)") ) – 要记录的全局步值
* **num_thresholds** ( [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3 中).12)") ) – 用于绘制曲线的阈值数量。
* **walltime** ( [*float*](https://docs.python.org/3/library/functions.html#float "(在 Python v3.12)") ) – 可选覆盖事件纪元后的默认 walltime (time.time()) 秒


 例子：


```
from torch.utils.tensorboard import SummaryWriter
import numpy as np
labels = np.random.randint(2, size=100)  # binary label
predictions = np.random.rand(100)
writer = SummaryWriter()
writer.add_pr_curve('pr_curve', labels, predictions, 0)
writer.close()

```


 添加_自定义_标量


 ( *layout
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_custom_scalars)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_custom_scalars "此定义的永久链接")


 通过收集“标量”中的图表标签来创建特殊图表。请注意，对于每个 SummaryWriter() 对象只能调用该函数一次。因为它只向tensor板提供元数据，所以可以在训练循环之前或之后调用该函数。


 Parameters


**布局** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – {categoryName: *charts
* } ，其中 *charts
* 也是一个字典{chartName: *ListOfProperties
* }。 *ListOfProperties
* 中的第一个元素是图表的类型( **Multiline** 或 **Margin** 之一)，第二个元素应该是包含您在 add_scalar 函数中使用的标签的列表，这些标签将被收集到新图表。


 例子：


```
layout = {'Taiwan':{'twse':['Multiline',['twse/0050', 'twse/2330']]},
             'USA':{ 'dow':['Margin',   ['dow/aaa', 'dow/bbb', 'dow/ccc']],
                  'nasdaq':['Margin',   ['nasdaq/aaa', 'nasdaq/bbb', 'nasdaq/ccc']]}}

writer.add_custom_scalars(layout)

```


 添加_网格


 ( *标签
* , *顶点
* , *颜色



 =
 


 无*，*面孔



 =
 


 无
* , *config_dict



 =
 


 无
* , *全局_step



 =
 


 无
* , *墙上时间



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_mesh)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_mesh "此定义的永久链接")


 将网格或 3D 点云添加到 TensorBoard。可视化基于Three.js，因此它允许用户与渲染的对象进行交互。除了顶点、面等基本定义外，用户还可以进一步提供相机参数、光照条件等。请查看<https://thirdjs.org/docs/index.html#manual/en/introduction/Creating-a-scene>供高级使用。


 参数 
* **tag** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 数据标识符
* ** vertices** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 顶点的 3D 坐标列表。
* **颜色** ( [*torch.Tensor*]( tensors.html#torch.Tensor "torch.Tensor") ) – 每个顶点的颜色
* **面** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 索引每个三角形内的顶点数。 (可选)
* **config_dict** – 具有 ThreeJS 类名称和配置的字典。
* **global_step** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 要记录的全局步值
* **walltime** ( [*float*](https://docs.python.org/3/library/functions. html#float "(in Python v3.12)") ) – 可选覆盖事件纪元后的默认 walltime (time.time()) 秒


 形状：


 顶点：


 (B,N,3)


 (B、N、3)


 ( 乙 ,



 N
 

 ,
 



 3
 

 )
 


 。 (批次、顶点数、通道)


 颜色：


 (B,N,3)


 (B、N、3)


 ( 乙 ,



 N
 

 ,
 



 3
 

 )
 


 。对于 uint8 类型，值应位于 [0, 255] 中；对于 float 类型，值应位于 [0, 1] 中。


 面孔：


 (B,N,3)


 (B、N、3)


 ( 乙 ,



 N
 

 ,
 



 3
 

 )
 


 。对于 uint8 类型，值应位于 [0, number_of_vertices] 中。


 例子：


```
from torch.utils.tensorboard import SummaryWriter
vertices_tensor = torch.as_tensor([
    [1, 1, 1],
    [-1, -1, 1],
    [1, -1, -1],
    [-1, 1, -1],
], dtype=torch.float).unsqueeze(0)
colors_tensor = torch.as_tensor([
    [255, 0, 0],
    [0, 255, 0],
    [0, 0, 255],
    [255, 0, 255],
], dtype=torch.int).unsqueeze(0)
faces_tensor = torch.as_tensor([
    [0, 2, 3],
    [0, 3, 1],
    [0, 1, 2],
    [1, 3, 2],
], dtype=torch.int).unsqueeze(0)

writer = SummaryWriter()
writer.add_mesh('my_mesh', vertices=vertices_tensor, colors=colors_tensor, faces=faces_tensor)

writer.close()

```


 添加_hparams


 ( *hparam_dict
* 、 *metric_dict
* 、 *hparam_domain_discrete



 =
 


 无
* , *运行_name



 =
 


 无
* ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.add_hparams)[¶](#torch.utils.tensorboard.writer.SummaryWriter.add_hparams "此定义的永久链接")


 添加一组要在 TensorBoard 中进行比较的超参数。


 参数 
* **hparam_dict** ( [*dict*](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 每个键 -字典中的值对是超参数的名称及其对应的值。值的类型可以是 bool 、 string 、 float 、 int 或 None 之一。
* **metric_dict** ( [*dict*] (https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.12)") ) – 字典中的每个键值对都是指标的名称及其对应的值。请注意，此处使用的密钥在tensor板记录中应该是唯一的。否则，您通过`add_scalar`添加的值将显示在hparam插件中。在大多数情况下，这是不需要的。
* **hparam_domain_discrete** – (Optional[Dict[str, List[Any]]]) 包含超参数名称及其可以保存的所有离散值的字典
* **run _name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要包含的运行名称作为 logdir 的一部分。如果未指定，将使用当前时间戳。


 例子：


```
from torch.utils.tensorboard import SummaryWriter
with SummaryWriter() as w:
    for i in range(5):
        w.add_hparams({'lr': 0.1*i, 'bsize': i},
                      {'hparam/accuracy': 10*i, 'hparam/loss': 10*i})

```


 预期结果：


[![_images/add_hparam.png](_images/add_hparam.png)](_images/add_hparam.png)



 flush
 


 ( ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.flush)[¶](#torch.utils.tensorboard.writer.SummaryWriter.flush "此定义的永久链接")


 将事件文件刷新到磁盘。调用此方法可确保所有待处理事件已写入磁盘。


 close
 


 ( ) [[source]](_modules/torch/utils/tensorboard/writer.html#SummaryWriter.close)[¶](#torch.utils.tensorboard.writer.SummaryWriter.close "此定义的永久链接")