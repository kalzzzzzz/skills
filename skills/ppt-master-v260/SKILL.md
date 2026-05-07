---
name: ppt-master
description: >
  AI-driven multi-format SVG content generation system. Converts source documents
  (PDF/DOCX/URL/Markdown) into high-quality SVG pages and exports to PPTX through
  multi-role collaboration. Use when user asks to "create PPT", "make presentation",
  "生成PPT", "做PPT", "制作演示文稿", or mentions "ppt-master".
metadata: { "version": "1.8", "copaw": { "emoji": "📑" } }
---

# PPT Master Skill

> AI-driven multi-format SVG content generation system. Converts source documents into high-quality SVG pages through multi-role collaboration and exports to PPTX.

**Core Pipeline**: `Source Document → Create Project → Template Option → Strategist → [Image_Generator] → Executor → Post-processing → Export`

> [!CAUTION]
>
> ## 🚨 Global Execution Discipline (MANDATORY)
>
> **This workflow is a strict serial pipeline. The following rules have the highest priority — violating any one of them constitutes execution failure:**
>
> 1. **SERIAL EXECUTION** — Steps MUST be executed in order; the output of each step is the input for the next. Non-BLOCKING adjacent steps may proceed continuously once prerequisites are met, without waiting for the user to say "continue"
> 2. **BLOCKING = HARD STOP** — Steps marked ⛔ BLOCKING require a full stop; the AI MUST wait for an explicit user response before proceeding and MUST NOT make any decisions on behalf of the user
> 3. **NO CROSS-PHASE BUNDLING** — Cross-phase bundling is FORBIDDEN. (Note: the Eight Confirmations in Step 4 are ⛔ BLOCKING — the AI MUST present recommendations and wait for explicit user confirmation before proceeding. Once the user confirms, all subsequent non-BLOCKING steps — design spec output, SVG generation, speaker notes, and post-processing — may proceed automatically without further user confirmation)
> 4. **GATE BEFORE ENTRY** — Each Step has prerequisites (🚧 GATE) listed at the top; these MUST be verified before starting that Step
> 5. **NO SPECULATIVE EXECUTION** — "Pre-preparing" content for subsequent Steps is FORBIDDEN (e.g., writing SVG code during the Strategist phase)
> 6. **NO SUB-AGENT SVG GENERATION** — Executor Step 6 SVG generation is context-dependent and MUST be completed by the current main agent end-to-end. Delegating page SVG generation to sub-agents is FORBIDDEN
> 7. **SEQUENTIAL PAGE GENERATION ONLY** — In Executor Step 6, after the global design context is confirmed, SVG pages MUST be generated sequentially page by page in one continuous pass. Grouped page batches (for example, 5 pages at a time) are FORBIDDEN
> 8. **SPEC_LOCK RE-READ PER PAGE** — Before generating each SVG page, Executor MUST `read_file <project_path>/spec_lock.md`. All colors / fonts / icons / images MUST come from this file — no values from memory or invented on the fly. Executor MUST also look up the current page's `page_rhythm` tag and apply the matching layout discipline (`anchor` / `dense` / `breathing` — see executor-base.md §2.1). This rule exists to resist context-compression drift on long decks and to break the uniform "every page is a card grid" default
> 9. **默认使用中药控股模板** — 当用户未提及任何模板时，默认使用`tcm_company_template`（中药控股模板）。仅在用户明确要求"自由设计"或"不用模板"时，才进入自由设计路径。

> [!IMPORTANT]
>
> ## 🌐 Language & Communication Rule
>
> - **Response language**: match the user's input and source materials. Explicit user override (e.g., "请用英文回答") takes precedence.
> - **Template format**: `design_spec.md` MUST follow its original English template structure (section headings, field names) regardless of conversation language. Content values may be in the user's language.

> [!IMPORTANT]
>
> ## 🔌 Compatibility With Generic Coding Skills
>
> - `ppt-master` is a repository-specific workflow, not a general application scaffold
> - Do NOT create `.worktrees/`, `tests/`, branch workflows, or generic engineering structure by default
> - On conflict with a generic coding skill, follow this skill unless the user explicitly says otherwise

## Main Pipeline Scripts

| Script                                             | Purpose                                                                                                                                                   |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `${SKILL_DIR}/scripts/source_to_md/pdf_to_md.py`   | PDF to Markdown                                                                                                                                           |
| `${SKILL_DIR}/scripts/source_to_md/doc_to_md.py`   | Documents to Markdown — native Python for DOCX/HTML/EPUB/IPYNB, pandoc fallback for legacy formats (.doc/.odt/.rtf/.tex/.rst/.org/.typ)                   |
| `${SKILL_DIR}/scripts/source_to_md/excel_to_md.py` | Excel workbooks to Markdown — supports .xlsx/.xlsm; legacy .xls should be resaved as .xlsx                                                                |
| `${SKILL_DIR}/scripts/source_to_md/ppt_to_md.py`   | PowerPoint to Markdown                                                                                                                                    |
| `${SKILL_DIR}/scripts/source_to_md/web_to_md.py`   | Web page to Markdown                                                                                                                                      |
| `${SKILL_DIR}/scripts/source_to_md/web_to_md.cjs`  | Node.js fallback for WeChat / TLS-blocked sites (use only if `curl_cffi` is unavailable; `web_to_md.py` now handles WeChat when `curl_cffi` is installed) |
| `${SKILL_DIR}/scripts/project_manager.py`          | Project init / validate / manage                                                                                                                          |
| `${SKILL_DIR}/scripts/analyze_images.py`           | Image analysis                                                                                                                                            |
| `${SKILL_DIR}/scripts/image_gen.py`                | AI image generation (multi-provider)                                                                                                                      |
| `${SKILL_DIR}/scripts/svg_quality_checker.py`      | SVG quality check                                                                                                                                         |
| `${SKILL_DIR}/scripts/total_md_split.py`           | Speaker notes splitting                                                                                                                                   |
| `${SKILL_DIR}/scripts/finalize_svg.py`             | SVG post-processing (unified entry)                                                                                                                       |
| `${SKILL_DIR}/scripts/svg_to_pptx.py`              | Export to PPTX                                                                                                                                            |
| `${SKILL_DIR}/scripts/update_spec.py`              | Propagate a `spec_lock.md` color / font_family change across all generated SVGs                                                                           |

For complete tool documentation, see `${SKILL_DIR}/scripts/README.md`.

## Template Index

| Index                   | Path                                                | Purpose                                                                                                                     |
| ----------------------- | --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Layout templates        | `${SKILL_DIR}/templates/layouts/layouts_index.json` | Query available page layout templates                                                                                       |
| Visualization templates | `${SKILL_DIR}/templates/charts/charts_index.json`   | Query available visualization SVG templates (charts, infographics, diagrams, frameworks)                                    |
| Icon library            | `${SKILL_DIR}/templates/icons/`                     | See `${SKILL_DIR}/templates/icons/README.md`; search icons on demand with `ls templates/icons/<library>/ \| grep <keyword>` |

## Standalone Workflows

| Workflow          | Path                           | Purpose                                                                                                         |
| ----------------- | ------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| `create-template` | `workflows/create-template.md` | Standalone template creation workflow                                                                           |
| `verify-charts`   | `workflows/verify-charts.md`   | Chart coordinate calibration — run after SVG generation if the deck contains data charts                        |
| `visual-edit`     | `workflows/visual-edit.md`     | Browser-based visual editor for fine-grained edits — run only when the user explicitly requests it after export |

---

## Workflow

### Step 1: Source Content Processing

🚧 **GATE**: User has provided source material (PDF / DOCX / EPUB / URL / Markdown file / text description / conversation content — any form is acceptable).

When the user provides non-Markdown content, convert immediately:

| User Provides                     | Command                                                                                                                                                             |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PDF file                          | `python3 ${SKILL_DIR}/scripts/source_to_md/pdf_to_md.py <file>`                                                                                                     |
| DOCX / Word / Office document     | `python3 ${SKILL_DIR}/scripts/source_to_md/doc_to_md.py <file>`                                                                                                     |
| XLSX / XLSM / Excel workbook      | `python3 ${SKILL_DIR}/scripts/source_to_md/excel_to_md.py <file>`                                                                                                   |
| CSV / TSV                         | Read directly as plain-text table source                                                                                                                            |
| PPTX / PowerPoint deck            | `python3 ${SKILL_DIR}/scripts/source_to_md/ppt_to_md.py <file>`                                                                                                     |
| EPUB / HTML / LaTeX / RST / other | `python3 ${SKILL_DIR}/scripts/source_to_md/doc_to_md.py <file>`                                                                                                     |
| Web link                          | `python3 ${SKILL_DIR}/scripts/source_to_md/web_to_md.py <URL>`                                                                                                      |
| WeChat / high-security site       | `python3 ${SKILL_DIR}/scripts/source_to_md/web_to_md.py <URL>` (requires `curl_cffi`; falls back to `node web_to_md.cjs <URL>` only if that package is unavailable) |
| Markdown                          | Read directly                                                                                                                                                       |

**✅ Checkpoint — Confirm source content is ready, proceed to Step 2.**

---

### Step 2: Project Initialization

🚧 **GATE**: Step 1 complete; source content is ready (Markdown file, user-provided text, or requirements described in conversation are all valid).

```bash
python3 ${SKILL_DIR}/scripts/project_manager.py init <project_name> --format <format>
```

Format options: `ppt169` (default), `ppt43`, `xhs`, `story`, etc. For the full format list, see `references/canvas-formats.md`.

Import source content (choose based on the situation):

| Situation                                   | Action                                                                                                    |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Has source files (PDF/MD/etc.)              | `python3 ${SKILL_DIR}/scripts/project_manager.py import-sources <project_path> <source_files...> --move`  |
| User provided text directly in conversation | No import needed — content is already in conversation context; subsequent steps can reference it directly |

> ⚠️ **MUST use `--move`** (not copy): all source files — Step 1's generated Markdown, original PDFs / MDs / images — go into `sources/` via `import-sources --move`. After execution they no longer exist at the original location. Intermediate artifacts (e.g., `_files/`) are handled automatically.

**✅ Checkpoint — Confirm project structure created successfully, `sources/` contains all source files, converted materials are ready. Proceed to Step 3.**

---

### Step 3: Template Option

🚧 **GATE**: Step 2 complete; project directory structure is ready.

---

**执行路径判定**（按优先级从上到下匹配，命中即停止）：

| 优先级 | 用户意图判断                                                                    | 执行路径                               |
| ------ | ------------------------------------------------------------------------------- | -------------------------------------- |
| 1      | 用户**指定了具体模板名称**（如"用 mckinsey 模板"、"academic_defense"）          | → **路径 B：用户指定模板**             |
| 2      | 用户**明确要求自由设计**（如"自由设计"、"free design"、"不用模板"、"不加模板"） | → **路径 C：自由设计**                 |
| 3      | 用户**询问有哪些模板**（如"有哪些模板"、"template list"）                       | → **路径 D：列出模板**                 |
| 4      | 以上均不符合（即用户未提及模板相关话题）                                        | → **路径 A：默认模板（中药控股模板）** |

---

#### 路径 A：默认模板 — 中药控股模板（用户未提及模板时）

这是最常见的路径。用户没有提到"模板"二字，也没有指定风格 → 直接使用中药控股红色模板，**不询问，不等候**：

```bash
cp ${SKILL_DIR}/templates/layouts/tcm_company_template/*.svg <project_path>/templates/
cp ${SKILL_DIR}/templates/layouts/tcm_company_template/design_spec.md <project_path>/templates/
cp ${SKILL_DIR}/templates/layouts/tcm_company_template/*.png <project_path>/images/ 2>/dev/null || true
cp ${SKILL_DIR}/templates/layouts/tcm_company_template/*.jpg <project_path>/images/ 2>/dev/null || true
```

> ⚠️ `tcm_company_template` 模板路径直接硬编码，不通过 `layouts_index.json` 查询。

---

#### 路径 B：用户指定模板

当用户命名了具体模板或风格映射时触发：

1. 读取模板索引：`read_file ${SKILL_DIR}/templates/layouts/layouts_index.json`
2. 在 index 中匹配模板名称 / label / keywords
3. 若匹配成功，复制模板文件：

```bash
cp ${SKILL_DIR}/templates/layouts/<template_name>/*.svg <project_path>/templates/
cp ${SKILL_DIR}/templates/layouts/<template_name>/design_spec.md <project_path>/templates/
cp ${SKILL_DIR}/templates/layouts/<template_name>/*.png <project_path>/images/ 2>/dev/null || true
cp ${SKILL_DIR}/templates/layouts/<template_name>/*.jpg <project_path>/images/ 2>/dev/null || true
```

4. 若匹配失败（用户说的模板不存在），告知用户当前可用模板列表，并询问是改用 `zhongyao_red` 还是走自由设计 → ⛔ **BLOCKING**（等待用户选择后再继续）。

> 用户可能用中文描述风格而非精确名称（如 "麦肯锡那种"、"学术答辩样式"、"政府红头文件风格"），此时应从 `keywords` 和 `summary` 字段做语义匹配，给出最佳推荐并请用户确认。

---

#### 路径 C：自由设计（用户明确要求时）

用户明确表达了"自由设计"、"不用模板"、"free design" → 跳过模板复制步骤，直接进入 Step 4。

> ⚠️ 仅在用户**明确表达**时才走此路径。路径 A 的默认行为不可理解为"用户没说话就自由设计"。

---

#### 路径 D：用户询问模板列表

1. 读取并展示 `layouts_index.json` 中的所有模板（label + summary）
2. 额外补充说明：系统还有一个**默认模板 `tcm_company_template`（中药控股红色模板）**，当用户不提模板时默认使用
3. ⛔ **BLOCKING**：等待用户选择具体模板，或表示"自由设计"

---

#### 🛡️ 模板完整性验证（所有路径，进入 Step 4 前必须执行）

无论哪条路径，进入 Step 4 前执行以下验证，确保后续步骤有据可依：

| 路径               | 验证内容                                                                                               |
| ------------------ | ------------------------------------------------------------------------------------------------------ |
| A / B （使用模板） | 确认 `templates/` 目录下存在至少一个 `.svg` 文件和 `design_spec.md`；确认 `images/` 下模板背景图已就位 |
| C （自由设计）     | 确认 `templates/` 和 `images/` 目录均**为空**（或不存在模板遗留文件），避免 Step 4 误读旧模板          |
| D （询问列表）     | 等待用户选择后，回到对应路径执行                                                                       |

验证通过后，Agent 在对话中输出当前生效的模板名称（如 `当前模板：tcm_company_template`），以便用户确认。

---

**✅ Checkpoint — 模板路径已确定，模板文件已就位（或确认自由设计），`templates/` 和 `images/` 目录状态正确。输出当前模板名称后，直接进入 Step 4。**

### Step 4: Strategist Phase (MANDATORY — cannot be skipped)

🚧 **GATE**: Step 3 complete; free-design path taken, or (if triggered) template files copied into the project.

First, read the role definition:

```
Read references/strategist.md
```

> ⚠️ **Mandatory gate**: before writing `design_spec.md`, Strategist MUST `read_file templates/design_spec_reference.md` and follow its full I–XI section structure. See `strategist.md` Section 1.

**Eight Confirmations** (full template: `templates/design_spec_reference.md`):

⛔ **BLOCKING**: present the Eight Confirmations as a bundled recommendation set and **wait for explicit user confirmation or modification** before outputting Design Specification & Content Outline. This is the single core confirmation point — once confirmed, all subsequent steps proceed automatically.

1. Canvas format
2. Page count range
3. Target audience
4. Style objective
5. Color scheme
6. Icon usage approach
7. Typography plan
8. Image usage approach

If the user provided images, run analysis **before outputting the design spec**:

```bash
python3 ${SKILL_DIR}/scripts/analyze_images.py <project_path>/images
```

> ⚠️ **Image handling**: NEVER directly read / open / view image files (`.jpg`, `.png`, etc.). All image info comes from `analyze_images.py` output or the Design Spec's Image Resource List.

**Output**:

- `<project_path>/design_spec.md` — human-readable design narrative
- `<project_path>/spec_lock.md` — machine-readable execution contract (skeleton: `templates/spec_lock_reference.md`); Executor re-reads before every page

**🔍 模板一致性检查（Mandatory）**：

- 如果 Step 3 使用了模板（默认 `tcm_company_template` 或用户指定）：
  1. Strategist 必须先 `read_file <project_path>/templates/design_spec.md`，理解模板的设计约束（色板、字体、布局规则、页面节奏等）
  2. Eight Confirmations 中的 **Color scheme** 必须继承模板主色/辅色系，不可凭空自创
  3. Eight Confirmations 中的 **Typography plan** 必须继承模板字体方案
  4. Eight Confirmations 中的 **Style objective** 必须与模板风格定位一致
  5. 输出 `<project_path>/design_spec.md` 时需明确引用模板名称，标注"基于 xxx 模板"
  6. 输出 `<project_path>/spec_lock.md` 时，颜色/字体字段必须从模板 `design_spec.md` 提取，而非自行编造

- 如果 Step 3 为自由设计路径，跳过此检查，Strategist 自行确定设计方向。

**✅ Checkpoint — Phase deliverables complete, auto-proceed to next step**:

```markdown
## ✅ Strategist Phase Complete

- [x] Eight Confirmations completed (user confirmed)
- [x] Design Specification & Content Outline generated
- [x] Execution lock (spec_lock.md) generated
- [ ] **Next**: Auto-proceed to [Image_Generator / Executor] phase
```

---

### Step 5: Image Acquisition Phase (Conditional)

🚧 **GATE**: Step 4 complete; Design Specification & Content Outline generated and user confirmed.

> **Trigger**: At least one row in the resource list has `Acquire Via: ai` and/or `Acquire Via: web`. If every row is `user` or `placeholder`, skip to Step 6.

**Always load the common framework**:

```
Read references/image-base.md
```

Then **lazy-load the path-specific reference** for each row that actually needs it:

| Acquire Via            | Load reference (only if any such row exists) | Run                                                |
| ---------------------- | -------------------------------------------- | -------------------------------------------------- |
| `ai`                   | `references/image-generator.md`              | `python3 ${SKILL_DIR}/scripts/image_gen.py ...`    |
| `web`                  | `references/image-searcher.md`               | `python3 ${SKILL_DIR}/scripts/image_search.py ...` |
| `user` / `placeholder` | (skip)                                       | (skip)                                             |

A deck with only `ai` rows never loads `image-searcher.md`; a deck with only `web` rows never loads `image-generator.md`. A mixed deck loads both, processes each row through its own path, and writes both `image_prompts.md` and `image_sources.json`.

Workflow:

1. Extract all rows with `Status: Pending` and `Acquire Via ∈ {ai, web}` from the design spec
2. Generate prompts (ai rows) and/or run search (web rows) per [image-base.md](references/image-base.md) §2 dispatch table
3. Verify every row reaches a terminal status: `Generated` (ai success), `Sourced` (web success), or `Needs-Manual`

**✅ Checkpoint — Confirm acquisition attempted for every row, proceed to Step 6**:

```markdown
## ✅ Image Acquisition Phase Complete

- [x] image_prompts.md created (when any ai rows processed)
- [x] image_sources.json created (when any web rows processed)
- [x] Each row: status is `Generated` / `Sourced` / `Needs-Manual` (no `Pending` remaining)
```

> On acquisition failure, do NOT halt — follow the Failure Handling rule in [image-base.md](references/image-base.md) §5: retry once, then mark the row `Needs-Manual`, report to user, and continue to Step 6.

---

### Step 6: Executor Phase

🚧 **GATE**: Step 4 (and Step 5 if triggered) complete; all prerequisite deliverables are ready.

Read the role definition based on the selected style:

```
Read references/executor-base.md          # REQUIRED: common guidelines
Read references/shared-standards.md       # REQUIRED: SVG/PPT technical constraints
Read references/executor-general.md       # General flexible style
Read references/executor-consultant.md    # Consulting style
Read references/executor-consultant-top.md # Top consulting style (MBB level)
```

> Only read executor-base + shared-standards + one style file.

**Design Parameter Confirmation (Mandatory)**: before the first SVG, output key design parameters from the spec (canvas dimensions, color scheme, font plan, body font size). See executor-base.md §2.

**Per-page spec_lock re-read (Mandatory)**: before **each** SVG page, `read_file <project_path>/spec_lock.md` and use only its colors / fonts / icons / images. Resists context-compression drift on long decks. See executor-base.md §2.1.

> ⚠️ **Main-agent only**: SVG generation MUST stay in the current main agent — page design depends on full upstream context. Do NOT delegate to sub-agents.
> ⚠️ **Generation rhythm**: generate pages sequentially, one at a time, in the same continuous context. Do NOT batch (e.g., 5 per group).

**Visual Construction Phase**: generate SVG pages sequentially, one at a time, in one continuous pass → `<project_path>/svg_output/`

**Quality Check Gate (Mandatory)** — after all SVGs, BEFORE speaker notes:

```bash
python3 ${SKILL_DIR}/scripts/svg_quality_checker.py <project_path>
```

- Any `error` (banned SVG features, viewBox mismatch, spec_lock drift, etc.) MUST be fixed before proceeding — return to Visual Construction, regenerate that page, re-run check.
- `warning` entries (low-res image, non-PPT-safe font tail, etc.): fix when straightforward, otherwise acknowledge and release.
- Run against `svg_output/` (not after `finalize_svg.py` — finalize rewrites SVG and masks violations).

**Logic Construction Phase**: generate speaker notes → `<project_path>/notes/total.md`

**✅ Checkpoint — Confirm all SVGs and notes are fully generated and quality-checked. Proceed directly to Step 7 post-processing**:

```markdown
## ✅ Executor Phase Complete

- [x] All SVGs generated to svg_output/
- [x] svg_quality_checker.py passed (0 errors)
- [x] Speaker notes generated at notes/total.md
```

> **Chart pages?** If this deck contains data charts (bar / line / pie / radar / etc.), run the standalone [`verify-charts`](workflows/verify-charts.md) workflow before Step 7 to calibrate coordinates. AI models routinely introduce 10–50 px errors when mapping data to pixel positions; verify-charts eliminates that class of error. Skip if no chart pages.

---

### Step 7: Post-processing & Export

🚧 **GATE**: Step 6 complete; all SVGs generated to `svg_output/`; speaker notes `notes/total.md` generated.

🚧 **Image readiness GATE** (when Step 5 left ai rows in `Needs-Manual`): every expected file must exist at `project/images/<filename>` before running 7.1.

> If files are missing: PAUSE, list the missing filenames, point the user to `images/image_prompts.md` (each `### Image N:` block is paste-ready for ChatGPT / Gemini / Midjourney) and the required placement `project/images/<filename>`. Resume Step 7.1 only after all expected files are in place. `finalize_svg.py` and `svg_to_pptx.py` do not detect missing files at this layer — proceeding with gaps produces a deck with broken image references.

> ⚠️ Run the three sub-steps **one at a time** — each must complete successfully before the next.
> ❌ **NEVER** combine them into a single code block or shell invocation.

Canonical three-command pipeline (mirrors `references/shared-standards.md` §5):

**Step 7.1** — Split speaker notes:

```bash
python3 ${SKILL_DIR}/scripts/total_md_split.py <project_path>
```

**Step 7.2** — SVG post-processing (icon embedding / image crop & embed / text flattening / rounded rect to path):

```bash
python3 ${SKILL_DIR}/scripts/finalize_svg.py <project_path>
```

**Step 7.3** — Export PPTX (embeds speaker notes by default):

```bash
python3 ${SKILL_DIR}/scripts/svg_to_pptx.py <project_path>
# Output:
#   exports/<project_name>_<timestamp>.pptx           ← main native pptx (reads svg_output/, high fidelity)
#   backup/<timestamp>/<project_name>_svg.pptx        ← SVG preview pptx (reads svg_final/)
#   backup/<timestamp>/svg_output/                    ← Executor SVG source backup
```

> The two products now read from different sources by design: native pptx
> consumes `svg_output/` so the converter can preserve high-fidelity primitives
> (icon `<use>` placeholders, image `preserveAspectRatio` → `srcRect`, rounded
> rect `rx/ry` → `prstGeom roundRect`). The legacy/preview pptx still consumes
> `svg_final/` because PowerPoint's internal SVG parser cannot handle those
> primitives. Pass `-s output` or `-s final` to force a single source on both
> products if you need the older single-source behaviour.

**Optional animation flags** (the defaults already enable rich entrance animations — adjust only when the user asks for something different):

- `-t <effect>` — page transition. Default `fade`. Options: `fade` / `push` / `wipe` / `split` / `strips` / `cover` / `random` / `none`.
- `-a <effect>` — per-element entrance animation. Default `mixed` (auto-vary across the deck). Pass `none` to disable, or pick a specific effect like `fade`. Requires top-level `<g id="...">` groups (already required by Executor).
- `--animation-trigger {on-click,with-previous,after-previous}` — Start mode (matches PowerPoint's animation-pane Start dropdown). Default `after-previous` (click-free cascade; pace via `--animation-stagger`). Use `on-click` for presenter-paced reveals, or `with-previous` for all-at-once.
- `--auto-advance <seconds>` — kiosk-style auto-play.

**Optional recorded narration** (only when the user asks for narrated/video export):

Run the standalone [`generate-audio`](workflows/generate-audio.md) workflow. The AI picks a narration backend (`edge` by default, or a configured cloud provider such as ElevenLabs / MiniMax / Qwen / CosyVoice for high-quality or cloned voices), asks the user once (backend + voice + rate/settings + embed-or-not, all with recommended values), then executes `notes_to_audio.py` and (if chosen) re-exports the PPTX with `--recorded-narration audio`.

Do NOT call `notes_to_audio.py` directly without going through the workflow — `--voice` / `--voice-id` is required and the workflow produces the locale/provider-aware recommendation that makes the choice meaningful.

Full effect list, anchor logic, and limits: [`references/animations.md`](references/animations.md).

> ❌ **NEVER** substitute `cp` for `finalize_svg.py` — finalize performs multiple critical processing steps
> ❌ **NEVER** force `-s output` for the legacy/preview pptx (PowerPoint's internal SVG parser drops icons and rounded corners). The default auto-split already gives native the high-fidelity source it needs without touching legacy.
> ❌ **NEVER** use `--only` (it suppresses one of the two output files)

> Post-export iteration: whenever the user asks to change anything on a generated slide ("改一下", "调字号", "那里看着不对", "把图片换大点"), the [`visual-edit`](workflows/visual-edit.md) workflow is available — surface it as an option. If the user describes the change with enough specificity to apply directly ("第 3 页副标题字号改 32"), edit the SVG directly instead; if they're vaguely pointing at "somewhere" on the deck, run the workflow.

---

## Role Switching Protocol

Before switching roles, **MUST first read** the corresponding reference file. Output marker:

```markdown
## [Role Switch: <Role Name>]

📖 Reading role definition: references/<filename>.md
📋 Current task: <brief description>
```

---

## Reference Resources

| Resource                     | Path                                |
| ---------------------------- | ----------------------------------- |
| Shared technical constraints | `references/shared-standards.md`    |
| Canvas format specification  | `references/canvas-formats.md`      |
| Image layout specification   | `references/image-layout-spec.md`   |
| SVG image embedding          | `references/svg-image-embedding.md` |
| Icon library                 | `templates/icons/README.md`         |

---

## Notes

- Local preview: `python3 -m http.server -d <project_path>/svg_final 8000`
- **Troubleshooting**: on generation issues (layout overflow, export errors, blank images, etc.), check `docs/faq.md` for known solutions
