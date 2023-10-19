# Intégration des applications

- [Intégration d'application](#application-onboarding)
  - [Informations requises](#informations-requises)
    - [Commun](#commun)
    - [Expérimentation](#expérimentation)
    - [Dev, PréProd, Prod](#dev-preprod-prod)
  - [1. Créer un Monorepo `tier34`](#1-create-tier34-monorepo)
  - [2. Construire le projet de l'application](#2-build-the-applications-project)
    - [Détails du forfait](#package-details)
  - [LA FIN](#la-fin)

--------------------------------------

## Information requise

### Commun

1. Convention de dénomination pour l'ID du projet : `<code-client><code-environnement><code-région><classification-données>`-`<propriétaire du projet>`-`<chaîne définie par l'utilisateur>`

    *Remarquez les 2 "-" avant et après `<propriétaire du projet>`
    - code client (2 caractères)
    - code-environnement (1 caractère) : "e" - expérimentation, "d" - dev, "u" - preprod, "p" - prod
    - code-région (1 caractère) : les projets "m" sont toujours une ressource globale/multi-région
    - classification des données (1 caractère) : "u" ou "a" ou "b"
    - propriétaire du projet : chaîne (**la longueur totale de la chaîne d'identification du projet ne peut pas dépasser 30 caractères**)
    - chaîne définie par l'utilisateur : chaîne (**la longueur totale de la chaîne d'identification du projet ne peut pas dépasser 30 caractères**) :

2. ID du compte de facturation à associer à ce projet

### Expérimentation

1. utilisateur, groupe ou de serviceAccount avec rôle d'éditeur au niveau du projet

    **L'utilisateur ou le groupe doit exister dans une identité Google Cloud (n'importe quel domaine existant)**

### Développeur, PréProd, Prod

1. ID de projet hôte (avec un VPC partagé) existant pour connecter ce projet de charge de travail/service

## 1. Créer un Monorepo `tier34`

- Pour l'expérimentation, vous n'avez pas besoin de cette étape car tous les packages sont déployés dans un seul monorepo `gcp-experimentation-tier1`.

- Pour Dev, PreProd et Prod, suivez la section "Créer un nouveau monorepo de déploiement" dans [Repositories.md](../Landing%20Zone%20Operations/Repositories.md) pour créer un `gcp-<x-project-id> -tier34` monodépôt.

> **!!! Vous devez remplacer le code d'environnement de l'ID du projet par le caractère "x" car ce dépôt contiendra la configuration de tous les environnements. Cela s'applique uniquement au nom du dépôt. N'utilisez pas x comme code d'environnement pour votre identifiant de projet dans `setters.yaml` car cela déclencherait une erreur de validation Gatekeeper : `le webhook d'admission "validation.gatekeeper.sh" a refusé la demande` !!!**

## 2. Construire le projet de l'application

Vous construirez le projet de l'application en ajoutant des packages au monorepo `tier2`.

À un niveau élevé, le processus ci-dessous doit être complété pour chaque package :

1. Configurez votre modification, suivez l'étape 1 de [Changing.md](./Changing.md#step-1---setup)
2. Ajoutez un package, suivez l'étape 2A de [Changing.md](./Changing.md#a-add-a-package)
3. Générez des fichiers hydratés, suivez l'étape 3 de [Changing.md](./Changing.md#step-3---hydrate).
4. Publiez les modifications dans le référentiel, suivez l'étape 4 de [Changing.md](./Changing.md#step-4---publish).
5. Une fois le PR fusionné, notez la nouvelle version de la balise ou validez SHA. Cela sera nécessaire dans la section suivante.
6. Synchronisez et promouvez la configuration, suivez l'étape 5 de [Changing.md](./Changing.md#step-5---synchronize--promote-configs).

### Détails du forfait

> **!!! Il est important que toutes les étapes répertoriées ci-dessus soient effectuées pour chaque package avant de passer au package suivant. !!!**

1. Le package de configuration du projet client
    - Pour l'expérimentation, vous n'avez pas besoin de ce package.

    - Pour Dev, PreProd et Prod, vous déployez ce [package](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/client-project-setup) dans le `gcp-<client -name>-tier2` dépôt.

      - Détails du package (lors de l'exécution de [étape 2A](../Landing%20Zone%20Operations/Changing.md#a-add-a-package)) :

          ```shell
          exporter TIER='tier2'

          export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

          export PKG_PATH='solutions/client-project-setup'

          # la version à obtenir, située dans le package CHANGELOG.md, utiliser 'main' si non disponible
          exporter VERSION=''

          # remplacez <x-project-id> par project-id avec le caractère "x" comme code d'environnement
          export LOCAL_DEST_DIRECTORY='projects/<x-project-id>'
          ```

          > **!!! Vous devez remplacer le code d'environnement de l'ID du projet par le caractère "x" car ce dossier contiendra la configuration de tous les environnements. Cela s'applique uniquement au nom du dossier. N'utilisez pas x comme code d'environnement pour votre identifiant de projet dans `setters.yaml` car cela déclencherait une erreur de validation Gatekeeper : `le webhook d'admission "validation.gatekeeper.sh" a refusé la demande` !!!**

      - Personnalisation :

          ```shell
          # remplacez <x-project-id> par project-id avec le caractère "x" comme code d'environnement
          export FILE_TO_CUSTOMIZE='projects/<x-project-id>/client-project-setup/setters.yaml'
          ```

2. Le package du projet client :

    - Pour l'expérimentation, vous déployez ce [package](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/experimentation/client-project) dans le dépôt `gcp-experimentation-tier1`.

      - Détails du package (lors de l'exécution de [étape 2A](../Landing%20Zone%20Operations/Changing.md#a-add-a-package)) :

        ```shell
        exporter TIER='tier1'

        export REPO_URI='https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'

        export PKG_PATH='solutions/expérimentation/projet-client'

        # la version à obtenir, située dans le package CHANGELOG.md, utiliser 'main' si non disponible
        exporter VERSION=''

        export LOCAL_DEST_DIRECTORY='projects/<project-id>'
        ```

      - Personnalisation :

          ```shell
          export FILE_TO_CUSTOMIZE='projects/<project-id>/client-project/setters.yaml'
          ```

    - Pour Dev, PreProd et Prod, vous n'avez pas besoin de ce package.

3. Secrets de synchronisation des dépôts
   - Pour l'Expérimentation, vous n'avez pas besoin d'exécuter cette étape.

   - Pour Dev, PreProd et Prod
       - Vous devez maintenant ajouter le secret `git-creds` aux espaces de noms `<project-id>-tier3` et `<project-id>-tier4` qui ont été créés par le package de configuration du projet client. Ce secret doit autoriser l'accès en lecture au monorepo Azure DevOps `tier34`. Il est utilisé par l'opérateur configsync.
          > **!!! Les secrets doivent être renseignés sur chaque cluster de contrôle de configuration (Dev, PreProd et Prod) !!!**

        1. Exportez les variables nécessaires

            ```shell
            export NAMESPACE=<id-projet>-tier3
            export GIT_USERNAME=<git username> # Pour Azure Devops, il s'agit du nom de l'organisation
            exporter TOKEN=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
            ```

        2. Créez le secret `git-creds` avec la valeur requise pour accéder aux référentiels git

           ```shell
           kubectl crée des git-creds génériques secrets --namespace=${NAMESPACE} --from-literal=username=${GIT_USERNAME} --from-literal=token=${TOKEN}
           ```

        3. Répétez l'opération pour `<project-id>-tier4`

        4. Répétez pour tous les autres environnements

## LA FIN
Toutes nos félicitations! Vous avez terminé le déploiement du projet d'un client selon la mise en œuvre de SSC.
