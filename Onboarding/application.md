# Workload Onboarding

## Required Information
### Common

1. Naming convention for project-id : `<client-code><environment-code><region-code><data-classification>`-`<project-owner>`-`<user defined string>`

    *Notice the 2 "-" before and after `<project-owner>`
    - client-code (2 characters)
    - environment-code (1 character)
    - region-code (1 character) : "m" projects are always a global/multi-region resource
    - data-classification (1 character): "u" or "a" or "b"
    - project-owner: string (**total project-id string lenght cannot exceed 30 characters**)
    - user defined string: string (**total project-id string lenght cannot exceed 30 characters**):
    
1. Billing Account ID to be associated with this project

### Experimentation

1. user, group or serviceAccount with editor role at project level
  
    **user or group has to exist in a Google Cloud Identity (any existing domain)**


### DEV, UAT, PROD (UNDER CONSTRUCTION)

1. Host Project ID (with a Shared VPC) that exist to connect this workload/service project

## Pre-requisite

1. Ensure service account `projects-sa` has been granted `Billing Account User` role on the Billing Account.
1. Locally clone the landing zone repo for this environment
1. Create a branch from main

## Add client's application to the landing zone repository

1. Move into source-base folder
    ```
    cd source-base
    ```
1. Get the workload package
    
    *Some folders may need to be created beforehand like `<data classification>`
    - Experimentation
      ```
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/project/project-experimentation@main ./landing-zone/hierarchy/clients/<client name>/applications/<project-id>
      ```

    - DEV, UAT, PROD
      ```
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/project/project-experimentation@main ./landing-zone/hierarchy/clients/<client name>/applications/<data classification>/<project-id>
      ```

1. To modify any of the files in this package (like setters.yaml) follow this generic guidance
  
    Refer to the `Add a Package` section of the [Changing.md](Changing.md)

1. Generate hydrated files

    Refer to the `Hydrate` section of the [Changing.md](Changing.md)

1. Add changes to repository
    
    Refer to the `Publish` section of the [Changing.md](Changing.md)