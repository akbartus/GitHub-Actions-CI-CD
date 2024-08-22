# GitHub Actions and CI/CD

This repository provides information the basics of GitHub Actions as well as CI/CD pipeline.

## GitHub Actions
### Definitions
CI/CD - Continuous Integration and Continuous Delivery

Automating all build, test and deployment tasks. 

IaC - Infrastructure as Code
Managing infrastructure and provisioning through code, rather than manually. 

### Github Actions

GitHub Actions are scripts used to automate tasks in a software development workflow. 
Scripts are written in ".yml" format  

Three types of GitHub Actions:
* container actions - environment is part of such actions code. Only run on Linux.
* JavaScript actions - environment code is not included in such actions. Environment needs to be specified to run these actions. Supports Linux, MacOS, Windows environments.
* Composite actions - combining multiple actions as a single action


Container Action example | action.yml
```yml
name: "Hello Actions"
description: "Greet someone"
author: "octocat@github.com"

inputs:
    MY_NAME:
      description: "Who to greet"
      required: true
      default: "World"

runs:
    uses: "docker"
    image: "Dockerfile"

branding:
    icon: "mic"
    color: "purple"
```

### GitHub Actions Workflow 
It is a process to automate software development lifecycle tasks.

To create workflow:
1. Create .yml file in "<b>.github/workflows</b>" director


workflow.yml
```yml
name: A workflow for my Hello World file
on: push # trigger, can be also [push, pull_request]
jobs: # at least one job should be present
  build:
    name: Hello world action
    runs-on: ubuntu-latest # runner
    steps: # job also has steps with actions need to be completed
    - uses: actions/checkout@v1 # check's out repository
    - uses: ./action-a # custom action is in this directory
      with:
        MY_NAME: "Mona" # sets MY_NAME value in container action 
```

Workflow must have at least one job. Each job should have a runner (server) and steps need to be completed.
It is also possible to define in yml custom actions.

```
├── .github
│   └── workflows
│       └── workflow.yml
└── action-a
    ├── action.yml # custom action
    └── ... (other files for the action)
```

Runners can be of 2 types:
1. GitHub hosted
2. Self-hosted.

Example of self-hosted runner
```yml
name: Build and Test on Self-Hosted ARM32 Runner

on: push

jobs:
  build:
    name: Build on ARM32 Runner
    runs-on: [self-hosted, linux, ARM32] 
    #self-hosted is written, OS system used, OS architecture

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16'

    - name: Install dependencies
      run: npm install

    - name: Run tests
      run: npm test

```

### Components of GitHub Actions Workflow

Components are the following:

* A workflow - an automated process added to repository.
* Job - a section of the workflow associated with runner. It is first component of within the workflow.
* Steps - individual task that runs commands in a job.
* Actions - standalone commands that are executed.


### Configuring Workflows to run at scheduled time
Cron syntax is used for this.

```
* * * * * command_to_run
- - - - -
| | | | |
| | | | +---- Day of the week (0 - 7) (Sunday is both 0 and 7)
| | | +------ Month (1 - 12)
| | +-------- Day of the month (1 - 31)
| +---------- Hour (0 - 23)
+------------ Minute (0 - 59)

1. Minute
2. Hour
3. Month Day
4. Month
5. Week day
```

For example the following workflow will run every 15 minutes. */15 - every 15.

```yml
on:
  schedule:
    - cron:  '*/15 * * * *'
```

This means 01:00 every day:
```yml
on:
  schedule:
    - cron:  '0 1 * * *'
```

For ease it is possible to use crontab.guru

### Configuring Workflows to run for manual events
It is possible to trigger workflow manually using "workflow dispatch" event. It can be done by clicking manually on "Run workflow" button

```yml
on:
  workflow_dispatch: # to run manually
    inputs:
      logLevel:
        description: 'Log level'     
        required: true
        default: 'warning'
      tags:
        description: 'Test scenario tags'
```

It is also possible to run workflow by sending request from external source by using "repository_dispatch" event. 

```yml
on:
  repository_dispatch: #
    types: [opened, deleted]
```


```cURL
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"on-demand-test","client_payload":{"unit":false,"integration":true}}'
```
```js
// Octokit.js
// https://github.com/octokit/core.js#readme
const octokit = new Octokit({
  auth: 'YOUR-TOKEN'
})

await octokit.request('POST /repos/{owner}/{repo}/dispatches', {
  owner: 'OWNER',
  repo: 'REPO',
  event_type: 'on-demand-test',
  client_payload: {
    unit: false,
    integration: true
  },
  headers: {
    'X-GitHub-Api-Version': '2022-11-28'
  }
})
```

### Sample GitHub Workflows

Simple workflow examples: 

Create file in the following folder: 
.github/workflows/helloworld.yml

```yml
name: Hello World

on:
  push:
    branches:
      - main

jobs:
  hello_world:
    runs-on: ubuntu-latest
    steps:
      - name: Echo current time
        run: echo "The current server time is $(date)"


```

Create file in the following folder:

.github/workflows/scheduledworkflow.yml
```yml

name: Scheduled Workflow 

on:
  schedule:
    - cron: '*/2 * * * *'

jobs:
  hellow_world:
    runs-on: ubuntu-latest
    steps:
      - name: Echo current time
        run: echo "The vurrent server time is ${date}"
```

It is also possible to specify multiple events in 
a workflow. In such as case any of them will trigger the workflow. If they occur at the same time, multiple workflows will ru. 

Multiple events workflow example:

First create workflow file on "main" branch. And see what happens after any push event.

.github/workflows/multi.yml

```yml
name: CI on Multiple Events
on: 
  push:
    branches:
      - main
      - dev
  pull_request:
    branches:
      -main
  
jobs:
  multiple_events:
    runs-on: ubuntu-latest
    steps:
      - name: "Echo basic information"
      run: |
        echo "REF: $GITHUB_REF"
        echo "Job ID: $GITHUB_JOB"
        echo "Action: $GITHUB_ACTION"
        echo "Actor: $GITHUB_ACTOR"


```

Create dev branch. Switch there. Create new pull request. Make it from Main < Dev. And then go to main branch and see there under actions.

### Manual Events: Triggering Workflow through Github CLI

To run workflow manually, the workflow must be configured to run on the <b>workflow_dispatch</b>. It can contain up to 10 inputs.

For reference check: https://cli.github.com/manual/gh_workflow_run


#### Installing Github CLI
1. Install github CLI: https://cli.github.com/ 
2. Open Command Line tool. 
3. Type: gh auth login and follow instructions to authorize connection to your account.

Use this:
Add this workflow to the main branch:

.github/workflows/manual.yml


```yml
name: Manual workflow

on:
  workflow_dispatch:
    # Inputs the workflow accepts.
    inputs:
      name:
        description: 'Person to greet'
        # Default value if no value is explicitly provided
        default: 'World'
        # Input has to be provided for the workflow to run
        required: true
        # The data type of the input
        type: string

jobs:
  # This workflow contains a single job called "greet"
  greet:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Runs a single command using the runners shell
    - name: Send greeting
      run: echo "Hello ${{ inputs.name }}"
```

In the command line tool write if no input:

<b>gh run manual.yml</b>

With input:

<b>gh workflow run manual.yml -f name=Bobbi</b>



### Manual Events: Triggering Workflow through Webhook

Webhook is a public facing URL than can be sent an http request to trigger events from external sources.

It will trigger only if workflow is on default branch:

.github/workflows/webhook.yml


```yml
name: "Webhook Event Example"
on:
  repository_dispatch:
    types:
      - webhook

jobs:
  respond-to-dispatch:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v2
      - name: Run a Script
        run: echo "Event of type $GITHUB_EVENT_NAME"

```

Also, personal access toke needs to be created (fine grained one). Then enter the following curl:

```cURL
curl -L -X POST -H "Accept: application/vnd.github+json" -H "Authorization: Bearer <personaltoken>" -H "X-GitHub-Api-Version: 2022-11-28" https://api.github.com/repos/author/reponame/dispatches -d "{\"event_type\":\"webhook\",\"client_payload\":{\"key\":\"value\"}}"
```

### Conditionals

jobs.job_id.if conditional can be used to prevent job from running unless a condition is met. For example:

```yml
jobs:
  respond-to-dispatch:
    if: github.repository == 'akbartus/reponame'
    runs-on: ubuntu-latest
```

Use example with expression syntax.

```yml
jobs:
  sample:
    if: ${{ ! startsWith(github.ref, 'refs/tags/')}}
    runs-on: ubuntu-latest
```

Expression syntax allows to use github actions in-built functions.



Example workflow with conditional:

.github/workflows/conditionalsample.yml

```yml
name: Conditional Sample
on: [push]
jobs: 
  do-something:
    if: github.repository == 'akbartus/github-actions-test'
    runs-on: ubuntu-latest
    steps:
      - name: "Hello World"
        run: echo "Hello World!"
  do-another-thing:
    runs-on: ubuntu-latest
    steps:
      - name: "Hello Second Time"
        run: echo "Hello second time!"
```


### Expressions

For further reference: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/evaluate-expressions-in-workflows-and-actions

GitHub has various string expressions. 

<b>Functions:</b>
* contains (search, item) - contains('Hello', 'llo')
* startsWith(searchString, searchValue) - startsWith('Hello World', 'He')
* endsWith(searchString, searchValue) - endsWith('Hello World', 'ld')
* toJSON(value) - toJSON(job)
* fromJSON(value) - fromJSON(job)
* format( string, replaceValue0, replaceValue1, ..., replaceValueN) - format('User {0} is working on repository {1}', env.USERNAME, env.REPOSITORY)

<b>Status Check Functions:</b>
* success()
* always()
* cancelled()
* failure()


Example workflow to show expressions:

.github/workflows/expressions.yml

```yml
name: Expressions
on:
  push:
    branches:
      - main
    
jobs: 
  expressions:
    runs-on: ubuntu-latest
    steps:
      - name: "Check if string contains substring"
        if: contains('This is my sample text', 'text')
        run: echo "This string contains substring"
      - name: "Format and echo string"
        run: echo ${{ format('Hello {0} {1} {2}', 'my', 'friend', 'Tom')}} 
```



### Runners more in detail

Runner determines the underlying compute and OS on which workflow will work.

Runners can be:
* Github-hosted: ubuntu-latest, windows-latest, macos-latest, ubuntu-22.04, etc.
* Self-hosted: external compute connected to GitHub.

```yml
runs-on: windows-latest

runs-on: [ubuntu-latest, macos-14]

runs-on: self-hosted
```

### GitHub hosted runner

Another example showing windows runner, on which installation of node.js occurs and runs app.js file. Note that hello.js file is created but not commited to repo. :

.github/workflows/nodeinstall.yml

```yml
name: Node.js CI

on:
  push:
    branches: [ main ]

jobs:
  build-node:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Show version
        run: node --version

      - name: Install dependencies
        run: npm install

      - name: Run app.js
        run: node app.js

      - name: Run Hello World
        run: |
          echo "console.log('Hello World');" > hello.js
          node hello.js

```


### GitHub self-hosted runners

It is possible to create self-hosted runners. 
To be able to do so, under settings:

Actions > Runners > New Self-Hosted Runner and follow the instructions

### Workflow Commands

Some examples of workflow commands are below. 

First step shows group message command. Second and third steps show setting environment variable.

.github/workflows/workflow-commands.yml

```yml
name: Worflow Commands
on: ['push']
jobs: 
  my-job:
    runs-on: ubuntu-latest
    steps: 
      # First step
      - name: Group Logging
        # multiline |
        run: |
          echo "::group:: My Group Message"
          echo "Msg 1"
          echo "Msg 2"
          echo "::endgroup::"
      # Second and Third steps
      - name: Step 1
        run: |
          echo "MY_VAL = hello" >> $GITHUB_ENV
      - name: Step 2
        run: |
          echo $MY_VAL
          
```

### Workflow Context

Contexts are a way to access informaton about workflow. Each context is an object, which can be string or another object. 

For more information: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs 

For example

```yml

env: 
  MY_SECRET: ${{ secrets.MY_SECRET }}
```

There are the following contexts in GitHub:
* github - information about the workflow run
* env - contains variables set in a workflow, job or step
* vars - contains variables set at the repository, organization or environment levels
* job - information about currently running job
* steps - information about steps which are running in the current job
* runner - information about the runner that is running current job
* secrets - contains the names and values of secrets that are available to a workflow run
* strategy - information about matrix execution strategy for current job
* matrix - contains matrix properties defined in the workflow that apply to current job
* needs - contains outputs of all jobs.
* inputs - contains inputs of a reusable or manually triggered workflow.


See below an example:

Take a special note of env because it sets environment variable, which is used inside of a step.

.github/workflows/contexts.yml


```yml
name: Context Examples
on: ['push']
jobs: 
  my-context:
    runs-on: ubuntu-latest
    steps: 
      
      - name: My Step
        run:  echo "Hello $MY_ACTION $MY_ACTOR"
        env:
          MY_ACTION: ${{ github.action }}
          MY_ACTOR: ${{ github.actor }}

          
```

### Dependent Jobs

A workflow can have several jobs, which run in parallel. Sometimes we want to define dependency on other jobs like so:
* jobs.<job_id>.needs
* jobs.<job_id>.if

This will let us do jobs in a certain sequence.

The example below will first do job1 and then moves to job2:

.github/workflows/job-dependency.yml

```yml
name: Job Dependency Example

on: ['push']

jobs: 
  job2:
    runs-on: ubuntu-latest
    # it tells that it needs job1
    needs: job1
    steps: 
      - name: StepA
        run: echo "World"

  job1:
    runs-on: ubuntu-latest
    steps:
      - name: StepB
        run: echo "Hello"
```

### Encrypted Secrets

Encrypted secrets are variables that allow you to pass sensitive information to github actions workflow.

Secrets are accessed through "secrets" context:
${{secrets.MY_SECRET}}

Secrets have 3 levels:
1. Organizaional level secret
2. Repository level secret
3. Environment level secret

Secrets must contain only alphanumeric character and underscore. Other characters are not allowed.

Secrets can be passed as inputs or environmental vairables. For example:

```yml
name: Example Using Secret as Input

on: [push]

jobs: 
  use-secret-as-input:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Custom Action using Secret
        uses: example/action@v1
        with:
          api-key: ${{ secrets.API_KEY }}
```

```yml
name: Using Secret as Environmental Variable
on: ['push']
jobs: 
  my-context:
    runs-on: ubuntu-latest
    steps: 
      
      - name: My Step
        run:  echo "Using API Key $API_KEY"
        env:
          API_KEY: ${{ secrets.API_KEY }}
         
```

Secrets ca be set using GitHub CLI:

* At repository level:
```sh
# sets from prompt 
gh secret set SECRET_NAME
# sets from txt file
gh secret set SECRET_NAME < secret.txt
```

* At environment level:
```sh
# sets from prompt 
gh secret set --env ENV_NAME SECRET_NAME
# list all secrets for env
gh secret list --env ENV_NAME
```

* At organizational level.


### Configuration Variables

Configuration variables allow to put non-sensitive information to workflow and uses "vars" context. For example:
${{ vars.APP_ID_EXAMPLE}}

It has same levels as secrets.

* At repository level:
```sh
# sets from prompt 
gh variable set MYVARIABLE
# sets from txt file
gh variable set MYVARIABLE < secret.txt
```

* At environment level:
```sh
# sets from prompt 
gh variable set  MYVARIABLE --env ENV_NAME
# list all variables for env
gh variable list --env ENV_NAME
```

### Default Environment Variables

These types of environment variables are set by GitHub. 

* CI
* GITHUB_ACTION
* GITHUB_ACTOR

etc.

### Setting Env Vars

Custom env vars can be set to workflow level, job level and step level.

It is also possible to set env vars dynamically during execution of aworkflow using: <b>$GITHUB_ENV</b>. 

For example:

.github/workflows/setenvvar.yml

```yml
name: Set Env Variables
on: [push]
jobs:
  setup-and-use:
    runs-on: ubuntu-latest
    steps:
      - name: Set Dynamic Env Var
        run: | 
          echo "DYNAMIC_VAR=Hello from Github Actions" >> $GITHUB_ENV
      - name: Use the Env Var
        run: | 
          echo "The value of DYNAMIC_VAR is: $DYNAMIC_VAR" 

```

### Adding Script to Workflow

It is possible to execute bash screepts in GitHub Actions workflow. For example:

.github/workflows/scriptrun.yml

```yml
name: Run Bash Script
on: [push]
jobs: 
  example-job: 
    runs-on: ubuntu-latest
    # this means base directory
    defaults:
      run: 
        # use scripts directory in the root folder
        working-directory: ./scripts
    steps:
      - name: Check out the repository runner
        uses: actions/checkout@v4

      - name: Make scripts executable
        run: chmod +x ./my-script.sh ./my-another-script.sh
          
      - name: Run as script
        run: ./my-script.sh

      - name: Run antother script
        run: ./my-another-script.sh
```

