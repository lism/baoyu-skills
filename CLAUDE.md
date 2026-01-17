# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code marketplace plugin providing AI-powered content generation skills. Skills use Gemini Web API (reverse-engineered) for text/image generation and Chrome CDP for browser automation.

## Architecture

```
skills/
├── baoyu-gemini-web/          # Core: Gemini API wrapper (text + image gen)
├── baoyu-xhs-images/          # Xiaohongshu infographic series (1-10 images)
├── baoyu-cover-image/         # Article cover images (2.35:1 aspect)
├── baoyu-slide-deck/          # Presentation slides with outlines
├── baoyu-article-illustrator/ # Smart illustration placement
├── baoyu-post-to-x/           # X/Twitter posting automation
└── baoyu-post-to-wechat/      # WeChat Official Account posting
```

Each skill contains:
- `SKILL.md` - YAML front matter (name, description) + documentation
- `scripts/` - TypeScript implementations
- `prompts/system.md` - AI generation guidelines (optional)

## Running Skills

All scripts run via Bun (no build step):

```bash
npx -y bun skills/<skill>/scripts/main.ts [options]
```

Examples:
```bash
# Text generation
npx -y bun skills/baoyu-gemini-web/scripts/main.ts "Hello"

# Image generation
npx -y bun skills/baoyu-gemini-web/scripts/main.ts --prompt "A cat" --image cat.png

# From prompt files
npx -y bun skills/baoyu-gemini-web/scripts/main.ts --promptfiles system.md content.md --image out.png
```

## Key Dependencies

- **Bun**: TypeScript runtime (via `npx -y bun`)
- **Chrome**: Required for `baoyu-gemini-web` auth and `baoyu-post-to-x` automation
- **No npm packages**: Self-contained TypeScript, no external dependencies

## Authentication

`baoyu-gemini-web` uses browser cookies for Google auth:
- First run opens Chrome for login
- Cookies cached in data directory
- Force refresh: `--login` flag

## Plugin Configuration

`.claude-plugin/marketplace.json` defines plugin metadata and skill paths. Version follows semver.

## Adding New Skills

**IMPORTANT**: All skills MUST use `baoyu-` prefix to avoid conflicts when users import this plugin.

1. Create `skills/baoyu-<name>/SKILL.md` with YAML front matter
   - Directory name: `baoyu-<name>`
   - SKILL.md `name` field: `baoyu-<name>`
2. Add TypeScript in `skills/baoyu-<name>/scripts/`
3. Add prompt templates in `skills/baoyu-<name>/prompts/` if needed
4. Register in `marketplace.json` plugins[0].skills array as `./skills/baoyu-<name>`
5. **Add Script Directory section** to SKILL.md (see template below)

### Script Directory Template

Every SKILL.md with scripts MUST include this section after Usage:

```markdown
## Script Directory

**Important**: All scripts are located in the `scripts/` subdirectory of this skill.

**Agent Execution Instructions**:
1. Determine this SKILL.md file's directory path as `SKILL_DIR`
2. Script path = `${SKILL_DIR}/scripts/<script-name>.ts`
3. Replace all `${SKILL_DIR}` in this document with the actual path

**Script Reference**:
| Script | Purpose |
|--------|---------|
| `scripts/main.ts` | Main entry point |
| `scripts/other.ts` | Other functionality |
```

When referencing scripts in workflow sections, use `${SKILL_DIR}/scripts/<name>.ts` so agents can resolve the correct path.

## Code Style

- TypeScript throughout, no comments
- Async/await patterns
- Short variable names
- Type-safe interfaces

## Image Generation Guidelines

Skills that require image generation MUST delegate to available image generation skills rather than implementing their own.

### Image Generation Skill Selection

1. Check available image generation skills in `skills/` directory
2. Read each skill's SKILL.md to understand parameters and capabilities
3. If multiple image generation skills available, ask user to choose preferred skill

### Generation Flow Template

Use this template when implementing image generation in skills:

```markdown
### Step N: Generate Images

**Skill Selection**:
1. Check available image generation skills (e.g., `baoyu-gemini-web`)
2. Read selected skill's SKILL.md for parameter reference
3. If multiple skills available, ask user to choose

**Generation Flow**:
1. Call selected image generation skill with:
   - Prompt file path (or inline prompt)
   - Output image path
   - Any skill-specific parameters (refer to skill's SKILL.md)
2. Generate images sequentially (one at a time)
3. After each image, output progress: "Generated X/N"
4. On failure, auto-retry once before reporting error
```

### Best Practices

- Always read the image generation skill's SKILL.md before calling
- Pass parameters exactly as documented in the skill
- Handle failures gracefully with retry logic
- Provide clear progress feedback to user
