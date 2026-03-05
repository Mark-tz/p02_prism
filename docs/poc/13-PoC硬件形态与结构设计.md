# 13 — PoC 硬件形态与结构设计

> 文档版本：v0.1.0 | 创建日期：2026-03-05 | 状态：草案
>
> 本文档定义 PoC 原型机的硬件形态、结构布局和关键组件设计。

---

## 1. 整机形态概览

```mermaid
flowchart TB
    subgraph ROBOT["PRISM PoC 整机结构"]
        direction TB

        subgraph TOP["上部 — 双臂系统"]
            RA["右臂（6/7 DOF）<br/>固定末端：夹爪+喷A+喷B"]
            LA["左臂（6/7 DOF）<br/>快换末端：ATC 耦合器"]
        end

        subgraph MID["中部 — 升降与感知"]
            LIFT["升降柱<br/>行程 300mm<br/>适配不同马桶高度"]
            CAM["感知模组<br/>RGB-D 相机<br/>+ 补光灯"]
        end

        subgraph BASE["下部 — 窄体底座"]
            CHASSIS["底盘框架<br/>窄体窄肩设计<br/>≤ 500mm 宽"]
            WHEELS["移动系统<br/>麦克纳姆轮 ×4"]
            TANK["液体系统<br/>清水/消毒剂/清洁剂"]
            TOOLBAY["工具库+自清洁仓<br/>Tool A/B/C 存放与消毒"]
            ELEC["电控仓<br/>计算平台/电源/驱动"]
        end
    end

    TOP --- MID --- BASE

    style TOP fill:#e3f2fd,stroke:#1565c0
    style MID fill:#fff3e0,stroke:#e65100
    style BASE fill:#e8f5e9,stroke:#2e7d32
```

---

## 2. 整机尺寸与约束

```
                 ┌─ RGB-D 相机
                 │  ┌─ 右臂
            ┌────┤  │    ┌─ 左臂
            │    │  │    │
            ▼    ▼  ▼    ▼
        ╔═══════════════════════╗  ← 最大高度 ~1400mm（臂展开）
        ║   双臂工作区域        ║     收纳高度 ~1100mm
        ╠═══════════════════════╣  ← 升降柱顶部 ~1000mm
        ║  升降柱 + 感知模组    ║
        ╠═══════════════════════╣  ← 底座顶部 ~600mm
        ║  液体箱 │ 自清洁仓    ║
        ║─────────┤             ║
        ║  电控仓 │ 工具库      ║
        ╠═══════════════════════╣
        ║  ○  麦克纳姆轮  ○    ║  ← 离地 ~80mm
        ╚═══════════════════════╝
         ←───── 500mm ─────→
         深度约 600mm
```

| 参数 | PoC 目标值 | 说明 |
|------|-----------|------|
| 宽度 | ≤ 500mm | 窄肩设计，未来适配 600mm 门宽 |
| 深度 | ≤ 600mm | 含液体箱 |
| 高度（收纳） | ≤ 1100mm | 升降柱收缩、双臂折叠 |
| 高度（工作） | ≤ 1400mm | 升降柱伸展、臂展开 |
| 总重量 | ≤ 70kg（含液体满载） | 包含双臂 + 底座 + 液体 |
| 液体容量 | 清水 5L + 消毒剂 2L + 清洁剂 2L | PoC 阶段够清洁 5+ 次 |

---

## 3. 双臂系统设计

### 3.1 机械臂选型

```mermaid
flowchart LR
    subgraph 选型对比["协作臂选型对比"]
        direction TB
        
        subgraph OPT_A["方案 A：工业协作臂"]
            A1["如 UR3e / 遨博 i5"]
            A2["✅ 成熟可靠"]
            A3["✅ 力控内置"]
            A4["❌ 重量大（~11kg/臂）"]
            A5["❌ 价格高（8-15万/臂）"]
        end

        subgraph OPT_B["方案 B：轻量协作臂"]
            B1["如 UFACTORY xArm 6/7"]
            B2["✅ 轻量（~8kg）"]
            B3["✅ 力矩传感内置"]
            B4["✅ ROS 2 支持"]
            B5["✅ 价格适中（3-5万/臂）"]
        end

        subgraph OPT_C["方案 C：自研关节臂"]
            C1["基于模组化关节"]
            C2["✅ 完全可控"]
            C3["✅ 量产成本最低"]
            C4["❌ 研发周期长"]
            C5["❌ PoC 阶段风险高"]
        end
    end

    style OPT_A fill:#f3e5f5,stroke:#6a1b9a
    style OPT_B fill:#e8f5e9,stroke:#2e7d32
    style OPT_C fill:#fce4ec,stroke:#b71c1c
```

**PoC 选型决策**：**方案 B — UFACTORY xArm 系列**（或同级国产轻量协作臂）

| 决策依据 | 说明 |
|---------|------|
| PoC 优先 | 成品臂大幅缩短研发周期，专注清洁功能验证 |
| 性价比 | 两臂 ~6-10 万元，PoC 预算可承受 |
| 技术适配 | 内置力矩传感、ROS 2 SDK、碰撞检测 |
| 可扩展 | 量产阶段可平滑过渡到自研关节方案 |

### 3.2 臂端配置

```mermaid
flowchart TB
    subgraph 右臂末端设计["右臂末端 — 固定集成"]
        direction TB
        RF["法兰盘连接"]
        RF --> BRACKET["L型安装支架"]
        BRACKET --> GRIP["平行电爪<br/>行程 0-80mm<br/>夹力 5-30N"]
        BRACKET --> SPA["喷淋头A（消毒剂）<br/>电磁阀控制<br/>雾化喷嘴"]
        BRACKET --> SPB["喷淋头B（清洁剂）<br/>电磁阀控制<br/>雾化喷嘴"]
        BRACKET --> ECAM["Eye-in-Hand 相机<br/>小型 RGB-D"]
    end

    subgraph 左臂末端设计["左臂末端 — 快换系统"]
        direction TB
        LF["法兰盘连接"]
        LF --> ATC_M["ATC 主盘<br/>气动/电动锁止<br/>电+气+水通道"]
        ATC_M -.->|"对接"| ATC_T["ATC 从盘<br/>安装在工具头上"]
    end

    style 右臂末端设计 fill:#e3f2fd,stroke:#1565c0
    style 左臂末端设计 fill:#fff3e0,stroke:#e65100
```

---

## 4. 快换工具系统（ATC）设计

### 4.1 ATC 耦合机构

| 参数 | 规格 | 说明 |
|------|------|------|
| 类型 | 气动锁止式 / 电动卡扣式 | PoC 可选电动卡扣降低气路复杂度 |
| 对接精度 | ±0.5mm 径向 / ±1° 角度 | 配合引导锥面 |
| 锁止力 | ≥ 50N | 防止作业中脱落 |
| 通道 | 电信号 × 4 + 气路 × 1（可选）+ 水路 × 1 | 支持工具电机供电 + 水路 |
| 对接时间 | ≤ 3 秒 | 含对准 + 锁止 + 确认 |
| 引导方式 | 视觉引导（ArUco 标记）+ 机械引导锥 | 粗定位 → 精定位 |

### 4.2 三种工具头

```mermaid
flowchart TB
    subgraph ToolA["Tool A — 干净区布头"]
        TA_ATC["ATC 从盘"]
        TA_PAD["超细纤维擦拭垫<br/>120×80mm<br/>可拆卸更换"]
        TA_SPRING["弹性浮动机构<br/>法线方向 5mm 浮动<br/>被动柔顺"]
        TA_ATC --- TA_SPRING --- TA_PAD
    end

    subgraph ToolB["Tool B — 污染区布头"]
        TB_ATC["ATC 从盘"]
        TB_PAD["超细纤维擦拭垫<br/>120×80mm<br/>加强型（耐磨）"]
        TB_SPRING["弹性浮动机构<br/>同 Tool A"]
        TB_ATC --- TB_SPRING --- TB_PAD
    end

    subgraph ToolC["Tool C — 马桶刷头"]
        TC_ATC["ATC 从盘<br/>含电机供电"]
        TC_MOTOR["微型电机<br/>200-400 RPM"]
        TC_BRUSH["弯曲刷杆<br/>尼龙刷丝<br/>适配 S 弯曲线"]
        TC_ATC --- TC_MOTOR --- TC_BRUSH
    end

    style ToolA fill:#e8f5e9,stroke:#2e7d32
    style ToolB fill:#fff3e0,stroke:#e65100
    style ToolC fill:#fce4ec,stroke:#b71c1c
```

---

## 5. 自清洁仓设计

```mermaid
flowchart TB
    subgraph SELFCLEAN["自清洁仓（核心非标件）"]
        direction TB
        
        subgraph DOCKS["工具停泊位"]
            DA["Dock A<br/>干净区布头"]
            DB["Dock B<br/>污染区布头"]
            DC["Dock C<br/>马桶刷头"]
        end

        subgraph CLEAN_SYS["清洁消毒系统"]
            CL1["高压水喷洗<br/>清除表面污物"]
            CL2["超声波清洗<br/>深度清洁（可选）"]
            CL3["UV-C 紫外消毒<br/>波长 254nm"]
            CL4["热风烘干<br/>防止细菌滋生"]
        end

        subgraph WATER["水路系统"]
            W1["清水入口"]
            W2["污水出口<br/>→ 污水箱"]
            W3["过滤器"]
        end

        DOCKS --> CLEAN_SYS
        CLEAN_SYS --> WATER
    end

    style SELFCLEAN fill:#f3e5f5,stroke:#6a1b9a
    style DOCKS fill:#e3f2fd,stroke:#1565c0
    style CLEAN_SYS fill:#fff3e0,stroke:#e65100
```

| 参数 | 规格 |
|------|------|
| 停泊位数量 | 3 个（A/B/C 各 1） |
| 清洗方式 | 高压水冲洗 + UV-C 消毒 |
| 清洗周期 | 30-60 秒（与下次任务并行） |
| UV-C 功率 | ≥ 5W（254nm） |
| 仓体材质 | 304 不锈钢（耐腐蚀、易清洁） |
| 密封 | 防溅设计，防止清洗液外溢 |
| 污水处理 | 过滤后排入污水箱 |

---

## 6. 底座系统

### 6.1 移动系统

| 参数 | PoC 规格 | 说明 |
|------|---------|------|
| 驱动方式 | 麦克纳姆轮 × 4 | 全向移动，适应狭窄空间横移 |
| 轮径 | Φ100mm | 平衡通过性与底座高度 |
| 电机 | 直流无刷 × 4，50W/个 | 带编码器 |
| 最大速度 | 0.5 m/s | PoC 低速为主 |
| PoC 用途 | 手动推入或遥控到位 | **PoC 不要求自主导航** |
| 扩展预留 | LiDAR 安装法兰 + 超声波安装位 | 为工程机阶段预留 |

### 6.2 升降柱

| 参数 | 规格 |
|------|------|
| 类型 | 电动直线升降柱 |
| 行程 | 300mm |
| 负载 | ≥ 30kg（支撑双臂） |
| 速度 | 20mm/s |
| 用途 | 适配不同高度马桶；调整臂的工作平面 |

### 6.3 液体系统

```mermaid
flowchart LR
    subgraph 液路["液体系统"]
        CW["清水箱 5L"] -->|隔膜泵| MV["三通阀"]
        DIS["消毒剂 2L"] -->|蠕动泵| MV
        CLN["清洁剂 2L"] -->|蠕动泵| MV

        MV -->|管路A| SPA["右臂喷A（消毒剂）"]
        MV -->|管路B| SPB["右臂喷B（清洁剂）"]
        MV -->|管路C| SC_W["自清洁仓冲洗"]

        SC_D["自清洁仓排水"] -->|过滤| DW["污水箱 5L"]
    end

    style 液路 fill:#e3f2fd,stroke:#1565c0
```

---

## 7. 传感器布局（PoC 阶段）

```mermaid
flowchart TB
    subgraph 传感器["PoC 传感器配置"]
        direction TB
        
        S1["头部 RGB-D 相机<br/>Intel RealSense D435i<br/>全局马桶定位 + 状态识别"]
        
        S2["右臂 Eye-in-Hand<br/>小型 RGB 相机<br/>按钮/盖子精确定位"]
        
        S3["左臂 ATC 传感器<br/>接近传感器 + 力传感器<br/>对接引导与确认"]
        
        S4["臂内置力矩传感器<br/>各关节力矩<br/>力控 + 碰撞检测"]
        
        S5["工具库位置传感器<br/>ArUco 标记<br/>精确对接引导"]
    end

    style 传感器 fill:#fff3e0,stroke:#e65100
```

| 传感器 | 数量 | 位置 | 用途 |
|--------|------|------|------|
| Intel RealSense D435i | 1 | 升降柱顶部 | 全局马桶定位、盖/圈状态识别 |
| 小型 RGB 相机 | 1 | 右臂末端 | 按钮/盖子精确定位 |
| 关节力矩传感器 | 内置 | 各臂关节 | 力控、碰撞检测 |
| 接近/位置传感器 | 3 | 工具库各 Dock | ATC 对接引导确认 |
| ArUco 标记 | 6+ | 工具头/Dock/马桶 | 视觉定位参考 |
| 液位传感器 | 4 | 各液体箱 | 余量监测 |

---

## 8. 电气架构

```mermaid
flowchart TB
    subgraph 电气架构["PoC 电气架构"]
        direction TB
        
        PSU["48V 电源<br/>（PoC 使用外接电源）<br/>工程机阶段换电池"]

        PSU --> ARM_DRV["臂控制器 × 2<br/>（协作臂自带）"]
        PSU --> DC24["DC-DC → 24V"]
        PSU --> DC12["DC-DC → 12V"]
        PSU --> DC5["DC-DC → 5V"]
        
        DC24 --> LIFT_M["升降柱电机"]
        DC24 --> WHEEL_M["底盘电机 × 4"]
        DC24 --> PUMP["水泵 + 蠕动泵"]
        DC24 --> VALVE["电磁阀"]
        
        DC12 --> UV["UV-C 灯"]
        DC12 --> TOOL_M["工具头电机（C刷）"]
        
        DC5 --> JETSON["Jetson Orin NX"]
        DC5 --> CAM_ALL["相机 × 2"]
        DC5 --> SENSOR["传感器"]
        DC5 --> SCREEN["触摸屏"]

        JETSON --> ARM_DRV
        JETSON --> STM32["STM32<br/>底盘+液路+自清洁仓控制"]
    end

    style 电气架构 fill:#e3f2fd,stroke:#1565c0
```

**PoC 阶段使用外接 48V 电源**（非电池），降低复杂度和安全风险。电池方案留待工程机阶段。

---

## 9. 可扩展性设计预留

| 预留项 | 预留方式 | 目标阶段 |
|--------|---------|---------|
| 移动底盘 LiDAR | 底座顶部保留法兰安装位 | 工程机 |
| 超声波避障 | 底座四周预留安装孔 | 工程机 |
| 电池仓 | 底座底部预留电池托架接口 | 工程机 |
| 地面清洁模组 | 底座前方预留安装空间 | 工程机 |
| 自动充电/补水对接口 | 底座侧面预留磁吸接口位 | 工程机 |
| 云平台通信 | 主控预装 Wi-Fi + 4G 模块 | 工程机 |
| 额外工具头 | 工具库预留第 4 个 Dock 位 | 迭代 |

---

> 上一篇：[12-PoC 功能定义与清洁工序](12-PoC功能定义与清洁工序.md) | 下一篇：[14-PoC 软件架构与技术方案](14-PoC软件架构与技术方案.md)
