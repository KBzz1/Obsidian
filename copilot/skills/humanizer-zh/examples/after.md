# 改写后：可直接落盘的概念笔记

---
tags:
  - 项目/个人健康管理agent
  - 第一章/控制平面
章节: 1
类型: 概念笔记
核心概念: Checkpoint / 中断恢复
状态: 已整理
---

> [!info] 概念归属
> 本笔记是「Checkpoint 恢复语义」的唯一归属地：恢复位置如何确定、中断后从哪里继续。并发下的版本控制见 [[版本控制与并发]]。

# Checkpoint 与中断恢复

Checkpoint 是任务执行状态的快照。每个节点执行结束后，状态写入 LangGraph Checkpoint；恢复时不重新执行已完成节点，而是从中断位置的 Checkpoint 继续。

## 恢复位置

恢复位置指恢复执行时从哪个检查点继续：任务进入 `WAITING_HUMAN` 状态时，恢复位置与这次中断的 Checkpoint 绑定。恢复由卡片动作驱动：用户点击确认卡片后，任务控制器先按 `task_id` 找到对应的任务，再凭 `task_thread_id` 读出这个任务自己的 Checkpoint。

两个 id 的分工：`task_id` 是 Harness 用来识别任务的业务身份，前端卡片和任务控制器都靠它确认正在操作哪个任务；`task_thread_id` 是这个任务自己的执行线程标识，LangGraph 的 Checkpoint 按它归档，所以定位恢复位置时要用它。

```python
class ResumeRequest(BaseModel):
    task_id: str          # 要恢复的任务（控制平面身份，卡片和任务控制器按它索引）
    task_thread_id: str   # 任务自己的执行线索，Checkpoint 按它归档
    action_id: str        # 用户点击的动作类型，直接交给图的 resume
```

```text
WAITING_HUMAN
   ↓ resume
RUNNING
```

## 与 HIL 的关系

`WAITING_HUMAN` 是持久化的稳定交互点，进入该状态前恢复所需状态已经落盘。中断触发的协议细节见 [[HIL 与恢复#Interrupt 协议]]。

## 待确认

- 恢复时“不重新执行已完成节点”是否同样跳过发送类副作用节点，原文未明确。

## 边界问答

**1. Q：** `task_id` 和 `task_thread_id` 可以合并成一个 id 吗？

**A：** 不能直接合并。`task_id` 是控制平面的任务身份，卡片和任务控制器都按它索引；`task_thread_id` 是任务自己的执行线索，LangGraph 检查点按它归档。恢复流程里先按 `task_id` 找到任务、再按 `task_thread_id` 读检查点，合并会让控制索引与执行线索耦合在一起。

**2. Q：** 检查点已经记录了执行位置，恢复时会不会重复执行已完成节点？

**A：** 恢复时不重新执行已完成节点，而是从中断位置的检查点继续；但发送类副作用节点是否同样跳过，原文未明确，见 [[#待确认]]。
