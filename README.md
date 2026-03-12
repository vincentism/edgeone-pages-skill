# EdgeOne Pages Skills

Official [Agent Skills](https://github.com/anthropics/agent-skills) for deploying projects to [EdgeOne Pages](https://edgeone.ai/products/pages).

## Installation

```bash
npx skills add user/edgeone-pages-skill
```

After installation, your AI coding agent will automatically detect when you want to deploy and use the appropriate skill.

## Available Skills

### `edgeone-pages-deploy`

Deploy frontend or full-stack projects to EdgeOne Pages with a single command.

**Triggers**: "deploy my project", "deploy to EdgeOne Pages", "publish this site", "push this live"

**What it does**:
- Installs the EdgeOne CLI (`edgeone`) if not present
- Authenticates via browser login (preferred) or API token (headless/CI)
- Supports both China and Global sites
- Deploys with automatic framework detection and build
- Returns the live preview URL and console link

**Supported project types**:
- Static frontend projects (HTML, React, Vue, Next.js, etc.)
- Full-stack projects with Edge Functions or Node Functions

## Skill Structure

```
skills/
└── edgeone-pages-deploy/
    └── SKILL.md          # Agent instructions for deployment
```

Each skill contains:
- `SKILL.md` — YAML frontmatter (name + description) followed by Markdown instructions for the agent

## Requirements

- **Node.js** ≥ 16
- **npm** (for CLI installation)
- A [Tencent Cloud](https://cloud.tencent.com/) account (China site) or [Tencent Cloud International](https://intl.cloud.tencent.com/) account (Global site)

## Links

- [EdgeOne Pages Documentation](https://edgeone.ai/document/pages)
- [EdgeOne Pages Console (China)](https://console.cloud.tencent.com/edgeone/pages)
- [EdgeOne Pages Console (Global)](https://console.intl.cloud.tencent.com/edgeone/pages)

## License

MIT
