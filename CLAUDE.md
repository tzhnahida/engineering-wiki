# LLM Wiki — Schema & Workflow

This file encodes how the LLM wiki operates. It defines the directory structure, conventions, and workflows for ingesting sources, answering queries, and maintaining the wiki.

## Directory Structure

```
_llm/
  raw/
    assets/           # Images and media downloaded from sources
    Datasheet.lnk     # Windows shortcut → F:\Projects\PCB\Library\Datasheet\ (chip datasheets)

主页.md               # Human entry point — categorized catalog
log.md                # Chronological, append-only operation record

元件/                 # Component pages — ICs, connectors, passives (flat, no subfolders)
知识/                 # Concept/knowledge pages — by discipline subfolder
  LLM-AI/             # LLM methodology, prompt engineering, chain-of-thought
  电源电子/           # DC-DC, COT, EMI, efficiency, protection, compensation
  研究方法/           # Research methodology, project evaluation
  姿态解算/           # Madgwick AHRS, ESKF
  通讯网络/           # IEEE 802.11 TSF, network protocols
  导航定位/           # ZUPT, SlimeVR protocol
  计算机视觉/         # Stereo vision, calibration, ellipse fitting, face mesh
  视频显示/           # HDMI protocol, physical layer, TMDS, video, EDID
  知识总览.md         # Knowledge index — one table per discipline
来源/                 # Source summary pages — one per ingested document
项目/                 # Project pages — organized by platform/domain
参考/                 # Reference materials
  ├── 论文/            # Academic papers (PDF)
  ├── 标准/            # Standards & specifications (IEEE, HDMI, etc.)
  └── 元件总表.md       # Datasheet inventory

Clippings/            # Web Clipper output — staging area; processed items MUST move to 参考/ or other permanent locations

CLAUDE.md             # This file — the complete rulebook
```

> [!note] Source PDFs Convention
> - Academic papers → `参考/论文/`
> - Standards/specifications (IEEE, HDMI, etc.) → `参考/标准/`
> - Chip datasheets → `F:\Projects\PCB\Library\Datasheet\` (via `_llm/raw/Datasheet.lnk`)
> Each knowledge page's `sources:` frontmatter links to the corresponding `来源/` summary page, which in turn references the PDF.
> `_llm/raw/` is the LLM staging workspace — source files move to `参考/` after ingestion.

> [!note] Path Mapping
> `raw/Datasheet.lnk` is a Windows shortcut pointing to `F:\Projects\PCB\Library\Datasheet\`.
> When accessing datasheets, use the absolute path `F:\Projects\PCB\Library\Datasheet\` directly.
> The files in that directory follow the naming convention: `Chip_<Category>_<SubCategory>_<PartNumber>.PDF`

## Naming Conventions

- **Component pages** (`元件/`): `[Part Number](Part Number.md)` → file `Part Number.md`
- **Knowledge pages** (`知识/<discipline>/`): `[Topic Name](Topic Name.md)` → file `Topic Name.md`
- **Source summaries** (`来源/`): `[YYYY-MM-DD - Source Title](YYYY-MM-DD - Source Title.md)` → file `YYYY-MM-DD - Source Title.md`
- **Project pages** (`项目/`): `[Project Name](Project Name.md)` → file `Project Name.md`
- **Datasheet files**: `Chip_<Category>_<SubCategory>_<PartNumber>.PDF` — e.g. `Chip_DCDC_TPS54302.PDF`

Use `PascalCase` or `Natural Language` names for wiki pages — whatever reads naturally. Keep names consistent once chosen. Use Obsidian wiki-links (`[...](....md)`) for all internal cross-references.

## Page Format

### Knowledge pages

```yaml
---
type: concept
tags: [tag1, tag2]
created: 2026-06-26
updated: 2026-06-26
sources: ["[2026-06-26 - Source Title](2026-06-26 - Source Title.md)"]   # MANDATORY — links to 来源/ summaries
---
```

### Component pages

```yaml
---
type: entity
tags: [electronics, category, subcategory]
created: 2026-06-07
updated: 2026-06-17
sources: ["[2026-06-26 - Source Title](2026-06-26 - Source Title.md)"]   # Optional — datasheet source
---
```

### Source summaries

```yaml
---
type: source
tags: [domain, format]
created: 2026-06-26
updated: 2026-06-26
---
```

The body should use clear markdown headings (## level for sections), wiki-links to other pages, `> [!note]` callouts for observations, and Mermaid diagrams (not ASCII art).

## Cross-Referencing Rules (CRITICAL)

```
参考/论文/ (PDF)  →  来源/ (summary)  →  知识/ (concept)  ←  项目/ (project)
                                                              ↖  参见 links
```

| From | To | Allowed? |
|------|----|----------|
| 知识/ | 元件/ | ✅ 互链 — concepts reference components they apply to |
| 知识/ | 知识/ | ✅ 交叉引用 — e.g. HDMI 协议概述 → HDMI 物理层 |
| 知识/ | 来源/ | ✅ MANDATORY — every knowledge page must have `sources:` |
| 知识/ | 项目/ | ❌ FORBIDDEN — knowledge is pure, no project context |
| 项目/ | 知识/ | ✅ 项目页 `参见` 索引所用知识 |
| 项目/ | 元件/ | ✅ 项目 BOM 引用元器件 |
| 元件/ | 知识/ | ✅ 如 HDMI Type A → HDMI 物理层 |
| 来源/ | 参考/论文/ | ✅ 来源摘要引用 PDF 文件名 |
| log.md | ANY | ❌ Never use wikilinks in log.md |

**Principle**: Knowledge flows one direction — from sources to concepts to projects. Knowledge pages are pure domain knowledge; projects apply knowledge but knowledge never references projects.

## Workflows

### Ingest

When the user adds new content (from `Clippings/` or external):

1. **Read** the source document
2. **Discuss** key takeaways with the user — what to emphasize, what connections matter
3. **Move** source PDF to `参考/论文/` (if applicable)
4. **Write** a source summary in `来源/`
5. **Create/update** knowledge pages in `知识/<discipline>/`
6. **Fill** `sources:` frontmatter in every affected knowledge page
7. **Update** `知识/知识总览.md` with new entries
8. **Update** `主页.md` with any new or significantly changed pages
9. **Append** an entry to `log.md`

### Query

When the user asks a question:

1. **Read** `主页.md` to find relevant pages
2. **Read** the identified pages for detailed content
3. **Synthesize** an answer with citations to wiki pages (use wiki-links)
4. **File valuable answers** back into `知识/` or `项目/` as new pages

### Lint

Periodically (or on request), health-check the wiki:

- Look for **contradictions** between pages
- Identify **stale claims** superseded by newer sources
- Find **orphan pages** with no inbound links
- Note **missing pages** for important concepts mentioned but not defined
- Check **all knowledge pages have `sources:` filled**
- Verify **no knowledge page links to project pages**
- Validate **all wikilinks resolve** (no broken links)
- Suggest **new questions** and **new sources** to explore
- Update `主页.md` and `log.md` with lint results

## Index File Formats

### `主页.md` — categorized entry catalog

```markdown
# Wiki 主页
| | | |
|---|---|---|
| 🗂️ **项目** | [项目总览](项目总览.md) | Project index |
| 🔩 **元件** | [元件总表](元件总表.md) | Component datasheet index |
| 📖 **知识** | [知识总览](知识总览.md) | Knowledge index by discipline |

## 知识分类
| 领域 | 入口 | 涵盖 |
|------|------|------|

## 活跃项目
- [Project Name](Project Name.md) — Description
```

### `知识/知识总览.md` — knowledge index by discipline

```markdown
# 知识总览

## Discipline Name
| 页面 | 内容 | 来源 |
|------|------|------|
| [subfolder/Page Name](subfolder/Page Name.md) | One-line summary | [Source](Source.md) |
```

## Log File Format

`log.md` is append-only, chronological (newest first). Each entry:

```markdown
## [YYYY-MM-DD] type | Subject
```

Types: `init`, `ingest`, `query`, `lint`, `maintain`, `redesign`, `correction`, `audit`, `knowledge`, `project`, `design`, `concept`

**Never use wikilinks in log.md.** Log entries are for human readers only. Wikilinks pollute the graph view.

## Git Conventions

- **Commit language is English.** All commit messages MUST be written in English, following international convention. Use [Conventional Commits](https://www.conventionalcommits.org/) format: `<type>(<scope>): <description>` — e.g. `feat(knowledge): add DC-DC compensation theory`, `fix(links): repair broken cross-references`.
- **Commit every change.** After ANY file modification (creating, editing, or deleting wiki pages), commit immediately with a descriptive English message, then push.
  - One logical change per commit — don't batch unrelated edits.
  - After a multi-step workflow (e.g., a full ingest), commit at each meaningful checkpoint.
- **Push after commit.** Always `git push` after committing. The remote is the source of truth.

## Clippings Lifecycle

`Clippings/` is a **transient staging area**, not permanent storage. The Web Clipper drops content here; the LLM processes it promptly:

1. Content arrives in `Clippings/` → treat as an ingest trigger
2. After ingestion, the source file **MUST move** to its permanent home:
   - PDFs → `参考/论文/` (or `_llm/raw/` for datasheets)
   - Markdown articles → summarize into `来源/`, then delete or archive
   - Images/media → `_llm/raw/assets/`
3. **Never leave processed files in Clippings.** An empty Clippings folder means everything is triaged.

## Principles

- **Raw sources are immutable.** Never modify source PDFs.
- **You own the wiki.** You create, update, and maintain all wiki pages. The user reads and directs.
- **The wiki compounds.** Every ingest, query, and lint should leave the wiki richer than before.
- **Cross-reference generously.** Use wiki-links freely — except: knowledge→project (forbidden), log.md (forbidden).
- **Sources are mandatory.** Every knowledge page must have `sources:` frontmatter linking to `来源/`.
- **Knowledge is pure.** No project-specific context in knowledge pages — projects index knowledge, not vice versa.
- **PDFs live in `参考/论文/`.** After ingestion, source documents move there from staging.
- **When in doubt, write it down.** If information is worth noting, structure it into the wiki.
- **Clippings is staging, not storage.** Content dropped into `Clippings/` via Web Clipper must be ingested and moved out. Never let clippings accumulate unprocessed.
