# 第 5 课：中断 / 调试 / 外设（课件·待细化）

目标：讲清"突发事件怎么被处理"。

待细化内容：
- CLINT（mtime/msip）与 PLIC（优先级）
- 中断响应全流程：mtvec → 保存现场 → mret
- JTAG→DTM→DM 调试链路
- 外设挂载方式（UART/GPIO 举例）

实操：跑 tb 中断用例跟踪流程；sirv-e-sdk 编译 demo_gpio 仿真验证。
验收题（示例）：中断和异常的区别？mret 恢复了哪些状态？