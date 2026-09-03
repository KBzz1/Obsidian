---
tags:
  - 交接
  - 元文档
---

# 交接说明：记忆系统与新 AI 衔接

> 本文档写给**接手的下一个 AI**。它解释这套 DSH 知识体系（Obsidian Vault + OpenViking 记忆）如何运转、记忆里存了什么、两个项目现在到哪一步、以及你必须遵守的写作约定。接手前先通读本文档，再按需深入各项目目录。

## 1. 系统总览：三块怎么协同

| 层 | 是什么 | 角色 |
| --- | --- | --- |
| **DSH 框架** | Agent 工作框架，整合 Obsidian 与 OpenViking | 你的运行环境 |
| **Obsidian Vault** | `/mnt/c/Users/97949/Documents/KB_310` | **知识的唯一事实来源（source of truth）** |
| **OpenViking**（用户口中的 llmviking） | 语义检索 + 长期记忆层，server 在 `http://127.0.0.1:1933` | 跨会话召回上下文，**不是权威副本** |

- DSH 组件：`@deepseek-ai/dsh-mcp-client`（Obsidian MCP，读写 Markdown 原文）+ `@openviking/dsh-memory-plugin`（语义检索/层级上下文/Memory）。**不要手工再加 mcp-openviking**，避免重复通道。
- 同步链路：Vault 的 `.md` 修改 → `obsidian-ov-sync.sh` 常驻监听 → `ov add-resource` → `viking://resources/obsidian/KB_310/...`。**OpenViking 里 vault 的副本由同步维护，不要手工维护其 `.abstract.md`/`.overview.md` 等生成物。**
- OpenViking 自身另有 `memories` 层（见下节），由 OpenViking 从会话里自动抽取，与 vault 同步是两回事。

## 2. 记忆里存了什么（viking:// URI 地图）

OpenViking 里有两套互相独立的记忆：

### 2.1 resources —— vault 的同步副本
- 前缀 `viking://resources/obsidian/KB_310/...`，权威永远是 vault 文件本身。
- 结构镜像 vault：`找工作啊啊啊啊/资料/`（面经、教学资料链接）+ `找工作啊啊啊啊/项目梳理/个人健康管理agent/`（七章笔记）。

### 2.2 memories —— OpenViking 自动抽取的长期记忆
- `viking://user/default/memories/profile.md`：用户画像（kbzz1：重庆大学研究生、AI Agent 应用工程方向求职、智慧麻醉/慢阻肺/开题/综述/大论文科研线）。
- `viking://user/default/memories/identity.md` / `soul.md`：助手自我定义（identity 基本是空模板，soul 是通用价值观）。
- `viking://user/default/memories/preferences/kbzz1/*.md`：**用户偏好，直接影响你的输出风格**（写作、英文规则、Canvas、周报、整理方式等，见第 6 节）。
- `viking://user/default/memories/entities/`：实体记忆——`项目/个人健康管理agent.md`、`项目/闪卡app.md`（两项目最新状态，改动后要更新）；`工具/`（humanizer_zh、weekly-report-zh、obsidian 整理技能集）、`框架/dsh.md`、`知识库/`、`知识库系统/openviking.md`。
- `viking://user/default/memories/events/2026/08/...`：按日归档的事件时间线。

### 2.3 怎么检索
- 会话开始会注入 `<openviking-context>` 块，**先看它**，命中就不必再查。
- 不够时：`search`（`mode="context"` 首选，问「我知道 X 的什么」）→ `find`（快速排名）→ `grep`/`glob`（精确文本/文件名）→ `read`/`list`（展开 URI，`read` 支持批量）。
- `viking://` 是虚拟路径，**不要**传给文件系统工具；vault 文件才用 `read` 或 Obsidian MCP。

## 3. 工作目录与环境

- DSH 工作目录：`/home/kbzz1/obsidian_workspace`（含 `AGENTS.md`、`obsidian-ov-sync.sh`、`.dsh/skills/`、`docs/agent_review/` 重构草稿）。**不是知识库存储目录。**
- Vault：`/mnt/c/Users/97949/Documents/KB_310`（Windows 挂载）。
- **写入通道**：bash 对 vault 默认只读（EROFS）；常规笔记修改**走 Obsidian MCP**（`vault_write`/`vault_patch` 等）。写 `.canvas` JSON 用 `vault_write` 原样写入（`obsidian create` 会把 `\n` 当换行，破坏 JSON 转义）。
- 关键环境：OpenViking server `127.0.0.1:1933`（conda env `openviking`）；`ov` CLI 在 `/home/kbzz1/miniconda3/envs/openviking/bin/ov`。
- 同步：`obsidian-ov-sync.sh` 常驻监听 vault 变化（INTERVAL=5s，DEBOUNCE=10s）后 `ov add-resource --include "*.md"`。**注意：该守护进程当前未在运行**（接手时先确认是否需要拉起）。

## 4. 项目 A：个人健康管理 Agent（知识库主线）

定位：**面试导向的 AI Agent / Harness 学习项目**，健身/饮食（Workout/Diet/DailyPlan）只是业务载体，目标不是做真实健身 App。文档在 `找工作啊啊啊啊/项目梳理/个人健康管理agent/`，共七章。

### 七章结构（每章 = `1.总览` + `2.流程图.canvas` + 概念笔记 + `困难以及应对办法`）
1. **1.Harness 控制平面与任务状态编排** — Conversation ≠ Task；四层状态分层。
2. **2.上下文工程与能力渐进装载** — Runtime State ≠ Model Context；上下文装配器、ContextPolicy、状态栏、程序化派生、KV Cache。
3. **3. 长期记忆、知识 RAG 与证据装配** — Task State ≠ 长期用户事实/外部知识；三层数据模型、证据检索与装配。
4. **4. Tool Runtime 与安全** — LLM Tool Call ≠ 真实业务动作；工具面、统一工具运行时、护栏、可靠执行、ExecutionRecord（Attempt 级）。
5. **5.多智能体协作与异步任务执行** — 单领域执行 ≠ 跨领域整体编排；会话主管（Conversation Supervisor）+ 领域智能体 + DomainTask/Artifact 协议。
6. **6.生产运行时治理** — Request ≠ Task 资源成本；任务预算/准入/有界队列/背压/服务级熔断/轻量监控。
7. **第七章（测试与评估）** — 最终回答质量 ≠ 执行链路正确性；统一观测与执行轨迹、分层评估、Rubric 与 LLM-as-Judge、Regression。

### 关键跨章数据模型（贯穿全书，别重复造）
`run_id`（Task 一次连续执行阶段，Resume 生成新 run_id；同步 DomainTask 嵌套父 Run 无独立 run_id，仅独立异步 DomainTask 才有自有 run_id/trace_id）、`ContextManifest`（含 `context_policy_version`，`evidence_refs` 只存 `Evidence.evidence_id` 单一身份）、`RetrievalRecord`、`Evidence`（`evidence_id` 与 `source_id` 分离）、`EvidenceBundle.bundle_id`、`ToolInvocation`（`created_run_id` 跨 Run 稳定）、`caller`（统一，不用 caller_agent）。

### 当前状态（截至 2026-08-31）
七章正文**已全部整理完**。之后启动了**「叙事骨架审核、剪枝与重构」**，目标是把七章压成一条总主线 + 七个「工程问题」+ Hero Scenario（早晨训练→工作日只能晚上训练+调整本周训练与饮食），只保留三条统一设计原则。重构设计草稿在 **`/home/kbzz1/obsidian_workspace/docs/agent_review/`**（`p0_核心骨架.md` 已定骨架，`ch1~ch7_重构设计.md` 各章草稿，`项目总览_v1.md`）。**下一步是据此重写 vault 里的 `项目总览.md` 并重构七章正文**——这是接手后最先要接续的主线工作。

## 5. 项目 B：闪卡 App（工程交付，独立于项目 A）

闪卡（shanka）flashcard App：FastAPI 后端（Python 3.12，conda env `shanka-backend`，仓库 `~/shanka_backend`）+ Android Compose 前端。详见实体记忆 `viking://user/default/memories/entities/项目/闪卡app.md`。要点：
- 后端仓库 `main/` 是唯一主线（v25 线已并入，aod 实验线已废弃删除，worktree 从 3 收敛到 1）。
- 前端交付形态是**可安装的 release APK**（`release.keystore`，CN=Shanka FlashCards，正式地址 `https://shanka.kbzz1.top` 经 Cloudflare Tunnel 回源），要求真机装机验证。
- 后端 2026-08-29 有「每日学习计划 V2.5」在途改动（47 项未提交，24 failed/825 passed），**用户拍板后端不提交不推送、只推前端**到 `KBzz1/Shanka_fork`。
- 已知敏感：`release.keystore` 签名密钥当前环境缺失；曾有一次 storePassword 泄漏到日志，建议轮换。

## 6. 写作与整理约定（必读，决定你所有输出）

- **英文克制三档**（2026-08-26 定，重要）：行业高频/标准名词保留英文（LLM、RAG、OCR、JWT、LangGraph、SQL、JSON、pgvector、DB、HTTP、ReAct）；生造/抽象概念用中文、英文最多括号备注一次（情节记忆/语义记忆/工具面/工具运行时/上下文装配器/会话主管）；字段名/枚举/模型名/数据集名保留原样（`user_id`、MOMENT、MIMIC-IV-ECG）。用户明确说过「不要用那么多英文」，连系统对象名也中文化。
- **单一事实来源 + 链接**：核心概念只在唯一笔记定义（canonical home），其他笔记用 `[[wikilink]]` 引用，不重复解释；短链接用裸文件名（`[[3.工具能力设计与 Tool Surface]]`），不写完整路径前缀。
- **章节编号**：`1.总览`、`2.流程图` 固定，概念笔记从 3 起按阅读顺序「数字+.」编号，「困难以及应对办法」排最后。
- **去 AI 味**：用 `humanizer-zh` skill（`/home/kbzz1/obsidian_workspace/.dsh/skills/humanizer-zh/`，vault 的 `copilot/skills/humanizer-zh` 是 git 版本化副本）；删空泛升华、模板连接词、机械三段式、「本质上/核心是」类强调、碎片条目、聊天残留；保留事实/数字/术语/代码/链接。
- **机制优先**：解释靠机制和原因，不靠换词复述；概念首次出现先讲清「是什么/为何存在/起什么作用」；边界问答用「Q：/A：」序号问答对收在笔记末尾。
- **Canvas 设计**：流程图面向「整体状态流转」理解，不是知识地图/文件排列；主线主数据流从左到右，辅助区上下放，颜色分层，避免大量交叉线和单一竖排；实体节点用主体名（Harness Router、Task Controller）不堆字段名；避免大量英文。
- **周报**：中文、克制直接、去 AI 味；项目是语法主语省略人称；每项「一句结论→截图→机制」；见 `weekly-report-zh` skill 与偏好记忆 `weekly_report_style.md`。
- **改动收敛**：不主动实现用户未要求的功能（如否决「新建个人资料页」）；试错临时文件清理干净；业务逻辑不确定的点先问用户。

## 7. 需要注意的坑与风险

- **vault 只读**：bash 直接写 `/mnt/c` 会 EROFS，务必走 Obsidian MCP（或经用户授权 `danger-full-access`）。
- **Obsidian MCP 视图与磁盘**：2026-08-26 曾观察到 MCP 视图与 `/mnt/c` 磁盘不一致；同步脚本确认 `/mnt/c/.../KB_310` 是 canonical vault。改前先 `vault_list`/`vault_get_document_map` 核对。
- **疑似泄露 token**：`个人健康管理agent/5.多智能体协作与异步任务执行/8.跨领域协作与冲突.md` 第 100 行附近有疑似 Bearer token（`Bearer 72da77c...`），全库仅此一处，**重构时删除**。
- **不要虚构**：不编造用户规模/并发量/线上事故/性能数字；区分「已实现/部分实现/雏形/已定稿设计未实现/未来改进」（本机无真实代码仓库，文档内嵌代码事实是主要实现证据）。
- **秘密不落文档**：不在笔记/日志记录 API Key、Token、keystore 密码。

## 8. 接手后的建议顺序

1. 读本文档 + 注入的 `<openviking-context>`。
2. 读 `viking://user/default/memories/entities/项目/个人健康管理agent.md` 与 `闪卡app.md` 拿最新状态。
3. 读 `docs/agent_review/p0_核心骨架.md`，确认叙事重构的骨架与下一步。
4. 用 `vault_list`/`vault_read` 打开 `项目总览.md` 与各章 `1.总览.md`，开始接续重构。
5. 动任何 vault 文件前确认同步守护进程是否在跑；不在则先拉起 `obsidian-ov-sync.sh`。
