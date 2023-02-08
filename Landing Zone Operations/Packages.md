# Landing Zone Packages
The landing zone solution uses some functionalities of [`kpt`](https://kpt.dev/book/02-concepts/) to manage [packages](https://kpt.dev/book/03-packages/) of YAML configs.

As a high level overview, a package will usually include files that are used specifically by kpt:
- `setters.yaml`: used to set customizable data.
- `Kptfile`: used to keep track of package versions and [declaratively set which functions](https://kpt.dev/book/04-using-functions/01-declarative-function-execution) should run during rendering. For example, [apply-setters](https://catalog.kpt.dev/apply-setters/v0.2/).

For any type of change, you should also have a clean git working tree (all files are staged and committed).  This will make it easier to visualize changes and revert if needed.

1. Confirm that your local git is on a branch with a clean working tree:
    ```bash
    git status
    ```

## Adding a Package
This is accomplished with the [`kpt pkg get`](https://kpt.dev/reference/cli/pkg/get/) command.

As a rule, packages should only be added in a deployment repo's `source-base` folder.  Customization is then performed in `source-customization/<env>`.

## Updating a Package
This is accomplished with the [`kpt pkg update`](https://kpt.dev/reference/cli/pkg/update/) command.  

The default `resource-merge` strategy is usually appropriate but can sometimes omit certain file structure changes (blank line between comments, etc.).  
In these cases, if the structural change is required as part of the update, it may be necessary to *carefully* use the `force-delete-replace` strategy.

A deployment repo's `source-base` folder should always contain unedited packages.  This is where they are also updated.

> **!!! IMPORTANT !!!** Once a package is updated, it's important to verify if there are changes for files that have been customized.  For example, `setters.yaml` files.

Follow these steps to update a package:

1. You can update and set these variables to make it easier to run subsequent commands:
    ```bash
    export PKG_PATH='<relative/path/to/pkg>'
    export VERSION='<version to update to>'
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
    1. Carefully review the changes with [VSCode's Git Source Control](https://code.visualstudio.com/docs/sourcecontrol/overview) viewer or by running `git diff`.  
    Paying close attention that it did not remove something it should not have (local subpackages, etc.). You can easily discard specific changes in VSCode's Source Control or with `git restore <file>`.  
    This strategy can also remove or modify `cnrm.cloud.google.com/blueprint:` annotations in many YAML files.  These changes will unfortunately create a large git diff but can be accepted.
    1. If the changes are as expected, proceed to the next step.
1. For each file under `source-customization/<env>`, verify if it changed in `source-base`.  
For example, if the landing-zone package is updated, compare `source-customization/dev/landing-zone/setters.yaml` with `source-base/landing-zone/setters.yaml`.
    - If a change is detected, manually update the file in `source-customization/<env>`.
1. Once all customizations have been reviewed locally, it's time to wrap things up.

## Removing a Package
This is accomplished by simply deleting the package files in `source-customization` and its customizations in `source-customization/<dev>`.

Follow these steps to remove a package:
1. Move into the `source-base` folder:
    ```bash
    cd source-base
    ```
1. You can update and set these variables to make it easier to run subsequent commands:
    ```bash
    export PKG_PATH='<relative/path/to/pkg>'
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
        rm --recursive "${env_subdir}/${PKG_PATH}"
    done
    ```
1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`.  If satisfied, it's time to wrap things up.