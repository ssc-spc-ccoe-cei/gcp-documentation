# Application Onboarding

- [Application Onboarding](#application-onboarding)
  - [Required Information](#required-information)
    - [Common](#common)
    - [Experimentation](#experimentation)
    - [Dev, PreProd, Prod](#dev-preprod-prod)
  - [1. Create `tier34` Monorepo](#1-create-tier34-monorepo)
  - [2. Build the Application's Project](#2-build-the-applications-project)
    - [Package Details](#package-details)
  - [THE END](#the-end)

--------------------------------------

## Required Information

### Common

1. Naming convention for project-id : `<client-code><environment-code><region-code><data-classification>`-`<project-owner>`-`<user defined string>`

    *Notice the 2 "-" before and after `<project-owner>`
    - client-code (2 characters)
    - environment-code (1 character)
    - region-code (1 character) : "m" projects are always a global/multi-region resource
    - data-classification (1 character): "u" or "a" or "b"
    - project-owner: string (**total project-id string length cannot exceed 30 characters**)
    - user defined string: string (**total project-id string length cannot exceed 30 characters**):

1. Billing Account ID to be associated with this project

### Experimentation

1. user, group or serviceAccount email with editor role at project level

    **user or group has to exist in a Google Cloud Identity (any existing domain)**

### Dev, PreProd, Prod

1. Host Project ID (with a Shared VPC) that exist to connect this workload/service project

## 1. Create `tier34` Monorepo

- For Experimentation, you do not require this step as all packages are deploy in a single monorepo `gcp-experimentation-tier1`.

- For Dev, PreProd and Prod, follow the "Create New Deployment Monorepo" section in [Repositories.md](./Repositories.md) to create one `gcp-<x-project-id>-tier34` monorepos.

> **!!! You need to replace the environment-code of the project-id with character "x" as this repo will contain the configuration for all environments. !!!**

## 2. Build the Application's Project

You will build the application's project by adding packages to the `tier2` monorepo.

At a high level, the process below needs to be completed for each package :

1. Setup your change, follow step 1 of [Changing.md](./Changing.md#step-1---setup)
1. Add a Package, follow step 2A of [Changing.md](./Changing.md#a-add-a-package)
1. Generate hydrated files, follow step 3 of [Changing.md](./Changing.md#step-3---hydrate).
1. Publish changes to repository, follow step 4 of [Changing.md](./Changing.md#step-4---publish).
1. Once the PR is merged, note the new tag version or commit SHA.  It will be required in the next section.
1. Synchronize and promote configuration, follow step 5 of [Changing.md](./Changing.md#step-5---synchronize--promote-configs).

### Package Details

> **!!! It's important that all of the steps listed above are completed for each package before proceeding with the next package. !!!**

1. The client project setup package
    - For Experimentation, you do not require this package.

    - For Dev, PreProd and Prod, you deploy this [package](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/client-project-setup) inside the `gcp-<client-name>-tier2` repo.

      - Package details:

          ```shell
          export TIER='tier2'

          export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

          export PKG_PATH='solutions/client-project-setup'

          # the version to get, located in the package's CHANGELOG.md, use 'main' if not available'
          export VERSION=''

          # replace <x-project-id> with project-id with character "x" as the environment code
          export LOCAL_DEST_DIRECTORY='projects/<x-project-id>/client-project-setup'
          ```

      - Customization:

          ```shell
          # replace <x-project-id> with project-id with character "x" as the environment code
          export FILE_TO_CUSTOMIZE='projects/<x-project-id>/client-project-setup/setters.yaml'
          ```

1. The client project package:

    - For Experimentation, you deploy this [package](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/experimentation/client-project) inside the `gcp-experimentation-tier1` repo.

      - Package details:

        ```shell
        export TIER='tier1'

        export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

        export PKG_PATH='solutions/experimentation/client-project'

        # the version to get, located in the package's CHANGELOG.md, use 'main' if not available'
        export VERSION=''

        export LOCAL_DEST_DIRECTORY='projects/<project-id>/client-project'
        ```

      - Customization:

          ```shell
          export FILE_TO_CUSTOMIZE='projects/<project-id>/client-project/setters.yaml'
          ```

    - For Dev, PreProd and Prod,  you do not require this package.

## THE END

Congratulations! You have completed the deployment of a client's project as per SSC implementation.
