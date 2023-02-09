# Implementing a change on the landing zone
There can be different types of changes on the landing zone but they all start and end the same way.  This document will go through the different steps.

Before proceeding, you should familiarize yourself with the concepts in "[Repository Structure.md](../Architecture/Repository%20Structure.md)".

The landing zone solution uses some functionalities of [`kpt`](https://kpt.dev/book/02-concepts/) to manage [packages](https://kpt.dev/book/03-packages/) of YAML configs.

As a high level overview, a package will usually include files that are used specifically by kpt:
- `setters.yaml`: used to set customizable data.
- `Kptfile`: used to keep track of package versions and [declaratively set which functions](https://kpt.dev/book/04-using-functions/01-declarative-function-execution) should run during rendering. For example, [apply-setters](https://catalog.kpt.dev/apply-setters/v0.2/).

## Step 1 - The Setup
For any type of change, you should start with a new branch and a clean git working tree (all files are staged and committed).  This will make it easier to visualize changes in [VSCode's Git Source Control](https://code.visualstudio.com/docs/sourcecontrol/overview) (or `git diff`) and revert if needed.  
You can confirm that a working tree is clean by running `git status`.

From your local environment:
1. Clone the repository requiring change:
    ```bash
    git clone <REPO URL>
    ```
1. Move into the new folder corresponding to that repo:
    ```bash
    cd <REPO NAME>
    ```
1. Create a new branch, use naming convention if applicable (for example, add issue/work item # in the branch name):
    ```bash
    git checkout -b <BRANCH NAME>
    ```
1. Clone and checkout the proper version of the tools sub module by running:
    ```bash
    bash modupdate.sh
    ```
## Step 2 - The Change
There are different types of changes.  Follow the appropriate section for instructions.

### A) Add a Package
This is accomplished with the [`kpt pkg get`](https://kpt.dev/reference/cli/pkg/get/) command.

As a rule, packages should only be added in a deployment repo's `source-base` folder and **never** manually edited from there.  All customizations are to be made from the `source-customization/<env>` folders.

Follow these steps for add a package:

1. Move into the `source-base` folder:
    ```bash
    cd source-base
    ```
1. You can update and set these variables to make it easier to run subsequent commands:
    ```bash
    # pkg repo url including directories
    # for example, 'https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/hierarchy/core-env'
    export PKG_URL=''

    # the version to get, look in the pkg CHANGELOG.md, use 'main' if not available
    # for example, '0.0.1'
    export VERSION=''

    # the destination folder to save the pkg
    # for example, 'landing-zone/hierarchy'
    export DEST_FOLDER=''
    ```
1. Add the package with the following command:
    ```bash
    kpt pkg get ${PKG_URL}@${VERSION} ${DEST_FOLDER}
    ```
1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`.
1. Review the package's documentation for specific instructions.  Most will require some sort of customization, such as editing the `setters.yaml`.  
It will need to be customized for each environment.  This is a manual process, a code snippet like below can help with the initial copy:
    ```bash
    # the file path to customize, relative to 'source-base'
    # for example, 'landing-zone/org-policies/setters.yaml'
    export FILE_TO_CUSTOMIZE=''

    for env_subdir in sandbox dev uat prod; do
        # check if env. folder exists in source-customization
        if [ -d "../source-customization/${env_subdir}" ]; then
            # copy the file, with no overwrite, keeping full path
            cp --no-clobber --parents "${FILE_TO_CUSTOMIZE}" "../source-customization/${env_subdir}"
        fi
    done
    ```
1. Ensure all files copied under `source-customization` have been edited to reflect each environment.
    > ***Advanced customization tip.***<br>
    In some cases, certain resource YAML files may need to be edited for some environments, but not others.  This should be avoided as much as possible because it  complicates the update process.<br>
    For example, to completely remove a specific org policy from sandbox:<br>
        - Copy that org policy's YAML file in the sandbox customization folder, ***making sure to maintain the same directory structure***.<br> 
        - Put the entire file in a comment block.<br>
        - The hydration process will then ignore this commented resource definition, effectively removing it.
1. Once all customizations have been reviewed locally, it's time for hydration.
1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`.  If satisfied, it's time for hydration.

### B) Modify a Package
By design, this is accomplished by modifying configs in the `source-customization/<env>`.  Files in other directories should never be modified manually.

Follow these steps to modify a package:

1. Modify the configs for each applicable environment in `source-customization/<env>`
1. Once all customizations have been reviewed locally, it's time for hydration.

### C) Update a Package
This is accomplished with the [`kpt pkg update`](https://kpt.dev/reference/cli/pkg/update/) command.  

The default `resource-merge` strategy is usually appropriate but can sometimes omit certain file structure changes (blank line between comments, etc.).  
In these cases, if the structural change is required as part of the update, it may be necessary to *carefully* use the `force-delete-replace` strategy.

A deployment repo's `source-base` folder should always contain unedited packages.  This is where they are also updated.

> **!!! IMPORTANT !!!** Once a package is updated, it's important to verify if there are changes for files that have been customized.  For example, `setters.yaml` files.

Follow these steps to update a package:

1. Move into the `source-base` folder:
    ```bash
    cd source-base
    ```
1. You can update and set these variables to make it easier to run subsequent commands:
    ```bash
    # the folder of the pkg to be updated
    # for example, 'landing-zone/hierarchy'
    export PKG_PATH=''

    # the version to update to
    # for example, '0.0.2'
    export VERSION=''
    ```
1. Update the package with the default `resource-merge` strategy:
    ```bash
    kpt pkg update ${PKG_PATH}@${VERSION}
    ```
1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`.
1. If the changes are as expected, proceed to the next step.  Otherwise, you can try these steps:
    1. Revert the changes through VSCode Source Control or by running:
        ```bash
        git restore .
        ```
    1. Update the package with the `force-delete-replace` strategy:
        ```bash
        kpt pkg update ${PKG_PATH}@${VERSION} --strategy=force-delete-replace
        ```
    1. Carefully review the changes with VSCode Source Control or by running `git diff`.  
    Paying close attention that it did not remove something it should not have (local subpackages, etc.). You can easily discard specific changes in VSCode's Source Control or with `git restore <file>`.  
    This strategy can also remove or modify `cnrm.cloud.google.com/blueprint:` annotations in many YAML files.  These changes will unfortunately create a large git diff but can be accepted.
    1. If the changes are as expected, proceed to the next step.
1. For each file under `source-customization/<env>`, verify if it changed in `source-base`.  
For example, if the landing-zone package is updated, compare `source-customization/dev/landing-zone/setters.yaml` with `source-base/landing-zone/setters.yaml`.
    - If a change is detected, manually update the file in `source-customization/<env>`.
1. Once all customizations have been reviewed locally, it's time for hydration.

### D) Remove a Package
This is accomplished by simply deleting the package files in `source-base` and its customizations in `source-customization/<env>`.

> **!!! IMPORTANT !!!** Before deleting a package, confirm that it does not have subpackages that are still needed.

Follow these steps to remove a package:
1. Move into the `source-base` folder:
    ```bash
    cd source-base
    ```
1. You can update and set these variables to make it easier to run subsequent commands:
    ```bash
    # the folder of the pkg to be removed
    # for example, 'landing-zone/logging'
    export PKG_PATH=''
    ```
1. Remove the package:
    ```bash
    rm --recursive ${PKG_PATH}
    ```
1. The package customizations now need to be removed, move to `source-customization`:
    ```bash
    cd ../source-customization
    ```
1. Remove the customization for each environment:
    ```bash
    for env_subdir in sandbox dev uat prod; do
        # check if env. folder exists in source-customization
        if [ -d "${env_subdir}/${PKG_PATH}" ]; then
            rm --recursive "${env_subdir}/${PKG_PATH}"
        fi
    done
    ```
1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`.  If satisfied, it's time for hydration.

## Step 3 - The Hydration
This is accomplished with the `hydrate.sh` script located in the tools submodule.  In part, it uses the [`kpt fn render`](https://kpt.dev/reference/cli/fn/render/) command.

> **!!! IMPORTANT !!!** `kpt fn render` should never be executed manually in any directory.  This ensures better package updates and minimizes hydration issues.

In summary, the script will:
- add (`source-base` + `source-customization/<env>`) to `temp-workspace/<env>`
- then hydrate `temp-workspace/<env>` to `deploy/<env>`

Follow these steps to hydrate your change:
1. Execute the hydration script ***from the root of the repository***:
    ```bash
    bash tools/scripts/kpt/hydrate.sh
    ```
1. Address errors, if any.
1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`, specifically the hydrated files in the `deploy/<env>` folders.  If satisfied, it's time for publishing.

## Step 4 - The Publishing
At this point, the changes only exist locally. They are now ready to be published for peer review and approval.

Follow these steps to publish the changes:
1. Prepare your commit by staging the files:
    ```bash
    git add .
    ```
1. Commit your changes:
    ```bash
    git commit -m '<MEANINGFULL MESSAGE GOES HERE>
    ```
1. Push your changes to the repo's origin:
    ```bash
    git push --set-upstream origin <branch name>
    ```
1. Create a new [pull requests (PR)](https://learn.microsoft.com/en-us/azure/devops/repos/git/pull-requests?view=azure-devops&tabs=browser) on the repo to merge this `<branch name>` into `main`.
1. Confirm all required checks are successful (approvals, tests, etc.).  If checks are failing, address them in your local branch then stage, commit and push them to origin.
1. Complete the pull request once all required checks are successful.
1. That's it!

# Synchronizing / Promoting Configs
This section contains additional information on how changes can be promoted between environments.

Changes to `infra` repos will only be applied to GCP when the `configsync` repo is updated.  The concept is similar for all repos

This example will focus on `gcp-tier1-infra` and `gcp-tier1-configsync`:

1. A change is made on `gcp-tier1-infra`.
    > **!!! It's important to add the `source-customization` for each environment.  This will ensure all environments are rendered, validated and tagged at the same time. !!!**
1. Once the PR is merged, note the new tag version or commit SHA.
1. At this point the changes have not been deployed to GCP. Changes of type "Modify a Package" are required in `gcp-tier1-configsync` to do so for each environment.
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