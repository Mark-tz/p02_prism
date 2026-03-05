# 12 — PoC 功能定义与清洁工序

> 文档版本：v0.1.0 | 创建日期：2026-03-05 | 状态：草案
>
> 本文档详细定义 PoC 阶段的马桶清洁功能和 19 步标准工序。

---

## 1. 清洁对象定义

### 1.1 目标马桶类型

| 参数 | PoC 首选 | 扩展目标 |
|------|---------|---------|
| 类型 | 连体式坐便器 | 分体式、壁挂式 |
| 冲水方式 | 按钮式 | 扳手式、感应式 |
| 尺寸范围 | 长 650-750mm × 宽 350-400mm × 高 380-420mm | 后续适配更多尺寸 |
| 品牌参考 | TOTO / 科勒 / 箭牌（主流型号） | 覆盖市场 Top 10 品牌 |
| 材质 | 陶瓷 | — |
| 有无水箱 | 有外露水箱 | 隐藏水箱型 |

### 1.2 清洁表面分区

```mermaid
flowchart TB
    subgraph 马桶表面分区["马桶清洁表面分区"]
        direction TB
        
        subgraph CLEAN["🟢 干净区（Clean Zone）"]
            CZ1["水箱外表面"]
            CZ2["马桶盖上表面"]
            CZ3["座圈上表面"]
        end

        subgraph DIRTY["🟡 污染区（Contaminated Zone）"]
            DZ1["马桶盖下表面"]
            DZ2["座圈下表面"]
            DZ3["马桶沿（外圈）"]
        end

        subgraph INNER["🔴 内壁区（Inner Zone）"]
            IZ1["马桶内壁"]
            IZ2["S弯上沿"]
        end

        subgraph INTERACT["⚙️ 交互区"]
            IT1["冲水按钮"]
            IT2["马桶盖铰链"]
            IT3["座圈铰链"]
        end
    end

    style CLEAN fill:#e8f5e9,stroke:#2e7d32
    style DIRTY fill:#fff3e0,stroke:#e65100
    style INNER fill:#fce4ec,stroke:#b71c1c
    style INTERACT fill:#e3f2fd,stroke:#1565c0
```

**防交叉污染核心原则**：干净区 → 污染区 → 内壁区，严格按污染程度递进，**不同分区使用不同工具头**。

---

## 2. 末端执行器与工具定义

### 2.1 右臂 — 固定配置

```mermaid
flowchart LR
    subgraph 右臂末端["右臂末端（固定集成）"]
        direction TB
        G["多功能夹爪<br/>开合力 5-30N<br/>行程 0-80mm"]
        SA["喷淋头 A<br/>消毒剂<br/>雾化喷射"]
        SB["喷淋头 B<br/>中性清洁剂<br/>雾化喷射"]
    end

    style 右臂末端 fill:#e3f2fd,stroke:#1565c0
```

| 组件 | 规格 | 功能 |
|------|------|------|
| 多功能夹爪 | 平行双指，自适应指尖，力传感器 | 按冲水按钮、开关马桶盖/座圈 |
| 喷淋头 A | 雾化喷嘴，流量可调 0-50 mL/s | 喷洒消毒剂（次氯酸/季铵盐类） |
| 喷淋头 B | 雾化喷嘴，流量可调 0-50 mL/s | 喷洒中性清洁剂 |

### 2.2 左臂 — 快换系统（ATC）

```mermaid
flowchart TB
    subgraph ATC["左臂快换系统 (Automatic Tool Changer)"]
        direction TB
        COUPLER["快换耦合机构<br/>气动/电动锁止<br/>对接精度 ±0.5mm"]
        
        subgraph TOOLS["三种可更换工具头"]
            TA["Tool A — 干净区布头<br/>超细纤维擦拭垫<br/>用于水箱/盖板/座圈上表面"]
            TB["Tool B — 污染区布头<br/>超细纤维擦拭垫<br/>用于盖板下表面/座圈下表面/马桶沿"]
            TC["Tool C — 马桶刷头<br/>尼龙刷丝 + 旋转电机<br/>用于马桶内壁刷洗"]
        end
    end

    COUPLER --> TA & TB & TC

    style ATC fill:#fff3e0,stroke:#e65100
    style TA fill:#e8f5e9,stroke:#2e7d32
    style TB fill:#fff3e0,stroke:#e65100
    style TC fill:#fce4ec,stroke:#b71c1c
```

| 工具 | 标识色 | 目标表面 | 工作方式 | 防污染逻辑 |
|------|--------|---------|---------|-----------|
| Tool A 布头 | 绿色 | 干净区（水箱/盖上/圈上） | 接触式擦拭，力控 2-5N | 仅接触干净表面，最先使用 |
| Tool B 布头 | 黄色 | 污染区（盖下/圈下/马桶沿） | 接触式擦拭，力控 3-8N | 接触潜在污染表面，A 之后使用 |
| Tool C 刷头 | 红色 | 内壁区（马桶内壁） | 旋转刷洗 200-400 RPM | 最后使用，最高污染级别 |

---

## 3. 标准清洁工序（19 步）

### 3.1 工序总览时序图

```mermaid
sequenceDiagram
    participant R as 右臂（固定）
    participant L as 左臂（快换）
    participant SC as 自清洁仓
    participant T as 马桶

    rect rgb(232, 245, 233)
        Note over R,T: Phase I — 准备与消毒（右臂主导）
        R->>T: ① 放下座圈和盖子
        R->>T: ② 外表面整体喷消毒剂（喷A）
        R->>T: ③ 按冲水按钮（第一次）
    end

    rect rgb(227, 242, 253)
        Note over R,T: Phase II — 干净区表面精密清洁
        L->>SC: 取 Tool A（干净区布头）
        L->>T: ④ 擦拭水箱外表面
        L->>T: ⑤ 擦拭马桶盖上表面
        R->>T: ⑥ 打开马桶盖
        R->>T: ⑦ 座圈上表面+盖下表面喷消毒剂（喷A）
        L->>T: ⑧ 擦拭座圈上表面
        L->>SC: ⑨ Tool A 归位自清洁
    end

    rect rgb(255, 243, 224)
        Note over R,T: Phase III — 污染区表面清洁
        L->>SC: 取 Tool B（污染区布头）
        L->>T: ⑩ 擦拭马桶盖下表面
        R->>T: ⑪ 打开座圈
        R->>T: ⑫ 座圈下表面+马桶沿喷消毒剂（喷A）
        L->>T: ⑬ 擦拭座圈下表面+马桶沿
        L->>SC: ⑭ Tool B 归位自清洁
    end

    rect rgb(252, 228, 236)
        Note over R,T: Phase IV — 马桶内壁清洁
        L->>SC: 取 Tool C（马桶刷头）
        R->>T: ⑮ 马桶内壁喷清洁剂（喷B）
        L->>T: ⑯ 马桶内壁刷洗（随喷随刷）
        L->>SC: ⑰ Tool C 归位自清洁
    end

    rect rgb(237, 231, 246)
        Note over R,T: Phase V — 收尾与自清洁
        R->>T: ⑱ 按冲水按钮（第二次）
        SC->>SC: ⑲ ABC 全部工具头自清洁消毒
    end
```

### 3.2 工序详细定义

| 工序 | 内容 | 执行臂 / 工具 | 操作对象 | 技术要点 | 防污染逻辑 |
|:----:|------|---------------|---------|---------|-----------|
| **Phase I — 准备与消毒** | | | | | |
| ① | 放下座圈和盖子（若开启） | 右臂 → 夹爪 | 座圈 + 盖子 | 力控合页操作；检测当前状态 | 交互动作 |
| ② | 外表面整体喷消毒剂 | 右臂 → 喷 A | 马桶整体外表面 | 均匀覆盖喷洒；预设轨迹 | 整体消毒，降低后续风险 |
| ③ | 按冲水按钮（第一次） | 右臂 → 夹爪 | 冲水按钮 | 力控按压 10-20N；识别按钮位置 | 冲去旧污 |
| **Phase II — 干净区清洁** | | | | | |
| ④ | 擦拭水箱外表面 | 左臂 → **Tool A** | 水箱 | 接触力 2-5N；覆盖式轨迹 | 干净区 A 布 |
| ⑤ | 擦拭马桶盖上表面 | 左臂 → Tool A | 盖上表面 | 接触力 2-5N；平面跟踪 | 干净区 A 布 |
| ⑥ | 打开马桶盖 | 右臂 → 夹爪 | 马桶盖 | 力控铰链操作；角度检测 | 交互动作 |
| ⑦ | 座圈上+盖下喷消毒剂 | 右臂 → 喷 A | 座圈上 + 盖下 | 分区喷洒 | 为后续擦拭消毒 |
| ⑧ | 擦拭座圈上表面 | 左臂 → Tool A | 座圈上表面 | 环形轨迹；力控 3-5N | A 布最后一步使用 |
| ⑨ | Tool A 归位自清洁 | 左臂 → ATC | 自清洁仓 | 对接精度 ±0.5mm | A 布消毒处理 |
| **Phase III — 污染区清洁** | | | | | |
| ⑩ | 擦拭马桶盖下表面 | 左臂 → **Tool B** | 盖下表面 | 力控 3-8N；曲面跟踪 | 污染区 B 布 |
| ⑪ | 打开座圈 | 右臂 → 夹爪 | 座圈 | 力控铰链操作 | 交互动作 |
| ⑫ | 座圈下+马桶沿喷消毒剂 | 右臂 → 喷 A | 座圈下 + 马桶沿 | 分区喷洒 | 为后续擦拭消毒 |
| ⑬ | 擦拭座圈下+马桶沿 | 左臂 → Tool B | 座圈下 + 马桶沿 | 复杂曲面跟踪；力控 5-8N | B 布最后一步使用 |
| ⑭ | Tool B 归位自清洁 | 左臂 → ATC | 自清洁仓 | 对接精度 ±0.5mm | B 布消毒处理 |
| **Phase IV — 内壁清洁** | | | | | |
| ⑮ | 马桶内壁喷清洁剂 | 右臂 → 喷 B | 内壁 | 环形喷洒覆盖 | 清洁剂辅助 |
| ⑯ | 马桶内壁刷洗（随喷随刷） | 左臂 → **Tool C** + 右臂 → 喷 B | 内壁 | 双臂协作；刷头旋转 200-400RPM；力控 5-15N | 最高污染级别 |
| ⑰ | Tool C 归位自清洁 | 左臂 → ATC | 自清洁仓 | 对接精度 ±0.5mm | C 刷消毒处理 |
| **Phase V — 收尾** | | | | | |
| ⑱ | 按冲水按钮（第二次） | 右臂 → 夹爪 | 冲水按钮 | 同 ③ | 冲去清洁剂和残留 |
| ⑲ | 所有工具头自清洁 | 自清洁仓 | ABC 全部 | 蒸汽/超声波/UV 消毒 | 全部工具复位为洁净状态 |

---

## 4. 双臂协作时序

```mermaid
gantt
    title 双臂协作时序（单次马桶清洁）
    dateFormat mm:ss
    axisFormat %M:%S

    section Phase I 准备
    ① 放下盖子（右臂）        :r1, 00:00, 15s
    ② 喷消毒剂（右臂）        :r2, after r1, 20s
    ③ 按冲水（右臂）           :r3, after r2, 10s

    section Phase II 干净区
    取 Tool A（左臂）          :l1, after r2, 10s
    ④ 擦水箱（左臂）           :l2, after r3, 30s
    ⑤ 擦盖上（左臂）           :l3, after l2, 25s
    ⑥ 开盖（右臂）             :r4, after l3, 10s
    ⑦ 喷消毒（右臂）           :r5, after r4, 15s
    ⑧ 擦座圈上（左臂）         :l4, after r5, 25s
    ⑨ 归位 A（左臂）           :l5, after l4, 10s

    section Phase III 污染区
    取 Tool B（左臂）          :l6, after l5, 10s
    ⑩ 擦盖下（左臂）           :l7, after l6, 25s
    ⑪ 开座圈（右臂）           :r6, after l7, 10s
    ⑫ 喷消毒（右臂）           :r7, after r6, 15s
    ⑬ 擦座圈下+沿（左臂）      :l8, after r7, 35s
    ⑭ 归位 B（左臂）           :l9, after l8, 10s

    section Phase IV 内壁
    取 Tool C（左臂）          :l10, after l9, 10s
    ⑮ 喷清洁剂（右臂）         :r8, after l10, 10s
    ⑯ 刷洗内壁（双臂协作）     :l11, after r8, 60s
    ⑰ 归位 C（左臂）           :l12, after l11, 10s

    section Phase V 收尾
    ⑱ 按冲水（右臂）           :r9, after l12, 10s
    ⑲ 全部自清洁（仓内）       :sc, after r9, 60s
```

**预估总耗时**：约 **7-9 分钟**（不含等待自清洁仓完成的时间，自清洁与下次任务并行）。

---

## 5. 清洁工序状态机

```mermaid
stateDiagram-v2
    [*] --> IDLE : 就绪

    IDLE --> PHASE_I : 收到清洁指令
    
    state PHASE_I {
        [*] --> S01_CLOSE_LID
        S01_CLOSE_LID --> S02_SPRAY_DISINFECT
        S02_SPRAY_DISINFECT --> S03_FLUSH_1
        S03_FLUSH_1 --> [*]
    }

    PHASE_I --> PHASE_II : Phase I 完成

    state PHASE_II {
        [*] --> ATC_PICK_A
        ATC_PICK_A --> S04_WIPE_TANK
        S04_WIPE_TANK --> S05_WIPE_LID_TOP
        S05_WIPE_LID_TOP --> S06_OPEN_LID
        S06_OPEN_LID --> S07_SPRAY_SEAT
        S07_SPRAY_SEAT --> S08_WIPE_SEAT_TOP
        S08_WIPE_SEAT_TOP --> S09_ATC_RETURN_A
        S09_ATC_RETURN_A --> [*]
    }

    PHASE_II --> PHASE_III : Phase II 完成

    state PHASE_III {
        [*] --> ATC_PICK_B
        ATC_PICK_B --> S10_WIPE_LID_BTM
        S10_WIPE_LID_BTM --> S11_OPEN_SEAT
        S11_OPEN_SEAT --> S12_SPRAY_SEAT_BTM
        S12_SPRAY_SEAT_BTM --> S13_WIPE_SEAT_BTM
        S13_WIPE_SEAT_BTM --> S14_ATC_RETURN_B
        S14_ATC_RETURN_B --> [*]
    }

    PHASE_III --> PHASE_IV : Phase III 完成

    state PHASE_IV {
        [*] --> ATC_PICK_C
        ATC_PICK_C --> S15_SPRAY_INNER
        S15_SPRAY_INNER --> S16_BRUSH_INNER
        S16_BRUSH_INNER --> S17_ATC_RETURN_C
        S17_ATC_RETURN_C --> [*]
    }

    PHASE_IV --> PHASE_V : Phase IV 完成

    state PHASE_V {
        [*] --> S18_FLUSH_2
        S18_FLUSH_2 --> S19_SELF_CLEAN
        S19_SELF_CLEAN --> [*]
    }

    PHASE_V --> IDLE : 清洁完成

    PHASE_I --> ERROR : 异常
    PHASE_II --> ERROR : 异常
    PHASE_III --> ERROR : 异常
    PHASE_IV --> ERROR : 异常
    ERROR --> IDLE : 人工复位
```

---

## 6. 视觉感知需求（PoC 阶段）

| 感知任务 | 用途 | 方案 | 精度要求 |
|---------|------|------|---------|
| 马桶整体定位 | 确定马桶基准坐标系 | RGB-D + 3D 模型匹配 | 位置 ±5mm，姿态 ±2° |
| 马桶盖/座圈状态识别 | 判断开/关/半开 | RGB 图像分类 | 准确率 ≥ 95% |
| 冲水按钮定位 | 引导夹爪按压 | RGB-D + 模板匹配 | ±3mm |
| 清洁表面法线估计 | 擦拭/刷洗时力控方向 | 深度图法线计算 | ±5° |
| 工具库位姿检测 | ATC 对接引导 | ArUco 标记 + 深度相机 | ±0.5mm |
| 清洁效果评估 | 清洁前后对比 | RGB 图像差异检测 | 定性评估 |

---

## 7. 力控需求

| 场景 | 接触力范围 | 控制策略 | 安全阈值 |
|------|-----------|---------|---------|
| 擦拭水箱/盖板（平面） | 2-5 N | 阻抗控制，法线方向恒力 | > 10 N 退缩 |
| 擦拭座圈（曲面） | 3-8 N | 阻抗控制 + 曲面法线跟踪 | > 15 N 退缩 |
| 刷洗内壁 | 5-15 N | 阻抗控制 + 旋转力矩限制 | > 25 N 急停 |
| 按冲水按钮 | 10-20 N | 位置-力混合控制 | > 30 N 急停 |
| 开关盖子/座圈 | 3-10 N | 柔顺控制，跟随铰链运动 | > 20 N 急停 |
| ATC 对接 | 5-15 N | 引导对接，卡锁确认 | > 25 N 急停 |

---

> 上一篇：[11-PoC 总体规划](11-PoC总体规划与目标定义.md) | 下一篇：[13-PoC 硬件形态](13-PoC硬件形态与结构设计.md)
