# How to complete some Health Checks

## 2023-03-14 | Version: 0.01

### Vue d'ensemble

Certaines méthodes et emplacements différents pour faciliter les vérifications de l'état et le dépannage de l'infrastructure de la zone d'atterrissage et de l'environnement client.

---

## Index

Infrastructure

- [Config Sync](#config-sync)
- [Kubernetes Config Controller (KCC)](#kubernetes-config-controller)
- [gcloud](#gcloud)
- [Comment configurer des alertes](#setting-alerts)

Coeur de l'environnement

- [Central logging project's Logs Explorer](#central-logging-projects-logs-explorer)

Environnement client

- [GCP Logging tip/tricks](#check-the-logs-by-using-the-gui)

>Remarque : Il y aura prochainement un projet lié spécifiquement à l'exploitation forestière. Ce document devra être mis à jour en conséquence une fois le projet disponible.

---

## Infrastructure

### Config Sync

Général

"nomos" est un outil permettant de résoudre les problèmes de Config Sync.

Comment installer Nomos

``` gcloud
gcloud components install nomos
```

Vérifier l'état général de Config Sync

``` nomos
nomos status
```

Cela répétera la commande nomos status toutes les 5 secondes

``` nomos
nomos status --poll=5s
```

Vérification de l'état à l'aide de l'interface graphique

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Sélectionnez le projet qui exécute le cluster Kubernetes
3. Menu hamburger > Kubernetes Engine
4. Dans le menu de gauche, développez « Configuration et politique »
5. Cliquez sur "Configurer"
6. Jetez un œil aux pages « Tableau de bord » et « Packages ».

Des informations complémentaires peuvent être trouvées sur le site de Google pour [Dépannage de la synchronisation de configuration](https://cloud.google.com/anthos-config-management/docs/how-to/troubleshooting-config-sync)

---

### Kubernetes Config Controller

Vérifiez si KCC est en cours d'exécution

``` kubectl
kubectl get pod -n cnrm-system
```

``` kubectl
kubectl get events -A
kubectl get events -A --watch
```

``` kubectl
kubectl get gcp -A
kubectl get gcp -A |grep False
```

``` kubectl
kubectl describe {resource object} -n {namespace}

exemple:
kubectl describe folder.resourcemanager.cnrm.cloud.google.com/tests -n hierarchy
```

Des commandes kubectl supplémentaires peuvent être trouvées sur [https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

Jetez un œil au navigateur d'objets Kubernetes dans l'interface graphique

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Sélectionnez le projet qui exécute le projet Kubernetes
3. Menu hamburger > Kubernetes Engine
4. Cliquez sur "Navigateur d'objets"
5. Développez les différents noms d'objets et cliquez sur les liens liés à la ressource sur laquelle vous souhaitez plus de détails.

Vérifiez les journaux à l'aide de l'interface graphique

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Sélectionnez le projet qui exécute le cluster Kubernetes
3. Menu hamburger > Kubernetes Engine
4. Développez le menu de gauche, cliquez sur "Clusters"
5. Cliquez sur Nom du cluster
6. Jetez un œil aux pages "Détails", "Nœuds", "Stockage" et "Journaux" dans le sous-menu.

OU

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Sélectionnez le projet qui exécute le cluster Kubernetes
3. Menu Hamburger > Journalisation > Explorateur de journaux

---

### gcloud

Répertorier uniquement les fichiers journaux contenant des entrées de journal

``` gcloud
gcloud logging logs list
```

---

### Comment configurer des alertes

Comment définir des alertes pour ne pas manquer des événements importants
> TODO - which alerts should be set?
---

## Coeur de l'Environment

### Central logging project's Logs Explorer

Comment afficher les journaux situés dans le projet de journalisation centrale. À partir de l'explorateur de journaux de ce projet de journalisation centrale, vous pourrez voir les compartiments de journaux.

   1. [https://console.cloud.google.com](https://console.cloud.google.com)
   2. Sélectionnez le projet de journalisation centralisée
   3. Menu Hamburger > Journalisation
   4. Dans le menu de gauche, jetez un oeil à "Logs Explorer"
   5. En haut, cliquez sur « Affiner la portée »
   6. Sélectionnez « Portée par stockage »
   7. Sélectionnez le compartiment
   8. Cliquez sur Appliquer

Les types de compartiments de journaux que vous devriez voir sont ci-dessous.

Organisation d’expérimentation

- Compartiment de journaux pour les journaux de sécurité (Cloud Audit, journaux Access Transparency et journaux d'accès aux données)
- Compartiment de journaux pour les journaux de plate-forme et de composants pour les ressources sous le dossier tests
- Compartiment de journaux pour la plate-forme client et les journaux de composants pour les ressources sous le dossier clients

Organisations de développement, de pré-prod et de production

- Compartiment de journaux pour les journaux de sécurité (Cloud Audit, journaux Access Transparency et journaux d'accès aux données)
- Compartiment de journaux pour les journaux de plate-forme et de composants pour les ressources sous les dossiers services et services-infrastructure
- Compartiment de journaux pour la plate-forme client et les journaux de composants pour les ressources sous le dossier clientN/applications-infrastructure
- Compartiment de journaux pour la plate-forme client et les journaux de composants pour les ressources sous clientN/auto et clientN/applications (à déterminer – nécessite un examen)

---

## Environnement client

### Check the logs by using the GUI

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Sélectionnez le projet que vous souhaitez explorer
3. Menu Hamburger > Journalisation
4. Dans le menu de gauche, jetez un oeil à "Logs Explorer", "Logs Dashboard"
