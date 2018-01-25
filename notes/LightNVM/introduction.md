[toc]


## <font color="#00ffff">1. Open-Channel Solid State Drives</font>
向主机端暴露内部并行性并允许其管理。为host提供3个特性: I/O isolation, predictable latencies 和 Software-defined non-volatile memory。
- I/O Isolation: 将SSD的容量划分成一系列IO channels，对应设备的并行单元LUNs。
- Predictable latency：在主机端控制when,where and how I/O 提交给SSD
- Software-Defined Non-Volatile Memory：将SSD的FTL集成到host中，workload的优化可以被应用到自包含的FTL、FS或application自身。
<br/><br/><br/>

![figure1](https://openchannelssd.readthedocs.io/en/latest/LightNVMArch.png)

host:<br/>
|----实现通用FTL功能，data placement，I/O scheduling 和 GC <br/>
|---向用户空间暴露传统块设备(<font color="green">不是很明白向上暴露？？</font>)

controller: <br/>
|----处理Media-centric metadata, error handling, scrubbing and wear-leveling
(<font color="green">physical page addresssing???</font>)


## <font color="#00ffff">2. Getting Started</font>
安装：

三个组件:
- linux kernel 4.12+ (包含了PBLK组件)
- Latest nvme-client (管理nvme设备)
- qemu with Open-Channel support (模拟oc-ssd)

## <font color="#00ffff">3. Command Line</font>
通过nvme-cli可以用于create,delete,manage LightNVM的targe和devices。(<font color="green">target？？？</font>)

命令：
```
nvme lnvm --help
nvme lnvm list        #列出用LightNVM注册的设备
nvme list info        #列出available targets和其版本
#设备在第一次使用之前要先注册（pre-4.11）
nvme lnvm init -d $DEVICE
#向一个通过gennvm media manager注册的设备顶部添加一个target？？？
nvme lnvm create -d $DEVICE -n $TARGET_NAME -t $TARGET_TYPE -b $LUN_BEGIN -e $LUN_END
#移除target实例
nvme lnvm remove -n targetname
```
- $DEVICE: Backend device. lnvm devices to list available devices.
- $TARGET_NAME: Name of the target to be exposed -> /dev/$TARGET_NAME
- $TARGET_TYPE: Target type. Targets need to be compiled individually before they can be instantiated at run-time. For now, rrpc and pblk are the available implementations.
- $LUN_BEGIN: Lower bound of the LUN range allocated to the target.
- $LUN_END: Higher bound of the LUN range allocated to the target.

成功注册target之后，就可以对/dev/$TARGET_NAME进行读写。如：使用NVMe设备nvme0n1，将其中LUNs 0到63分配给一个pblk实例：
```
nvme lnvm create -d nvme0n1 -n mydevice -t pblk -b 0 -e 63
```

## <font color="#00ffff">4. Projects</font>
Open-Channel SSD架构划分为多个pieces。
- Linux Kernel Support (https://github.com/OpenChannelSSD/linux) <br/>
Implements support for Open-Channel SSDs in the kernel. It is the core of identifying, managing, and using the Open-Channel SSDs.
- liblightnvm Library (https://github.com/OpenChannelSSD/liblightnvm) <br/>
A library that abstracts the underlying "raw" Open-Channel SSD device and provides abstractions such as append only, bad block management, etc.
(<font color="green">为什么安装的时候没看到这个？？？</font>)
- LightNVM Conditioning Tool (https://github.com/OpenChannelSSD/lnvm) <br/>
Provides the lnvm-tool cli management tool for verifying and conditioning Open-Channel SSDs.
- LightNVM Test Tools (https://github.com/OpenChannelSSD/lightnvm-hw) <br/>
Various tools to test kernel and target implementations.
- QEMU with NVMe Open-Channel Support (https://github.com/OpenChannelSSD/qemu-nvme) <br/>
Implements support for exposing a virtual open-channel SSD. Very useful for development.

## <font color="#00ffff">5. Source files</font>
### Linux kernel
代码结构：

LightNVM框架被划分为对设备驱动的支持，核心部分由management、block manager和targets组成。

当前NVMe设备驱动在
```
drivers/nvme/host/pci.c
```
已经通过以下方式扩展了对Lightnvm设备的支持:
```
drivers/nvme/host/lightnvm.c
```
Lightnvm核心的源代码放置在
```
drivers/lightnvm/core.c
include/linux/lightnvm.h
```
pblk和rrpc targets放在：
```
drivers/lightnvm/pblk.[ch]
drivers/lightnvm/rrpc.[ch]
```
### Kernel API
LightNVM API:
```
/include/linux/lightnvm.h
```

user-space API:  
```
/include/uapi/linux/lightnvm.h
```
导出的符号在：
```
/drivers/lightnvm/core.c
```
包含下列functions的文档：

| Target specific |  |
| - | :- |
| nvm_register_target | Register a target with the LightNVM subsystem |
| nvm_unregister_target | Unregister a target with the LightNVM subsystem |
| nvm_dev_dma_free | Frees device DMA-able memory |
| nvm_addr_to_generic_mode | Convert from device-specific to generic addressing |
| nvm_generic_to_addr_mode | Convert from generic to device-specific addressing |
| nvm_set_rqd_ppalist | Sets nvm_rqd with associated ppa list |

| Media manager specific |  |
| - | :- |
| nvm_register_mgr | Register media manager with LightNVM subsystem |
| nvm_unregister_mgr | Unregister a media manager from the LightNVM subsystem |
| nvm_get_blk_unlocked |  Get free logical block from logical lun |
| nvm_put_blk_unlocked | Put logical block to logical lun |
| nvm_get_blk | Get free logical block from logical lun |
| nvm_put_blk | Put logical block to logical lun |

| Device specific |  |
| - | :- |
| nvm_register | Register a device with the LightNVM subsystem |
| nvm_unregister | Unregister a device from the LightNVM subsystem |
| nvm_submit_io | submit an io to the underlying media media manager |
| nvm_erase_blk | submit erase of the generic nvm_block |
| nvm_erase_ppa | Erase the block(s) associated with physical addresses |
| nvm_submit_ppa | Submits a list of ppas to device synchronously.|
| nvm_end_io |  Complete a nvm_rq request |

如何调用API见示例：
```
Media manager
  gennvm - /drivers/lightnvm/gennvm.c

Target
  pblk - /drivers/lightnvm/pblk.c
  rrpc - /drivers/lightnvm/rrpc.c

Device-side
  NVMe - /drivers/nvme/host/lightnvm.c
  null_blk - /drivers/block/null_blk.c
```
