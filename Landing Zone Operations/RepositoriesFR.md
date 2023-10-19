# Git Repositories

- [Dépôts Git](#git-dépôts)
  - [Créer un nouveau monorepo de déploiement](#create-new-deployment-monorepo)
    - [1. Construire le Monorepo](#1-build-the-monorepo)
    - [2. Ajouter une protection de branche](#2-add-branch-protection)
    - [3. Vérifier les autorisations du compte de service](#3-verify-service-account-permissions)
    - [4. Ajouter des pipelines](#4-ajouter-des pipelines)
  - [Mettre à jour le dépôt de déploiement](#update-deployment-repo)
    - [Mise à jour à partir du modèle](#update-from-template)
    - [Mettre à jour le sous-module `tools`](#update-tools-submodule)

--------------------------------------

Documentation pour configurer et gérer git monorepos utilisé avec Config Sync pour déployer des ressources GCP.

SSC utilise Azure Devops Repositories (AzDO Repos) et Pipelines comme solution git.

SSC implémente un déploiement [Gitops-Git](https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/tree/main/solutions/landing-zone-v2#gitops---git).
Comme illustré dans le diagramme [Gitops](../Architecture/Repository%20Structure.md#Gitops), l'opérateur ConfigSync observe plusieurs dossiers du [monorepos](https://monorepo.tools/).

## Créer un nouveau monorepo de déploiement

Les monorepo de déploiement sont initialement créés de la même manière, à partir du clonage d'un modèle monorepo. Ces étapes devront être répétées pour chaque nom de monorepo. Par exemple, `gcp-env-tier1`, `gcp-<client-name>-tier2` ou `gcp-<x-project-id>-tier34`.

Les informations d’identification git devront être définies de manière appropriée pour votre organisation AzDO.

### 1. Construisez le Monorepo

> Si votre organisation AzDO a défini des politiques de branche à l'échelle du projet sur les référentiels, vous devrez peut-être travailler avec des branches/demandes d'extraction ou vous accorder temporairement l'autorisation de « contourner les politiques lors du push » sur le monorepo.

1. Créez un nouveau référentiel **vide** (pas de README.md) pour contenir les configurations dans [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/repos/git/create -new-repo?view=azure-devops). Une fois créé, copiez l'URL HTTPS. Cela sera nécessaire pour la prochaine étape.

2. Mettez à jour et exportez les variables ci-dessous :
    ```shell
    export NEW_REPO_NAME='<your new monorepo name>'
    export NEW_REPO_URL='<your new monorepo URL>'
    ```

3. Clonez le modèle dans un dossier nommé comme votre nouveau monorepo :

    - tier1

        ```shell
        git clone https://github.com/ssc-spc-ccoe-cei/gcp-tier1-template.git ${NEW_REPO_NAME}
        ```

    - tier2

        ```shell
        git clone https://github.com/ssc-spc-ccoe-cei/gcp-tier2-template.git ${NEW_REPO_NAME}
        ```

    - tier34

        ```shell
        git clone https://github.com/ssc-spc-ccoe-cei/gcp-tier34-template.git ${NEW_REPO_NAME}
        ```

4. Déplacez-vous dans le nouveau dossier correspondant au nouveau monorepo :

    ```shell
    cd ${NEW_REPO_NAME}
    ```

5. Changez la télécommande du monorepo :

    ```shell
    # Remove the origin pointing to the template monorepo
    git remote remove origin
    # Add the new origin with your monorepo url
    git remote add origin ${NEW_REPO_URL}
    ```

6. Confirmez la nouvelle configuration à distance, elle ne devrait répertorier que votre nouvelle URL monorepo pour la récupération et le push :

    ```shell
    git remote --verbose
    ```

7. Modifiez `modversions.yaml` pour épingler le sous-module `tools` à une [balise de version] spécifique (https://github.com/ssc-spc-ccoe-cei/gcp-tools/releases) ou validez SHA.
8. Exécutez ce qui suit pour obtenir la version appropriée du sous-module outils :

    ```shell
    bash modupdate.sh
    ```

9. Supprimez les répertoires de pipelines qui ne sont pas requis :
    - Si le monorepo est hébergé dans Azure DevOps :

        ```Shell
        # supprimer le répertoire des pipelines '.github'
        rm --recursive '.github'
        ```

    - Si le monorepo est hébergé dans GitHub :

        ```Shell
        # supprimer le répertoire des pipelines '.azure-pipelines'
        rm --recursive '.azure-pipelines'
        ```

10. Supprimez les sous-répertoires d'environnement qui ne sont pas requis. **Ne supprimez jamais les fichiers '.gitkeep' dans les dossiers restants.**
    - Si le monorepo est `gcp-experimentation-tier1` :

        ```Shell
        # supprime les sous-répertoires 'dev', 'preprod' et 'prod'
       for root_dir in csync tier1; do
          for env_subdir in dev preprod prod; do
              rm --recursive "${root_dir}/deploy/${env_subdir}/"
              rm --recursive "${root_dir}/source-customization/${env_subdir}/"
          done
        done
        ```

    - Si le monorepo est `gcp-env-tier1` :

       ```shell
        # remove the 'experimentation' sub-directory
        for root_dir in csync tier1; do
          for env_subdir in experimentation; do
              rm --recursive "${root_dir}/deploy/${env_subdir}/"
              rm --recursive "${root_dir}/source-customization/${env_subdir}/"
          done
        done
        ```

11. Personnalisez le fichier `csync/source-customization/<env>/**/<root/repo>-sync-git/setters.yaml` pour tous les environnements.

12. Générez des fichiers hydratés, suivez l'étape 3 de [Changing.md](./Changing.md#step-3---hydrate).

13. Passez en revue vos modifications locales, puis préparez-les, validez-les et transférez-les vers votre nouveau monorepo. **Vous devrez appuyer sur principal.**

    ```coquille
    git ajouter .
    git commit -m 'initialisation de monorepo à partir du modèle'
    git push --set-upstream origin main
    ```

14. Le monorepo est créé ! Il est désormais temps de protéger la branche principale.

### 2. Ajouter une protection de branche

Il est recommandé de protéger la branche « principale » et d'utiliser des demandes d'extraction pour toute modification apportée aux monorepos.

Ces paramètres peuvent également être définis au niveau du projet AzDO.

À tout le moins, le [Exiger un nombre minimum de réviseurs](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops&tabs=browser#require_reviewers ) la politique de branche doit être activée sur la branche « principale ». Ce faisant, la branche ne peut pas être supprimée et les modifications doivent être apportées via une pull request.

Pour activer la stratégie :

1. Accédez à **Paramètres du projet > Dépôts/Dépôts > {dépôt} > Politiques > Politiques de branche > principal**
1. Activez **Exiger un nombre minimum de réviseurs**, le *Nombre minimum de réviseurs* sera par défaut de 2.
    - Vous pouvez également activer *Lorsque de nouvelles modifications sont appliquées :* > *Réinitialiser tous les votes d'approbation*

Ces autres stratégies peuvent également être activées si nécessaire :

- **Vérifier les éléments de travail liés** : *Obligatoire*
- **Vérifier la résolution des commentaires** : *Obligatoire*
- **Limiter les types de fusion** : autoriser uniquement la *fusion Squash*

> À FAIRE : Politique supplémentaire pour les "réviseurs automatiquement inclus" avec filtres de chemin.

### 3. Vérifiez les autorisations du compte de service

Un compte de service AzDO doit être utilisé pour authentifier Config Sync. Il nécessite un accès en lecture au monorepo.

En fonction de votre organisation, cela peut être défini à différents niveaux et avec des groupes. Ces étapes supposeront que le compte de service a déjà été configuré.

Confirmer:

1. Accédez à **Paramètres du projet > Dépôts/Dépôts > {dépôt} > Sécurité**.
1. Recherchez et cliquez sur l'utilisateur ou le groupe du compte de service approprié.
1. Vérifiez que l'autorisation **Lecture** est définie sur **Autoriser**. Toutes les autres autorisations doivent être « Non défini » ou « Refuser ».

Un [jeton d'accès personnel (PAT)](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs= Windows) avec la portée de **Code (Lecture)** devra également être créé pour le compte de service. Cela ne devrait être fait qu’une seule fois par compte de service. **Notez la date d'expiration, elle devra être régénérée périodiquement.**

### 4. Ajouter des pipelines

Le monorepo est désormais créé et la branche principale est protégée. Des [Pipelines](./Pipelines.md) peuvent être créés.

- **Tous les monorepos de déploiement doivent avoir le pipeline `validate-yaml`.**

- Pour utiliser le versioning sémantique lors des opérations de déploiement, les monorepos peuvent être configurés avec un pipeline de balisage git, tel que [version-tagging](https://github.com/ssc-spc-ccoe-cei/gcp-tools/tree/ main/pipeline-samples/version-tagging).

## Mettre à jour le dépôt de déploiement

Cette section sert à mettre à jour le monorepo de déploiement lui-même, et non les configurations YAML.

Comme pour tout autre changement, ils doivent être effectués via un processus de relations publiques.

### Mise à jour à partir du modèle

Les modèles restent assez statiques, mais ils peuvent parfois contenir des changements importants.

Suivez ces étapes pour mettre à jour votre monorepo de déploiement avec les modifications apportées aux modèles :

- [gcp-tier1-template](https://github.com/ssc-spc-ccoe-cei/gcp-tier1-template)
- [gcp-tier2-template](https://github.com/ssc-spc-ccoe-cei/gcp-tier2-template)
- [gcp-tier34-template](https://github.com/ssc-spc-ccoe-cei/gcp-tier34-template)

TODO : tester et ajouter des étapes

### Mettre à jour le sous-module `tools`

Le sous-module outils peut facilement être mis à jour vers une nouvelle version.

Suivez ces étapes pour

1. Clonez le monorepo de déploiement et extrayez une nouvelle branche.
2. Modifiez `modversions.yaml` pour épingler le sous-module `tools` à une nouvelle [balise de version](https://github.com/ssc-spc-ccoe-cei/gcp-tools/releases) ou validez SHA.
3. Exécutez ce qui suit pour obtenir la version appropriée du sous-module outils :

    ```shell
    bash modupdate.sh
    ```

4. Si votre monorepo contient des pipelines créés à partir de « tools/pipeline-samples », comparez les fichiers YAML pour voir si des modifications sont nécessaires.
5. Organisez, validez et poussez votre branche à créer un PR.
