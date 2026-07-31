# 第 0 课：环境与仿真流程（课件）

配套仓库：~/hkq/e203_mod（vsim 目录）
配套书：《手把手教你设计CPU：RISC-V处理器篇》第 17.4 节

## 1. 本课目标

成为"仿真操作员"：能独立完成 改代码 → 仿真 → 看波形 → 看结果 的闭环。
以后每一课的练习都靠这套流程验证。

## 2. 四个脚本的分工

| 文件 | 角色 |
|---|---|
| vsim/Makefile | 顶层编排：变量定义、目录准备、选择测试、批量回归 |
| vsim/bin/run.makefile | 工具执行：VCS 编译/运行、Verdi 开波形（换工具只改这里） |
| vsim/README.md | 官方说明：书 17.4 节、demo_gpio/dhrystone/coremark 用法 |
| vsim/CMDE.md | 四命令速查 |

设计核心：Makefile 会在 run/ 目录创建指向 bin/run.makefile 的符号链接，
再用 make -C run 转发执行，所以"项目逻辑"和"EDA 工具逻辑"分离。

## 3. 四个命令

### make install CORE=e203

1. 创建 install/tb、install/rtl
2. 拷贝 ../tb/tb_top.v 到 install/tb，并用 sed 把里面的 e200 替换成 e203（按 CORE）
3. 拷贝 ../rtl/e203 到 install/rtl

关键结论：VCS 编译的是 install/rtl 和 install/tb，不是 rtl/ 和 tb/ 本身。

### make compile

调用 VCS，命令与关键选项：

| 选项 | 作用 |
|---|---|
| -timescale=1ns/1ns | 时间单位/精度 |
| -full64 | 64 位编译 |
| -R | 编译后立即运行 |
| +v2k | 支持 Verilog-2001 |
| -sverilog | 支持 SystemVerilog |
| -debug_access+all | 允许 Verdi 读取信号/波形 |
| -l vcs.log | 编译日志 |
| +incdir+.../core/+.../perips/ | include 搜索路径（e203_defines.v 靠它找到） |

增量编译：compile.flg 文件比所有 RTL/TB 都新时跳过编译；
强制重编用 rm run/compile.flg（或 make clean 后重来）。

### make run_test TESTNAME=rv32ui-p-add

流程：
1. 先 compile
2. 创建 run/rv32ui-p-add/ 目录
3. 进入目录执行：../simv +DUMPWAVE=1 +TESTCASE=<路径> |& tee rv32ui-p-add.log

plusarg 由 tb_top.v 里的 $value$plusargs 读取：
- +TESTCASE：决定用 $readmemh 加载哪个 .verilog 测试程序进 ITCM
- +DUMPWAVE=1：tb 执行 $fsdbDumpfile("ware.fsdb") 导出波形

### make wave

用与编译相同的 RTL/TB 文件直接拉起 Verdi（-top tb_top），
仿真跑完后自动加载 ware.fsdb。

## 4. TESTNAME vs TESTCASE（易错点）

- TESTNAME：只给名字（rv32ui-p-add），路径由 Makefile 自动拼 —— 推荐，日常都用它
- TESTCASE：给完整路径。不要写 TESTCASE=$PWD/... 再通过 Windows 侧的 shell 传参，
  因为 $PWD 会被本地展开成 Windows 路径（如 D:0_Files...），导致测试程序加载失败

## 5. log 怎么看

关键三行：

    Total cycle_count value:     24388
    The valid Instruction Count: 15222
    TEST_PASS

- cycle_count：仿真总周期数（越短越快）
- Instruction Count：执行的有效指令数
- TEST_PASS / TEST_FAIL：结果

失败时常见现象：
- Cannot open file 'D:0_Files...'：TESTCASE 路径错误（用了 $PWD）
- Fatal ... CHECK_THE_X_VALUE：ITCM 出现 X（通常也是测试程序没加载进去）

## 6. 常见问题排查表

| 现象 | 处理 |
|---|---|
| 改了 RTL 但仿真没变化 | 重新 make install（同步到 install/rtl），再 rm run/compile.flg 强制重编 |
| 报错 Cannot open file（Windows 路径） | 改用 TESTNAME= 传参 |
| 仿真 X 值 Fatal | 检查测试程序是否成功加载进 ITCM |
| SSH 报 Bad permissions on config | 连接时加 -F none 跳过配置文件 |
| 想彻底重来 | make clean（只删 run/ 和 install/，不动源码）|

## 7. 批量回归

    make regress_prepare   # 编译一次 + 清旧日志
    make regress_run       # 逐个跑所有用例（DUMPWAVE=0 加速）
    make regress_collect   # 汇总 PASS/FAIL/NOT_FINISHED

- SELF_TESTS 自动收集 generated/ 下的 rv32*.dump，并按内核裁剪
  （e203 只跑 rv32uc/um/ua/ui/mi，浮点系列只在 e205f+ 出现）
- find_test_fail.csh 判定：log 里恰好 1 条 "Test Result Summary" 且无 TEST_FAIL → PASS

## 8. 演示记录（课堂已跑）

| 用例 | 指令数 | 周期数 | 结果 |
|---|---|---|---|
| rv32ui-p-add | 16560 | 25758 | TEST_PASS |
| rv32mi-p-csr | 14340 | 23406 | TEST_PASS |
| rv32ui-p-addi | 15222 | 24342 | TEST_PASS |

## 9. 实操清单

- [ ] 亲手走一遍 make install / make run_test / make wave
- [ ] Verdi 里找到 ifu_pc 信号并看到它递增变化
- [ ] 跑 rv32mi-p-csr，把 log 三行关键输出抄进笔记
- [ ] 体验强制重编：rm run/compile.flg 后重跑 run_test
- [ ] 写 teaching/notes/phase0_quickstart.md 并提交仓库

## 10. 验收题（先作答，再看参考答案）

1. 为什么第一次必须 make install，之后可以跳过？
2. simv 是在哪个目录生成的？make wave 打开的波形文件叫什么？
3. 你改了 RTL 但重新仿真行为没变化，第一步应该做什么？

### 参考答案（做完验收再看）

1. install 会生成 install/rtl 与 install/tb，并做 e200→e203 的改名；
   VCS 编译的是 install/ 下的文件，所以首次必须 install。
   之后若 rtl/tb 源文件没改可以跳过；但一旦改了 rtl，必须重新 make install。
2. simv 生成在 vsim/run/ 目录（compile 是在 run 目录下执行的）；
   make wave 打开 Verdi 并加载 ware.fsdb（由 tb 的 $fsdbDumpvars 生成）。
3. 第一步：重新 make install，把修改同步到 install/rtl；
   然后 rm run/compile.flg（或 make clean）强制 VCS 重新编译。