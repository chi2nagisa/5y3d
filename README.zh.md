# 五年段位三年模拟 (5y3d)

> 太鼓之达人段位道场理论成绩推算工具

根据玩家历史最佳成绩，自动推算在各版本段位道场中的理论合格结果（金合格 / 赤合格 / 不合格 / 未完全游玩），并判定 Perfect / FullCombo / Normal 边框状态。

---

## 技术架构

- **纯静态单文件**：所有逻辑集中在 `index.html`，无构建工具、无框架、无打包
- **数据驱动**：段位配置与玩家成绩完全通过 JSON 文件加载，不硬编码
- **浏览器存储**：成绩数据与手动修改缓存到 `localStorage`

---

## 数据规范

### 段位数据 `dani_datas/dani_data.json`

根对象：`{ "versions": [...] }`，按时间顺序排列。

每个 `version` 包含：
- `id`：版本标识，如 `green`, `blue`, `nijiiro_2025`
- `name_zh`：中文版本名
- `danis`：段位数组

每个 `dani` 包含：
- `id`：段位标识
- `sort_order`：排序序号（0~24）
- `total_notes`：三曲总音符数
- `songs`：3 首课题曲，`song_id`, `name`, `name_zh`, `difficulty`, `stars`, `notes`
- `clear_conditions`：合格条件数组

**添加新版本段位数据**：直接在 `versions` 数组末尾追加对象，确保字段符合规范即可。

### 外传数据 `dani_datas/gaiden_data.json`

结构与 `dani_data.json` 一致，但 `danis` 顺序遵循 JSON 中的原始顺序（不使用 `sort_order`）。

### 成绩数据格式

对象数组，每个元素：
- `song_no` (Integer)：歌曲唯一标识，对应段位数据中的 `song_id`
- `level` (Integer)：难度，`1~5` 分别对应 简单/普通/困难/魔王(表)/魔王(裏)
- `high_score` (Integer)：单首歌曲总得分
- `high_score_result` (Integer[5])：
  - `[0]` 良判定数
  - `[1]` 可判定数
  - `[2]` 不可判定数
  - `[3]` 连打数
  - `[4]` 最大连击数

**数据来源**：
- donder 查分器 API：`https://hasura.llx.life/api/rest/donder/get-score?id={id}`
- 本地 JSON 文件上传
- 默认回退：`score_example.json`（开发用，不提交）

---

## 核心功能实现

### 合格计算逻辑

`computeDaniResult(dani)` 对每首课题曲匹配成绩后，按条件类型聚合：

| 条件类型 | shared 计算 | per_song 计算 |
|---------|------------|--------------|
| `good` | 三曲 `[0]` 之和 | 每曲 `[0]` |
| `ok` | 三曲 `[1]` 之和 | 每曲 `[1]` |
| `bad` | 三曲 `[2]` 之和 | 每曲 `[2]` |
| `roll` / `rolls` | 三曲 `[3]` 之和 | 每曲 `[3]` |
| `combo` | 三曲 `[4]` 之和 | 每曲 `[4]` |
| `hits` | `good + ok + roll` 合计 | 每曲分别计算 |
| `score` | 三曲 `high_score` 之和 | 每曲 `high_score` |
| `gauge` | **仅展示，不参与计算** | **仅展示，不参与计算** |

### 附加框判定

- **Perfect（虹框）**：三曲全部 `良数 === 音符数`
- **FullCombo（金框）**：三曲全部 `连击数 === 音符数`，但不满足全良
- **Normal**：以上均不满足

### 手动成绩编辑

- 编辑入口：详情面板每首课题曲旁的「编辑」按钮
- 存储位置：`localStorage['5y3d_manual_scores']`，与真实成绩物理隔离
- 读取优先级：手动修改 > 真实成绩 > 示例数据
- 重置：导入数据面板的「重置手动修改」按钮

### 中日文曲名切换

- 默认显示中文（`songs[].name_zh`）
- `name_zh` 为 `null` 或空字符串时回退日文（`songs[].name`）
- 语言偏好持久化到 `localStorage['5y3d_lang']`

---

## 本地开发

```bash
python -m http.server 8080
# 访问 http://localhost:8080
```

**文件结构**：

```
.
├── index.html                 # 页面入口（HTML + CSS + JS）
├── dani_datas/
│   ├── dani_data.json        # 本传段位配置（v3.0）
│   └── gaiden_data.json      # 外传段位配置
└── dani_score_logo/           # 合格状态图标（WebP）
```

---

## 版本历史

| 版本 | 日期 | 主要内容 |
|------|------|---------|
| v1.5.1 | 2026-05-11 | 课题曲列表两行布局，消除重叠 |
| v1.5.0 | 2026-05-11 | 曲名中日文切换 |
| v1.4.0 | 2026-05-11 | 手动成绩编辑与假设分析 |
| v1.3.1 | 2026-05-11 | 曲名列宽度修复 |
| v1.3.0 | 2025-04-19 | 查分页面（donder API + JSON 上传）|
| v1.2.0 | 2025-04-19 | 外传段位矩阵实际渲染 |
| v1.1.1 | 2025-04-19 | score 条件支持、外传数据修复 |
| v1.1.0 | 2025-04-17 | 本传/外传 Tab 切换 |
| v1.0.0 | 2025-04-09 | 初始版本：矩阵展示、合格计算、详情面板 |

---

## 已知问题 / TODO

- [ ] 外传模式详情面板在窄屏下可能横向滚动（extra 矩阵列数少，详情内容宽）
- [ ] 颜色版大量曲目 `song_id = -1`，无法匹配成绩
- [ ] 手动编辑未支持 `high_score` 修改（score 条件的手动假设分析不完整）
