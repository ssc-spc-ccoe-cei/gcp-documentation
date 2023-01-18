# Workload Onboarding

## Required Information
### Common

1. Naming Convention for project-id : "\<tenant-code>\<environment-code>\<region-code>\<data-classification>-\<project-owner>-\<user defined string>"
    - tenant-code (2 characters)
    - environment-code (1 character)
    - region-code (1 character) : "m" projects are always a global/multi-region resource
    - data-classification (1 character): "u" or "a" or "b"
    - project-owner (**total project-id string lenght cannot exceed 30 characters**)
    - user defined string (**total project-id string lenght cannot exceed 30 characters**):
    
1. Billing Account ID to be associated with this project

### Sandbox

  1. user, group or serviceAccount with editor role at project level
  
      **user or group has to exist in a Google Cloud Identity (any existing domain)**


### DEV, UAT, PROD

  1. TBD

## Pre-requisite

1. locally clone the landing zone repo for this environment
1. create a branch of main

## Add tenant's workload to the landing zone repository

1. Move into source-base folder
    ```
    cd source-base
    ```
1. Get the workload package
    *Some folders may need to be created beforehand like `<data classification>`
    - Sandbox
      ```
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/project/project-sandbox@main ./landing-zone/hierarchy/Tenants/<tenant name>/Workloads/<project-id>
      ```

    - DEV, UAT, PROD
      ```
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/project/project-sandbox@main ./landing-zone/hierarchy/Tenants/<tenant name>/Workloads/<data classification>/<project-id>
      ```

1. To modify any of the files in this package (like setters.yaml) follow this generic guidance
  
    Refer to the `Make Code Changes` section of the [Changing.md](../Landing%20Zone%20Operations/Changing.md#Make%20code%20changes)

1. Generate hydrated files

    Refer to the `Generate hydrated files` section of the [Changing.md](../Landing%20Zone%20Operations/Changing.md#Generate%20hydrated%20files)

1. Add changes to repository
    
    Refer to the `Add changes to repository` section of the [Changing.md](../Landing%20Zone%20Operations/Changing.md#Add%20changes%20to%20repository)