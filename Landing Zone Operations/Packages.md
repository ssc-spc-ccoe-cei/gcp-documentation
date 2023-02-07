# Landing Zone Packages
The landing zone solution uses some functionalities of [`kpt`](https://kpt.dev/book/02-concepts/) to manage [packages](https://kpt.dev/book/03-packages/) of YAML configs.

As a high level overview, a package will usually include files that are used specifically by kpt:
- `setters.yaml`: used to set customizable data.
- `Kptfile`: used to keep track of package versions and [declaratively set which functions](https://kpt.dev/book/04-using-functions/01-declarative-function-execution) should run during rendering. For example, [apply-setters](https://catalog.kpt.dev/apply-setters/v0.2/)

## Adding a Package
This is accomplished with the [`kpt pkg get`](https://kpt.dev/reference/cli/pkg/get/) command.

As a rule, packages should only be added in a deployment repo's `source-base` folder.

## Updating a Package
This is accomplished with the [`kpt pkg update`](https://kpt.dev/reference/cli/pkg/update/) command.

A deployment repo's `source-base` folder should always contain unedited packages.  This is where they are also updated.

Before proceeding it's a good idea to have a clean git working tree (all files are staged and committed).  This will make it easier to visualize changes and revert if needed.

## Removing a Package
This is accomplished by simply deleting the files.