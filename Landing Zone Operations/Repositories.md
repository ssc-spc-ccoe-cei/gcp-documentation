
# Git Repositories

SSC is using Azure Devops Repositories (AzDO Repos) and Pipelines as its git solution.


## Create a Deployment Repo

Deployment repos are initially created the same way.  Templates are cloned from github.com and updated to reflect the current need before being deployed.

There are three repository templates that are available:

[Tier 1 Template](https://github.com/ssc-spc-ccoe-cei/gcp-tier1-template)
[Tier 2 Template](https://github.com/ssc-spc-ccoe-cei/gcp-tier2-template)
[Tier 3/4 Template](https://github.com/ssc-spc-ccoe-cei/gcp-tier34-template)

Git credentials will need to be set appropriately for your AzDO org.

### 1. Build the Repo

> **XXX** I'm not familiar with what this blurb means, so I don't know if it stays/goes - If your AzDO org has project-wide branch policies set on repositories, you may need to work with branches / Pull Requests or temporarily give yourself permission to "Bypass policies when pushing" on the repo.

1. In [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/repos/git/create-new-repo?view=azure-devops), create a new **empty** repository (i.e. uncheck the option to include README.md).  Once the repo has been created, copy its URL, as it will be required for the next step.

1. Update and export the variables below:

    ```shell
    export NEW_REPO_NAME='<new repo name>'
    export NEW_REPO_URL='<new repo URL>'
    ```
1. Copy the URL of the template that you'd like to copy:
    Tier 1 Template - https://github.com/ssc-spc-ccoe-cei/gcp-tier1-template
    Tier 2 Template - https://github.com/ssc-spc-ccoe-cei/gcp-tier2-template
    Tier 3/4 Template - https://github.com/ssc-spc-ccoe-cei/gcp-tier34-template
    
1. Enter the following command, but ensure you add the appropriate URL first:

    ```shell
    git clone https://<the URL of the appropriate template> ${NEW_REPO_NAME}
    ```
    It will copy the template repository into your newly-created repository.

1. Move into the new folder corresponding to the new repo:

    ```shell
    cd ${NEW_REPO_NAME}
    ```

1. Change the repo's remote:

    ```shell
    # Remove the origin pointing to the template repo
    git remote remove origin
    # Add the new origin with your repo url
    git remote add origin ${NEW_REPO_URL}
    ```

1. Confirm the new remote configuration, it should only list your new repo URL for fetch and push:

    ```shell
    git remote --verbose
    ```

1. Edit `modversions.yaml` so that the `tools` submodule will be the appropriate version.

1. Run the following to get the proper version of the tools submodule:

    ```shell
    bash modupdate.sh
    ```

1. The template allows for either AzDo or GitHub pipelines.  Remove the pipelines directories that are not required:
    - If the repo is hosted in Azure DevOps:

        ```shell
        # remove the '.github' pipelines directory
        rm --recursive '.github'
        ```

    - If the repo is hosted in GitHub:

        ```shell
        # remove the '.azure-pipelines' pipelines directory
        rm --recursive '.azure-pipelines'
        ```

1. **XXX** I don't think these removals are still necessary.  I think that we are keeping each of the environment folders in the repos.  **Remove the environment sub-directories which are not required.** **Never delete '.gitkeep' files in folders that remain.**
    - If the repo is for experimentation. For example, `gcp-experimentation-tier1-infra`:

        ```shell
        # remove the 'dev', 'preprod' and 'prod' sub-directories
        for env_subdir in dev preprod prod; do
            rm --recursive "deploy/${env_subdir}/"
            rm --recursive "source-customization/${env_subdir}/"
        done
        ```

    - If the repo is for dev, preprod and prod. For example, `gcp-tier1-infra`:

        ```shell
        # remove the 'experimentation' sub-directory
        for env_subdir in experimentation; do
            rm --recursive "deploy/${env_subdir}/"
            rm --recursive "source-customization/${env_subdir}/"
        done
        ```

1. Review your local changes then stage, commit and push them to your new repo.  **As this is a new repo, you will need to push to main.**

    ```shell
    git add --a
    git commit -m 'initializing repo from template'
    git push --set-upstream origin main
    ```

1. The repo is created! It's now time to protect the main branch.

### 2. Add Branch Protection

It's recommended to protect the `main` branch and use pull requests for any changes to the repos.

**Note:**  These settings are set at the AzDO Project level.

The branch policy should [require a minimum number of reviewers](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops&tabs=browser#require_reviewers).  This is enabled on the `main` branch. 

By doing so, the branch cannot be deleted and all changes must be made via pull request, thereby ensuring peer-review of any changes before deployment.

To enable this policy:

1. Navigate to **Project Settings > Repos/Repositories > {repo} > Policies > Branch Policies > main**
1. Toggle on **Require a minimum number of reviewers**.  Unless instructed otherwise, leave *Minimum number of reviewers* at its default of 2.
    - Enable *When new changes are pushed:* > *Reset all approval votes*

Enable the following policies:

- **Check for linked work items**: *Required*
- **Check for comment resolution**: *Required*
- **Limit merge types**: only allow *Squash merge*

**XXX**I don't understand this section.  Is it valid?  Ask Stéphane  > TODO: Extra policy for tier2 "configsync" repos only. "Automatically included reviewers" with path filter on setters-version.yaml

### 3. Verify Service Account Permissions

An AzDO service account is used for authenticating Config Sync.  It requires read access to the repo.

Depending on your organization, this could be set at different levels and with groups.  **Note: These steps will assume the service account has already been configured.**

To confirm:

1. Navigate to **Project Settings > Repos/Repositories > {repo} > Security**.
1. Find and click on the appropriate service account user or group. **XXX** Do we need to give more information for this step?
1. Confirm the **Read** permission is set to **Allow**.  All other permissions should be "Not set" or "Deny".

A [personal access token (PAT)](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows) with scope of **Code (Read)** will also need to be created for the service account.  This should only need to be done once per service account.  **Note the expiration date, it will need to be periodically re-generated.** **XXX**Do we need to explain how to create the PAT?

### 4. Add Pipelines

The repo is now created and has been pushed to AzDo/GitHub.  Additionally, the main branch is protected.  Pipelines can now be created.

**All deployment repos should have the `validate-yaml` and `version-tagging.yaml` pipelines.** **XXX**The  validate.yaml pipeline is not in the Template.  Must ask Stéphane if that is an oversight.  

The purpose of the version-tagging.yaml pipeline is to use versioning during deployment operations.  

Open the [Pipelines.md](https://github.com/ssc-spc-ccoe-cei/gcp-documentation/blob/main/Landing%20Zone%20Operations/Pipelines.md) document and perform the instructions in **Add Pipeline** to implement the `version-tagging.yaml` pipeline.  Pipeline.

***XXX*** The following section on 'fixing' the Pipeline should be in the Pipelines.md document.

The default settings in AzDo will produce an error (the screen output is 'You need the Git 'GenericContribute' permission to perform this action") when trying to run the pipeline, so changes must be made:

- In AzDo click the gear icon at the bottom left

- On the left, click on "Repositories" under the Repos title

- Select YOUR repo

- Click the 'Security' link under YOUR repo name

- Find 'iac-gcp-dev Build Service (gc-cpa)' under Users (not Groups)

- Change the 'Contribute' permission to 'Allow'


## Update Deployment Repo

This section is for updating the deployment repo itself, not the YAML configs.

As with any other change, they should be done through a PR process.

### Update from Template

The templates remain fairly static, but on occasion, important changes may be performed to them.

Follow these steps to update your deployment repo with changes from the templates:

**XXX**Don't know what these steps are.  Are the instructions on a on a case-by-case basis?  Talk to Stéphane.

### Update `tools` Submodule

The tools submodule can easily be updated to a new version.

Follow these steps to

1. Clone the deployment repo and checkout a new branch.
1. Edit `modversions.yaml` so that the `tools` submodule will be the appropriate version.
1. Run the following to get the proper version of the tools submodule:

    ```shell
    bash modupdate.sh
    ```

1. **XXX**This one is new to me.  Ask Stéphane if still valid.  If your repo contains pipelines that were created from `tools/pipeline-samples`, compare the YAML files to see if changes are required.
1. Stage, commit and push your branch to create a PR.