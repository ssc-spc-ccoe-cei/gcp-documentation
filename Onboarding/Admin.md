# Platform or Security Admin Onboarding

## Pre-requisite

1. locally clone the experimentation landing zone repo
1. create a branch of main

## Add admin folder(s) to the landing zone repository

1. Move into the landing folder

    ```shell
    cd source-base
    ```

1. get the hierarchy/admin-experimentation package

      ```shell
      kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/hierarchy/admin-experimentation@main ./landing-zone/hierarchy/tests/admins/<admin name>
      ```

1. To modify any of the files in this package (like setters.yaml) follow this generic guidance

    Refer to the `Add a Package` section of the [Changing.md](Changing.md)

1. Generate hydrated files

    Refer to the `Hydrate` section of the [Changing.md](Changing.md)

1. Add changes to repository

    Refer to the `Publish` section of the [Changing.md](Changing.md)
