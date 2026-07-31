# 蜂鸟 E203 学习计划（e203_mod）

环境：VCS O-2018.09-SP2 + Verdi；自制 K325t 板卡；riscv-tests 已预编译
配套书籍：《手把手教你设计CPU：RISC-V处理器篇》（vsim 流程见第 17.4 节）
仓库：~/hkq/e203_mod → https://github.com/whoiskaiii/e203_mod
笔记约定：全部存 notes/ 目录，每阶段一个 md，动手前先 git commit

## 阶段总览

| 阶段 | 时间 | 主题 | 产出 |
|---|---|---|---|
| 0 | 1-2 天 | 环境与工具链 | 独立跑通仿真 + 打开波形 |
| 1 | 第 1 周 | RISC-V 基础 + E203 全局架构 | 模块层级图 |
| 2 | 第 2-3 周 | IFU / EXU 流水线核心 | 时序图 + 一处真实改动 |
| 3 | 第 4 周 | LSU / ITCM-DTCM / ICB 总线 | 存储改动 + 总线时序笔记 |
| 4 | 第 5 周 | 中断 / 调试 / 外设 | 中断响应流程图 |
| 5 | 第 6-8 周 | 综合与 FPGA 实践 | 板卡跑分 + 自定义指令/综合报告 |

## 阶段 0：环境与工具链（1-2 天）

- [ ] 通读 vsim/README.md、vsim/CMDE.md、vsim/Makefile、vsim/bin/run.makefile，理解 install/compile/run_test/wave 与 TESTNAME/DUMPWAVE
- [ ] 跑通 rv32ui-p-add：cd vsim && make install CORE=e203 && make run_test TESTNAME=rv32ui-p-add，看到 TEST_PASS
- [ ] make wave 打开 Verdi（ware.fsdb），练习：打开波形、加信号、搜索、缩放
- [ ] 再跑 rv32mi-p-csr、rv32ui-p-addi，对比两个 log 的指令数与周期数
- [ ] 看懂 Makefile 中 regress_prepare/regress_run/regress_collect 的用法
- [ ] 产出 notes/phase0_quickstart.md（三个命令 + 常见报错 + wave 操作要点）

验收：不看笔记 5 分钟内独立跑通一个用例并打开波形。

## 阶段 1：RISC-V 基础 + E203 全局架构（第 1 周）

- [ ] 泛读 doc/riscv-spec-v2.2.pdf：RV32I 指令格式、寄存器、CSR
- [ ] 对照 doc/riscv-privileged-v1.10.pdf 理解 mstatus/mepc/mcause/mtvec/mie/mip
- [ ] 通读 doc/ 三份中文 PDF（E203 内核/SoC/快速上手简介）
- [ ] 读 rtl/e203/core/config.v 和 e203_defines.v，整理配置表（ITCM/DTCM 基址容量、ECC、AMO、EAI）
- [ ] 从上往下读顶层：soc/e203_soc_top.v → subsys/e203_subsys_top.v → e203_subsys_main.v → e203_subsys_mems.v / e203_subsys_perips.v → core/e203_cpu_top.v → e203_cpu.v → e203_core.v
- [ ] 画出模块层级图，每个模块一句话职责（notes/hierarchy.md）
- [ ] Verdi 打开全层级，对照层级图逐个找到实例
- [ ] 产出 notes/phase1_overview.md：指令/数据/中断三条通路

验收：能说出 0x8000_0000 / 0x9000_0000 是什么，以及取指→译码→执行→访存→写回的模块路径。

## 阶段 2：IFU / EXU（第 2-3 周）

### 第 2 周 IFU

- [ ] e203_ifu.v + e203_ifu_ifetch.v：两级流水划分、PC 计算
- [ ] e203_ifu_minidec.v：mini 译码器如何预判分支/跳转
- [ ] e203_ifu_litebpu.v：轻量分支预测 BTB
- [ ] e203_ifu_ift2icb.v：取指请求转 ICB 事务
- [ ] Verdi 跟踪 rv32ui-p-add 的 PC 序列和一条 beq/jal
- [ ] 动手：改 litebpu 参数或策略，对比周期数

### 第 3 周 EXU

- [ ] e203_exu.v / e203_exu_decode.v / e203_exu_disp.v：译码与派遣
- [ ] e203_exu_alu.v 及 alu_dpath/rglr/bjp/csrctrl/muldiv
- [ ] e203_exu_commit.v / e203_exu_oitf.v / wbck / longpwbck：乱序完成、有序提交
- [ ] e203_exu_regfile.v / e203_exu_branchslv.v / e203_exu_excp.v
- [ ] Verdi 跟踪 addi 与 beq 完整生命周期，画两周期流水时序图
- [ ] 动手：在 alu_dpath 加自定义运算，跑回归 PASS 并 commit

验收：能讲清一条指令从取指到提交每个周期每个模块的职责。

## 阶段 3：LSU / 存储 / ICB（第 4 周）

- [ ] e203_lsu.v / e203_lsu_ctrl.v：访存流水、字节/半字拼接
- [ ] e203_itcm_ctrl.v / e203_dtcm_ctrl.v / e203_srams.v
- [ ] general/sirv_sram_icb_ctrl.v / sirv_sim_ram.v
- [ ] fab/sirv_icb1to2_bus.v / icb1to8 / icb1to16：命令/返回通道、仲裁
- [ ] core/e203_biu.v：总线接口单元
- [ ] 跑 rv32mi-*（load/store）与 rv32ua-*（AMO）回归
- [ ] 动手：改 config.v 的 E203_CFG_DTCM_ADDR_WIDTH（16→18），仿真确认 PASS
- [ ] Verdi 抓一次完整 ICB 读写事务时序
- [ ] 产出 notes/phase3_icb.md

验收：能画出一次 load 从 LSU 到 DTCM 返回的完整信号流。

## 阶段 4：中断 / 调试 / 外设（第 5 周）

- [ ] subsys/e203_subsys_clint.v + perips/sirv_clint_top.v：mtime/msip
- [ ] subsys/e203_subsys_plic.v + perips/sirv_plic_top.v：优先级/阈值
- [ ] core/e203_irq_sync.v + e203_exu_csr.v：mtvec/mepc/mcause/mstatus 切换
- [ ] 跑 tb 中断用例（EXT_IRQ/SFT_IRQ/TMR_IRQ），Verdi 跟踪完整中断流程
- [ ] debug/sirv_jtag_dtm.v / sirv_debug_module.v / debug_csr / debug_rom / debug_ram：JTAG→DTM→DM
- [ ] perips/sirv_uart_top.v / sirv_gpio_top.v：外设寄存器接口
- [ ] sirv-e-sdk/software/demo_gpio 编译并在仿真验证
- [ ] 产出 notes/phase4_irq.md：中断响应流程图

验收：能画出外部中断从触发到 mret 的完整流程。

## 阶段 5：综合与 FPGA（第 6-8 周）

### 第 6 周 复现板卡流程

- [ ] 通读 fpga/my_325t/Makefile、script/board.tcl、constrs/*.xdc
- [ ] Vivado 重建 my_325t 工程：综合→实现→bitstream
- [ ] 下载到 K325t，跑通 hello world（串口输出）
- [ ] 产出板卡环境复现记录

### 第 7 周 性能测试

- [ ] 仿真跑 riscv-tools/fpga_test4sim/dhrystone4sim、coremark4sim
- [ ] FPGA 跑同一套程序，对比仿真与板卡差异（时钟/复位/异步）
- [ ] 产出跑分记录表

### 第 8 周 进阶（二选一）

- [ ] 方案 A：DC 综合 e203（参考 sync_fifo_ai_gen 的 syn/pt），分析时序报告
- [ ] 方案 B：EAI 自定义指令（读 e203_extend_csr.v → 设计 → 改 RTL → 仿真 → 上板），再对比 hbirdv2 的 NICE
- [ ] 产出自定义指令设计文档 或 综合报告笔记

验收：板卡稳定跑 dhrystone/coremark；能说清自定义指令在流水线中的译码与执行。

## 全程纪律

1. 动手实验前先 git commit 存底；一次只改一处；改完跑回归
2. 看不懂代码先开 Verdi 看行为，再回来看代码
3. 每阶段产出 notes/ 笔记；阶段 2 结束前能把 IFU/EXU 讲给别人听