# 蜂鸟 E203 教学计划（e203_mod）

## 课程进度

| 课 | 主题 | 状态 |
|---|---|---|
| 0 | 环境与仿真流程 | 教案已细化；讲授/演示已完成；等待实操 + 验收 |
| 1 | RISC-V 指令集 + E203 架构地图 | 教案已细化；待开讲 |
| 2 | IFU 取指 | 待细化 |
| 3 | EXU 执行 | 待细化 |
| 4 | LSU / 存储 / ICB | 待细化 |
| 5 | 中断 / 调试 / 外设 | 待细化 |
| 6 | FPGA + 毕业项目 | 待细化 |

## 一、教学大纲（总体）

### 课程定位

- 课程名称：蜂鸟 E203 RISC-V 处理器源码精讲与实战
- 面向对象：具备数字电路基础，想深入理解 RISC-V CPU 设计与验证的学习者
- 终极目标：
  1. 能独立读懂 E203 的全部 RTL（内核 + SoC）
  2. 能修改内核：加自定义指令、改存储配置、换外设，并在仿真与 FPGA 上验证
  3. 具备衔接 e203_hbirdv2（NICE 扩展）或自研 CPU 的能力
- 教学法：讲解 + 实操 + 验收；先波形后代码；每课必练、练完必验

### 前置要求

- 会 Verilog 基本语法：模块、always、wire/reg、例化
- 会基本 Linux 命令行（cd/make/grep）
- 不需要 RISC-V 基础（第 1 课从零讲起）
- 工具已就绪：VCS + Verdi、riscv-tests、自制 K325t 板卡

### 课程全景（3 阶段 7 课）

阶段 A 地基：工具链与世界观
- 课 0 环境与仿真流程 —— 建立"改代码 → 仿真 → 看波形"的循环
- 课 1 RISC-V 指令集 + E203 架构地图 —— 建立全局认知

阶段 B 内核核心（全课程重点）
- 课 2 IFU 取指 —— 指令怎么进来
- 课 3 EXU 执行 —— 指令怎么算
- 课 4 LSU / 存储 / ICB —— 数据怎么读写
- 阶段 B 结束：中期小测

阶段 C 系统与落地
- 课 5 中断 / 调试 / 外设 —— 突发事件与 SoC 完整性
- 课 6 FPGA 上板 + 毕业项目 —— 真硬件验证

### 学习路径

课0(工具链) → 课1(基础+地图) → 课2(IFU) → 课3(EXU) → 课4(LSU/存储/总线) → [中期小测] → 课5(中断/调试/外设) → 课6(FPGA+毕业项目)

### 每课通用流程

1. 摸底：2 个问题，看起点
2. 讲解：大白话 + 代码 + 波形
3. 演示：老师在 VM 上跑给你看
4. 实操：你动手做，卡住先自己试 10 分钟再求助
5. 验收：2-3 道题 + 1 个实操，全过才晋级
6. 复盘：要点写进 notes/，提交仓库

### 验收体系

- 每课：口答/笔答 2-3 题 + 1 个实操必须跑通
- 阶段 B 结束：中期小测
- 课程结束：毕业项目（二选一：EAI 自定义指令上板 / DC 综合报告）

### 教材与资料

- 主教材：《手把手教你设计CPU：RISC-V处理器篇》（仿真流程见第 17.4 节）
- 规范文档：doc/riscv-spec-v2.2.pdf、doc/riscv-privileged-v1.10.pdf、doc/riscv-debug-spec-0.11nov12.pdf
- 源码：~/hkq/e203_mod（rtl / tb / vsim / sirv-e-sdk / fpga）
- 验证手段：riscv-tests 预编译用例、VCS + Verdi、K325t 板卡

### 节奏建议

- 每周约 2 课，全程 3-4 周（可按你的时间自定快慢）
- 每课正课 1.5-2 小时 + 课后练习

## 二、各课内容明细

### 课 0：环境与仿真流程（教案）

**目标**：成为"仿真操作员"，建立 改代码 → 仿真 → 看波形 的循环

**知识要点（讲解提纲）**

1. 四个脚本分工：Makefile（编排）/ bin/run.makefile（工具执行）/ README.md + CMDE.md（说明书）
2. 四命令流水线：
   - make install：拷贝 rtl + tb 到 install/ 并 sed 改名（e200 → e203）
   - make compile：VCS 编译 install/rtl 与 install/tb，生成 simv
   - make run_test：simv +DUMPWAVE +TESTCASE 运行并记录 log
   - make wave：Verdi 加载 ware.fsdb 看波形
3. 关键参数：TESTNAME（名字）vs TESTCASE（完整路径）；DUMPWAVE=0/1；CORE=e203
4. log 阅读：Total cycle_count / valid Instruction Count / TEST_PASS 或 TEST_FAIL
5. 三个易错点：
   - 改 RTL 后必须重新 make install（VCS 编译的是 install/rtl，不是 rtl/）
   - 编译有缓存 compile.flg；强制重编用 rm run/compile.flg 或 make clean
   - 不要用 TESTCASE=$PWD/...（在 Windows 侧 $PWD 会被展开成错误路径），一律用 TESTNAME=
6. 批量回归：regress_prepare / regress_run / regress_collect + find_test_fail.csh 的判定逻辑

**演示记录（已完成）**

- rv32ui-p-add：TEST_PASS（16560 条指令 / 25758 周期）
- rv32mi-p-csr：TEST_PASS（14340 条指令 / 23406 周期）
- rv32ui-p-addi：TEST_PASS（15222 条指令 / 24342 周期）

**实操清单**

- [ ] 亲手跑一遍 make install / make run_test / make wave
- [ ] Verdi 里找到 ifu_pc 信号并看到它变化
- [ ] 跑 rv32mi-p-csr 并把 log 三行关键输出抄下来
- [ ] 体验强制重编：rm run/compile.flg 后重跑 run_test
- [ ] 写 notes/phase0_quickstart.md 并提交仓库

**验收题（答案由老师课堂口头点评）**

1. 为什么第一次必须 make install，之后可以跳过？
2. simv 是在哪个目录生成的？make wave 打开的波形文件叫什么？
3. 改了 RTL 但重新仿真行为没变化，第一步应该做什么？

### 课 1：RISC-V 指令集 + E203 架构地图（教案）

**目标**：建立全局地图——看到任何信号/模块都知道它在整个系统里的位置

**知识要点（讲解提纲）**

1. RISC-V 是什么：开放指令集架构；RV32I 基础指令集；x0 恒为 0；指令格式 R/I/S/B/U/J
2. CSR 与特权级：M/S/U 三级；E203 以 M 模式为主；关键 CSR：mstatus/mepc/mcause/mtvec/mie/mip
3. 异常与中断：同步 vs 异步；trap 流程（跳 mtvec → 保存现场 → 处理 → mret）
4. E203 微架构：2 级流水（ifetch+idecode / execute）vs 经典 5 级；为什么这样设计（低功耗低成本）；单发射、乱序完成（OITF）
5. 模块层级：soc → subsys(top/main/mems/perips) → cpu_top → cpu → core(ifu/exu/lsu) + itcm/dtcm + biu + clint/plic + debug
6. 地址空间表（来自 config.v）：
   - ITCM 0x8000_0000（指令紧耦合存储）
   - DTCM 0x9000_0000（数据紧耦合存储）
   - CLINT 0x0200_0000、PLIC 0x0C00_0000
   - PPI 0x1000_0000、FIO 0xF000_0000
7. config.v / e203_defines.v 读法：宏开关（ECC/EAI/AMO/SHARE_MULDIV/HAS_ITCM/HAS_DTCM）
8. 测试程序怎么进入处理器：tb 用 $readmemh 把 .verilog 文件加载进 ITCM

**实操清单**

- [ ] 跟我一起画出模块层级图（第一版我带你画）
- [ ] 填写地址空间表（每个地址是什么、由谁访问）
- [ ] Verdi 打开全层级，对照层级图找实例
- [ ] 给每个顶层模块写一句话职责
- [ ] 读 config.v 列出一份配置表

**验收题（答案由老师课堂口头点评）**

1. 一条指令在 E203 里被分成哪两个流水阶段？
2. 0x8000_0000 和 0x9000_0000 是什么？在哪个文件里配置？
3. mret 指令是干什么的？它和哪些 CSR 配套使用？

**参考资料**：doc/riscv-spec-v2.2.pdf、doc/riscv-privileged-v1.10.pdf、doc/ 三份中文 PDF、rtl/e203/core/config.v

### 课 2-6：待细化

| 课 | 主题 | 目标 | 核心实操 |
|---|---|---|---|
| 2 | IFU 取指 | 讲清两级流水前半段 | 跟踪 PC + 改分支预测参数 |
| 3 | EXU 执行 | 讲清执行与乱序完成 | 画时序图 + 加自定义 ALU 运算 |
| 4 | LSU / 存储 / ICB | 画出一笔 load 的完整旅程 | 改 DTCM 容量 + 抓 ICB 波形 |
| 5 | 中断 / 调试 / 外设 | 画出中断响应全流程 | 跑中断用例 + demo_gpio |
| 6 | FPGA + 毕业项目 | 在真硬件上验证所学 | 上板跑分 + 自定义指令/综合 |

> 原则：细化一课、讲授一课、验收一课。

## 三、上课纪律

1. 每课开始先回答摸底题，不猜、不会就说"不会"
2. 练习必须亲手做，做不出来先自己试 10 分钟再求助
3. 我讲的任何一段，随时可以要求"再讲一遍"或"换个角度"
4. 每课验收通过后，我会更新本文件"课程进度"表