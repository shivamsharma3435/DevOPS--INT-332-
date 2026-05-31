### UNIT V: CONTINUOUS INTEGRATION WITH GITHUB ACTIONS
### WEEK 11: WORKFLOW AUTOMATION & TRIGGERS

## 1. UNDERSTANDING WORKFLOW AUTOMATION
GitHub Actions makes it easy to automate all your software workflows directly within GitHub. It allows you to build, test, and deploy your code right from the repository where it is hosted.
Continuous Integration (CI) is the practice of merging all developers' working copies to a shared mainline several times a day. Automated builds and tests run on every merge to verify the integration.

## 2. WORKFLOW DIRECTORY STRUCTURE
To use GitHub Actions, you must define workflows using YAML files.
These files MUST be placed in a specific, hidden directory at the root of your Git repository:
`.github/workflows/`
If you name a file `.github/workflows/ci.yml`, GitHub will automatically parse and execute it based on its triggers.

## 3. KEY COMPONENTS OF ACTIONS
- Workflows: A configurable automated process made up of one or more jobs.
- Events/Triggers: A specific activity in a repository that triggers a workflow run (e.g., a push, a pull request).
- Jobs: A set of steps that execute on the same runner. Jobs run in parallel by default.
- Steps: Individual tasks within a job. A step can either run a shell script or invoke an Action.
- Actions: Standalone commands that are combined into steps to create a job (e.g., `actions/checkout`).
- Runners: A server that runs your workflows when they're triggered. Each runner can run a single job at a time.

## 4. WORKFLOW TRIGGERS
Workflows are highly event-driven. Common triggers include:
- `push`: Runs when code is pushed to specific branches (e.g., `branches: [ "main" ]`).
- `pull_request`: Runs when a PR is opened, ensuring the incoming code passes tests before it can be merged.
- `schedule`: Uses Cron syntax to run workflows at specific times (e.g., nightly builds).
- `workflow_dispatch`: Creates a "Run workflow" button in the GitHub UI, allowing manual execution. You can also define custom input fields for manual runs.

============
PRACTICALS 
============

### Practical 1: Basic CI Workflow for Node.js
**Question/Scenario:**
Write a GitHub Actions YAML workflow named "Node CI" that triggers on pushes to the "main" branch. It should checkout the repository, set up Node.js 18, install dependencies, and run `npm test`.

**Solution / Code:**
```yaml
name: Node CI

on:
  push:
    branches: [ "main" ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm test
```

### Practical 2: Manual Workflow with Inputs (Workflow Dispatch)
**Question/Scenario:**
You need a deployment workflow that is triggered manually by developers. It should prompt the user for an "environment" string input (defaulting to 'staging') and then print "Deploying to [environment]".

**Solution / Code:**
```yaml
name: Manual Deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target Deployment Environment'
        required: true
        default: 'staging'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Execute Deployment
        run: echo "Deploying application to the ${{ github.event.inputs.environment }} environment!"
```
*Explanation:* The `workflow_dispatch` trigger activates the manual UI in GitHub. The inputs are accessed using the `github.event.inputs` context object.
### Windows Lab — GitHub Actions
Copy `labs/unit-5-github-actions/.github/workflows/java-ci.yml` to your repo and push.


=====================================================
UNIT V: CONTINUOUS INTEGRATION WITH GITHUB ACTIONS
(JOBS, STEPS, MATRIX STRATEGIES & CACHING)
=====================================================

1. JOBS & MATRIX STRATEGIES
A matrix strategy lets you use variables in a single job definition to automatically create multiple job runs that are based on the combinations of the variables.
For example, if you are building an open-source library, you want to ensure it works on Node 14, 16, and 18, and on both Ubuntu and Windows. A matrix strategy will spawn 6 parallel jobs automatically to test every combination.

2. STEPS & SHELL COMMANDS
Steps execute sequentially within a Job. Using the `run` keyword, you can execute native shell commands.
You can write multi-line shell scripts using the pipe `|` character:
```yaml
- run: |
    echo "Starting build process"
    make build
    make test
```

3. USING MARKETPLACE ACTIONS
You don't have to write everything from scratch. GitHub Marketplace contains thousands of pre-built actions.
- Language-specific setup: `actions/setup-python`, `actions/setup-java` configure the environment PATHs perfectly.
- Cloud Authentication: Actions like `aws-actions/configure-aws-credentials` safely inject IAM roles into the runner.
- Security Scanning: Actions for SonarQube or Snyk can scan your code for vulnerabilities automatically.

4. USING CACHING FOR FASTER BUILDS
Downloading internet dependencies (`node_modules`, `~/.m2`) on every run wastes minutes of compute time. The `actions/cache` action stores these folders between runs.
It uses a hash of your lockfile (like `package-lock.json`) as the cache key. If the lockfile hasn't changed, it restores the massive dependency folder from cache in seconds instead of re-downloading it.

5. MULTI-JOB WORKFLOWS
By default, all jobs in a workflow run simultaneously in parallel.
To create a pipeline (e.g., Build -> Test -> Deploy), you use the `needs` keyword.
```yaml
jobs:
  test:
    ...
  deploy:
    needs: test
    ...
```
The `deploy` job will not start until the `test` job successfully finishes.

=============
PRACTICALS
==============

### Practical 3: Implementing a Matrix Testing Strategy
**Question/Scenario:**
Write a workflow job that tests a Python application across three versions of Python (3.8, 3.9, 3.10) using the matrix strategy.

**Solution / Code:**
```yaml
jobs:
  matrix_test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.8", "3.9", "3.10"]
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install dependencies and Test
        run: |
          pip install pytest
          pytest tests/
```
*Explanation:* GitHub will dynamically spawn 3 independent runners. The `${{ matrix.python-version }}` expression injects the current iteration's value.

### Practical 4: Caching Maven Dependencies
**Question/Scenario:**
Your Maven build takes 5 minutes to download jars. Write a caching step that caches the `~/.m2/repository` directory. The cache key should be based on the runner's OS and a hash of the `pom.xml` file.

**Solution / Code:**
```yaml
steps:
  - uses: actions/checkout@v3
  
  - name: Cache Maven packages
    uses: actions/cache@v3
    with:
      path: ~/.m2/repository
      key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
      restore-keys: |
        ${{ runner.os }}-maven-
        
  - name: Build with Maven
    run: mvn -B package --file pom.xml
```
*Explanation:* `hashFiles('**/pom.xml')` generates a unique string based on the contents of the POM. If you add a new dependency, the POM hash changes, causing a cache miss, and downloading the new dependencies cleanly.
### Windows Lab — Matrix build
Add `strategy.matrix` with Java 17 and 21 in workflow YAML.