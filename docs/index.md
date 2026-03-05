# PRISM — 公共卫生间智能清洁机器人

> **P**ublic **R**estroom **I**ntelligent **S**anitation **M**achine

---

## 项目简介

PRISM 的终极目标是研发一台能够**全自动完成公共卫生间清洁作业**的智能机器人，同时配套完善的**智慧管理平台**形成完整的闭环运营体系。

项目文档分为两大维度：

<div class="grid cards" markdown>

-   :material-cloud-outline:{ .lg .middle } **平台维度 — 智慧管理**

    ---

    感知、调度、质检、公众互动、数据分析的云端管理平台

    [:octicons-arrow-right-24: 查看平台文档](platform/00-项目概述与总体流程.md)

-   :material-robot-outline:{ .lg .middle } **机器人维度 — 本体研发**

    ---

    机器人本体的需求、形态、功能、技术选型、研发规划

    [:octicons-arrow-right-24: 查看机器人文档](robot/06-机器人需求分析.md)

-   :material-toilet:{ .lg .middle } **PoC — 马桶清洁切入**

    ---

    以马桶清洁为切入点的 PoC 原型：双臂协作、19 步工序、快换工具

    [:octicons-arrow-right-24: 查看 PoC 文档](poc/11-PoC总体规划与目标定义.md)

</div>

---

## 文档索引

### 平台维度

| 序号 | 文档 | 说明 |
|:----:|------|------|
| 00 | [项目概述与总体流程](platform/00-项目概述与总体流程.md) | 项目背景、愿景、总体流程 |
| 01 | [需求分析](platform/01-需求分析.md) | 利益相关方、用户故事、需求清单 |
| 02 | [可行性分析](platform/02-可行性分析.md) | 技术 / 经济 / 操作 / 法律可行性 |
| 03 | [功能梳理](platform/03-功能梳理.md) | 功能模块树、核心业务流程 |
| 04 | [技术选型](platform/04-技术选型.md) | 架构方案、技术栈对比、选型决策 |
| 05 | [项目规划与里程碑](platform/05-项目规划与里程碑.md) | WBS、里程碑、资源计划 |

### 机器人维度

| 序号 | 文档 | 说明 |
|:----:|------|------|
| 06 | [机器人需求分析](robot/06-机器人需求分析.md) | 作业场景、环境约束、清洁对象、用户故事 |
| 07 | [机器人形态探究](robot/07-机器人形态探究.md) | 构型对比、运动底盘、清洁机构、工业设计 |
| 08 | [功能定义与系统架构](robot/08-机器人功能定义与系统架构.md) | 感知 / 规划 / 执行 / 人机交互 / 自维护子系统 |
| 09 | [机器人技术选型](robot/09-机器人技术选型.md) | 传感器 / 算力平台 / 驱动 / 清洁模组 / 通信选型 |
| 10 | [研发规划与里程碑](robot/10-机器人研发规划与里程碑.md) | 原型 → 工程机 → 量产的阶段规划 |

### PoC — 马桶清洁

| 序号 | 文档 | 说明 |
|:----:|------|------|
| 11 | [PoC 总体规划与目标](poc/11-PoC总体规划与目标定义.md) | 战略定位、验证范围、成功标准 |
| 12 | [PoC 功能定义与清洁工序](poc/12-PoC功能定义与清洁工序.md) | 19 步标准工序、双臂协作时序、状态机 |
| 13 | [PoC 硬件形态与结构设计](poc/13-PoC硬件形态与结构设计.md) | 双臂 / 底座 / ATC 快换 / 自清洁仓 |
| 14 | [PoC 软件架构与技术方案](poc/14-PoC软件架构与技术方案.md) | ROS 2 节点 / 行为树 / 视觉 / 力控 |
| 15 | [PoC 开发规划与团队配置](poc/15-PoC开发规划与团队配置.md) | 甘特图 / 人员 / 预算 / 风险 |

---

## 术语表

| 术语 | 全称 / 解释 |
|------|------------|
| PRISM | Public Restroom Intelligent Sanitation Machine |
| SLAM | Simultaneous Localization and Mapping，即时定位与地图构建 |
| ROS | Robot Operating System，机器人操作系统 |
| LiDAR | Light Detection and Ranging，激光雷达 |
| DOF | Degrees of Freedom，自由度 |
| IoT | Internet of Things，物联网 |
| OTA | Over-The-Air，空中升级 |
| MTBF | Mean Time Between Failures，平均故障间隔 |
| BMS | Battery Management System，电池管理系统 |
| IP67 | 防护等级：防尘 6 级 + 防水 7 级 |
