# 小丑牌 Web — 项目说明

## 定位

课堂演示项目，用于教学「AI 全流程开发」。目标用户为零基础学员，通过亲手实现这款游戏完整体验开发流程。

## 目录结构

```
index.html      # 游戏主文件（单文件，含 HTML/CSS/JS）
PRD.html        # 产品需求文档（只读，参考用）
DESIGN.html     # 设计规范文档（只读，参考用）
agents/         # Claude Code 自定义 Agent 定义
slash/          # Claude Code 自定义斜杠命令
```

## 技术栈

- 纯 HTML + CSS + 原生 JS，无任何依赖
- 单文件交付，浏览器直接打开即用

## 运行方式

```bash
open index.html   # macOS
# 或直接在浏览器中打开 index.html
```

## 核心游戏逻辑（index.html）

- `buildDeck()` — 构建并洗牌 52 张牌堆
- `detectHand(cards)` — 识别 10 种牌型
- `calcScore(cards)` — 按公式计算得分
- `playHand()` / `discardHand()` — 出牌 / 弃牌操作
- `render()` — 统一刷新 DOM（每次状态变更后调用）
- `state` — 全局游戏状态对象

## 设计 Token（CSS 变量）

| 变量 | 值 | 用途 |
|------|-----|------|
| `--color-table` | #2d5a3d | 绿色桌面背景 |
| `--color-accent` | #e8682a | 出牌按钮、选中描边 |
| `--color-discard` | #a83232 | 弃牌按钮 |
| `--color-gold` | #d4a017 | 目标分、胜利边框 |
| `--color-card-red` | #d33f3f | 红花色（♥♦） |

## 已知待修复

- 含 J/Q/K/10 中任意两张的顺子无法正确识别（isStraight 用 value 去重导致 10 碰撞）
- 下一版本（v1.1.0）计划：暗色视觉升级 / HUD Toast / 进度条
