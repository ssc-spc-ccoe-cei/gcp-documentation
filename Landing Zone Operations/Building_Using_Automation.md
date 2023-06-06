# Landing zone building

<!-- vscode-markdown-toc -->
* [Requirements](#Requirements)
* [1. Create `tier1` monorepo](#Createtier1monorepo)
* [2. Bootstrap the Config Controller Project](#BootstraptheConfigControllerProject)
* [3. Build the Core Landing Zone](#BuildtheCoreLandingZone)
	* [Package Details](#PackageDetails)
* [4. Perform the post-deployment steps](#Performthepost-deploymentsteps)
* [THE END](#THEEND)

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

Shared Services Canada has automated a portion of the deployment process for the [landing-zone-v2](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#Organization) solution in the Google's Pubsec Toolkit repo.

This document will describe how the automated scripts can be used for building a landing zone.

**We recommend that you read the entire procedure before initiating it**

**Important** SSC is using Azure Devops Repositories and Pipelines as its git solution.

## <a name='Requirements'></a>Requirements

Shared Services Canada uses the "[Multiple GCP organizations](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#multiple-gcp-organizations)" architecture.

Review the requirements listed [here](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#requirements).

## <a name='Createtier1monorepo'></a>1. Create `tier1` monorepo

SSC implements a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.
As illustrated in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator is observing our deployment monorepos.

Follow the "Create New Deployment Monorepo" section in [Repositories.md](./Repositories.md) to create one of the two `tier1` monorepos:

- `gcp-experimentation-tier1`: if building an experimentation landing-zone.
- `gcp-env-tier1`: if building a dev, preprod or prod landing-zone.


## <a name='BootstraptheConfigControllerProject'></a>2. Bootstrap the Config Controller Project

The automated script creates a project, the FW settings, a Cloud router, a Cloud NAT, a private service connect endpoint and the Anthos Config Controller cluster. It also creates a root-sync.yaml file.

The script requires a `.env` file to deploy the environment.

1. Start a new change for your `tier1` monorepo, follow the "Step 1 - Setup" section of [Changing.md](./Changing.md).
    - Experimentation

        monorepo name = `gcp-experimentation-tier1`
    - DEV, PREPROD, PROD

        monorepo name = `gcp-env-tier1`
1. Your terminal should now be at the root of your `tier1` monorepo, on a new branch and with the tools submodule populated.
1. If it doesn't exist, create a `bootstrap/<env>` directory.  It will be used to backup files generated during bootstrapping.
1. Copy the example.env file from the `tools/scripts/bootstrap` folder.

    ```shell
    cp tools/scripts/bootstrap/.env.sample bootstrap/<ENV>/.env
    ```

2. **Important** Customize the new file with the appropriate values for the landing zone you are building.

3. Export a `TOKEN` variable.  Set its value to the PAT which has read access to the `tier1` monorepo.

    ```shell
    export TOKEN='xxxxxxxxxxxxxxx'
    ```

1. Run the setup-kcc automated script:

    ```shell
    bash tools/scripts/bootstrap/setup-kcc.sh <PATH TO .ENV FILE>
    ```

1. Once the script has completed, you need to move the `root-sync.yaml` file into the `bootstrap/<env>` folder of the `tier1` monorepo.

    ```shell
    mv root-sync.yaml bootstrap/<ENV>
    ```

1. Validate the Config Controller deployment.  You should see a synchronized `root-sync` to your `...<tier1 monorepo>/csync/deploy/<env>@main` monorepo containing no resource.

    ```shell
    nomos status --contexts gke_${PROJECT_ID}_northamerica-northeast1_krmapihost-${CLUSTER}
    ```

## <a name='BuildtheCoreLandingZone'></a>3. Build the Core Landing Zone

We will build the core landing zone by adding a collection of packages to the `tier1` monorepo.
At a high level, the process below needs to be completed for each package :

1. Setup your change, follow step 1 of [Changing.md](./Changing.md#step-1---setup)
1. Add a Package, follow step 2A of [Changing.md](./Changing.md#a-add-a-package)
1. Generate hydrated files, follow step 3 of [Changing.md](./Changing.md#step-3---hydrate).
1. Publish changes to repository, follow step 4 of [Changing.md](./Changing.md#step-4---publish).
1. Once the PR is merged, note the new tag version or commit SHA.  It will be required in the next section.
1. Synchronize and promote configuration, follow step 5 of [Changing.md](./Changing.md#step-5---synchronize--promote-configs).

### <a name='PackageDetails'></a>Package Details

You will add the 2 packages below to your `tier1` monorepo.
> **!!! It's important that all of the steps listed above are completed for each package before proceeding with the next package. !!!**

The details below are required when performing step 2A "Add a Package" of [Changing.md](./Changing.md)

1. The gatekeeper-policies package:
    - Package details:

        ```shell
        export TIER='tier1'

        export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

        export PKG_PATH='solutions/gatekeeper-policies'

        # the version to get, located in the package's CHANGELOG.md, use 'main' if not available
        export VERSION=''

        export LOCAL_DEST_DIRECTORY='gatekeeper-policies'
        ```

    - Customization:

        ```shell
        export FILE_TO_CUSTOMIZE='gatekeeper-policies/naming-rules/project/setters.yaml'
        ```

1. The core landing zone package:
    - Package details:

      ```shell
      export TIER='tier1'

      export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

      # for experimentation
      export PKG_PATH='solutions/experimentation/core-landing-zone'
      # OR
      # for dev, preprod, prod
      export PKG_PATH='solutions/core-landing-zone'

      # the version to get, located in the package's CHANGELOG.md, use 'main' if not available
      export VERSION=''

      export LOCAL_DEST_DIRECTORY='core-landing-zone'
      ```

    - Customization:

        ```shell
        export FILE_TO_CUSTOMIZE='core-landing-zone/setters.yaml'
        ```

## <a name='Performthepost-deploymentsteps'></a>4. Perform the post-deployment steps

Some resources from the `core-landing-zone` package won't be able to deploy until the new `projects-sa` is granted `billing.user` role.

Perform step 5 from this [procedure](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#5-perform-the-post-deployment-steps) to fix this.

## <a name='THEEND'></a>THE END

Congratulations! You have completed the deployment of your core landing zone as per SSC implementation.
