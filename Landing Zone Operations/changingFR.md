# Implémenter un changement sur la zone d'accueil

- [Mise en œuvre d'un changement sur la zone d'accueil](#implementing-a-change-on-the-landing-zone)
  - [Étape 1 - Configuration](#étape-1---configuration)
  - [Étape 2 - Modifier](#étape-2---changer)
    - [A) Ajouter un package](#a-add-a-package)
    - [B) Modifier un package](#b-modify-a-package)
    - [C) Mettre à jour un package](#c-update-a-package)
    - [D) Supprimer un package](#d-remove-a-package)
  - [Étape 3 – Hydrater](#étape-3---hydrater)
  - [Étape 4 - Publier](#étape-4---publier)
  - [Étape 5 - Synchroniser/Promouvoir les configurations](#step-5---synchronize--promote-configs)

--------------------------------------

Il peut y avoir différents types de changements sur la zone d'accueil, mais ils commencent et se terminent tous de la même manière. Ce document passera par les différentes étapes.

Avant de continuer, vous devez vous familiariser avec les concepts de « [Repository Structure.md](../Architecture/Repository%20Structure.md) ».

La solution de zone d'atterrissage utilise certaines fonctionnalités de [`kpt`](https://kpt.dev/book/02-concepts/) pour gérer les [packages](https://kpt.dev/book/03-packages/) des configurations YAML.

À titre d'aperçu de haut niveau, un package comprendra généralement des fichiers spécifiquement utilisés par kpt :

- `setters.yaml`: utilisé pour définir des configurations personnalisables.
- `Kptfile`: utilisé pour garder une trace des versions du package et [définir de manière déclarative quelles fonctions](https://kpt.dev/book/04-using-functions/01-declarative-function-execution) doivent s'exécuter pendant le rendu. Par exemple, [apply-setters](https://catalog.kpt.dev/apply-setters/v0.2/).

## Étape 1 - Configuration

Pour tout type de changement, vous devez commencer avec une nouvelle branche et une arborescence de travail git propre (tous les fichiers sont préparés et validés). Cela facilitera la visualisation des modifications dans [Git Source Control de VSCode](https://code.visualstudio.com/docs/sourcecontrol/overview) (ou `git diff`) et l'annulation si nécessaire.
Vous pouvez confirmer qu'une arborescence de travail est propre en exécutant `git status`.

Depuis votre environnement local :

1. Clonez le référentiel nécessitant une modification :

    ```shell
    git clone <URL DU REPO>
    ```

2. Déplacez-vous dans le nouveau dossier correspondant à ce dépôt :

    ```shell
    cd <NOM DU REPO>
    ```

3. Créez une nouvelle branche, utilisez la convention de dénomination le cas échéant (par exemple, ajoutez le numéro de problème/élément de travail dans le nom de la branche) :

    ```shell
    git checkout -b <NOM DE LA BRANCHE>
    ```

4. Exécutez ce qui suit pour obtenir la version appropriée du sous-module outils :

    ```shell
    bash modupdate.sh
    ```

## Étape 2 - Modifier

Il existe différents types de changements. Suivez la section appropriée pour obtenir des instructions.

### A) Ajouter un package

Ceci est accompli avec la commande [`kpt pkg get`](https://kpt.dev/reference/cli/pkg/get/).

En règle générale, les packages ne doivent être ajoutés que dans le dossier `tierX/source-base` d'un monorepo de déploiement et **jamais** modifiés manuellement à partir de là. Toutes les personnalisations doivent être effectuées à partir des dossiers `tierX/source-customization/<env>`.

Suivez ces étapes pour ajouter un package :

1. Vous pouvez mettre à jour et définir ces variables pour faciliter l'exécution des commandes suivantes :

    ```shell
    # valeur de niveau X
    exporter NIVEAU=''

    # URI du dépôt git contenant le package
    # par exemple, 'https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git'
    exporter REPO_URI=''

    # sous-répertoire du package, relatif à la racine du dépôt
    # par exemple, 'solutions/core-landing-zone'
    exporter PKG_PATH=''

    # la version à obtenir, située dans le package CHANGELOG.md, utiliser 'main' si non disponible
    # par exemple, '0.0.1'
    exporter VERSION=''

    # le répertoire de destination local pour enregistrer le package, par rapport à la racine du référentiel
    # par exemple, 'tier1/source-base/core-landing-zone'
    export LOCAL_DEST_DIRECTORY=''
    ```

2. Créer le répertoire de destination local

    ```shell
    cd ${TIER}/source-base
    # if folder doesnt exist then create it
    if [ -n "${LOCAL_DEST_DIRECTORY}" ] && [ ! -d "${LOCAL_DEST_DIRECTORY}" ]; then
      mkdir -p ${LOCAL_DEST_DIRECTORY}
    fi
    ```

3. Ajoutez le package avec la commande suivante :

    ```shell
    kpt pkg get ${REPO_URI}/${PKG_PATH}@${VERSION} ${LOCAL_DEST_DIRECTORY}
    ```

4. Vérifiez les modifications avec la visionneuse de contrôle de source intégrée à VSCode ou en exécutant « git diff ».
5. Consultez la documentation du package pour obtenir des instructions spécifiques. La plupart nécessiteront une sorte de personnalisation, comme la modification du fichier « setters.yaml ».
Il devra être personnalisé pour chaque environnement. Il s'agit d'un processus manuel, un extrait de code comme ci-dessous peut vous aider avec la copie initiale :

    ```shell
    # the file path to customize, relative to 'source-base'
    # for example, 'core-landing-zone/setters.yaml'
    export FILE_TO_CUSTOMIZE=''
    ```

    ```shell
    for env_subdir in experimentation dev preprod prod; do
        # check if env. folder exists in source-customization
        if [ -d "../source-customization/${env_subdir}" ]; then
            # copy the file, with no overwrite, keeping full path
            cp --no-clobber --parents "${FILE_TO_CUSTOMIZE}" "../source-customization/${env_subdir}"
        fi
    done
    ```

6. Assurez-vous que tous les fichiers copiés sous « source-customization » ont été modifiés pour refléter chaque environnement.
    > ***Astuce de personnalisation avancée.***
    Dans certains cas, certains fichiers de ressources YAML peuvent devoir être modifiés pour certains environnements, mais pas pour d'autres. Cela doit être évité autant que possible car cela complique le processus de mise à jour.
    Par exemple, pour supprimer complètement une stratégie d'organisation spécifique du test :
        - Copiez le fichier YAML de cette stratégie d'organisation dans le dossier de personnalisation de l'expérimentation, ***en vous assurant de conserver la même structure de répertoires***.
        - Mettez l'intégralité du fichier dans un bloc de commentaires.
        - Le processus d'hydratation ignorera alors cette définition de ressource commentée, la supprimant ainsi.
7. Vérifiez toutes les personnalisations avec la visionneuse de contrôle de source intégrée à VSCode ou en exécutant « git diff ». Si vous êtes satisfait, passez à [Étape 3 – Hydrater] (#étape 3---hydrater).

### B) Modifier un package

De par sa conception, cela est accompli en modifiant les configurations dans `tierX/source-customization/<env>`. Les fichiers dans d'autres répertoires ne doivent jamais être modifiés manuellement.

***La personnalisation standard ne doit impliquer que le fichier `setters.yaml`.***

Suivez ces étapes pour modifier un package :

1. Modifiez les configurations pour chaque environnement applicable dans `tierX/source-customization/<env>`
2. Une fois que toutes les personnalisations ont été examinées localement, passez à [Étape 3 – Hydrater](#step-3---hydrate).

### C) Mettre à jour un package

Ceci est accompli avec la commande [`kpt pkg update`](https://kpt.dev/reference/cli/pkg/update/).

La stratégie par défaut de « fusion de ressources » est généralement appropriée mais peut parfois omettre certains changements dans la structure des fichiers (ligne vierge entre les commentaires, etc.).
Dans ces cas, si le changement structurel est requis dans le cadre de la mise à jour, il peut être nécessaire d'utiliser *avec précaution* la stratégie « forcer-supprimer-remplacer ».

Le dossier « source-base » d'un monorepo de déploiement doit toujours contenir des packages non modifiés. C'est ici qu'ils sont également mis à jour.

> **!!! IMPORTANT !!!** Une fois qu'un package est mis à jour, il est important de vérifier s'il y a des modifications pour les fichiers qui ont été personnalisés. Par exemple, les fichiers « setters.yaml ».

Suivez ces étapes pour mettre à jour un package :

1. Vous pouvez mettre à jour et définir ces variables pour faciliter l'exécution des commandes suivantes :

    ```shell
    # the folder of the pkg to be updated
    # for example, 'tier1/source-base/core-landing-zone'
    export PKG_PATH=''

    # the version to update to
    # for example, '0.0.2'
    export VERSION=''
    ```

2. Mettez à jour le package avec la stratégie de « resource-merge » par défaut :

    ```shell
    kpt pkg update ${PKG_PATH}@${VERSION}
    ```

3. Vérifiez les modifications avec la visionneuse de contrôle de source intégrée à VSCode ou en exécutant `git diff`.
4. Si les modifications correspondent à vos attentes, passez à l'étape suivante. Sinon, vous pouvez essayer ces étapes :
    1. Annulez les modifications via VSCode Source Control ou en exécutant :

        ```shell
        git restore .
        ```

    2. Mettre à jour le paquet avec la stratégie `force-delete-replace` :

        ```shell
        kpt pkg update ${PKG_PATH}@${VERSION} --strategy=force-delete-replace
        ```

    3. Examinez attentivement les modifications avec VSCode Source Control ou en exécutant `git diff`.
    En faisant très attention à ce qu'il n'ait pas supprimé quelque chose qu'il n'aurait pas dû avoir (sous-paquets locaux, etc.). Vous pouvez facilement ignorer les modifications spécifiques dans le contrôle de source de VSCode ou avec `git restaurer <file>`.
    Cette stratégie peut également supprimer ou modifier les annotations « cnrm.cloud.google.com/blueprint: » dans de nombreux fichiers YAML. Ces changements créeront malheureusement une différence git importante mais peuvent être acceptés.
    4. Si les modifications correspondent à vos attentes, passez à l'étape suivante.
5. Pour chaque fichier sous « source-customization/<env> », vérifiez s'il a changé dans « source-base ».
Par exemple, si le package de la zone d'atterrissage est mis à jour, comparez « tier1/source-customization/dev/core-landing-zone/setters.yaml » avec « tier1/source-base/core-landing-zone/setters.yaml ». .
    - Si un changement est détecté, mettez à jour manuellement le fichier dans `source-customization/<env>`.
6. Une fois que toutes les personnalisations ont été examinées localement, passez à [Étape 3 - Hydrater](#step-3---hydrate).

### D) Supprimer un paquet

Ceci est accompli en supprimant simplement les fichiers du package dans `tierX/source-base` et ses personnalisations dans `tierX/source-customization/<env>`.

> **!!! IMPORTANT !!!** Avant de supprimer un package, vérifiez qu'il ne contient pas de sous-packages encore nécessaires.

Suivez ces étapes pour supprimer un package :

1. Déplacez-vous dans le dossier « source-base » :

    ```shell
    cd <tierX>/source-base
    ```

2. Vous pouvez mettre à jour et définir ces variables pour faciliter l'exécution des commandes suivantes :

    ```shell
    # the folder of the pkg to be removed
    # for example, 'core-landing-zone'
    export PKG_PATH=''
    ```

3. Supprimer un paquet :

    ```shell
    rm --recursive ${PKG_PATH}
    ```

4. Les personnalisations du package doivent maintenant être supprimées, passez à « source-customization » :

    ```shell
    cd ../source-customization
    ```

5. Supprimez la personnalisation pour chaque environnement :

    ```shell
    for env_subdir in experimentation dev preprod prod; do
        # check if env. folder exists in source-customization
        if [ -d "${env_subdir}/${PKG_PATH}" ]; then
            rm --recursive "${env_subdir}/${PKG_PATH}"
        fi
    done
    ```

6. Passez en revue les modifications avec la visionneuse de contrôle de source intégrée à VSCode ou en exécutant « git diff ». Si vous êtes satisfait, passez à [Étape 3 – Hydrater] (#étape 3---hydrater).

## Étape 3 – Hydrater

Ceci est accompli avec le script `hydrate.sh` situé dans le sous-module outils. En partie, il utilise la commande [`kpt fn render`](https://kpt.dev/reference/cli/fn/render/).

> **!!! IMPORTANT !!!** `kpt fn render` ne doit jamais être exécuté manuellement dans aucun répertoire. Cela garantit de meilleures mises à jour des packages et minimise les problèmes d’hydratation.

En gros, le script fera :

- ajout (`source-base` + `source-customization/<env>`) à `temp-workspace/<env>`
- ensuite hydrater `temp-workspace/<env>` à `deploy/<env>`

Suivez ces étapes pour hydrater votre changement :

1. Exécutez le script d'hydratation ***depuis la racine du référentiel*** :

    ```shell
    bash tools/scripts/kpt/hydrate.sh
    ```

2. Corrigez les erreurs, le cas échéant.
3. Examinez les modifications avec la visionneuse de contrôle de source intégrée à VSCode ou en exécutant « git diff », en particulier les fichiers hydratés dans les dossiers « deploy/<env> ». Si vous êtes satisfait, il est temps de publier.

## Étape 4 - Publier

À ce stade, les changements n’existent que localement. Ils sont maintenant prêts à être publiés pour examen et approbation par les pairs.

Suivez ces étapes pour publier les modifications :

1. Préparez votre commit en préparant les fichiers :

    ```shell
    git add .
    ```

2. Commit vos changements:

    ```shell
    git commit -m '<MEANINGFUL MESSAGE GOES HERE>'
    ```

3. Poussez vos modifications vers l'origine du monorepo :

    ```bash
    git push --set-upstream origin <branch name>
    ```

4. Créez une nouvelle [pull request (PR)](https://learn.microsoft.com/en-us/azure/devops/repos/git/pull-requests?view=azure-devops&tabs=browser) sur le monorepo pour fusionner ce `<nom de la branche>` dans `main`.
5. Confirmez que toutes les vérifications requises ont réussi (approbations, tests, etc.). Si les vérifications échouent, traitez-les dans votre succursale locale, puis préparez-les, validez-les et transmettez-les à l'origine.
6. Complétez la demande d'extraction une fois que toutes les vérifications requises ont réussi.

## Étape 5 - Synchroniser/Promouvoir les configurations

Cette section contient des informations sur la façon dont les modifications peuvent être promues entre les environnements.

Les modifications apportées aux monorepos de déploiement ne seront appliquées à GCP que lorsque le dossier « csync/deploy/<env> » sera mis à jour.

Cet exemple se concentrera sur le monorepo `gcp-env-tier1` :

1. Une modification est effectuée dans le dossier « tier1 ».
    > **!!! Il est important d'ajouter la « personnalisation de la source » pour chaque environnement. Cela garantira que tous les environnements sont rendus, validés et balisés en même temps. !!!**
1. Une fois le PR fusionné, notez la nouvelle version de la balise ou validez SHA.
1. À ce stade, les modifications n'ont pas été déployées sur GCP. Des modifications de type « Modifier un package » sont requises dans le dossier « csync/deploy/<env> » pour chaque environnement.
1. `dev` :
    - Définissez `version:` dans `csync/source-customization/dev/root-sync-git/setters-version.yaml` sur la nouvelle balise ou validez SHA noté précédemment.
    - Hydratez le monorepo et créez un PR.
    - Une fois le PR fusionné, l'opérateur de synchronisation de configuration récupérera les configurations mises à jour dans `csync/deploy/dev`.
    - Confirmez la synchronisation de toutes les ressources à partir du [Tableau de bord Config Sync](https://console.cloud.google.com/kubernetes/config_management/dashboard) ou en exécutant « nomos status ».
    - Valider les fonctionnalités de la zone d'atterrissage et de la charge de travail pour l'environnement « dev » dans GCP. Passez à `preprod` en cas de succès, redémarrez le processus sinon.
1. `préprod` :
    - Définissez `version:` dans `csync/source-customization/preprod/root-sync-git/setters-version.yaml` sur la même valeur que `dev`.
    - Hydratez le monorepo et créez un PR.
    - Une fois le PR fusionné, l'opérateur de synchronisation de configuration récupérera les configurations mises à jour dans `csync/deploy/preprod`.
    - Confirmez la synchronisation de toutes les ressources à partir du [Tableau de bord Config Sync](https://console.cloud.google.com/kubernetes/config_management/dashboard) ou en exécutant « nomos status ».
    - Valider les fonctionnalités de la zone d'atterrissage et de la charge de travail pour l'environnement « preprod » dans GCP. Passez à « prod » en cas de succès, redémarrez le processus sinon.
1. `prod` :
    - Définissez `version:` dans `csync/source-customization/prod/root-sync-git/setters-version.yaml` sur la même valeur que `preprod`.
    - Hydratez le monorepo et créez un PR.
    - Une fois le PR fusionné, l'opérateur de synchronisation de configuration récupérera les configurations mises à jour dans `csync/deploy/prod`.
    - Confirmez la synchronisation de toutes les ressources à partir du [Tableau de bord Config Sync](https://console.cloud.google.com/kubernetes/config_management/dashboard) ou en exécutant « nomos status ».
    - Valider les fonctionnalités de la zone d'atterrissage et de la charge de travail pour l'environnement « prod » dans GCP. Redémarrez le processus en cas d'échec.
