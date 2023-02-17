# Git Repositories
Documentation to setup and manage git repositories used with Config Sync to deploy GCP resources.

SSC is using Azure Devops Repositories (AzDO Repos) and Pipelines as its git solution.

SSC implements a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.
As illustrated in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator requires an Infra repo and a ConfigSync repo.

## Create New Deployment Repo
Deployment repos are initially created the same way, from cloning [gcp-repo-template](https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git).  These steps will need to be repeated for each repo name.  For example, `gcp-experimentation-tier1-infra`, `gcp-tier1-configsync`, etc.

The git credentials will need to be appropriately set for your AzDO org.

### 1. Build the Repo

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
1. Run the following to get the proper version of the tools submodule:
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
    - If the repo is for experimentation. For example, `gcp-experimentation-tier1-infra`:
        ```bash
        # remove the 'dev', 'uat' and 'prod' sub-directories
        for env_subdir in dev uat prod; do
            rm --recursive "deploy/${env_subdir}/"
            rm --recursive "source-customization/${env_subdir}/"
        done
        ```
    - If the repo is for dev, uat and prod. For example, `gcp-tier1-infra`:
        ```bash
        # remove the 'experimentation' sub-directory
        for env_subdir in experimentation; do
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

### 2. Add Branch Protection
It's recommended to protect the `main` branch and use pull requests for any changes to the repos.

These settings could also be set at the AzDO Project level.

At the very least, the [Require a minimum number of reviewers](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops&tabs=browser#require_reviewers) branch policy should be enabled on the `main` branch. By doing so, the branch cannot be deleted and changes must be made via pull request.

To enable the policy:
1. Navigate to **Project Settings > Repos/Repositories > {repo} > Policies > Branch Policies > main**
1. Toggle on **Require a minimum number of reviewers**, the *Minimum number of reviewers* will default to 2.
    - You can also enable *When new changes are pushed:* > *Reset all approval votes*

These other policies can also be enabled as needed:
- **Check for linked work items**: *Required*
- **Check for comment resolution**: *Required*
- **Limit merge types**: only allow *Squash merge*

> TODO: Extra policy for tier2 "configsync" repos only. "Automatically included reviewers" with path filter on setters-version.yaml

### 3. Verify Service Account Permissions
An AzDO service account should be used for authenticating Config Sync.  It requires read access to the repo.

Depending on your organization, this could be set at different levels and with groups.  These steps will assume the service account has already been configured.

To confirm:
1. Navigate to **Project Settings > Repos/Repositories > {repo} > Security**.
1. Find and click on the appropriate service account user or group.
1. Confirm the **Read** permission is set to **Allow**.  All other permissions should be "Not set" or "Deny".

A [personal access token (PAT)](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows) with scope of **Code (Read)** will also need to be created for the service account.  This should only need to be done once per service account.  **Note the expiration date, it will need to be periodically re-generated.**

### 4. Add Pipelines
The repo is now created and the main branch is protected.  [Pipelines](./Pipelines.md) can be created.

- All deployment repos should have the "configs-validation" pipeline.  TODO: update doc when it exists

- To use semantic versioning during deployment operations, the `Infra` repos can be setup with a git tagging pipeline, such as [version-tagging](https://github.com/ssc-spc-ccoe-cei/gcp-tools/tree/main/pipeline-samples/version-tagging).

## Update Deployment Repo
This section is for updating the deployment repo itself, not the YAML configs.

As with any other change, they should be done through a PR process.

### Update from Template
The [gcp-repo-template](https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git) remains fairly static, but it can sometimes contain important changes.

Follow these steps to update your deployment repo with changes from `gcp-repo-template`:

TODO: test and add steps

### Update `tools` Submodule
The tools submodule can easily be updated to a new version.

Follow these steps to 
1. Clone the deployment repo and checkout a new branch.
1. Edit `modversions.yaml` to pin the `tools` submodule to a new [release tag](https://github.com/ssc-spc-ccoe-cei/gcp-tools/releases) or commit SHA.
1. Run the following to get the proper version of the tools submodule:
    ```bash
    bash modupdate.sh
    ```
1. If your repo contains pipelines that were created from `tools/pipeline-samples`, compare the YAML files to see if changes are required.
1. Stage, commit and push your branch to create a PR.