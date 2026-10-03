# Tasma workflows

Read this file before you create, change or delete a workflow, or answer a question
about workflows. The commands and their rules are in `SKILL.md`.

This file has 4 sections: Reference, Create a workflow, Change and delete a
workflow, Common questions.

## Reference

- **Workflow.** An ordered list of steps. All projects share the workflows. A project
  lists the workflows that its tasks can use.
- **Where it is.** A workflow is the folder `<workflows_path>/<name>/` with the file
  `workflow.yml`. To find `workflows_path`, run `tasma config view`. If its row has
  the mark `(built-in default)`, the user did not set it, and the folder is
  `~/.tasma/workflows`. `tasma workflow show <name>` prints the path of
  `workflow.yml` (`config`), the workflow instructions and the steps. The folder can
  also hold other files, for example documents, scripts or templates that a document
  names. `workflow delete` removes the folder with all its files.
- **Who writes `workflow.yml`.** The daemon writes it, through `workflow create` and
  `workflow edit`. Do not write it by hand: a hand edit skips the checks.
- **Workflow name.** 1 to 255 characters from `a-z`, `0-9`, `-` and `_`. It starts
  and ends with a letter or a digit. A workflow cannot get a new name.
- **Step name.** Characters from `a-z`, `0-9`, `-`, `_` and `:`. It starts and ends
  with a letter or a digit. It is unique in the workflow and has no `,`. A useful
  form is `<who>:<action>`, for example `dev:research` or `user:review`.
- **Owner.**
  - `agent`: an AI chat does the step.
  - `human`: the user does the step, so the step is a gate.
  - An `agent` step can also contain a gate. Its step document names the gate.
- **Instruction documents.** Markdown files. Tasma does not read their content. The
  agent reads them in this order: the workflow instructions, the project
  instructions, the step instruction.
  - Each step must have a document.
  - Workflow instructions are optional. They hold the rules that all steps of the
    workflow share.
  - One document can have more than one user: a step or the instructions of a
    workflow, and the instructions of a project. To find the users of a document,
    run `tasma workflow show` for each workflow in `tasma workflow list`, and
    `tasma project view <TAG>` for each project in `tasma project list`. Each output
    that prints the path of the document is a user.
- **Write and delete documents.** These rules apply to each procedure in this file.
  - **A new document:** before you write it, make sure that its path does not exist.
    A folder can hold the documents of other workflows. If a file is at that path,
    find its users, tell the user, and do not overwrite the file without the user's
    approval. Use a different file name. If the user wants the step to use the
    existing file, do not write it: give its path to the command, and the file
    becomes a document with more than one user.
  - **An edit:** before you edit a document, find its users. If another workflow or
    a project uses it, tell the user before you edit.
  - **A delete:** delete a document only if the user asks, and only if no other
    workflow and no project uses it.
- **Tasma does not move a task.** Only the agent or the user changes the step. So the
  step document must name the next step for each result.
- **Projects.** `workflows` in the project configuration lists the workflows that its
  tasks can use.
  - The first workflow in the list is the default for `task create`.
  - A project with no list cannot use a workflow.
  - `project edit --workflow` replaces the list, and each workflow in it must exist.
- **Tasks after a change.**
  - A task on a removed step keeps that step. Tasma reports it as `step-stale` until
    someone sets a new step.
  - A task whose workflow was deleted shows `workflow-missing`. This applies also to
    closed tasks and to tasks with no step.
  - A change to a document takes effect in the next chat that reads the document.
- **Find the tasks of a workflow.** `tasma task list` has no workflow filter, and
  `--step` does not find a task that has a workflow and no step. `task create` gives
  a new task the first workflow of its project, so most tasks that did not start have
  a workflow and no step. To find all tasks of a workflow, run
  `tasma task list --project <TAG>` for each project in `tasma project list`. Then
  run `tasma task view <id>` for each task, and keep the tasks whose `workflow` is
  that workflow.

## Create a workflow

### The interview

Ask one question in each message. Do not ask what the user already said. Before the
first question, run `tasma workflow list`. If a workflow of the same kind exists,
offer to start from a copy of it.

1. **Purpose:** what work goes through the workflow, and what result ends it.
2. **Steps:** propose a list of steps in order from the purpose. The user corrects
   it.
3. **For each step:**
   - the owner (`agent` or `human`);
   - what the step reads: the body, the comments, the code, other files;
   - what it does;
   - what it writes: a comment, the body, files;
   - the next step when the result is good, and the next step when the result is
     bad;
   - whether an `agent` step stops for the user (a gate).
4. **Shared rules:** rules that all steps follow. These become the workflow
   instructions. They are optional.
5. **Name and title.**
6. **Folder for the documents:**
   - If the user set `workflows_path` (its row in `tasma config view` has no
     `(built-in default)` mark), propose
     `<the folder that contains workflows_path>/<name>/`.
   - If not, propose `~/tasma-workflows/<name>/`.

   The user confirms the folder or gives a different one. The folder must obey these
   rules:
   - It does not exist, or it is empty. A folder with files can hold the documents
     of other workflows. If the proposed folder has files, tell the user and ask for
     a different folder.
   - It is not inside `workflows_path`. `workflow create` fails when
     `<workflows_path>/<name>/` exists, and `tasma workflow list` reports a folder in
     `workflows_path` without `workflow.yml` as `workflow-missing`.
   - It is not under `~/.tasma`.
7. **Projects:** which projects use the workflow, and whether it is their default.

### The procedure

1. Do the interview.
2. Show a table of the steps: name, owner, next step for a good result, next step for
   a bad result, gates. Wait for the user's approval.
3. Write the documents into the folder, by the rule for a new document (see "Write
   and delete documents" in the Reference). Each step document answers the questions
   of interview item 3: purpose, input, work, output, next step for each result,
   gates. Each step document tells the agent to write its report as a comment with
   the step name as the title. Show the documents and wait for the user's approval.
4. Run `tasma workflow create <name> --title <title> --step <step>,<owner>,<file> ...
   [--instruction <file> ...]`. If the command fails, read the error, correct the
   command and run it again.
5. Run `tasma workflow show <name>` and compare the output with the approved table.
6. For each project from interview item 7:
   1. Run `tasma project view <TAG>` to get the current list.
   2. Run `tasma project edit <TAG> --workflow ...` with the old workflows and the new
      one. Put the new one first only if it is the default.
7. Tell the user the name, the folder and the projects.

## Change and delete a workflow

### Change

1. Run `tasma workflow show <name>` to get the steps and the paths. Read the
   documents that the change concerns.
2. **A document with more than one user:** before you write, edit or delete a
   document, obey "Write and delete documents" in the Reference.
3. **Change of text** (what a step does, its gates, its next steps): edit the
   document. Show the user what changed. Do not run `workflow edit`.
4. **Change of structure:**
   - **Add a step:** write its document first, by the rule for a new document. Put
     it in the folder of the other documents of the workflow if that folder is
     outside `~/.tasma`. If not, put it in a folder that the user confirms, by the
     rule of interview item 6. Then change the next-step text in the documents of the
     steps before it.
   - **Remove or rename a step:** first find the tasks on that step with
     `tasma task list --step <step> --project <TAG>` for each project that lists the
     workflow, and tell the user. Two workflows can have a step with the same name,
     so keep only the tasks whose workflow is this one (`tasma task view <id>` shows
     it). Change each document that names the step as the next step.
   - Run `tasma workflow edit <name> --step ...` with all the steps. Copy the steps
     that do not change from the `workflow show` output.
   - After the edit, show the tasks from the `step-stale` notes. Offer to move them
     with `tasma task edit <id> --step <new step>`. Move them only after the user
     agrees.
   - **The title:** `--title`. **The workflow instructions:** the full list with
     `--instruction`, or `--clear instructions`.
5. **A new name for a workflow:** the CLI cannot rename a workflow. Before the user
   agrees, tell the user that `workflow create` writes only the title, the steps and
   the workflow instructions. Other content of the old `workflow.yml`, for example
   other keys or YAML comments, is lost. Do these steps only if the user agrees:
   1. Do the shared-file check (step 3 of "Delete") for the old workflow.
   2. `workflow delete` removes all files inside `<workflows_path>/<old>/`. Copy all
      files of that folder, except `workflow.yml`, to a folder that the user
      confirms, by the rule of interview item 6. Keep the relative layout of the
      files. Each copy obeys the rule for a new document. Then search the text of
      the copies for the path of the old folder, in the full form and in the `~/`
      form. Where a copy names a file in the old folder, change the path to the path
      of the copy, and show the user the change.
   3. Create a new workflow with the same steps. Give the paths of the copies from
      item 2, and the other paths as they are.
   4. Find the tasks of the old workflow (see "Find the tasks of a workflow" in the
      Reference).
   5. Add the new workflow to each project that lists the old workflow. If the old
      workflow is the first in the list, put the new one first. A project can have
      tasks of the old workflow and not list it. `task edit --workflow <new>` fails in
      a project that does not list the new workflow. Add the new workflow to such a
      project only if the user agrees. If the user does not agree, tell the user
      about the tasks of that project and do not move them.
   6. Move the tasks:
      - A task on a step of the old workflow:
        `tasma task edit <id> --workflow <new> --step <step>`.
      - A task with no step: `tasma task edit <id> --workflow <new>`.
      - A task on a step that the old workflow does not have (`step-stale`): ask the
        user for a step of the new workflow, then run
        `tasma task edit <id> --workflow <new> --step <step>`. If the user gives no
        step, run `tasma task edit <id> --workflow <new>` only after the user agrees.
        The task keeps its stale step.
   7. Remove the old workflow from the projects.
   8. Delete the old workflow with `tasma workflow delete <old>`.
6. Run `tasma workflow show <name>` and confirm the result.

### Delete

1. Delete only when the user asks for the deletion in their own words.
2. Find the projects that list the workflow: run `tasma project list`, then
   `tasma project view <TAG>` for each project. Find the tasks of the workflow (see
   "Find the tasks of a workflow" in the Reference). Tell the user. The tasks are not
   deleted. They show `workflow-missing`.
3. **The shared-file check:** `tasma workflow delete` removes the folder
   `<workflows_path>/<name>/` and all files in it. List all files in that folder,
   except `workflow.yml`, and give the user the list. The list can have files that
   are not documents. Then find the users of each file in the list:
   - the users of a document (see "Instruction documents" in the Reference);
   - each document of another workflow or of a project whose text names the file or
     the folder. Search for the path in the full form and in the `~/` form.

   If another workflow or a project uses a file, tell the user and stop. If not,
   continue only after the user approves the deletion of the files in the list.
4. Remove the workflow from each project with `tasma project edit <TAG> --workflow
   ...` and the other workflows. If it is the only workflow in the list, use
   `--clear workflows`.
5. Run `tasma workflow delete <name>`.
6. Documents outside that folder stay. Give the user their paths. Find the users of
   each of these documents (see "Instruction documents" in the Reference). Mark each
   document that another workflow or a project uses, and tell the user that you do
   not delete it. Delete the other documents only by the rule for a delete (see
   "Write and delete documents" in the Reference).

## Common questions

Give these answers with no research.

- **What is a gate?** A point where the work stops until the user acts. A `human`
  step is a gate. An `agent` step is a gate when its document says so, for example
  "wait for the user's approval of the spec".
- **Why did the task not go to the next step?** Tasma does not move tasks. The agent
  sets the next step because the step document tells it to. If a document does not
  name the next step, add the next step to it.
- **Where are my workflows and documents?** `tasma config view` shows
  `workflows_path`. `tasma workflow show <name>` shows the path of each file.
- **Can two workflows use the same step document?** Yes. A change to the document
  applies to both workflows.
- **What happens to tasks in progress when I change a workflow?** A change to a
  document takes effect in their next chat. A task on a removed step keeps the step
  and shows `step-stale` until someone sets a new step.
- **Why can my task not use workflow X?** The project does not list it. Add it with
  `tasma project edit <TAG> --workflow ...` and the full list.
- **How do I make a workflow the default for new tasks?** Put it first in the project
  list.
- **Can a task skip a step?** Yes. The agent tells the user why and proposes the
  steps to skip, as the "Skip steps" rule in `SKILL.md` says.
- **How do I keep my workflows in git?** Move the workflow folders to a folder in a
  git repo, then run `tasma config edit --workflows-path <folder>`. The command does
  not move files. It fails if a workflow that a project lists is not in the new
  folder. The agent does not move the workflow folders: the user moves them.
- **Can the agent edit `workflow.yml` by hand?** No. Only `workflow create` and
  `workflow edit` write it.
