# Art Bible: 地下排练室 (Underground Rehearsal Room)

*Created: 2026-04-23*
*Status: Draft — Sections 1-4 complete, 5-9 pending*

---

## 1. Visual Identity Statement

### One-Line Visual Rule

> **"像从垃圾堆里捡来的、被雨水泡过又晒干的演出传单——你几乎看不清上面的字，但你感受得到那股不服输的劲儿。"**

### Supporting Principles

| Principle | Definition | Serves Pillar | Design Test |
|-----------|------------|---------------|-------------|
| **垃圾不是隐喻，是字面意思** | 污渍、划痕、咖啡渍、胶带痕迹、手写字体、复印件的噪点——这不是瑕疵，是生活的痕迹 | 人物是活的 | 如果一个UI元素看起来像是刚从印刷厂出来的完美成品，那它就不对 |
| **反叛是默认状态** | 每一帧都在说"去你的规则"。倾斜的构图、截断的文字、故意的不对齐、反传统的配色——视觉本身就是一种态度 | 成长是可见的 | 如果画面看起来"平衡"或"和谐"，那就打破它 |
| **不完美是最高标准** | 完美是敌人，polish 是犯罪。队员有黑眼圈、衣服皱巴巴、姿势僵硬——因为真实的人就是这样。排练室不是"温馨的"，是"危险的" | 选择是有意义的 | 如果你看到某个元素想"这里应该再修一下"——停下，它就是对的 |

---

## 2. Mood & Atmosphere

### Emotional Targets by Game State

| Game State | Mood Target | Lighting Character | Atmospheric Adjectives | Energy Level |
|------------|-------------|-------------------|------------------------|--------------|
| **Rehearsal Room (Daily/Idle)** | 压抑但充满希望——暴风雨前的宁静 | 昏暗暖黄灯泡，只有一盏灯亮着，阴影吞噬角落；随时间推移灯光逐渐变亮 | 昏暗、拥挤、潮湿、温暖、充满期待 | Restrained but tense |
| **Narrative Events (Story)** | 真实、raw——像偷看某人的日记 | 动态变化：争吵时冷硬刺眼的侧光；和解时暖柔的顶光；深夜谈心时只有手机屏幕冷光 | 原始、私密、紧张、脆弱、真实 | Fluctuating |
| **Performance (Climax)** | 释放、狂喜、失控——从压抑到爆发的瞬间 | 刺眼舞台灯光从黑暗中爆发；高对比度；观众席是黑暗海洋，只有舞台是亮的 | Blinding、deafening（visually）、chaotic、liberating、highlighted | Explosive |
| **Menu/UI** | 像翻阅一本旧日记——每一页都有故事 | 无外部灯光；UI本身就是光源——像在被窝里用手电筒看笔记本 | 陈旧、个人化、拼贴、手写、私密 | Static but characterful |

### Visual Evolution Arc

- **Early (Basement)**: 70% darkness + 30% warm yellow → oppressive but warm
- **Mid (Local Livehouse)**: 50% darkness + 50% warm/cool interplay → struggle and hope coexist
- **Late (Music Festival)**: 20% darkness + 80% blinding light → dream come true, but at what cost?

---

## 3. Shape Language

### Character Silhouette Philosophy

- 队员不是标准身材，而是**真实的、不完美的人**——驼背、姿势随意、衣服总是歪的
- **Distinguishing at thumbnail size**: 通过标志性反叛特征区分
  - 吉他手：永远垂下来遮住眼睛的刘海 + 永远反戴的帽子
  - 贝斯手：oversized 破洞卫衣 + 总是半耷拉的眼皮
  - 鼓手：手臂上的绷带（不是受伤，是风格）+ 夸张的手势
- **Silhouette test**: 遮住脸只看轮廓，能否认出是谁？如果不行，说明反叛特征不够鲜明

### Environment Geometry

- **没有直线**——所有的线都是歪的、倾斜的、手撕的边缘
- 排练室：歪斜的乐器架、用胶带固定住的椅子、贴着天花板垂下来的电线、层层叠叠的海报（新的盖在旧的上面，旧的已经被撕掉一半）
- 演出场地：临时搭建的舞台、晃动的灯光架、人影在烟雾中晃动的不规则形状

### UI Shape Grammar

- UI不是完美的矩形——是从杂志/传单上**剪下来**的不规则形状
- 按钮像手撕的纸片，边缘参差不齐
- 信息面板像拼贴画——不同元素有不同的角度和重叠方式
- 文字不是整齐排列的——有些字大一点、有些小一点、有些斜着

### Hero vs. Supporting Shapes

- **Hero shapes (draw the eye)**: 角色的脸和表情、演出时的舞台灯光、叙事事件中的关键道具
- **Supporting shapes (recede)**: 背景、环境细节、UI边框——它们应该被"弄脏"，但不能抢戏

---

## 4. Color System

### Primary World Palette

| Color | Role | Why This Color |
|-------|------|----------------|
| **Dirty Yellow / Smoke Yellow** | Primary | 像被烟熏黄的排练室墙壁、像老报纸——"生活过的痕迹" |
| **Rust Red** | Accent | 像生锈的管道、像地下室的砖墙、像干涸的血（隐喻性的） |
| **Deep Brown / Coffee Brown** | Base | 像潮湿的地板、像旧皮箱、像吉他上的磨损 |
| **Smoke Grey / Concrete Grey** | Shadow | 像烟雾、像混凝土、像复印五次后的灰色噪点 |
| **Dark Mold Green** | Detail accent | 像墙缝里长出的野草——少量的生命力，在不可能的地方生长 |

### Semantic Color Vocabulary

| Meaning | Color | Usage |
|---------|-------|-------|
| Conflict / Passion | Rust Red | 队员吵架、演出高潮、关键决策 |
| Growth / Hope | Dirty Gold (not bright gold) | 技能提升、新内容解锁、里程碑 |
| Melancholy / Reflection | Cold Grey-Blue | 失败后的叙事事件、深夜场景 |
| Vitality | Dark Mold Green | 稀有的积极事件、野草般的韧性 |
| Mundane / Ordinary | Smoke Grey | 普通的排练日、背景UI |

### Performance Moment Pop Colors

- **Neon Pink / Neon Green** — stage lights exploding from darkness
- **High-contrast Bright White** — spotlight cutting through the dark
- These colors are **reserved for performance scenes only** — the stark contrast with daily low-saturation creates emotional impact

### UI Palette

- **Background**: Darker than the world — like reading a notebook in a dark room
- **Text**: Marker-white / pale yellow, handwritten feel
- **Button active**: Rust Red (like a stamped seal)
- **Button disabled**: Darker smoke grey (like water-damaged ink)

### Colorblind Safety

- Performance success/failure distinguished by **icon shape** (✓ vs ✗) + color + sound
- Character emotion states use **expression icon** + color double-encoding
- Red/green pairs avoided for critical gameplay information — use red/blue or rust/smoke instead

---

## 5. Character Design Direction

*[To be designed — Phase 3]*

---

## 6. Environment Design Language

*[To be designed — Phase 3]*

---

## 7. UI/HUD Visual Direction

*[To be designed — Phase 3]*

---

## 8. Asset Standards

*[To be designed — Phase 3]*

---

## 9. Reference Direction

*[To be designed — Phase 4]*
