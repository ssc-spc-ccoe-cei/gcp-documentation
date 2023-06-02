# Repos Structure and Roles

SSC adopted the [monorepo](https://monorepo.tools/) structure for repositories. Monorepos are great for managing multiple solutions inside a single repository.

The diagrams below show all the repositories **types** and the **roles** that are granted to each team.

### Tier1

![img](img/tier1.png)

### Tier2

![img](img/tier2.png)

### Tier34

![img](img/tier34.png)

## Gitops

The current solution is using [Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview) with git repos to pull configurations for deploying GCP infrastructure.

The diagram below describes the Gitops process.

![img](img/gitops.png)

The process to implement a code change goes like this:

1. The contributor will create a branch and make code changes in a tierX folder of the monorepo and open a pull request.
2. The CI process validates the change.
3. The reviewers can approve or deny the PR.
4. Once the pull request is completed, the branch is merged into the main branch and a new git tag specifying a new version is created for affected tierX folder.
5. The contributor will create a branch and modify the tag value in the folder csync/source-customization/`env`/tierX by setting it to the new version that got created in step 4.
6. The CI process validates the change.
7. The reviewers can approve or deny the PR.
8. The branch is merged into the main branch. No new tag is created.
9. The ConfigSync operator running in Config Controller which is already observing `HEAD` for the csync folder will pickup the new commit.
10. It will start observing that new version of the tierX folder
11. It deploys resources accordingly in GCP.
 &nbsp;

## Git

The git repos are organized in different categories:
TODO: continue working here
- `gcp-documentation` (this repo) contains documentation for the landing zone.
- Deployment repos:
  - `gcp-tier1-configsync` is where the initial `root-sync` object is pointing and is used to manage the root sync objects and versioning of the tier1 infrastructure in experimentation, dev, preprod and prod.
  - `gcp-tier1-infra` contains the landing zone configs for dev, preprod and prod.
  - `gcp-experimentation-tier1-infra` contains the landing zone configs for experimentation.
- `gcp-tools` contains common scripts and pipeline templates used by repos above as a git submodule.
- `gcp-blueprints-catalog` **private repo** contains SSC specific packages that are used to build a landing zone.

## Deployment Repos

These repos have a common directory structure with slight variations.

Below is a brief explanation of key repo components.  Some directories include sub-directories for each environment it configures (experimentation, dev, preprod, prod).  For simplicity, they will be expressed below as `<env_subdirs>`.

- `repo root`:
  - `.azure-pipelines` or `.github`: pipelines YAML files.
  - `bootstrap`: (only in the "infra" repos)
    - `<env_subdirs>`: contains `.env` file needed to create the management project which includes the Config Controller GKE cluster.
  - `deploy`: ("WET" folder) its content is automatically generated and must not be edited manually.
    - `<env_subdirs>`: contains the hydrated YAML files to be deployed.  **This is the directory used by Config Sync.**
  - `source-base`: ("DRY" folder) contains a collection of *unedited* packages that will be deployed to all environments.
  - `source-customization`:
    - `<env_subdirs>`: contains the customization files that will overwrite the base for the environment.  Directory and file names must be replicated.  For example, to customize `source-base/landing-zone/setters.yaml`, it should be copied, then edited, in `source-customization/<env_subdir>/landing-zone/setters.yaml`.
  - `tools`: git submodule from [gcp-tools](https://github.com/ssc-spc-ccoe-cei/gcp-tools)
  - `temp-workspace`: used temporarily during hydration, included in `.gitignore`.
  - `pre-commit-config.yaml`: the pre-commit will trigger `tools/scripts/kpt/hydrate.sh` to ensure all changes to source-base and/or source-customization were hydrated.
  - `modupdate.sh`: script to checkout the git submodules, i.e. tools.
  - `modversions.yaml`: file used by modupdate.sh to specify which versions of git sub modules to checkout.

Here are the links to the repositories templates for each deployment repos:

- [gcp-tier1-template](https://github.com/ssc-spc-ccoe-cei/gcp-tier1-template)
- [gcp-tier2-template](https://github.com/ssc-spc-ccoe-cei/gcp-tier2-template)
- [gcp-tier34-template](https://github.com/ssc-spc-ccoe-cei/gcp-tier34-template)

### Git Submodule: `tools`

As mentioned above, the [gcp-tools](https://github.com/ssc-spc-ccoe-cei/gcp-tools) repo is configured as a git submodule in `.gitmodules`.  This git configuration is limited to specifying a branch.

To overcome this limitation and checkout a specific tag or commit SHA, run `modupdate.sh` to checkout the version configured in `modversions.yaml`.

### Hydration Process

The tools submodule contains a `hydrate.sh` script to hydrate the configs with `kpt`.  The script must be executed when any changes to `source-base` and/or `source-customization` are made.  It can be executed locally or in a pre-commit hook and also in a validation pipeline during PR creation.

At a high level, the script will:

1. Find properly stuctured 'deploy', 'source-base' and 'source-customization' directories, then for each environment (experimentation, dev, preprod, prod), check if `source-customization/` contains that sub directory.  If so:
    - Create a `temp-workspace/<env>` directory to copy the `source-base/` and then copy `source-customization/<env>/`, this adds customization specific to that environment.
    - Validate that setters file(s) are customized.
    - Run `kpt fn render` and remove local configs in `temp-workspace/<env>`.
    - Check if newly hydrated files in `temp-workspace/<env>` are different than `deploy/<env>`.  If so, copy them to `deploy/<env>`.
    - Validate rendered files in `deploy/<env>` with `kubeval` and `nomos vet`.
1. If no errors are found and changes are detected, optionally push to the branch or exit the script with failure.
A failure ensures proper behavior when running from a pre-commit hook or a pipeline.

![img](img/hydrate-script-flowchart.png)

## Versioning

It is good practice to pin a repo reference to a specific commit using tags or commit SHA.

A simple version tagging pipeline can be used to increment a repo's version with git tags.  Samples can be found in the [gcp-tools](https://github.com/ssc-spc-ccoe-cei/gcp-tools/tree/main/pipeline-samples/version-tagging) repo.

Alternatively, the commit SHA can be used.  Although not as intuitive as tags, it is considered more secure.
