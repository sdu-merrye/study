# Liblightnvm : User space I/O library for Open-Channel SSDs
liblightnvm为用户空间提供方法来与open-chennel SSDs进行交互和获取其信息。


库的核心是提供了一个接口，用于使用物理寻址(physical addressing)执行矢量化I/O (vectorized I/O)。将命令行指令进行封装，包括从通用物理地址映射到特定设备。这个接口
对于由Open-channel SSDs提供的vectorized I/O进行实验很有用。这个库还支持对这些抽象进行分段，并允许用户构造和提交“RAW”命令。
