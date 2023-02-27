# Landing zone building

> TODO: deprecate this documentation in favor of automation

Building the landing zone for Shared Services Canada is very similar to the process described for the [landing-zone-v2](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/solutions/landing-zone-v2/README.md#Organization) solution in the Google's Pubsec Toolkit repo.

This document will highlight the discrepancies instead of repeating the full process.

**We recommend that you read the entire procedure before initiating it**

**Important** SSC is using Azure Devops Repositories and Pipelines as it's git solution.

# [Organization](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/solutions/landing-zone-v2/README.md#Organization)
Shared Services Canada uses the "Multiple GCP organizations" achitecture.

# Setup
## 1. Complete the boostrap procedure
- Step: Create firewall rules
    
    make sure you implement the allow-egress-azure rule

## 2. Create your landing zone
- Step : Customize Packages
    
    **DO NOT customize** the packages

## 3. Deploy the infrastructure using either kpt or gitops-git or gitops-oci
****This procedure completely overrides the procedure on Google's pubsec repo.**

As you saw in the [Gitops](../Architecture/Repository%20Structure.md#Gitops) diagram, the ConfigSync operator requires an Infra repo and a ConfigSync repo. To do so, we implement a [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git) deployment.

### Infra repo
This section will implement the Infra repo
1. Create a new `tier1-infra` repo in your Azure Devops project
    - Experimentation
          
        repo name = `gcp-experimentation-tier1-infra`
    - DEV, PREPROD, PROD

        repo name = `gcp-tier1-infra`
1. Clone locally the `gcp-repo-template` in order to build the new `tier1-infra` repo
    - Experimentation
      ```bash
      git clone https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git gcp-experimentation-tier1-infra
      ```
    
    - DEV, PREPROD, PROD
        ```bash
        git clone https://github.com/ssc-spc-ccoe-cei/gcp-repo-template.git gcp-tier1-infra
        ```
1. Move into the new folder corresponding to that repo
    - Experimentation
        ```bash
        cd gcp-experimentation-tier1-infra
        ```
    - DEV, PREPROD, PROD
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
1. Generate and store a `.env` file

    In order to keep track of the variables that were used when building this landing zone, we recommend that you generate a `.env` file that includes the variables and values that were used when executing the `## 1. Complete the boostrap procedure`

    Generate that file and copy it into this `tier1-infra` repo under the folder `bootstrap/<env>`

1. Copy the landing-zone folder created when performing `2. Create your landing zone` to this new repo/source-base.
    ```bash
    copy -R ../landing-zone source-base
    ```
1. Customize the landing zone package and all of it subpackages
    
    Refer to the `Add a Package` section of the [Changing.md](Changing.md)

1. Generate hydrated files

    Refer to the `Hydrate` section of the [Changing.md](Changing.md)

1. Add changes to repository
    
    Refer to the `Publish` section of the [Changing.md](Changing.md).
    
    **You will need to push to main when running git push**
    
    `git push --set-upstream origin main`

1. Excellent ! You have completed your `tier1-infra` repository

### ConfigSync
The ConfigSync operator requires a ConfigSync repo to identify what version of the `tier1-infra` it should observe. To implement this, we will :
1. Create a ConfigSync repo.
2. Create a git-creds secret.
3. Deploy a root-sync to config controller.

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
    
    Refer to the `Add a Package` section of the [Changing.md](Changing.md)

1. Generate hydrated files

    Refer to the `Hydrate` section of the [Changing.md](Changing.md)

1. Add changes to repository
    
    Refer to the `Publish` section of the [Changing.md](Changing.md).
    
    **You will need to push to main when running git push**
    
    `git push --set-upstream origin main`

1. Excellent ! You have completed your `tier1-configsync` repository

### 2. Create a git-creds secret.
Now that the `tier1-configsync` repo is ready, we need to tell the ConfigSync operator to observe it. In this step, you will create a kubernetes secret that grants "code read" permission on `tier1-infra` and `tier1-configsync` repo.
1. Ensure your envrionment still have these variables define
    ```
    echo $GIT_USERNAME
    echo $TOKEN
    ```
    If not, run this
    ```
    export GIT_USERNAME=<git username> # For Azure Devops, this is the name of the Organization
    export TOKEN=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    ```

1. Create `git-creds` secret with the required value to access the git repositories    
    ```bash
    kubectl create secret generic git-creds --namespace="config-management-system" --from-literal=username=${GIT_USERNAME} --from-literal=token=${TOKEN}
    ```

### 3. Deploy a root-sync to config controller.
Now let's deploy that first root-sync that will make the magic happen and get your landing zone deployed in GCP.

1. Configure the `RootSync` object

    Create a file using the example below
    ```yaml
    cat <<EOF>> root-sync.yaml
    apiVersion: configsync.gke.io/v1beta1
    kind: RootSync
    metadata:
      name: root-sync
      namespace: config-management-system
    spec:
      sourceFormat: unstructured
      git:
        repo: <REPO_URL> # This is the url to your gcp-tier1-configsync repo
        branch: main
        dir: deploy/<env> # env should be replaced by experimentation or dev or preprod or prod depending on what landing zone you are building
        revision: HEAD
        auth: token
        secretRef:
          name: git-creds
    EOF
    ```

1. Deploy the Config Sync Manifest
    ```bash
    kubectl apply -f root-sync.yaml
    kubectl wait --for condition=established --timeout=10s crd/rootsyncs.configsync.gke.io
    ```
1. Store root-sync.yaml file in `tier1-infra` repo

    In order to keep track of that root-sync.yaml manual deployment that was used when building this landing zone, we recommend that you store this file into the `tier1-infra` repo under the folder `bootstrap/<env>`. This file will not be used in any way, it is just a reference.

# 4. Validate the landing zone deployment
Perform this procedure as described

# 5. Perform the post-deployment steps
Perform this procedure as described

# THE END
Congratulations! You have completed the deployment of your landing zone as per SSC implementation.