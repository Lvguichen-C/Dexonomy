# 200 g 质量设置与模型检查

本次使用与 100 g 实验逐字节相同的 `evaluate_lifts.py`，SHA256 为：

```text
164fd4e4eab1472a7cf4eb92bf5abf88747f78c1df58b19a0667ce8ce75f5d50
```

通用测试器的默认质量仍为 0.1 kg，因此本次必须显式传入 **`--mass 0.2`**。该参数通过 `prepare_scene(input, 0.2)` 重算物体密度，并实际进入 MuJoCo 编译模型；不仅改变报告文字。

```bash
/workspace/shadow-reproduction/run_cpu.sh python \
  /workspace/shadow-lift-sweep-200g/evaluate_lifts.py \
  --grasp-root /workspace/shadow-lift-sweep-200g/output/diameter_sweep_200g_shadow/grasp_data \
  --outdir /workspace/shadow-lift-sweep-200g/lift-evaluation \
  --mass 0.2 --workers 2 --trace-fps 10 --resume
```

每个候选的 `metrics.json` 同时记录 `config.mass_kg`、编译后的 `object_mass_kg` 和实际重力载荷 `object_gravitational_weight_N`。200 g 的重力载荷约为 1.962 N。几何、抓取控制目标、执行器增益、力上限、摩擦与求解器设置均沿用原实验。

`validate_mass_models.py` 已分别在独立子进程中编译 12 种直径的模型。结果见 `mass-validation.json`：

- 全部编译质量介于 0.199999989 kg 和 0.200000020 kg 之间。
- 全部开启重力 `[0,0,-9.81]`，步长 4 ms。
- 全部目标物体有自由关节，没有物体焊接约束，主惯性均有限且严格为正。
- 对 20 mm 的 100 g/200 g 模型做直接比较：执行器 gain/bias、控制范围、力范围、手根焊接 `solimp`/`solref` 均逐元素完全相同。

另对原 20 mm `24_Tip_Pinch/0_1_grasp` 执行了一次完整 200 g 诊断，结果在 `mass-validation/full-trial/`。该样本未通过：物体实际升高约 -0.210 mm，留在/落回桌面，结束无手物接触；数值有限、零警告、零时间重置。该诊断不计入正式配对候选总数，也没有据此调大夹持力或修改控制目标。

## 模型 XML 归档

`model.xml` 的初始构造快照早于运行时开启重力，正式评价进程全部结束后执行：

```bash
/workspace/shadow-reproduction/run_cpu.sh python \
  /workspace/shadow-lift-sweep-200g/finalize_runtime_models.py --mass 0.2 --apply
```

该工具保留原始 `construction_model.xml`，仅把真实运行时的重力设置写入归档 `model.xml`，随后重新编译并检查 200 g、4 ms 和重力开启。它不重新执行、修改或重新分类任何物理轨迹。此版本的根路径和质量默认值均指向本次 200 g 实验。
