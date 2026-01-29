# OpenCode Configuration

My personal [OpenCode](https://opencode.ai/docs) configuration.

Before OpenCode existed, I maintained a personal "second brain". A collection of notes on programming patterns, product management frameworks, and psychology principles. I found it valuable to transform these notes into skills that the AI can execute, making my accumulated knowledge actionable during coding sessions.

Feel free to:
- Browse and adapt skills for your own config
- Use agents as templates for your workflows
- Share tips and tricks you've found useful

## Structure

```
.
├── AGENTS.md          # Personal preferences & coding guidelines
├── opencode.json      # Main config (plugins, MCP servers, providers)
├── agents/            # Custom agents for specialized tasks
├── skills/            # executable knowledge notes
├── commands/          # Custom slash commands
└── scripts/           # Helper scripts 
```

## Custom Agents

| Agent | Purpose |
|-------|---------|
| [`code-reviewer`](./agents/code-reviewer.md) | Comprehensive code review |
| [`code-simplifier`](./agents/code-simplifier.md) | Refactor for clarity and maintainability |
| [`deep-thinker`](./agents/deep-thinker.md) | Structured thinking for complex problems |
| [`effort-estimator`](./agents/effort-estimator.md) | Estimate development effort |
| [`git-commit`](./agents/git-commit.md) | Generate conventional commit messages |
| [`requirements-analyzer`](./agents/requirements-analyzer.md) | Analyze feature requirements |
| [`skill-creator`](./agents/skill-creator.md) | Create new skills with proper structure |
| [`talk`](./agents/talk.md) | Conversational interactions |

## Skills Library

Skills are **executable knowledge notes** — my personal "second brain" converted into actionable guidance for the AI agent.

| Category | Skills |
|----------|--------|
| TypeScript | [best practices](./skills/typescript-best-practices/SKILL.md), [advanced types](./skills/typescript-advanced-types/SKILL.md), [`satisfies` operator](./skills/typescript-satisfies-operator/SKILL.md), [interface vs type](./skills/typescript-interface-vs-type/SKILL.md) |
| React | [`useState`](./skills/react-use-state/SKILL.md), [`useCallback`](./skills/react-use-callback/SKILL.md), [`key` prop](./skills/react-key-prop/SKILL.md), [`"use client"` boundaries](./skills/react-use-client-boundary/SKILL.md) |
| CSS | [container queries](./skills/css-container-queries/SKILL.md), [Tailwind v4 best practices](./skills/code-architecture-tailwind-v4-best-practices/SKILL.md) |
| Architecture | [naming conventions](./skills/naming-cheatsheet/SKILL.md), [project structure](./skills/project-structure/SKILL.md), [wrong abstraction patterns](./skills/code-architecture-wrong-abstraction/SKILL.md) |
| Product Frameworks | [Jobs-to-be-Done](./skills/jobs-to-be-done/SKILL.md), [Business Model Canvas](./skills/business-model-canvas/SKILL.md), [Hooked Model](./skills/hooked-model/SKILL.md), [Fogg Behavior Model](./skills/fogg-behavior-model/SKILL.md), [PEST analysis](./skills/pest-analysis/SKILL.md), [product decisions](./skills/making-product-decisions/SKILL.md) |
| UX Psychology | [cognitive load](./skills/cognitive-load/SKILL.md), [cognitive biases](./skills/cognitive-biases/SKILL.md), [cognitive fluency](./skills/cognitive-fluency-psychology/SKILL.md), [Hick's law](./skills/hicks-law/SKILL.md), [progressive disclosure](./skills/progressive-disclosure/SKILL.md), [trust signals](./skills/trust-psychology/SKILL.md), [halo effect](./skills/halo-effect-psychology/SKILL.md) |
| Behavioral Design | [loss aversion](./skills/loss-aversion-psychology/SKILL.md), [status quo bias](./skills/status-quo-bias/SKILL.md), [social proof](./skills/social-proof-psychology/SKILL.md), [curiosity gap](./skills/curiosity-gap/SKILL.md), [self-initiated triggers](./skills/self-initiated-triggers/SKILL.md), [visual cues & CTAs](./skills/visual-cues-cta-psychology/SKILL.md) |
| Decision Making | [hypothesis trees](./skills/hypothesis-tree/SKILL.md), [five whys](./skills/five-whys/SKILL.md), [graph thinking](./skills/graph-thinking/SKILL.md), [game theory (tit-for-tat)](./skills/game-theory-tit-for-tat/SKILL.md) |
| Agile | [Kanban](./skills/kanban/SKILL.md), [theme-epic-story hierarchy](./skills/theme-epic-story/SKILL.md), [user stories](./skills/user-story-fundamentals/SKILL.md) |
| Product Management | [what not to do as PM](./skills/what-not-to-do-as-product-manager/SKILL.md) |

## MCP Servers

- **Context7** — Real-time library/API documentation
- **Figma MCP** — Design file integration (local)

## Plugins

- `@mohak34/opencode-notifier` — Desktop notifications
- `@franlol/opencode-md-table-formatter` — Markdown table formatting

## Resources

- [OpenCode Documentation](https://opencode.ai/docs)
- [Skills Guide](https://opencode.ai/docs/skills)
- [Agents Guide](https://opencode.ai/docs/agents)

## License

[MIT](./LICENSE)
