# Git Repositories
Documentation to setup and manage git repositories used with Config Sync to deploy GCP resources.

SSC is using Azure Devops Repositories (AzDO Repos) and Pipelines as its git solution.

SSC implements a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.
As illustrated in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator requires an Infra repo and a ConfigSync repo.

## Create Deployment Repo
Deployment repos are created the same way, from cloning [gcp-repo-template](https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git).  These steps will need to be repeated for each repo name.  For example, `gcp-sandbox-tier1-infra`, `gcp-tier1-configsync`, etc.

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
1. Stage, commit and push the local changes to your new repo.  **You will need to push to main.**
    ```bash
    git add .
    git commit -m 'initializing repo from template'
    git push --set-upstream origin main
    ```
1. The repo is now created! It's now time to protect the main branch.

### Branch Protection
It's recommended to protect the `main` branch and use Pull Requests (PR) for any changes to the repos.

These settings can be set at the AzDO Project level.

At the very least, the [Require a minimum number of reviewers](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops&tabs=browser#require_reviewers) branch policy should be set to 2.

To do so:
1. Navigate to **Project Settings > Repos/Repositories > {repo} > Policies > Branch Policies > main**
1. Toggle on **Require a minimum number of reviewers**, the *Minimum number of reviewers* will default to 2.
    - You can also enable *When new changes are pushed:* > *Reset all approval votes*