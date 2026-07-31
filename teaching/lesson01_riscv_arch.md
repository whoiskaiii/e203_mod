# 第 1 课：RISC-V 指令集 + E203 架构地图（课件）

配套资料：doc/riscv-spec-v2.2.pdf、doc/riscv-privileged-v1.10.pdf、doc/ 三份中文 PDF
代码入口：rtl/e203/core/config.v、rtl/e203/soc/e203_soc_top.v

## 1. 本课目标

建立全局地图：看到任何模块/信号，都能说出它在整个系统里的位置和职责。

## 2. RISC-V 是什么

- 开放指令集架构（ISA），不收授权费；指令集是"软件与硬件的契约"
- 模块化设计：RV32I 是基础整数指令集，M/A/F/D/C 是可选扩展
- E203 支持：RV32IMAC（整数 + 乘除 + 原子 + 压缩指令）
- 32 个通用寄存器 x0~x31；x0 恒为 0（写它等于丢弃）
- 指令编码固定 32 位（压缩指令 RV32C 是 16 位）
- 六种基本指令格式：R（寄存器）/ I（立即数）/ S（存储）/ B（分支）/ U（上立即数）/ J（跳转）

## 3. CSR 与特权级

- 特权级：M（机器）/ S（监督）/ U（用户），数字越大权限越高
- E203 以 M 模式为主（配套软件也基本只跑 M 模式）
- CSR = 控制状态寄存器，用 csrrw/csrrs/csrrc 等指令读写
- 关键 CSR：

| CSR | 作用 |
|---|---|
| mstatus | 全局状态：MIE 中断使能、MPP 之前特权级等 |
| mtvec | 异常/中断入口地址（trap 后跳这里）|
| mepc | 被中断/异常打断的 PC |
| mcause | 中断/异常原因编号 |
| mie / mip | 中断使能 / 中断挂起（M 模式）|

## 4. 异常与中断

- 异常（Exception）：同步，指令本身引起（非法指令、访存错误、ebreak）
- 中断（Interrupt）：异步，外部事件引起（定时器、软件、外部）
- trap 统一流程：
  1. 根据 mtvec 跳到处理入口
  2. 硬件保存现场：mepc = 当前 PC，mcause = 原因，mstatus 保存旧状态
  3. 软件处理
  4. mret 返回：恢复 mepc/mstatus
- E203 里相关模块：e203_irq_sync、CLINT（定时器/软件中断）、PLIC（外部中断）

## 5. E203 微架构总览

- 2 级流水：IFU 级（ifetch + idecode）+ EXU 级（execute）
- 对比经典 5 级（取指/译码/执行/访存/写回）：
  E203 用面积和频率换低功耗低成本，适合 MCU 定位
- 单发射（每周期最多派发 1 条）、顺序派发、乱序完成（OITF 保证有序提交）
- 内核地图（core 层）：
  ifu（取指）→ exu（执行，含 alu/乘除/CSR/分支）→ lsu（访存）
  + itcm_ctrl / dtcm_ctrl（紧耦合存储）
  + biu（总线接口）/ clk_ctrl / reset_ctrl / srams

## 6. 模块层级（从上到下）

    e203_soc_top（SoC 顶层，含时钟/复位/IO）
      └─ e203_subsys_top
          ├─ e203_subsys_main（CPU 子系统主体）
          │    └─ e203_cpu_top → e203_cpu → e203_core（ifu/exu/lsu/itcm/dtcm/biu）
          ├─ e203_subsys_mems（存储器：ITCM/DTCM 底层 RAM）
          ├─ e203_subsys_perips（外设：UART/GPIO/SPI/I2C/PWM...）
          ├─ e203_subsys_clint（定时器/软件中断）
          └─ e203_subsys_plic（外部中断控制器）
      + debug（JTAG 调试：sirv_jtag_dtm / sirv_debug_module）
      + mems/sirv_mrom（启动 ROM）

## 7. 地址空间表（来源：core/config.v）

| 区域 | 基址 | 用途 |
|---|---|---|
| ITCM | 0x8000_0000 | 指令紧耦合存储（CPU 取指）|
| DTCM | 0x9000_0000 | 数据紧耦合存储（CPU 访存）|
| CLINT | 0x0200_0000 | 定时器 mtime、软件中断 msip |
| PLIC | 0x0C00_0000 | 外部中断源管理 |
| PPI | 0x1000_0000 | 外设寄存器区 |
| FIO | 0xF000_0000 | 快速 IO 区 |

记忆技巧：ITCM 在前、DTCM 在后（0x80/0x90），中断控制 0x02/0x0C，外设 0x10。

## 8. config.v / e203_defines.v 怎么读

- config.v 是"配置总开关"，常见宏：
  E203_CFG_HAS_ITCM / HAS_DTCM、ITCM_ADDR_WIDTH / DTCM_ADDR_WIDTH、
  HAS_ECC、HAS_EAI、SUPPORT_AMO、SUPPORT_SHARE_MULDIV、
  SUPPORT_MCYCLE_MINSTRET、DEBUG_HAS_JTAG、IRQ_NEED_SYNC
- e203_defines.v 由 config.v 推导出具体参数（地址、宽度、宏别名）
- 改配置 = 编辑 config.v（加/取消注释），但改动必须重新 make install + 编译验证

## 9. 测试程序怎么进入处理器

- tb/tb_top.v 用 $readmemh({testcase, ".verilog"}, itcm_mem) 把测试程序
  （riscv-tests/isa/generated/rv32ui-p-add.verilog）加载进 ITCM
- 所以每个用例 = 一个 .verilog 十六进制文件，直接当内存初值

## 10. 实操清单

- [ ] 跟我一起画出模块层级图（第一版老师带画，之后自己补全）
- [ ] 填写地址空间表（每个地址是什么、被谁访问）
- [ ] Verdi 打开全层级，对照层级图逐个找到实例
- [ ] 给每个顶层模块写一句话职责（写进 notes/phase1_arch.md）
- [ ] 读 config.v，列出一份配置表（宏名 + 含义 + 当前值）

## 11. 验收题（先作答，再看参考答案）

1. 一条指令在 E203 里被分成哪两个流水阶段？
2. 0x8000_0000 和 0x9000_0000 是什么？在哪个文件里配置？
3. mret 指令是干什么的？它和哪些 CSR 配套使用？

### 参考答案（做完验收再看）

1. 取指阶段（IFU：ifetch + idecode）和执行阶段（EXU：execute），
   即"两级流水"，访存和写回都并进执行级。
2. ITCM（指令紧耦合存储）和 DTCM（数据紧耦合存储），
   在 rtl/e203/core/config.v 里通过 E203_CFG_ITCM_ADDR_BASE /
   E203_CFG_DTCM_ADDR_BASE 配置。
3. mret 是从 M 模式异常/中断处理程序返回的指令：
   恢复 mepc（回到被打断的 PC）和 mstatus（恢复中断使能等），
   配套 CSR 是 mepc / mcause / mtvec / mstatus。