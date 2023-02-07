# Git Repositories
Documentation to setup and manage git repositories used with Config Sync to deploy GCP resources.

SSC is using Azure Devops Repositories (AzDO Repos) and Pipelines as its git solution.

SSC implements a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.
As illustrated in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator requires an Infra repo and a ConfigSync repo.

## Create New Deployment Repo
Deployment repos are initially created the same way, from cloning [gcp-repo-template](https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git).  These steps will need to be repeated for each repo name.  For example, `gcp-sandbox-tier1-infra`, `gcp-tier1-configsync`, etc.

The git credentials will need to be appropriately set for your AzDO org.

> If your AzDO org has project-wide branch policies set on repositories, you may need to work with branches / Pull Requests or temporarily give yourself permission to "Bypass policies when pushing" on the repo.

1. Create a new **empty** repository (no README.md) to hold the configs in [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/repos/git/create-new-repo?view=azure-devops).  Once created, copy the HTTPS URL.  It will be required for the next step.

1. Update and export the variables below:
    ```bash
    export NEW_REPO_NAME='<your new repo name>'
    export NEW_REPO_URL='<your new repo URL>'
    ```

1. Clone the [gcp-repo-template](https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git) in a folder named like your new repo:

    ```bash
    git clone https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git ${NEW_REPO_NAME}
    ```
1. Move into the new folder corresponding to the new repo:
    ```bash
    cd ${NEW_REPO_NAME}
    ```
1. Change the repo's remote:
    ```bash
    # Remove the origin pointing to the template repo
    git remote remove origin
    # Add the new origin with your repo url
    git remote add origin ${NEW_REPO_URL}
    ```
1. Confirm the new remote configuration, it should only list your new repo URL for fetch and push:
    ```bash
    git remote --verbose
    ```
1. Edit `modversions.yaml` to pin the `tools` submodule to a specific [release tag](https://github.com/ssc-spc-ccoe-cei/gcp-tools/releases) or commit SHA.
1. Clone and checkout the proper version of the tools sub module by running:
    ```bash
    bash modupdate.sh
    ```
1. Remove pipelines directories which are not required:
    - If the repo is hosted in Azure DevOps:
        ```bash
        # remove the '.github' pipelines directory
        rm --recursive '.github'
        ```
    - If the repo is hosted in GitHub:
        ```bash
        # remove the '.azure-pipelines' pipelines directory
        rm --recursive '.azure-pipelines'
        ```
1. For **`Infra`** repos, remove the environment sub-directories which are not required. **Never delete '.gitkeep' files in folders that remain.**
    - If the repo is for sandbox. For example, `gcp-sandbox-tier1-infra`:
        ```bash
        # remove the 'dev', 'uat' and 'prod' sub-directories
        for env_subdir in dev uat prod; do
            rm --recursive "deploy/${env_subdir}/"
            rm --recursive "source-customization/${env_subdir}/"
        done
        ```
    - If the repo is for dev, uat and prod. For example, `gcp-tier1-infra`:
        ```bash
        # remove the 'sandbox' sub-directory
        for env_subdir in sandbox; do
            rm --recursive "deploy/${env_subdir}/"
            rm --recursive "source-customization/${env_subdir}/"
        done
        ```
1. Review your local changes then stage, commit and push them to your new repo.  **You will need to push to main.**
    ```bash
    git add .
    git commit -m 'initializing repo from template'
    git push --set-upstream origin main
    ```
1. The repo is created! It's now time to protect the main branch.

### Add Branch Protection
It's recommended to protect the `main` branch and use Pull Requests (PR) for any changes to the repos.

These settings could also be set at the AzDO Project level.

At the very least, the [Require a minimum number of reviewers](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops&tabs=browser#require_reviewers) branch policy should be set to 2.

To do so:
1. Navigate to **Project Settings > Repos/Repositories > {repo} > Policies > Branch Policies > main**
1. Toggle on **Require a minimum number of reviewers**, the *Minimum number of reviewers* will default to 2.
    - You can also enable *When new changes are pushed:* > *Reset all approval votes*

These other policies can also be enabled as needed:
- **Check for linked work items**: *Required*
- **Check for comment resolution**: *Required*
- **Limit merge types**: only allow *Squash merge*

### Extra Settings for `ConfigSync` Repos
TODO:...

### Add Pipelines
The repo is now created and the main branch is protected.  [Pipelines](./Pipelines.md) can be created.

- All deployment repos should have the "configs-validation" pipeline.  TODO: update doc when it exists

- To use semantic versioning during deployment operations, the `Infra` repos can be setup with a git tagging pipeline, such as [version-tagging](https://github.com/ssc-spc-ccoe-cei/gcp-tools/tree/main/pipeline-samples/version-tagging).

## Synchronizing / Promoting Configs
TODO: place this in proper markdown file

Changes to `infra` repos will only be applied to GCP when the `configsync` repo is updated.  The concept is similar for all repos, this example will focus on `gcp-tier1-infra` and `gcp-tier1-configsync`.

1. Change configs in `gcp-tier1-infra`.
    > **!!! It's important to add the `source-customization` for each environment.  This will ensure all environments are rendered, validated and tagged at the same time. !!!**
1. Once the PR is merged, note the new tag version or commit SHA.
1. At this point the changes have not been deployed to GCP. Change configs in `gcp-tier1-configsync` to do so for each environment.
1. `dev`:
    - Set `version:` in `source-customization/dev/tier1-root-sync/setters-version.yaml` to the new tag or commit SHA noted earlier.
    - Hydrate the repo and create a PR.
    - Once the PR is merged the config sync operator will pick up the updated configs in `gcp-tier1-infra/deploy/dev`.
    - Validate the `dev` environment in GCP.  Proceed to `uat` if successful, restart the process if not.
1. `uat`:
    - Set `version:` in `source-customization/uat/tier1-root-sync/setters-version.yaml` to the same value as `dev`.
    - Hydrate the repo and create a PR.
    - Once the PR is merged the config sync operator will pick up the updated configs in `gcp-tier1-infra/deploy/uat`.
    - Validate the `uat` environment in GCP.  Proceed to `prod` if successful, restart the process if not.
1. `prod`:
    - Set `version:` in `source-customization/prod/tier1-root-sync/setters-version.yaml` to the same value as `uat`.
    - Hydrate the repo and create a PR.
    - Once the PR is merged the config sync operator will pick up the updated configs in `gcp-tier1-infra/deploy/prod`.
    - Validate the `prod` environment in GCP.  Restart the process if not successful.