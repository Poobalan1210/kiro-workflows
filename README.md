# kiro-workflows

Workflow recipes for [Kiro](https://kiro.dev). A recipe is a small file that tells Kiro which agents to run, in what order, and on which model. Kiro runs it in the background while you keep working in the chat.

## issue-to-deploy

[`workflows/issue-to-deploy.workflow.yaml`](workflows/issue-to-deploy.workflow.yaml) takes a GitHub issue all the way to a staging deploy.

| Step | What it does | Model |
|---|---|---|
| intake | Reads the issue and writes a brief with the acceptance criteria | Qwen3 Coder Next |
| plan | Reads the code, plans the change, finds your test and build commands | MiniMax M2.5 |
| build | Makes the change, adds tests, commits | Qwen3 Coder Next |
| code-review | Checks the change against the issue and runs the tests itself | Claude Sonnet 5 |
| security-review | Looks for vulnerabilities and runs your audit tools | GLM-5 |
| gate | Approves only if both reviews approve | Claude Haiku 4.5 |
| verify | Runs every check and writes the PR description | DeepSeek 3.2 |
| ship | Pushes, opens a draft PR, waits for CI | Claude Haiku 4.5 |
| deploy | Asks you, then deploys to staging | Claude Haiku 4.5 |

Build, the two reviews, and the gate repeat until both reviewers approve, up to three rounds. The two reviews run side by side, each in a fresh session. If CI fails, the run stops before deploying. It never merges and never deploys to production.

### What you need

- Kiro with Workflows turned on. In the IDE, open Workspace Configuration (in Agent Focus), choose Workflows, and switch it on.
- git, a GitHub remote, and the GitHub CLI (`gh`) installed and signed in.
- Access to the models in the recipe. Haiku 4.5 and Sonnet 5 need a paid Kiro plan, and some of the others are marked Experimental. Check each `modelId` against your model picker, or delete the line to use your chat's model.

### Install it

```sh
# Save the recipe where every project can use it
mkdir -p ~/.kiro/workflows
curl -fsSL https://raw.githubusercontent.com/Poobalan1210/kiro-workflows/main/workflows/issue-to-deploy.workflow.yaml \
  -o ~/.kiro/workflows/issue-to-deploy.workflow.yaml

# In your project: keep the run notes out of git
echo ".kiro/workflow-output/" >> .git/info/exclude
```

Read the recipe before you run it. It runs commands, commits, and pushes a branch. To share it with your team, commit it to `.kiro/workflows/` on your default branch instead.

### Run it

Start from an up-to-date default branch with a clean working tree, then ask in a Kiro chat:

> Run the issue-to-deploy workflow for issue 142. To deploy, run `make deploy-staging`, then check that https://staging.example.com/health returns 200.

To work on several issues at once, give each one its own git worktree and open it in its own Kiro window.

### Change it

It's a plain YAML file. Swap the models, add or remove steps, add your team's rules to the review prompts, or point the intake step at another tracker. The [Kiro Workflows docs](https://kiro.dev/docs/workflows/) explain every field.
