# GitHub Actions Practice Q&A

Legend:
- ✅ = correct option, ❌ = incorrect option. The explanation sits directly under each option.
- ~~Strikethrough~~ = the practice test's answer key or wording was wrong, followed by the corrected answer.
- Doc base: https://docs.github.com/en/actions

---

## Question 1
As an engineer, you are trying to leverage an organization secret in a repo. The value received in the workflow is not the same as that set in the secret. What is the most likely reason for the difference?

- ❌ The Encrypt Secret setting was not configured for the secret.
  - No such setting exists. Secrets are always encrypted and encryption never changes the value.
- ❌ There is a different value specified at the enterprise level.
  - Actions secrets are defined at organization, repository and environment level, not enterprise level.
- ❌ There is a different value specified at the workflow level.
  - Workflows can't define secrets. They only reference them with `${{ secrets.NAME }}`.
- ❌ The Codespace secret doesn't match the expected value.
  - Codespaces secrets are a separate store that Actions doesn't read.
- ✅ **There is a different value specified at the repo level.**
  - If the same secret name exists at several levels, the lowest level wins: environment, then repository, then organization. A same-named repo secret overrides the org secret.

Docs: [Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)

---

## Question 2
What can be used to set a failed status of an action from its code?

- ❌ composite run step
  - A composite action groups steps. It doesn't set a failed status by itself.
- ❌ output variable
  - Outputs pass data between steps and have no effect on status.
- ❌ JavaScript dist/ folder
  - Holds the compiled, bundled code and is unrelated to status.
- ✅ **a non-zero exit code**
  - GitHub treats a non-zero exit code from the action's process as a failure. In JavaScript actions, `core.setFailed()` logs the error and exits with code 1.
- ❌ @actions/github toolkit
  - An Octokit client and context helper for the API. The failure helper is in `@actions/core`.

Docs: [Setting exit codes for actions](https://docs.github.com/en/actions/creating-actions/setting-exit-codes-for-actions)

---

## Question 3
You are working on developing reusable workflows for your organization. What keyword should be included as part of the reusable workflow event triggers?

- ❌ workflow_run
  - Fires when another workflow run is requested or completes. It doesn't make a workflow callable.
- ❌ pull_request
  - Fires on pull request activity.
- ❌ check_run
  - Fires on check run activity.
- ✅ **workflow_call**
  - A reusable workflow must declare `on: workflow_call` so other workflows can call it with `uses:`.
  - ~~The key's explanation says it is triggered by the `workflow_run` event.~~ `workflow_call` is its own trigger and has nothing to do with `workflow_run`.

```yaml
on:
  workflow_call:
    inputs:
      env_name:
        type: string
        required: true
```

Docs: [Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

---

## Question 4
How can you send warning annotations that appear in GitHub Actions workflow runs for JavaScript Actions?

- ❌ `core.notice('...')`
  - ~~The key says this function does not exist.~~ It does exist and creates a *notice* annotation. It is wrong here because the question asks for a *warning*.
- ❌ `core.warn('...')`
  - This function doesn't exist in `@actions/core`.
- ❌ `core.info('...')`
  - Writes a plain log line with no annotation.
- ✅ **`core.warning('...')`**
  - Emits a `::warning::` workflow command, which shows as a warning annotation on the run summary.

Docs: [Setting a warning message](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-a-warning-message)

---

## Question 5
Which run: command will set a step's output?

- ✅ **`run: echo 'OUTPUT=Test' >> $GITHUB_OUTPUT`**
  - Appending `name=value` to the `$GITHUB_OUTPUT` file sets an output, readable as `steps.<id>.outputs.OUTPUT`.
- ❌ `run: echo '::set-env name=OUTPUT::Text'`
  - Deprecated and disabled for security reasons. It also sets an environment variable, not an output.
- ❌ `run: export OUTPUT=Text`
  - Only affects that shell process.
- ❌ `run: echo ${{ $GITHUB_OUTPUT=Text }}`
  - Invalid syntax. Expressions can't assign values.

Docs: [Setting an output parameter](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-output-parameter)

---

## Question 6
Which three scopes are available to define custom environment variables within a workflow file?

- ❌ all jobs on a single Actions runner, by using `runner.env` at the top of the workflow file
  - `runner.env` is not a workflow keyword. The `runner` context is read-only.
- ~~❌~~ ✅ **the contents of a job within a workflow, by using `jobs.<job_id>.env`**
  - ~~The key marked this wrong.~~ This is a valid scope. The variables are available to every step in that job.
- ~~✅~~ ❌ within the run attribute of a job step
  - ~~The key marked this correct.~~ Setting a variable inside a script, or via `$GITHUB_ENV`, is not a scope in the workflow file.
- ✅ **the entire workflow, by using `env` at the top level of the workflow file**
  - Available to all jobs and steps.
- ~~❌~~ ✅ **a specific step within a job, by using `jobs.<job_id>.steps[*].env`**
  - ~~The key marked this wrong.~~ This is the narrowest scope, limited to that step.
- ❌ the entire stage, by using `env` at the top of the defined build stage
  - Actions has no stages, only workflows, jobs and steps.

Correct answers: workflow `env`, job `env` and step `env`.

Docs: [Defining environment variables](https://docs.github.com/en/actions/learn-github-actions/variables), [`env` syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#env)

---

## Question 7
You need an action that executes quickly on all hosted runners without pulling large images, is easy to publish once and reuse across many repos, and can accept inputs, set outputs and interact with the GitHub API. Which action type fits best?

- ❌ Composite action because it can bundle Node modules for faster execution
  - Composite actions are YAML step lists. They don't bundle Node modules.
- ❌ Composite action because it allows YAML-only definitions and reuses other actions
  - Valid and portable, but it has no native toolkit or API integration. It only chains steps and actions.
- ✅ **JavaScript action because it executes on the runner with Node.js and ships via a single repository release**
  - It runs directly on the runner with no image pull, works on Linux, Windows and macOS, and can use `@actions/core` and `@actions/github`.
- ❌ Docker action because it isolates the environment and reduces dependency management
  - Needs a Linux runner and a container build or pull, which is slower and not cross-platform.

Docs: [About custom actions](https://docs.github.com/en/actions/creating-actions/about-custom-actions)

---

## Question 8
Which two files are required for a Docker container action in addition to the source code?

- ✅ **Dockerfile**
  - Defines the image the action runs in.
- ❌ metadata.yml
  - Not a valid name. The metadata file must be `action.yml` or `action.yaml`.
- ✅ **action.yml**
  - Metadata with `runs.using: docker` and the `image` to run.
- ❌ Actionfile
  - Not a valid name.

Docs: [Creating a Docker container action](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action)

---

## Question 9
Which two workflow commands send information from the runner?

- ❌ populating variables in a Dockerfile
  - A Docker build concern, not a workflow command.
- ✅ **setting output parameters**
  - The runner processes `$GITHUB_OUTPUT` and reports the values to the workflow for later steps and jobs.
- ✅ **setting a debug message**
  - `::debug::` sends a message from the runner to the logs.
- ❌ reading from environment variables
  - Local input to a step. Nothing is sent from the runner.

Docs: [Workflow commands](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions)

---

## Question 10
Disabling a workflow stops it from being triggered without deleting the file. Which two scenarios is temporarily disabling a workflow most useful for?

- ❌ A workflow is configured to run on self-hosted runners
  - Runner management is separate. Pause or fix the runner instead.
- ✅ **A workflow sends requests to a service that is down.**
  - Disabling prevents failing runs and pointless requests until the service recovers.
- ❌ A runner needs to have diagnostic logging enabled.
  - Logging is configured on the runner and is unrelated to the workflow.
- ❌ A workflow needs to be changed from running on a schedule to a manual trigger
  - Edit the `on:` block. Disabling doesn't change triggers.
- ✅ **A workflow error produces too many, or wrong, requests, impacting external services negatively.**
  - Disabling stops the damage immediately while you fix it.

Docs: [Disabling and enabling a workflow](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/disabling-and-enabling-a-workflow)

---

## Question 11
Which statement is true about using default environment variables?

- ❌ The environment variables can be read in workflows using the `ENV: variable_name` syntax.
  - Wrong syntax. Use `${{ env.NAME }}` in expressions or `$NAME` in a shell.
- ✅ **The `GITHUB_WORKSPACE` environment variable should be used to access files from within the runner.**
  - ~~The key marked this wrong.~~ It is the default working directory on the runner, and where `actions/checkout` puts the repo.
- ❌ The environment variables can be set in the `defaults:` sections of the workflow
  - `defaults` only sets the default `run` shell and working directory. Use `env:`.
- ~~✅~~ ❌ The environment variables created should be prefixed with `GITHUB_` to ensure they can be accessed in workflows
  - ~~The key marked this correct.~~ The `GITHUB_` prefix is reserved. You can't set custom variables or secrets starting with it.

Docs: [Default environment variables](https://docs.github.com/en/actions/learn-github-actions/variables#default-environment-variables)

---

## Question 12
You are using a Docker container action in your workflow. What is required for the action to run successfully?

- ❌ The job env must be set to a Linux environment.
  - `env` holds variables. The OS is chosen with `runs-on`.
- ❌ The action must be published to the GitHub Marketplace.
  - Not required. Actions can come from any accessible repo, or from a local path.
- ✅ **The job runs-on must specify a Linux machine with Docker installed.**
  - Docker container actions run only on Linux. GitHub-hosted Ubuntu runners include Docker. Self-hosted ones must install it.
- ❌ The referenced action must be hosted on Docker Hub.
  - The image can be built from a Dockerfile in the action repo or pulled from any registry.

Docs: [About custom actions, Docker container actions](https://docs.github.com/en/actions/creating-actions/about-custom-actions#docker-container-actions)

---

## Question 13
Your self-hosted runner sometimes loses connection while running jobs. How should you troubleshoot the issue?

- ❌ Locate the self-hosted runner in your repository's settings page and download its log archive
  - ~~The key says this is the more reliable source.~~ The settings page shows status only and has no runner log archive.
- ❌ Set the DEBUG environment variable to true before starting the runner
  - The runner doesn't use a `DEBUG` variable.
- ✅ **Access the self-hosted runner's installation directory and look for log files in the _diag folder.**
  - ~~The key's explanation says this may not be as useful.~~ It is the documented place for the runner logs (`Runner_*.log` and `Worker_*.log`), which record connectivity problems.
- ❌ Start the runner with the --debug flag
  - The runner has no such flag.

Docs: [Monitoring and troubleshooting self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/monitoring-and-troubleshooting-self-hosted-runners#reviewing-the-self-hosted-runner-application-log-files)

---

## Question 14
You have a 10-MB data set required in a specific workflow. Which steps should you perform so the dataset is stored encrypted and can be decrypted during the workflow? (Choose three.)

- ✅ **Encrypt the dataset.**
  - For example with GPG.
- ✅ **Commit the encrypted dataset to the same repository as the workflow**
  - The workflow can then check it out and decrypt it at runtime.
- ~~✅~~ ❌ Leverage the actions/download-secret action in the workflow
  - ~~The key marked this correct.~~ No such action exists.
- ❌ Store the dataset in a GitHub encrypted secret
  - Secrets are limited to 48 KB, so 10 MB won't fit.
- ❌ Compress the dataset
  - Compression doesn't encrypt, and the file would still exceed 48 KB.
- ~~❌~~ ✅ **Store the encryption keys in a GitHub encrypted secret**
  - ~~The key marked this wrong.~~ The passphrase or key must be stored as a secret so the workflow can decrypt the file.

Correct answers: encrypt the dataset, commit the encrypted dataset, store the key as a secret.

Docs: [Storing large secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#storing-large-secrets)

---

## Question 15
Which syntax correctly accesses a job output (output1) of an upstream job (job1) from a dependent job?

- ❌ `${{needs.job1.output1}}`
  - Missing `outputs`.
- ❌ `${{depends.job1.output1}}`
  - `depends` is not a context, and `outputs` is missing.
- ❌ `${{job1.outputs.output1}}`
  - Not a context. Job outputs are reachable only through `needs`.
- ✅ **`${{needs.job1.outputs.output1}}`**
  - The dependent job lists the upstream job in `needs`, and outputs are exposed as `needs.<job_id>.outputs.<name>`.

Docs: [Defining outputs for jobs](https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs)

---

## Question 16
You're creating an action that runs a security scanner CLI that only ships as a Linux binary and depends on specific system libraries. You want an identical environment across runs and repos. Which action type is the best fit?

- ❌ Composite action because you can define `container:` at the job level and reuse steps
  - `container:` is a job-level setting, not a composite action feature. Composite actions use the runner's environment.
- ❌ Composite action because it can sequence multiple shell steps and call other actions
  - Depends on whatever the runner image contains, so it isn't reproducible.
- ❌ JavaScript action because it runs natively on the runner without container overhead
  - Runs with the runner's libraries, the opposite of isolation.
- ✅ **Docker action because you can package the CLI and its system dependencies in an image**
  - The image gives an identical environment on every run.

Docs: [About custom actions](https://docs.github.com/en/actions/creating-actions/about-custom-actions)

---

## Question 17
Which default environment variable specifies the branch or tag that triggered a workflow?

- ❌ GITHUB_BRANCH
  - Not a default variable.
- ❌ GITHUB_TAG
  - Not a default variable.
- ✅ **GITHUB_REF**
  - Holds the full ref, such as `refs/heads/main` or `refs/tags/v1.0`. `GITHUB_REF_NAME` is the short name and `GITHUB_REF_TYPE` is `branch` or `tag`.
- ❌ ENV_BRANCH
  - Not a default variable.

Docs: [Default environment variables](https://docs.github.com/en/actions/learn-github-actions/variables#default-environment-variables)

---

## Question 18
You are reaching your organization's storage limit for GitHub artifacts and packages. What two options should you do to prevent the storage limit from being reached?

- ❌ Use self-hosted runners for all workflow runs.
  - Artifacts are still uploaded to GitHub storage.
- ❌ Disable branch protections in the repository.
  - No effect on storage.
- ✅ **Delete artifacts from the repositories manually**
  - Frees space immediately.
- ✅ **Configure the artifact and log retention period.**
  - Shorter retention makes artifacts and logs expire automatically.
- ❌ Configure the repo to use Git Large File Storage.
  - Git LFS has its own quota and doesn't reduce artifact or package storage.

Docs: [Configuring the retention period](https://docs.github.com/en/organizations/managing-organization-settings/configuring-the-retention-period-for-github-actions-artifacts-and-logs-in-your-organization), [Removing workflow artifacts](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/removing-workflow-artifacts)

---

## Question 19
Which command will set the `$ABC` environment variable within a script so it can be used in subsequent job steps?

- ❌ `run: echo '::set-env name=ABC::bar'`
  - Deprecated and disabled.
- ✅ **`run: echo 'ABC=bar' >> $GITHUB_ENV`**
  - The runner reads `$GITHUB_ENV` after the step and exports each line to later steps in the job.
- ❌ `run: echo ${{ $ABC=bar }}`
  - Invalid expression syntax.
- ❌ `export ABC=bar`
  - Only affects the current step's shell.

Docs: [Setting an environment variable](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-environment-variable)

---

## Question 20
What is the right method to ensure users approve a workflow before the next step proceeds?

- ❌ granting users workflow approval permissions
  - No such permission. Approvals are configured on environments.
- ❌ creating a branch protection rule and only allow certain users access
  - Governs merges and pushes, not job gating.
- ❌ granting users repository approval permissions
  - No such permission.
- ✅ **adding users as required reviewers for an environment**
  - A job targeting that environment pauses until a reviewer approves.

Docs: [Using environments for deployment, required reviewers](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#required-reviewers)

---

## Question 21
Which action type should be used to bundle a series of run steps into a reusable custom action?

- ❌ Bash script action
  - Not an action type.
- ❌ Docker container action
  - Runs a container, not a bundle of steps.
- ✅ **Composite action**
  - `runs.using: composite` packages multiple `run` and `uses` steps as one action.
- ❌ JavaScript action
  - Runs Node code.

Docs: [Creating a composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

---

## Question 22
What is the smallest scope for an environment variable?

- ❌ workflow env mapping
  - Widest scope.
- ❌ a job
  - Applies to all steps in the job.
- ❌ workflow settings
  - Not a variable scope.
- ✅ **a step**
  - Step-level `env` applies to that step only.

Docs: [`jobs.<job_id>.steps[*].env`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsenv)

---

## Question 23
How can you identify a Docker container action on GitHub?

- ✅ **The action.yml metadata file has the runs.using value set to Docker.**
  - `runs.using: 'docker'` is what defines the action type.
- ❌ The action.yml metadata file references a Dockerfile file.
  - Part of many Docker actions, but not the identifier. `image` can also be a registry image like `docker://alpine:3`.
- ❌ The action's repository name includes the keyword 'Docker.'
  - Names have no effect.
- ❌ The action's repository includes @actions/core in the root directory.
  - That is a JavaScript toolkit.

```yaml
runs:
  using: 'docker'
  image: 'Dockerfile'
```

Docs: [Metadata syntax, runs for Docker container actions](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#runs-for-docker-container-actions)

---

## Question 24
As a developer, you need to create a custom action written in Python. Which action type should you choose?

- ❌ Python Action
  - ~~The key says these are designed for Python.~~ There is no Python action type. The types are JavaScript, Docker container and composite.
- ❌ JavaScript action
  - Requires Node code.
- ❌ Docker Container Action
  - Works, but needs a Linux runner and an image build. The test's intended answer is composite.
- ✅ **Composite run step**
  - A composite action runs Python via `run` steps (`shell: python`, after `actions/setup-python`). It works on any OS.

Docs: [About custom actions, types of actions](https://docs.github.com/en/actions/creating-actions/about-custom-actions#types-of-actions)

---

## Question 25
Your workflow reuses downloaded dependencies between runs. Using the GitHub cache action, which two input parameters are required?

- ❌ restore-keys
  - ~~The key says it isn't a valid input.~~ It is a valid but *optional* input for fallback key prefixes.
- ✅ **key: the key created when saving a cache and the key used to search for a cache**
  - Required.
- ✅ **path: the file path on the runner to cache or restore**
  - Required.
- ❌ dependency: the name and version of a package to cache or restore
  - Not an input.
- ❌ ref: the ref name of the branch to access and restore a cache created
  - Not an input.
- ❌ cache-hit
  - An *output* that tells you whether an exact match was found, not an input.

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
```

Docs: [Caching dependencies](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows), [actions/cache](https://github.com/actions/cache)
