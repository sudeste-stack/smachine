# Contributing

GPL-3.0 — fork, modify, redistribute; see [LICENSE](../LICENSE).

## Workflow

1. Fork [sudeste-stack/smachine](https://github.com/sudeste-stack/smachine).
2. Branch, change, open a PR.
3. Keep changes narrow; put each fact in one place and link to it.

## Writing rules (AI and human)

[AGENTS.md](../AGENTS.md) is the single source for style, modularity, and
hallucination-prevention rules in this repo. In particular, for docs:

- No duplicated facts — link to the file that owns the topic.
- No unverified package names, paths, or systemd syntax.
- Externalize long artifacts: UI screens to `repo/screens/`, tables to
  `repo/tables/`, configs to `repo/configs/`, examples to `repo/examples/`.
- Docs describe *what* and *why*; *how* lives in the code.

## Doc map (single source per topic)

| Topic | Owner |
|---|---|
| System layout, boot flow, services, users | [ARCHITECTURE.md](ARCHITECTURE.md) |
| Setup, step by step | [INSTALL.md](INSTALL.md) |
| First-boot menu, what each question means | [SETUP.md](SETUP.md) |
| Configuration, where settings live | [CONFIGURATION.md](CONFIGURATION.md) |
| Updates, snapshots, rollback, troubleshooting | [MAINTENANCE.md](MAINTENANCE.md) |
