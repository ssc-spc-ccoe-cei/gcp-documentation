# Tenant Onboarding

### Pre-requisite
1. locally clone the landing zone repo for this environment
1. create a branch of main

## Add tenant folder(s) to the landing zone repository

1. Move into source-base folder
    ```
    cd source-base
    ```
1. get the hierarchy/tenant package
    - Sandbox
      ```
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/hierarchy/tenant-sandbox@main ./landing-zone/hierarchy/Workloads/<tenant name>
      ```

    - DEV, UAT, PROD
      ```
      # Automation
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/hierarchy/tenant-env@main ./landing-zone/hierarchy/Automation/<tenant name>

      # Workloads
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/hierarchy/tenant-env@main ./landing-zone/hierarchy/Workloads/<tenant name>

      # Workloads-Infrastructure
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/hierarchy/tenant-env@main ./landing-zone/hierarchy/Networking/Workloads-Infrastructure/<tenant name>
      ```

1. Add tenant Host Project(s) and Shared VPC
    - Sandbox
      
      n/a

    - DEV, UAT, PROD

       TODO: complete this steps

1. To modify any of the files in these packages (like setters.yaml) follow this generic guidance
  
    Refer to the `Make Code Changes` section of the ![Changing.md](Changing.md)

1. Generate hydrated files

    Refer to the `Generate hydrated files` section of the ![Changing.md](Changing.md)

1. Add changes to repository
    
    Refer to the `Add changes to repository` section of the ![Changing.md](Changing.md).


## Add tenant Tier2-ConfigSync
TODO: complete this steps

