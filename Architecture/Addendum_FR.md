# Addendum aux ressources Google Cloud

- [Addendum aux ressources Google Cloud](#google-cloud-resources-addendum)
  - [Liens intéressants](#fun-links)
  - [Config Controller](#config-controller)
  - [Policy Controller](#policy-controller)
  - [Config Sync](#config-sync)
    - [Outils et utilitaires](#tools-and-utilities)
  - [kpt](#kpt)
  - [DNS](#dns)
    - [Connaissance général des DNS](#general-dns-knowledge)
    - [Concepts clés des DNS dans GCP](#gcp-dns-key-terms)
    - [Documentation spécifique sur Google Cloud](#google-cloud-specific-documentation)
    - [Zones DNS](#dns-zones)
    - [DNS Internes](#internal-dns)
    - [Zones privées](#private-zones)
      - [Options](#options)
    - [Zones DNS publiques et externes](#public-external-dns-zone)
      - [Résolution anycast](#anycast-resolution)
    - [Liaison entre projets](#cross-project-binding)
  - [Cloud Armor](#cloud-armor)
    - [Cloud Armour englobe](#cloud-armour-encompasses)
    - [Cloud Armor Policies \& Ordering](#cloud-armor-policies--ordering)
    - [Comment ça fonctionne](#how-it-works)
    - [Exemple de portée d'une politique](#example-policy-scopes)
    - [Documentation d'intérêt sur Cloud Armor](#cloud-armor-documentation-of-interest)
  - [Cloud Load Balancing](#cloud-load-balancing)
  - [Création de règles de Firewall](#firewall-rules-creation)
  - [VPC Service Controls](#vpc-service-controls)
    - [Démarrage](#quick-start)
    - [Information générale](#general-information)
  - [Remote Access avec IAP](#remote-access-with-iap)
    - [Accéder à une machine virtuelle windows avec IAP](#iap-access-into-a-windows-vm)

--------------------------------------

## Liens intéressants

Enjoy!

L'[infographie ultime sur les produits GCP](https://googlecloudcheatsheet.withgoogle.com/) pour les développeurs

Vérifiez la latence de vos régions en utilisant [GPCping](https://gcping.com/)

## Config Controller

[Config Controller](https://cloud.google.com/anthos-config-management/docs/concepts/config-controller-overview)

Config Controller fournit un plan de contrôle géré, basé sur Kubernetes. De plus, les instances Config Controller sont préinstallées avec Policy Controller, Config Sync et Config Connector. En utilisant ces composants, vous pouvez tirer parti des outils et des flux de travail de Kubernetes pour gérer les ressources Google Cloud et assurer la cohérence à l'aide d'un flux de travail GitOps.
Pour en savoir plus, consultez la présentation de Config Controller.

## Policy Controller

[Policy Controller](https://cloud.google.com/anthos-config-management/docs/concepts/policy-controller)

Policy Controller permet l’application de politiques entièrement programmables pour vos clusters. Ces politiques agissent comme des « protections » et empêchent toute modification apportée à la configuration de l'API Kubernetes de violer les contrôles de sécurité, opérationnels ou de conformité.
Vous pouvez définir des politiques pour bloquer activement les requêtes API non conformes, ou simplement pour auditer la configuration de vos clusters et signaler les violations. Policy Controller est basé sur le projet open source Open Policy Agent Gatekeeper - [OPA](https://open-policy-agent.github.io/gatekeeper/website/docs/operations/) et est livré avec une bibliothèque complète de politiques préfaites pour des contrôles communs de sécurité et de conformité.
En plus de contrôler activement votre environnement Kubernetes, vous pouvez éventuellement utiliser Policy Controller pour analyser la conformité de la configuration avant le déploiement. Cela permet de fournir des commentaires précieux pendant le processus de modification de la configuration et garantit que toutes les modifications non conformes sont détectées tôt avant qu'elles puissent être rejetées lors de l'application.

## Config Sync

[Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview)

Config Sync est un service GitOps inclut dans la solution Anthos. Config Sync repose sur un noyau open source et permet aux opérateurs de cluster et aux administrateurs de plateforme de déployer des configurations à partir d'une seule source de vérité. Le service a la flexibilité de prendre en charge un ou plusieurs clusters et n'importe quel nombre de référentiels par cluster ou "name space". Les clusters peuvent être dans un environnement hybride ou multi-cloud.

Config Sync rapproche en permanence l'état de Config Controller avec les fichiers stockés dans un ou plusieurs référentiels Git. Cette stratégie GitOps vous permet de gérer et de déployer des configurations courantes avec un processus auditable, transactionnel, révisable et contrôlé par les versions.

Premiers pas avec [Config Sync](https://cloud.google.com/anthos-config-management/docs/tutorials/config-sync-multi-repo)

### Outils et utilitaires

[Nomos](https://cloud.google.com/anthos-config-management/docs/how-to/nomos-command)
[Kubectl](https://cloud.google.com/anthos-config-management/docs/how-to/configure-config-sync-kubectl)

## kpt

[KPT Readme.md](https://github.com/GoogleContainerTools/kpt/#readme) est un projet open source utilisé pour hydrater des fichiers yaml, obtenir des packages, appliquer des fonctions, rechercher et remplacer dans les manifestes yaml. kpt est une chaîne d'outils centrée sur les packages qui permet un outil de création et d'activation de configuration WYSIWYG.

kpt [installation](https://kpt.dev/installation/)

kpt [functions catalog](https://catalog.kpt.dev/?id=curated-functions-catalog)

**Important:**  Veuillez noter que kpt est disponible via les composants gcloud, mais vous souhaiterez peut-être utiliser une version spécifique ou une version plus récente que celle fournie par le SDK gcloud.

## DNS

[DNS](https://cloud.google.com/dns/docs/dns-overview)

Présentation générale de certains types de ressources Google Cloud pour accélérer la conception et le déploiement de votre *charge de travail*.

### Connaissance général des DNS

- DNS server types *(Authoritative, Recursive)*
- Zones *(Public, Private, Delegated, Split brain/horizon)*
- Records *(A, AAAA, Cname, Mx, NS, PTR, SOA)*

### Concepts clés des DNS dans GCP

[Key Terms](https://cloud.google.com/dns/docs/key-terms)

### Documentation spécifique sur Google Cloud

[Cloud DNS Overview](https://cloud.google.com/dns/docs/overview/)

[Cloud Domains Overview](https://cloud.google.com/domains/docs/overview)

### Zones DNS

### DNS Internes

Les noms DNS internes sont des noms créés automatiquement par Google Cloud. Disponible dans un seul VPC par défaut, résolu par le serveur de métadonnées (169.254.169.254). Google Cloud crée, met à jour et supprime automatiquement ces enregistrements DNS. Veuillez consulter la [Documentation de la zone DNS interne](https://cloud.google.com/compute/docs/internal-dns).

### Zones privées

Gérez les noms de domaine personnalisés pour les machines virtuelles et autres ressources GCP sans exposer les données DNS à l'Internet public. Veuillez consulter la [Documentation de la zone privée](https://cloud.google.com/dns/docs/zones#create-private-zone).

Sauf si vous avez spécifié un autre serveur de noms dans une stratégie de serveur sortant, Google Cloud tente d'abord de trouver un enregistrement dans une zone privée (ou une zone de transfert ou une zone d'appairage) autorisée pour votre réseau VPC avant de rechercher l'enregistrement dans une zone publique.

#### Options

- [Forward Queries to another server](https://cloud.google.com/dns/docs/zones/forwarding-zones)
- [DNS Peering](https://cloud.google.com/dns/docs/zones/peering-zones)
- [Managed Reverse Lookup Zone](https://cloud.google.com/dns/docs/zones/managed-reverse-lookup-zones)

### Zones DNS publiques et externes

Créez des enregistrements DNS dans les zones publiques pour publier votre service sur Internet. Veuillez consulter la [Documentation de la zone publique](https://cloud.google.com/dns/docs/dns-overview#public_zone).

#### Résolution anycast
>>
>> Google DNS est associé à une certaine renommée pour ses propriétés de résilience et de disponibilité globale, avec des résolveurs accessibles au public tels que :
>>> 8.8.8.8
>>> 8.8.8.4

Ces résolveurs sont des adresses "anycast" avec des requêtes acheminées vers l'emplacement le plus proche annonçant l'adresse.

### Liaison entre projets

La liaison entre projets est utilisée pour gérer les zones DNS privées qui desservent un autre projet dans la même organisation.

[Cross project binding Overview](https://cloud.google.com/dns/docs/zones/zones-overview#cross-project_binding)
[Creating a Zone with cross project binding](https://cloud.google.com/dns/docs/zones/cross-project-binding)

## Cloud Armor

[Cloud Armor](https://cloud.google.com/armor/docs/cloud-armor-overview)

Google Cloud Armor est un service de défense DDoS et d'applications qui aide à protéger votre *service*. Cloud Armor est étroitement couplé au Global HTTPS(S) Cloud Load Balancer. Cloud Armor protège vos services réseau, généralement derrière un équilibreur de charge, contre le DDoS et d'autres attaques basées sur le Web. L’application est gérée au point de présence (POP) périphérique, aussi près que possible du trafic source. Le trafic peut être filtré et défendu au niveau de la couche 7 du modèle OSI.

### Cloud Armour Englobe

- Policies/Rules
- Allowlist / Denylist IP's

### Cloud Armor Policies & Ordering

- Rules (allow) [Priority 1]
- Rules (Deny) [Priority 2]

### Comment ça fonctionne

- Créer une politique avec des règles +1
  - Fournir des plages IP auxquelles appliquer la règle
  - Sur correspondance règle/IP : Autoriser/Refuser le trafic
    - Chevaucher les règles avec des priorités différentes
  - Appliquer la politique aux cibles
    - Cible = Services backend à charge équilibrée

### Exemple de portée d'une politique

- Créez plusieurs règles, qui se chevauchent pour créer des règles plus fines.
- Prévention des injections SQL (CSS), contrôle d'accès basé sur la géolocalisation.
- Permet le blocage du trafic périphérique.

### Documentation d'intérêt sur Cloud Armor

[Cas d'usage](https://cloud.google.com/armor/docs/common-use-cases)
[Politiques de sécurité](https://cloud.google.com/armor/docs/configure-security-policies#https-load-balancer)
[Référence sur le langage Cloud Armor](https://cloud.google.com/armor/docs/rules-language-reference)
[Cloud Armor Standard VS Managed Protection Plus](https://cloud.google.com/armor/docs/managed-protection-overview#standard_versus_plus)

## Cloud Load Balancing

[Cloud Load Balancing](https://cloud.google.com/load-balancing/docs/load-balancing-overview)

Un équilibreur de charge répartit le trafic utilisateur sur plusieurs instances de vos applications. En répartissant la charge, l'équilibrage de charge réduit le risque que vos applications rencontrent des problèmes de performances.
Cloud Load Balancing est un service géré entièrement distribué et défini par logiciel. Il n'est pas basé sur le matériel, vous n'avez donc pas besoin de gérer une infrastructure d'équilibrage de charge physique.

Google Cloud propose les fonctionnalités d'équilibrage de charge suivantes :

- Adresse IP anycast unique
- Équilibrage de charge défini par logiciel
- Mise à l'échelle automatique transparente
- Équilibrage de charge des couches 4 et 7
- Equilibrage de charge externe et interne
- Équilibrage de charge mondial et régional
- Prise en charge des fonctionnalités avancées

## Création de règles de Firewall

[Création de règles de Firewall](https://cloud.google.com/vpc/docs/firewalls)

Les règles de pare-feu Virtual Private Cloud (VPC) s'appliquent à un projet et un réseau donnés.
Les règles de pare-feu VPC vous permettent d'autoriser ou de refuser les connexions vers ou depuis des instances de machines virtuelles (VM) dans votre réseau VPC. Les règles de pare-feu VPC activées sont toujours appliquées, protégeant vos instances quels que soient leur configuration et leur système d'exploitation, même si elles n'ont pas démarré.

## VPC Service Controls

[VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs/overview)

*Important* [Terminologie](https://cloud.google.com/vpc-service-controls/docs/overview#terminology)

Un périmètre de service crée une limite de sécurité autour des ressources Google Cloud. Vous pouvez configurer un périmètre pour contrôler les communications depuis les machines virtuelles (VM) vers un service Google Cloud (API) et entre les services Google Cloud. Un périmètre permet une communication gratuite au sein du périmètre mais, par défaut, bloque la communication vers les services Google Cloud à travers le périmètre. Le périmètre ne bloque l’accès à aucune API ou service tiers sur Internet. [Document de référence.](https://cloud.google.com/vpc-service-controls/docs/overview#isolate)

### Démarrage

[Apprendre](https://cloud.google.com/vpc-service-controls/docs/set-up-service-perimeter) comment configurer un périmètre de service à l'aide de VPC Service Controls dans la console Google Cloud.

### Information générale

Les contrôles de service VPC permettent de bloquer ou de restreindre les services API au niveau du projet ou du réseau VPC.

- Protège contre l'exfiltration de données
- Créer un périmètre
  - Appliquer à un projet
  - Ajouter des services à restreindre comme l'API de stockage

[Access Context Levels](https://cloud.google.com/vpc-service-controls/docs/use-access-levels)

## Remote Access avec IAP

### Accéder à une machine virtuelle windows avec IAP

IAP Desktop est une application Windows qui vous permet de gérer plusieurs connexions Bureau à distance aux instances de VM Windows. IAP Desktop se connecte aux instances de VM à l'aide du transfert TCP Identity-Aware Proxy et ne nécessite pas que les instances de VM disposent d'une adresse IP publique.
Avant de vous connecter à l'aide d'IAP Desktop, assurez-vous que les conditions préalables suivantes sont remplies :

- Vous avez configuré votre VPC pour autoriser le trafic IAP vers votre instance de VM.
- Vous avez téléchargé et installé IAP Desktop sur votre ordinateur local.

[Présentation du transfert TCP IAP](https://cloud.google.com/iap/docs/tcp-forwarding-overview)

La fonctionnalité de transfert TCP d'IAP vous permet de contrôler qui peut accéder aux services administratifs tels que SSH et RDP sur vos backends depuis l'Internet public. La fonctionnalité de transfert TCP empêche ces services d'être ouvertement exposés à Internet. Au lieu de cela, les requêtes adressées à vos services doivent passer les contrôles d’authentification et d’autorisation avant d’atteindre leur ressource cible.
