# Git Repositories

- [Git Repositories](#git-repositories)
  - [Create New Deployment Monorepo](#create-new-deployment-monorepo)
    - [1. Build the Monorepo](#1-build-the-monorepo)
    - [2. Add Branch Policies](#2-add-branch-policies)
    - [3. Verify Service Account Permissions](#3-verify-service-account-permissions)
    - [4. Add Pipelines](#4-add-pipelines)
  - [Update Deployment Repo](#update-deployment-repo)
    - [Update from Template](#update-from-template)
    - [Update `tools` Submodule](#update-tools-submodule)

--------------------------------------

Documentation to setup and manage git monorepos used with Config Sync to deploy GCP resources.

SSC is using Azure Devops Repositories (AzDO Repos) and Pipelines as its git solution.

SSC implements a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.
As illustrated in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator observes several folders from the [monorepos](https://monorepo.tools/).

## Create New Deployment Monorepo

Deployment monorepos are initially created the same way, from cloning a monorepo template.  These steps will need to be repeated for each monorepo name.  For example, `gcp-env-tier1`, `gcp-<client-name>-tier2` or `gcp-<x-project-id>-tier34`.

The git credentials will need to be appropriately set for your AzDO org.

It's important to note that **`.gitkeep` files must never be deleted.**

### 1. Build the Monorepo

> If your AzDO org has project-wide branch policies set on repositories, you may need to work with branches / Pull Requests or temporarily give yourself permission to "Bypass policies when pushing" on the monorepo.

1. Create a new **empty** repository (no README.md) to hold the configs in [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/repos/git/create-new-repo?view=azure-devops).  Once created, copy the HTTPS URL.  It will be required for the next step.

1. Update and export the variables below:

    ```shell
    export NEW_REPO_NAME='<your new monorepo name>'
    export NEW_REPO_URL='<your new monorepo URL>'
    ```

1. Clone the template in a folder named like your new monorepo by running one of the commands below:

    - For tier 1:

        ```shell
        git clone https://github.com/ssc-spc-ccoe-cei/gcp-tier1-template.git ${NEW_REPO_NAME}
        ```

    - For tier 2:

        ```shell
        git clone https://github.com/ssc-spc-ccoe-cei/gcp-tier2-template.git ${NEW_REPO_NAME}
        ```

    - For tier 3/4:

        ```shell
        git clone https://github.com/ssc-spc-ccoe-cei/gcp-tier34-template.git ${NEW_REPO_NAME}
        ```

1. Move into the new folder corresponding to the new monorepo:

    ```shell
    cd ${NEW_REPO_NAME}
    ```

1. Change the monorepo's remote:

    ```shell
    # Remove the origin pointing to the template monorepo
    git remote remove origin
    # Add the new origin with your monorepo url
    git remote add origin ${NEW_REPO_URL}
    ```

1. Confirm the new remote configuration, it should only list your new monorepo URL for fetch and push:

    ```shell
    git remote --verbose
    ```

1. Edit `modversions.yaml` to pin the `tools` submodule to a specific [release tag](https://github.com/ssc-spc-ccoe-cei/gcp-tools/releases) or commit SHA.
1. Run the following to pull the proper version of the tools submodule:

    ```shell
    bash modupdate.sh
    ```

1. Remove pipelines directories which are not required:
    - If the monorepo is hosted in Azure DevOps:

        ```shell
        # remove the '.github' pipelines directory
        rm --recursive '.github'
        ```

    - If the monorepo is hosted in GitHub:

        ```shell
        # remove the '.azure-pipelines' pipelines directory
        rm --recursive '.azure-pipelines'
        ```

1. For tier1 repos, remove the environment sub-directories which are not required.
    - If the monorepo is `gcp-experimentation-tier1`:

        ```shell
        # find and remove the 'dev', 'preprod' and 'prod' sub-directories
        for env_to_rm in dev preprod prod; do
          for dir_to_rm in $(find . -type d -name "${env_to_rm}" | sort); do
            echo "removing: ${dir_to_rm}"
            rm --recursive "${dir_to_rm}"
          done
        done
        ```

    - If the monorepo is `gcp-env-tier1`:

        ```shell
        # find and remove the 'experimentation' sub-directories
        for env_to_rm in experimentation; do
          for dir_to_rm in $(find . -type d -name "${env_to_rm}" | sort); do
            echo "removing: ${dir_to_rm}"
            rm --recursive "${dir_to_rm}"
          done
        done
        ```

1. If the repo is not used to deploy native Kubernetes resources, remove the `kubernetes` sub-directories:
    ```shell
    # find and remove the 'kubernetes' sub-directories
    for env_to_rm in kubernetes; do
        for dir_to_rm in $(find . -type d -name "${env_to_rm}" | sort); do
        echo "removing: ${dir_to_rm}"
        rm --recursive "${dir_to_rm}"
        done
    done
    ```

1. Customize the file `csync/tier<N>/configcontroller/source-customization/<env>/**/<root|repo>-sync-git/setters.yaml` for **all** environments.

1. Generate hydrated files, follow step 3 of [Changing.md](./Changing.md#step-3---hydrate).

1. Review your local changes then stage, commit and push them to your new monorepo.  **You will need to push to main.**

    ```shell
    git add .
    git commit -m 'initializing monorepo from template'
    git push --set-upstream origin main
    ```

1. The monorepo is created! It's now time to protect the main branch.

### 2. Add Branch Policies

It's recommended to protect the `main` branch and use pull requests for any changes to the monorepos.

These settings could also be set at the AzDO Project level.

At the very least, the [Require a minimum number of reviewers](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops&tabs=browser#require_reviewers) branch policy should be enabled on the `main` branch. By doing so, the branch cannot be deleted and changes must be made via pull request.

To enable the policy on a specific repo:

1. Navigate to **Project Settings > Repos/Repositories > {repo} > Policies > Branch Policies > main**
1. Toggle on **Require a minimum number of reviewers**, the *Minimum number of reviewers* will default to 2.
    - You can also enable *When new changes are pushed:* > *Reset all approval votes*

These other policies can also be enabled as needed:

- **Check for linked work items**: *Required*
- **Check for comment resolution**: *Required*
- **Limit merge types**: only allow *Squash merge*

> TODO: Extra policy for "Automatically included reviewers" with path filters.

### 3. Verify Service Account Permissions

An AzDO service account should be used for authenticating Config Sync.  It requires read access to the monorepo.

Depending on your organization, this could be set at different levels and with groups.  These steps assume the AzDO service account has already been configured.

To confirm:

1. Navigate to **Project Settings > Repos/Repositories > {repo} > Security**.
1. Find and click on the appropriate service account user or group.
1. Confirm the **Read** permission is set to **Allow**.  All other permissions should be "Not set" or "Deny".

> *Note: A [personal access token (PAT)](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows) with scope of **Code (Read)** will also need to be created for the service account.  This should only need to be done once per service account.  **Note the expiration date, it will need to be periodically re-generated.***

### 4. Add Pipelines

The monorepo is now created and the main branch is protected.  [Pipelines](./Pipelines.md) can be created.

1. **All deployment monorepos should have the [`validate-yaml`](https://github.com/ssc-spc-ccoe-cei/gcp-tools/tree/main/pipeline-samples/validate-yaml) pipeline.**

- Optional (for future use): to use semantic versioning during deployment operations, the monorepos can be setup with a git tagging pipeline, such as [version-tagging](https://github.com/ssc-spc-ccoe-cei/gcp-tools/tree/main/pipeline-samples/version-tagging).

## Update Deployment Repo

This section is for updating the deployment monorepo itself, not the YAML configs.

As with any other change, they should be done through a PR process.

### Update from Template

The templates remain fairly static, but they can sometimes contain important changes.

Follow these steps to update your deployment monorepo with changes from the templates:

- [gcp-tier1-template](https://github.com/ssc-spc-ccoe-cei/gcp-tier1-template)
- [gcp-tier2-template](https://github.com/ssc-spc-ccoe-cei/gcp-tier2-template)
- [gcp-tier34-template](https://github.com/ssc-spc-ccoe-cei/gcp-tier34-template)

TODO: test and add steps

### Update `tools` Submodule

The tools submodule can easily be updated to a new version.

Follow these steps to

1. Clone the deployment monorepo and checkout a new branch.
1. Edit `modversions.yaml` to pin the `tools` submodule to a new [release tag](https://github.com/ssc-spc-ccoe-cei/gcp-tools/releases) or commit SHA.
1. Run the following to get the proper version of the tools submodule:

    ```shell
    bash modupdate.sh
    ```

1. If your monorepo contains pipelines that were created from `tools/pipeline-samples`, compare the YAML files to see if changes are required.
1. Stage, commit and push your branch to create a PR.
