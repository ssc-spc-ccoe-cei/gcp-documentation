# Landing zone building

Building the landing zone for Shared Services Canada is very similar to the process described for the [landing-zone-v2](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/solutions/landing-zone-v2/README.md#Organization) solution in the Google's Pubsec Toolkit repo.

This document will describe how automated scripts can be used for building.

**We recommend that you read the entire procedure before initiating it**

**Important** SSC is using Azure Devops Repositories and Pipelines as it's git solution.

# [Organization](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/solutions/landing-zone-v2/README.md#Organization)
Shared Services Canada uses the "Multiple GCP organizations" achitecture.

# Setup
## 1. Repo Setup

As you saw in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator requires an Infra repo and a ConfigSync repo. To do so, we implement a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.

### Infra repo
This section will implement the Infra repo
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

1. Excellent ! You have your `tier1-infra` repository ready for further steps.

### ConfigSync
The ConfigSync operator requires a ConfigSync repo to identify what version of the `tier1-infra` it should observe. To implement this, we will :

1. Create a ConfigSync repo.


### 1. Create a ConfigSync repo.
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
    
    Refer to the `Make Code Changes` section of the ![Changing.md](Changing.md#Make%20code%20changes)

1. Generate hydrated files

    Refer to the `Generate hydrated files` section of the ![Changing.md](Changing.md#Generate%20hydrated%20files)

1. Add changes to repository
    
    Refer to the `Add changes to repository` section of the ![Changing.md](Changing.md#Add%20changes%20to%20repository).
    
    **You will need to push to main when running git push**
    
    `git push --set-upstream origin main`

1. Excellent ! You have your `tier1-configsync` repository for further steps.



## 2. Get the GCP-Tools Repo
- Step: Clone the gcp-tools github repo
    
    git clone  https://github.com/ssc-spc-ccoe-cei/gcp-tools.git

## 3. Create the .env file
- Step: There is an example.env file in the gcp-tools repo , the same should be used to create a local .env file with appropriate inputs that will be used by setup-kcc script to create the KCC cluster and project.

## 4. Running the setup-kcc bootstrap procedure
- Step: There are two ways to run the setup-kcc bash script.
If we want to use the local .env file as it is we can run:

```
bash setup-kcc.sh local
```

The command above will use the local .env file to create project , FW settings , Clour router , PSC endpoint and the Kcc config controller cluster. It will also create a root-sync.yaml file.

At the end we need to copy the two files `.env` and `root-sync.yaml` copy it into this `tier1-infra` repo under the folder `bootstrap/<env>`

If we check in the .env file in the tier1-Infra-REPO (Or a .env file alredy exists in the repo) , then we can use:

```
bash setup-kcc.sh <ENV> <tier1-infra-REPO-URL>

<ENV> is dev/uat/prod (ENV name inside the bootstrap folder in tier1 Infra repo)
```


In this case the .env file from the ENV specific folder from tier1-infra-REPO gets used to spin the Kcc resources.
At the end we still will need to check in the  `root-sync.yaml` file to the `tier1-infra` repo


# 5. Create your landing zone

## Fetch the packages

You will be running the following commands from a linux machine. GCP Cloud Shell can be used to serve that purpose.


We will be using kpt to obtain the packages. For more information on the `kpt get` command, please refer to this link : https://kpt.dev/reference/cli/pkg/get/

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
1. etc.
    
1. Customize Packages

    Review and customize all packages `setters.yaml` with the unique configuration of your landing zone. 


1. Clone the `tier1-infra` repo created earlier or cd into that repo if already cloned
    ```bash
    git clone <REPO_URL>
    cd <REPO_FOLDER>
    ```
Copy the landing-zone folder created when performing `2. Create your landing zone` to this new repo/source-base.
    ```bash
    copy -R ../landing-zone source-base
    ```
1. Customize the landing zone package and all of it subpackages
    
    Refer to the `Make Code Changes` section of the [Changing.md](Changing.md#Make%20code%20changes)

1. Generate hydrated files

    Refer to the `Generate hydrated files` section of the [Changing.md](Changing.md#Generate%20hydrated%20files)

1. Add changes to repository
    
    Refer to the `Add changes to repository` section of the [Changing.md](Changing.md#Add%20changes%20to%20repository).
    
    **You will need to push to main when running git push**
    
    `git push --set-upstream origin main`

1. Excellent ! You have completed your `tier1-infra` repository
    

# 4. Validate the landing zone deployment
Perform this procedure as described

# 5. Perform the post-deployment steps
Perform this procedure as described

# THE END
Congratulations! You have completed the deployment of your landing zone as per SSC implementation.