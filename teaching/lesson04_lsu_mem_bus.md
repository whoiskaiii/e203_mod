# 第 4 课：LSU / 存储 / ICB（课件·待细化）

目标：讲清"数据是怎么被读写的"。

待细化内容：
- LSU 流水、load/store 类型、字节/半字拼接
- ITCM/DTCM 控制器、SRAM 接口
- ICB 协议：命令/返回通道、仲裁、qos
- BIU：内核与外设世界的桥梁

实操：跑 rv32mi-*/rv32ua-* 回归；改 DTCM 容量验证；Verdi 抓 ICB 事务。
验收题（示例）：一次 load 的完整旅程？ICB 与 AXI 最大区别？