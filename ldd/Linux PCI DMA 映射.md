Linux PCI DMA 映射:

一、Linux系统中DMA的映射。

        Linux DMA映射有两种：一致性映射和流式映射。两者的区别是：

一致性映射可以保证DMA的缓冲区设备端和驱动程序端都同时可见，而流式映射在同一时刻只有一方可见，另外的一方如果要查看DMA缓冲区需要先使缓冲区对另一端不可见；
一致性映射的接口比流式映射的简单，但是不能申请超过128K的空间。

二、一致性映射

       一致性映射只有两个操作函数：
建立一致性映射：void *dma_alloc_coherent(struct device *dev, size_t size, dma_addr_t *dma_handler,  int flag);
释放一致性映射：void dma_free_coherent(struct device *dev, size_t size, void *vadd,  dma_addr_t   dma_handler);
          建立一致性映射的时候，需要传递，设备的的数据结构指针、需要DMA的大小和这段内存区的标志。
          函数返回的地址是供CPU驱动操作的，而DMA的地址则通过dma_handler返回的地址是告诉PCI设备的DMA地址属于总线地址，在某些系统上是和物理地址一致的。

三、流式映射

        流式映射可以使用大段的内存做DMA映射，这些内存先有驱动程序通过kmalloc函数或__get_free_pags函数来申请，然后调用流式DMA映射的接口映射DMA，然后得到DMA地址。
        流式映射不同于一致性映射，设备和CPU不能同时访问流式DMA的内存。一但DMA内存被映射这段内存将属于设备，在映射被撤销之前，CPU是不能访问这段内存的。与此相同当映射被撤销之后这段内存就属于CPU设备将不能再访问这段内存区域。同时流式映射还要告诉内核DMA数据流的流动方向：

DMA_TO_DEVICE         数据从CPU发送到设备
DMA_FROM_DEVICE   数据从设备发送到CPU
DMA_BIDIRECTIONAL  数据双向流动

        流式映射的操作函数有3中：分别用来映射：一个内存页、一段内存、分散/聚集映射。调用这些函数之后，成功映射之后返回dma_addr_t 类型的地址，这个地址就是需要告诉设备的DMA总线地址，在释放映射的时候需要告知这个地址。

四、分散/聚集映射

        映射一段连续的内存区比较简单，如果需要使用超过4M的DMA内存区域就需要使用分散/聚集映射，因为内置中最多只能申请到4M的内存块。对应的解决方法就是：

先申请多个4M的内存块，如需要8M的内存就申请4个4M的内存块；
从这几个内存块中找到2个物理地址是连续的内存块，然后将其他的内存块释放；
对比这个内存块的虚拟地址，找到虚拟地址小的那个地址作为CPU访问DMA缓冲区的虚拟地址；
调用sg_init_tab初始化struct scatterlist 结构体数组，然后对数组中的每个元素调用sg_set_buf来设置连续的内存地址和长度；
调用dma_map_sg函数来映射，映射成功之后通过sg_dma_address函数来获取每个scatterlist元素的DMA地址，然后将DMA地址中小的那个作为DMA总线地址；
在中间过程中使用dma_sync_sg_for_cpu来获取CPU访问DMA的控制权，使用后再调用dma_sync_sg_for_device再将DMA缓冲区的控制权还给设备；
在最后不在使用的时候，使用dma_unmap_sg来释放分散/聚集映射，注意这个函数中使用struct scatterlist数组的必须是和dma_map_sg相同的数组；
释放第二步保留的内存块

          注意：
申请的内存块必须是物理地址连续的内存块，例如使用__get_free_pages
在初始化struct scatterlist数组的时候，需要先调用sg_init_tab(数组， 数组大小)，然后对数组中每个元素调用sg_set_buf(scatterlist元素， 申请到的内存块地址， 内存块的大小)
dma_map_sg、dma_sync_sg_for_cpu、dma_sync_sg_for_device、dma_unmap_sg使用的scatrerlist 数组必须是同一个数组
       

  【算法】检查多个内存块物理地址是否是连续的算法：

首先，找出这些内存块物理地址的最大值和最小值；
然后用最大值减去最小值，用相减后得到的差值除以内存块的大小；
如果除后得到的商，等于申请的内存块个数 - 1 ，那么说明这几个内存块的物理地址是连续的。















