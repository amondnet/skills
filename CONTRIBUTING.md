# Contributing

## Development Setup

```bash
bun install
```

## CLI Commands

| Command | Description |
|---------|-------------|
| `bun start init` | Add new submodules defined in `meta.ts` |
| `bun start sync` | Pull latest submodule updates |
| `bun start check` | Check for available upstream updates |
| `bun start cleanup` | Remove unused submodules and skills |

Add `-y` flag to skip interactive prompts.

## Adding a New Project

1. Add to `submodules` in `meta.ts`:

```ts
export const submodules = {
  'mobx.dart': 'https://github.com/amondnet/mobx.dart',
  'new-project': 'https://github.com/org/repo',
}
```

2. Run `bun start init -y` to clone the submodule
3. Ask your agent to: *"Generate skills for new-project"*

## Generating Skills

See `AGENTS.md` for detailed generation guidelines.

## Repository Structure

```
.
├── meta.ts              # Project sources and vendor configs
├── AGENTS.md            # Skill generation guidelines
├── instructions/        # Per-project generation instructions
├── sources/             # Cloned repos (generate skills from docs)
├── skills/              # Output directory (generated skills)
└── scripts/cli.ts       # CLI tool
```
