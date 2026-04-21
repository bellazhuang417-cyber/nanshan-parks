# 南山公园探索者 · Nanshan Parks Explorer

🌲 深圳南山 14 个公园的交互地图 · Interactive map of 14 parks in Nanshan, Shenzhen.

## 在线访问 · Live

| 版本 Version | 链接 URL | 说明 Notes |
|---|---|---|
| **v10**（推荐 / recommended） | https://bellazhuang417-cyber.github.io/nanshan-parks/v10/ | 按活动意图（Routine）组织的新版 |
| v9 | https://bellazhuang417-cyber.github.io/nanshan-parks/ | 旧版，POI 平铺视图 |

---

## 中文

**探索南山** 是一个帮你在南山 14 个公园中快速找到"想去哪"的工具。

核心改动（v10 相对 v9）：
- **按活动意图选公园**：日出 / 日落 / 晨跑 / 骑行四条 Routine，点击即可看到所有推荐地点
- **精选 + 穷举**：编辑精选地点排前面带 ★ 标记，其他地点也完整列出不遗漏
- **按时段人流预估**：5 时段 × 工作日/周末 的热图，来源标注清晰（非实时）
- **设施图层独立**：停车场、洗手间、步道轨迹和 Routine 分开，互不干扰
- **数据真实性**：未核验的 POI 描述带 ⚠️ 标记，轨迹数据来自 OpenStreetMap 真实几何
- **移动端适配**：手机上面板和卡片变底部 bottom sheet，支持下拉关闭

完整设计见 [`/v10/docs/PRD_v10.md`](./v10/docs/PRD_v10.md)。

## English

**Nanshan Parks Explorer** helps you find *"where to go for what you want to do"* across 14 parks in Shenzhen Nanshan district.

Key changes (v10 vs v9):
- **Intent-driven navigation**: Four Routines (Sunrise / Sunset / Morning Run / Cycling), each surfacing recommended spots
- **Curated + exhaustive list**: Editorial picks first (★ badge), all other qualifying spots still listed
- **Time-slot crowd pattern**: 5 slots × weekday/weekend heatmap, with clear source labeling (not real-time)
- **Separated facility layer**: Parking, restrooms, paths isolated from Routine POIs
- **Data honesty**: Unverified POI descriptions are tagged; track geometry uses real OpenStreetMap data
- **Mobile-first**: Bottom sheets on phone with drag-to-close

See [`/v10/docs/PRD_v10.md`](./v10/docs/PRD_v10.md) for the full spec.

---

## 数据说明 · Data Notes

- 公园 / POI 描述：v10.0 沿用 v9 的 LLM 生成内容，未经实地核验，带 `unverified` 标记 · Park and POI descriptions are LLM-generated and unverified in v10.0
- 步道/骑行道轨迹：OpenStreetMap Overpass API（2026-04-20） · Paths from OpenStreetMap Overpass API
- 人流预估：基于公园描述的启发式映射，非实时 · Crowd pattern is derived from descriptions, not real-time

## 更新方式 · Deploy

修改对应版本目录后推 `main` 分支，GitHub Pages 自动发布。v9 保留在根目录不动，新版本放在子路径（如 `/v10/`）。
Push to `main` — GitHub Pages auto-deploys. v9 stays at root; new versions live in subpaths (e.g. `/v10/`).
