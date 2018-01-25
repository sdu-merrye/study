# pblk: Host-based FTL for Open-Channel SSDs
The Physical Block Device(pblk)实现了基于sector的、全相连的、基于host的FTL，
它暴露一个传统的block I/O接口。pblk的主要职责有：
- (L2P table) 在L2P table中将逻辑地址映射到物理地址(4kB的粒度)
- (L2P table) 维持L2P table的数据完整性和数据一致性，同时在掉电时恢复
- 管理 controller- 和 media-specific 约束
- 处理 write和erase errors
- Implement garbage collection (GC) routines to garbage collect blocks
- 当从FS产生flush后实现padding，以维持数据一致性

pblk初始化为一个用户定义的LUNs的范围。一个设备可以被划分并被不同pblk实例共享，每个
实例拥有设备的一部分capacity和throughput，每个实例管理设备上不同的parallel units。
