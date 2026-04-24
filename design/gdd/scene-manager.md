# 场景管理器 (Scene Manager)

> **Status**: Designed (pending review)
> **Author**: [user + agents]
> **Last Updated**: 2026-04-23
> **Implements Pillar**: 节奏是轻松的

## Overview

场景管理器是《地下排练室》的**场景切换与状态保持中枢**，负责管理游戏内所有场景的加载、卸载、转场动画和场景间数据传递。它是玩家在游戏世界中"移动"的基础设施——从主界面到排练计划、从排练计划到叙事事件、从叙事事件回到主界面——所有场景切换都由场景管理器统一调度。

**系统边界**：
- **包含**：场景注册表、场景切换逻辑、转场动画、场景生命周期管理、场景间数据传递接口
- **不包含**：具体场景的内部逻辑（属于各UI/玩法系统）、资源加载细节（属于资源加载系统）、存档读写（属于存档/加载系统）

**玩家如何感知**：
- 玩家点击"去排练" → 屏幕淡出 → 排练室场景淡入
- 玩家处理完事件点击"返回" → 事件界面滑出 → 主界面恢复
- 转场动画的风格符合垃圾摇滚美学——像翻阅一本旧日记，不是生硬的切屏

**为什么必须存在**：
没有场景管理器，各个界面之间无法有序切换，游戏将变成一团混乱的叠加面板。它是"节奏是轻松的"支柱的体验保障——每一次转场都应该流畅、不突兀、有风格。

---

## Player Fantasy

场景管理器没有直接的玩家幻想——玩家感受到的是它创造的效果：

> 你点击"排练计划"，当前的地下排练室画面像被风吹散的传单一样碎裂、飘走，然后排练计划的界面像从另一个口袋掏出来的皱巴巴的笔记本一样展开。你没有"切换场景"的感觉——你只是在乐队的世界里从一个房间走到了另一个房间。

**核心情感**：**"流畅感"**。转场不应该是"等待加载"的空白——它应该是叙事的一部分。垃圾摇滚的粗犷不等于粗糙，我们的转场要在混乱中有一种刻意的节奏。

**参考体验**：
- 《星露谷物语》的场景切换——黑屏+环境音效，简洁但有仪式感
- 《塞尔达传说》的地图切换——标志性的缩放转场，每一次都知道"我要去哪里"

---

## Detailed Design

### Core Rules

#### 1. 场景分类体系

所有场景分为三类：

| 场景类型 | 说明 | MVP 场景数量 | 切换行为 |
|----------|------|-------------|----------|
| **主场景 (Main Scene)** | 游戏的核心常驻场景 | 1 | 不卸载，其他场景在它之上叠加 |
| **子场景 (Sub Scene)** | 从主场景进入的功能场景 | 4 | 堆叠式切换，保留返回路径 |
| **覆盖层 (Overlay)** | 弹窗、提示、菜单 | 3+ | 在当前场景之上叠加，不影响下层 |

#### 2. MVP 场景清单

| 场景ID | 场景名称 | 类型 | 说明 |
|--------|----------|------|------|
| `boot` | 启动画面 | 子场景 | 游戏启动时的加载画面，显示 Logo 和加载进度 |
| `main_dashboard` | 主界面/仪表盘 | 主场景 | 核心场景——显示角色状态、排练计划入口、事件通知 |
| `rehearsal_planning` | 排练计划 | 子场景 | 设置排练曲目、分配队员、设定重点 |
| `narrative_event` | 叙事事件 | 子场景 | 展示和处理触发的叙事事件 |
| `performance` | 演出 | 子场景 | 演出准备、演出过程、演出结果 |
| `settings_menu` | 设置菜单 | 覆盖层 | 音量、通知、存档管理 |
| `event_detail` | 事件详情 | 覆盖层 | 主界面上弹出的事件详情面板 |
| `unlock_notification` | 解锁提示 | 覆盖层 | 内容解锁时的浮动提示 |

#### 3. 场景切换规则

**切换类型**：

| 切换类型 | 视觉效果 | 使用场景 | 数据行为 |
|----------|----------|----------|----------|
| `FADE` | 淡出(0.3s) → 切换 → 淡入(0.3s) | 主场景↔子场景 | 下层场景保持运行 |
| `SLIDE` | 新场景从右滑入(0.4s)，旧场景向左滑出 | 子场景之间 | 旧场景暂停，保留状态 |
| `POP` | 覆盖层从底部/中心弹出(0.2s) | 打开覆盖层 | 下层场景保持运行 |
| `DISSOLVE` | 像素化溶解过渡(0.5s) | 特殊时刻（首次演出等） | 可选全屏切换 |
| `INSTANT` | 无动画，立即切换 | 调试、错误恢复 | 立即完成 |

**切换约束**：
- 一次只能进行一个场景切换（切换中拒绝新请求）
- 切换时间上限：1.0 秒（超过则强制完成，记录警告）
- 覆盖层最多叠加 3 层（防止无限嵌套）

#### 4. 场景生命周期

每个场景经历以下生命周期：

```
REGISTERED → LOADING → ENTERING → ACTIVE → PAUSED → EXITING → UNLOADED
```

| 状态 | 说明 | 触发条件 |
|------|------|----------|
| **REGISTERED** | 已在场景注册表中，但未加载 | 游戏初始化时 |
| **LOADING** | 正在加载场景资源 | 触发切换后 |
| **ENTERING** | 场景已加载，正在播放进入动画 | 资源加载完成后 |
| **ACTIVE** | 场景完全可见，接收输入 | 进入动画完成后 |
| **PAUSED** | 场景仍在内存，但被上层场景覆盖 | 有覆盖层或子场景在其上时 |
| **EXITING** | 正在播放退出动画 | 触发离开 |
| **UNLOADED** | 场景已卸载，释放内存 | 退出动画完成后（仅子场景） |

**主场景特殊规则**：
- `main_dashboard` 永远不会进入 `UNLOADED` 状态
- 当子场景 `ACTIVE` 时，主场景进入 `PAUSED`（不更新逻辑，但仍渲染）
- 当所有子场景和覆盖层都关闭后，主场景自动恢复 `ACTIVE`

#### 5. 场景间数据传递

场景管理器提供**数据传递接口**，但不存储业务数据：

```
SceneManager.navigate_to(scene_id, transition_type, context_data)
```

**context_data**：一个 `Dictionary`，包含目标场景需要的初始化数据。

**示例**：
```gdscript
# 从主界面导航到叙事事件场景
SceneManager.navigate_to("narrative_event", "FADE", {
    "event_instance_id": "evt_001",
    "event_template_id": "rel_argument_rehearsal"
})

# 从排练计划返回主界面
SceneManager.navigate_to("main_dashboard", "FADE", {
    "refresh_dashboard": true
})
```

**数据生命周期**：
- `context_data` 只传递给目标场景的 `_on_enter()` 方法
- 场景切换完成后，`context_data` 被清除（不持久化）
- 需要持久化的数据由各个系统自己管理（通过存档/加载系统或全局状态）

#### 6. 场景注册表

场景管理器维护一个注册表，定义所有可用场景：

```gdscript
var scene_registry = {
    "boot": {"path": "res://scenes/boot.tscn", "type": "sub"},
    "main_dashboard": {"path": "res://scenes/main_dashboard.tscn", "type": "main"},
    "rehearsal_planning": {"path": "res://scenes/rehearsal_planning.tscn", "type": "sub"},
    "narrative_event": {"path": "res://scenes/narrative_event.tscn", "type": "sub"},
    "performance": {"path": "res://scenes/performance.tscn", "type": "sub"},
    "settings_menu": {"path": "res://scenes/settings_menu.tscn", "type": "overlay"},
    "event_detail": {"path": "res://scenes/event_detail.tscn", "type": "overlay"},
    "unlock_notification": {"path": "res://scenes/unlock_notification.tscn", "type": "overlay"}
}
```

**动态注册**：系统可以在运行时向注册表添加新场景（如 DLC 场景），但不能修改已注册场景的路径。

#### 7. 返回栈 (Back Stack)

子场景之间维护一个返回栈，支持"返回"操作：

```
main_dashboard → rehearsal_planning → narrative_event
```

- 按"返回"：从 `narrative_event` 返回到 `rehearsal_planning`
- 再按"返回"：从 `rehearsal_planning` 返回到 `main_dashboard`
- 返回主场景后，返回栈清空

**覆盖层不进入返回栈**——关闭覆盖层直接回到它下面的场景。

**返回栈深度上限**：5 层（防止无限嵌套）。超过时，最早的非主场景被移除。

#### 8. 转场动画细节

**FADE 转场**：
- 淡出：当前场景透明度 1.0 → 0.0，0.3 秒，ease-in
- 中间帧：全黑 0.1 秒（可自定义颜色——MVP 用黑色）
- 淡入：新场景透明度 0.0 → 1.0，0.3 秒，ease-out

**SLIDE 转场**：
- 新场景从右侧进入：position.x = +screen_width → 0，0.4 秒，ease-out
- 旧场景向左滑出：position.x = 0 → -screen_width，0.4 秒，ease-in
- 两个动画同时进行

**DISSOLVE 转场（特殊时刻）**：
- 使用 Godot 的 Shader 实现像素化溶解效果
- 像老照片被水渍慢慢侵蚀的感觉
- 只在里程碑事件（首次演出、首次解锁等）使用

### States and Transitions

场景管理器本身的状态：

| 状态 | 说明 | 触发条件 |
|------|------|----------|
| **IDLE** | 无切换进行中 | 初始化完成或上一次切换结束 |
| **TRANSITIONING** | 正在执行场景切换 | 收到 `navigate_to` 请求 |
| **PAUSED** | 切换被暂停 | 游戏进入后台（保存后暂停） |

```
┌─────────┐    navigate_to()     ┌─────────┐    切换完成        ┌─────────┐
│  IDLE    │ ───────────────────► │ TRANSITIONING │ ───────────► │  IDLE    │
└────┬────┘                      └────┬────┘                  └─────────┘
     │                                │
     │ 游戏进入后台                     │ 游戏回到前台
     ▼                                ▼
┌─────────┐                      ┌─────────┐
│ PAUSED   │                      │ PAUSED   │
└─────────┘                      └─────────┘
```

**切换中拒绝新请求**：如果场景管理器处于 `TRANSITIONING` 状态，新的 `navigate_to` 请求被排队（不是丢弃），在当前切换完成后依次执行。

### Interactions with Other Systems

**输入（本系统接收的数据）**：

| 来源系统 | 输入数据 | 作用 |
|----------|----------|------|
| 存档/加载系统 | 加载完成信号 | 从 `boot` 场景切换到 `main_dashboard` |
| 放置排练系统 | "打开排练计划"请求 | 切换到 `rehearsal_planning` 场景 |
| 叙事事件系统 | "有未处理事件"信号 | 切换到 `narrative_event` 场景 |
| 演出判定系统 | "开始演出"请求 | 切换到 `performance` 场景 |
| 进度解锁系统 | "内容解锁"通知 | 在 `main_dashboard` 上弹出 `unlock_notification` 覆盖层 |
| 设置菜单UI | "打开设置"请求 | 弹出 `settings_menu` 覆盖层 |

**输出（本系统提供的数据）**：

| 目标系统 | 输出数据 | 作用 |
|----------|----------|------|
| 所有UI系统 | 当前活跃场景ID | UI 根据当前场景调整显示 |
| 音频系统 | 场景切换事件 | 触发 BGM 切换 |
| 存档/加载系统 | "需要保存"信号 | 场景切换前触发自动存档 |
| 所有系统 | `context_data` 传递 | 场景间传递初始化数据 |

---

## Formulas

### 1. 转场时间公式 (Transition Duration)

The transition duration formula is defined as:

`transition_duration = base_duration × speed_multiplier`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| base_duration | base | float | 0.2-0.5 | 转场类型的基础时长（秒） |
| speed_multiplier | speed | float | 0.5-2.0 | 速度倍率（玩家设置或系统状态） |

**转场类型基础时长**：

| 转场类型 | base_duration |
|----------|--------------|
| FADE | 0.3 |
| SLIDE | 0.4 |
| POP | 0.2 |
| DISSOLVE | 0.5 |
| INSTANT | 0.0 |

**Output Range:** 0.0 到 1.0 秒
**Example:** FADE 转场，speed_multiplier = 1.0 → `duration = 0.3 × 1.0 = 0.3` 秒

**最大时长限制**：`min(transition_duration, max_transition_duration)`，其中 `max_transition_duration = 1.0` 秒。

### 2. 场景加载优先级公式 (Scene Load Priority)

场景加载时，如果有多个场景等待加载，按优先级排序：

`load_priority = type_priority × urgency_multiplier`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| type_priority | type_p | int | 1-10 | 场景类型基础优先级 |
| urgency_multiplier | urgency | float | 1.0-2.0 | 紧急程度倍率 |

**场景类型优先级**：

| 场景类型 | type_priority |
|----------|--------------|
| 主场景 (main) | 10 |
| 覆盖层 (overlay) | 8 |
| 子场景 (sub) | 5 |
| 启动画面 (boot) | 3 |

**Output Range:** 3 到 20
**Example:** 主场景加载，urgency = 1.0 → `priority = 10 × 1.0 = 10`

### 3. 返回栈深度公式 (Back Stack Depth)

```
effective_depth = min(actual_depth, max_stack_depth)
```

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| actual_depth | depth | int | 0-∞ | 当前返回栈实际深度 |
| max_stack_depth | max | int | 5 | 返回栈最大深度 |

**溢出处理**：当 `actual_depth > max_stack_depth` 时，移除栈底（最早）的非主场景。

---

## Edge Cases

- **If 切换过程中再次收到 navigate_to 请求**：请求进入队列，当前切换完成后按 FIFO 顺序执行。不丢弃请求。
- **If 切换时间超过 max_transition_duration (1.0秒)**：强制结束转场动画，立即显示目标场景。记录警告日志。
- **If 目标场景文件不存在或损坏**：显示错误覆盖层"场景加载失败"，提供"返回主界面"按钮。不崩溃。
- **If 返回栈为空时玩家按返回键**（如主界面按返回）：触发"确认退出游戏"弹窗（移动端标准行为）。
- **If 覆盖层已达最大叠加层数 (3层)**：打开新的覆盖层时，自动关闭最旧的覆盖层（不是阻止打开）。
- **If 游戏进入后台时正在转场**：暂停转场动画，游戏回到前台后继续。场景状态保持 `TRANSITIONING`。
- **If 主场景被意外请求卸载**：拒绝操作，记录错误。主场景永远不会被卸载。
- **If context_data 包含无法序列化的数据**（如循环引用）：在传递给目标场景前进行深度拷贝和清理，移除不可序列化的对象。
- **If 连续快速点击多个场景按钮**（如 0.5 秒内点击 3 个不同场景）：只执行第一次有效的切换请求，后续请求进入队列。队列长度上限为 3，超过的请求被丢弃。
- **If 从子场景返回到主场景时，主场景的数据已过期**（如离线收益已计算）：主场景的 `_on_resume()` 方法负责刷新数据。场景管理器只触发恢复信号，不处理数据刷新。
- **If 目标场景加载资源时失败**（如图片缺失）：使用占位资源继续加载，显示"资源加载中..."提示，后台重试加载。

---

## Dependencies

**上游依赖**：
| 系统 | 依赖数据 | 必要性 |
|------|----------|--------|
| 存档/加载系统 | 加载完成信号 | 软依赖 — 用于从启动画面切换到主界面 |
| 资源加载系统 | 场景资源加载 | 软依赖 — 场景管理器调用资源加载系统加载 .tscn 文件 |

**下游被依赖**：
| 系统 | 依赖数据 | 数据流向 |
|------|----------|----------|
| 所有UI系统 | 场景切换接口 | Scene Manager → UI 场景激活/暂停 |
| 音频系统 | 场景切换事件 | Scene Manager → BGM 切换触发 |
| 存档/加载系统 | "切换前保存"信号 | Scene Manager → 触发自动存档 |
| 放置排练系统 | 排练计划场景入口 | Scene Manager → 打开排练计划 |
| 叙事事件系统 | 事件场景入口 | Scene Manager → 打开事件处理 |
| 演出判定系统 | 演出场景入口 | Scene Manager → 打开演出界面 |

---

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 过低效果 | 过高效果 | 关联 |
|--------|--------|----------|----------|----------|------|
| `fade_duration` | 0.3 | 0.1-0.5 | 转场太快，玩家感到突兀 | 转场太慢，玩家等待不耐烦 | 用户体验 |
| `slide_duration` | 0.4 | 0.2-0.6 | 滑动太快，失去流畅感 | 滑动太慢，操作响应迟钝 | 用户体验 |
| `max_stack_depth` | 5 | 3-8 | 返回路径太短，操作受限 | 返回路径太长，玩家迷失 | 导航 |
| `max_overlay_depth` | 3 | 2-5 | 覆盖层太少，功能受限 | 覆盖层太多，界面混乱 | UI |
| `max_transition_duration` | 1.0 | 0.5-2.0 | 强制完成太快，动画跳跃 | 强制完成太慢，卡顿感强 | 稳定性 |
| `transition_queue_size` | 3 | 2-5 | 快速操作容易丢请求 | 队列过长，延迟响应 | 响应性 |
| `enable_dissolve_effect` | true | true/false | 特殊时刻缺少仪式感 | 无影响 | 视觉 |
| `dissolve_color` | 黑色 | — | — | — | 视觉风格 |

---

## Visual/Audio Requirements

**转场动画视觉风格**：

| 转场类型 | 视觉描述 | 音频 |
|----------|----------|------|
| FADE | 像一盏老灯泡慢慢熄灭又亮起——不是纯黑，而是带一点暖黄余烬的暗 | 环境音渐弱 → 短暂静音 → 新场景环境音渐强 |
| SLIDE | 像翻动一沓旧传单——页面从边缘滑入，有轻微的纸张摩擦感 | 翻纸声（轻微，0.1秒） |
| POP | 像从口袋里掏出一张皱巴巴的纸条——弹出的瞬间有轻微的弹性 | 橡皮筋轻弹声 |
| DISSOLVE | 像老照片被水渍慢慢侵蚀——像素化溶解，边缘不规则 | 失真收音机噪音渐变 |

**特殊要求**：
- 转场期间显示一个极简的加载指示器（一枚生锈的回形针在转圈）
- 加载时间超过 0.5 秒时显示"加载中..."文字
- 所有转场动画必须可跳过（点击屏幕任意位置立即完成转场）——放置游戏玩家不耐烦等待

---

## UI Requirements

场景管理器**不直接提供 UI**，但定义了以下 UI 行为：

**返回按钮行为**：
- 子场景左上角显示"←"返回按钮
- 覆盖层显示"×"关闭按钮或"完成"按钮
- 物理返回键（Android）行为：子场景 → 返回上一级；主场景 → 显示退出确认

**转场遮罩**：
- 转场期间全屏遮罩，拦截所有输入
- 遮罩颜色：`dissolve_color`（默认黑色，透明度 0-100% 渐变）

---

## Acceptance Criteria

- **GIVEN** 当前场景为 `main_dashboard`，**WHEN** 调用 `navigate_to("rehearsal_planning", "FADE")`，**THEN** 在 0.3 秒内完成淡入淡出切换
- **GIVEN** 场景管理器处于 `TRANSITIONING` 状态，**WHEN** 收到新的 `navigate_to` 请求，**THEN** 请求进入队列，当前切换完成后执行
- **GIVEN** 返回栈深度为 5（已达上限），**WHEN** 打开新的子场景，**THEN** 移除栈底最早的非主场景，新场景入栈
- **GIVEN** 覆盖层已达 3 层上限，**WHEN** 打开新的覆盖层，**THEN** 自动关闭最旧的覆盖层
- **GIVEN** 目标场景文件不存在，**WHEN** 尝试切换，**THEN** 显示错误覆盖层，提供"返回主界面"按钮
- **GIVEN** 从 `rehearsal_planning` 按返回键，**WHEN** 返回栈中有 `main_dashboard`，**THEN** 返回到主界面，排练计划场景卸载
- **GIVEN** 主场景 `main_dashboard` 被请求卸载，**WHEN** 执行卸载操作，**THEN** 拒绝操作并记录错误
- **GIVEN** 转场动画进行中，**WHEN** 玩家点击屏幕，**THEN** 立即完成转场（跳过剩余动画）
- **GIVEN** 转场时间超过 1.0 秒，**WHEN** 达到上限，**THEN** 强制完成转场，记录警告
- **GIVEN** 游戏进入后台时正在转场，**WHEN** 游戏回到前台，**THEN** 继续转场，场景状态正确

---

## Open Questions

- **Q1**: 是否需要支持场景的"预加载"（如预测玩家下一步操作提前加载场景资源）？这会增加内存占用但减少等待时间。
- **Q2**: 是否需要实现"场景快照"功能——切换到子场景前保存主场景的渲染画面，作为返回时的背景？还是每次都重新渲染主场景？
- **Q3**: DISSOLVE 转场效果需要 Shader 实现，MVP 是否值得投入？还是先用简单的 FADE/SLIDE 替代？
- **Q4**: 是否需要支持横竖屏切换时的场景适配？移动端游戏通常锁定方向，但如果未来支持平板，可能需要考虑。
- **Q5**: 场景切换时的"加载中"提示是否需要有取消按钮？如果加载卡住，玩家是否需要有退出途径？
