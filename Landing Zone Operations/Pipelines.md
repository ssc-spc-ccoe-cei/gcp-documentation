# Pipelines

- [Pipelines](#pipelines)
  - [Azure DevOps YAML Pipelines](#azure-devops-yaml-pipelines)
    - [Add Pipeline](#add-pipeline)
    - [Add PR Trigger](#add-pr-trigger)
    - [Delete Pipeline](#delete-pipeline)
  - [GitHub Actions Workflows](#github-actions-workflows)

--------------------------------------

Documentation to manage and configure pipelines.

This documentation assumes that proper YAML files are already committed to the repo. Samples can be found in the [gcp-tools](https://github.com/ssc-spc-ccoe-cei/gcp-tools/tree/main/pipeline-samples/) repo.

## Azure DevOps YAML Pipelines

The [Azure DevOps pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops) need to be manually created using existing YAML definition files located in your repo.  The files could be located in any directory, we'll use the `.azure-pipelines/` directory as a convention.

### Add Pipeline

Repeat the following steps in Azure DevOps for each YAML pipeline definition files:

Navigate to **Pipelines > Pipelines**:

1. Click the **New pipeline** button.
1. Select **Azure Repos Git (YAML)**.
1. Select your repo.
1. Select **Existing Azure Pipelines YAML file**.
1. Select the appropriate file in the **Path** dropdown and click **Continue**.
1. Review the YAML file, click the down arrow next to **Run** and click **Save**.
1. The pipeline is now added!
However, its name defaults to the repo's name which is not ideal if the repo contains multiple pipelines and/or the project contains multiple repos.
Click the **"three dots" > Rename/move** to provide a meaningful name and optionally move to a folder.
For example, if your repo's name is `my-repo` and the pipeline YAML file is `firstpipeline.yaml`, the pipeline could be renamed to something like `my-repo__firstpipeline` in a `my-repo` folder.

> *During a pipeline’s first execution, it may be required to manually authorize it to use agent pools (depending on security settings) and/or access other resources (repos, variable groups, etc.).*

### Add PR Trigger

A pipeline may need to run when a Pull Request (PR) is created/updated. In Azure DevOps, this [trigger](https://learn.microsoft.com/en-us/azure/devops/pipelines/repos/azure-repos-git?view=azure-devops&tabs=yaml#pr-triggers) cannot be defined in the YAML definitions.  A [build validation policy](https://docs.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops&tabs=browser#build-validation) must be created to accomplish this.

To do so, for each repo / pipeline requiring a PR trigger:

Navigate to **Project Settings > Repos/Repositories > {repo} > Policies > Branch Policies > main**, click the **“+”** next to **Build Validation** and set the following options:

1. **Build Pipeline**: select the name of the pipeline to trigger on PRs
1. **Trigger**: *Automatic*
1. **Policy requirement**: *Required*
1. **Build expiration**: *Immediately*
1. **Display name**: optional name

> *Adding a branch policy will protect the branch.*

### Delete Pipeline

Navigate to **Pipelines > Pipelines**:

1. Find the pipeline to be deleted and click its **"three dots"** on the right side of the screen.
1. Click **Delete**.
1. Follow the confirmation prompt.

You can now delete the appropriate YAML file from your repo's `.azure-pipelines` directory.

## GitHub Actions Workflows

Properly formatted YAML file in `.github/workflows` should automatically be added.

Please refer to [GitHub's documentation](https://docs.github.com/en/actions/using-workflows/about-workflows).
