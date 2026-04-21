# nanshan-parks-v10 — 项目说明

> 继承 `~/Documents/AI cowork space/CLAUDE.md`。本文件只写本项目专属上下文。

## 是什么

南山公园导览产品 v10 本地版。单页应用，纯静态（Leaflet + 原生 JS），无后端。
公共 v9 在 https://bellazhuang417-cyber.github.io/nanshan-parks/ ，v10 发布到同仓库 `/v10/` 子路径（尚未发布）。

## 目录结构

```
nanshan-parks-v10/
├── index.html           # 主应用（单文件，CSS+JS 内联）
├── app.html             # 调试期副本，可删
├── debug.html           # 诊断页，3 个 JSON endpoint 健康检查
├── data/
│   ├── parks.json       # 14 个公园 + 71 POI（含 crowdPattern / data_status）
│   ├── routines.json    # 4 个 Routine（sunrise/sunset/morning_run/cycling）
│   └── tracks.geojson   # OSM 抓的 cycleway+footway 几何，622 features
└── docs/
    └── PRD_v10.md       # 完整 PRD（含 §7 移动端设计）
```

## 关键设计决策（改代码前必读）

1. **全局 `L` 是 Leaflet** —— 不能定义同名函数。i18n helper 叫 `tr(obj)`，不是 `L()`。
2. **精选 vs 穷举** —— routines.json 每个 entry 有 `featured:true|false` 字段；渲染时精选在前 + 橙色 ★ 徽章。
3. **数据真实性是最高原则** —— 所有 LLM 生成内容带 `data_status:'unverified'` + ⚠️ 标记；crowdPattern 有 `source.zh/en` + `verified_at` + `confidence:'low'`，绝不假装实时。
4. **Routine 面板动画 460ms** —— `map.invalidateSize()` 和 `fitToRoutine()` 必须在动画结束后调用，否则 Leaflet 用旧容器尺寸算 tile，出现灰方块。
5. **打开 POI 卡片时** —— 调 `panIntoVisible(latLng)` 把 POI 移到"扣除卡片/面板后的可视中心"，否则中文卡片过高会盖住 marker。
6. **跨园骑行分组**（routines.json → cycling.groups） —— `crosspark` 必须渲染在 `inpark` 前面。

## 数据多语言约定

所有面向用户的字段都是 `{zh:'中文', en:'English'}`。新增字段时务必双语，否则语言切换会漏译。渲染走 `tr(obj)`。

## 本地开发

```bash
cd nanshan-parks-v10
python3 -m http.server 8787
# 打开 http://localhost:8787/index.html
```

## 未完成

- Task #12 移动端交互适配（PRD §7 已定标准，代码待按标准检查）
- Task #13 发布到 GitHub Pages `/v10/`

## 明确不做

- 不引入构建工具（Vite/Webpack 等），保持单 HTML 可直接打开
- 不接任何后端 / 账号系统
- 不伪造实时数据，任何无法核验的描述都要带 unverified 标记
