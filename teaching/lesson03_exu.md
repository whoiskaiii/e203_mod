# 第 3 课：EXU 执行（课件·待细化）

目标：讲清"指令是怎么被算出来的"。

待细化内容：
- 译码→派遣→执行→提交路径（decode/disp/exu）
- alu 子模块分工：dpath/rglr/bjp/csrctrl/muldiv
- OITF：乱序完成、有序提交，为什么需要
- 写回与长流水线写回（wbck/longpwbck）

实操：画 addi/beq 完整时序图；在 alu_dpath 加自定义运算并跑通。
验收题（示例）：E203 最多几条指令"在飞"？muldiv 为什么单独有 longpwbck？