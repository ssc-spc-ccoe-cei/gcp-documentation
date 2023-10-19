# Intégration de la plateforme ou de l'administrateur de sécurité

- [Intégration de l'administrateur de la plateforme ou de la sécurité](#platform-or-security-admin-onboarding)
  - [Informations requises](#informations-requises)
  - [1. Créer le dossier administrateur](#1-build-the-admin-folder)
    - [Détails du forfait](#package-details)
  - [LA FIN](#la-fin)

--------------------------------------

## Information requise

1. Le nom de l'administrateur

2. L'e-mail du groupe ou de l'utilisateur pour accorder l'autorisation sur le dossier admin

## 1. Créez le dossier Admin

Ce package crée un dossier dans GCP et accorde des privilèges d'administrateur sur ce dossier à un utilisateur. Cet utilisateur peut utiliser ce dossier pour expérimenter des solutions dans GCP.

Vous allez créer ce dossier admin en ajoutant un package au monorepo `gcp-experimentation-tier1`.

À un niveau élevé, le processus ci-dessous doit être complété pour chaque package :

3. Configurez votre modification, suivez l'étape 1 de [Changing.md](./Changing.md#step-1---setup)
4. Ajoutez un package, suivez l'étape 2A de [Changing.md](./Changing.md#a-add-a-package)
5. Générez des fichiers hydratés, suivez l'étape 3 de [Changing.md](./Changing.md#step-3---hydrate).
6. Publiez les modifications dans le référentiel, suivez l'étape 4 de [Changing.md](./Changing.md#step-4---publish).
7. Une fois le PR fusionné, notez la nouvelle version de la balise ou validez SHA. Cela sera nécessaire dans la section suivante.
8. Synchronisez et promouvez la configuration, suivez l'étape 5 de [Changing.md](./Changing.md#step-5---synchronize--promote-configs).

### Détails du paquet

> **!!! Il est important que toutes les étapes répertoriées ci-dessus soient effectuées pour chaque package avant de passer au package suivant. !!!**

1. Le package de configuration du projet client
    - Pour l'expérimentation, vous déployez ce [package](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/experimentation/admin-folder) dans le dépôt `gcp-experimentation-tier1`.

      - Détails du package (lors de l'exécution de [étape 2A](../Landing%20Zone%20Operations/Changing.md#a-add-a-package)) :

          ```shell
          export TIER='tier1'

          export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

          export PKG_PATH='solutions/experimentation/admin-folder'

          # the version to get, located in the package's CHANGELOG.md, use 'main' if not available'
          export VERSION=''

          # replace <admin-name> with the name of the admin
          export LOCAL_DEST_DIRECTORY='admins/<admin-name>'
          ```

      - Personnalisation:

          ```shell
          # replace <admin-name> with the name of the admin
          export FILE_TO_CUSTOMIZE='admins/<admin-name>/admin-folder/setters.yaml'
          ```

- Pour Dev, PreProd et Prod, vous ne déployez pas ce package.

## LA FIN

Toutes nos félicitations! Vous avez terminé le déploiement d'un dossier admin conformément à la mise en œuvre de SSC.
