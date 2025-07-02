---
author: muju
title: "github workflow"
description: ""
categories: ["Devops"]
tags: ["Linux", "Github"]
image:
  path: /assets/devops/gh-actions/actions.webp
  lqip: data:image/webp;base64,UklGRpQAAABXRUJQVlA4IIgAAABwBACdASoUABQAPpFAmkmlo6IhKAqosBIJYwC7ABEdGShyt+sx9hg+4UhwgAD++l54Ig5t+pTplW0/+EzAezj7gIGOyTZmXCGzEx6vy+42amWghdMKU6CcQeOKwTnECgC3yrUVI8KhZ0IQgeCxm2TrtB+KhveWjQy90eICU9twNVPa4AYMQAAA
published: true
hidden: true
toc: true
---
# GH workflow

github workflow is a way to automate the process of building, testing, and deploying code. It is a set of instructions that you can define in a yaml file. You can define the steps in the workflow and the workflow will run the steps in the order that you define. You can also define the conditions for running the workflow. For example, you can define a workflow to run when a pull request is created or when a push is made to the main branch.

## GH workflow syntax

The syntax for a github workflow is defined in a yaml file. The yaml file is a text file that is defined by a set of rules. The rules are defined in the [yaml spec](https://yaml.org/spec/1.2/spec.html).

The following is an example of a yaml file that defines a github workflow.

```yaml
name: my workflow
on:
  push:
    branches:
      - main
jobs:
  my job:
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3
      - name: run a command
        run: echo "hello world"
```

## GH default variables

GitHub provides several built-in contexts to access info dynamically:

### 1. Workflow & Job Information

| Variable                  | Description                                                      | Usage Example              |
| ------------------------- | ---------------------------------------------------------------- | -------------------------- |
| `GITHUB_WORKFLOW`         | The name of the workflow                                         | `$GITHUB_WORKFLOW`         |
| `GITHUB_RUN_ID`           | Unique ID for the current run                                    | `$GITHUB_RUN_ID`           |
| `GITHUB_RUN_NUMBER`       | The run number of the workflow                                   | `$GITHUB_RUN_NUMBER`       |
| `GITHUB_JOB`              | ID of the current job                                            | `$GITHUB_JOB`              |
| `GITHUB_ACTION`           | Name of the action currently running                             | `$GITHUB_ACTION`           |
| `GITHUB_ACTIONS`          | Always set to `true` in GitHub Actions                           | `$GITHUB_ACTIONS`          |
| `GITHUB_ACTOR`            | The user who triggered the workflow                              | `$GITHUB_ACTOR`            |
| `GITHUB_REPOSITORY`       | Repository name (e.g. `user/repo`)                               | `$GITHUB_REPOSITORY`       |
| `GITHUB_REPOSITORY_OWNER` | Owner of the repository                                          | `$GITHUB_REPOSITORY_OWNER` |
| `GITHUB_EVENT_NAME`       | Event that triggered the workflow (e.g., `push`, `pull_request`) | `$GITHUB_EVENT_NAME`       |
| `GITHUB_EVENT_PATH`       | Path to the event payload JSON file                              | `$GITHUB_EVENT_PATH`       |
| `GITHUB_SHA`              | Commit SHA that triggered the workflow                           | `$GITHUB_SHA`              |
| `GITHUB_REF`              | Git ref (branch or tag)                                          | `$GITHUB_REF`              |
| `GITHUB_REF_NAME`         | Short name of the ref                                            | `$GITHUB_REF_NAME`         |
| `GITHUB_REF_TYPE`         | Type of the ref (`branch` or `tag`)                              | `$GITHUB_REF_TYPE`         |

### 2. File & Directory Paths

| Variable             | Description                                           | Usage Example                        |
| -------------------- | ----------------------------------------------------- | ------------------------------------ |
| `GITHUB_WORKSPACE`   | Directory where the repository is checked out         | `$GITHUB_WORKSPACE`                  |
| `GITHUB_ACTION_PATH` | Directory path where the GitHub Action is located     | `$GITHUB_ACTION_PATH`                |
| `GITHUB_ENV`         | File path for setting environment variables in a step | `echo "FOO=bar" >> $GITHUB_ENV`      |
| `GITHUB_PATH`        | File path to add values to the `PATH` variable        | `echo "/custom/bin" >> $GITHUB_PATH` |

### 3. Runner & Environment Information

| Variable            | Description                                    | Usage Example        |
| ------------------- | ---------------------------------------------- | -------------------- |
| `RUNNER_NAME`       | Name of the runner executing the job           | `$RUNNER_NAME`       |
| `RUNNER_OS`         | OS of the runner (e.g., Linux, Windows, macOS) | `$RUNNER_OS`         |
| `RUNNER_ARCH`       | Architecture of the runner (e.g., X64, ARM)    | `$RUNNER_ARCH`       |
| `RUNNER_TEMP`       | Path to a temp directory                       | `$RUNNER_TEMP`       |
| `RUNNER_TOOL_CACHE` | Tool cache directory for actions/tool caching  | `$RUNNER_TOOL_CACHE` |

### 4. GitHub Token & Secrets

| Variable           | Description                                     | Usage Example              |
| ------------------ | ----------------------------------------------- | -------------------------- |
| `GITHUB_TOKEN`     | Auto-generated token for the job for API access | `$GITHUB_TOKEN`            |
| `secrets.<SECRET>` | Your custom or org-wide secrets (via GitHub UI) | {% raw %}`${{ secrets.MY_SECRET }}`{% endraw %} |

### 5. Built-in Workflow Contexts

| Context    | Description                                         | Example Usage                              |
| ---------- | --------------------------------------------------- | ------------------------------------------ |
| `github`   | Metadata about the repository and event             | {% raw %}`${{ github.repository }}`{% endraw %}                 |
| `env`      | User-defined environment variables                  | {% raw %}`${{ env.ENV_VAR_NAME }}`{% endraw %}                  |
| `secrets`  | Encrypted secrets                                   | {% raw %}`${{ secrets.MY_SECRET }}`{% endraw %}                 |
| `runner`   | Runner metadata                                     | {% raw %}`${{ runner.os }}`{% endraw %}                         |
| `job`      | Info about current job (e.g., status)               | {% raw %}`${{ job.status }}`{% endraw %}                        |
| `matrix`   | Matrix values for the current job                   | {% raw %}`${{ matrix.node_version }}`{% endraw %}               |
| `strategy` | Matrix strategy values                              | {% raw %}`${{ strategy.job-index }}`{% endraw %}                |
| `steps`    | Access previous step outputs                        | {% raw %}`${{ steps.step_id.outputs.some_output }}`{% endraw %} |
| `needs`    | Access outputs from dependent jobs                  | {% raw %}`${{ needs.build.outputs.artifact_url }}`{% endraw %}  |
| `inputs`   | Inputs to reusable workflows or `workflow_dispatch` | {% raw %}`${{ inputs.username }}`{% endraw %}                   |
| `vars`     | Repository/organization-level variables             | {% raw %}`${{ vars.SITE_NAME }}`{% endraw %}                    |

### 6. GitHub Actions Triggers

| Trigger (`on:`)               | Description                                                                 |
|-------------------------------|-----------------------------------------------------------------------------|
| `push`                        | Triggered when a commit is pushed to the repository                         |
| `pull_request`                | Triggered when a pull request is opened, synchronized, etc.                 |
| `workflow_dispatch`           | Manual trigger via GitHub UI or API                                        |
| `repository_dispatch`         | Triggered from external systems via API call                               |
| `schedule`                    | Triggers at scheduled times using CRON syntax                              |
| `check_run`                   | Triggered for check runs (create, update, rerequest)                       |
| `check_suite`                 | Triggered for check suites (completed, requested)                          |
| `create`                      | Triggered when a branch or tag is created                                  |
| `delete`                      | Triggered when a branch or tag is deleted                                  |
| `deployment`                  | Triggered when a deployment is created                                     |
| `deployment_status`          | Triggered when the deployment status changes                               |
| `discussion`                  | Triggered on GitHub Discussions activity (created, edited, etc.)           |
| `discussion_comment`          | Triggered when a comment is created or edited in a discussion              |
| `fork`                        | Triggered when someone forks the repository                                |
| `gollum`                      | Triggered when a wiki page is created or updated                           |
| `issue_comment`               | Triggered when an issue or pull request comment is created/edited/deleted |
| `issues`                      | Triggered on issue activities (open, close, label, etc.)                   |
| `label`                       | Triggered when a label is created, edited, or deleted                      |
| `member`                      | Triggered when a collaborator is added, removed, or changed                 |
| `merge_group`                 | Triggered when a merge group is created (GitHub Merge Queue)               |
| `milestone`                   | Triggered when a milestone is created, closed, or deleted                  |
| `page_build`                  | Triggered when GitHub Pages is built                                       |
| `project`                     | Triggered when a project is created, closed, or reopened                   |
| `project_card`                | Triggered on changes to project cards                                      |
| `project_column`              | Triggered on changes to project columns                                    |
| `public`                      | Triggered when a repository is made public                                 |
| `pull_request_review`         | Triggered when a PR review is submitted                                    |
| `pull_request_review_comment` | Triggered when a PR review comment is made                                 |
| `pull_request_target`         | Same as `pull_request` but runs in the context of the base repo            |
| `release`                     | Triggered on release events (created, published, etc.)                     |
| `registry_package`            | Triggered when a package is published, updated, or deleted                 |
| `status`                      | Triggered when the status of a commit changes                              |
| `watch`                       | Triggered when someone stars the repository                                |
| `workflow_call`               | Triggered by another workflow using `uses:`                                |
| `workflow_run`                | Triggered when another workflow run finishes                               |

```yaml
on:
  push:
    branches: [main]
  pull_request:
    types: [opened, reopened, synchronize]
  workflow_dispatch:
```

## GH workflow steps

A github workflow is defined by a set of steps. Each step is defined in a job. A job is a set of steps that run on the same runner. The steps in a job run in the order that you define them.

The following is an example of a job that defines two steps.

```yaml
name: my workflow
on:
  push:
    branches:
      - main
jobs:
  my job:
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3
      - name: run a command
        run: echo "hello world"
```

## GH workflow conditions

You can define conditions for running a workflow. The conditions are defined in the on section of the workflow. The following is an example of a workflow that runs when a pull request is created.

```yaml
name: my workflow
on:
  pull_request:
    branches:
      - main
jobs:
  my job:
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3
      - name: run a command
        run: echo "hello world"
```

## GH workflow outputs

You can define outputs for a workflow. The outputs are defined in the jobs section of the workflow. The following is an example of a workflow that defines an output.

```yaml
name: my workflow
on:
  pull_request:
    branches:
      - main
jobs:
  my job:
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3
      - name: run a command
        run: echo "hello world"
    outputs:
      my output:
        value: hello world
```

## GH workflow secrets

You can define secrets for a workflow. The secrets are defined in the jobs section of the workflow. The following is an example of a workflow that defines a secret.

```yaml
name: my workflow
on:
  pull_request:
    branches:
      - main
jobs:
  my job:
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3
      - name: run a command
        env:
          MY_SECRET: {% raw %}${{ secrets.MY_SECRET }}{% endraw %}
        run: echo $MY_SECRET
```

## GH workflow events

You can define events for a workflow. The events are defined in the on section of the workflow. The following is an example of a workflow that defines an event.

```yaml
name: my workflow
on:
  pull_request:
    branches:
      - main    
  push:
    branches:
      - main
  release:
    types:
      - created
```

## GH workflow variables

You can define variables for a workflow. The variables are defined in the jobs section of the workflow. The following is an example of a workflow that defines a variable.

```yaml
name: workflow-variable-demo

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      GLOBAL_MESSAGE: "Hello from the job level"
      BUILD_ENV: "production"

    steps:
      - name: Show job-level variables
        run: |
          echo "GLOBAL_MESSAGE = $GLOBAL_MESSAGE"
          echo "BUILD_ENV = $BUILD_ENV"

      - name: Define step-level variables
        env:
          STEP_MESSAGE: "Hello from the step level"
        run: |
          echo "STEP_MESSAGE = $STEP_MESSAGE"

      - name: Use vars context (requires pre-defined repo/organization variable)
        run: |
          echo "Repository Variable = {% raw %}${{ vars.MY_REPO_VARIABLE }} {% endraw %}"
```

## GH selfhost runners

```bash
useradd runner -m -d /home/runner -s /bin/bash
su runner 
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.325.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.325.0/actions-runner-linux-x64-2.325.0.tar.gz
echo "5020da7139d85c776059f351e0de8fdec753affc9c558e892472d43ebeb518f4  actions-runner-linux-x64-2.325.0.tar.gz" | shasum -a 256 -c
tar xzf ./actions-runner-linux-x64-2.325.0.tar.gz
./config.sh --url https://github.com/$GITHUB_USERNAME/$REPO --token {% raw %}${{ secrets.PERSONAL_ACCESS_TOKEN }}{% endraw %}
./run.sh &
```

Create a systemd service unit file

```bash
chmod +x /home/runner/run.sh
sudo nano /etc/systemd/system/github-runner.service
```

```ini
[Unit]
Description=GitHub Self-Hosted Runner
After=network.target

[Service]
Type=simple
User=runner
WorkingDirectory=/home/runner
ExecStart=/home/runner/run.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable github-runner
sudo systemctl start github-runner
```

# Use this YAML in your workflow file for each job
runs-on: self-hosted

## Examples by Use Case
### Lint and Test Node.js App
```yaml
name: Node Lint & Test
on: push
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npm test
```

### Deploy to GitHub Pages
```yaml
name: Deploy Pages
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: {% raw %}${{ secrets.GITHUB_TOKEN }}{% endraw %}
          publish_dir: ./public
```

### Cache Dependencies 
```yaml
- name: Cache Node Modules
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: {% raw %}${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}{% endraw %}
    restore-keys: |
      {% raw %}${{ runner.os }}-node-{% endraw %}
```

###  Python Test Example with Matrix
```yaml
name: Python Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.8, 3.9, 3.10]
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python {% raw %}${{ matrix.python-version }}{% endraw %}
        uses: actions/setup-python@v4
        with:
          python-version: {% raw %}${{ matrix.python-version }}{% endraw %}
      - run: pip install -r requirements.txt
      - run: pytest
```

### Reusable Workflows
Define a reusable workflow:
```yaml
# .github/workflows/deploy.yml
on:
  workflow_call:
    inputs:
      env:
        required: true
        type: string
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to {% raw %}${{ inputs.env }}{% endraw %}"
```
Call it from another workflow:
```yaml
jobs:
  call-deploy:
    uses: ./.github/workflows/deploy.yml
    with:
      env: production
```
### Docker Image Push with Secrets
Handling Secrets

1. Go to repo Settings → Secrets → Actions
2. Add new secret (e.g., AWS_ACCESS_KEY)
3. Use in workflow:
env:
  AWS_ACCESS_KEY_ID: {% raw %}${{ secrets.AWS_ACCESS_KEY }}{% endraw %}

```yaml
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Login to Docker Hub
        run: echo "{% raw %}${{ secrets.DOCKER_PASSWORD }}" | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin {% endraw %}
      - name: Build & Push
        run: |
          docker build -t myimage:latest .
          docker tag myimage:latest myrepo/myimage:latest
          docker push myrepo/myimage:latest
```

### Matrix Strategy for Multiple Versions
```yaml
strategy:
  matrix:
    node: [14, 16, 18]

steps:
  - uses: actions/setup-node@v3
    with:
      node-version: {% raw %}${{ matrix.node }} {% endraw %}
```

### Trigger Other Workflows
```yaml
name: Trigger Other Workflow
on:
  push:
    branches: [main]

jobs:
  trigger:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger downstream
        uses: benc-uk/workflow-dispatch@v1
        with:
          workflow: test.yml
          token: {% raw %}${{ secrets.PERSONAL_ACCESS_TOKEN }} {% endraw %}
```
## Useful Resources

* [GitHub Actions Docs](https://docs.github.com/en/actions){:target="_blank"}
* [GitHub Actions Marketplace](https://github.com/marketplace?type=actions){:target="_blank"}
* [YAML Validator](https://www.yamllint.com){:target="_blank"}
* [Awesome GitHub Actions](https://github.com/sdras/awesome-actions){:target="_blank"}
