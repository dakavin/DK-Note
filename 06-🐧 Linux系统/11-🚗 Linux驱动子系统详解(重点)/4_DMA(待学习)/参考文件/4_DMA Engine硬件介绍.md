## DMA channels

一个DMA controller可以“同时”进行的DMA传输的个数是有限的，这称作DMA channels。当然，这里的channel，只是一个逻辑概念，因为：

> 鉴于总线访问的冲突，以及内存一致性的考量，从物理的角度看，不大可能会同时进行两个（及以上）的DMA传输。因而DMA channel不太可能是物理上独立的通道；
> 
> 很多时候，DMA channels是DMA controller为了方便，抽象出来的概念，让consumer以为独占了一个channel，实际上所有channel的DMA传输请求都会在DMA controller中进行仲裁，进而串行传输；
> 
> 因此，软件也可以基于controller提供的channel（我们称为“物理”channel），自行抽象更多的“逻辑”channel，软件会管理这些逻辑channel上的传输请求。实际上很多平台都这样做了，在DMA Engine framework中，不会区分这两种channel（本质上没区别）。

## DMA request lines

  

DMA传输是由CPU发起的：CPU会告诉DMA控制器，帮忙将xxx地方的数据搬到xxx地方。CPU发完指令之后，就当甩手掌柜了。而DMA控制器，除了负责怎么搬之外，还要决定一件非常重要的事情（特别是有外部设备参与的数据传输）：

> 何时可以开始数据搬运？

因为，CPU发起DMA传输的时候，并不知道当前是否具备传输条件，例如source设备是否有数据、dest设备的FIFO是否空闲等等。那谁知道是否可以传输呢？设备！因此，需要DMA传输的设备和DMA控制器之间，会有几条物理的连接线（称作DMA request，DRQ），用于通知DMA控制器可以开始传输了。

这就是DMA request lines的由来，通常来说，每一个数据收发的节点（称作endpoint），和DMA controller之间，就有一条DMA request line（memory设备除外）。

  

最后总结：

  

> DMA channel是Provider（提供传输服务），DMA request line是Consumer（消费传输服务）。在一个系统中DMA request line的数量通常比DMA channel的数量多，因为并不是每个request line在每一时刻都需要传输。

## 传输参数

在最简单的DMA传输中，只需为DMA controller提供一个参数----transfer size，它就可以欢快的工作了：

> 在每一个时钟周期，DMA controller将1byte的数据从一个buffer搬到另一个buffer，直到搬完“transfer size”个bytes即可停止。

  

不过这在现实世界中往往不能满足需求，因为有些设备可能需要在一个时钟周期中，传输指定bit的数据，例如：

  

> memory之间传输数据的时候，希望能以总线的最大宽度为单位（32-bit、64-bit等），以提升数据传输的效率；
> 
> 而在音频设备中，需要每次写入精确的16-bit或者24-bit的数据；
> 
> 等等。

  

因此，为了满足这些多样的需求，我们需要为DMA controller提供一个额外的参数----transfer width。

另外，当传输的源或者目的地是memory的时候，为了提高效率，DMA controller不愿意每一次传输都访问memory，而是在内部开一个buffer，将数据缓存在自己buffer中：

  

> memory是源的时候，一次从memory读出一批数据，保存在自己的buffer中，然后再一点点（以时钟为节拍），传输到目的地；
> 
> memory是目的地的时候，先将源的数据传输到自己的buffer中，当累计一定量的数据之后，再一次性的写入memory。

  

这种场景下，DMA控制器内部可缓存的数据量的大小，称作burst size----另一个参数。

  

## scatter-gather

一般情况下，DMA传输一般只能处理在物理上连续的buffer。但在有些场景下，我们需要将一些非连续的buffer拷贝到一个连续buffer中（这样的操作称作scatter gather，挺形象的）。

对于这种非连续的传输，大多时候都是通过软件，将传输分成多个连续的小块（chunk）。但为了提高传输效率（特别是在图像、视频等场景中），有些DMA controller从硬件上支持了这种操作。

注2：具体怎么支持，和硬件实现有关，这里不再多说（只需要知道有这个事情即可，编写DMA controller驱动的时候，自然会知道怎么做）。