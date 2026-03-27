## ADDED Requirements

### Requirement: Translation content format
The translation file SHALL preserve the original content and append the Chinese translation in a side-by-side format.

#### Scenario: Translation file contains original and translated content
- **WHEN** creating a translation file
- **THEN** the file MUST contain the original English content first, followed by the Chinese translation

#### Scenario: Section-by-section translation
- **WHEN** translating a document with multiple sections
- **THEN** each section's Chinese translation MUST appear immediately after its corresponding English section

### Requirement: Side-by-side translation format
The translation file SHALL use a specific format where English appears first, followed by Chinese translation.

#### Scenario: YAML frontmatter
- **WHEN** translating frontmatter (name, description, etc.)
- **THEN** MUST keep English field first, then add Chinese translation on next line
- **AND** identifier fields (name, model) that don't need translation should only appear once in English

```yaml
---
name: code-reviewer
description: Expert code review specialist...
description: 专家代码审查专家...
tools: ["Read", "Grep", "Glob", "Bash"]
tools: ["读取", "搜索", "Glob", "终端"]
model: sonnet
---
```

#### Scenario: Heading translation
- **WHEN** translating headings (##, ###)
- **THEN** MUST keep English heading, then add Chinese heading on next line
- **AND** Chinese heading MUST use the same level (##)

```markdown
## Review Process
## 审查流程
```

#### Scenario: List item translation
- **WHEN** translating list items (- or 1. )
- **THEN** English item MUST come first with its bullet/number
- **AND** Chinese translation MUST be on next line with indentation (2 spaces)
- **AND** Chinese text MUST be bolded using **

```markdown
- **Report** if you are >80% confident it is a real issue
  **报告**如果你有 >80% 的信心确定这是一个真正的问题
```

#### Scenario: Numbered list translation
- **WHEN** translating numbered list items (1., 2., etc.)
- **AND** the list item contains bold text
- **THEN** English item MUST keep its number
- **AND** Chinese translation MUST NOT have a number, but MUST be bolded

```markdown
1. **Gather context** — Run `git diff --staged`...
**收集上下文** — 运行 `git diff --staged`...
```

#### Scenario: Code block comments translation
- **WHEN** translating code block comments
- **THEN** English comment MUST come first
- **AND** Chinese translation MUST follow on next line

```typescript
// BAD: SQL injection via string concatenation
// 差：通过字符串连接导致 SQL 注入
const query = `SELECT * FROM users WHERE id = ${userId}`;
```

#### Scenario: Table translation
- **WHEN** translating tables
- **AND** the table contains both English and Chinese content
- **THEN** each row MUST have English first, Chinese on next line

```markdown
| Severity | Count | Status |
| 严重程度 | 数量 | 状态 |
|----------|-------|--------|
| CRITICAL | 0     | pass   |
| CRITICAL | 0    | 通过 |
```
