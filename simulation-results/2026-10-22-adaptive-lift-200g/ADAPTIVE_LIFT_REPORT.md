> 在线交付版。完整代码、资产、全部指标与轨迹见以下结果包；解压后阅读 `shadow-lift-sweep-200g/ADAPTIVE_LIFT_REPORT.md`。

- [adaptive_lift_results.zip](adaptive_lift_results.zip)

# Shadow 200 g 圆柱抓取、抬升持握与释放复测

本轮覆盖直径 1、5、10、20、30、40、50、60、70、80、90、100 mm；每个圆柱高度为直径的 4 倍、质量为 200 g。沿用已经冻结的 314 个候选控制目标与同一控制逻辑，仅将自由物体质量从 100 g 改为 200 g，重新执行全部动力学仿真，没有新增合成、优化或调参。

原 100 g 的成功标签、轨迹和展示样本不能作为 200 g 的结果；本报告只以新 200 g 测试指标和新轨迹为依据。释放退离方向仍由相同策略根据实际持握物体位姿计算，因此可随负载响应改变，不要求所有阶段逐帧控制输出完全相同。

## 新 200 g 实际结果

| 直径 / 高度 mm | 继承初始提案 | 冻结控制候选 | 新测候选 | 抬升持握通过 | 释放通过 | 全流程通过 |
|---|---:|---:|---:|---:|---:|---:|
| 1 / 4 | 30 | 1 | 1 | 0 | 0 | 0 |
| 5 / 20 | 30 | 26 | 26 | 0 | 2 | 0 |
| 10 / 40 | 70 | 34 | 34 | 2 | 3 | 2 |
| 20 / 80 | 70 | 57 | 57 | 7 | 7 | 7 |
| 30 / 120 | 70 | 60 | 60 | 28 | 60 | 28 |
| 40 / 160 | 70 | 52 | 52 | 47 | 49 | 47 |
| 50 / 200 | 30 | 15 | 15 | 14 | 15 | 14 |
| 60 / 240 | 30 | 19 | 19 | 15 | 19 | 15 |
| 70 / 280 | 30 | 10 | 10 | 1 | 10 | 1 |
| 80 / 320 | 30 | 20 | 20 | 11 | 20 | 11 |
| 90 / 360 | 30 | 10 | 10 | 0 | 10 | 0 |
| 100 / 400 | 30 | 10 | 10 | 0 | 10 | 0 |
| 合计 | 520 | 314 | 314 | 125 | 205 | 125 |

原评价器在本轮 200 g 下的抬升持握通过数为 125，全流程通过数为 125；附加提升过程采样门后，最终对应为 125 和 125。原始标志保存在 base_evaluator_outcomes 中。

每个尺寸的成功数描述这批冻结控制在 200 g 下的表现，不是重新优化后 Shadow 的最大 200 g 操作能力。初始提案数只是控制来源的历史文件数，不表示本轮重新优化；旧候选生成预算本来不均，不能当成各直径统一搜索预算下的统计成功率。

## 动作流程与物理判据

全部测试开启重力，实际编译物体质量应为 0.2 kg。动作流程：接近 2.0 s → 按选定抓型闭合和夹紧 → 夹紧目标受力保持 1.0 s → 在 5.0 s 内平稳提升手根目标 100 mm → 到位持握 5.0 s → 松指 1.5 s → 退离 1.0 s → 释放后观察 3.0 s。

100 mm 是手根控制目标，物体的实际升高另行测量。到位后完整 5 s 持握阶段，每个 4 ms 物理步（250 Hz）都检查：物体实际升高 95–105 mm、相对预期位置误差及持握平移漂移小于 20 mm、相应旋转误差和漂移小于 15°、不接触桌面、结束时至少 2 个手物接触点、状态有限且无警告或意外时间重置。

5 s 提升过程另按保存的实际状态样本复核，通常 10 Hz 并含准确阶段边界：三维 smoothstep 位置跟踪误差不超过 20 mm、相对闭合姿态转角不超过 15°、从已达到高度向下回滑不超过 2 mm、每个提升样本至少 2 个手物接触点、样本及阶段覆盖完整。接触点数不等于独立手指数，刚离桌前可仍有桌面接触。

提升阶段的采样复核不能证明未保存的 4 ms 子步也满足条件；它与到位后 250 Hz 逐步持握检查不同。最终抬升持握与全流程标志必须同时通过原评价器和提升采样门。释放通过独立要求手物接触清零、物体实际落回桌面且相对开始松指时下落至少 20 mm；完整流程要求两项都通过。

夹紧受力保持阶段记录实际法向接触力；物体在离桌前仍可能由桌面承重，未人为添加侧向外力。200 g 重力载荷约 1.962 N；原有 squeeze 控制没有为加倍负载重新调强。

## 失败类型

本轮 189 个未通过结果中，77 个仅实际升高不足 95 mm，其余提升、持握和释放条件均通过；2 个存在朝向偏差，其中一个兼高度不足。另有 11 个夹持末接触不足、97 个有初始接触但未有效抬起、2 个先上升后失去接触并回落。

5、90、100 mm 展示样本实际升高分别约 92.80、94.65、94.82 mm，因此未通过高度判据；1 mm 展示样本未离桌。失败不能全部解释为掉落，也不能据此断言其他控制方案必然无法完成。

- [逐尺寸失败分析](FAILURE_ANALYSIS.md)
- [全部候选的分类与证据](FAILURE_ANALYSIS.json)

## 同一候选的 100 g / 200 g 配对比较

配对检查完成：`True`。100 g 与 200 g 最终抬升持握通过数分别为 266 / 125，全流程通过数分别为 266 / 125。

候选文件按 SHA-256 和相对候选身份一一配对，核对接近至持握结束的控制目标、除质量外的协议及实际模型质量。释放退离的反馈响应不要求逐帧恒同。逐尺寸通过→失败、失败→通过和逐候选实际升高、持握余量变化见下列比较文件。代表动画会根据各负载的新结果重新选择，可能更换候选，但整个候选控制集合不变。

- [负载配对比较说明](LOAD_COMPARISON.md)
- [全部配对数值](load-comparison.json)
- [控制和模型配置独立审计](controlled-comparison-verification.json)
- [新 200 g 实际模型质量核验](MASS_VALIDATION.md)
- [模型质量数值](mass-validation.json)

## 候选来源与复现边界

本轮 314 个控制和 520 个历史初始化文件逐字节继承自 100 g 实验，包括 scene_path 和全部数值数组；新仿真在编译模型前显式使用 --mass 0.2 重算密度。没有复制旧评价标签或轨迹。几何资产保持原样；统一质量而非统一材料密度，故不同尺寸的密度不同。

几何自适应模板选择与合成属于 100 g 控制来源的历史过程，本轮没有再次执行。原 adaptive-plan.json 中的 0.1 kg 字段也仅描述历史来源，不是本轮实际质量。setup-provenance.json 记录 314 个控制及初始化的来源 SHA；新的 mass-validation.json 与运行时模型文件验证实际质量为 0.2 kg。

在同一控制集合上复测可以比较质量变化的影响，但不能据此断言重新规划、重新调参或采用其他模板时某一尺寸必然无法完成。全部失败仍保留于结果文件。

## 新动画与完整数据

- [12 种直径 200 g 总览](all_12_diameters.mp4)
- [新 200 g 完整数值汇总](adaptive_lift_summary.json)
- [直径 1 mm 的新 200 g 动画](d001_full_sequence.mp4)：未通过（真实试验）
- [直径 5 mm 的新 200 g 动画](d005_full_sequence.mp4)：未通过（真实试验）
- [直径 10 mm 的新 200 g 动画](d010_full_sequence.mp4)：全流程通过
- [直径 20 mm 的新 200 g 动画](d020_full_sequence.mp4)：全流程通过
- [直径 30 mm 的新 200 g 动画](d030_full_sequence.mp4)：全流程通过
- [直径 40 mm 的新 200 g 动画](d040_full_sequence.mp4)：全流程通过
- [直径 50 mm 的新 200 g 动画](d050_full_sequence.mp4)：全流程通过
- [直径 60 mm 的新 200 g 动画](d060_full_sequence.mp4)：全流程通过
- [直径 70 mm 的新 200 g 动画](d070_full_sequence.mp4)：全流程通过
- [直径 80 mm 的新 200 g 动画](d080_full_sequence.mp4)：全流程通过
- [直径 90 mm 的新 200 g 动画](d090_full_sequence.mp4)：未通过（真实试验）
- [直径 100 mm 的新 200 g 动画](d100_full_sequence.mp4)：未通过（真实试验）

本轮每个尺寸都有实际物理轨迹，动画展示记录的真实状态；回放不重新积分，也不以画面判定成功。全部候选指标与失败、完整轨迹、模型、候选控制和复现代码保留于结果包。所有视频按新 200 g 指标重新选择，未复用旧 100 g 动画。

## 协议审计、环境与采样范围

提升采样审计覆盖 314 个新候选，应用与最终计数核对为 `True`。资产和协议依赖的指纹用于检查缓存失效；该指纹不能反向证明更早未记录的状态。

- [新协议审查](PROTOCOL_REVIEW.md)
- [新提升采样审计](motion-protocol-summary.json)

基线源码：`d561ff9df8545deef7a3f9e7cfb4045ef66f6913`；Git 跟踪源码未改动：`True`。

```json
{
  "python": "3.10.22",
  "mujoco": "3.3.3",
  "torch": "2.5.1+cpu",
  "warp-lang": "1.8.0",
  "numpy": "2.2.6",
  "trimesh": "4.5.3",
  "qpsolvers": "4.8.0"
}
```

## 从已打包控制复现

本包已含同一组控制和冻结评价器，不需要重新合成抓取。新机器先克隆 Lvguichen-C/Dexonomy，检出本报告列出的基线提交，初始化 third_party/mujoco_menagerie 子模块，并按上面的环境版本安装 CPU 依赖。该基线提交记录了子模块版本；MuJoCo、Torch、Warp、NumPy 的版本应与报告一致。

以下命令面向当前云环境；解压到其他目录时先重定位候选 NPY 中绝对 scene_path 和场景引用的网格路径，并相应修改启动器、代码中的 /workspace 路径及已安装环境。新包的 setup-provenance.json 保存原 SHA 和来源，重定位后的文件应重新建立清单，不应冒称仍与原始包逐字节相同。

```bash
R=/workspace/shadow-lift-sweep-200g
CPU=/workspace/shadow-reproduction/run_cpu.sh
EVALUATOR=$R/evaluate_lifts.py
# 当前环境保留同一冻结源文件；解压包已将其放入新目录。
if [ ! -f "$EVALUATOR" ]; then EVALUATOR=/workspace/shadow-lift-sweep/evaluate_lifts.py; fi
"$CPU" python "$EVALUATOR" --grasp-root "$R/output/diameter_sweep_200g_shadow/grasp_data" --outdir "$R/lift-evaluation" --mass 0.2 --workers 2
"$CPU" python "$R/verify_controlled_comparison.py" --root "$R" --require-complete
"$CPU" python "$R/validate_motion_protocol.py" --root "$R" --evaluation-root "$R/lift-evaluation" --apply
"$CPU" python "$R/finalize_runtime_models.py" --root "$R" --evaluation-root "$R/lift-evaluation" --mass 0.2 --apply
"$CPU" python "$R/render_sweep.py" --evaluation-root "$R/lift-evaluation" --outdir "$R/animations" --fps 10 --reuse
"$CPU" python "$R/finalize_montage.py"
"$CPU" python "$R/compare_loads.py" --baseline-root /workspace/shadow-lift-sweep/lift-evaluation --new-root "$R/lift-evaluation" --outdir "$R"
"$CPU" python "$R/summarize_sweep.py" --evaluation-root "$R/lift-evaluation"
```

协议应用必须等最后一个评价进程结束后执行；否则新写入的指标会缺少提升门。总览后处理器只组合已有单项视频，固定 10 fps 和完整帧数，不修改物理轨迹或单项视频。包内评价目录使用 lift-evaluation，与本环境一致；100 g 基线旧包若使用 evaluation 名称，应先将该目录重命名为 lift-evaluation，并重定位其中指标记录的绝对输入和轨迹路径。运行时模型导出、接触几何回放和视频都不能替代实际 200 g 仿真。运行顺序与参数解释另见包内 PROTOCOL_REVIEW.md、MASS_VALIDATION.md、ANIMATION_METHOD.md。

100 g 对照基线：[已发布的原实验 Release](https://github.com/Lvguichen-C/Dexonomy/releases/tag/shadow-adaptive-lift-sweep-20261022)。配对比较需要原基线的完整指标和轨迹；新包不把旧物理轨迹混作 200 g 输出。

## 本轮 200 g 展示样本的接触数值局限

仅使用新 200 g 轨迹的接触复核，共 12 个展示样本，其中 8 个全流程通过。通过样本提升阶段的最大手物重叠为 0.737 mm，持握阶段为 0.714 mm。

MuJoCo 采用软接触；这些数值不是严格刚性几何的无穿透证明。接触几何复核采用保存状态采样，不构成新的动力学执行或未保存子步观测。完整分阶段数值见本轮 contact-diagnostics.json；没有复制旧 100 g 的接触数字。
