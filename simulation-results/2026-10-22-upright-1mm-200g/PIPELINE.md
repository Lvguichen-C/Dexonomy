# Shadow 1 mm 竖直圆柱连续抓取管线

本轮执行的固定顺序：**远离静置 → 垂直接近 → 闭合夹紧 → 受力保持 → 平稳提升 → 到位持握 → 释放退离**。结果见 `experiment-summary.json`，不能把完整执行流程等同于抓取通过。

## 输入与环境

- 圆柱直径 **1 mm**、高度 **4 mm**、总质量 **200 g**，初始中心 `[0,0,0.002] m`、初始轴竖直；保持用户规定的固定质量，未替换为更长杆或加配重的组合体。
- MuJoCo **3.3.3**，物理步长 **1 ms**，重力 `[0,0,-9.81] m/s²`；圆柱始终为自由关节，物体没有 weld、弹簧、吸附、额外外力、跟随手运动的位姿回写。
- Shadow 来源：`Lvguichen-C/Dexonomy` 提交 `d561ff9df8545deef7a3f9e7cfb4045ef66f6913`，手模型 `assets/hand/shadow/right.xml`。
- 原始评价器与旧结果保持冻结。本轮通过独立 `run_upright_trials.py` 显式适配起点、时标及诊断；`controller_profiles.py` 是单列的控制参数对照。

## 连续执行阶段

| 阶段 | 目标与时长 | 实际检查 |
|---|---|---|
| 远离静置 `start_pause` | 原预抓取根目标上方 **100 mm**，静置 **0.5 s** | 实测全部碰撞几何的最小手物间距；本轮均约 **115 mm**，无手物接触 |
| 接近 `approach` | **2 s** 竖直平滑下降至预抓取目标 | 连续控制、连续积分，不重置物体；本轮该阶段没有手物接触 |
| 闭合夹紧 `approach_close_squeeze` | 预抓取、抓取、压紧目标依序执行 | 原时标 **0.7 s**；另设 **1.4 / 2.1 s** 慢闭合对照 |
| 受力保持 `loaded_hold` | **1 s**，重力持续生效 | 分别记录手物夹紧力、桌面/托座支撑力，不能混算 |
| 平稳提升 `lift` | 手根目标升 **100 mm / 5 s** | 记录实际圆柱升高；不能用手的命令升程替代物体升程 |
| 到位持握 `postlift_hold` | **5 s** | 每个物理步检查高度、漂移、接触、支撑；1 ms 共 **5000 步** |
| 释放 `release_open/retreat/settle` | 松指 **1.5 s**、退离 **1 s**、观察 **3 s** | 完整保留真实末态；未曾抬起的样本不能因手离开而获得释放成功 |

阶段内部只有手部目标控制与 `mj_step`。初次初始化后，禁止 `reset_qpos`、重置时钟或回写物体位姿。源输入中的 20/50 mm 接近偏移不再作为实际执行值：本轮所有正式试验统一使用上方 100 mm，并在 provenance 中明确覆盖。

数值步长从旧 4 ms 改为 1 ms 时，原生闭合使用 `step=5, substep=20`，保持每控制段 0.1 s；慢闭合对应 `substep=40/60`。不能沿用 `substep=5` 而使真实闭合加速四倍。

## 支撑条件与有限控制对照

基线为七个唯一控制变体（中心、X/Y 各±0.15 mm、闭合×2/×3），分别运行无托与浅托，共 **14 次**。仅改变接近偏移的重复来源提案未计数。

浅托是固定 world 几何：孔径至少 **1.2 mm**、外径 **5 mm**、壁高 **0.75 mm**、底厚 **0.2 mm**。底面支撑顶为 z=0，桌面为 z=-0.2 mm，杆初始位置不变。32 段环壁向上开放，fixture priority=1、实际 condim=3。没有把托座附着到手，也没有锁住杆。独立的施力拔出探针仅检验托座可脱离，不属于抓取成功试验。

中心提案另有四次控制敏感性试验：两种支撑条件各测试手根 weld `solref=[.005,1]`，以及该改动加位置执行器增益×10。原基线 weld 为 `[.02,1]`；执行器力矩限幅、杆参数和接触材质不变。它们单列在 `controller_ablation/`，不会替换原基线。

## 判定与数据

目标为实际圆柱升高 **100±5 mm**，到位后连续持握 5 s、全程无桌面/托座支撑，并通过原位置/朝向/接触与数值稳定性门；之后实际释放。每一步记录真实手物穿透，并对照杆半径 **0.5 mm**。大幅软接触重叠不能充当有效夹持证据。

每次保存 `metrics.json`、权威编译模型 `model.mjb`、参考 `model.xml`、**25 Hz** 完整状态轨迹 `trace.npz`、**1000 Hz** 逐步接触诊断 `diagnostics.npz`。动画以 **10 fps** 显示保存状态，仅 `mj_forward` 回放，无物体运动补造、状态插值或几何放大；近景只是相机放大。

全套复跑（原工作区）：

```bash
source /workspace/dexonomy-onboarding/activate.sh
export OMP_NUM_THREADS=1 OPENBLAS_NUM_THREADS=1 MKL_NUM_THREADS=1 LP_NUM_THREADS=1 MUJOCO_GL=egl
export PYTHONPATH=/workspace/shadow-reproduction/runtime
python /workspace/shadow-lift-sweep-200g/upright-1mm-20261022/run_upright_trials.py --workers 2 --resume
```

主下载包中提供全部动画、轨迹、控制输入和源代码；原始 MJB 在无托/浅托两个模型 ZIP 中，各 9 个。需要完整复现时把三个包解压到同一目录。原脚本中的绝对路径用于标识来源，换机器运行时需要重定位。MJB 可直接用相同版本 MuJoCo 读取并检查，不依赖原手部 mesh 文件路径。
