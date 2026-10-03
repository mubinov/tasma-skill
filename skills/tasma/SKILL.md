---
name: tasma
description: 'Manages tasks in Tasma, a local task manager, through the tasma CLI: tasks, comments, projects and workflow steps. Use when the user names Tasma, or when the first message of a conversation has no other context and could be about a local task manager, for example "next step", "next", "research task XX", "show task XX" or "create a task".'
license: MIT
allowed-tools: Bash(tasma:*)
---

## Contents

This file has the 5 sections below. The last section is "Hard rules and failures".
If your copy of this file ends before that section, read the full file.

1. Tasma
2. Run a workflow step
3. Direct requests
4. Commands
5. Hard rules and failures

## Tasma

Tasma is a local task manager. Its data is in `~/.tasma`. A local daemon
manages this data. The `tasma` CLI sends all requests to the daemon.

- **Project.** A project has a tag `<TAG>`, a name and a folder path. A
  project includes its folder and all subfolders of that folder. A command
  that needs a project gets it from the working directory.
- **Task.** The id of a task is `<TAG>-<number>`. A task has a title, a
  status, a priority, labels, a parent, blockers, a body and comments. A
  task can also have a workflow and a step.
- **Workflow.** A workflow is an ordered list of steps. All projects share
  the workflows. A project selects the workflows that its tasks can use.
  Each step has a name, an owner and a step instruction. The owner is
  `agent` or `human`.
- **Step.** The step of a task shows the current stage of the task in its
  workflow. The step must be a step of the workflow of the task. A task
  without a workflow has no step. Tasma does not change the step by itself.
  The step changes only when the agent or the user sets it.
- **Status.** Each project has its own list of statuses. A project without
  its own statuses takes them from the main configuration. A status in
  `final_statuses` means that the task is closed. The step and the status
  are different fields. A change to one field does not change the other.
- **Comment.** A comment has a number, a title, an author and a body. A
  collapsed comment shows only its title.
- **Instruction documents.** These are text files for the agent. The three
  kinds are the workflow instructions, the project instructions and the step
  instruction.

## Run a workflow step

Each step runs in a new chat. A new chat has no knowledge of the previous chats.
The task keeps the state of the work. The step shows the current stage. The body
contains the approved text. The comments contain the report of each step.

### Find the task

Use this procedure when the user asks to run a step of a task, for example
"next step", "continue" or "research task XX".

1. If the request names a task id, use that task.
2. If the current git branch is `<TAG>-<number>` and that task exists, use that task.
3. If not, find the most suitable task to continue or to start. If the request
   contains words of the title or the text of the task, run
   `tasma task list --search '<text>'` with these words as `<text>`. Replace each
   `'` in the words with a space: the single quotes stop the shell from changing
   `$` and backticks. Write the task id in the first line of your reply.

### Start the task

If the task has no step, start its workflow. Set the first step of the workflow.
Set the status that the project uses for work in progress. If the task has no
workflow, ask the user which workflow to use.

### Read the instruction documents

1. Run `tasma task view <TAG>-<number>`. Read the workflow, the step, the body and
   the comments.
2. Run `tasma workflow show <workflow>` and `tasma project view <TAG>`. They show the
   paths of the instruction documents.
3. Read the documents in this order: the workflow instructions, the project
   instructions, the step instruction.
4. Do the work that the documents tell you to do.

If an instruction document says something different from this skill, the document wins.

### Gates

- Each `human` step is a gate. Its step instruction tells what to show and what to
  wait for.
- An `agent` step can also contain a gate. The instruction documents name these gates.
- At a gate, stop and wait for the user.

### End the step

- When a step is complete, write a hand-off as your last message:
  1. `# Done: <from-step> → <to-step>`, and a summary of one line.
  2. The name of the next step.
  3. `# Required from user`, with each action that only the user can do. If there is
     no such action, do not write this block.
- Do not start the next step in the same chat.
- A task is done when the last step of its workflow is complete. Then set a final
  status. If the last step is a `human` step, set the final status when the user
  approves. Do not ask the user again to close the task.

### Comments

- After the user approves the body, do not change the body. Each step writes its
  report as a new comment.
- The title of a comment is the step name. If the step runs more than one time, add
  the round number, for example `<step> #2`. Put the title in `--title`, not in the body.
- A step that checks the work writes its result in a comment, pass or fail.
- When a new comment replaces a decision in an old comment, collapse the old comment.
  Do not edit it and do not delete it. In the new comment, write which comment it
  replaces and why.

### Skip steps

A task that does not fit its workflow can skip steps. Tell the user why, and propose
the steps to skip.

## Direct requests

### Create a task

When the user says "create a task" or "new task", only create the task. Do not
research, plan, fix or test anything.

- Write the words of the user in the body. Do not change them. If the user gives a
  log or an error, the body is that log or error.
- Do not add your interpretation. Do not add a cause, a fix, a plan or a checklist.
- You can add a short fact that you know, for example "Follow-up of
  <TAG>-<number>". Write it in the title or in the body. Do not add a label unless
  the user asks for it.
- Do not start the task. Do not set a step. Use the default status of the project,
  unless the user names a status or says that the task is ready.
- If the user names a workflow, add `--workflow <workflow>`. If not, do not add it.
- If the request is too vague to be a task, ask the user first. Ask only the
  questions that make the task clear. Write the answers of the user in the body. Do
  not investigate and do not guess.

The next steps read the body as the words of the user. Text that you add looks
like a requirement to them.

### View and list tasks

- To show a task, run `tasma task view <TAG>-<number>`.
- To list tasks, run `tasma task list` with filters.

### Split a task

- Split a task only when its parts can be done separately. A large task that you
  cannot divide stays one task.
- A pre-task blocks the current task. Create the pre-task. On the current task, set
  `--blocked-by <pre-task id>` and the status that the project uses for ready tasks.
  Add a comment that names the pre-task.
- A follow-up task does not block the current task. Create it with the status that
  the project uses for tasks that are not ready.
- Give a new task a ready status only when its work is clear enough to start
  without more questions.
- Propose the split to the user. Create the tasks only after the user approves.
- In the body of each new task, write its relation: `Pre-task for <TAG>-<number>`,
  `Follow-up of <TAG>-<number>` or `Split from <TAG>-<number>`.

### Create a project

If no project includes the working directory, `task list` and `task create` stop
with exit code 2. Tell the user, and offer `tasma project create --path <folder>`.
Run it only after the user agrees.

### Delete

Run `tasma task delete`, `tasma comment delete` or `tasma project delete` only when
the user asks for that deletion in their own words. Do not delete anything on your
own decision. `tasma project delete` removes all tasks of the project and does not
ask for confirmation.

## Commands

Use these commands directly. Run `tasma <group> <command> --help` only after a
command fails.

### Tasks

| Purpose | Command |
|---|---|
| Show a task | `tasma task view <TAG>-<number>` |
| Show a task with the bodies of collapsed comments | `tasma task view <TAG>-<number> --full` |
| List tasks | `tasma task list [--search <text>] [--status <s>] [--priority <p>] [--step <s>] [--label <l>] [--parent <id>] [--blocked \| --unblocked] [--project <TAG>]` |
| Create a task | `tasma task create --title <title> [--body-file -] [--status <s>] [--priority <p>] [--workflow <w>] [--parent <id>] [--label <l>] [--blocked-by <id>] [--project <TAG>]` |
| Change fields | `tasma task edit <TAG>-<number> [--title <title>] [--status <s>] [--priority <p>] [--workflow <w>] [--step <s>] [--parent <id>] [--label <l>] [--blocked-by <id>] [--order <n>]` |
| Change the body | `tasma task edit <TAG>-<number> --body-file - [--append]` |
| Remove a field | `tasma task edit <TAG>-<number> --clear <field>` |
| Delete a task | `tasma task delete <TAG>-<number>` |

### Comments

| Purpose | Command |
|---|---|
| List the comments | `tasma comment list <TAG>-<number>` |
| Show one comment | `tasma comment view <TAG>-<number> <n>` |
| Add a comment | `tasma comment add <TAG>-<number> --title <title> --body-file -` |
| Change a comment | `tasma comment edit <TAG>-<number> <n> [--title <title>] [--body-file - [--append]]` |
| Collapse a comment | `tasma comment edit <TAG>-<number> <n> --collapsed` |
| Expand a comment | `tasma comment edit <TAG>-<number> <n> --clear collapsed` |
| Delete a comment | `tasma comment delete <TAG>-<number> <n>` |

### Projects and workflows

| Purpose | Command |
|---|---|
| Show the project of the working directory | `tasma project current` |
| List the projects | `tasma project list` |
| Show the configuration of a project | `tasma project view <TAG>` |
| Create a project | `tasma project create --path <path> [--name <name>] [--tag <TAG>]` |
| Change the configuration of a project | `tasma project edit <TAG> [--name <name>] [--path <path>] [--status <s>] [--default-status <s>] [--final-status <s>] [--priority <p>] [--workflow <w>] [--instruction <path>]` |
| Change the tag of a project | `tasma project rename <old> <new>` |
| Delete a project | `tasma project delete <TAG>` |
| Show the main configuration | `tasma config view` |
| Change the main configuration | `tasma config edit [--status <s>] [--default-status <s>] [--final-status <s>] [--priority <p>] [--workflows-path <path>]` |
| List the workflows | `tasma workflow list` |
| Show a workflow | `tasma workflow show <workflow>` |

### Text with more than one line

Send the body on standard input with a quoted heredoc:

```bash
tasma comment add <TAG>-<number> --title "<title>" --body-file - <<'TASMA_BODY'
<text>
TASMA_BODY
```

The quotes on `'TASMA_BODY'` stop the shell from changing `$` and backticks. Use
`--body <text>` only for one short line.

### Rules

- `task list` and `task create` get the project from the working directory.
  `--project <TAG>` names a different project.
- A command with a task id works in any folder.
- A write command prints only the id on stdout. Notes and errors go to stderr. Each
  note and error starts with `tasma:`.
- To set a workflow and a step on a task without a workflow, give `--workflow` and
  `--step` in one command.
- `--label` and `--blocked-by` replace the stored list. So do `--status`,
  `--final-status`, `--priority`, `--workflow` and `--instruction` of
  `project edit`, and `--status`, `--final-status` and `--priority` of
  `config edit`. To add one value, give all the old values and the new value.
- `--body` and `--body-file` replace the body. `--append` adds the text after the
  body. It works only with `task edit` and `comment edit`.
- A value that starts with `-` needs the form `--<flag>=<value>`, for example
  `--order=-1`.
- `--clear` removes one field. For `task edit`, the field names are `priority`,
  `labels`, `parent`, `blocked_by`, `step`, `workflow`, `order` and `body`. For
  `comment edit`, they are `author`, `collapsed` and `body`. For `project edit`,
  they are `name`, `statuses`, `default_status`, `final_statuses`, `priorities`,
  `workflows` and `instructions`. For `config edit`, they are `statuses`,
  `default_status`, `final_statuses`, `priorities` and `workflows_path`.
- A task has a link: `tasma://task/<TAG>-<number>`. It opens the task in the macOS
  application. When you name a task in the chat, write it as a markdown link with the
  id as the text: `[<TAG>-<number>](tasma://task/<TAG>-<number>)`. Inside a task body
  or a comment, write the id alone.
- A task can be very large. `task view` does not show the bodies of collapsed
  comments. `comment list` shows the number, the size and the title of each
  comment. To read one comment, run `comment view`.

## Hard rules and failures

### Use only the CLI

- Do not read, edit or delete files under `~/.tasma` with other tools. Use the
  `tasma` CLI for all data of projects, tasks and comments. A task file can be too
  large to read, and a direct edit skips the checks of the daemon.
- Read the instruction documents at the paths that `tasma workflow show` and
  `tasma project view` print. This is the only exception. The CLI does not print
  the text of these documents.

### `tasma` is not installed

If the shell cannot find `tasma`, stop. Tell the user that the tasma CLI is not
installed, and give this link: https://github.com/mubinov/tasma.

### A command fails

- Exit code 1. Tasma refused the request. Read the error on stderr, correct the
  command and run it again.
- Exit code 2. The command has a usage error, or no project includes the working
  directory. Run `tasma <group> <command> --help`. For a missing project, see
  "Create a project".
- Exit code 3. The CLI cannot connect to the daemon. In Codex, first do the steps in
  "Codex sandbox". Then run `tasma daemon status`. If it fails, stop, tell the user
  and show the error. Do not stop processes, and do not change files under `~/.tasma`.

### Codex sandbox

In the default Codex sandbox, `tasma` cannot connect to the local daemon. The
command fails with exit code 3, and the error looks like a daemon that does not run.
Run the same command again with `sandbox_permissions: "require_escalated"`, a short
`justification` and `prefix_rule: ["tasma"]`. Do not ask the user to change the
Codex or Claude Code configuration.
