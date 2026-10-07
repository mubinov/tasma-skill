# Tasma projects

Read this file before you create, change, rename or remove a project, or answer a
question about projects. The commands and their rules are in `SKILL.md`.

This file has 6 sections: Reference, Create a project, Change a project, Rename the
tag, Remove a project, Common questions.

## Reference

- **Name and tag.** A command names a project by its tag. When the user names a
  project by its name, find its tag in `tasma project list`.
- **Storage.** The tasks and the configuration of a project are in
  `~/.tasma/projects/<TAG>/`. They are not in the project folder (the folder that
  `path` names). No `tasma` command writes into the project folder.
- **Tag.**
  - When the user gives no tag, the daemon makes it from the folder name (the last
    part of the path): up to 4 letters and digits, upper case. When that tag is
    taken, the daemon adds a number, for example `APP2`.
  - The daemon keeps only the Latin letters A-Z and the digits of the folder name,
    and removes the digits at the start. When nothing is left, it makes no tag
    (`tag-not-generated`), for example for the folder names `2024` and `проект`.
  - A tag that the user gives is an upper-case letter, then upper-case letters or
    digits, 2 to 8 characters. The CLI does not convert lower case.
  - The daemon refuses a tag that is taken (`project-exists`).
- **Path.** The path must be an existing folder. Two projects cannot have the same
  folder (`path-taken`). A project inside the folder of another project is allowed.
  In that folder, the inner project answers.
- **Statuses and priorities.**
  - Statuses, the default status, the final statuses and the priorities come from
    `tasma config view` when the project does not set them, and from the built-in
    defaults when the main configuration does not set them. `--clear <field>`
    returns a field to these values.
  - When the project and the main configuration do not set the default status, it is
    the first status of the list. When the project and the main configuration do not
    set the final statuses, the only final status is the last status of the list. So
    an edit of the statuses list can change the default status and the final
    statuses. A status that becomes final closes its tasks, and they stop blocking the
    tasks that they block. A status that stops being final opens its tasks again, and
    the tasks that they block are blocked again.
  - `tasma project view` shows the values, not where they come from.
- **Workflows and project instructions.** These are set only on the project.
  - A new project has no workflows, and its new tasks get no workflow.
  - The first workflow in the list is the default for `task create`.
- **Lists.** `--status`, `--final-status`, `--priority`, `--workflow` and
  `--instruction` replace the stored list. Give the full list, with the flag one time
  for each value, for example `--status Open --status Active --status Closed`.
- **Tasks after a change.** The daemon does not check the tasks when a project
  changes. A task keeps a status, a priority or a workflow that the project no longer
  lists, and Tasma shows no note about it.

## Create a project

1. Run `tasma project list`. If a project has the folder, tell the user and stop.
2. Ask one question in each message. Do not ask what the user already said.
   1. **Folder:** propose the working directory.
   2. **Name:** propose the folder name.
   3. **Workflows:** show the workflows of `tasma workflow list`. The user picks
      none, one or more, and the default. A `tasma: note: workflow-missing` line on
      stderr names a folder in `workflows_path` that is not a workflow. Do not offer
      it.
3. Run `tasma project create --path <folder> --name <name>`. If the user gave a tag,
   add `--tag <tag>`. If the folder name gives no tag (see "Tag" in the Reference),
   ask the user for a tag before you run the command, and add `--tag <tag>`.
4. If the user picked workflows, run `tasma project edit <TAG> --workflow ...` with
   the default first.
5. Run `tasma project view <TAG>`. Tell the user the tag, the folder and the
   workflows.

## Change a project

1. Run `tasma project view <TAG>` to get the current values.
2. **Name, path and project instructions.**
   - Name: `--name`.
   - Path: `--path`. No files move. The new folder must exist, and it must not be the
     folder of another project.
   - Project instructions: the full list with `--instruction`, or
     `--clear instructions`. Each file must exist. Tasma does not create the file:
     to add a new document, write the file first, then give its path.
3. **Statuses and priorities.**
   1. Before the edit, find the tasks that have a value the edit removes:
      `tasma task list --project <TAG> --status <value>` or `--priority <value>`.
      Tell the user the tasks.
   2. When the edit changes the statuses list, keep the default status and the final
      statuses, unless the user asks to change them. Compare the current values from
      `tasma project view` with the new list:
      - If the default status is not the first status of the new list, add
        `--default-status <current>` to the edit.
      - If the final statuses are not exactly one status, the last status of the new
        list, add `--final-status` with the current final statuses.
      - If the new list does not have one of these values, ask the user for the new
        value before the edit. Add the answer to the edit with `--default-status` or
        `--final-status`.
   3. For each field that the edit sets, compare the value that `tasma project view`
      showed in step 1 with the value that `tasma config view` shows. If the two are
      the same, the project takes this field from the main configuration now. Tell the
      user: after the edit, the project keeps its own value, a change of the main
      configuration does not change it, and `--clear <field>` returns the field to the
      main configuration.
   4. Run the edit. The default status and each final status, from the project or
      from the main configuration, must be in the new statuses list.
   5. Offer to move the tasks with `tasma task edit <id> --status <new>` or
      `--priority <new>`. Move them only after the user agrees.
4. **Workflows.**
   1. Before the edit, find the tasks of each workflow that the edit removes, by
      "Find the tasks of a workflow" in `workflows.md`, only in this project. Tell the
      user.
   2. Run the edit.
   3. Offer to move the tasks to a workflow of the new list. Move them only after the
      user agrees. Run `tasma workflow show <new>` to get its steps. Then move each
      task:
      - A task with no step: `tasma task edit <id> --workflow <new>`.
      - A task on a step that the new workflow has:
        `tasma task edit <id> --workflow <new> --step <step>`.
      - A task on a step that the new workflow does not have: ask the user for a
        step of the new workflow, then run
        `tasma task edit <id> --workflow <new> --step <step>`.
5. Run `tasma project view <TAG>` and confirm the result.

## Rename the tag

`tasma project rename` changes only the tag. When the user asks for a new name, for
example "Rename the project Meadow to Green Meadow", change the name with `--name` by
"Change a project".

1. Before the rename, tell the user what changes and what does not:
   - The task ids change, for example `<OLD>-12` becomes `<NEW>-12`. The `parent` and
     `blocked_by` fields change with them.
   - The bodies and the comments of the tasks, the text in other projects, and git
     branches keep the old ids.
2. Run `tasma project rename <OLD> <NEW>` only after the user agrees.
3. Find the text that names an old id. For each project in `tasma project list`, run
   `tasma task list --project <TAG> --search '<OLD>-'`. The search is a
   case-insensitive substring match that also checks the ids. For example, for the
   old tag `AB`, each task of a project `CAB` matches. Check each match with
   `tasma task view <id> --full`, so that the bodies of collapsed comments show,
   and keep only the tasks whose text names an old id. Tell
   the user these tasks. Do not change their text.

## Remove a project

1. Remove only when the user asks for the removal in their own words.
2. Run `tasma task list --project <TAG>`. It lists all tasks, closed tasks included.
   Count them. Each `tasma: excluded:` line on stderr is a task file that the daemon
   cannot read. Count these files separately.
3. Tell the user: `tasma project delete` removes the project and all its tasks (give
   the count), and also the task files that the daemon cannot read (name them). It
   removes the whole folder `~/.tasma/projects/<TAG>/` and every file in it, also
   files that `tasma task list` does not show. Tasma keeps no backup. The project
   folder (the path of the project) does not change.
4. Wait for an explicit confirmation. Then run `tasma project delete <TAG>`.

## Common questions

Give these answers with no research.

- **Where is the data of my project?** In `~/.tasma/projects/<TAG>/`, not in the
  project folder.
- **Why does my new task have no workflow?** The project lists no workflow. Add one
  with `tasma project edit <TAG> --workflow <workflow>`.
- **Can a project be inside the folder of another project?** Yes. In that folder, the
  inner project answers.
- **Does a change of the project path move my files?** No. Tasma does not move,
  change or delete files in the project folder.
- **Can I undo a project delete?** No. Tasma keeps no backup.
- **How do I give a project its own statuses?** Run
  `tasma project edit <TAG> --status ...` with the full list, by step 3 of "Change a
  project": that step keeps the default status and the final statuses. To use the
  statuses of the main configuration again, also follow step 3 of "Change a
  project". Step 3.1 finds the tasks with a status that the list of
  `tasma config view` does not have. Compare the default status and the final
  statuses of `tasma project view` and `tasma config view`, and tell the user which
  of them change. Then run `tasma project edit <TAG> --clear statuses --clear
  default_status --clear final_statuses`.
