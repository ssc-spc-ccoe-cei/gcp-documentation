# Implementing a change on the landing zone
There can be different types of changes but they all start and end the same way.  This document will go through the different steps.

1. ## Repo cloning
    - Clone locally the repository requiring code change
        ```
        git clone <REPO URL>
        ```
1. ## Create branch
    - Create a branch with a name representing the issue/work item
        ```
        git checkout -b <BRANCH NAME>
        ```
1. ## Make code changes 
- Adding a package
- Modifying a package
- Updating a package
- Removing a package
    - Edit any file as required

    - Customizing the content of a `source-base` package.
    
        The folder `source-customization/<env_subdirs>`: contains the customization files that will overwrite the base for the environment.  Directory and file names must be replicated.  For example, to customize `source-base/landing-zone/setters.yaml`, it should be copied, then edited, in `source-customization/<env_subdir>/landing-zone/setters.yaml`.

    &nbsp;
1. ## Generate hydrated files
    - Execute hydration script ***from the root of the repository***
      ```
      bash tools/scripts/kpt/hydrate.sh
      ```
1. ## Add changes to repository
    1. Review changes using a git tool or just by running `git diff`
    1. Prepare your commit by staging the files by running `git add .`
    1. Commit your changes by running `git commit -m '<MEANINGFULL MESSAGE GOES HERE>`
    1. Push your changes to origin by running `git push --set-upstream origin <branch name>`
    1. Open a pull request process to merge this `<branch name>` into `main`
    1. Wait for branch policies requirements (approvals, tests, etc.) to be met
    1. Complete the pull request