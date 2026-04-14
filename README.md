# 5y3d — 太鼓之达人段位道场理论成绩

> **"5 Years of Dan, 3 Mocks per Rank"**

一个纯静态前端页面，用于展示玩家在《太鼓之达人》各版本段位道场中的**理论合格结果**。根据历史最佳成绩自动推算在各版本、各段位中能否通过赤合格 / 金合格，并展示 Perfect / FullCombo 边框状态。

![license](https://img.shields.io/badge/license-MIT-blue.svg)

---

## ✨ 功能特点

- **多版本矩阵展示**：支持从 GREEN 到虹版 2025 共 10 个版本的段位道场数据
- **自动理论计算**：根据玩家成绩自动判定每段位的合格状态（金合格 / 赤合格 / 不合格 / 未完全游玩）
- **边框状态判定**：
  - 🌈 **Perfect（虹框）**：三曲全部全良
  - ⭐ **FullCombo（金框）**：三曲全部全连
  - ⚪ **Normal（普通框）**：常规通过
- **详情面板**：点击任意单元格，展开查看课题曲、合格条件与玩家成绩的详细对比
- **里谱面兼容**：支持 `difficulty: 5`（魔王裏）的成绩匹配

---

## 🚀 快速开始

由于页面通过 `fetch()` 加载本地 JSON 文件，**不能直接用 `file://` 协议打开**。请使用任意静态服务器：

```bash
# Python 3
python -m http.server 8080

# Node.js
npx serve .
```

然后在浏览器中访问：

```
http://localhost:8080
```

---

## 📁 项目结构

```
.
├── index.html                    # 页面入口（HTML + CSS + JS）
├── danni_datas/
│   └── danni_data.json           # 段位配置数据（v3.0 规范，10 个版本）
├── dani_score_logo/              # 合格状态图标（WebP）
│   ├── GoldNormalClear.webp
│   ├── GoldFullComboClear.webp
│   ├── GoldPerfectClear.webp
│   ├── RedNormalClear.webp
│   ├── RedFullComboClear.webp
│   ├── RedPerfectClear.webp
│   └── NotClear.webp
├── score_example.json            # 玩家成绩示例数据
├── danni_data_k2p6pre.json       # 旧格式/备用段位数据
└── dev_doc/                      # 开发文档
    ├── AGENTS.md                 # AI 代理协作规范（必读）
    ├── requirements_doc.md       # 需求文档
    ├── json_structure_design.md  # v3.0 数据结构规范
    ├── DESIGN.md                 # 技术规划存档
    ├── TODO.md                   # 开发任务跟踪
    └── VERSION.md                # 版本变更记录
```

---

## 🎮 数据来源与匹配

### 段位数据
- **文件**：`danni_datas/danni_data.json`
- **版本覆盖**：GREEN、BLUE、YELLOW、RED、虹 2020 ~ 虹 2025 共 10 个版本
- **段位覆盖**：初級 → 達人，共 25 个段位
- **歌曲匹配率**：约 77.5% 的课题曲已补充 `song_id`，可直接与玩家成绩关联；未匹配曲目（主要为颜色版流行曲、虹版高难新曲）标记为 `-1`

### 成绩数据
- **文件**：`score_example.json`
- **匹配规则**：通过 `song_no`（歌曲 ID）+ `level`（难度 1~5）与段位课题曲精确匹配
- **核心字段**：`high_score_result` 数组，分别对应 [良, 可, 不可, 连打, 最大连击数]

---

## 🛠️ 技术栈

- **HTML5 + CSS3 + Vanilla JavaScript (ES6+)**
- 无构建工具、无框架、无打包流程
- 现代浏览器即可运行

---

## 📄 相关文档

| 文档 | 说明 |
|------|------|
| [`dev_doc/AGENTS.md`](dev_doc/AGENTS.md) | AI 编码代理协作规范与开发流程 |
| [`dev_doc/requirements_doc.md`](dev_doc/requirements_doc.md) | 功能需求与验收标准 |
| [`dev_doc/json_structure_design.md`](dev_doc/json_structure_design.md) | v3.0 JSON 数据结构设计规范 |
| [`dev_doc/DESIGN.md`](dev_doc/DESIGN.md) | 技术规划与架构设计存档 |
| [`dev_doc/TODO.md`](dev_doc/TODO.md) | 开发任务跟踪 |
| [`dev_doc/VERSION.md`](dev_doc/VERSION.md) | 版本变更记录 |

---

## 📌 当前版本

**v1.0.0** — 已实现多版本段位矩阵展示、理论合格自动计算、详情面板交互。

---

## 📜 License

MIT
