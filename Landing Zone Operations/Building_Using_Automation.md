# Landing zone building

Shared Services Canada has automated a portion of the deployment process for the [landing-zone-v2](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/solutions/landing-zone-v2/README.md#Organization) solution in the Google's Pubsec Toolkit repo.

This document will describe how the automated scripts can be used for building a landing zone.

**We recommend that you read the entire procedure before initiating it**

**Important** SSC is using Azure Devops Repositories and Pipelines as it's git solution.

# Organization
Shared Services Canada uses the "[Multiple GCP organizations](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#multiple-gcp-organizations)" achitecture.

# Setup
## 1. Repo Setup

SSC implements a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.
As illustrated in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator requires an Infra repo and a ConfigSync repo. 

### ConfigSync Repo
The ConfigSync operator requires a ConfigSync repo to identify what version of the `tier1-infra` it should observe.

1. Create a new `gcp-tier1-configsync` repo in your Azure Devops project

1. Clone locally the repository template repo in order to build the new `gcp-tier1-configsync`
    ```bash
    git clone https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git gcp-tier1-configsync
    ```
1. Move into the new folder corresponding to that repo
    ```bash
    cd gcp-tier1-configsync
    ```
1. Change repo upstream
    ```bash
    # Remove the origin pointing to the template repo
    git remote remove origin
    # Add the new origin with your repo url
    git remote add origin <YOUR NEW REPO URL>
    ```  
1. Move into the `source-base` folder
    ```bash
    cd source-base
    ```
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


### Infra repo

Let's create the other repo observed by ConfigSync: `tier1-infra`.

1. Create a new `tier1-infra` repo in your Azure Devops project
    - Sandbox
          
        repo name = `gcp-sandbox-tier1-infra`
    - DEV, UAT, PROD

        repo name = `gcp-tier1-infra`
1. Clone locally the `gcp-repo-template` in order to build the new `tier1-infra` repo
    - Sandbox
      ```bash
      git clone https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git gcp-sandbox-tier1-infra
      ```
    
    - DEV, UAT, PROD
        ```bash
        git clone https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git gcp-tier1-infra
        ```
1. Move into the new folder corresponding to that repo
    - Sandbox
        ```bash
        cd gcp-sandbox-tier1-infra
        ```
    - DEV, UAT, PROD
        ```bash
        cd gcp-tier1-infra
        ```
1. Change repo upstream
    ```bash
    # Remove the origin pointing to the template repo
    git remote remove origin
    # Add the new origin with your repo url
    git remote add origin <YOUR NEW REPO URL>
    ```
    **You will need to push to main when running git push**
    
    `git push --set-upstream origin main`

1. Excellent ! You have your `tier1-infra` repository ready for next steps.


## 2. Pull the tools submodule

Populate the `tools` folder with the content from the `gcp-tools` repository. Ensure you are still at the root of the `tier1-infra` repo and execute `modupdate.sh`
    ```bash
    bash modupdate.sh
    ```
    
## 3. Create the .env file

The automated script requires a `.env` file to deploy the environment.

1. Copy the example.env file from the `tools/scripts/bootstrap` folder.
    ```bash
    cp tools/scripts/bootstrap/.env.sample bootstrap/<ENV>/.env
    ```

2. **Important** Customize the new file with the approriate values for the landing zone you are building.

## 4. Running the setup-kcc automated script

The script creates a project, the FW settings, a Cloud router, a Cloud NAT, a private service connect endpoint and the Anthos Config Controller cluster. It also creates a root-sync.yaml file.
    ```
    bash setup-kcc.sh <PATH TO .ENV FILE>
    ```

Once the script has completed, you need to move the `root-sync.yaml` file into the `bootstrap/<env>` folder of the `tier1-infra` repo.
    ```bash
    mv root-sync.yaml bootstrap/<ENV>
    ```

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

# 6. Validate the landing zone deployment
Perform this [procedure](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#4-validate-the-landing-zone-deployment) as described

# 7. Perform the post-deployment steps
Perform this [procedure](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/landing-zone-v2/README.md#5-perform-the-post-deployment-steps) as described

# THE END
Congratulations! You have completed the deployment of your landing zone as per SSC implementation.
