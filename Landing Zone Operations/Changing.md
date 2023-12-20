# Implementing a change on the landing zone

- [Implementing a change on the landing zone](#implementing-a-change-on-the-landing-zone)
  - [Step 1 - Setup](#step-1---setup)
  - [Step 2 - Change](#step-2---change)
    - [A) Add a Package](#a-add-a-package)
    - [B) Modify a Package](#b-modify-a-package)
    - [C) Update a Package](#c-update-a-package)
    - [D) Remove a Package](#d-remove-a-package)
  - [Step 3 - Hydrate](#step-3---hydrate)
  - [Step 4 - Publish](#step-4---publish)
  - [Step 5 - Synchronize / Promote Configs](#step-5---synchronize--promote-configs)

--------------------------------------

There can be different types of changes on the landing zone but they all start and end the same way.  This document will go through the different steps.

Before proceeding, you should familiarize yourself with the concepts in "[Repository Structure.md](../Architecture/Repository%20Structure.md)".

The landing zone solution uses some functionalities of [`kpt`](https://kpt.dev/book/02-concepts/) to manage [packages](https://kpt.dev/book/03-packages/) of YAML configs.

As a high level overview, a package will usually include files that are used specifically by kpt:

- `setters.yaml`: used to set customizable data.
- `Kptfile`: used to keep track of package versions and [declaratively set which functions](https://kpt.dev/book/04-using-functions/01-declarative-function-execution) should run during rendering. For example, [apply-setters](https://catalog.kpt.dev/apply-setters/v0.2/).

## Step 1 - Setup

For any type of change, you should start with a new branch and a clean git working tree (all files are staged and committed).  This will make it easier to visualize changes in [VSCode's Git Source Control](https://code.visualstudio.com/docs/sourcecontrol/overview) (or `git diff`) and revert if needed.
You can confirm that a working tree is clean by running `git status`.

From your local environment:

1. Update and export the variables below:

    ```shell
    export REPO_NAME='<the deployment repo name>'
    export REPO_URL='<the deployment repo URL>'

    # use naming convention if applicable (for example, add issue/task number in the branch name)
    export BRANCH_NAME=''
    ```

1. Clone the repository requiring change:

    ```shell
    git clone ${REPO_URL}
    ```

1. Move into the new folder corresponding to that repo:

    ```shell
    cd ${REPO_NAME}
    ```

1. Create a new branch, :

    ```shell
    git checkout -b ${BRANCH_NAME}
    ```

1. Run the following to get the proper version of the tools submodule:

    ```shell
    bash modupdate.sh
    ```

## Step 2 - Change

There are different types of changes.  Follow the appropriate section for instructions.

### A) Add a Package

This is accomplished with the [`kpt pkg get`](https://kpt.dev/reference/cli/pkg/get/) command.

As a rule, packages should only be added in a deployment monorepo's `tier<N>/<technology>/source-base` folder and **never** manually edited from there.  All customizations are to be made from the `tier<N>/<technology>/source-customization/<env>` folders.

Follow these steps to add a package:

1. You can update and set these variables to make it easier to run subsequent commands:

    ```shell
    # tier<N> value: 'tier1', 'tier2', 'tier3' or 'tier4'
    export TIER=''

    # the technology value: 'configcontroller' or 'kubernetes'
    export TECHNOLOGY=''

    # URI of the git repo containing the package
    # for example, 'https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'
    export REPO_URI=''

    # subdirectory of the package, relative to root of the package repo
    # for example, 'solutions/core-landing-zone'
    export PKG_PATH=''

    # the version to get, located in the package CHANGELOG.md, use 'main' if not available
    # for example, '0.0.1'
    export VERSION=''

    # leave blank by default, the package will be added to the root of 'source-base'
    # set this variable to add the package in subdirectory, relative to 'source-base'
    # for example, to add a new 'client-setup' in proper subdirectory, set the variable below to 'clients/<client-name>'
    export LOCAL_DEST_DIRECTORY=''
    ```

1. Change working directory to proper `source-base` and create the local destination directory if it's not the default:

    ```shell
    # move to root of repo, then the appropriate 'source-base'
    cd $(git rev-parse --show-toplevel)
    cd "${TIER}/${TECHNOLOGY}/source-base"

    # if the variable is set and the folder does not exist, then create it
    if [ -n "${LOCAL_DEST_DIRECTORY}" ] && [ ! -d "${LOCAL_DEST_DIRECTORY}" ]; then
      mkdir -p ${LOCAL_DEST_DIRECTORY}
    fi
    ```

1. Add the package with the following command:

    ```shell
    kpt pkg get ${REPO_URI}/${PKG_PATH}@${VERSION} ${LOCAL_DEST_DIRECTORY}
    ```

1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`.
1. Review the package's documentation for specific instructions.  Most will require some sort of customization, such as editing the `setters.yaml`.
It will need to be customized for each environment.  This is a manual process, a code snippet like below can help with the initial copy:

    ```shell
    # the path of the file to customize, relative to 'source-base'
    # for example, 'core-landing-zone/setters.yaml'
    export FILE_TO_CUSTOMIZE=''
    ```

    ```shell
    for env_subdir in experimentation dev preprod prod; do
        # check if env. folder exists in source-customization
        if [ -d "../source-customization/${env_subdir}" ]; then
            # copy the file, with no overwrite, keeping full path
            cp --no-clobber --parents "${FILE_TO_CUSTOMIZE}" "../source-customization/${env_subdir}"
        fi
    done
    ```

1. Ensure all files copied under `source-customization` have been edited to reflect each environment.
    > ***Advanced customization tip.***
    In some cases, certain resource YAML files may need to be edited for some environments, but not others.  This should be avoided as much as possible because it  complicates the update process.
    For example, to completely remove a specific org policy from experimentation:
        - Copy that org policy's YAML file in the experimentation customization folder, ***making sure to maintain the same directory structure***.
        - Put the entire file in a comment block.
        - The hydration process will then ignore this commented resource definition, effectively removing it.
1. Review all customizations with VSCode's built-in Source Control viewer or by running `git diff`.  If satisfied, proceed to [Step 3 - Hydrate](#step-3---hydrate).

### B) Modify a Package

By design, this is accomplished by modifying configs in the `tier<N>/<technology>/source-customization/<env>`. Files in other directories should never be modified manually.

***Standard customization should only involve the `setters.yaml` file.***

Follow these steps to modify a package:

1. Modify the configs for each applicable environment in `tier<N>/<technology>/source-customization/<env>`
1. Once all customizations have been reviewed locally, proceed to [Step 3 - Hydrate](#step-3---hydrate).

### C) Update a Package

This is accomplished with the [`kpt pkg update`](https://kpt.dev/reference/cli/pkg/update/) command.

The default `resource-merge` strategy is usually appropriate but can sometimes omit certain file structure changes (blank line between comments, etc.).
In these cases, if the structural change is required as part of the update, it may be necessary to *carefully* use the `force-delete-replace` strategy.

A deployment monorepo's `source-base` folder should always contain unedited packages.  This is where they are also updated.

> **!!! IMPORTANT !!!** Once a package is updated, it's important to verify if there are changes for files that have been customized.  For example, `setters.yaml` files.

Follow these steps to update a package:

1. You can update and set these variables to make it easier to run subsequent commands:

    ```shell
    # tier<N> value: 'tier1', 'tier2', 'tier3' or 'tier4'
    export TIER=''

    # the technology value: 'configcontroller' or 'kubernetes'
    export TECHNOLOGY=''

    # subdirectory of the local package to update, relative to 'source-base'
    # for example, 'core-landing-zone'
    export PKG_PATH=''

    # the version to update to
    # for example, '0.0.2'
    export VERSION=''
    ```

1. Change working directory to proper `source-base`:

    ```shell
    # move to root of repo, then the appropriate 'source-base'
    cd $(git rev-parse --show-toplevel)
    cd "${TIER}/${TECHNOLOGY}/source-base"
    ```

1. Update the package with the default `resource-merge` strategy:

    ```shell
    kpt pkg update ${PKG_PATH}@${VERSION}
    ```

1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`.
1. If the changes are as expected, proceed to the next step.  Otherwise, you can try these steps:
    1. Revert the changes through VSCode Source Control or by running:

        ```shell
        git restore .
        ```

    1. Update the package with the `force-delete-replace` strategy:

        ```shell
        kpt pkg update ${PKG_PATH}@${VERSION} --strategy=force-delete-replace
        ```

    1. Carefully review the changes with VSCode Source Control or by running `git diff`.
    Paying close attention that it did not remove something it should not have (local subpackages, etc.). You can easily discard specific changes in VSCode's Source Control or with `git restore <file>`.
    This strategy can also remove or modify `cnrm.cloud.google.com/blueprint:` annotations in many YAML files.  These changes will unfortunately create a large git diff but can be accepted.
    1. If the changes are as expected, proceed to the next step.
1. For each file under `source-customization/<env>`, verify if it changed in `source-base`.
For example, if the landing-zone package is updated, compare `tier1/configcontroller/source-customization/dev/core-landing-zone/setters.yaml` with `tier1/configcontroller/source-base/core-landing-zone/setters.yaml`.
    - If a change is detected, manually update the file in `source-customization/<env>`.
1. Once all customizations have been reviewed locally, proceed to [Step 3 - Hydrate](#step-3---hydrate).

### D) Remove a Package

This is accomplished by simply deleting the package files in `tier<N>/<technology>/source-base` and its customizations in `tier<N>/<technology>/source-customization/<env>`.

> **!!! IMPORTANT !!!** Before deleting a package, confirm that it does not have subpackages that are still needed.

Follow these steps to remove a package:

1. You can update and set these variables to make it easier to run subsequent commands:

    ```shell
    # tier<N> value: 'tier1', 'tier2', 'tier3' or 'tier4'
    export TIER=''

    # the technology value: 'configcontroller' or 'kubernetes'
    export TECHNOLOGY=''

    # subdirectory of the local package to remove, relative to 'source-base'
    # for example, 'core-landing-zone'
    export PKG_PATH=''
    ```

1. Change working directory to proper `source-base`:

    ```shell
    # move to root of repo, then the appropriate 'source-base'
    cd $(git rev-parse --show-toplevel)
    cd "${TIER}/${TECHNOLOGY}/source-base"
    ```

1. Remove the package:

    ```shell
    rm --recursive ${PKG_PATH}
    ```

1. Remove the customization for each environment:

    ```shell
    for env_subdir in experimentation dev preprod prod; do
        # check if env. folder exists in source-customization, then delete it
        if [ -d "../source-customization/${env_subdir}/${PKG_PATH}" ]; then
            rm --recursive "../source-customization/${env_subdir}/${PKG_PATH}"
        fi
    done
    ```

1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`.  If satisfied, proceed to [Step 3 - Hydrate](#step-3---hydrate).

## Step 3 - Hydrate

This is accomplished with the `hydrate.sh` script located in the tools submodule.  In part, it uses the [`kpt fn render`](https://kpt.dev/reference/cli/fn/render/) command.

> **!!! IMPORTANT !!!** `kpt fn render` should never be executed manually in any directory.  This ensures better package updates and minimizes hydration issues.

In summary, the script will:

- add (`source-base` + `source-customization/<env>`) to `temp-workspace/<env>`
- then hydrate `temp-workspace/<env>` to `deploy/<env>`

Follow these steps to hydrate your change:

1. Execute the hydration script ***from the root of the repository***:

    ```shell
    cd $(git rev-parse --show-toplevel)
    bash tools/scripts/kpt/hydrate.sh
    ```

1. Address errors, if any.
1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`, specifically the hydrated files in the `deploy/<env>` folders.  If satisfied, it's time for publishing.

## Step 4 - Publish

At this point, the changes only exist locally. They are now ready to be published for peer review and approval.

Follow these steps to publish the changes:

1. Update and export the variables below:

    ```shell
    # the branch name created in step 1
    export BRANCH_NAME=''
    export COMMIT_MESSAGE=''
    ```

1. Prepare your commit by staging the files:

    ```shell
    git add .
    ```

1. Commit your changes:

    ```shell
    git commit -m "${COMMIT_MESSAGE}"
    ```

1. Push your changes to the monorepo's origin:

    ```bash
    git push --set-upstream origin ${BRANCH_NAME}
    ```

1. Create a new [pull requests (PR)](https://learn.microsoft.com/en-us/azure/devops/repos/git/pull-requests?view=azure-devops&tabs=browser) on the monorepo to merge this `<branch name>` into `main`.
1. Confirm all required checks are successful (approvals, tests, etc.).  If checks are failing, address them in your local branch then stage, commit and push them to origin.
1. Complete the pull request once all required checks are successful.

## Step 5 - Synchronize / Promote Configs

This section contains information on how changes can be promoted between environments.

Changes to deployment monorepos will only be applied to GCP when the `csync/tier<N>/<technology>/deploy/<env>` folder is updated.

For simplicity, this example will focus on a `configcontroller` resource change in the `gcp-env-tier1` monorepo:

1. A change is made in folder `tier1/configcontroller`.
    > **!!! It's important to add the `source-customization` for each environment.  This will ensure all environments are rendered, validated and tagged at the same time. !!!**
1. Once the PR is merged, note the new tag version or commit SHA.
1. At this point the changes have not been deployed to GCP. Changes of type "Modify a Package" are required in folder `csync/tier1/configcontroller/` for each environment.
1. `dev`:
    - Set `version:` in `csync/tier1/configcontroller/source-customization/dev/root-sync-git/setters-version.yaml` to the new tag or commit SHA noted earlier.
    - Hydrate the monorepo and create a PR.
    - Once the PR is merged the Config Sync operator will pick up the updated configs in `csync/tier1/configcontroller/deploy/dev`, then the specific commit for `tier1/configcontroller/deploy/dev`.
    - Confirm synchronization of all resources from the [Config Sync Dashboard](https://console.cloud.google.com/kubernetes/config_management/dashboard) or by running `nomos status`.
    - Validate landing zone and workload functionalities for the `dev` environment in GCP.  Proceed to `preprod` if successful, restart the process if not.
1. `preprod`:
    - Set `version:` in `csync/tier1/configcontroller/source-customization/preprod/root-sync-git/setters-version.yaml` to the same value as `dev`.
    - Hydrate the monorepo and create a PR.
    - Once the PR is merged the Config Sync operator will pick up the updated configs in `csync/tier1/configcontroller/deploy/preprod`, then the specific commit for `tier1/configcontroller/deploy/preprod`.
    - Confirm synchronization of all resources from the [Config Sync Dashboard](https://console.cloud.google.com/kubernetes/config_management/dashboard) or by running `nomos status`.
    - Validate landing zone and workload functionalities for the `preprod` environment in GCP.  Proceed to `prod` if successful, restart the process if not.
1. `prod`:
    - Set `version:` in `csync/tier1/configcontroller/source-customization/prod/root-sync-git/setters-version.yaml` to the same value as `preprod`.
    - Hydrate the monorepo and create a PR.
    - Once the PR is merged the Config Sync operator will pick up the updated configs in `csync/tier1/configcontroller/deploy/prod`, then the specific commit for `tier1/configcontroller/deploy/prod`.
    - Confirm synchronization of all resources from the [Config Sync Dashboard](https://console.cloud.google.com/kubernetes/config_management/dashboard) or by running `nomos status`.
    - Validate landing zone and workload functionalities for the `prod` environment in GCP.  Restart the process if not successful.
