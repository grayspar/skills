# Grayspar skills

Skills that teach coding agents to use [Grayspar](https://grayspar.com), the cloud for private apps. With them, an agent such as Claude Code, Codex, Cursor or Antigravity deploys the app you're working on, shares it with the people you choose, and reads its logs.

| Skill | What it does |
| --- | --- |
| [grayspar](skills/grayspar/SKILL.md) | Deploys apps to Grayspar and shares them with chosen people, using the `grayspar` CLI |

## Install

For any agent the [skills CLI](https://github.com/vercel-labs/skills) supports:

```sh
npx skills add grayspar/skills
```

In Claude Code, as a plugin:

```
/plugin marketplace add grayspar/skills
/plugin install grayspar@grayspar
```

Then ask your agent to deploy the app and share it. The skill installs the Grayspar CLI when it is missing, after asking you, and has you sign in in your browser.

## License

[Apache-2.0](LICENSE)
