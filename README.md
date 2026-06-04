# Blockstudio Agent Skill

[![skills.sh](https://skills.sh/b/jdailey12/blockstudio-agent-skill)](https://skills.sh/jdailey12/blockstudio-agent-skill)

Agent skill for building with [Blockstudio](https://www.blockstudio.dev/) in WordPress, including editable site sections, file-based pages, static/Astro site rebuilds, and Gutenberg editor UX verification.

The skill follows the same docs pattern as lightweight framework skills: it provides compact operating guidance, small examples, and links to official Blockstudio docs instead of bundling the full documentation corpus.

## Install

Install with the open skills CLI:

```bash
npx skills add jdailey12/blockstudio-agent-skill --skill build-blockstudio-wordpress -a codex
```

For all supported agents, omit `-a codex` and choose interactively:

```bash
npx skills add jdailey12/blockstudio-agent-skill --skill build-blockstudio-wordpress
```

## Skill

### `build-blockstudio-wordpress`

Use for Blockstudio WordPress work such as custom blocks, `block.json`, file-based pages, patterns, Tailwind, ACF migration, RichText, InnerBlocks, Interactivity API, RPC, database, cron, registries, static/Astro site cloning, content modeling, CPT-backed sections, editor CSS, media fields, and backend editing verification.

The skill tells agents to consult:

- Official docs: https://www.blockstudio.dev/docs
- AI context docs: https://www.blockstudio.dev/docs/dev/ai
- Upstream planning context: https://raw.githubusercontent.com/inline0/blockstudio/refs/heads/main/includes/llm/blockstudio-llm.txt
- Project-specific `blockstudio-llm.txt` when available

## Local Validation

```bash
python3 /Users/jonathandailey/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/build-blockstudio-wordpress
python3 /Users/jonathandailey/.agents/skills/skill-creator/scripts/package_skill.py skills/build-blockstudio-wordpress dist
npx skills add . --skill build-blockstudio-wordpress -a codex --list
```

## License

MIT
