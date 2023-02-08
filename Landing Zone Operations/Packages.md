# Landing Zone Packages
TODO: merge this documentation in Changing.md


The landing zone solution uses some functionalities of [`kpt`](https://kpt.dev/book/02-concepts/) to manage [packages](https://kpt.dev/book/03-packages/) of YAML configs.

As a high level overview, a package will usually include files that are used specifically by kpt:
- `setters.yaml`: used to set customizable data.
- `Kptfile`: used to keep track of package versions and [declaratively set which functions](https://kpt.dev/book/04-using-functions/01-declarative-function-execution) should run during rendering. For example, [apply-setters](https://catalog.kpt.dev/apply-setters/v0.2/).

For any type of change, you should also have a clean git working tree (all files are staged and committed).  This will make it easier to visualize changes in [VSCode's Git Source Control](https://code.visualstudio.com/docs/sourcecontrol/overview) and revert if needed.

Confirm that your local git is on a branch with a clean working tree:
```bash
git status
```

## Adding a Package
This is accomplished with the [`kpt pkg get`](https://kpt.dev/reference/cli/pkg/get/) command.

As a rule, packages should only be added in a deployment repo's `source-base` folder and **never** manually edited from there.  All customizations are to be made from the `source-customization/<env>` folders.

Follow these steps for add a package:

1. Move into the `source-base` folder:
    ```bash
    cd source-base
    ```
1. You can update and set these variables to make it easier to run subsequent commands:
    ```bash
    # pkg repo url including directories, for example,
    # 'https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/hierarchy/core-env'
    export PKG_URL=''

    # the version to get, for example,
    # '0.0.1'
    export VERSION=''

    # the destination folder to save the pkg, for example,
    # 'landing-zone/hierarchy'
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
    # the file path to customize, relative to 'source-base', for example,
    # 'landing-zone/org-policies/setters.yaml'
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
    In some cases, certain resource YAML files may need to be edited for some environments, but not others.  This should be avoided as much as possible because it will complicate the update process.<br>
    For example, to completely remove a specific org policy from sandbox:<br>
    Copy that org policy's YAML file in the sandbox customization folder, ***making sure to maintain the same directory structure***.<br> 
    Put the entire file in a comment block.<br>
    The hydration process will then ignore this commented resource definition, effectively removing it.
1. Once all customizations have been reviewed locally, it's time for hydration.
1. Review the changes with VSCode's built-in Source Control viewer or by running `git diff`.  If satisfied, it's time for hydration.

## Modifying a Package
By design, this is accomplished by modifying configs in the `source-customization/<env>`.  Files in other directories should never be modified manually.

Follow these steps to modify a package:

1. Modify the configs for each applicable environment in `source-customization/<env>`
1. Once all customizations have been reviewed locally, it's time for hydration.

## Updating a Package
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
    # the folder of the pkg to be updated, for example,
    # 'landing-zone/hierarchy'
    export PKG_PATH=''

    # the version to update to, for example,
    # '0.0.2'
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

## Removing a Package
This is accomplished by simply deleting the package files in `source-base` and its customizations in `source-customization/<env>`.

Follow these steps to remove a package:
1. Move into the `source-base` folder:
    ```bash
    cd source-base
    ```
1. You can update and set these variables to make it easier to run subsequent commands:
    ```bash
    # the folder of the pkg to be removed, for example,
    # 'landing-zone/hierarchy'
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