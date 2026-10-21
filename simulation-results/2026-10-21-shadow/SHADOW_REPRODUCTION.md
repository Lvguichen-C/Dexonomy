> 在线交付版：目录、代码和逐步轨迹见[完整结果包](shadow_reproduction_results.zip)。解压后阅读 `shadow-reproduction/SHADOW_REPRODUCTION.md`。

# Shadow 细圆柱抓取复现与连续动力学验证

本次运行 Shadow 手的 `24_Tip_Pinch` 模板，依次执行模板转换、物体接触匹配、MuJoCo 抓取优化和原生六向受力评价，并额外检查开启重力后的离桌抬升。这里交付的是本次实际计算的候选、日志、动画和测量结果。

这是按所提供方案重建的实验。原始圆柱模型、场景文件及修改后的完整源码未随方案提供，因此本次明确记录了资产重建假设；不把其他环境中报告的候选数和成功数当成本次结果。

## 实际结果

<!-- RESULTS:BEGIN -->

本表从实际结果文件生成；详细候选路径和 SHA-256 见 `results-summary.json`。

| 直径 / 高度（mm） | init 保留 | grasp 通过 | 原生六向通过 | grasp 拒绝 |
|---|---:|---:|---:|---|
| 10 / 40 | 50 | 33 | 0 | grasp/contact: 16; grasp/qp: 1 |
| 15 / 60 | 50 | 39 | 3 | grasp/qp: 11 |
| 20 / 80 | 50 | 50 | 35 | 0 |
| 30 / 120 | 50 | 50 | 50 | 0 |
| 40 / 160 | 50 | 49 | 48 | grasp/contact: 1 |
| 合计 | 250 | 221 | 136 | |

init 数为保留的初始提案数；原生六向测试关闭重力。重力开启的抬升验证单独列在动画目录的 metrics.json 中。

<!-- RESULTS:END -->

`init` 是初始提案，`grasp` 是通过原生优化和过滤后保存的候选，`succ_grasp` 才是通过原生六向测试的候选。原生初始化过滤器会补齐保留数量，因此不能把初始化日志里的 “valid” 数字解释为已验证的成功抓取。

逐个成功候选的文件路径、SHA-256、各直径拒绝原因及实际配置，见 [results-summary.json](results_summary.json)。抓取过滤的逐次接触信息和 QP 残差在 audit/native（`audit/native/`）；实际结果文件在 output/tip20_shadow（`output/tip20_shadow/`）。统计脚本按真实文件计数，并检查成功链接是否有效。

最终共保留 250 个初始提案，221 个通过抓取合成过滤，其中 136 个通过原生六向受力评价、85 个未通过；另外 29 个在合成阶段被接触或 QP 条件拒绝。10 mm 圆柱的 33 个合成候选均未通过六向测试，这一失败结果保留在统计与动画中。

最终全体评价前已刷新生成的成功链接；此前 d20 阶段的 35 个成功路径保存在 `previous-success-snapshot.json`。最终 136 个成功链接均来自本轮全候选评价，统计没有把旧链接混入新增结果。

## 动画与额外抬升

[全部 221 个合成候选动画总览](all_candidates.mp4) 包括 136 个原生成功候选和 85 个原生失败候选；逐个动画、候选路径、接触和数值检查可由 动画索引（`animations/all_candidates/index.json`） 查找。总览以 10 fps 播放，共 486 帧、48.6 秒。

总览展示每个候选在重力关闭、无外部受力条件下连续执行原生预抓取→闭合→挤压控制，然后持握 1 秒。画面的 PASS/FAIL 标签来自最终原生六向评价结果；总览片段本身没有重复六向受力测试。221 个片段全部使用真实连续积分，有限状态检查通过，无非预期重置和 MuJoCo 警告。29 个合成阶段被拒绝的提案仅提供初始化及拒绝诊断，没有把它们包装成完整抓取轨迹。

20 mm 圆柱的代表性成功样本为 `24_Tip_Pinch/cylinder_thin_table_20/0_1_grasp.npy`：

- [原生六向受力测试动画](shadow_native_fc.mp4)：分别显示接近、闭合、挤压和每一方向的受力持握。
- [开启重力后的离桌抬升动画](shadow_gravity_lift.mp4)：物体初始放在桌面，闭合后手向上移动 100 mm，随后持握 2 秒。
- [完整数值记录](representative_metrics.json)：包括每方向位移/转角、接触、警告、重置检查和抬升指标。

该样本的独立抬升测得物体升高约 97.26 mm，相对预期抬升位置偏差约 6.42 mm，闭合后的转角变化约 7.60°；结束时有 2 个手物接触、0 个物桌接触。验证在 4 ms 步长下进行，没有非预期时间重置或 MuJoCo 警告。正式动画旁的 `metrics.json` 是最终复核依据。

还分别对 15、30、40 mm 圆柱中按文件名字典序排列的第一个原生成功候选进行同样的抬升验证，每种直径只测试一个，均通过。选择没有按抬升成功与否继续搜索；20 mm 则先测了 `0_0`，记录失败后再测第一个原生成功候选 `0_1`。重渲染相同候选不算新增独立样本。

| 直径（mm） | 候选 | 抬升高度（mm） | 位置偏差（mm） | 转角变化（°） | 额外抬升 |
|---:|---|---:|---:|---:|---|
| 15 | `0_8_grasp` | 99.51 | 4.81 | 14.45 | 通过 |
| 20 | `0_1_grasp` | 97.26 | 6.42 | 7.60 | 通过 |
| 30 | `0_0_grasp` | 98.51 | 5.06 | 6.37 | 通过 |
| 40 | `0_0_grasp` | 99.12 | 4.94 | 3.98 | 通过 |

额外重力抬升共测试 5 个不同候选，4 个通过、1 个失败：较早的 d20 `0_0` 原生受力评价失败，其抬升转角约 17.15°，也超过 15° 限值。d10 的第一个合成候选只记录了原生评价失败，没有运行额外抬升。`representative-validation-summary.json` 保存各样本的选择规则、结果及指标路径。这些补充测试不能解释为全部 136 个原生成功候选均已通过重力抬升。

动画渲染的是连续积分得到的真实状态：每个独立试验开始时初始化一次，随后只发送控制目标并调用 `mj_step`。没有逐帧重设关节位置、物体位置或速度。原生六向动画中，方向切换代表一次新的独立试验，允许且需要从初始状态重新开始。

## 源码、环境和 CPU 兼容

仓库为 `Lvguichen-C/Dexonomy`，本次基线提交：

```text
d561ff9df8545deef7a3f9e7cfb4045ef66f6913
```

本次不修改仓库受 Git 跟踪的源文件。生成资产位于被忽略的 `assets/object/small_cyl`，兼容启动器、诊断和渲染代码位于本报告目录。`results-summary.json` 记录生成统计时的提交号与源文件清洁状态。

环境为 Python 3.10.22、MuJoCo 3.3.3、PyTorch 2.5.1 CPU、Warp 1.8.0、NumPy 2.2.6、trimesh 4.5.3、qpsolvers 4.8.0、Hydra 1.3.2，无 GPU。最终精确版本由统计脚本读取已安装包并保存。

原代码三处 Warp 查询无条件调用 `wp.stream_from_torch(cpu_device)`，Warp 1.8 的该接口只接受 CUDA 流。这里通过 `runtime/sitecustomize.py` 将 **CPU 设备** 对应的流改为 `None`，让 Warp 按其正常 CPU 路径运行；CUDA 设备仍交给原函数。

这一兼容处理不改查询内核、梯度、损失函数、抓取过滤器或 MuJoCo 参数。`cpu_warp_smoke.json` 记录了实际 CPU 网格查询、反向梯度及碰撞检查。通过 `run_cpu.sh` 启动，可自动加载运行时兼容层与已安装的 Python 环境。

## 重建资产与物体参数

原方案给出了 d20 的统一缩放 `0.02` 和中心高度 `0.04 m`，但没有原始网格和完整尺寸。这里采用以原点为中心、直径 1、高度 4 的封闭圆柱，沿圆周离散 64 段；所有尺寸按直径统一缩放。因此五种场景的尺寸如下。

| 圆柱直径（mm） | 高度（mm） | 桌面上中心高度（mm） |
|---:|---:|---:|
| 10 | 40 | 20 |
| 15 | 60 | 30 |
| 20 | 80 | 40 |
| 30 | 120 | 60 |
| 40 | 160 | 80 |

每个桌面场景均把圆柱底部放在 z=0 的桌面。已生成 floating 场景以保证资产目录完整，本次抓取结果统计针对 table_top 场景。所有直径的评价质量都按官方基准归一化为 0.1 kg，而不是保持相同材料密度；这是原生 `eval_data.py` 的处理方式。

`create_assets.py` 完整记录网格、XML、URDF、体积信息和场景的生成过程。`assets-manifest.json` 包含尺寸假设和所有资产的哈希；`assets-validation.json` 验证了 d20 的实际尺寸、有限状态和编译质量约 0.1 kg。

## 实际配置与命令顺序

共同设置为 `hand=shadow`、`tmpl_name=24_Tip_Pinch`、`n_worker=1`、`tmpl_upd_mode=disabled`、`skip_traj=true`、`debug_view=false`、`obj_mass=0.1`。禁用模板反馈用于保持使用原始模板，避免成功候选在后续步骤反馈成新模板；没有删除任何接触、碰撞、关节限位或 QP 过滤条件。

初始化使用原生完整预算：5 个 epoch、每物体 1024 个表面点、8 个平面内旋转、100 次匹配优化，每场景每 epoch 保留 10 个初始提案。单场景 batch size 为 1，随后五场景仍以 batch size 1 顺序处理。随机种子为源码入口设置的 NumPy/PyTorch/Python seed 12。

先完成 d20，再以相同输出目录扩展到五个直径；第二次 init 的 `skip_done=true` 保留已有 d20 结果。不同进程会重置 seed，场景排序/排列、worker 采样与已有结果的跳过都会影响后续随机数消耗，因此不能把“只运行一次五直径”当作逐文件相同的命令。已保存的初始候选可用于直接重评。

以下命令适用于本次云环境；目录结构移到其他机器时，需要同步修改启动器和保存的场景路径。

```bash
REPRO=/workspace/shadow-reproduction

"$REPRO/run_cpu.sh" python "$REPRO/create_assets.py"

"$REPRO/run_cpu.sh" dexrun op=tmpl hand=shadow n_worker=1 debug_name=24_Tip_Pinch \
  exp_name=shadow_repro save_root="$REPRO/output"

# 第一阶段：只生成 d20 的完整 5-epoch 初始候选。
"$REPRO/run_cpu.sh" dexrun op=init hand=shadow exp_name=tip20 \
  tmpl_name=24_Tip_Pinch n_worker=1 tmpl_upd_mode=disabled \
  save_root="$REPRO/output" op.device=cpu \
  op.object.cfg_path=/workspace/Dexonomy/assets/object/small_cyl/scene_cfg/cylinder_thin/table_top/d20.npy \
  op.object.n_cfg=1 op.object.batch_size=1

"$REPRO/run_cpu.sh" python "$REPRO/audit_grasp.py" \
  --audit-dir "$REPRO/audit/native" \
  op=grasp hand=shadow exp_name=tip20 tmpl_name=24_Tip_Pinch \
  n_worker=1 tmpl_upd_mode=disabled save_root="$REPRO/output" log_id=audited

"$REPRO/run_cpu.sh" dexrun op=eval hand=shadow exp_name=tip20 \
  tmpl_name=24_Tip_Pinch n_worker=1 tmpl_upd_mode=disabled \
  save_root="$REPRO/output" log_id=d20full

# 第二阶段：扩展其他直径；已保存的 d20 初始候选保持原样。
"$REPRO/run_cpu.sh" dexrun op=init hand=shadow exp_name=tip20 \
  tmpl_name=24_Tip_Pinch n_worker=1 tmpl_upd_mode=disabled \
  save_root="$REPRO/output" op.device=cpu \
  'op.object.cfg_path=/workspace/Dexonomy/assets/object/small_cyl/scene_cfg/cylinder_thin/table_top/*.npy' \
  op.object.n_cfg=5 op.object.batch_size=1 log_id=all_diameters

# 用新的日志名处理全体候选。
# 本次最终评价前另行保存了 previous-success-snapshot.json，
# 并刷新原生成功链接，以保证最终成功集合完全来自这次评价。
"$REPRO/run_cpu.sh" python "$REPRO/audit_grasp.py" \
  --audit-dir "$REPRO/audit/native" \
  op=grasp hand=shadow exp_name=tip20 tmpl_name=24_Tip_Pinch \
  n_worker=1 tmpl_upd_mode=disabled save_root="$REPRO/output" log_id=all_diameters

"$REPRO/run_cpu.sh" dexrun op=eval hand=shadow exp_name=tip20 \
  tmpl_name=24_Tip_Pinch n_worker=1 tmpl_upd_mode=disabled \
  save_root="$REPRO/output" log_id=all_diameters

"$REPRO/run_cpu.sh" python "$REPRO/summarize_results.py"
```

每次运行的完整 Hydra 配置与命令覆盖项在 `output/tip20_shadow/log/*/.hydra/`。报告中的命令说明实验顺序，精确执行参数以这些存档和 `results-summary.json` 的 `configurations` 为准。

原生抓取优化预算为 20×10 个 2 ms 子步，接触目标间隙 1 mm。最终要求关节不越限、手手/手物不穿透、必需指节在 2 mm 内接触、QP 残差小于 0.001，摩擦参数为 `[0.6,0.02]`。预抓取目标间隙为 45 mm，并检查至少 40 mm 的手物间隙。

这里保留原生提前停止逻辑。方案中提到的 `grasp.early_stop` 是额外修改的开关，本仓库源码没有读取这个键，本次也未增加或关闭它。抓取诊断通过只读记录原方法已经计算的值完成，不增加接触查询或模拟步。

## 原生评价与独立抬升的含义

原生评价使用 4 ms 步长、自由物体和由 mocap 控制的自由根手，关闭全局重力。每方向先从预抓取开始连续执行接近、闭合与挤压，再施加大小为 `0.1×9.8=0.98 N` 的物体质心力。

对 `+X/-X/+Y/-Y/+Z/-Z` 六个方向分别持握 2 秒，每 0.2 秒检查一次相对目标的平移小于 50 mm、旋转小于 15°；六方向全部通过才写入 `succ_grasp`。桌面可在部分方向提供支撑，因此原生通过不等于物体已经离桌。

独立抬升测试另外打开重力 `[0,0,-9.81]`，保留原生预抓取→闭合→挤压的控制，在 1 秒内把手根目标提高 100 mm，再持握 2 秒。通过条件为物体升高至少 80 mm、相对预期抬升位置偏差小于 20 mm、相对闭合时旋转变化小于 15°，最终至少 2 个手物接触且没有物桌接触。

两类测试分别列出，只有有对应 `gravity_lift` 指标的样本才能称为完成独立抬升验证。本次没有把代表性抬升结论扩展成所有成功候选都通过抬升。

## 交付文件与检查方式

- [SHADOW_REPRODUCTION.md](SHADOW_REPRODUCTION.md)：本说明。
- [results-summary.json](results_summary.json)、`RESULTS_TABLE.md`：候选统计、成功样本索引、运行配置与 SHA-256。
- `PIPELINE_AUDIT.md`：原始源码流程、行为差异及评价解释。
- `run_cpu.sh`、runtime（`runtime/`）、`create_assets.py`：CPU 运行与资产重建。
- `audit_grasp.py`、`summarize_results.py`、`render_validate.py`：诊断、统计和真实动力学渲染。
- `output/tip20_shadow/`：本次 init、grasp、成功候选链接和原生运行日志。
- `animations/`：动画、实际轨迹和逐项指标；对应媒体的校验值由汇总脚本生成。

可以直接用已保存的抓取候选重新评价，或从初始化提案开始重跑优化。跨机器重评前应修正候选中的 `scene_path` 与场景资产根路径。`results-summary.json` 中的成功候选 SHA-256 可检查文件传输是否完整。
