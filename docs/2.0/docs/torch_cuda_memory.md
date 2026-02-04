# 了解 CUDA 内存使用情况 [¶](#understanding-cuda-memory-usage "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/torch_cuda_memory>
>
> 原始地址：<https://pytorch.org/docs/stable/torch_cuda_memory.html>


 为了调试 CUDA 内存使用情况，PyTorch 提供了一种生成内存快照的方法，该快照可记录任意时间点已分配 CUDA 内存的状态，并可选择记录导致该快照的分配事件的历史记录。


 然后可以将生成的快照拖放到 [pytorch.org/memory_viz](https://pytorch.org/memory_viz) 上托管的交互式查看器上，该查看器可用于探索快照。


# 生成快照 [¶](#generate-a-snapshot "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/torch_cuda_memory>
>
> 原始地址：<https://pytorch.org/docs/stable/torch_cuda_memory.html>


 记录快照的常见模式是启用内存历史记录，运行要观察的代码，然后使用 pickled 快照保存文件：


```
# enable memory history, which will
# add tracebacks and event history to snapshots
torch.cuda.memory._record_memory_history()

run_your_code()
torch.cuda.memory._dump_snapshot("my_snapshot.pickle")

```


# 使用可视化工具 [¶](#using-the-visualizer "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/torch_cuda_memory>
>
> 原始地址：<https://pytorch.org/docs/stable/torch_cuda_memory.html>


 打开 [pytorch.org/memory_viz](https://pytorch.org/memory_viz) 并将 pickled 快照文件拖/放到可视化工具中。可视化工具是一个在计算机本地运行的 JavaScript 应用程序。它不上传任何快照数据。


## 活动内存时间线 [¶](#active-memory-timeline "此标题的永久链接")


 活动内存时间线显示特定 GPU 上快照中随时间变化的所有实时tensor。平移/缩放绘图以查看较小的分配。将鼠标悬停在已分配的块上可查看分配该块时的堆栈跟踪以及其地址等详细信息。当存在大量数据时，可以调整细节滑块以减少分配并提高性能。


![_images/active_memory_timeline.png](_images/active_memory_timeline.png)


## 分配器状态历史记录 [¶](#allocator-state-history "此标题的固定链接") 


 分配器状态历史记录在左侧的时间轴中显示各个分配器事件。在时间线中选择一个事件以查看该事件的分配器状态的可视化摘要。此摘要显示了从 cudaMalloc 返回的每个单独段以及如何将其分割为单独分配或可用空间的块。将鼠标悬停在段和块上可查看分配内存时的堆栈跟踪。将鼠标悬停在事件上可查看事件发生时的堆栈跟踪，例如释放tensor时。内存不足错误将报告为 OOM 事件。在 OOM 期间查看内存状态可以深入了解分配失败的原因，即使保留内存仍然存在。


![_images/allocator_state_history.png](_images/allocator_state_history.png) 堆栈跟踪信息还报告发生分配的地址。地址 b7f064c000000_0 指的是地址 7f064c000000 处的 (b) 锁，即“_0” ”这个地址被分配的时间。这个唯一的字符串可以在活动内存时间轴中查找，并在活动状态历史记录中搜索，以检查分配或释放tensor时的内存状态。


# 快照 API 参考 [¶](#snapshot-api-reference "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/torch_cuda_memory>
>
> 原始地址：<https://pytorch.org/docs/stable/torch_cuda_memory.html>


 火炬.cuda.内存。


 _记录_记忆_历史


 (*已启用



 =
 


 '全部'
* , *上下文



 =
 


 '全部'
* , *堆栈



 =
 


 '全部'
* , *max_entries



 =
 


 9223372036854775807
* , *设备



 =
 


 无
* ) [[source]](_modules/torch/cuda/memory.html#_record_memory_history)[¶](#torch.cuda.memory._record_memory_history "此定义的永久链接")


 启用与内存分配相关的堆栈跟踪记录，这样您就可以知道在 [`torch.cuda.memory._snapshot()`](#torch.cuda.memory._snapshot "torch.cuda.memory. _快照”)。


 除了保留每个当前分配和释放的堆栈跟踪之外，这还可以记录所有分配/释放事件的历史记录。


 使用 [`torch.cuda.memory._snapshot()`](#torch.cuda.memory._snapshot "torch.cuda.memory._snapshot") 检索此信息，并使用 _memory_viz.py 中的工具来检索此信息可视化快照。


 Python 跟踪收集速度很快(每个跟踪 2us)，因此如果您预计需要调试内存问题，您可以考虑在生产作业中启用此功能。


 C++ 跟踪收集也很快(~50ns/帧)，对于许多典型程序来说，每个跟踪大约需要 2us，但可能会根据堆栈深度而变化。


 参数 
* **启用** ( *Literal
* *[
* *无
* *,
* *"state"
* *,
* *"all"
* *]
* *,
* *可选
* ) – 无，禁用记录内存历史记录。 “state”，保存当前分配的内存的信息。 “all” ，另外保留所有分配/释放调用的历史记录。默认为“all”。
* **context** ( *Literal
* *[
* *None
* *,
* *"state"
* *,
* *" alloc"
* *,
* *"all"
* *]
* *,
* *可选
* ) – None ，不记录任何回溯。 “state”，记录当前分配的内存的回溯。 “alloc”，另外保留 alloc 调用的回溯。 “all” ，另外保留免费调用的回溯。默认为“all”。
* **stacks** ( *Literal
* *[
* *"python"
* *,
* *"all"
* *]
* *,
* *可选
* ) – “python” ，在回溯“all”中包括 Python、TorchScript 和感应器框架，另外包括 C++ 框架默认为“all”。
* **max_entries** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*,
* *可选
* ) – 在记录的历史记录中保留最多 max_entries 分配/释放事件。


 火炬.cuda.内存。


 _快照


 ( *设备



 =
 


 无
* ) [[source]](_modules/torch/cuda/memory.html#_snapshot)[¶](#torch.cuda.memory._snapshot "此定义的永久链接")


 保存调用时 CUDA 内存状态的快照。状态表示为具有以下结构的字典。


```
class Snapshot(TypedDict):
    segments : List[Segment]
    device_traces: List[List[TraceEntry]]

class Segment(TypedDict):
    # Segments are memory returned from a cudaMalloc call.
    # The size of reserved memory is the sum of all Segments.
    # Segments are cached and reused for future allocations.
    # If the reuse is smaller than the segment, the segment
    # is split into more then one Block.
    # empty_cache() frees Segments that are entirely inactive.
    address: int
    total_size: int # cudaMalloc'd size of segment
    stream: int
    segment_type: Literal['small', 'large'] # 'large' (>1MB)
    allocated_size: int # size of memory in use
    active_size: int # size of memory in use or in active_awaiting_free state
    blocks : List[Block]

class Block(TypedDict):
    # A piece of memory returned from the allocator, or
    # current cached but inactive.
    size: int
    requested_size: int # size requested during malloc, may be smaller than
                        # size due to rounding
    address: int
    state: Literal['active_allocated', # used by a tensor
                'active_awaiting_free', # waiting for another stream to finish using
                                        # this, then it will become free
                'inactive',] # free for reuse
    frames: List[Frame] # stack trace from where the allocation occurred

class Frame(TypedDict):
        filename: str
        line: int
        name: str

class TraceEntry(TypedDict):
    # When `torch.cuda.memory._record_memory_history()` is enabled,
    # the snapshot will contain TraceEntry objects that record each
    # action the allocator took.
    action: Literal[
    'alloc'  # memory allocated
    'free_requested', # the allocated received a call to free memory
    'free_completed', # the memory that was requested to be freed is now
                    # able to be used in future allocation calls
    'segment_alloc', # the caching allocator ask cudaMalloc for more memory
                    # and added it as a segment in its cache
    'segment_free',  # the caching allocator called cudaFree to return memory
                    # to cuda possibly trying free up memory to
                    # allocate more segments or because empty_caches was called
    'oom',          # the allocator threw an OOM exception. 'size' is
                    # the requested number of bytes that did not succeed
    'snapshot'      # the allocator generated a memory snapshot
                    # useful to coorelate a previously taken
                    # snapshot with this trace
    ]
    addr: int # not present for OOM
    frames: List[Frame]
    size: int
    stream: int
    device_free: int # only present for OOM, the amount of
                    # memory cuda still reports to be free

```


 退货


 快照字典对象


 火炬.cuda.内存。


 _dump_快照


 ( *文件名



 =
 


 'dump_snapshot.pickle'
* ) [[source]](_modules/torch/cuda/memory.html#_dump_snapshot)[¶](#torch.cuda.memory._dump_snapshot "此定义的永久链接")


 将 torch.memory._snapshot() 字典的腌制版本保存到文件中。该文件可以通过 pytorch.org/memory_viz 上的交互式快照查看器打开


 Parameters


**文件名** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* *可选
* ) – 名称要创建的文件的名称。默认为“dump_snapshot.pickle”。