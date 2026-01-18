---
name: skill-creator
description:
  Create Claude Code custom skills with proper structure and metadata. Use when
  building new skills, setting up skill directories, or packaging skills for
  distribution.
---

# Skill Creator

A guide for creating Claude Code custom skills following Anthropic's official
guidelines.

## When to Use This Skill

- Creating a new Claude Code skill from scratch
- Setting up the correct file structure for a skill
- Writing skill metadata (frontmatter)
- Packaging a skill for distribution
- Converting existing documentation into a skill

## Required Structure

Every skill needs a directory with at minimum one `Skill.md` file:

```
my-skill/
└── Skill.md
```

For complex skills with additional resources:

```
my-skill/
├── Skill.md
├── REFERENCE.md       # Detailed reference docs
└── scripts/
    └── helper.py      # Executable code (optional)
```

## Skill.md Format

The `Skill.md` file must begin with YAML frontmatter:

```markdown
---
name: my-skill-name
description:
  Brief description of what this skill does and when Claude should use it.
  Maximum 200 characters.
---

# Skill Title

Your skill content here...
```

### Required Metadata Fields

| Field         | Max Length | Purpose                                |
| ------------- | ---------- | -------------------------------------- |
| `name`        | 64 chars   | Human-friendly identifier              |
| `description` | 200 chars  | Tells Claude when to invoke this skill |

### Optional Metadata Fields

| Field          | Example                      | Purpose                    |
| -------------- | ---------------------------- | -------------------------- |
| `dependencies` | `python>=3.8, pandas>=1.5.0` | Required software packages |

## Progressive Disclosure

Structure your skill using progressive disclosure:

1. **Frontmatter** - First level: name and description
2. **Markdown body** - Second level: detailed instructions
3. **REFERENCE.md** - Third level: in-depth documentation

Reference additional files in your Skill.md so Claude knows when to access them.

## Executable Code

Skills can include executable scripts for advanced functionality:

**Supported languages:**

- Python (with pandas, numpy, matplotlib)
- JavaScript/Node.js
- File editing packages
- Visualization tools

**Important:** Dependencies must be pre-installed. Additional packages cannot be
installed at runtime.

## Packaging for Distribution

1. Ensure folder name matches skill name
2. Create a ZIP file of the folder
3. ZIP must contain the skill folder as root (not files directly)

**Correct structure:**

```
my-skill.zip
└── my-skill/
    └── Skill.md
```

**Incorrect structure:**

```
my-skill.zip
└── Skill.md  # Wrong! Missing parent folder
```

## Testing Checklist

**Before packaging:**

- [ ] Skill.md has valid YAML frontmatter
- [ ] `name` is ≤64 characters
- [ ] `description` is ≤200 characters and clearly explains when to use
- [ ] All referenced files exist in correct locations
- [ ] No hardcoded secrets or API keys

**After enabling:**

- [ ] Try prompts that should trigger the skill
- [ ] Verify Claude loads the skill (check reasoning)
- [ ] Test edge cases and error scenarios

## Best Practices

**Do:**

- Create focused, single-purpose skills
- Write specific descriptions for accurate invocation
- Include example inputs and outputs
- Test incrementally after each change
- Follow open standards (agentskills.io)

**Avoid:**

- One massive skill that does everything
- Vague descriptions that confuse Claude
- Hardcoding sensitive information
- Skipping the testing phase

---

## Starter Templates

### Template 1: Documentation/Guide Skill

```markdown
---
name: my-guide
description:
  Apply [topic] best practices and guidelines. Use when working on [context] or
  need guidance about [subject].
---

# [Topic] Guide

Brief introduction to what this guide covers.

## When to Use

- [Use case 1]
- [Use case 2]
- [Use case 3]

## Core Principles

### Principle 1: [Name]

Explanation and examples.

### Principle 2: [Name]

Explanation and examples.

## Step-by-Step Process

1. **[Step name]** - Description
2. **[Step name]** - Description
3. **[Step name]** - Description

## Examples

### Good Example

\`\`\` [code or text example] \`\`\`

### Bad Example

\`\`\` [code or text example] \`\`\`

## Common Mistakes

- [Mistake 1] - How to avoid
- [Mistake 2] - How to avoid
```

### Template 2: Code Generation Skill

```markdown
---
name: generate-something
description:
  Generate [type of code] following project conventions. Use when creating new
  [components/modules/files] or need boilerplate for [context].
---

# [Type] Generator

Creates [type of code] with consistent structure and patterns.

## When to Use

- Creating new [component type]
- Need boilerplate for [context]
- Setting up [feature type]

## Generated Structure

\`\`\` [directory structure] \`\`\`

## Required Inputs

| Input | Description   | Example       |
| ----- | ------------- | ------------- |
| Name  | [description] | `MyComponent` |
| Type  | [description] | `form`        |

## Output Format

### [File 1]

\`\`\`[language] // Template with placeholders [code template] \`\`\`

### [File 2]

\`\`\`[language] [code template] \`\`\`

## Conventions

- [Convention 1]
- [Convention 2]
- [Convention 3]

## Customization Options

- **[Option 1]**: [description]
- **[Option 2]**: [description]
```

### Template 3: Analysis/Review Skill

```markdown
---
name: analyze-something
description:
  Analyze [subject] for [purpose]. Use when reviewing [context], investigating
  [issues], or evaluating [criteria].
---

# [Subject] Analyzer

Systematic analysis of [subject] to identify [outcomes].

## When to Use

- Reviewing [context]
- Investigating [type of issues]
- Evaluating [criteria]
- Before [action/decision]

## Analysis Framework

### Phase 1: [Name]

- [ ] Check [item]
- [ ] Verify [item]
- [ ] Review [item]

### Phase 2: [Name]

- [ ] Examine [item]
- [ ] Test [item]
- [ ] Validate [item]

### Phase 3: [Name]

- [ ] Assess [item]
- [ ] Measure [item]
- [ ] Document [item]

## Evaluation Criteria

| Criteria     | Weight | Description   |
| ------------ | ------ | ------------- |
| [Criteria 1] | High   | [Description] |
| [Criteria 2] | Medium | [Description] |
| [Criteria 3] | Low    | [Description] |

## Output Template

\`\`\`markdown

## Analysis Summary

**Subject:** [what was analyzed] **Date:** [date]

### Findings

1. [Finding 1]
2. [Finding 2]

### Recommendations

- [Recommendation 1]
- [Recommendation 2]

### Risk Level

[Low/Medium/High] - [justification] \`\`\`

## Red Flags

- [Red flag 1] - indicates [problem]
- [Red flag 2] - indicates [problem]
```

---

## Resources

- [Official Guide](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills)
- [Skill Templates Repository](https://github.com/anthropics/skills/tree/main/skills)
- [Open Standards](https://agentskills.io)
