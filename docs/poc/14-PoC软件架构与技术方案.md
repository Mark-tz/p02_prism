# 14 — PoC 软件架构与技术方案

> 文档版本：v0.1.0 | 创建日期：2026-03-05 | 状态：草案
>
> 本文档定义 PoC 原型机的软件架构、核心算法方案和技术栈选型。

---

## 1. 软件架构总览

```mermaid
flowchart TB
    subgraph APP["应用层"]
        TM["任务管理器<br/>清洁工序状态机"]
        BT["行为树<br/>BehaviorTree.CPP"]
        HRI["人机交互<br/>触摸屏UI / 遥控"]
        LOG["日志与录制<br/>rosbag2"]
    end

    subgraph SKILL["技能层"]
        SK1["ArmSkill<br/>双臂运动技能"]
        SK2["ATCSkill<br/>快换对接技能"]
        SK3["SpraySkill<br/>喷洒控制技能"]
        SK4["WipeSkill<br/>擦拭力控技能"]
        SK5["BrushSkill<br/>刷洗力控技能"]
        SK6["GripSkill<br/>夹爪操作技能"]
        SK7["CleanBaySkill<br/>自清洁仓控制"]
    end

    subgraph PERCEPT["感知层"]
        VP1["马桶检测与定位<br/>3D Pose Estimation"]
        VP2["盖/圈状态识别<br/>Classification"]
        VP3["按钮定位<br/>Template Matching"]
        VP4["工具库定位<br/>ArUco Detection"]
        VP5["清洁质量评估<br/>Image Diff"]
    end

    subgraph CTRL["控制层"]
        MC1["臂运动控制<br/>MoveIt 2 + IK"]
        MC2["力控模块<br/>阻抗/柔顺控制"]
        MC3["底盘控制<br/>（PoC 遥控模式）"]
        MC4["升降柱控制"]
        MC5["液路控制<br/>泵+阀"]
    end

    subgraph DRV["驱动层"]
        D1["臂驱动<br/>xArm SDK / ROS 2"]
        D2["相机驱动<br/>RealSense ROS 2"]
        D3["MCU 通信<br/>CAN / Serial"]
    end

    APP --> SKILL --> CTRL --> DRV
    PERCEPT --> SKILL

    style APP fill:#e3f2fd,stroke:#1565c0
    style SKILL fill:#fff3e0,stroke:#e65100
    style PERCEPT fill:#f3e5f5,stroke:#6a1b9a
    style CTRL fill:#e8f5e9,stroke:#2e7d32
    style DRV fill:#fce4ec,stroke:#b71c1c
```

---

## 2. ROS 2 节点拓扑（PoC）

```mermaid
flowchart TB
    subgraph 感知
        N_CAM["/realsense_node<br/>RGB-D 数据"]
        N_ECAM["/eye_in_hand_cam<br/>右臂末端相机"]
        N_DET["/toilet_detector<br/>马桶定位 + 状态识别"]
        N_ARUCO["/aruco_detector<br/>工具库标记检测"]
    end

    subgraph 规划
        N_BT["/task_manager<br/>BehaviorTree 主节点"]
        N_MOVEIT_R["/moveit_right_arm<br/>右臂 MoveIt 2"]
        N_MOVEIT_L["/moveit_left_arm<br/>左臂 MoveIt 2"]
    end

    subgraph 技能
        N_WIPE["/wipe_skill<br/>擦拭力控"]
        N_BRUSH["/brush_skill<br/>刷洗力控"]
        N_ATC["/atc_skill<br/>快换对接"]
        N_SPRAY["/spray_skill<br/>喷洒控制"]
        N_GRIP["/grip_skill<br/>夹爪操作"]
    end

    subgraph 驱动
        N_ARM_R["/xarm_right<br/>右臂驱动"]
        N_ARM_L["/xarm_left<br/>左臂驱动"]
        N_MCU["/mcu_bridge<br/>STM32 通信<br/>底盘/液路/自清洁仓"]
    end

    subgraph 管理
        N_HRI["/hri_node<br/>UI + 操作面板"]
        N_DIAG["/diagnostics<br/>状态监控"]
        N_BAG["/rosbag_recorder<br/>数据录制"]
    end

    N_CAM -->|/camera/color, /camera/depth| N_DET
    N_ECAM -->|/eye_hand/image| N_DET
    N_DET -->|/toilet_pose, /lid_state| N_BT
    N_ARUCO -->|/tool_dock_poses| N_ATC

    N_BT -->|Action| N_MOVEIT_R & N_MOVEIT_L
    N_BT -->|Action| N_WIPE & N_BRUSH & N_ATC & N_SPRAY & N_GRIP

    N_MOVEIT_R -->|/joint_cmd| N_ARM_R
    N_MOVEIT_L -->|/joint_cmd| N_ARM_L
    N_SPRAY & N_ATC -->|Serial cmd| N_MCU

    N_ARM_R & N_ARM_L -->|/joint_states, /ft_sensor| N_MOVEIT_R & N_MOVEIT_L
    N_DIAG -->|/diagnostics| N_HRI

    style 感知 fill:#f3e5f5,stroke:#6a1b9a
    style 规划 fill:#e3f2fd,stroke:#1565c0
    style 技能 fill:#fff3e0,stroke:#e65100
    style 驱动 fill:#e8f5e9,stroke:#2e7d32
    style 管理 fill:#fce4ec,stroke:#b71c1c
```

---

## 3. 行为树设计

### 3.1 顶层行为树

```mermaid
flowchart TB
    ROOT["Root<br/>Sequence"] --> CHK["CheckReady<br/>Condition"]
    ROOT --> PH1["Phase I<br/>Sequence"]
    ROOT --> PH2["Phase II<br/>Sequence"]
    ROOT --> PH3["Phase III<br/>Sequence"]
    ROOT --> PH4["Phase IV<br/>Sequence"]
    ROOT --> PH5["Phase V<br/>Sequence"]
    ROOT --> DONE["ReportComplete<br/>Action"]

    PH1 --> S01["CloseLid"] --> S02["SprayDisinfectAll"] --> S03["PressFlush"]
    
    PH2 --> ATCPickA["ATC_Pick_A"]
    ATCPickA --> S04["WipeTank"]
    S04 --> S05["WipeLidTop"]
    S05 --> S06["OpenLid"]
    S06 --> S07["SpraySeatTop"]
    S07 --> S08["WipeSeatTop"]
    S08 --> S09["ATC_Return_A"]

    style ROOT fill:#1565c0,color:#fff
    style PH1 fill:#e8f5e9,stroke:#2e7d32
    style PH2 fill:#e3f2fd,stroke:#1565c0
    style PH3 fill:#fff3e0,stroke:#e65100
    style PH4 fill:#fce4ec,stroke:#b71c1c
    style PH5 fill:#f3e5f5,stroke:#6a1b9a
```

### 3.2 错误恢复策略

```mermaid
flowchart TB
    ACTION["执行动作"] --> CHECK{"成功?"}
    CHECK -->|是| NEXT["下一步"]
    CHECK -->|否| RETRY{"重试次数<3?"}
    RETRY -->|是| RESET["重置位姿"] --> ACTION
    RETRY -->|否| SEVERITY{"严重程度"}
    SEVERITY -->|轻微| SKIP["跳过并标记<br/>继续下一步"]
    SEVERITY -->|中等| PAUSE["暂停<br/>等待人工确认"]
    SEVERITY -->|严重| ESTOP["急停<br/>安全位复位"]
```

**错误分级**：

| 等级 | 示例 | 处理方式 |
|------|------|---------|
| 轻微 | 擦拭覆盖率不足 | 重试 1-2 次，仍失败则跳过并标记 |
| 中等 | ATC 对接失败 | 暂停，等待人工检查后继续 |
| 严重 | 碰撞/力矩超限/通信中断 | 急停，双臂回安全位，等待人工复位 |

---

## 4. 视觉算法方案

### 4.1 马桶定位

```mermaid
flowchart LR
    INPUT["RGB-D 图像"] --> SEG["实例分割<br/>YOLOv8-Seg"]
    SEG --> PC["点云提取<br/>感兴趣区域"]
    PC --> ICP["ICP 配准<br/>与 CAD 模型对齐"]
    ICP --> POSE["6DOF 位姿<br/>(x,y,z,rx,ry,rz)"]
    
    style INPUT fill:#e3f2fd,stroke:#1565c0
    style POSE fill:#e8f5e9,stroke:#2e7d32
```

| 步骤 | 算法 | 说明 |
|------|------|------|
| 检测 | YOLOv8-Seg | 马桶实例分割，提取 Mask |
| 点云 | RealSense D435i | 深度图 → 有组织点云 → Mask 裁剪 |
| 配准 | ICP / Super4PCS | 与预建 CAD 模型对齐，输出 6DOF Pose |
| 细化 | ArUco 标记（辅助） | 马桶旁放置标记板辅助初始化（PoC） |

### 4.2 盖/圈状态识别

| 方案 | 说明 |
|------|------|
| 方法 | 图像分类 — ResNet-18 / MobileNet |
| 类别 | 全关（盖+圈）、盖开圈关、全开 |
| 训练数据 | 实拍 500+ 张 + 数据增强 |
| 推理 | TensorRT FP16，Jetson 上 < 10ms |

### 4.3 力控曲面跟踪

```mermaid
flowchart LR
    TARGET["目标力 Fd"] --> IMP["阻抗控制器<br/>M·ẍ + D·ẋ + K·x = Fd - Fe"]
    FT["力矩传感器 Fe"] --> IMP
    IMP --> DELTA["位置修正 Δx"]
    POSE_CMD["轨迹位置 Xd"] --> SUM["Xd + Δx"]
    DELTA --> SUM
    SUM --> ARM["臂关节控制"]
    ARM --> FT

    style TARGET fill:#e3f2fd,stroke:#1565c0
    style IMP fill:#fff3e0,stroke:#e65100
```

**关键参数（PoC 初始值，需现场调参）**：

| 参数 | 擦拭模式 | 刷洗模式 | 说明 |
|------|---------|---------|------|
| 目标力 Fd | 3 N | 10 N | 法线方向 |
| 惯性 M | 5 kg | 10 kg | 虚拟惯性 |
| 阻尼 D | 100 Ns/m | 200 Ns/m | 虚拟阻尼 |
| 刚度 K | 500 N/m | 300 N/m | 虚拟刚度 |
| 控制频率 | 500 Hz | 500 Hz | 臂控制器内环 |

---

## 5. 技术栈汇总

| 层级 | 选型 | 版本 | 说明 |
|------|------|------|------|
| OS | Ubuntu 22.04 + PREEMPT-RT | LTS | Jetson Orin NX |
| 中间件 | ROS 2 Humble | LTS | 通信框架 |
| 臂规划 | MoveIt 2 | latest | 运动规划 + IK |
| 行为规划 | BehaviorTree.CPP v4 | 4.x | 任务编排 |
| 视觉-检测 | YOLOv8 | 8.x | 马桶分割/分类 |
| 视觉-定位 | Open3D + ICP | latest | 点云配准 |
| 视觉-标记 | OpenCV ArUco | 4.x | ATC 引导定位 |
| 推理加速 | TensorRT | 8.x | FP16/INT8 推理 |
| 力控 | 自研（基于臂 SDK） | — | 阻抗/柔顺控制 |
| 下位机 | FreeRTOS + STM32 HAL | — | 液路/自清洁仓 |
| 仿真 | Gazebo Fortress + MoveIt | — | 离线调试验证 |
| 可视化 | RViz2 + Groot2 | — | 状态/行为树调试 |

---

## 6. 仿真方案

```mermaid
flowchart LR
    subgraph 仿真环境["Gazebo + MoveIt 2 仿真"]
        SIM_WORLD["卫生间场景模型<br/>马桶 + 地面 + 墙壁"]
        SIM_ROBOT["PRISM URDF 模型<br/>双臂 + 底座 + 工具"]
        SIM_SENSOR["虚拟传感器<br/>RGB-D + 力矩"]
    end

    subgraph 真机["真机控制"]
        REAL_ROBOT["实际硬件"]
    end

    SIM_WORLD & SIM_ROBOT & SIM_SENSOR --> ROS2["ROS 2 话题/服务<br/>（接口统一）"]
    REAL_ROBOT --> ROS2

    ROS2 --> APP_STACK["应用层代码<br/>（仿真/真机 共用）"]

    style 仿真环境 fill:#e3f2fd,stroke:#1565c0
    style 真机 fill:#e8f5e9,stroke:#2e7d32
```

**仿真优先策略**：所有清洁工序先在 Gazebo 中验证轨迹和逻辑，通过后再上真机。减少真机调试风险和耗时。

---

## 7. 数据录制与回放

| 录制内容 | Topic | 用途 |
|---------|-------|------|
| 相机图像 | /camera/color, /camera/depth | 视觉算法优化 |
| 关节状态 | /joint_states | 运动回放 |
| 力矩数据 | /ft_sensor | 力控参数调优 |
| TF 树 | /tf, /tf_static | 坐标系验证 |
| 行为树状态 | /bt_status | 流程问题排查 |
| 诊断信息 | /diagnostics | 异常分析 |

每次清洁作业自动录制 rosbag2，形成**可回溯的数据资产**，为后续 AI 训练和算法优化积累素材。

---

> 上一篇：[13-PoC 硬件形态](13-PoC硬件形态与结构设计.md) | 下一篇：[15-PoC 开发规划](15-PoC开发规划与团队配置.md)
