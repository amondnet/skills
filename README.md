# Minsu Lee's Skills

[Agent Skills](https://agentskills.io/home) generated and maintained from source documentation.

Based on [antfu/skills](https://github.com/antfu/skills) — a proof-of-concept for generating and maintaining agent skills from project docs.

## Install

```bash
pnpx skills add amondnet/skills --skill='*'
```

## Development Setup

```bash
bun install
```

## Usage

### Generate skills from source

1. Update `meta.ts` with project sources
2. Run `bun start init -y` to clone submodules
3. Ask your agent to: *"Generate skills for mobx.dart"*

### CLI Commands

| Command | Description |
|---------|-------------|
| `bun start init` | Add new submodules defined in `meta.ts` |
| `bun start sync` | Pull latest submodule updates |
| `bun start check` | Check for available upstream updates |
| `bun start cleanup` | Remove unused submodules and skills |

Add `-y` flag to skip interactive prompts.

## Structure

```
.
├── meta.ts              # Project sources and vendor configs
├── AGENTS.md            # Skill generation guidelines
├── instructions/        # Per-project generation instructions
├── sources/             # Cloned repos (generate skills from docs)
├── skills/              # Output directory (generated skills)
└── scripts/cli.ts       # CLI tool
```

## Adding a New Project

Add to `submodules` in `meta.ts`:

```ts
export const submodules = {
  'mobx.dart': 'https://github.com/amondnet/mobx.dart',
  'new-project': 'https://github.com/org/repo',
}
```

See `AGENTS.md` for detailed generation guidelines.

## License

MIT
