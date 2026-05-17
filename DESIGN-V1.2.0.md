# 小丑牌 Web V1.2.0 · Joker 系统 · 设计规范

**Version** V1.2.0 · 2026-05-17  
**基于** V1.0.0 DESIGN.html · 延续迭代，不推翻重建

---

## 0. 风格延续说明

V1.2.0 的视觉语言与 V1.0.0 完全一脉相承：深绿绒布桌面、强调橙按钮、JetBrains Mono 数字、扑克牌白底黑边——这些视觉锚点一个不动。

V1.2.0 只做三件事：
1. **在已有页面上「加一层」**：HUD 里塞入金币 + 关卡信息，HUD 下方插入 Joker 槽位区；
2. **新增一个全新页面**：商店弹窗（覆盖式，Z 轴最上层）；
3. **为 Joker 卡片创造一套新组件**：复用扑克牌的圆角、阴影、hover 位移逻辑，仅把配色从黑/红换成紫/金系。

所有现有 CSS 变量（`--color-table`、`--color-accent`、`--radius-card` 等）保持原值，新 token 追加到 `:root`，不覆盖。

---

## 1. 新增 Design Tokens

以下变量追加到原 `:root` 末尾：

```css
/* ── Joker 卡片色系 ── */
--color-joker-bg:        #1e1433;   /* Joker 卡底色：深紫 */
--color-joker-border:    #6b3fa0;   /* Joker 默认边框：中紫 */
--color-joker-text:      #e8d5ff;   /* Joker 卡内正文：浅紫白 */
--color-joker-price:     #f5c842;   /* Joker 价格标签：明金黄 */
--color-joker-flash:     #f5c842;   /* 触发闪烁边框：明金黄（与金币色统一） */

/* ── Joker 类型标签色（8 张差异化） ── */
--color-type-unconditional: #7c6bdb; /* 无条件（小丑）：蓝紫 */
--color-type-hand:          #e8952a; /* 牌型条件（欢乐/狂热）：橙黄 */
--color-type-suit-diamond:  #e06060; /* 方块条件（贪婪）：偏红 */
--color-type-suit-heart:    #d33f3f; /* 红心条件（色欲）：正红 */
--color-type-suit-spade:    #5a8fcf; /* 黑桃条件（暴怒）：钢蓝 */
--color-type-suit-club:     #4caa6e; /* 梅花条件（卑鄙）：草绿 */
--color-type-count:         #9b3535; /* 数量条件（半个小丑）：暗红 */

/* ── HUD 新增 ── */
--color-coin:            #f5c842;   /* 金币数值：明金黄 */

/* ── 槽位 ── */
--color-slot-empty:      rgba(255,255,255,0.15);  /* 空槽虚线色 */
--color-slot-empty-text: rgba(255,255,255,0.30);  /* 空槽文字色 */

/* ── 商店弹窗 ── */
--color-shop-bg:         #12101a;   /* 商店背景：深紫黑 */
--color-shop-header:     #1e1b2e;   /* 商店头栏底色 */
--color-shop-divider:    rgba(255,255,255,0.08); /* 分隔线 */

/* ── 浮字色 ── */
--color-float-mult:      #f5c842;   /* "+X Mult" 浮字：金黄 */
--color-float-chips:     #5dd67a;   /* 芯片浮字备用：分绿（可选） */
```

**使用场景速查：**

| Token | 用途 |
|---|---|
| `--color-joker-bg` | Joker 卡片背景，区别于扑克牌白底 |
| `--color-joker-border` | Joker 默认边框，hover/激活时换为 `--color-joker-flash` |
| `--color-joker-flash` | 触发动画时的金边颜色，与金币色保持统一 |
| `--color-coin` | HUD 金币数值、商店金币显示 |
| `--color-type-*` | 8 张 Joker 的类型色带，用在卡片顶部色条或名称底色 |
| `--color-shop-bg` | 商店弹窗整体背景，刻意选深紫黑拉开与桌面绿的对比 |

---

## 2. Joker 卡片组件规范

### 2.1 尺寸

| 场景 | 宽 | 高 | 备注 |
|------|----|----|------|
| 主游戏槽位 | 76px | 110px | 比扑克牌（88×128）窄一档，暗示"特殊" |
| 商店陈列 | 120px | 172px | 放大以便查看效果描述 |
| 商店中"已装备槽位"小图 | 60px | 86px | 仅展示名称，提示已装备 |

圆角沿用：`border-radius: var(--radius-card)` → **12px**

### 2.2 卡面结构

```
┌──────────────────────────────┐  ← 76px wide
│ [色条 4px，类型色]            │  ← 顶部色条，颜色见类型色表
│                              │
│  小丑                        │  ← 名称：14px / Noto Sans SC / 700 / #e8d5ff
│  ────────────────            │  ← 分隔线 rgba(255,255,255,0.15)
│  每次出牌                    │  ← 效果描述第 1 行：11px / 400 / rgba(255,255,255,0.7)
│  +4 倍率                     │  ← 效果描述第 2 行：同上
│                              │
│  [仅商店显示] $4  [购买]     │  ← 价格 + 购买按钮（槽位视图中隐藏此行）
└──────────────────────────────┘
```

内边距：`padding: 8px 10px`（槽位版）/ `padding: 14px 16px`（商店版）

### 2.3 四种状态（CSS 关键属性）

**默认（Default）**
```css
background: var(--color-joker-bg);            /* #1e1433 */
border: 1.5px solid var(--color-joker-border); /* #6b3fa0 */
box-shadow: 0 2px 8px rgba(0,0,0,0.4);
transition: transform 200ms ease, box-shadow 200ms ease, border-color 200ms ease;
```

**悬停（Hover）** — 商店中"待购买"时
```css
transform: translateY(-4px);
box-shadow: 0 6px 18px rgba(107,63,160,0.5);
border-color: #a070e0;   /* 比默认紫更亮 */
cursor: pointer;
```

**触发激活（Active Flash）** — 出牌结算时，JS 挂 `.joker--triggered` 类
```css
animation: joker-flash 0.4s ease 2;   /* 见第 8 章 keyframes */
border-color: var(--color-joker-flash); /* #f5c842 */
box-shadow: 0 0 0 2.5px #f5c842, 0 0 12px rgba(245,200,66,0.5);
```

**空槽位（Empty Slot）**
```css
background: transparent;
border: 2px dashed var(--color-slot-empty); /* rgba(255,255,255,0.15) */
box-shadow: none;
display: flex;
align-items: center;
justify-content: center;
color: var(--color-slot-empty-text);        /* rgba(255,255,255,0.30) */
font-size: 22px;    /* "+" 号 */
```

**禁用（Disabled）** — 商店购买按钮禁用时卡片本身样式不变，仅购买按钮变灰
```css
/* 卡片本身不降 opacity，只有购买按钮禁用 */
.btn-buy:disabled {
  background: rgba(255,255,255,0.1);
  color: rgba(255,255,255,0.35);
  cursor: not-allowed;
  box-shadow: none;
}
```

### 2.4 8 张 Joker 视觉差异表

顶部色条（4px 高度横条）用于一眼区分类型。文字色均为 `--color-joker-text` (#e8d5ff)。

| # | 名称 | 类型 | 顶部色条 hex | 辅助特征 |
|---|------|------|-------------|---------|
| 1 | 小丑 | 无条件 | `#7c6bdb`（蓝紫） | 最基础，色调最"素" |
| 2 | 欢乐小丑 | 牌型-对子 | `#e8952a`（橙黄） | 暖色调，"欢乐" |
| 3 | 狂热小丑 | 牌型-三条 | `#d4750a`（深橙） | 比欢乐更深，"狂热" |
| 4 | 贪婪小丑 | 花色-♦ | `#e06060`（方块红） | 与扑克牌红心红相近 |
| 5 | 色欲小丑 | 花色-♥ | `#d33f3f`（红心红） | 正红，与花色保持一致 |
| 6 | 暴怒小丑 | 花色-♠ | `#5a8fcf`（黑桃钢蓝） | 黑桃用蓝色表达"冷厉" |
| 7 | 卑鄙小丑 | 花色-♣ | `#4caa6e`（梅花绿） | 绿色，暗示"卑鄙如草" |
| 8 | 半个小丑 | 数量条件 | `#9b3535`（暗红） | 残缺感，用沉郁暗红 |

> 工程提示：用 `data-joker-id` 属性 + CSS `[data-joker-id="joker"] .joker-stripe { background: #7c6bdb; }` 实现，不需要 8 个独立 class。

---

## 3. HUD 改造方案

### 3.1 设计决策

金币 **不**放在 4 列 HUD 里凑成第 5 列——那样每列太窄，金币数字会和分数挤在一起，视觉权重不够。

方案：**HUD 拆成两行**：
- **第 1 行（28px 高）**：关卡信息（左） + 金币（右），背景略深
- **第 2 行（56px 高）**：原有 4 列（目标分 / 当前分 / 出牌剩余 / 弃牌剩余）

总 HUD 高度：**84px**（原 64px + 28px 新增行）

### 3.2 ASCII 线框对比

**旧版 HUD（64px）：**
```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│    目标分          当前分         剩余出牌         剩余弃牌                                  │  64px
│     300            0               4                3                                        │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

**新版 HUD（84px）：**
```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│  第 1 关 / 共 3 关 · 目标 300 分                                          $ 8              │  28px（行1）
├──────────────────┬──────────────────┬──────────────────┬───────────────────────────────────┤
│    目标分        │    当前分        │   剩余出牌       │         剩余弃牌                   │  56px（行2）
│     300          │      0           │      4           │            3                       │
└──────────────────┴──────────────────┴──────────────────┴───────────────────────────────────┘
```

### 3.3 CSS 关键属性

```css
/* 行 1：关卡 + 金币 */
.hud-row1 {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  height: 28px;
  background: rgba(0,0,0,0.5);
  border-bottom: 1px solid rgba(255,255,255,0.06);
}
.hud-level {
  font-size: 11px;
  font-weight: 600;
  color: rgba(255,255,255,0.6);
  letter-spacing: 0.5px;
}
.hud-coins {
  font-family: 'JetBrains Mono', monospace;
  font-size: 16px;
  font-weight: 700;
  color: var(--color-coin);        /* #f5c842 */
  letter-spacing: 0.5px;
  font-variant-numeric: tabular-nums;
}
.hud-coins::before { content: '$ '; font-size: 12px; opacity: 0.8; }

/* 行 2：原 4 列，高度从 64px 缩至 56px */
.hud-row2 {
  display: flex;
  align-items: center;
  height: 56px;
  background: rgba(0,0,0,0.35);
  backdrop-filter: blur(4px);
  border-bottom: 1px solid rgba(255,255,255,0.08);
}
/* hud-item / hud-label / hud-value 样式不变，复用 V1.0.0 */
```

---

## 4. Joker 槽位区

### 4.1 位置与尺寸

- **在主游戏页面**：位于 HUD（84px）下方、出牌区上方
- **区域高度**：**130px**（卡片 110px + 上下内边距各 10px）
- **宽度**：与游戏容器等宽（1080px），水平居中显示 3 个槽位

### 4.2 ASCII 线框（主游戏页完整布局）

```
┌─────────────────────────────────────────── 1080px ────────────────────────────────────────────┐
│  第 1 关 / 共 3 关 · 目标 300 分                                              $ 8             │ ← 28px 行1
├───────────────────────┬───────────────────────┬────────────────────┬─────────────────────────┤
│       目标分          │       当前分          │     剩余出牌       │       剩余弃牌          │ ← 56px 行2
│        300            │         0             │        4           │          3              │
├───────────────────────────────────────────────────────────────────────────────────────────────┤
│                         Joker 槽位区 (background: rgba(0,0,0,0.20))                           │ ← 130px
│         ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐                 │
│         │▌ 小丑            │    │ ╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌ │    │ ╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌ │                 │
│         │─────────────────│    │                  │    │                  │                 │
│         │ 每次出牌         │    │       +          │    │       +          │                 │
│         │ +4 倍率          │    │      空槽         │    │      空槽         │                 │
│         └──────────────────┘    └──────────────────┘    └──────────────────┘                 │
│                           Slot 1              Slot 2              Slot 3                     │
├───────────────────────────────────────────────────────────────────────────────────────────────┤
│                                  出牌区（flex: 1 弹性撑满）                                    │
│                  [同花顺]                                                                      │
│           [♥A] [♥K] [♥Q] [  +  ] [  +  ]                                                     │
│                         选中 1–5 张牌后点击「出牌」                                            │
├───────────────────────────────────────────────────────────────────────────────────────────────┤
│  手牌 · 8 张                                                                                  │
│  [J♥] [7♠] [3♦] [K♣] [9♥] [4♠] [2♦] [8♣]                                                   │
├───────────────────────────────────────────────────────────────────────────────────────────────┤
│                    [出牌]    [弃牌]    [重新开始]                                              │
└───────────────────────────────────────────────────────────────────────────────────────────────┘
```

总高度：84 + 130 + （出牌区弹性） + 手牌区 + 操作栏 ≈ **保持 640px**。  
出牌区需要相应压缩：原先 `flex: 1` 会自动吸收剩余高度，添加 Joker 槽位后出牌区会缩小约 130px，建议工程师验证牌不会被截断（5 张 128px 高的牌 + badge + hint 约需 190px，剩余空间够用）。

### 4.3 空/满状态 CSS

```css
/* 槽位容器 */
.joker-zone {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  height: 130px;
  background: rgba(0,0,0,0.20);
  border-bottom: 1px solid rgba(255,255,255,0.06);
  padding: 0 24px;
}

/* 单个槽位（空状态） */
.joker-slot {
  width: 76px;
  height: 110px;
  border-radius: 12px;
  border: 2px dashed rgba(255,255,255,0.15);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 4px;
  color: rgba(255,255,255,0.30);
}
.joker-slot .slot-plus { font-size: 22px; line-height: 1; }
.joker-slot .slot-label { font-size: 9px; letter-spacing: 1px; text-transform: uppercase; }

/* 已装备时：复用 .joker-card 组件，嵌入 .joker-slot */
.joker-slot.filled {
  border: none; /* 由 joker-card 自带边框 */
  background: transparent;
}
```

---

## 5. 商店页规范

### 5.1 覆盖层结构

商店是**全屏遮罩 + 居中弹窗**，z-index 在游戏弹窗层级之上。

```css
.shop-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.72);
  backdrop-filter: blur(6px);
  z-index: 200;                     /* 游戏容器 z-index: 0，弹窗层 z-index: 100 */
  display: flex;
  align-items: center;
  justify-content: center;
}

.shop-modal {
  width: 720px;
  background: var(--color-shop-bg);  /* #12101a */
  border-radius: 20px;
  border: 1px solid rgba(255,255,255,0.1);
  box-shadow: 0 24px 64px rgba(0,0,0,0.7);
  overflow: hidden;
}
```

### 5.2 完整 ASCII 线框

```
┌─────────────────────── 720px ───────────────────────────┐
│  商店                                     金币: $ 8     │  ← header 60px
│  第 1 关已通过！选购 Joker 强化牌组                      │    background: #1e1b2e
├─────────────────────────────────────────────────────────┤  ← 分隔线 1px rgba(255,255,255,0.08)
│                                                         │
│   ┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐  │
│   │▌小丑            │  │▌欢乐小丑        │  │▌半个小丑         │  │ ← 商品区 228px
│   │                 │  │                 │  │                  │  │
│   │ 每次出牌        │  │ 出牌含对子时    │  │ 出牌 ≤3 张时     │  │
│   │ +4 倍率         │  │ +8 倍率         │  │ +20 倍率         │  │
│   │                 │  │                 │  │                  │  │
│   │     $ 4         │  │     $ 4         │  │      $ 6         │  │ ← 价格 16px gold
│   │  [ 购  买 ]     │  │  [ 购  买 ]     │  │  [ 购  买 ]      │  │ ← 按钮
│   └─────────────────┘  └─────────────────┘  └──────────────────┘  │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  当前 Joker 槽位：                                       │  ← footer 64px
│  [小丑 76×52] [空槽 76×52] [空槽 76×52]                 │    background: #1e1b2e
│                                   [ 继续下一关 → ]      │
└─────────────────────────────────────────────────────────┘
```

### 5.3 各部分规范

**Header（60px）**
```css
.shop-header {
  padding: 0 28px;
  height: 60px;
  background: var(--color-shop-header);   /* #1e1b2e */
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 2px;
}
.shop-title {
  font-size: 18px;
  font-weight: 700;
  color: #fff;
}
.shop-subtitle {
  font-size: 12px;
  color: rgba(255,255,255,0.5);
}
.shop-coins {
  font-family: 'JetBrains Mono', monospace;
  font-size: 20px;
  font-weight: 700;
  color: var(--color-coin);   /* #f5c842 */
  font-variant-numeric: tabular-nums;
}
/* header 内 flex row: title+subtitle 左 / coins 右 */
```

**商品区（228px 高，含上下 padding 各 24px）**
- 3 张 Joker 卡横排，`justify-content: center; gap: 24px`
- 每张商店版 Joker 卡：**120px × 172px**
- 卡内布局（从上到下）：
  - 色条：4px
  - 名称：14px / 700 / #e8d5ff，padding-top 10px
  - 分隔线
  - 效果描述：11px / 400 / rgba(255,255,255,0.65)，至多 2 行
  - 弹性空间
  - 价格：`$ X`，16px / JetBrains Mono / #f5c842
  - 购买按钮：高 32px，宽 100%，`border-radius: 8px`（非胶囊，节省竖向空间）

**购买按钮状态**
```css
/* 默认 */
.btn-buy {
  background: var(--color-accent);     /* #e8682a */
  color: #fff;
  font-weight: 700;
  font-size: 13px;
  border: none;
  border-radius: 8px;
  height: 32px;
  cursor: pointer;
  transition: opacity 150ms ease, transform 150ms ease;
}
.btn-buy:hover { opacity: 0.88; transform: translateY(-1px); }

/* 金币不足 */
.btn-buy.insufficient {
  background: rgba(168,50,50,0.5);     /* 偏红暗色 */
  color: rgba(255,255,255,0.5);
  cursor: not-allowed;
}

/* 槽位已满 */
.btn-buy.slots-full {
  background: rgba(255,255,255,0.1);
  color: rgba(255,255,255,0.3);
  cursor: not-allowed;
}

/* 已购买 */
.btn-buy.purchased {
  background: rgba(93,214,122,0.25);   /* 绿色暗调 */
  color: #5dd67a;
  cursor: default;
}
```

**Footer（64px）**
```css
.shop-footer {
  height: 64px;
  background: var(--color-shop-header);  /* #1e1b2e */
  border-top: 1px solid var(--color-shop-divider);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 28px;
}
/* 左侧：当前槽位小卡（60×86px 迷你版 Joker 卡 或 虚线空槽） */
/* 右侧：继续按钮 */
.btn-continue {
  background: var(--color-accent);   /* #e8682a */
  color: #fff;
  font-weight: 700;
  font-size: 14px;
  border-radius: 999px;
  padding: 10px 26px;
  border: none;
  cursor: pointer;
  transition: transform 150ms ease, box-shadow 150ms ease;
  box-shadow: 0 4px 14px rgba(232,104,42,0.45);
}
.btn-continue:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 18px rgba(232,104,42,0.6);
}
```

**提示 Toast（金币不足 / 槽位已满）**
- 出现在商品区顶部，`position: absolute; top: 16px; left: 50%; transform: translateX(-50%)`
- 背景：`rgba(168,50,50,0.85)` / 圆角 `999px` / padding `6px 18px`
- 字号：12px，白色，`font-weight: 600`
- 自动 2s 后消失（opacity transition）

---

## 6. 关卡过渡 / 弹窗规范

三种弹窗复用 V1.0.0 的弹窗壳体（黑底 + 金边），只修改内容和按钮。

### 6.1 弹窗壳体（延续 V1.0.0）

```css
.modal-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,0.65);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}
.modal-box {
  background: #0d0d0d;
  border: 2px solid var(--color-gold);    /* #d4a017 */
  border-radius: 16px;
  padding: 36px 48px;
  text-align: center;
  min-width: 320px;
  max-width: 440px;
}
```

### 6.2 三种弹窗差异表

| 弹窗 | 触发条件 | 主标题 | 副文本 | 主按钮 | 副按钮 | 边框色 |
|------|----------|--------|--------|--------|--------|--------|
| 过关弹窗 | 第 1/2 关达目标分 | 第 X 关通过！ | 得分 `XXXX` | **进入商店** | — | `#d4a017`（金） |
| 通关弹窗 | 第 3 关达 2000 分 | 恭喜通关！ | 总得分 `XXXX` | **重新开始** | — | `#f5c842`（亮金） |
| 失败弹窗 | 出牌耗尽未达目标 | 游戏失败！ | 还差 `XXX` 分 | **重新开始** | — | `#a83232`（弃牌红） |

### 6.3 各弹窗排版细节

**过关弹窗（140px 高内容区）**
```
┌────────────────────────────┐
│   第 1 关通过！             │  ← 24px / 700 / #d4a017
│                            │
│        得分 1280           │  ← 36px / JetBrains Mono / 700 / #5dd67a
│                            │
│     [ 进入商店 → ]         │  ← btn-play 样式，宽 160px
└────────────────────────────┘
```

**通关弹窗（160px 高内容区）**
```
┌────────────────────────────┐
│      恭喜通关！             │  ← 28px / 700 / #f5c842
│                            │
│    总得分 12480            │  ← 40px / JetBrains Mono / 700 / #5dd67a
│    金币剩余 $ 3            │  ← 14px / JetBrains Mono / #f5c842
│                            │
│     [ 重新开始 ]           │  ← btn-play 样式
└────────────────────────────┘
```

**失败弹窗（140px 高内容区，边框改红）**
```
┌────────────────────────────┐
│      游戏失败！             │  ← 24px / 700 / #e06060
│                            │
│    还差 520 分             │  ← 32px / JetBrains Mono / 700 / rgba(255,255,255,0.8)
│                            │
│     [ 重新开始 ]           │  ← btn-discard 样式（红色按钮）
└────────────────────────────┘
```

CSS 差异只需切换 `border-color` 和标题 `color`，其余壳体复用。

---

## 7. 出牌结算动效

### 7.1 Joker 金边闪烁

触发条件：出牌结算，某 Joker 满足触发条件 → JS 给对应槽位卡片挂 `.joker--triggered` class，动画结束后移除。

```css
@keyframes joker-flash {
  0%   { border-color: var(--color-joker-border); box-shadow: 0 2px 8px rgba(0,0,0,0.4); }
  50%  { border-color: var(--color-joker-flash);  box-shadow: 0 0 0 2.5px #f5c842, 0 0 16px rgba(245,200,66,0.6); }
  100% { border-color: var(--color-joker-border); box-shadow: 0 2px 8px rgba(0,0,0,0.4); }
}

.joker--triggered {
  animation: joker-flash 0.4s ease 2;
  /* iteration-count: 2 → 总时长 0.8s，亮→暗→亮→暗共 2 个周期 */
}
```

**参数说明：**
- 单次周期：`0.4s`（亮起 + 熄灭各 0.2s）
- 循环次数：`2`（总时长 0.8s）
- Easing：`ease`（头尾缓，中间快，闪烁感明显）
- 边框宽度在闪烁期间临时变为 2.5px（box-shadow 模拟），不改 border-width 避免布局抖动

### 7.2 "+X Mult" 浮字动画

```css
@keyframes float-up-fade {
  0%   { opacity: 1; transform: translateY(0); }
  100% { opacity: 0; transform: translateY(-24px); }
}

.float-text {
  position: absolute;
  /* 由 JS 动态设置 top / left，锚点为对应 Joker 卡片的垂直中心 */
  font-family: 'JetBrains Mono', monospace;
  font-size: 16px;
  font-weight: 700;
  color: var(--color-float-mult);   /* #f5c842 */
  text-shadow: 0 1px 4px rgba(0,0,0,0.6);
  pointer-events: none;
  white-space: nowrap;
  animation: float-up-fade 0.8s ease-out forwards;
  z-index: 150;
}
```

**参数说明：**
- 浮动方向：垂直向上 24px
- 时长：0.8s，`ease-out`（快速飞出，缓慢消失）
- 消失方式：opacity 1 → 0（不用 `display:none`，保持动画流畅）
- 文字格式：`+4 Mult` / `+8 Mult`，格式固定（符号 + 数字 + 空格 + Mult）
- 多张 Joker 同时触发时：各自独立生成一个 `.float-text` 节点，偏移 left 值相互错开 4px 避免完全重叠
- 动画结束后 JS 移除 DOM 节点（监听 `animationend` 事件）

---

## 8. 响应式 / 缩放建议

V1.2.0 **继续不做响应式**（与 PRD 第 11 章一致），沿用 V1.0.0 的 `transform: scale()` 整体缩放方案。

新增内容对尺寸的影响：
- HUD 从 64px 扩展到 84px（+20px）
- Joker 槽位区新增 130px
- **游戏容器总高度建议从 640px 扩展到 720px**（+80px）
- 视口缩放公式更新为：`scale = Math.min(viewportWidth / 1080, viewportHeight / 720)`

| 视口 | 缩放比 | 容器显示尺寸 |
|------|--------|------------|
| 1920×1080 | 1.0× | 1080×720（填满高度） |
| 1440×900 | 0.83× | 897×598 |
| 1280×800 | 0.74× | 800×533 |
| 1024×768 | 0.72× | 778×518（以宽限制） |

> 商店弹窗（720px 宽）在缩放容器内是 `position: absolute`，随容器一起缩放，不需要单独处理。

---

## 9. Z-index 分层规范

```
z-index: 0    → 游戏桌面背景（.game-viewport）
z-index: 10   → 手牌、出牌区卡片（正常层）
z-index: 20   → 选中/hover 的卡片（translateY 上浮时）
z-index: 50   → Joker 槽位区
z-index: 100  → 过关/通关/失败弹窗（.modal-overlay）
z-index: 150  → 浮字动画（.float-text，需在弹窗之上）
z-index: 200  → 商店遮罩层（.shop-overlay，最顶层）
```

---

## 10. 工程交班

1. **复用优先**：Joker 卡片的圆角（`--radius-card: 12px`）、hover 位移（`translateY(-4px)`）、阴影结构直接从 `.poker-card` 样式复制，改色不改结构；商店继续按钮样式直接用 `.btn-play`，无需新 class。

2. **容器高度**：游戏容器建议从 `640px` 扩展到 `720px`（HUD+20、Joker 槽+130，出牌区 flex:1 吸收压缩），缩放公式同步更新（见第 8 章）。

3. **动效实现**：Joker 闪烁用 CSS `@keyframes` + JS 挂/摘 class（`classList.add → animationend → classList.remove`）；浮字用 JS 动态 `createElement` + `style.top/left` 定位 + 监听 `animationend` 后 `remove()`；不需要引入任何动画库。

4. **商店弹窗 HTML 结构**：建议直接写在游戏容器（`.game-viewport`）内部，用 `position: absolute; inset: 0` 覆盖，这样自动跟随 `transform: scale()` 一起缩放，不需要额外计算商店弹窗的尺寸适配。

5. **`data-joker-id` 属性**：Joker 卡片渲染时加 `data-joker-id="joker"` 等属性，CSS 用 `[data-joker-id="joker"] .joker-stripe { background: #7c6bdb; }` 实现 8 张视觉差异，避免生成 8 个独立 class，维护成本低。

---

## 附：完整新增 Tokens 速查

```css
/* 追加到 :root，不修改现有变量 */
:root {
  --color-joker-bg:          #1e1433;
  --color-joker-border:      #6b3fa0;
  --color-joker-text:        #e8d5ff;
  --color-joker-price:       #f5c842;
  --color-joker-flash:       #f5c842;
  --color-type-unconditional:#7c6bdb;
  --color-type-hand:         #e8952a;
  --color-type-suit-diamond: #e06060;
  --color-type-suit-heart:   #d33f3f;
  --color-type-suit-spade:   #5a8fcf;
  --color-type-suit-club:    #4caa6e;
  --color-type-count:        #9b3535;
  --color-coin:              #f5c842;
  --color-slot-empty:        rgba(255,255,255,0.15);
  --color-slot-empty-text:   rgba(255,255,255,0.30);
  --color-shop-bg:           #12101a;
  --color-shop-header:       #1e1b2e;
  --color-shop-divider:      rgba(255,255,255,0.08);
  --color-float-mult:        #f5c842;
}
```

---

*DESIGN-V1.2.0.md · 小丑牌 Joker 系统 · 2026-05-17*
