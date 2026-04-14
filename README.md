# forjis-plugins

Standard plugin repository for Forjis — the AI-powered software development pipeline. This repo bundles reusable agents, skills, hooks, and plugin definitions that Forjis projects can pull in via their `build.forjis` configuration.

This repository does **not** contain the Forjis runtime itself. It is a pure resource repo consumed by the Forjis resolver.

## Usage

Reference this repo from your project's `build.forjis`:

```yaml
version: 1

repositories:
  - type: git
    url: https://github.com/<your-org>/forjis-plugins.git
    ref: main

plugins:
  - name: software-dev
```

The Forjis resolver will fetch the repo, read `manifest.yaml`, and make the listed plugins, agents, skills, and hooks available to your pipeline.

## Repository Layout

```
manifest.yaml         # Top-level index of contents
agents/               # Agent definitions (*.md)
skills/               # Skill definitions (one directory per skill)
hooks/                # Hook definitions (*.md)
plugins/              # Plugin bundles (*.forjis.yaml)
```

### Plugins

| Plugin        | Purpose                                              |
|---------------|------------------------------------------------------|
| `software-dev`| General software development agents and skills      |
| `salesforce`  | Salesforce (Apex, LWC, SOQL, testing) resources      |

### Agents

Role-specialized agent definitions covering the full development lifecycle: explorer, analyst, architect, developer, reviewer, designer, assessor, setup, and finish — each available in fullstack, backend, frontend, and API variants.

### Skills

Language and framework skills including Java, Go, JS/TS, Node-TS, SolidJS, React Native, Android, Swift, shadcn, UI, Postgres, Salesforce (Apex/LWC/SOQL/testing), and meta-skills (`skill-creator`, `mcp-builder`, `theme-factory`).

### Hooks

- `security-check` — security validation hook
- `code-quality` — code quality validation hook

## License

This repository is licensed under the [MIT License](LICENSE).

Some subdirectories contain third-party content distributed under their own license terms. Where a subdirectory includes its own `LICENSE` (or `LICENSE.txt`) file, that license governs the content of that directory and its subdirectories.
