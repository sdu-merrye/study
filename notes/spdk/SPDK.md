[TOC]

# changelog
## V17.10:Logical Volumes
### New dependencies
libuuid：for logical volumes
libnuma：for DPDK 17.08

### Block Device Abstraction Layer(bdev)
添加了一个fio插件，使IO能够路由到bdev layer.

spdk_bdev_unmap() 以字节偏移量和长度(bytes)作为参数，而不要求用户提供一系列的SCSI unmap描述符。这限制了unmap到一个连续的范围空间。

spdk_bdev_write_zeros()它允许所有指定的blocks被zerosd out。

增加了新的API函数它们以blocks为单位接收IO参数而不是以bytes为单位：
- spdk_bdev_read_blocks(), spdk_bdev_readv_blocks()
- spdk_bdev_write_blocks(), spdk_bdev_writev_blocks()
- spdk_bdev_write_zeroes_blocks()
- spdk_bdev_unmap_blocks()
bdev layer通过对排队I/O进行稍后重试来处理临时的out-of-memory I/O failures.

### Linux AIO bdev
AIO bdev现在允许用户覆盖the auto-detected block size.

### NVMe driver
the NVMe driver现在识别the NVMe 1.3 Namespace Optimal I/O Boundary field. NVMe 1.3设备可以报告一个最佳的I/O边界，当splitting I/O requests时driver会将这个考虑在内。

==================================================================================
# SPDK Directory Structure
## Overview
SPDK本质上是C lib的集合，目的是可以被应用直接使用。但是仓库中也含有许多examples和成熟的应用。

## Application
app 顶级目录中包括4个应用：
- app/iscsi_tgt: 一个iSCSI target
- app/nvmf_tgt: 一个NVMe-oF target
- app/iscsi_top: 信息工具(类似top),在iSCSI target中跟踪activity
- app/trace: 一个用于从iSCSI和NVMe-oF target中处理trace points的工具
- app/vhost: 一个vhost的应用，将virtio controller提供给基于QEMU的VMs并且处理提交给这些controllers的I/O

编译后，应用程序二进制文件将处于各自的目录中，所有这些都可以在没有参数的情况下运行，以打印出命令行参数。对于iSCSI和NVMe-oF target，它们都需要一个配置文件（-C选项）。完整的注释的配置文件的例子在etc/spdk目录。

## Build Collateral
build目录中包含所有在build过程中建立的静态库。lib目录和include/spdk目录为SPDK发行版的官方输出。

## Documentation
doc目录下包含SPDK的文档。API文档通过Doxygen直接用代码生成，但更多的文档和更长的解释也在这个目录中，这个目录也包含Doxygen config文件。

为了build这个文档，只用在这个doc目录下运行make。

## Examples
examples顶级目录包含一系列例子以供参考。这是一个学习SPDK如何工作的好地方，特别地，可以看一下examples/nvme/hello_world.

## include
include目录包含所有的头文件。the public API放在include/spdk中，我们强烈建议应用将顶级include目录和其中的spdk/...头文件放入include的path中，如：#include "spdk/nvme.h"
大多数头文件都和lib目录下的一个library相关。一下是一些stand alone的头文件：
- assert.h
- barrier.h
- endian.h
- fd.h
- mmio.h
- queue.h 和 queue_extras.h
- string.h

还有一个spdk_internal目录包含在spdk中广泛include的库的头文件，但这不是public API的部分且不会被安装在用户的系统中。

## Libraries
lib目录包含的是SPDK的核心。每个组件都是一个C库且在lib目录下有自己的目录

### Block Device Abstraction Layer
bdev目录包含一个block设备抽象层，它目前正被iSCSI和NVMe-oF targets使用。public interface是 include/spdk/bdev.h.这个库在写的时候缺乏清晰定义的功能，做了许多的事情：
- 从一个通用的block协议翻译为一个特定的协议如NVMe或者翻译为系统调用类似libaio.目前有3个块设备后端模块能够插入-libaio,SPDK NVMe,CephRBD,以及一个基于RAM的后端调用malloc。
- 提供一种从物理设备（例如RAID等）中构造虚拟块设备的机制。
- 处理数据缓冲区的内存分配。
这个layer也可以用一般的方式做I/O排队和划分。

### Configuration File Parser
conf目录包含配置文件解析器。the public header是 include/spdk/conf.h。配置文件格式类似INI，除了指令是"Name Value"而不是"Name = Value".这也是iSCSI和NVMe-oF targets的配置格式

## Makefile Fragments
mk目录包含一些用于构建系统的共享的Makefile片段

## Script
scripts目录包含了一些操作的简单脚本。最重要2个是check_format.sh,它会使用astyle和pep8来检查C,C++和python编码风格与我们定义的公约；setup.sh，它将设备从内核驱动中binds或unbinds。

## Tests
test目录包含了所有对SPDK组件以及其整个仓库的子目录镜像结构的测试。tests是unit tests和functional tests的混合。

============================================================================================
# Memory Management for User Space Drivers
下面是一个试图解释为什么所有的数据缓冲区传递到spdk必须使用spdk_dma_malloc()或其siblings来分配，为什么spdk依靠DPDK的基本功能来实现内存管理？

计算平台一般讲物理内存分割为4KiB的段称为pages。page数目为0-n从可寻址内存开始。OS然后通过任意复杂的映射来将4KiB虚拟内存页映射到物理页上。

物理内存被attached到channels上，其中每个内存channel提供固定的带宽。为了优化总体内存带宽，物理寻址通常设置为在通道之间自动交织。比如，page 0 可能放在channel 0，page 1在 channel 1，etc，这种是写入memory时自动利用所有可用channels。实际上，<font color="green">interleaving是在比整个page更细的粒度上完成的???</font>。

现代计算平台支持硬件加速MMU(memory Management unit)虚拟到物理的翻译，MMU支持多种不同的page size。在目前的x86_64系统中，支持4KiB，2MiB，1GiB。通常，OS默认使用4KiB。

NVMe drivers从/到系统memory中利用Direct memory access(DMA)来转换数据。特别地，他们在PCI总线上发送消息以请求数据传输。在IOMMU存在的情况下，这些信息中包含physical memory address。这些数据传输不需要CPU的参与，MMU负责访问内存相关。

NVMe设备对这些传输可能在内存的物理布局上有额外的要求。the NVMe 1.0规范要求所有物理内存以PRP list的列表描述。为了由PRP列表进行描述，内存必须包含以下属性：
- 内存被划分为4KiB的页，我们称为 device pages.
- 第一个device page可以是一个局部页开始的在任何4字节对齐的地址，它可以拓展到当前物理页的尾部，但不能超出
- 如果有一个以上的device page，the first device page必须在物理4KiB页边界停止
- 最后一个device page以物理4KiB页边界开始，但不要求以物理4KiB页边界结束

规范允许device pages为4KiB以外的大小，但所有已知设备都是利用4KiB来写入。

NVMe 1.1规范增加了完全灵活的分散聚集表支持，但功能是可选的，大多数今天可用的设备不支持。
<br/>
<br/>
<br/>


用户空间驱动程序在常规进程的上下文中运行，因此可以访问虚拟内存。为了正确地对具有物理地址的设备进行编程，必须实现一些地址转换的方法。

在linux上最简单的方法是检查一个进程的/proc/self/pagemap。这个文件包含了虚拟地址到物理地址的映射。对Linux 4.0，访问这些映射需要root权限。然而，OS不保证虚拟到物理映射是静态的。操作系统对PCI设备是否直接将数据传送到一组物理地址没有可见性，因此必须非常小心地将DMA请求与页面移动协调起来。当操作系统标记了一个页面时，使得不能修改虚拟到物理地址映射，这称为pinning the page

虚拟到物理映射改变是有一些原因的。到目前为止，最常见的原因是页面交换到磁盘。然而，操作系统在压缩过程中也会移动页面，这些过程将相同的虚拟页压缩到同一物理页上以节省内存。有些操作系统也能够进行透明的内存压缩，这提升了热附加内存的可能，这可能触发物理地址的重新平衡以优化interleaving。

POSIX提供mlock调用，它强制内存的虚拟页总是由一个物理页支持。实际上，这是禁用了swapping.但这并不保证虚拟到物理地址映射是静态的。mlock调用不应与pin调用混淆，实时上POSIX没有定义一个API以pinning memory.因此，分配pinned memory的机制是OS特有的。

SPDK依赖DPDK来分配pinned memory.在linux中，DPDK通过分配hugepages(默认2MiB)来实现。Linux内核对待hugepages与对待常规4KiB pages不同，特别地，OS不会改变他们的物理位置。这不是有意的，并且将来的版本可能会发生变化，但今天是这样的，而且已经有好几年了(可以看一看后面IOMMU部分对未来的解决办法)。DPDK经过煞费苦心的hugepages分配，这样它可以将物理页串在一起以使得运行时间尽可能长，这样，它可以容纳大于单页的物理上连续分配的页。

希望解释清楚了为什么所有的数据缓冲区传递到spdk必须使用spdk_dma_malloc()或其siblings分配。缓冲区必须被特别分配，以便它们被固定，以便知道其物理地址。

# IOMMU Support
许多平台包含一个额外的硬件部分称为I/O memory management unit(IOMMU).IOMMU很像常规的MMU，除了它为外围设备提供了虚拟地址空间（如PCI总线上）。MMU知道系统上每个进程的虚拟到物理地址映射，因此IOMMU将一个特定设备与这些映射中的一个关联起来，然后允许用户在其进程中指定任意bus address到一个虚拟地址。所有PCI设备和系统内存之间的DMA操作通过IOMMU翻译，从bus address到虚拟地址再到物理地址。这允许操作系统自由地修改虚拟到物理地址映射，而不会中断正在进行的DMA操作。Linux提供一个device driver,vfio-pci,它允许用户与他们的进程来配置IOMMU。

这是一个未来的证明，硬件加速解决方案以执行DMA操作进/出用户空间进程以及形成了spdk和DPDK的内存管理策略的长期基础。我们强烈建议应用程序部署和使用vfio和IOMMU，这是现今完全支持的。

==============================================================================================
# SPDK Porting Guide
spdk通过实现**env**库接口移植到新的环境中。**env** 接口为drivers提供API以分配物理连续的且固定的memory，执行PCI操作<font color="green">（配置cycles以及mapping BARS???）</font>，虚拟到物理地址翻译以及管理memory pool. **env** API定义在**include/spdk/env.h**中。

spdk包括基于数据平面开发工具包(DPDK)的对env库的默认实现（DPDK）。这个实现可以在lib/env_dpdk找到。

DPDK是目前只支持Linux和FreeBSD。想用spdk到其他操作系统的用户，或想用除DPDK以外的用户态驱动框架，需要实现一个新的env库的版本。新的实现可以被集成到spdk中，通过在CONFIG中更新如下操作：
```
CONFIG_ENV?=$(SPDK_ROOT_DIR)/lib/env_dpdk
```

==============================================================================================
# Blobstore
## Introduction
blobstore是一个持久的,掉电安全的block allocator，设计用来作为本地存储系统以支持更高级别的存储服务，通常代替传统的文件系统。这些更高层次的服务可以是本地数据库或键/值存储（MySQL，rocksdb），他们可以是专用的应用（SAN，NAS），或分布式存储系统（如Ceph,Cassandra）.它不是设计为一个通用的文件系统，但是，它是故意不符合POSIX标准。为了避免混乱，没有使用file或objects，而不是使用术语"blob"。该设计允许异步、无缓存、并行读写到一个块设备上地一组blocks上，称为"blobs"。Blobs通常都很大，至少有几百KB，一直是底层块大小的倍数。

blobstore设计主要是对“下一代”的媒体，这意味着该设备支持快速随机读取和写入，无需后台垃圾收集。然而，在实践中设计也将运行在NAND。很明显地没有必要在旋转媒体上做过多优化努力。

## Design Goals
blobstore的目的是解决一些本地数据库使用传统的POSIX文件系统时的一些问题。这些数据库被假定为拥有整个存储设备，不需要跟踪访问时间，只需要一个非常简单的目录层次结构。这些假设允许在传统的POSIX文件系统和块堆栈上进行显著优化设计。

异步I/O可以比同步I/O快一个数量级以上，因此类似libaio的解决方案已成为流行。然而，libaio不是在所有情况下都是实际异步的。blobstore在所有的情况下将提供真正的异步操作，没有任何隐藏的锁或stalls。

随着NVMe的到来，存储设备现在有一个硬件接口，允许threads提交高度并行I/O且没有锁。不幸的是，在设备上放置数据需要一些中央协调以避免冲突。blobstore将需要协调的操作和不需要协调的操作分离开来，并允许用户显式地将I/O与channels连接起来。不同通道上的操作是并行的，一直到硬件，没有锁或协调。

随着介质访问延迟的提高，**in-memory缓存策略正在发生变化且通常内核页缓存是一个瓶颈**。_许多数据库变为在o_direct模式下打开文件，以完全避免页面缓存，且编写自己的缓存层。_ 随着下一代介质的推出及其额外的预期延迟减少，这一策略将变得更加流行。为此，blobstore将对所有数据执行no in-memory caching，基本上所有的blob操作概念上等同于o_direct。这意味着有与o_direct类似的限制，blobstore数据只能读或写在单位页上（4kib），虽然内存对齐要求的严格性比o_direct少得多（页面甚至可以由分散的缓冲区组成）。我们完全承认DRAM缓存对性能仍然是至关重要的，但将缓存设计的细节留给更高层。

存储设备使用DMA引擎将数据从主机内存中pull出来，且DMA引擎对物理地址进行操作且引入了对齐的限制。此外，为了避免数据corruption，数据必须不能被OS paged out即使它已经被传输到disk上。传统上，操作系统解决这个问题可以通过将用户数据复制到为这一目的而分配的特殊的内核缓冲区中，而后I/O操作在这里to/from执行，也或者以锁来标记所有用户页面为锁定的且不可动的。从历史上看，执行copy或locking的时间与在存储设备中的I/O时间是没有相关性的，但这个结论不再是这种情况。blobstore将提供零拷贝，无锁的读写存取装置。为此，用于BLOB数据的memory必须在blobstore之前注册，最好是在应用程序启动和退出的I/O路径上，以便它可以固定的物理地址，也可以验证alignment要求。

硬件设备必然是受一些最大队列深度限制的。对NVMe设备队列深度可以相当大（允许多达64K！），但一般队列较小（128—1024每队列）。在较重负载下，数据库可能产生超过硬件队列深度足够的请求，这就需要在软件中排队。操作系统通常是在通用块层，但可能会导致意想不到的停顿或需要锁。当队列满时，blobstore将失败的请求给予相应的错误代码以避免这种情况。这使得blobstore容易保证其承诺的永不block，但可能会要求用户提供自己的queueing layer。

## The Basics
Blobstore定义了一个3个单元的磁盘空间结构。最小的是_logical blocks_,它由磁盘自身暴露，标号0-N，其中N是disk中blocks的数目。一个logical block通常为512B或4KiB.

blobstore定义一个page为在blobstore创建时固定数目的logical blocks.组成一个page的logical blocks是连续的。 pages从磁盘的开始处编号，第一个页为0号页，第二页是1号页。一个page通常为4kib大小，所以它实际上是8或1个logical blocks。该设备必须能够执行至少在页大小上的原子读写操作。

最大的单元是cluster,它是在blobstore创建时定义的一组固定数目的pages.组成cluster的pages是连续的.cluster从disk开始处进行编号，一个cluster通常是1MiB，即256 pages.

在这3个基本单元之上，blobstore定义了3个原语。最基本的是blob，blob是一个有序的cluster的链表再加一个identifier.Blobs在掉电和重启时页存在。所有的blob通过共享元数据的集合来描述，集合称为blobstore。blob上的I/O操作是通过channels提交的。channels与threads捆绑在一起，但是多个线程可以在自己的通道上同时向同一个blob提交I/O操作。

Blobs通过在指定的虚拟地址空间中指定一个偏移量，以页为单位读取和写入数据块。首先通过确定访问哪个cluster，然后转换成一组logical blocks来翻译这个偏移量。这个翻译是使用平凡的方法，只使用基本的数学而没有映射数据结构。不同于读和写，blobs以cluster为单位来resized。

Blobs由他们的元数据来描述，它包含了一个不连续的pages集合并存储在磁盘的保留区中。metadata的每个page被称为metadata page.Blobs不与其他blobs共享metadata pages，实际上，设计依赖于后端存储设备所支持的原子写入单元大于或等于页大小。大多数由NAND和下一代媒体支持的设备支持这种原子写入能力，但通常磁介质不会。

元数据区的容量是固定的且是在blobstore创建时定义的。大小是可配置的，但默认情况下一个页是分配给每个cluster。对于1mib cluster和4kib页，会导致0.4%的元数据开销。

## Conventions
设备上的数据格式是指定的巴科斯范式(Backus-Naur Form)。所有的数据都以小端格式存储在介质中。未指定的数据必须被清零。

## Media Format
blobstore拥有整个存储设备，设备从一开始就分成clusters，如cluster 0 从第一个逻辑块开始。
```
LBA 0                                   LBA N
+-----------+-----------+-----+-----------+
| Cluster 0 | Cluster 1 | ... | Cluster N |
+-----------+-----------+-----+-----------+
```
或者用形式标记：
```
<media-format> ::= <cluster0> <cluster>*
```
集群0是特殊的，具有以下格式，其中page 0是cluster的第一页：
```
+--------+-------------------+
| Page 0 | Page 1 ... Page N |
+--------+-------------------+
| Super  |  Metadata Region  |
| Block  |                   |
+--------+-------------------+
```
或正式的：
```
<cluster0> ::= <super-block> <metadata-region>
```
super block是一个单页位于该分区的开始部分。它包含有关blobstore基本信息。元数据区域是cluster 0的剩余部分且可以延伸到额外的clusters。
```
<super-block> ::= <sb-version> <sb-len> <sb-super-blob> <sb-params>
                  <sb-metadata-start> <sb-metadata-len> <crc>
<sb-version> ::= u32
<sb-len> ::= u32 # Length of this super block, in bytes. Starts from the
                 # beginning of this structure.
<sb-super-blob> ::= u64 # Special blobid set by the user that indicates where
                        # their starting metadata resides.

<sb-md-start> ::= u64 # Metadata start location, in pages
<sb-md-len> ::= u64 # Metadata length, in pages
<crc> ::= u32 # Crc for super block
```
<sb-params>数据包含由用户指定的当BLOB store最初格式化的params参数。
```
<sb-params> ::= <sb-page-size> <sb-cluster-size> <sb-bs-type>
<sb-page-size> ::= u32 # page size, in bytes.
                       # Must be a multiple of the logical block size.
                       # The implementation today requires this to be 4KiB.
<sb-cluster-size> ::= u32 # Cluster size, in bytes.
                          # Must be a multiple of the page size.
<sb-bs-type> ::= char[16] # Blobstore type
```
每个blob的元数据在元数据区域被分配一组非连续的页。这些pages形成了一个链表，链表中的第一个page在更新时会被就地重写，而其他pages将被写到一个新的地方。这就要求后端设备支持大于或等于页大小的原子写入大小，以保证操作是原子的。本节也可看到原子的细节部分。
每一页的定义是：
```
<metadata-page> ::= <blob-id> <blob-sequence-num> <blob-descriptor>*
                    <blob-next> <blob-crc>
<blob-id> ::= u64 # The blob guid
<blob-sequence-num> ::= u32 # The sequence number of this page in the linked
                            # list.

<blob-descriptor> ::= <blob-descriptor-type> <blob-descriptor-length>
                        <blob-descriptor-data>
<blob-descriptor-type> ::= u8 # 0 means padding, 1 means "extent", 2 means
                              # xattr. The type
                              # describes how to interpret the descriptor data.
<blob-descriptor-length> ::= u32 # Length of the entire descriptor

<blob-descriptor-data-padding> ::= u8

<blob-descriptor-data-extent> ::= <extent-cluster-id> <extent-cluster-count>
<extent-cluster-id> ::= u32 # The cluster id where this extent starts
<extent-cluster-count> ::= u32 # The number of clusters in this extent

<blob-descriptor-data-xattr> ::= <xattr-name-length> <xattr-value-length>
                                 <xattr-name> <xattr-value>
<xattr-name-length> ::= u16
<xattr-value-length> ::= u16
<xattr-name> ::= u8*
<xattr-value> ::= u8*

<blob-next> ::= u32 # The offset into the metadata region that contains the
                    # next page of metadata. 0 means no next page.
<blob-crc> ::= u32 # CRC of the entire page
```
描述符不能跨越元数据页。

## Atomicity
blobstore的元数据是缓存的，必须显式被用户同步。数据没有被缓存，但是，当元数据同步后一个写操作完成的数据可以被认为是durable的。元数据不经常改变的，事实上只有一下这些明确的操作后才必须同步：
- resize
- set xattr
- remove xattr

其他操作不会弄脏元数据。此外，每个块的元数据是独立的，所以只需要在特定的脏的blob上进行同步操作。

元数据由页链接列表组成。对元数据的更新首先要将page 2到N写到一个新的地址上，就地重写page 1以自动更新链表，然后擦除旧链的其余部分。绝大多数时候，blob只包含一个元数据页，所以这个操作非常有效。为了使该方案工作，对第一页的写入必须是原子的，这需要从后端设备中获得硬件支持。最重要的是，如果不是所有的都是NVMe固态硬盘，那期望使用4kib的原子写单位。设备在其NVMe identity data中指定其原子写的单位--特别是在 AWUN field中。

=================================================================================================
# BlobFS（Blobstore Filesystem）
## BlobFS Getting started Guide
## RocksDB integration
（RockDB集成到SPDK中的步骤，省略。。。）
## FUSE
blobfs提供FUSE插件以挂在spdk blobfs作为内核文件系统用于检查和调试。FUSE插件需要fuse3并且当系统上监测到fuse3时将会被自动build。
FUSE插件的限制如下：
## Limitation
- blobfs目前主要用rocksdb来测试，所以当使用用例与RocksDB使用文件系统的方式不同时可能会导致一些问题。blobfs在最初的版本后将在更大范围的情况下进行测试。
- 目前只支持同步API。已经开发了一个异步API，但尚未彻底测试，因此还不是公共接口的一部分。这将会在将来的版本中添加。
- 文件重命名不是原子的，将在未来版本中进行修复。
- blobfs目前支持不支持文件目录，而是只有一个平面的命名空间。文件名当前是存储在每个blob的xattrs中。这意味着文件名查找是O(n)的操作。一个spdk BTree实现正在进行，在未来的版本这将支撑blobfs的目录支持。
- 对文件的写入必须始终附加到文件的结尾。在将来的版本中将添加对文件中任何位置的写入支持。

===============================================================================================
# Block Device Abstraction Layer
## SPDK bdev Gettiing Started Guide
在spdk应用的块存储由spdk bdev层提供。spdk bdev由一下组成：
- 一个driver模块的API，实现bdev drivers
- 一个应用API，用于枚举和claiming SPDK 块设备和在这些设备上的性能操作(read,write,unmap,etc)
- bdev drivers，for NVMe,malloc(ramdisk),Linux AIO和Ceph RBD
- 配置，通过SPDK配置文件或者JSON RPC

## Configuring block devices
spdk块设备通常是通过一个spdk配置文件配置。这些块设备可以与更高层次的抽象，如iSCSI目标节点、NVMe-oF命名空间或vhost-scsi控制器关联。这一节将描述如何配置的spdk中已经包括了的spdk bdev drivers。

spdk配置文件通常是通过命令行传递给你的spdk应用。有关详细信息，请参阅应用程序的帮助工具。
