# Landing zone building

Shared Services Canada has automated a portion of the deployment process for the [landing-zone-v2](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#Organization) solution in the Google's Pubsec Toolkit repo.

This document will describe how the automated scripts can be used for building a landing zone.

**We recommend that you read the entire procedure before initiating it**

**Important** SSC is using Azure Devops Repositories and Pipelines as its git solution.

## Requirements

Shared Services Canada uses the "[Multiple GCP organizations](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#multiple-gcp-organizations)" architecture.

Review the requirements listed [here](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/docs/landing-zone-v2/README.md).

SSC implements a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.
As illustrated in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator requires an Infra repo and a ConfigSync repo.

The steps assume these repos have already been created by following the "Create New Deployment Repo" section in [Repositories.md](./Repositories.md):

- One of the two infra repos:
  - `gcp-experimentation-tier1-infra`: if building a experimentation landing-zone.
  - `gcp-tier1-infra`: if building a dev, preprod or prod landing-zone.
- `gcp-tier1-configsync`: to identify which git revision of `tier1-infra` the Config Sync operator should observe.

## Setup

## 1. Bootstrap the Config Controller Project

The automated script creates a project, the FW settings, a Cloud router, a Cloud NAT, a private service connect endpoint and the Anthos Config Controller cluster. It also creates a root-sync.yaml file.

The script requires a `.env` file to deploy the environment.

1. Start a new change for your `tier1-infra` repo, follow the "Step 1 - Setup" section of [Changing.md](./Changing.md).
    - Experimentation

        repo name = `gcp-experimentation-tier1-infra`
    - DEV, PREPROD, PROD

        repo name = `gcp-tier1-infra`
1. Your terminal should now be at the root of your `tier1-infra` repo, on a new branch and with the tools submodule populated.
1. If it doesn't exist, create a `bootstrap/<env>` directory.  It will be used to backup files generated during bootstrapping.
1. Copy the example.env file from the `tools/scripts/bootstrap` folder.

    ```shell
    cp tools/scripts/bootstrap/.env.sample bootstrap/<ENV>/.env
    ```

2. **Important** Customize the new file with the appropriate values for the landing zone you are building.

3. Export a `TOKEN` variable.  Set its value to the PAT which has read access to the repo.

    ```shell
    export TOKEN='xxxxxxxxxxxxxxx'
    ```

1. Run the setup-kcc automated script:

    ```shell
    bash tools/scripts/bootstrap/setup-kcc.sh <PATH TO .ENV FILE>
    ```

1. Once the script has completed, you need to move the `root-sync.yaml` file into the `bootstrap/<env>` folder of the `tier1-infra` repo.

    ```shell
    mv root-sync.yaml bootstrap/<ENV>
    ```

1. Validate the Config Controller deployment.  You should see a synchronized `root-sync` to your `...gcp-tier1-configsync/deploy/<env>@main` repo containing no resource.
Perform step 4 from this [procedure](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#4-validate-the-landing-zone-deployment).

## 2. Build the Landing Zone Packages

We will build the landing zone by adding a collection of packages to the `tier1-infra` repo.

### Add Packages

Follow step 2A "Add a Package" of [Changing.md](./Changing.md) to add each of these packages:

1. The gatekeeper-policies package:
    - Package details:

        ```shell
        export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

        export PKG_PATH='solutions/gatekeeper-policies'

        # the version to get, located in the package's CHANGELOG.md
        export VERSION=''

        export LOCAL_DEST_DIRECTORY='landing-zone/gatekeeper-policies'
        ```

    - Customization:

        ```shell
        export FILE_TO_CUSTOMIZE='landing-zone/gatekeeper-policies/naming-rules/project/setters.yaml'
        ```

1. The core-landing-zone package:
    - Package details:

        ```shell
        export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

        export PKG_PATH='solutions/core-landing-zone'

        # the version to get, located in the package's CHANGELOG.md
        export VERSION=''

        export LOCAL_DEST_DIRECTORY='landing-zone'
        ```

    - Customization:

        ```shell
        export FILE_TO_CUSTOMIZE='landing-zone/setters.yaml'
        ```

### Complete the Infra Repo

The packages are now added and customized in the `tier1-infra` repo, it's time to hydrate and publish them.

1. Generate hydrated files, follow step 3 "Hydrate" of [Changing.md](./Changing.md).

1. Publish changes to repository, follow step 4 "Publish" of [Changing.md](./Changing.md).

1. Once the PR is merged, note the new tag version or commit SHA.  It will be required in the next section.

## 3. Synchronize the Landing Zone

Your landing zone is now built and published in the `tier1-infra` repo, but Config Sync is not aware yet.  This is where the `gcp-tier1-configsync` repo comes in to fill the gap by creating a new Root Sync.

1. Follow step 1-4 of [Changing.md](./Changing.md) to **add** the tier1 Root Sync package (step 2A).  If the package already exists from bootstrapping other environments, **modify** it for the new environment (step 2B).
    - Package details:

        ```shell
        export REPO_URI='TODO: publish package'

        export PKG_PATH='TODO: publish package'

        # the version to get, located in the package's CHANGELOG.md
        export VERSION=''

        export LOCAL_DEST_DIRECTORY='tier1-root-sync'
        ```

    - Customization: you will need to customize 2 files.
        > **!!! IMPORTANT !!!** Only customize for the environment your are bootstrapping.

        - `tier1-root-sync/setters.yaml`: general settings to create the new Root Sync.  Some values to customize are:
            - `env`: the environment you are bootstrapping.
            - `repo-url`: the URL of your `tier1-infra` repo.
            - `repo-dir`: the `deploy/<env>` folder to observe.
        - `tier1-root-sync/setters-version.yaml`: setting to control the version of `tier1-infra` to observe.
            - `version`: the new tag or commit SHA noted earlier.

1. Once your PR is merged, Config Sync will pick up the new Root Sync to create.  This new Root Sync will in turn pick up all the resources you have built and hydrated earlier in the `tier1-infra/deploy/<env>` directory.

## 4. Validate the landing zone deployment

Perform step 4 from this [procedure](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/docs/landing-zone-v2/README.md#4-validate-the-landing-zone-deployment).

You should now see two Root Syncs:

- `root-sync`: to your `...gcp-tier1-configsync/deploy/<env>@main` repo containing 1 rootsync resource which defines...
- the root sync to your `...tier1-infra/deploy/<env>@<version>` repo containing the landing zone resources.

## 5. Perform the post-deployment steps

Perform step 5 from this [procedure](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/docs/landing-zone-v2/README.md#5-perform-the-post-deployment-steps).

## THE END

Congratulations! You have completed the deployment of your landing zone as per SSC implementation.
