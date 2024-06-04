---
# DO NOT TOUCH — Managed by doc writer
ContentId: 891072bb-c46d-4392-800a-84d747072ce3
DateApproved: 05/02/2024

# Summarize the whole topic in less than 300 characters for SEO purpose
MetaDescription: Use Continuous Integration for testing Visual Studio Code extensions (plug-ins).
---

## Automated publishing

You can also configure the CI to publish a new version of the extension automatically.

The publish command is similar to publishing from a local environment using [`vsce`](https://github.com/microsoft/vscode-vsce), but you must somehow provide the Personal Access Token (PAT) in a secure way. By storing the PAT as a `VSCE_PAT` **secret variable**, `vsce` will be able to use it. Secret variables are never exposed, so they are safe to use in a CI pipeline.

## Azure Pipelines

<a href="https://azure.microsoft.com/services/devops/"><img alt="Azure Pipelines" src="/assets/api/working-with-extensions/continuous-integration/pipelines-logo.png" width="318" /></a>

[Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) is great for running VS Code extension tests as it supports running the tests on Windows, macOS, and Linux. For Open Source projects, you get unlimited minutes and 10 free parallel jobs. This section explains how to set up an Azure Pipelines for running your extension tests.

First, create a free account on [Azure DevOps](https://azure.microsoft.com/services/devops/) and create an [Azure DevOps project](https://azure.microsoft.com/features/devops-projects/) for your extension.

Then, add the following `azure-pipelines.yml` file to the root of your extension's repository. Other than the `xvfb` setup script for Linux that is necessary to run VS Code in headless Linux CI machines, the definition is straight-forward:

```yaml
trigger:
  branches:
    include:
    - main
\
pool:
  vmImage: $(imageName)

steps:


- bash: |
    echo ">>> Compile vscode-test"
    yarn && yarn compile
    echo ">>> Compiled vscode-test"
    cd sample
    echo ">>> Run sample integration test"
    yarn && yarn compile && yarn test
  displayName: Run Tests
  env:

```

Finally, [create a new pipeline](https://learn.microsoft.com/azure/devops/pipelines/create-first-pipeline) in your DevOps project and point it to the `azure-pipelines.yml` file. Trigger a build and voilà:

![pipelines](images/continuous-integration/pipelines.png)

You can enable the build to run continuously when pushing to a branch and even on pull requests. See [Build pipeline triggers](https://learn.microsoft.com/azure/devops/pipelines/build/triggers) to learn more.

### Azure Pipelines automated publishing

1. Set up `VSCE_PAT` as a secret variable using the [Azure DevOps secrets instructions](https://learn.microsoft.com/azure/devops/pipelines/process/variables?tabs=classic%2Cbatch#secret-variables).
2. Install `vsce` as a `devDependencies` (`npm install @vscode/vsce --save-dev` or `yarn add @vscode/vsce --dev`).
3. Declare a `deploy` script in `package.json` without the PAT (by default, `vsce` will use the `VSCE_PAT` environment variable as the Personal Access Token).

```json
"scripts": {
  "deploy": "vsce publish --yarn"
}
```

4
``

The [condition](https://learn.microsoft.com/azure/devops/pipelines/process/conditions) property tells the CI to run the publish step only in certain cases.

In our example, the condition has three checks:

- `succeeded()` - Publish only if the tests pass.
- `startsWith(variables['Build.SourceBranch'], 'refs/tags/')` - Publish only if a tagged (release) build.
- `eq(variables['Agent.OS'], 'Linux')` - Include if your build runs on multiple agents (Windows, Linux, etc.). If not, remove that part of the condition.

Since `VSCE_PAT` is a secret variable, it is not immediately usable as an environment variable. Thus, we need to explicitly map the environment variable `VSCE_PAT` to the secret variable.

## GitHub Actions

You can also configure GitHub Actions to run your extension CI. In headless Linux CI machines `xvfb` is required to run VS Code, so if Linux is the current OS run the tests in an Xvfb enabled environment:


        node-version: 18.x
    - run: npm install
    - run: xvfb-run -a npm test
      if: runner.os == 'Linux'
    - run: npm test
      if: runner.os != 'Linux'
```

### GitHub Actions automated publishing

1. Set up `VSCE_PAT` as an encrypted secret using the [GitHub Actions secrets instructions](https://docs.github.com/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository).
2. Install `vsce` as a `devDependencies` (`npm install @vscode/vsce --save-dev` or `yarn add @vscode/vsce --dev`).
3. Declare a `deploy` script in `package.json` without the PAT.

```json
"scripts": {
  "deploy": "vsce publish --yarn"
}
```

4. Configure the CI so the build will also run when tags are created:

5. Add a `publish` job to the pipeline that calls `npm run deploy` with the secret variable.

```yaml
- name: Publish
  if:
The [if](https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idif) property tells the CI to run the publish step only in certain cases.

In our example, the condition has three checks:

- `success()` - Publish only if the tests pass.
- `startsWith(github.ref, 'refs/tags/')` - Publish only if a tagged (release) build.
- `matrix.os == 'ubuntu-latest'` - Include if your build runs on multiple agents (Windows, Linux, etc.). If not, remove that part of the condition.

## GitLab CI

GitLab CI can be used to test and publish the extension in headless Docker containers. This can be done by pulling a preconfigured Docker image, or installing `xvfb` and the libraries required to run Visual Studio Code during the pipeline.
```

### GitLab CI automated publishing

1. Set up `VSCE_PAT` as a masked variable using the [GitLab CI documentation](https://docs.gitlab.com/ee/ci/variables/README.html#mask-a-cicd-variable).
2. Install `vsce` as a `devDependencies` (`npm install @vscode/vsce --save-dev` or `yarn add @vscode/vsce --dev`).
3. Declare a `deploy` script in `package.json` without the PAT.

```json
"scripts": {
  "deploy": "vsce publish --yarn"
}
```

4. Add a `deploy` job that calls `npm run deploy` with the masked variable which will only trigger on tags.

```yaml
deploy:
  on

## Common questions

### Do I need to use Yarn for continuous integration?

All of the above examples refer to a hypothetical project built with [Yarn](https://yarnpkg.com/), but can be adapted to use [npm](https://www.npmjs.com/), [Grunt](https://gruntjs.com/), [Gulp](https://gulpjs.com/), or any other JavaScript build tool.
