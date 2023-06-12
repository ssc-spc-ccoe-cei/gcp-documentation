# Platform or Security Admin Onboarding

- [Platform or Security Admin Onboarding](#platform-or-security-admin-onboarding)
  - [Required Information](#required-information)
  - [1. Build the Admin Folder](#1-build-the-admin-folder)
    - [Package Details](#package-details)
  - [THE END](#the-end)

--------------------------------------

## Required Information

1. The admin name

1. The Group or User email to grant permission on the admin folder

## 1. Build the Admin Folder

This package creates a folder in GCP and grant admin privileges on that folder to a user. This user can use this folder to experiment solutions in GCP.

You will build this admin folder by adding a package to the `gcp-experimentation-tier1` monorepo.

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
    - For Experimentation, you deploy this [package](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/experimentation/admin-folder) inside the `gcp-experimentation-tier1` repo.

      - Package details:

          ```shell
          export TIER='tier1'

          export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

          export PKG_PATH='solutions/experimentation/admin-folder'

          # the version to get, located in the package's CHANGELOG.md, use 'main' if not available'
          export VERSION=''

          # replace <admin-name> with the name of the admin
          export LOCAL_DEST_DIRECTORY='admins/<admin-name>'
          ```

      - Customization:

          ```shell
          # replace <admin-name> with the name of the admin
          export FILE_TO_CUSTOMIZE='admins/<admin-name>/admin-folder/setters.yaml'
          ```

    - For Dev, PreProd and Prod,  you do not deploy this package.

## THE END

Congratulations! You have completed the deployment of an admin folder as per SSC implementation.
