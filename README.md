# tasma-skill

An agent skill for [Tasma](https://github.com/mubinov/tasma), a local task manager.
With this skill, Claude Code and Codex use the `tasma` CLI to run the steps of your
Tasma workflows and to manage tasks and comments.

## What the skill does

- Runs the steps of your Tasma workflows. It reads the workflow, project and step
  instructions, and follows them.
- Creates, shows and lists tasks, and writes comments.

## Requirements

The `tasma` CLI on your `PATH`. See https://github.com/mubinov/tasma.

## Install

Use one install method for each agent. If you install the skill with two methods,
the agent loads two copies.

### npx skills (recommended)

```bash
npx skills add mubinov/tasma-skill -g
```

### Claude Code plugin

```
/plugin marketplace add mubinov/tasma-skill
/plugin install tasma@tasma
```

### Codex plugin

```bash
codex plugin marketplace add mubinov/tasma-skill
codex plugin add tasma@tasma
```

## Use

- Ask the agent in your own words to work with your Tasma tasks.
- To start the skill by name, write `/tasma` in Claude Code or `$tasma` in Codex. If
  you installed the plugin, write `/tasma:tasma` or `$tasma:tasma`.
- In Codex, the first `tasma` command asks for your approval, because the sandbox
  blocks the connection to the local daemon. You can approve all commands that
  start with `tasma`.

## Update

| Method | Command |
|---|---|
| npx skills | `npx skills update tasma -g` |
| Claude Code plugin | `claude plugin marketplace update tasma`, then `claude plugin update tasma@tasma` |
| Codex plugin | `codex plugin marketplace upgrade tasma`, then `codex plugin add tasma@tasma` |

Claude Code does not update third-party plugins automatically. To turn on automatic
updates, run `/plugin`, open **Marketplaces**, select `tasma` and select
**Enable auto-update**.

## Remove

| Method | Command |
|---|---|
| npx skills | `npx skills remove --global tasma` |
| Claude Code plugin | `claude plugin uninstall tasma@tasma` |
| Codex plugin | `codex plugin remove tasma@tasma` |

## License

MIT
