# 音频系统 (Audio System)

> **Status**: Designed (pending review)
> **Author**: [user + agents]
> **Last Updated**: 2026-04-23
> **Implements Pillar**: 节奏是轻松的

## Overview

音频系统是《地下排练室》的**音频播放与管理中枢**，负责背景音乐（BGM）、音效（SFX）和环境音（Ambient）的播放、切换、混音和音量控制。它是"节奏是轻松的"支柱的感官延伸——即使玩家没有刻意注意，音频也在潜意识层面塑造着"地下排练室"的氛围感。

**系统边界**：
- **包含**：音频播放接口、BGM 管理、SFX 管理、音量控制、音频 Bus 架构、场景音频切换
- **不包含**：具体的音频文件（属于音频资产）、音频编辑/制作流程、实时音频合成（MVP 不实现）

**MVP 策略**：MVP 使用**占位音频**——简单的循环节拍、合成音效、环境噪声。重点是建立正确的音频架构，让后续替换高质量音频时不需要改动代码。

**玩家如何感知**：
- 主界面：低沉的吉他扫弦循环，像有人在不远的角落 tuning 吉他
- 排练场景：节奏感更强的鼓点节拍，暗示"正在发生"的练习
- 事件场景：根据事件情绪切换——争执事件加入失真噪音，和解事件加入温暖的和弦
- 演出场景：观众低语声 + 舞台准备声，营造紧张感
- 转场：简短的音效（翻纸声、橡皮筋轻弹）

**为什么必须存在**：
没有音频，游戏世界就是"死"的。放置游戏尤其依赖音频来填补"玩家不在时"的时间感——当你回来听到不同的 BGM，你知道世界在你离开的时候发生了变化。

---

## Player Fantasy

音频系统没有直接的玩家幻想——玩家感受到的是它营造的氛围：

> 你打开游戏，一阵低沉而略带失真的吉他声飘出来。不是精心制作的游戏音乐，而是像真的有人在隔壁房间排练——你能听到弦音中的生涩和热忱。你点开一个争执事件，背景音突然变得刺耳，像有人把音箱音量调到了失真边缘。当你选择"和解"选项后，噪音慢慢消退，回到那个温暖的、略带慵懒的排练室氛围。

**核心情感**：**"这里真的有人在搞音乐"**。音频不应该是"完美的游戏配乐"，而应该是"真实的地下乐队声音"——有瑕疵、有情绪、有态度。

**参考体验**：
- 《Nana》（动漫）的配乐——摇滚的生涩和激情并存
- 《海盗电台》（电影）——地下电台的嘈杂和浪漫
- 《星露谷物语》的 BGM 切换——每个场景有独特的情绪色彩

---

## Detailed Design

### Core Rules

#### 1. 音频分类体系

| 音频类型 | 说明 | 同时播放数 | MVP 数量 | 音频格式 |
|----------|------|-----------|----------|----------|
| **BGM (背景音乐)** | 场景主题音乐，循环播放 | 1 | 4-5 首 | OGG (压缩率高，适合循环) |
| **SFX (音效)** | 短暂的一次性音效 | 无限制 | 10-15 个 | WAV (低延迟播放) |
| **Ambient (环境音)** | 持续的环境噪声 | 1-2 | 2-3 个 | OGG |
| **UI Sound (UI 音效)** | 按钮点击、切换反馈 | 无限制 | 5-8 个 | WAV |

#### 2. 音频 Bus 架构

采用分层 Bus 架构，支持独立音量控制和效果处理：

```
Master Bus (主总线)
├── BGM Bus (背景音乐)
│   └── 效果器: 可选的轻微 Reverb
├── SFX Bus (音效)
│   └── 效果器: 可选的轻微 Distortion (垃圾摇滚风格)
├── Ambient Bus (环境音)
│   └── 效果器: Low-pass Filter
└── UI Bus (UI 音效)
    └── 效果器: 无
```

**Bus 用途**：
- 独立音量控制：玩家可以在设置中分别调节 BGM 和 SFX 音量
- 统一效果处理：如所有 SFX 添加轻微失真，强化垃圾摇滚风格
- 音频隔离：暂停 BGM 时不影响 SFX 播放

#### 3. BGM 管理

**BGM 播放规则**：
- 每个场景有默认 BGM（见下方 BGM 清单）
- BGM 切换时：当前 BGM 淡出（0.5-1.0 秒）→ 新 BGM 淡入（0.5-1.0 秒）
- BGM 循环播放，无缝循环（音频文件首尾需处理）
- 特殊事件可临时覆盖场景 BGM（如演出时覆盖为演出专属 BGM）

**BGM 状态**：
```gdscript
enum BGMState {
    STOPPED,      # 停止
    FADE_IN,      # 淡入中
    PLAYING,      # 正常播放
    FADE_OUT,     # 淡出中
    PAUSED        # 暂停（游戏进入后台）
}
```

**MVP BGM 清单**：

| BGM ID | 场景 | 风格描述 | 情绪 | 循环 |
|--------|------|----------|------|------|
| `bgm_main_dashboard` | 主界面 | 慵懒的吉他扫弦，像有人在角落 tuning | 放松、期待 | 是 |
| `bgm_rehearsal` | 排练计划/排练中 | 节奏感鼓点 + 低音吉他，暗示"正在练习" | 专注、进行 | 是 |
| `bgm_event_positive` | 正面事件 | 温暖的和弦进行，略带希望感 | 温馨、感动 | 否（一次性） |
| `bgm_event_negative` | 负面事件 | 失真吉他噪音 + 不和谐和弦 | 紧张、冲突 | 否（一次性） |
| `bgm_performance` | 演出场景 | 观众低语 + 舞台准备声，逐渐加强 | 紧张、兴奋 | 是 |

**占位音频策略（MVP）**：
- BGM 使用简单的合成器循环或免版税音乐片段
- 长度：每首 30-60 秒，循环播放
- 音质：不需要高保真，但需要有明确的旋律/节奏轮廓

#### 4. SFX 管理

**SFX 播放规则**：
- SFX 为一次性播放，不循环
- 可同时播放多个 SFX（受限于音频通道数）
- 同一 SFX 在短时间内（0.1 秒）多次触发，只播放一次（防止按键连击导致的爆音）

**MVP SFX 清单**：

| SFX ID | 触发场景 | 风格描述 | 优先级 |
|--------|----------|----------|--------|
| `sfx_ui_click` | 任何 UI 按钮点击 | 轻微的纸张摩擦声 | 低 |
| `sfx_ui_back` | 返回操作 | 翻纸声 | 低 |
| `sfx_ui_confirm` | 确认/保存 | 吉他拨弦确认音 | 中 |
| `sfx_transition_fade` | FADE 转场 | 环境音渐弱 | 中 |
| `sfx_transition_slide` | SLIDE 转场 | 翻纸声 | 中 |
| `sfx_mood_up` | 心情提升 | 轻快的吉他拨弦 | 中 |
| `sfx_mood_down` | 心情下降 | 走调的音符 | 中 |
| `sfx_skill_growth` | 技能成长 | 轻微的鼓点递进 | 中 |
| `sfx_event_trigger` | 事件触发 | 通知铃声（生锈的铃铛声） | 高 |
| `sfx_unlock` | 内容解锁 | 短暂的和弦爆发 | 高 |
| `sfx_chemistry_trigger` | 配合技触发 | 铁锈红色火花声（失真噪音） | 高 |
| `sfx_performance_start` | 演出开始 | 观众鼓掌 + 舞台灯光声 | 高 |
| `sfx_performance_end` | 演出结束 | 观众反应（根据结果变化） | 高 |

**占位音频策略（MVP）**：
- SFX 使用简单的合成音效或免版税音效片段
- 时长：0.1-2 秒
- 音质：不需要高保真，但需要有明确的识别度

#### 5. 环境音（Ambient）

**Ambient 播放规则**：
- 环境音在特定场景下持续播放，与 BGM 混合
- 音量通常低于 BGM（20-30%）
- 场景切换时，环境音随 BGM 一起淡入淡出

**MVP Ambient 清单**：

| Ambient ID | 场景 | 描述 |
|------------|------|------|
| `ambient_basement` | 排练室相关场景 | 轻微的回声、远处的水滴声、管道震动声 |
| `ambient_crowd` | 演出相关场景 | 观众低语、脚步声、舞台设备声 |

#### 6. 音量控制系统

**音量层级**：

```gdscript
# 音量范围：0.0 - 1.0
master_volume = 1.0   # 主音量，影响所有音频
bgm_volume = 0.7      # BGM 相对音量
sfx_volume = 1.0      # SFX 相对音量
ambient_volume = 0.3  # 环境音相对音量
ui_volume = 0.8       # UI 音效相对音量
```

**实际音量计算**：
```
实际 BGM 音量 = master_volume × bgm_volume
实际 SFX 音量 = master_volume × sfx_volume
```

**静音规则**：
- 游戏进入后台时：BGM 和 Ambient 暂停，SFX 和 UI 音效继续（如果触发）
- 玩家手动静音：所有音频停止，但保留播放状态（恢复时继续）
- 系统电话/通知打断：临时降低音量 50%，打断结束后恢复

#### 7. 音频切换接口

```gdscript
# 播放 BGM（带淡入淡出）
AudioManager.play_bgm(bgm_id, fade_duration)

# 停止 BGM（带淡出）
AudioManager.stop_bgm(fade_duration)

# 播放 SFX
AudioManager.play_sfx(sfx_id, volume_override)

# 播放 Ambient
AudioManager.play_ambient(ambient_id, fade_duration)

# 停止 Ambient
AudioManager.stop_ambient(fade_duration)

# 设置音量
AudioManager.set_volume(bus_name, volume)

# 暂停所有音频（进入后台）
AudioManager.pause_all()

# 恢复所有音频（回到前台）
AudioManager.resume_all()
```

### States and Transitions

音频系统本身的状态：

| 状态 | 说明 | 触发条件 |
|------|------|----------|
| **IDLE** | 无音频播放 | 游戏启动前或所有音频停止 |
| **PLAYING** | 正常播放中 | 有 BGM 或 Ambient 正在播放 |
| **FADING** | 淡入/淡出进行中 | BGM 切换或音量调整 |
| **PAUSED** | 音频暂停 | 游戏进入后台或玩家静音 |
| **MUTED** | 手动静音 | 玩家在设置中选择静音 |

```
游戏启动
    │
    ▼
┌─────────┐    播放请求      ┌─────────┐    切换BGM       ┌─────────┐
│  IDLE   │ ───────────────► │ PLAYING │ ──────────────► │ FADING  │
└────┬────┘                  └────┬────┘                 └────┬────┘
     │                            │                           │
     │ 游戏进入后台                │ 恢复                      │ 淡入/淡出完成
     ▼                            ▼                           ▼
┌─────────┐◄───────────────────┐  │                      ┌─────────┐
│ PAUSED  │                    │  │                      │ PLAYING │
└─────────┘                    │  │                      └─────────┘
                               │  │
     设置静音                   │  │
     ▼                         │  │
┌─────────┐                    │  │
│ MUTED   │◄───────────────────┘  │
└─────────┘                       │
     │                            │
     │ 取消静音                    │
     └────────────────────────────┘
```

### Interactions with Other Systems

**输入（本系统接收的数据）**：

| 来源系统 | 输入数据 | 作用 |
|----------|----------|------|
| 场景管理器 | 场景切换事件 | 切换 BGM 和 Ambient |
| 设置菜单UI | 音量设置变更 | 调整各 Bus 音量 |
| 事件模板系统 | 事件触发类型 | 播放对应情绪的 BGM/SFX |
| 角色属性系统 | 状态变化事件 | 播放心情/精力变化 SFX |
| 内容数据库 | 内容解锁事件 | 播放解锁 SFX |
| 资源加载系统 | 音频资源加载完成 | 开始播放 |

**输出（本系统提供的数据）**：

| 目标系统 | 输出数据 | 作用 |
|----------|----------|------|
| 设置菜单UI | 当前音量值 | 显示在设置界面 |
| 存档/加载系统 | 音量设置 | 保存玩家偏好 |
| 场景管理器 | 音频切换完成信号 | 转场动画同步 |

---

## Formulas

### 1. 实际音量计算公式 (Effective Volume)

The effective volume formula is defined as:

`effective_volume = master_volume × bus_volume × override_volume`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| master_volume | master | float | 0.0-1.0 | 主音量设置 |
| bus_volume | bus | float | 0.0-1.0 | 该音频所在 Bus 的音量 |
| override_volume | override | float | 0.0-2.0 | 播放时的临时音量覆盖（用于强调效果） |

**Output Range:** 0.0 到 2.0（超过 1.0 时可能失真，用于特殊效果）
**Example:** master=0.8, bgm_bus=0.7, override=1.0 → `effective = 0.8 × 0.7 × 1.0 = 0.56`

### 2. 淡入淡出时间公式 (Fade Duration)

The fade duration formula is defined as:

`fade_duration = base_duration × urgency_multiplier`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| base_duration | base | float | 0.5-1.0 | 基础淡入淡出时间（秒） |
| urgency_multiplier | urgency | float | 0.5-2.0 | 紧急程度倍率 |

**场景倍率**：

| 场景 | urgency_multiplier |
|------|-------------------|
| 正常场景切换 | 1.0 |
| 事件触发（需要立即引起注意） | 0.5 |
| 演出开始/结束（仪式感） | 2.0 |

**Output Range:** 0.25 到 2.0 秒
**Example:** 正常场景切换 → `duration = 0.8 × 1.0 = 0.8` 秒

### 3. BGM 情绪强度公式 (BGM Intensity)

用于动态调整 BGM 的强度层（如从简单的吉他扫弦加入鼓点和贝斯）：

`intensity = base_intensity + event_modifier + time_modifier`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| base_intensity | base | float | 0.0-1.0 | 基础强度（根据场景） |
| event_modifier | event | float | -0.3-0.3 | 事件情绪修正（正面事件 +0.3，负面事件 -0.3） |
| time_modifier | time | float | 0.0-0.2 | 时间修正（深夜 +0.1，营造安静氛围） |

**Output Range:** 0.0 到 1.5（超过 1.0 时触发更激烈的 BGM 变奏）
**Example:** 排练场景 base=0.5，无特殊事件 → `intensity = 0.5 + 0 + 0 = 0.5`

**MVP 简化**：MVP 不实现动态强度切换，每首 BGM 固定强度。Post-MVP 考虑分层 BGM 系统。

---

## Edge Cases

- **If BGM 资源加载失败**：静默处理——不播放 BGM，不报错，不阻塞游戏。记录警告日志。
- **If SFX 在短时间内（0.1 秒）被连续触发多次**：去重处理——只播放第一次，后续触发忽略。防止 UI 连击导致的爆音。
- **If 游戏进入后台时正在播放 BGM**：暂停所有音频，游戏回到前台后自动恢复。
- **If 设备静音模式开启**（iOS/Android 物理静音开关）：尊重系统设置，所有音频静音。但 UI 点击音效仍可播放（可选，由设置控制）。
- **If 电话/通知打断游戏**：临时降低所有音频音量到 20%，打断结束后 2 秒内恢复到原音量。
- **If BGM 切换请求到达时前一首 BGM 还在淡入**：中断当前淡入，立即开始淡出，然后淡入新 BGM。
- **If 玩家同时触发多个高优先级 SFX**（如解锁 + 事件触发）：按优先级排序播放，或同时播放（如果音频通道允许）。
- **If 音频文件损坏或格式不支持**：跳过该音频，使用默认静音，记录错误日志。
- **If 内存不足导致音频无法加载**：优先保留 BGM，释放不常用的 SFX 资源。关键 SFX（如 UI 反馈）保留。
- **If 玩家调整音量时音频正在播放**：实时生效，不需要重新加载音频。

---

## Dependencies

**上游依赖**：
| 系统 | 依赖数据 | 必要性 |
|------|----------|--------|
| 资源加载系统 | 音频文件加载 | 软依赖 — 音频系统通过资源加载系统获取音频文件 |
| 存档/加载系统 | 音量设置持久化 | 软依赖 — 音量设置保存到存档中 |

**下游被依赖**：
| 系统 | 依赖数据 | 数据流向 |
|------|----------|----------|
| 场景管理器 | BGM 切换触发 | Scene Manager → Audio System |
| 设置菜单UI | 音量调节接口 | Settings UI ↔ Audio System |
| 所有触发音效的系统 | SFX 播放接口 | Gameplay Systems → Audio System |
| 存档/加载系统 | 音量设置数据 | Audio System → Save/Load |

---

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 过低效果 | 过高效果 | 关联 |
|--------|--------|----------|----------|----------|------|
| `master_volume_default` | 0.8 | 0.5-1.0 | 整体音量太小 | 默认音量过大 | 用户体验 |
| `bgm_volume_default` | 0.7 | 0.3-1.0 | BGM 几乎听不见 | BGM 盖过 SFX | 平衡 |
| `sfx_volume_default` | 1.0 | 0.5-1.0 | 音效反馈微弱 | 音效刺耳 | 反馈 |
| `ambient_volume_default` | 0.3 | 0.1-0.5 | 环境无氛围 | 环境音太吵 | 氛围 |
| `fade_duration_default` | 0.8 | 0.3-2.0 | 切换突兀 | 切换拖沓 | 体验 |
| `sfx_debounce_time` | 0.1 | 0.05-0.3 | 连击爆音 | 正常快速操作被过滤 | 稳定性 |
| `bgm_loop_crossfade` | 0.0 | 0.0-0.5 | 循环可能有明显断点 | 增加 CPU 开销 | 音质 |
| `audio_channel_max` | 16 | 8-32 | 复杂场景音频被截断 | 过多通道占用资源 | 性能 |

---

## Visual/Audio Requirements

音频系统**定义音频资产的规格要求**：

**BGM 规格**：
- 格式：OGG Vorbis，128-192 kbps
- 长度：30-60 秒（循环播放）
- 循环点：首尾需无缝衔接
- 风格：低保真、略带噪音——像从旧磁带转录的排练录音

**SFX 规格**：
- 格式：WAV 16-bit / OGG
- 长度：0.1-2 秒
- 风格：避免过于"游戏化"的音效——用真实乐器声、环境声替代合成音

**Ambient 规格**：
- 格式：OGG Vorbis，96-128 kbps
- 长度：10-30 秒（循环播放）
- 风格：低音量、持续、不抢戏

**垃圾摇滚音频风格指南**：
- 允许轻微的失真和噪音——这不是瑕疵，是风格
- 乐器音色调制：吉他略带过载，鼓声不追求"完美"的 studio 质感
- 混响：排练室应有明显的 room reverb，像真的在地下室
- 动态范围：不需要过度压缩，保留自然的音量起伏

---

## UI Requirements

音频系统**不直接提供 UI**，但支持设置菜单的音量控制：

**设置菜单音量面板**（由设置菜单UI实现）：
- Master 音量滑块（0-100%）
- BGM 音量滑块（0-100%）
- SFX 音量滑块（0-100%）
- 静音开关
- 滑动时实时预览音量变化

**音量变化反馈**：
- 调整 SFX 音量时播放 `sfx_ui_click` 作为预览
- 调整 BGM 音量时 BGM 实时响应

---

## Acceptance Criteria

- **GIVEN** 主场景 `main_dashboard`，**WHEN** 场景切换完成，**THEN** 播放 `bgm_main_dashboard`
- **GIVEN** BGM 正在播放，**WHEN** 切换到排练场景，**THEN** 当前 BGM 淡出 0.8 秒，新 BGM 淡入 0.8 秒
- **GIVEN** 玩家点击 UI 按钮，**WHEN** 触发 `sfx_ui_click`，**THEN** 播放音效，音量 = master × sfx_volume
- **GIVEN** `sfx_ui_click` 在 0.1 秒内被触发 3 次，**WHEN** 去重处理，**THEN** 只播放 1 次
- **GIVEN** master_volume = 0.8, bgm_volume = 0.7，**WHEN** 计算实际 BGM 音量，**THEN** 结果为 0.56
- **GIVEN** 游戏进入后台，**WHEN** 音频系统收到暂停信号，**THEN** BGM 和 Ambient 暂停，SFX 继续
- **GIVEN** 游戏从后台回到前台，**WHEN** 音频系统收到恢复信号，**THEN** 所有暂停的音频恢复播放
- **GIVEN** BGM 资源加载失败，**WHEN** 尝试播放，**THEN** 静默跳过，不报错，记录警告
- **GIVEN** 玩家调整音量滑块，**WHEN** 音量值变化，**THEN** 正在播放的音频实时响应新音量
- **GIVEN** 设备进入静音模式，**WHEN** 检测系统静音状态，**THEN** 所有音频静音（除非设置覆盖）

---

## Open Questions

- **Q1**: MVP 的占位音频是否全部使用免版税资源？还是需要简单的原创合成？
- **Q2**: 是否需要实现动态 BGM 分层系统（如根据游戏进度切换 BGM 的配器）？MVP 暂不实现，但架构是否需要预留？
- **Q3**: 音频是否需要支持多语言（如不同语言的配音）？当前设计假设无语音，仅有音乐和音效。
- **Q4**: 演出场景是否需要实现"掌声强度根据演出分数变化"？这需要演出判定系统提供分数数据来驱动音频。
- **Q5**: 是否需要实现音频的可访问性功能（如视觉化音频提示，帮助听障玩家）？
