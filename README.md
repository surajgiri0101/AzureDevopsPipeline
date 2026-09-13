# AzureDevopsPipeline

# Integrate Playwright with Azure DevOps Pipeline

There are two ways to set up Playwright with Azure DevOps (ADO):

- **Option 1** – Using a YAML file
- **Option 2** – Without using a YAML file (Classic Editor)

Let's walk through both.

---

## Option 1: Using YAML File

### Step 1: Create a new project
Create a new project in ADO, then click on the **Project** tab.

![Create project](image)

### Step 2: Create a new repository
Go to **Repos** and click **New repository**.

![New repository](image)

### Step 3: Name the repository
Enter a repository name and click **Create**.

![Create repository](image)

### Step 4: Clone the repository
Click **Clone**, copy the URL, and clone the repo to your local system.

### Step 5: Add your Playwright project files
Add all the Playwright framework folders inside the cloned repository.

### Step 6: Push to Azure DevOps
Commit and push all the folders to the ADO repository.

![Push folders](image)

### Step 7: Create a pipeline
Repository is ready — go to **Pipelines → Create Pipeline**.

### Step 8: Select Azure Repos Git
Click **Azure Repos Git**.

### Step 9: Select your repository
Choose the repository you just created.

### Step 10: Select "Starter Pipeline"

### Step 11: Add the pipeline YAML
Paste the following into `azure-pipelines.yml`:

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '18'
  displayName: 'Install Node.js'
- script: npm ci
  displayName: 'npm ci'
- script: npx playwright install --with-deps
  displayName: 'Install Playwright browsers'
- script: npx playwright test
  displayName: 'Run Playwright tests'
  env:
    CI: 'true'
```

> **Using a self-hosted agent?** Replace the `pool` section with:
> ```yaml
> pool:
>   name: AgentPoolName
>   demands:
>   - agent.name -equals AgentName
> ```

### Step 12: Save and run
Click **Save and run**.

### Step 13: Job queued
You'll see the job queued.

### Step 14: Verify build status
Click on the job to check its status.

### Step 15: Publish the Playwright report
Update `azure-pipelines.yml` to also publish test results and the HTML report:

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '18'
  displayName: 'Install Node.js'
- script: npm ci
  displayName: 'npm ci'
- script: npx playwright install --with-deps
  displayName: 'Install Playwright browsers'
- script: npx playwright test
  displayName: 'Run Playwright tests'
  env:
    CI: 'true'

- task: PublishTestResults@2
  displayName: 'Publish test results'
  inputs:
    searchFolder: 'test-results'
    testResultsFormat: 'JUnit'
    testResultsFiles: 'e2e-junit-results.xml'
    mergeTestResults: true
    failTaskOnFailedTests: true
    testRunTitle: 'My End-To-End Tests'
  condition: succeededOrFailed()

- task: PublishPipelineArtifact@1
  inputs:
    targetPath: playwright-report
    artifact: playwright-report
    publishLocation: 'pipeline'
  condition: succeededOrFailed()
```

### Step 16: Verify the report
From the job page, navigate to the **Artifacts** folder, download the `playwright-report`, and verify the results.

---

## Option 2: Without Using a YAML File (Classic Editor)

### Step 1
Repeat Steps 1–6 from Option 1.

### Step 2: Create a new pipeline
Go to **Pipelines → New Pipeline**.

### Step 3: Use the classic editor
Click **Use the classic editor**, then **Continue**.

### Step 4: Start with an empty job
Click **Empty job**.

### Step 5: Add Node.js installer task
Click **+**, search for **Node**, and add the **Node.js tool installer** task.

### Step 6: Configure Node version
Set the Node version to **v16** (Playwright supports Node v14 and above).

### Step 7: Install Playwright & dependencies
Click **+**, add a **Command line** task:
- **Display name:** `Install Playwright & Dependencies`
- **Script:**
  ```bash
  npm install && npx playwright install
  ```

Under **Advanced**, click the info icon and select **Link** to enable the working directory setting for this task.

### Step 8: Add npm task to run tests
Click **+**, search for **npm**, and add the **npm** task:
- **Display name:** e.g. `Run Playwright Tests`
- **Command:** `custom`
- **Command and arguments:** `run tests`

This task refers to the script defined in your `package.json`.

### Step 9: Save and run
Click **Save & queue**, add a commit message, then **Save and run**.

### Step 10: Check the job
Click on the job to view the run.

### Step 11: Publish the Playwright report
Add a **Publish Pipeline Artifacts** task to upload the `playwright-report` folder.

### Step 12: Publish test results
Add a **Publish Test Results** task.

Run the pipeline — artifacts and test results will now be published together.

---

## Summary

| Approach | Best for |
|---|---|
| **Option 1 (YAML)** | Version-controlled, repeatable pipelines; recommended for most teams |
| **Option 2 (Classic Editor)** | Quick setup without touching YAML, good for getting started |

Both options result in Playwright tests running in Azure DevOps with test results and HTML reports published as pipeline artifacts.
