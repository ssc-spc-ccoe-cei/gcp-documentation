# Landing zone building

Shared Services Canada has automated a portion of the deployment process for the [landing-zone-v2](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#Organization) solution in the Google's Pubsec Toolkit repo.

This document will describe how the automated scripts can be used for building a landing zone.

**We recommend that you read the entire procedure before initiating it**

**Important** SSC is using Azure Devops Repositories and Pipelines as its git solution.

# Organization
Shared Services Canada uses the "[Multiple GCP organizations](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#multiple-gcp-organizations)" achitecture.

# Setup
## 1. Repo Setup

SSC implements a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.
As illustrated in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator requires an Infra repo and a ConfigSync repo. 

The steps assume these repos have already been created by following the "Create New Deployment Repo" section in [Repositories.md](./Repositories.md):
- One of the two infra repos:
    - `gcp-sandbox-tier1-infra`: if building a sandbox landing-zone.
    - `gcp-tier1-infra`: if building a dev, uat or prod landing-zone.
- `gcp-tier1-configsync`: to identify which git revision of `tier1-infra` the Config Sync operator should observe.

1. Start a new change for your `tier1-infra` repo, refer to "Step 1 - The Setup" section of [Changing.md](./Changing.md).
    - Sandbox

        repo name = `gcp-sandbox-tier1-infra`
    - DEV, UAT, PROD

        repo name = `gcp-tier1-infra`
1. Your terminal should now be at the root of your `tier1-infra` repo, on a new branch.

## 2. Create the .env file

The automated script requires a `.env` file to deploy the environment.

From the root of your `tier1-infra` repo:

1. If it doesn't exist, create a `bootstrap/<env>` directory.  It will be used to backup files generated during bootstrapping.
1. Copy the example.env file from the `tools/scripts/bootstrap` folder.
    ```bash
    cp tools/scripts/bootstrap/.env.sample bootstrap/<ENV>/.env
    ```

2. **Important** Customize the new file with the approriate values for the landing zone you are building.

3. Export a `TOKEN` variable.  Set its value to the PAT which has read access to the repo.
    ```bash
    export TOKEN='xxxxxxxxxxxxxxx'
    ```

## 3. Run the setup-kcc automated script

1. The script creates a project, the FW settings, a Cloud router, a Cloud NAT, a private service connect endpoint and the Anthos Config Controller cluster. It also creates a root-sync.yaml file.
    ```
    bash tools/scripts/bootstrap/setup-kcc.sh <PATH TO .ENV FILE>
    ```

1. Once the script has completed, you need to move the `root-sync.yaml` file into the `bootstrap/<env>` folder of the `tier1-infra` repo.
    ```bash
    mv root-sync.yaml bootstrap/<ENV>
    ```
1. Validate the Config Controller deployment.  You should see that the `root-sync` is successfully synchronized and contains zero resource.
Perform this [procedure](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#4-validate-the-landing-zone-deployment) as described

## 5. Create your landing zone

We will be using kpt to obtain the packages. For more information on the `kpt get` command, please refer to this link : https://kpt.dev/reference/cli/pkg/get/

1. Move into `source-base` folder
    ```bash
    cd source-base
    ```
1. Get the landing zone package
    ```bash
    kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/landing-zone-v2@main ./landing-zone
    ```
1. Get the hierarchy package
    - Sandbox
      ```bash
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/hierarchy/core-sandbox@main ./landing-zone/hierarchy
      ```

    - DEV, UAT, PROD
      ```bash
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/hierarchy/core-env@main ./landing-zone/hierarchy
      ```
1. Get the gatekeeper policies package
    ```bash
    kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/gatekeeper-policies@main ./landing-zone/gatekeeper-policies
    ```
1. Get the organization policies package
    ```bash
    kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/org-policies@main ./landing-zone/org-policies
    ```
1. Get the logging package
    
    TODO: TBD
    
1. Customize the landing zone package and all of it subpackages
    
    Refer to the `Make Code Changes` section of the [Changing.md](Changing.md#Make%20code%20changes)

1. Generate hydrated files

    Refer to the `Generate hydrated files` section of the [Changing.md](Changing.md#Generate%20hydrated%20files)

1. Add changes to repository
    
    Refer to the `Add changes to repository` section of the [Changing.md](Changing.md#Add%20changes%20to%20repository).
    
    **You will need to push to main when running git push**
    
    `git push --set-upstream origin main` 

### ConfigSync Repo

1. Get the root-sync kpt package
    ```bash
    TODO: kpt pkg get URL ./tier1-root-sync
    ```
1. Customize the root-sync package
    
    Refer to the `Make Code Changes` section of the [Changing.md](Changing.md#Make%20code%20changes)

1. Generate hydrated files

    Refer to the `Generate hydrated files` section of the [Changing.md](Changing.md#Generate%20hydrated%20files)

1. Add changes to repository
    
    Refer to the `Add changes to repository` section of the [Changing.md](Changing.md#Add%20changes%20to%20repository).
    
    **You will need to push to main when running git push**
    
    `git push --set-upstream origin main`

1. Excellent ! You have your `tier1-configsync` repository ready for next steps..


# 6. Validate the landing zone deployment
Perform this [procedure](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#4-validate-the-landing-zone-deployment) as described

# 7. Perform the post-deployment steps
Perform this [procedure](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#5-perform-the-post-deployment-steps) as described

# THE END
Congratulations! You have completed the deployment of your landing zone as per SSC implementation.
