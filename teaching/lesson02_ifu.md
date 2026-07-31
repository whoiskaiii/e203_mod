# 第 2 课：IFU 取指（课件·待细化）

目标：讲清"指令是怎么被拿进来的"。

待细化内容：
- ifetch/idecode 两级划分、PC 逻辑、指令对齐
- mini 译码器（e203_ifu_minidec.v）：取指阶段先"偷看"指令
- 轻量分支预测（e203_ifu_litebpu.v）：BTB 结构
- ift2icb（e203_ifu_ift2icb.v）：取指请求转 ICB 事务

实操：Verdi 跟踪 beq/jal 的 PC 变化；修改 litebpu 参数对比周期数。
验收题（示例）：分支预测错了会怎样？mini 译码器在流水线里承担什么角色？