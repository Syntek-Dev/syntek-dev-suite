# Markdown All in One Extension Integration Plan

**Last Updated**: 29/12/2025
**Version**: 1.3.1
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Overview

This plan outlines the comprehensive integration of VS Code's "Markdown All in One" extension (yzhang.markdown-all-in-one) with the Syntek Dev Suite. The extension provides keyboard shortcuts, automatic table of contents generation, GitHub Flavoured Markdown support, list editing, math support, HTML export, and more. This integration will enhance the markdown documentation workflow for all agents whilst maintaining compatibility for users without the extension.

---

## Table of Contents

- [Overview](#overview)
- [Table of Contents](#table-of-contents)
- [Requirements](#requirements)
  - [Core Requirements](#core-requirements)
  - [Non-Functional Requirements](#non-functional-requirements)
- [Extension Features Overview](#extension-features-overview)
  - [Keyboard Shortcuts](#keyboard-shortcuts)
  - [Table of Contents Management](#table-of-contents-management)
  - [List Editing](#list-editing)
  - [GitHub Flavoured Markdown](#github-flavoured-markdown)
  - [Math Support](#math-support)
  - [Auto-Completion](#auto-completion)
  - [HTML Export](#html-export)
  - [Syntax Decorations](#syntax-decorations)
- [Technical Design](#technical-design)
  - [Current State](#current-state)
  - [Target State](#target-state)
  - [Integration Strategy](#integration-strategy)
  - [Markdown Standards Alignment](#markdown-standards-alignment)
- [Implementation Phases](#implementation-phases)
  - [Phase 1: VS Code Workspace Configuration](#phase-1-vs-code-workspace-configuration)
  - [Phase 2: Documentation and User Guide](#phase-2-documentation-and-user-guide)
  - [Phase 3: CLAUDE.md Standards Update](#phase-3-claudemd-standards-update)
  - [Phase 5: Agent Skill Updates](#phase-5-agent-skill-updates)
  - [Phase 6: Testing and Validation](#phase-6-testing-and-validation)
  - [Phase 7: GitHub Integration (Optional)](#phase-7-github-integration-optional)
- [Affected Files](#affected-files)
  - [Configuration Files](#configuration-files)
  - [Documentation Files](#documentation-files)
  - [Template Files](#template-files)
  - [Agent Skill Files](#agent-skill-files)
- [Feature-Specific Configuration](#feature-specific-configuration)
  - [Table of Contents Settings](#table-of-contents-settings)
  - [List Formatting Settings](#list-formatting-settings)
  - [GitHub Flavoured Markdown Settings](#github-flavoured-markdown-settings)
  - [Export Settings](#export-settings)
  - [Display and Syntax Settings](#display-and-syntax-settings)
- [Agent Integration Guidelines](#agent-integration-guidelines)
  - [Markdown Generation Best Practices](#markdown-generation-best-practices)
  - [Table Formatting](#table-formatting)
  - [Task Lists](#task-lists)
  - [Code Blocks](#code-blocks)
  - [Math Expressions](#math-expressions)
- [User Workflow Enhancements](#user-workflow-enhancements)
  - [With Extension Installed](#with-extension-installed)
  - [Without Extension](#without-extension)
- [Risks and Mitigations](#risks-and-mitigations)
- [Open Questions](#open-questions)
- [Success Criteria](#success-criteria)
  - [Extension Integration](#extension-integration)
  - [Documentation](#documentation)
  - [Agent Compatibility](#agent-compatibility)
  - [User Experience](#user-experience)
  - [Graceful Degradation](#graceful-degradation)
  - [Testing](#testing)
- [References](#references)

---

## Requirements

### Core Requirements

1. **Graceful Degradation**: All markdown features MUST work without the extension (standard Markdown)
2. **Extension Enhancement**: Users with extension get enhanced editing experience (auto-update, shortcuts, formatting)
3. **Agent Compatibility**: Agent-generated markdown MUST be compatible with all extension features
4. **Standard Compliance**: All markdown MUST be valid CommonMark/GitHub Flavoured Markdown
5. **User Choice**: Optional extension installation with clear benefits
6. **Configuration Management**: Optimal extension settings pre-configured in workspace

### Non-Functional Requirements

1. **No Breaking Changes**: Existing workflows continue to work
2. **Zero User Configuration**: Works out-of-the-box after extension installation
3. **Clear Documentation**: Users understand what the extension provides
4. **Backwards Compatible**: Existing markdown files remain valid
5. **Cross-Platform**: Works on Windows, macOS, Linux

---

## Extension Features Overview

### Keyboard Shortcuts

| Shortcut               | Function             | Agent Impact                            |
| ---------------------- | -------------------- | --------------------------------------- |
| `Ctrl/Cmd + B`         | Toggle bold          | Agents use `**bold**` syntax            |
| `Ctrl/Cmd + I`         | Toggle italic        | Agents use `*italic*` syntax            |
| `Alt + S`              | Toggle strikethrough | Agents use `~~strikethrough~~` (GFM)    |
| `Ctrl + Shift + ]`     | Uplevel heading      | No impact (manual editing)              |
| `Ctrl + Shift + [`     | Downlevel heading    | No impact (manual editing)              |
| `Ctrl/Cmd + M`         | Toggle math          | Agents can include `$math$` inline      |
| `Alt + C`              | Check/uncheck task   | Agents generate `- [ ]` / `- [x]` lists |
| `Ctrl/Cmd + Shift + V` | Preview              | No impact (user feature)                |

**Agent Consideration:** Generate markdown using syntax that shortcuts work with (e.g., `**bold**` not `__bold__`).

### Table of Contents Management

- **Auto-generation**: Command Palette → "Create Table of Contents"
- **Auto-update on save**: TOC stays in sync with headings (configurable)
- **Heading exclusion**: `<!-- omit in toc -->` marker
- **Section numbering**: Add/remove numbered headings
- **Multi-TOC support**: Multiple TOCs in single document
- **Slugify modes**: GitHub, GitLab, Gitea, VS Code compatible

**Agent Consideration:** Always include "## Table of Contents" section in documentation.

### List Editing

- **Smart indentation**: Tab/Shift+Tab for nested lists
- **Auto-renumbering**: Ordered lists auto-correct sequence (1, 2, 3...)
- **Marker toggling**: Cycle through `- * +` markers
- **CommonMark compliant**: Follows standard list formatting rules

**Agent Consideration:** Use consistent list markers (`-` for unordered, `1.` for ordered).

### GitHub Flavoured Markdown

- **Tables**: Auto-formatting and alignment
- **Task lists**: `- [ ]` and `- [x]` checkbox support
- **Strikethrough**: `~~text~~` syntax
- **Auto-linking**: URLs become clickable links

**Agent Consideration:** Use GFM table syntax, task lists for checklists.

### Math Support

- **Inline math**: `$expression$`
- **Block math**: `$$expression$$`
- **KaTeX integration**: LaTeX-style math rendering
- **Macro support**: Custom math macros

**Agent Consideration:** Use math syntax for technical documentation where appropriate.

### Auto-Completion

- **File paths**: Auto-complete file references
- **Image paths**: Suggest images in project
- **Math functions**: KaTeX function suggestions
- **Reference links**: Auto-complete `[text][ref]` style links

**Agent Consideration:** Use relative file paths for cross-references.

### HTML Export

- **Print to HTML**: Export markdown as HTML file
- **Batch export**: Export multiple files
- **Theme support**: Light/dark themes
- **Image handling**: Absolute paths or Base64 embedding
- **Auto-export on save**: Optional continuous export

**Agent Consideration:** No impact on generation, user feature for publishing docs.

### Syntax Decorations

- **Strikethrough styling**: Visual strikethrough in editor
- **Code span highlighting**: Inline code highlighting
- **Plain theme option**: Minimal distraction mode
- **Large file optimisation**: Disable decorations for big files

**Agent Consideration:** No impact, visual editor enhancement.

---

## Technical Design

### Current State

**Existing Behaviour:**
- CLAUDE.md requires Table of Contents in all markdown docs
- Agents generate standard Markdown (CommonMark compatible)
- Manual TOC generation by agents
- No VS Code workspace configuration
- British English spelling in documentation
- GFM table syntax already in use

**Current Markdown Conventions:**
- Unordered lists use `-` marker
- Bold uses `**text**` syntax
- Tables use GFM pipe syntax
- Task lists use `- [ ]` / `- [x]` syntax
- Headings follow hierarchy (H1 title, H2 sections)

### Target State

**Desired Behaviour:**
1. VS Code workspace recommends "Markdown All in One" extension
2. Extension configured with optimal settings for Syntek standards
3. Users with extension get enhanced editing experience
4. Agents generate markdown compatible with all extension features
5. Documentation explains extension benefits and usage
6. Graceful degradation for users without extension

### Integration Strategy

**Principle: Enhancement, Not Dependency**

The extension enhances the user experience but is not required. All generated markdown must work without the extension.

**Three-Tier Approach:**

**Tier 1: With Extension (Full Experience)**
- Auto-updating TOCs
- Keyboard shortcuts for formatting
- Smart list editing
- Table auto-formatting
- Math rendering
- File path completion
- HTML export capability

**Tier 2: Without Extension (Base Experience)**
- Manual TOC (functional but static)
- Standard Markdown formatting
- Manual list management
- Standard GFM tables
- Math syntax visible but not rendered
- Manual file path entry

**Tier 3: Agent Generation (Compatibility)**
- Generate markdown compatible with extension features
- Use extension-friendly syntax conventions
- Include TOC placeholders for auto-generation
- Follow GFM standards for tables and task lists

### Markdown Standards Alignment

**Extension Configuration Aligned with Syntek Standards:**

| Syntek Standard          | Extension Setting          | Value                |
| ------------------------ | -------------------------- | -------------------- |
| British English spelling | No setting needed          | Handled in CLAUDE.md |
| Unordered list marker    | `toc.unorderedList.marker` | `-`                  |
| GitHub compatibility     | `toc.slugifyMode`          | `github`             |
| Task list support        | Built-in                   | Enabled              |
| Table formatting         | `tableFormatter.enabled`   | `true`               |
| Bold syntax              | `bold.indicator`           | `**`                 |
| Italic syntax            | `italic.indicator`         | `*`                  |
| Auto-update TOC          | `toc.updateOnSave`         | `true`               |
| Heading levels in TOC    | `toc.levels`               | `2..6`               |

---

## Implementation Phases

### Phase 1: VS Code Workspace Configuration

**Objective:** Create VS Code workspace configuration that recommends and configures the extension

**Tasks:**
- [ ] Create `.vscode/extensions.json` to recommend extension
- [ ] Create `.vscode/settings.json` with optimal extension settings
- [ ] Ensure `.vscode/` folder is tracked in Git
- [ ] Test configuration on fresh VS Code installation

**Deliverable:** VS Code workspace that prompts users to install extension with optimal settings

**Files Created:**
- `/.vscode/extensions.json`
- `/.vscode/settings.json`

**`.vscode/extensions.json` Content:**
```json
{
  "recommendations": [
    "yzhang.markdown-all-in-one"
  ],
  "unwantedRecommendations": []
}
```

**`.vscode/settings.json` Content:**
```json
{
  "markdown.extension.toc.levels": "2..6",
  "markdown.extension.toc.unorderedList.marker": "-",
  "markdown.extension.toc.updateOnSave": true,
  "markdown.extension.toc.slugifyMode": "github",
  "markdown.extension.list.indentationSize": "adaptive",
  "markdown.extension.orderedList.marker": "one",
  "markdown.extension.orderedList.autoRenumber": true,
  "markdown.extension.italic.indicator": "*",
  "markdown.extension.bold.indicator": "**",
  "markdown.extension.tableFormatter.enabled": true,
  "markdown.extension.syntax.decorations": true,
  "markdown.extension.syntax.plainTheme": false,
  "markdown.extension.completion.respectVscodeSearchExclude": true,
  "markdown.extension.print.absoluteImgPath": true,
  "markdown.extension.print.imgToBase64": false,
  "markdown.extension.print.includeVscodeStylesheets": true,
  "[markdown]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "yzhang.markdown-all-in-one"
  }
}
```

**Configuration Rationale:**

| Setting                    | Value        | Reason                                        |
| -------------------------- | ------------ | --------------------------------------------- |
| `toc.levels`               | `"2..6"`     | Skip H1 (document title) to avoid redundancy  |
| `toc.marker`               | `"-"`        | Consistent with Syntek's unordered list style |
| `toc.updateOnSave`         | `true`       | Keep TOC in sync automatically                |
| `toc.slugifyMode`          | `"github"`   | GitHub-compatible anchor links                |
| `list.indentationSize`     | `"adaptive"` | Follow document's own style                   |
| `orderedList.marker`       | `"one"`      | Use `1.` for all items (GitHub style)         |
| `orderedList.autoRenumber` | `true`       | Fix list numbering automatically              |
| `italic.indicator`         | `"*"`        | Single asterisk for italic                    |
| `bold.indicator`           | `"**"`       | Double asterisk for bold                      |
| `tableFormatter.enabled`   | `true`       | Auto-format GFM tables                        |
| `syntax.decorations`       | `true`       | Visual enhancements in editor                 |
| `formatOnSave`             | `true`       | Auto-format markdown on save                  |

---

### Phase 2: Documentation and User Guide

**Objective:** Create comprehensive user documentation explaining the extension and its benefits

**Tasks:**
- [ ] Create `docs/GUIDES/MARKDOWN-ALL-IN-ONE.md` guide
- [ ] Document all extension features and keyboard shortcuts
- [ ] Include before/after examples
- [ ] Add troubleshooting section
- [ ] Create quick reference table for shortcuts

**Deliverable:** User guide explaining extension features and how to use them

**Files Created:**
- `/docs/GUIDES/MARKDOWN-ALL-IN-ONE.md`

**Guide Sections:**
1. **Introduction**: What the extension does and why it's recommended
2. **Installation**: How to install the extension
3. **Feature Overview**: All features with examples
4. **Keyboard Shortcuts Reference**: Quick reference table
5. **Table of Contents**: How TOC auto-generation works
6. **List Editing**: Smart list features
7. **Tables**: GFM table formatting
8. **Task Lists**: Checkbox creation and toggling
9. **Math Support**: Inline and block math syntax
10. **HTML Export**: Exporting documentation
11. **Configuration**: Understanding workspace settings
12. **Troubleshooting**: Common issues and solutions
13. **Without Extension**: How features degrade gracefully

---

### Phase 3: CLAUDE.md Standards Update

**Objective:** Update CLAUDE.md to reference Markdown All in One extension and its features

**Tasks:**
- [ ] Add "Markdown Editing Extension" section under Documentation Standards
- [ ] Reference extension features that align with standards
- [ ] Update TOC requirements to mention auto-generation
- [ ] Add GFM table formatting guidance
- [ ] Include task list syntax
- [ ] Reference keyboard shortcuts guide

**Deliverable:** Updated CLAUDE.md with extension integration documentation

**File Changes:**
- `/CLAUDE.md` (Documentation Standards section)

**New Section to Add:**
```markdown
### Markdown Editing Extension

The Syntek Dev Suite is configured for the **Markdown All in One** VS Code extension, providing enhanced markdown editing capabilities.

**Installation:**
When opening a Syntek Dev Suite project in VS Code, you will be prompted to install recommended extensions. Install "Markdown All in One" for the best experience.

**Key Features:**
- **Auto-updating TOCs**: Table of Contents stays in sync with headings
- **Keyboard Shortcuts**: Format text quickly (Bold: Ctrl/Cmd+B, Italic: Ctrl/Cmd+I)
- **Smart Lists**: Auto-renumbering, indentation, and marker toggling
- **Table Formatting**: Auto-align GFM tables
- **Task Lists**: Toggle checkboxes with Alt+C
- **Math Support**: Render LaTeX-style math expressions
- **HTML Export**: Export documentation to HTML

**Graceful Degradation:**
All markdown generated by agents works without the extension. The extension enhances the editing experience but is not required.

**Documentation:**
See [docs/GUIDES/MARKDOWN-ALL-IN-ONE.md](/docs/GUIDES/MARKDOWN-ALL-IN-ONE.md) for complete feature guide.

**Extension ID:** `yzhang.markdown-all-in-one`
```

**Update Existing Sections:**

1. **Table of Contents Section:**
```markdown
**ALL markdown documentation files MUST include a Table of Contents at the top of the document**, immediately after the main heading.

With the Markdown All in One extension installed:
- TOC is automatically updated when you save the file
- Use `<!-- omit in toc -->` to exclude headings from TOC
- Extension command: "Create Table of Contents" to generate initial TOC

Without the extension:
- Agents generate manual TOC at document creation
- Manual updates needed if structure changes
```

2. **Add GitHub Flavoured Markdown Section:**
```markdown
### GitHub Flavoured Markdown

All markdown files should use GitHub Flavoured Markdown (GFM) syntax:

**Tables:**
```markdown
| Column 1 | Column 2 |
| -------- | -------- |
| Value 1  | Value 2  |
```

**Task Lists:**
```markdown
- [ ] Incomplete task
- [x] Completed task
```

**Strikethrough:**
```markdown
~~This text is crossed out~~
```

The Markdown All in One extension auto-formats tables and provides keyboard shortcuts for these features.
```

---

### Phase 4: Template Updates

**Objective:** Update all markdown templates to be fully compatible with extension features

**Tasks:**
- [ ] Update `examples/setup/SECTION-README-TEMPLATE.md`
- [ ] Update `examples/setup/README-TEMPLATE.md`
- [ ] Update any other markdown templates
- [ ] Add TOC section to all templates
- [ ] Ensure GFM table syntax is used
- [ ] Add task list examples where appropriate

**Deliverable:** Templates that generate extension-compatible markdown

**File Changes:**
- `/examples/setup/SECTION-README-TEMPLATE.md`
- `/examples/setup/README-TEMPLATE.md`
- `/examples/setup/SYNTEK-GUIDE-TEMPLATE.md` (if exists)

**Template Format Updates:**

1. **Consistent TOC Placement:**
```markdown
# Document Title

Brief introduction.

---

## Table of Contents

<!-- Auto-generated with Markdown All in One extension -->

- [Document Title](#document-title)
  - [Table of Contents](#table-of-contents)
  - [Section 1](#section-1)
  - [Section 2](#section-2)

---

## Section 1
...
```

2. **Table Format:**
```markdown
| Column 1 | Column 2 | Column 3 |
| -------- | -------- | -------- |
| Value    | Value    | Value    |
```

3. **Task List Format:**
```markdown
## Checklist

- [ ] Task 1
- [ ] Task 2
- [x] Completed task
```

---

### Phase 5: Agent Skill Updates

**Objective:** Update agent skills to generate markdown compatible with all extension features

**Tasks:**
- [ ] Update `skills/global-workflow/SKILL.md` with markdown generation guidelines
- [ ] Review all stack-specific skills for markdown examples
- [ ] Ensure agents use consistent markdown syntax
- [ ] Add guidance for GFM features (tables, task lists)
- [ ] Document TOC generation format

**Deliverable:** Agent skills that generate fully compatible markdown

**File Changes:**
- `/skills/global-workflow/SKILL.md`
- Stack-specific skills (review, no major changes expected)

**New Section in global-workflow/SKILL.md:**
```markdown
## Markdown Documentation Standards

When generating markdown documentation, follow these standards for compatibility with the Markdown All in One extension:

### Table of Contents

1. **Placement**: After document title and introduction, before main content
2. **Format**: Unordered list with `-` marker
3. **Heading**: Use `## Table of Contents`
4. **Indentation**: 2 spaces per nesting level
5. **Links**: Use anchor link format `[Text](#heading-slug)`
6. **Slug Format**: Lowercase, hyphens for spaces, remove special chars

**Example:**
\```markdown
## Table of Contents

- [Main Title](#main-title)
  - [Table of Contents](#table-of-contents)
  - [Section 1](#section-1)
    - [Subsection 1.1](#subsection-11)
\```

### Text Formatting

| Format            | Syntax     | Example          |
| ----------------- | ---------- | ---------------- |
| **Bold**          | `**text**` | `**important**`  |
| *Italic*          | `*text*`   | `*emphasise*`    |
| ~~Strikethrough~~ | `~~text~~` | `~~deprecated~~` |
| `Inline code`     | \`code\`   | \`function()\`   |

### Lists

**Unordered Lists:**
- Use `-` marker consistently
- 2 spaces for indentation

\```markdown
- Item 1
  - Nested item 1.1
  - Nested item 1.2
- Item 2
\```

**Ordered Lists:**
- Use `1.` marker for all items (auto-renumbering)
- 2 spaces for indentation

\```markdown
1. First step
   1. Sub-step 1.1
   1. Sub-step 1.2
1. Second step
\```

**Task Lists:**
- Use for checklists and action items
- Format: `- [ ]` for incomplete, `- [x]` for complete

\```markdown
- [ ] Task to do
- [x] Completed task
\```

### Tables

Use GitHub Flavoured Markdown table syntax:

\```markdown
| Column 1 | Column 2 | Column 3 |
| -------- | -------- | -------- |
| Value 1  | Value 2  | Value 3  |
| Value 4  | Value 5  | Value 6  |
\```

**Alignment:**
- Left: `|:---------|`
- Centre: `|:--------:|`
- Right: `|---------:|`

### Code Blocks

**Fenced Code Blocks:**
\```language
code here
\```

**Specify Language:**
Always specify language for syntax highlighting:
- `python`, `typescript`, `bash`, `json`, `sql`, `php`, `dart`, etc.

### Links and References

**Inline Links:**
\```markdown
[Link text](https://example.com)
\```

**Reference Links:**
\```markdown
[Link text][ref]

[ref]: https://example.com
\```

**Internal Links:**
\```markdown
See [Section 1](#section-1) for details.
\```

### Math Expressions

For technical documentation, use LaTeX-style math:

**Inline Math:**
\```markdown
The equation $E = mc^2$ shows the relationship.
\```

**Block Math:**
\```markdown
$$
f(x) = \int_{-\infty}^{\infty} e^{-x^2} dx
$$
\```

### Horizontal Rules

Use `---` for horizontal rules:
\```markdown
---
\```

### Headings

**Hierarchy:**
- `# Title` - Document title (once per file)
- `## Section` - Main sections
- `### Subsection` - Subsections
- `####` and beyond - As needed (avoid excessive nesting)

**Excluding from TOC:**
Add `<!-- omit in toc -->` to exclude heading:
\```markdown
## Internal Notes <!-- omit in toc -->
\```

### File References

Use relative paths for internal file references:
\```markdown
See [Setup Guide](./docs/SETUP/GUIDE.md) for details.
\```

### Best Practices

1. **Consistency**: Use same syntax throughout (e.g., `**bold**` not `__bold__`)
2. **British English**: Follow CLAUDE.md localisation settings
3. **Blank Lines**: Add blank lines before/after headings, code blocks, tables
4. **Line Length**: Keep lines under 120 characters where practical (no hard limit)
5. **TOC Maintenance**: Include TOC section; extension handles updates for users
```

---

### Phase 6: Testing and Validation

**Objective:** Validate that all extension features work correctly with agent-generated markdown

**Tasks:**
- [ ] Test TOC auto-generation and auto-update
- [ ] Test keyboard shortcuts with agent-generated text
- [ ] Test table auto-formatting
- [ ] Test task list toggling
- [ ] Test list editing (Tab/Shift+Tab)
- [ ] Test math rendering
- [ ] Test HTML export
- [ ] Test file path completion
- [ ] Validate without extension (graceful degradation)
- [ ] Test on Windows, macOS, Linux

**Deliverable:** Validated extension integration across all features and platforms

**Test Cases:**

| Feature          | Test Case                              | Expected Result                  |
| ---------------- | -------------------------------------- | -------------------------------- |
| **TOC**          | Create doc with agent, save in VS Code | TOC auto-updates with headings   |
| **TOC**          | Add `<!-- omit in toc -->` to heading  | Heading excluded from TOC        |
| **TOC**          | Change heading text                    | TOC link text updates on save    |
| **Keyboard**     | Select text, press Ctrl+B              | Text becomes bold with `**`      |
| **Keyboard**     | Select text, press Ctrl+I              | Text becomes italic with `*`     |
| **Keyboard**     | Select text, press Alt+S               | Text gets strikethrough `~~`     |
| **Keyboard**     | Cursor on task list, press Alt+C       | Checkbox toggles `[ ]` ↔ `[x]`   |
| **Lists**        | Press Tab in list item                 | Item indents correctly           |
| **Lists**        | Press Shift+Tab in nested item         | Item un-indents                  |
| **Lists**        | Renumber ordered list items            | Auto-renumbering fixes sequence  |
| **Tables**       | Create unaligned table, save           | Table auto-aligns columns        |
| **Tables**       | Add/remove table column                | Formatting adjusts automatically |
| **Math**         | Add `$x^2$` inline                     | Math renders in preview          |
| **Math**         | Add `$$` block                         | Math block renders in preview    |
| **Export**       | Run "Print to HTML"                    | HTML file created with styling   |
| **Completion**   | Type `![](`                            | File path suggestions appear     |
| **No Extension** | Open doc without extension             | Manual TOC links work            |
| **No Extension** | View table without extension           | Table displays correctly         |

---

### Phase 7: GitHub Integration (Optional)

**Objective:** Ensure exported/committed markdown renders correctly on GitHub

**Tasks:**
- [ ] Verify GitHub renders TOC links correctly
- [ ] Test GFM tables on GitHub
- [ ] Test task lists on GitHub
- [ ] Verify math expressions (may need GitHub Actions for rendering)
- [ ] Test cross-file links in GitHub
- [ ] Validate anchor links work in GitHub

**Deliverable:** Markdown that renders beautifully both in VS Code and on GitHub

**Note:** GitHub natively supports GFM but may not render KaTeX math. Consider adding MathJax/KaTeX rendering via GitHub Pages if math-heavy docs are needed.

---

## Affected Files

### Configuration Files

| File                       | Change Type | Description                             |
| -------------------------- | ----------- | --------------------------------------- |
| `/.vscode/extensions.json` | **Create**  | Recommend Markdown All in One extension |
| `/.vscode/settings.json`   | **Create**  | Configure extension settings            |

### Documentation Files

| File                                  | Change Type       | Description                                 |
| ------------------------------------- | ----------------- | ------------------------------------------- |
| `/CLAUDE.md`                          | **Update**        | Add Markdown All in One integration section |
| `/README.md`                          | **Update**        | Mention extension in documentation section  |
| `/docs/GUIDES/MARKDOWN-ALL-IN-ONE.md` | **Create**        | Comprehensive extension feature guide       |
| `/docs/GUIDES/README.md`              | **Create/Update** | Index of all guides                         |

### Template Files

| File                                         | Change Type | Description              |
| -------------------------------------------- | ----------- | ------------------------ |
| `/examples/setup/SECTION-README-TEMPLATE.md` | **Update**  | Add TOC, use GFM syntax  |
| `/examples/setup/README-TEMPLATE.md`         | **Update**  | Add TOC, use GFM syntax  |
| `/examples/setup/SYNTEK-GUIDE-TEMPLATE.md`   | **Review**  | Ensure GFM compatibility |

### Agent Skill Files

| File                               | Change Type | Description                        |
| ---------------------------------- | ----------- | ---------------------------------- |
| `/skills/global-workflow/SKILL.md` | **Update**  | Add markdown generation guidelines |
| Stack-specific skills              | **Review**  | Ensure markdown examples use GFM   |

---

## Feature-Specific Configuration

### Table of Contents Settings

```json
{
  "markdown.extension.toc.levels": "2..6",
  "markdown.extension.toc.unorderedList.marker": "-",
  "markdown.extension.toc.updateOnSave": true,
  "markdown.extension.toc.slugifyMode": "github",
  "markdown.extension.toc.plaintext": false,
  "markdown.extension.toc.orderedList": false
}
```

**Rationale:**
- Include H2-H6 (skip H1 document title)
- Use `-` marker for consistency
- Auto-update on save
- GitHub-compatible slugs
- Include links (not plain text)
- Use unordered lists for TOC

### List Formatting Settings

```json
{
  "markdown.extension.list.indentationSize": "adaptive",
  "markdown.extension.orderedList.marker": "one",
  "markdown.extension.orderedList.autoRenumber": true
}
```

**Rationale:**
- Adaptive indentation follows document style
- Use `1.` for all ordered items (GitHub style)
- Auto-fix list numbering

### GitHub Flavoured Markdown Settings

```json
{
  "markdown.extension.tableFormatter.enabled": true,
  "markdown.extension.italic.indicator": "*",
  "markdown.extension.bold.indicator": "**"
}
```

**Rationale:**
- Auto-format tables
- Single asterisk italic (GFM standard)
- Double asterisk bold (GFM standard)

### Export Settings

```json
{
  "markdown.extension.print.absoluteImgPath": true,
  "markdown.extension.print.imgToBase64": false,
  "markdown.extension.print.includeVscodeStylesheets": true
}
```

**Rationale:**
- Absolute image paths for portability
- Don't embed images (keep HTML files smaller)
- Include VS Code styling for consistency

### Display and Syntax Settings

```json
{
  "markdown.extension.syntax.decorations": true,
  "markdown.extension.syntax.plainTheme": false,
  "markdown.extension.completion.respectVscodeSearchExclude": true
}
```

**Rationale:**
- Enable visual syntax decorations
- Use themed decorations (not plain)
- Respect VS Code search exclude patterns

---

## Agent Integration Guidelines

### Markdown Generation Best Practices

When agents generate markdown files, follow these guidelines:

1. **TOC Inclusion**: Always include a "## Table of Contents" section
2. **Consistent Syntax**: Use `**bold**`, `*italic*`, `~~strikethrough~~`
3. **List Markers**: Use `-` for unordered, `1.` for ordered
4. **Table Format**: Use GFM pipe tables with header separators
5. **Task Lists**: Use `- [ ]` and `- [x]` for checkboxes
6. **Headings**: H1 for title, H2 for sections, H3+ for subsections
7. **Blank Lines**: Add blank lines before/after headings, code blocks, tables
8. **Code Blocks**: Always specify language for syntax highlighting

### Table Formatting

**Agent-Generated Table:**
```markdown
| Column 1     | Column 2 | Column 3      |
| ------------ | -------- | ------------- |
| Value 1      | Value 2  | Value 3       |
| Longer value | Short    | Medium length |
```

**Extension Behaviour:** Auto-aligns columns on save

### Task Lists

**Agent-Generated Task List:**
```markdown
## Implementation Checklist

- [ ] Phase 1: Setup
  - [ ] Create configuration
  - [ ] Install dependencies
- [ ] Phase 2: Development
  - [x] Implement feature A
  - [ ] Implement feature B
- [ ] Phase 3: Testing
```

**Extension Behaviour:** Alt+C toggles checkboxes

### Code Blocks

**Agent-Generated Code Block:**
````markdown
```typescript
interface User {
  id: string;
  name: string;
  email: string;
}
```
````

**Extension Behaviour:** Syntax highlighting in preview

### Math Expressions

**Agent-Generated Math:**
```markdown
The quadratic formula is $x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$.

For block equations:

$$
\int_{a}^{b} f(x) dx = F(b) - F(a)
$$
```

**Extension Behaviour:** Renders in preview (requires math support)

---

## User Workflow Enhancements

### With Extension Installed

**Documentation Creation:**
1. Agent generates markdown file with TOC section
2. User opens file in VS Code
3. TOC auto-updates as user edits headings
4. Tables auto-format on save
5. Keyboard shortcuts speed up formatting
6. Task lists toggle with Alt+C
7. Preview shows rendered content
8. Export to HTML available

**Editing Experience:**
- Press Ctrl+B to bold selected text
- Press Tab/Shift+Tab to indent/unindent lists
- Extension auto-renumbers ordered lists
- Tables align automatically
- TOC stays in sync without manual updates

### Without Extension

**Documentation Creation:**
1. Agent generates markdown file with manual TOC
2. User opens file in any editor
3. Manual TOC is functional but static
4. Tables display correctly but don't auto-format
5. Standard markdown features work
6. Manual formatting needed

**Editing Experience:**
- Manual formatting using markdown syntax
- TOC must be updated manually if structure changes
- Tables must be aligned manually
- All standard markdown features available

---

## Risks and Mitigations

| Risk                                      | Likelihood | Impact | Mitigation                                                   |
| ----------------------------------------- | ---------- | ------ | ------------------------------------------------------------ |
| Extension changes behaviour in updates    | Low        | Medium | Use stable settings, test with extension updates             |
| Users don't install extension             | High       | Low    | Graceful degradation: manual TOC and standard markdown work  |
| Extension conflicts with other extensions | Low        | Medium | Test with common markdown extensions, document compatibility |
| TOC format incompatibility                | Low        | Low    | Use standard markdown lists, extension detects automatically |
| Users confused by auto-updating TOC       | Low        | Low    | Document behaviour clearly in user guide                     |
| Math rendering not available              | Medium     | Low    | Document that math requires extension or GitHub Pages setup  |
| Export feature not used                   | High       | None   | Optional feature, no impact if unused                        |
| Settings override user preferences        | Low        | Medium | Use workspace settings (user can override in user settings)  |
| Extension not available on all platforms  | Very Low   | High   | Extension supports Windows, macOS, Linux                     |
| GitHub doesn't render advanced features   | Medium     | Low    | Ensure GFM compatibility, document limitations               |

**Critical Mitigations:**

1. **Graceful Degradation**: All features MUST work without extension
2. **Clear Documentation**: User guide explains what extension provides
3. **Standard Compliance**: Use only GFM and CommonMark syntax
4. **Testing**: Validate with and without extension

---

## Open Questions

- [ ] **Q: Should we enable auto-export to HTML on save?**
  - **Consideration:** Could be useful for generating static docs, but may slow down saves
  - **Recommendation:** Leave disabled by default, document how to enable

- [ ] **Q: Should math support be enabled by default?**
  - **Consideration:** Math syntax `$...$` might conflict if not used
  - **Recommendation:** Enable it, minimal conflict risk

- [ ] **Q: Should we configure custom KaTeX macros?**
  - **Consideration:** Useful for frequently used math expressions
  - **Recommendation:** Skip for now, add if technical docs become math-heavy

- [ ] **Q: Should we add Markdown All in One to syntek-dev-suite:init?**
  - **Consideration:** Setup agent could create `.vscode/` folder automatically
  - **Recommendation:** Yes, include in Phase 1 of setup workflow

- [ ] **Q: Should we create custom VS Code snippets for common markdown patterns?**
  - **Consideration:** Could speed up documentation writing
  - **Recommendation:** Future enhancement, not in initial integration

- [ ] **Q: Should we document GitHub Pages setup for math rendering?**
  - **Consideration:** Useful if docs need to be published online
  - **Recommendation:** Add to optional Phase 7

- [ ] **Q: Should we configure the extension's plain theme option?**
  - **Consideration:** Minimal distraction mode
  - **Recommendation:** Leave as user choice, document in guide

- [ ] **Q: Should we enable section numbering by default?**
  - **Consideration:** Extension can add `1.`, `1.1`, `1.2` prefixes to headings
  - **Recommendation:** No, keep clean headings, document as optional feature

---

## Success Criteria

Implementation is successful when:

### Extension Integration

- [x] VS Code workspace recommends "Markdown All in One" extension on open
- [x] Extension configuration is optimal for Syntek Dev Suite standards
- [x] All settings align with British English and GFM conventions
- [x] Configuration is tracked in Git and shared across team

### Documentation

- [x] Comprehensive user guide explains all extension features
- [x] CLAUDE.md references extension integration
- [x] README mentions extension recommendation
- [x] Keyboard shortcuts are documented in quick reference
- [x] Troubleshooting section covers common issues

### Agent Compatibility

- [x] All agents generate markdown compatible with extension features
- [x] TOC sections are included in all documentation
- [x] GFM tables use correct syntax
- [x] Task lists use correct checkbox syntax
- [x] Code blocks specify language for highlighting
- [x] Templates reflect extension-compatible format

### User Experience

- [x] Users with extension get auto-updating TOCs
- [x] Keyboard shortcuts work with agent-generated text
- [x] Tables auto-format on save
- [x] Task list checkboxes toggle with Alt+C
- [x] Lists indent/unindent with Tab/Shift+Tab
- [x] Math expressions render in preview
- [x] HTML export produces styled output

### Graceful Degradation

- [x] Users without extension see functional manual TOCs
- [x] All markdown features work in standard editors
- [x] GitHub renders all content correctly
- [x] No broken links or formatting issues
- [x] Agent-generated markdown is valid CommonMark/GFM

### Testing

- [ ] All extension features tested with agent-generated content
- [ ] Cross-platform testing (Windows, macOS, Linux)
- [ ] Both with and without extension scenarios validated
- [ ] GitHub rendering verified
- [ ] No regression in existing workflows

---

## References

**Extension Resources:**
- [Markdown All in One - VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)
- [GitHub Repository](https://github.com/yzhang-gh/vscode-markdown)
- [Official Documentation](https://markdown-all-in-one.github.io/docs/guide/table-of-contents.html)

**Markdown Standards:**
- [CommonMark Specification](https://commonmark.org/)
- [GitHub Flavoured Markdown](https://github.github.com/gfm/)
- [Markdown Guide](https://www.markdownguide.org/)

**VS Code Configuration:**
- [Workspace Recommended Extensions](https://code.visualstudio.com/docs/editor/extension-marketplace#_workspace-recommended-extensions)
- [Workspace Settings](https://code.visualstudio.com/docs/getstarted/settings)
- [Settings Precedence](https://code.visualstudio.com/docs/getstarted/settings#_settings-precedence)

**Syntek Dev Suite Documentation:**
- `/CLAUDE.md` - Documentation Standards
- `/skills/global-workflow/SKILL.md` - Global Workflow Standards
- `/examples/setup/SECTION-README-TEMPLATE.md` - Template Examples
- `/plugins/README.md` - Plugin Documentation
