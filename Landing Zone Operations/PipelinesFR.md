# Pipelines

- [Pipelines](#pipelines)
  - [Azure DevOps YAML Pipelines](#azure-devops-yaml-pipelines)
    - [Ajouter un Pipeline](#add-pipeline)
    - [Ajouter un déclenchement de PR](#add-pr-trigger)
    - [Détruire un Pipeline](#delete-pipeline)
  - [GitHub Actions Workflows](#github-actions-workflows)

--------------------------------------

Documentation pour gérer et configurer les pipelines.

Cette documentation suppose que les fichiers YAML appropriés sont déjà validés dans le dépôt. Des exemples peuvent être trouvés dans le dépôt [gcp-tools](https://github.com/ssc-spc-ccoe-cei/gcp-tools/tree/main/pipeline-samples/).

## Pipelines YAML Azure DevOps

Les [pipelines Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops) doivent être créés manuellement à l'aide des Fichiers de définition YAML situés dans votre dépôt. Les fichiers peuvent être situés dans n'importe quel répertoire, nous utiliserons le répertoire `.azure-pipelines/` comme convention.

### Ajouter un pipeline

Répétez les étapes suivantes dans Azure DevOps pour chaque fichier de définition de pipeline YAML :

Accédez à **Pipelines > Pipelines** :

1. Cliquez sur le bouton **Nouveau pipeline**.
1. Sélectionnez **Azure Repos Git (YAML)**.
1. Sélectionnez votre dépôt.
1. Sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Sélectionnez le fichier approprié dans la liste déroulante **Chemin** et cliquez sur **Continuer**.
1. Vérifiez le fichier YAML, cliquez sur la flèche vers le bas à côté de **Exécuter** et cliquez sur **Enregistrer**.
1. Le pipeline est maintenant ajouté !
Cependant, son nom par défaut est le nom du dépôt, ce qui n'est pas idéal si le dépôt contient plusieurs pipelines et/ou si le projet contient plusieurs dépôts.
Cliquez sur **"trois points" > Renommer/déplacer** pour fournir un nom significatif et éventuellement déplacer vers un dossier.
Par exemple, si le nom de votre dépôt est « my-repo » et que le fichier YAML du pipeline est « firstpipeline.yaml », le pipeline pourrait être renommé en quelque chose comme « my-repo__firstpipeline » dans un dossier « my-repo ».

> *Lors de la première exécution d'un pipeline, il peut être nécessaire de l'autoriser manuellement à utiliser des pools d'agents (selon les paramètres de sécurité) et/ou à accéder à d'autres ressources (dépôts, groupes de variables, etc.).*

### Ajouter un déclenchement de PR

> **!!! Vérifiez le fichier yaml du pipeline pour déterminer si un tel déclencheur est requis. Il y sera mentionné.**

Un pipeline devra peut-être s'exécuter lorsqu'une Pull Request (PR) est créée/mise à jour. Dans Azure DevOps, ce [trigger](https://learn.microsoft.com/en-us/azure/devops/pipelines/repos/azure-repos-git?view=azure-devops&tabs=yaml#pr-triggers) ne peut pas être défini dans les définitions YAML. Une [politique de validation de build](https://docs.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops&tabs=browser#build-validation) doit être créée pour accomplir ce.

Pour ce faire, pour chaque repo/pipeline nécessitant un déclencheur PR :

Accédez à **Paramètres du projet > Dépôts/Dépôts > {dépôt} > Politiques > Politiques de branche > principal**, cliquez sur le **« + »** à côté de **Validation de construction** et définissez les options suivantes :

1. **Build Pipeline** : sélectionnez le nom du pipeline à déclencher sur les PR
1. **Déclencheur** : *Automatique*
1. **Exigence du règlement** : *Obligatoire*
1. **Expiration de la build** : *Immédiatement*
1. **Nom d'affichage** : nom facultatif

> *L'ajout d'une stratégie de branche protégera la branche.*

### Supprimer le pipeline

Accédez à **Pipelines > Pipelines** :

1. Recherchez le pipeline à supprimer et cliquez sur ses **"trois points"** sur le côté droit de l'écran.
1. Cliquez sur **Supprimer**.
1. Suivez l'invite de confirmation.

Vous pouvez maintenant supprimer le fichier YAML approprié du répertoire « .azure-pipelines » de votre dépôt.

## Flux de travail des actions GitHub

Un fichier YAML correctement formaté dans « .github/workflows » doit être automatiquement ajouté.

Veuillez vous référer à la [documentation de GitHub](https://docs.github.com/en/actions/using-workflows/about-workflows).
