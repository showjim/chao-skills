# chao-skills

A Claude Code plugin providing a collection of skills for data parsing and content extraction.

## Included Skills

### stdf-reader
Parse and analyze STDF (Standard Test Data Format) semiconductor test files.
Convert STDF to CSV/XLSX, generate analysis reports, correlation reports,
PDF charts, and extract specific test data.

### twitter-article-reader
Extract readable content from Twitter/X articles and tweets using jina.ai.
Bypasses anti-bot measures to return clean markdown content.

## Installation

### Local Development / Testing
```bash
# Clone the repository
git clone https://github.com/showjim/chao-skills.git

# Run Claude Code with the plugin loaded
claude --plugin-dir /path/to/chao-skills
```

### From Marketplace (once published)
```
/plugin install chao-skills
```

## Usage

Inside Claude Code, invoke skills as:
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
   description: What this skill does and when Claude should use it.
   ---

   # Your Skill Title

   Detailed instructions for the skill...
   ```
3. The skill is automatically discovered — no changes to `plugin.json` needed.

## License

MIT
