# chao-skills

A collection of AI agent skills for data parsing and content extraction. Works with Claude Code, Cursor, Codex, Windsurf, and 40+ other agents via [skills.sh](https://skills.sh).

## Included Skills

### stdf-reader
Parse and analyze STDF (Standard Test Data Format) semiconductor test files.
Convert STDF to CSV/XLSX, generate analysis reports, correlation reports,
PDF charts, and extract specific test data.

### twitter-article-reader
Extract readable content from Twitter/X articles and tweets using jina.ai.
Bypasses anti-bot measures to return clean markdown content.

## Installation

### Install via skills.sh (recommended)

Works with Claude Code, Cursor, Codex, Windsurf, GitHub Copilot, and more:

```bash
# Install all skills
npx skills add showjim/chao-skills

# Install a specific skill
npx skills add showjim/chao-skills --skill stdf-reader
```

### Claude Code Plugin

```bash
git clone https://github.com/showjim/chao-skills.git
claude --plugin-dir /path/to/chao-skills
```

## Usage

Inside your AI agent, the skills are automatically loaded. For Claude Code plugin mode, invoke as:
- `/chao-skills:stdf-reader`
- `/chao-skills:twitter-article-reader`

## Adding New Skills

1. Create a new directory under `skills/`:
   ```
   skills/your-new-skill/
   ```
2. Add a `SKILL.md` inside it with YAML frontmatter:
   ```markdown
   ---
   name: your-new-skill
   description: What this skill does and when the agent should use it.
   ---

   # Your Skill Title

   Detailed instructions for the skill...
   ```
3. The skill is automatically discovered — no extra config needed.

## License

MIT
