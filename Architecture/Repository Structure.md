# Repos Structure and Roles
The diagram below shows all the infrastructure repository **types** and the **roles** that are granted to each team.
![img](img/tiers.png)

# Gitops
The current solution is using [Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview) with git repos to pull configurations for deploying GCP infrastructure.

The diagram below describes the Gitops process that involves 2 repositories. 
- Infra
- ConfigSync

The process to implement a code change goes like this:

1. The Infrastructure Admin will make code changes on the Infra repo and open a pull request. Then the CI process validates the change while reviewers can approve or deny it. Once the pull request is completed, the branch is merged into the main branch and a new git tag specifying a new version is created.
2. The Infrastructure Admin will modify the tag value in the ConfigSync repo by setting it to the new version that got created on the Infra repo. By doing so, The ConfigSync operator running in Config Controller will start observing that new version of the Infra Repo. 
 &nbsp;

![img](img/gitops.png)

# Git 

The git repos are organized in different categories:
- `gcp-documentation` (this repo) contains documentation for the landing zone.
- Deployment repos: two types exists, one to manage Config Sync repos, the other to store infrastructure configurations.
    - `gcp-tier1-configsync` is where the initial `root-sync` object is pointing and is used to manage the root sync objects and versioning of the tier1 infrastructure in sandbox, dev, uat and prod.
    - `gcp-tier1-infra` contains the landing zone configs for dev, uat and prod.
    - `gcp-sandbox-tier1-infra` contains the landing zone configs for sandbox.
- `gcp-tools` contains common scripts and pipeline templates used by repos above as a git submodule.
- `gcp-blueprints-catalog` **private repo** contains SSC specific packages that are used to build a landing zone.

TODO: tier2 and tier3 repos

## Deployment Repos
These repos have a common directory structure with slight variations. 

Below is a brief explanation of key repo components.  Some directories include sub-directories for each environment it configures (sandbox, dev, uat, prod).  For simplicity, they will be expressed below as `<env_subdirs>`.

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

**A repo template is available [here](https://github.com/ssc-spc-ccoe-cei/gcp-repo-template).**

### Git Submodule: `tools`
As mentionned above, the [gcp-tools](https://github.com/ssc-spc-ccoe-cei/gcp-tools) repo is configured as a git submodule in `.gitmodules`.  This git configuration is limited to specifying a branch.

A mechanism to checkout (pin) a specific tag or commit SHA is implemented with `modupdate.sh` and `modversions.yaml`.
```bash
# clone and checkout submodule by running:
bash modupdate.sh
```
### Hydration Process

> **!!!** `kpt fn render` should never be executed manually in any directory.  This ensures better package updates and minimizes hydration issues.

The tools submodule contains a script to hydrate the configs with `kpt`.  The script must be executed when any changes to `source-base` and/or `source-customization` are made.  It is configured to run as a pre-commit hook for local validation and also in a validation pipeline during PR creation (TODO: create pipeline).

```bash
bash tools/scripts/kpt/hydrate.sh
```

At a high level, the script will:
1. For each environment (sandbox, dev, uat, prod), check if `source-customization/` contains that sub directory.  If so:
    - Create a `temp-workspace/<env>` directory to copy the `source-base/` and then copy `source-customization/<env>/`, this adds customization specific to that environment.
    - Run `kpt fn render` and remove local configs in `temp-workspace/<env>`.
    - Check if newly hydrated files in `temp-workspace/<env>` are different than `deploy/<env>`.  If so, copy them to `deploy/<env>`.
    - Run `nomos vet` in `deploy/<env>` to validate syntax.
1. If any change was detected, exit the script with failure.  This will fail the pre-commit or the pipeline, the operator will have to address any errors or simply re-run `bash tools/scripts/kpt/hydrate.sh` if no errors were found.

In summary:

1. `source-base` + `source-customization/<env>` = `temp-workspace/<env>`
1. hydrate `temp-workspace/<env>` to `deploy/<env>`

## Versioning

It is good practice to pin a repo reference to a specific commit using tags or commit SHA.

A simple tagging pipeline can be used to increment a repo's version.  Samples can be found in the [gcp-tools](https://github.com/ssc-spc-ccoe-cei/gcp-tools/tree/main/pipeline-samples/tagging) repo.

Alternatively, the commit SHA can be used.  Although not as intuitive as tags, it is considered more secure.