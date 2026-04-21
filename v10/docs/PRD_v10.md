# 探索南山 v10 PRD

- **版本**：v10
- **日期**：2026-04-20
- **作者**：Bella（产品运营） + Claude（协同）
- **前置版本**：https://bellazhuang417-cyber.github.io/nanshan-parks/ （v9）

---

## 1. 为什么做 v10

v9 上线后通过实际使用和 review 发现三类核心问题：

| 类型 | 具体表现 | 根因 |
|---|---|---|
| **信息可信度** | 所有 POI 描述由 LLM 一次性生成，没有人核验，没有来源标注 | 缺少数据分层与来源字段 |
| **交互无意义** | "已探索 / 待探索"计数器，用户不知道它代表什么、也不在乎 | 为了凑功能做的 gamification，没有场景支撑 |
| **信息颗粒度粗** | "人流高/中/低"单一标签掩盖了时段差异；"停车场"和"日出"都在一条图例里 | 把性质不同的数据（设施 vs. 体验）混在一个图层 |

v10 的目标不是新增功能，而是把**信息架构重新组织**：让用户打开地图时，能按"我现在要干嘛"（Routine）去找地点，而不是"这里有什么"去翻 POI 列表。

---

## 2. 目标与非目标

### 目标
1. 用户可以按**活动意图**（日出 / 日落 / 晨跑 / 骑行）快速得到一组推荐地点
2. 所有数据都有**明确来源和可信度标注**，LLM 生成的内容和实际观测的内容要能区分
3. 设施图层（停车场 / 洗手间 / 步道轨迹）和体验图层（Routine POI）**分层展示**，互不干扰
4. 用户看到过时信息时能**一键反馈**（不是提交到后端，是本地标记 + 视觉提醒）

### 非目标
- **不做**实时人流（没有数据源，不伪造）
- **不做**用户账号 / 社区 / UGC 内容提交
- **不做**路线规划 / 导航（跳转到高德）
- **不覆盖**南山以外区域

---

## 3. 用户与场景

**主要用户**：深圳南山本地居民 / 短期居住者，想在周末或下班后找个地方运动或放松。

**典型场景**：
- "今晚想看日落，去哪拍照最好？" → 打开"看日落" Routine → 列表按精选排序
- "这周想晨跑，有哪些路线，哪条平坦哪条爬升大？" → "晨跑" Routine → 看 meta chip（3km / +0m / 塑胶跑道 / 休闲）
- "开车去，要停哪儿？" → 打开"停车场"图层 → 地图上看到所有停车点
- "骑行想跨公园跑长一点" → "骑行" Routine → 跨公园组置顶

---

## 4. 核心设计决策

### 4.1 三层信息架构

| 层 | 内容 | 入口 | 何时显示 |
|---|---|---|---|
| **Routine（体验层）** | 日出/日落/晨跑/骑行 | 顶部 chip 栏 | 用户点击 Routine 后激活 |
| **Facility（设施层）** | 停车场、洗手间、步道轨迹 | 右上"设施图层"面板 | 用户 toggle 后常驻 |
| **Base（基础层）** | 公园名标签、底图 | 自动 | 始终显示 |

**关键差异** vs v9：v9 把所有类型 POI 平铺在一张图上，用户看到 50 多个 pin 不知道从哪开始。v10 默认空地图，让用户先选"要干嘛"（Routine），再出现相关 POI。

### 4.2 精选 + 穷举的 Routine 列表

**问题**：Routine 覆盖 14 个公园共 39 条 POI，全铺会稀释"推荐感"，但只显示 4 条精选又让其他公园"消失"。

**解决**：
- 每个 Routine 列表 = 精选（★ 橙色徽章）在前 + 其余穷举在后
- 精选标准：编辑手工挑选的"最能代表这个活动的地点"
- 列表右上角带徽章区分，用户可以选择"直接选推荐"或"往下翻穷举"

### 4.3 按时段的人流热图

v9 是单一"人流 高/中/低"标签。v10 改为 5 时段 × 2 日类型（工作日/周末）矩阵：

| | 5-8 | 8-11 | 11-14 | 14-17 | 17-20 |
|---|---|---|---|---|---|
| 工作日 | 闲 | 中 | 闲 | 中 | 挤 |
| 周末 | 中 | 挤 | 挤 | 挤 | 挤 |

**来源诚实声明**：热图上明确标注"基于公园描述整理，非实时观测（2026-04-20）"。不假装是实时数据。

### 4.4 数据可信度标记

所有 POI 描述带 `data_status` 字段：
- `unverified` → 卡片标题旁显示 ⚠️ "未核验" 徽章
- 未来核验过的会升级为 `verified`（带核验日期和来源）

### 4.5 UGC 反馈闭环（轻量版）

用户在卡片上看到"信息过时"按钮 → 点击 → 本地 `localStorage` 标记该 POI → 下次打开卡片底部按钮变色提醒。

**不上报到后端** —— 只是给用户自己一个"我标过这条"的记忆。未来接后端时改接口即可。

### 4.6 地图数据分层（OSM Tier）

Track（步道/骑行道）用 OSM 真实几何而不是虚构连线。不同公园的 OSM 覆盖度差异大：

| Tier | 覆盖度 | 公园 |
|---|---|---|
| tier1 | OSM 有完整 cycleway + footway | 深圳湾、红树林、前海石 |
| tier2 | 有 footway 无 cycleway | 南山、大南山、前海演艺 |
| tier3 | 仅骨架 | 荔香、中山 |
| (无 OSM) | 用公园中心点 marker 替代 | 青青世界、月亮湾等 6 园 |

不同 tier 在图例里用不同样式。

---

## 5. 数据模型

### 5.1 `parks.json`

```json
{
  "id": 1,
  "name": {"zh":"深圳湾公园", "en":"Shenzhen Bay Park"},
  "description": {...},
  "center": [22.52, 113.93],
  "crowd": "high",
  "crowdPattern": {
    "weekday": {"dawn":"low","morning":"med","noon":"low","afternoon":"med","evening":"high"},
    "weekend": {"dawn":"med","morning":"high","noon":"high","afternoon":"high","evening":"high"},
    "source": {"zh":"基于公园描述整理，非实时观测", "en":"Derived from park description, not live data"},
    "verified_at": "2026-04-20",
    "confidence": "low"
  },
  "facilities": {"sunrise":true,"sunset":true,"cycling":true,...},
  "osm_park_id": "shenzhenbay",
  "osm_tier": "tier1",
  "pois": [{
    "id": 101,
    "type": "sunrise",
    "label": {"zh":"日出剧场","en":"Sunrise Theater"},
    "coordinates": [22.52, 113.93],
    "tip": {"zh":"...", "en":"..."},
    "bestTime": {"zh":"6:00-6:40","en":"6:00-6:40"},
    "crowd": "medium",
    "data_status": "unverified"
  }]
}
```

### 5.2 `routines.json`

```json
{
  "id": "sunrise",
  "kind": "activity",
  "name": {"zh":"看日出","en":"Watch Sunrise"},
  "icon": "sunrise",
  "color": "#5B9BD5",
  "description": {"zh":"...","en":"..."},
  "entries": [{
    "park_id": 1,
    "poi_id": 101,
    "why": {"zh":"...","en":"..."},
    "featured": true
  }]
}
```

Cycling 特殊结构（有 `groups`）：

```json
{
  "id": "cycling",
  "groups": [
    {"id":"crosspark", "entries":[{"route_name":{...}, "featured":true, "parks_covered":[2,1]}]},
    {"id":"inpark", "entries":[{"park_id":1,"poi_id":103,"featured":true}]}
  ]
}
```

### 5.3 `tracks.geojson`

OSM Overpass 抓取的 cycleway + footway 几何，按 `park_id` 分组，共 622 features。每个 feature 的 `properties.kind` ∈ `{cycleway, footway}`。

---

## 6. 交互规则

### 6.1 点击 Routine Chip
1. 该 Chip 激活（黑底白字）
2. 左侧面板滑入（中央 460ms 动画）
3. 地图上只保留该 Routine 的 POI marker
4. 动画结束后 `map.invalidateSize()` + `fitBounds` 到涉及的 POI

### 6.2 点击 POI（marker 或列表条目）
1. 右下卡片滑入
2. 地图 pan 偏移，确保 POI 不被卡片遮挡（根据卡片和左面板尺寸计算 padding）
3. 卡片显示：类型徽章 / 标题 + 未核验标记 / 所属公园 / tip / meta chips / 人流热图 / 信息过时 + 导航按钮

### 6.3 语言切换
- 顶部"EN / 中"按钮切换 `<body>` class
- 所有 `{zh, en}` 字段通过 `tr()` 函数选择
- 不刷新页面

### 6.4 信息过时反馈
点击"信息过时" → `localStorage.stale` 添加该 POI.id → 按钮变金黄色 + "已标记"文案 → 再点取消。

---

## 7. 移动端设计

### 7.1 断点与设备定位

| 断点 | 对应设备 | 主要调整 |
|---|---|---|
| `> 900px` | 桌面 / 平板横屏 | 完整布局：顶栏 + 左侧 Routine 面板（360px） + 右下卡片（360px） + 右上设施图层 |
| `641-900px` | 平板竖屏 | 同桌面，卡片宽度略收窄 |
| `≤ 640px` | 手机（主战场） | 面板和卡片变底部 bottom-sheet；设施图层折叠到右上角 FAB |

### 7.2 手机端布局决策

**Routine chip 栏**（顶部）
- chip 字号、touch target 不变（保持 44px 以上可点击高度）
- 超过屏宽时横向滚动（已实现）
- chip 间距从 4px → 6px 改善误触

**Routine 面板**（原左滑 360px）
- 改为**底部 bottom sheet**：从底部滑入，占屏高 75%，顶部有拖动条（visual only）
- 用户往下拖 → 关闭；点背景 overlay 也关闭
- 内容区可滚动，和地图 pan 不冲突

**POI 卡片**（原右下 360px）
- 变全宽 bottom sheet：左右 12px 边距，底部离屏 12px（避开 iOS 手势横条）
- 卡片打开时不再 pan 地图（手机屏幕太小，pan 没意义），改为**缩短卡片高度**（只显示标题+tip，crowd 热图折叠），用户上滑展开完整内容
- 默认收起状态 → 展开状态 两个高度

**设施图层**（原右上常驻面板）
- 改为右上角单个 ⚙ FAB，点击后弹出菜单
- 避免常驻面板挡地图

### 7.3 触控标准

- 所有可点击元素最小 **44×44px**（Apple HIG 推荐）
- 按钮间距 ≥ 8px，避免误触
- 不使用 hover 状态（手机无 hover），改用 active/pressed 颜色变化

### 7.4 地图交互

- 地图区域 `touch-action: pan-x pan-y` 允许单指拖动和双指缩放
- Leaflet 已默认支持触摸
- marker 的 iconSize 从 30×30 → 36×36（手机上更易点中）

### 7.5 iOS Safari 特殊处理

- `viewport height`：使用 `100dvh` 而非 `100vh`，避开 Safari 地址栏折叠导致的高度抖动
- **safe area**：卡片底部 `padding-bottom: max(12px, env(safe-area-inset-bottom))`
- **禁用双击缩放**：`<meta name="viewport" ... user-scalable=no>`（已设置）
- **状态栏遮挡**：顶栏加 `padding-top: env(safe-area-inset-top)`

### 7.6 滚动冲突

| 场景 | 处理 |
|---|---|
| 在 Routine 列表内滚动 | 列表本身滚动，地图不响应 |
| 在 POI 卡片内滚动 | 卡片滚动，地图不响应 |
| 在地图上滑动 | 地图 pan |
| 边缘区域 | 列表滚到顶继续下拖 → 关闭 bottom sheet |

通过 `touch-action` + `overscroll-behavior: contain` 实现。

### 7.7 性能

- 手机上关闭 OSM track 的实时渲染（tier3 公园 footway 过多会卡），改为**激活该公园 Routine 时才加载**
- 首屏只加载视口内 park label，其他延迟

### 7.8 不做的（明确）

- **不做原生 App** —— 工作量不匹配验证价值
- **不做横屏特殊布局** —— 手机默认就是竖屏，横屏走平板布局即可
- **不做手势（双指旋转等）** —— Leaflet 默认够用
- **不做 PWA 安装** —— v10 用浏览器直接访问

---

## 8. 数据规模（v10.0）

| 指标 | 数量 |
|---|---|
| 公园 | 14 |
| POI | 71 |
| Routine | 4 |
| Routine 条目（含精选+穷举） | 40 |
| OSM Track features | 622 |

---

## 9. 非功能要求

- **性能**：首屏 < 2s（本地 JSON 无后端）；所有 JSON 合计 < 400KB
- **适配**：桌面 + 移动 H5（≤640px 布局调整，面板 88vw，卡片全宽 bottom）
- **离线**：首次加载后底图不缓存，但数据 JSON 可通过 Service Worker 缓存（v10 暂不实现）
- **无依赖外部后端**：所有数据静态部署
- **部署**：GitHub Pages `/nanshan-parks/v10/` 子路径

---

## 10. 明确不承诺的

- 人流数据不实时
- POI 描述（v10.0）均未核验，是 v9 LLM 生成沿用
- OSM 轨迹数据版本 2026-04-20，之后可能过期
- UGC "信息过时" 只是本地记号，不回传任何服务

---

## 10. 后续迭代方向

### v10.1（近期）
- 为精选 POI 做实地核验（Bella + 朋友圈小范围）
- 接真实 crowd 数据（如果能找到源）

### v11（架构）
- 评论 / 打分功能（但必须先解决账号和 spam 问题）
- 多城市扩展（福田、宝安）
- 和 Klook POI 系统对齐 schema（为 GEO 项目铺路）

---

## 附：与 v9 对照

| 维度 | v9 | v10 |
|---|---|---|
| 顶部功能 | 已探索计数器 | Routine chip 栏 |
| POI 类型 | 全平铺在图上 | 按 Routine 激活时才显示 |
| 人流信息 | 单一标签 | 5 时段 × 2 日类型热图 |
| 设施图层 | 和 POI 混在一起 | 独立面板（停车/洗手间/轨迹） |
| 图标 | 混用 emoji + 自绘 | Lucide 统一线性图标 |
| 数据来源标注 | 无 | 每层都带 source + verified_at + confidence |
| 用户反馈 | 无 | localStorage 信息过时标记 |
| Track 数据 | 虚构连线 | OSM 真实几何 |
| Cross-park 路线 | 混在普通 POI 中 | 独立分组，置顶显示 |
