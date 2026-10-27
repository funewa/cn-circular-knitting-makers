---
name: cn-circular-knitting-makers
description: Build a no-fabrication, bilingual (中/EN) CSV of China circular-knitting-machine (针织大圆机/圆纬机) manufacturers, scaling toward ~300 verified host-machine makers. Use when the user wants a list/database of Chinese circular knitting machine / weft circular knitting machine / 大圆机 / 针织圆机 factories, exporters, or OEM makers.
version: 1.0.0
authors:
  - WorkBuddy user (Fuzhou cat-scratching-board OEM, overseas B2B lead-gen context)
---

# China Circular-Knitting-Machine Makers — Verified CSV Miner

## Goal
Produce / extend `china_circular_knitting_machine_makers.csv`: top ~300 China-based
manufacturers of **circular knitting machines (针织圆形纬编机 / 针织大圆机 / 圆纬机)**,
bilingual (中文 + English), UTF-8 BOM, 14 columns, **zero fabrication** — every row
must have a real source URL and survive a second-source check.

## Hard rules (learned the hard way)
1. **No fabrication.** Append a company ONLY if a real source URL confirms it makes
   circular knitting machines. Never invent a Chinese name to fill a row.
2. **Reject non-host-machine entities**, even if they appear in "circular knitting
   machine" searches:
   - Fabric mills / 织布厂 that only *use* machines (often hold 大圆机 patents but are
     users, not makers) → exclude.
   - Traders / agents (e.g. distributors of Santoni/MECMOR) → exclude.
   - Parts/accessory makers (大圆机三角/cams, 针筒, 加油机, 除尘, 针织机配件) → exclude
     to keep the "complete host-machine" scope pure.
   - **Baike "related companies" auto-suggest lists are usually UNRELATED machinery**
     (woodworking / casting / stone / shoe / paper / construction equipment) → verify
     each, expect most to be junk.
3. **Dedup on Chinese name (col 0).** Watch for 全角/半角 parenthesis duplicates
   (e.g. `圆达(厦门)` vs `圆达（厦门）`) — normalize and drop one.
4. CSV format: 14 columns
   `公司名称(中文), Company Name (EN), 品牌(中/英), 省份城市(中文), Region (EN),
   企业性质, Ownership, 主营产品(中文), Main Products (EN), 成立年份, 资质与备注,
   Notes, 数据可信度, 来源URL`.
   Write with `utf-8-sig` + `csv.QUOTE_ALL`; validate every row == 14 cols.

## Channels that WORK (use in this order)
1. **CTMA 圆纬机行业分会 108-member roster** —
   `https://ctma.net/contents/401/1226.html` (authoritative member list; great for
   catching missed host makers).
2. **Alibaba.com international** (English挂牌名 → Chinese registered name):
   - `https://www.alibaba.com/companies?SearchText=circular+knitting+machine`
   - `...SearchText=jacquard+knitting+machine`, `...SearchText=rib+knitting+machine`
   - Harvest seller display names; many small OEMs only surface here under EN names.
3. **Made-in-China.com** / **1688** / general **WebSearch** with keyword variants:
   `圆机` `针织圆机` `大圆机` `提花机` `罗纹机` + province/town qualifiers.
4. **China national patent applicants** (Google Patents / 佰腾 / 企查查专利) —
   search 大圆机/圆纬机/针筒/三角座/电脑提花/毛圈毛巾/调线嵌花 实用新型 →
   **申请人 = real host-maker name**. Strong for micro/OEM factories with no website.
5. **User-supplied name batches** — highest precision; the user (industry insider)
   names small makers (东崛, 宏祥, 佰源, 凯星, 欧大纬…) that search never surfaces.

## Channel that does NOT work for bulk
- **AnySearch `business.company`** is a *per-company* lookup (by name / credit code /
  LEI / ticker). It CANNOT bulk-pull "all companies whose 经营范围 contains 针织大圆机".
  Do not waste calls trying to use it as a bulk source.

## The ONLY path to ~300+ (bulk pull)
Free web + Alibaba + patents + CTMA yields ~150–160 *verifiable host makers* — a hard
ceiling, because small/OEM factories aren't indexed. To scale:
- **Connect 企查查 (qcc) / 天眼查 (tyc) connector** (user must authorize in the
  connector panel, or supply an API key). Then bulk-pull by 经营范围
  `针织大圆机 | 圆纬机 | 针织机械` and merge + dedup. This is the highest-leverage
  step and is currently the blocker.

## Workflow
1. Grep the CSV for candidate names BEFORE searching (dedup early).
2. Harvest names from the channels above; for each new name, run a second-source
   WebSearch to confirm it's a host-machine maker; reject traders/fabric-mills/parts.
3. Append via a Python script (utf-8-sig, QUOTE_ALL) that dedups on col 0 and asserts
   14 columns; print added/skipped/total.
4. Present the file with `present_files`.
5. Keep digging province-by-province (福建泉州/漳州/厦门, 浙江绍兴/诸暨/上虞, 江苏无锡/常州,
   广东汕头/佛山/东莞, 山东青岛/烟台/威海) and log each round to
   `.workbuddy/memory/YYYY-MM-DD.md`.
