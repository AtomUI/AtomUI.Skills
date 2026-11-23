# AtomUI Skills

[简体中文](README.md)

A collection of skills for AI coding assistants working in the AtomUI ecosystem. Draft Git commit messages, upgrade AtomUI dependencies in consuming projects, and apply API migrations.

The skills follow the Agent Skills open standard and support Claude Code, Codex, and Cursor. [Agent Skills specification](https://agentskills.io/specification)

## Quick start

Run these commands in the project where you want to use the skills. Node.js and `npx` are required.

**List available skills**

```bash
npx skills add AtomUI/AtomUI.Skills --list
```

**Install interactively**

```bash
npx skills add AtomUI/AtomUI.Skills
```

**Install a single skill by name**

```bash
npx skills add AtomUI/AtomUI.Skills --skill git-commit
```

Pass a skill name from the table below to `--skill`. It matches the `name` field in `SKILL.md`.

## Skills

<!-- The skills list is generated from catalog.json, SKILL.md, and skill.json. -->

| Skill | Purpose | Category | Version |
| --- | --- | --- | --- |
| [`git-commit`](skills/Development/git-commit/SKILL.md) | Drafts Git commit messages, checks the staged changes, and creates and verifies local commits when explicitly authorized. Use for “msg”, “commit msg”, “写提交信息”, “提交前检查”, “提交”, or “创建 commit” in a Git context. | `Development` | 1.0.0 |
| [`upgrade-atomui-package-references`](skills/Development/upgrade-atomui-package-references/SKILL.md) | Use only for AtomUI NuGet dependency upgrades and API Break migrations in downstream projects outside the AtomUI framework source repository, including “upgrade AtomUI to the latest version” or “请升级 AtomUI 到最新版本”. | `Development` | 1.0.0 |

## Contributing

1. Choose an enabled top-level category on atomui.net and create a directory at `skills/<category>/<skill-key>/`. [Skill template](template/AtomUIOSS/)
2. Write `SKILL.md` and `skill.json` with the same directory name, skill key, and `name`. [Authoring guidelines (Chinese)](docs/standards/skill-authoring.md) · [Metadata reference (Chinese)](docs/standards/skill-metadata.md)
3. Register the skill in its category and check that the catalog path matches the directory. [Catalog](catalog.json) · [Repository specification (Chinese)](docs/standards/skillshub-engineering.md)

## License

[MIT License](LICENSE)
