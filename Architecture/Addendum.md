# Google Cloud Resources Addendum

<!-- vscode-markdown-toc -->
	* [Fun Links](#FunLinks)
	* [Config Controller](#ConfigController)
		* [[Config Controller Overview](https://cloud.google.com/anthos-config-management/docs/concepts/config-controller-overview)](#ConfigControllerOverviewhttps:cloud.google.comanthos-config-managementdocsconceptsconfig-controller-overview)
	* [Policy Controller](#PolicyController)
		* [[Policy Controller Overview](https://cloud.google.com/anthos-config-management/docs/concepts/policy-controller)](#PolicyControllerOverviewhttps:cloud.google.comanthos-config-managementdocsconceptspolicy-controller)
	* [Config Sync](#ConfigSync)
		* [[Config Sync Overview](https://cloud.google.com/anthos-config-management/docs/config-sync-overview)](#ConfigSyncOverviewhttps:cloud.google.comanthos-config-managementdocsconfig-sync-overview)
		* [Tools and Utilities](#ToolsandUtilities)
	* [kpt](#kpt)
	* [DNS](#DNS)
		* [[DNS Overview](https://cloud.google.com/dns/docs/dns-overview)](#DNSOverviewhttps:cloud.google.comdnsdocsdns-overview)
		* [General DNS knowledge](#GeneralDNSknowledge)
		* [GCP DNS Key Terms](#GCPDNSKeyTerms)
		* [Google Cloud Specific Documentation](#GoogleCloudSpecificDocumentation)
		* [DNS Zones](#DNSZones)
		* [Internal DNS](#InternalDNS)
		* [Private Zones](#PrivateZones)
		* [Public, External DNS Zone](#PublicExternalDNSZone)
		* [Cross Project Binding](#CrossProjectBinding)
	* [Cloud Armor](#CloudArmor)
		* [[Cloud Armor Overview](https://cloud.google.com/armor/docs/cloud-armor-overview)](#CloudArmorOverviewhttps:cloud.google.comarmordocscloud-armor-overview)
		* [Cloud Armor Documentation of Interest](#CloudArmorDocumentationofInterest)
	* [Cloud Load Balancing](#CloudLoadBalancing)
		* [[Cloud Load Balancing Overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview)](#CloudLoadBalancingOverviewhttps:cloud.google.comload-balancingdocsload-balancing-overview)
	* [Firewall Rules Creation](#FirewallRulesCreation)
		* [[VPC Firewall Rules Overview](https://cloud.google.com/vpc/docs/firewalls)](#VPCFirewallRulesOverviewhttps:cloud.google.comvpcdocsfirewalls)
	* [VPC Service Controls](#VPCServiceControls)
		* [[VPC Service Controls Overview](https://cloud.google.com/vpc-service-controls/docs/overview)](#VPCServiceControlsOverviewhttps:cloud.google.comvpc-service-controlsdocsoverview)
		* [*Important* [Terminology](https://cloud.google.com/vpc-service-controls/docs/overview#terminology)](#ImportantTerminologyhttps:cloud.google.comvpc-service-controlsdocsoverviewterminology)
		* [Quick Start](#QuickStart)
		* [General information](#Generalinformation)
		* [[Access Context Levels](https://cloud.google.com/vpc-service-controls/docs/use-access-levels)](#AccessContextLevelshttps:cloud.google.comvpc-service-controlsdocsuse-access-levels)
	* [Remote Access with IAP](#RemoteAccesswithIAP)
		* [IAP access into a windows VM](#IAPaccessintoawindowsVM)
		* [[IAP TCP Forwarding Overview](https://cloud.google.com/iap/docs/tcp-forwarding-overview)](#IAPTCPForwardingOverviewhttps:cloud.google.comiapdocstcp-forwarding-overview)

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

--------------------------------------

### <a name='FunLinks'></a>Fun Links

Enjoy!

The ultimate [infographic on GCP products](https://googlecloudcheatsheet.withgoogle.com/) for Developers

Check your regions latency using [GPCping](https://gcping.com/)

### <a name='ConfigController'></a>Config Controller

#### <a name='ConfigControllerOverviewhttps:cloud.google.comanthos-config-managementdocsconceptsconfig-controller-overview'></a>[Config Controller Overview](https://cloud.google.com/anthos-config-management/docs/concepts/config-controller-overview)

Config Controller provides a managed control plane, based on Kubernetes. In addition, Config Controller instances come pre-installed with Policy Controller, Config Sync, and Config Connector. By using these components, you can leverage the tools and workflows of Kubernetes to manage Google Cloud resources and achieve consistency by using a GitOps workflow.
To learn more, see the Config Controller overview.

### <a name='PolicyController'></a>Policy Controller

#### <a name='PolicyControllerOverviewhttps:cloud.google.comanthos-config-managementdocsconceptspolicy-controller'></a>[Policy Controller Overview](https://cloud.google.com/anthos-config-management/docs/concepts/policy-controller)

Policy Controller enables the enforcement of fully programmable policies for your clusters. These policies act as "guardrails" and prevent any changes to the configuration of the Kubernetes API from violating security, operational, or compliance controls.
You can set policies to actively block non-compliant API requests, or simply to audit the configuration of your clusters and report violations. Policy Controller is based on the open source Open Policy Agent Gatekeeper - [OPA](https://open-policy-agent.github.io/gatekeeper/website/docs/operations/) project and comes with a full library of pre-built policies for common security and compliance controls.
In addition to actively controlling your Kubernetes environment, you can optionally use Policy Controller as a way to analyze configuration for compliance before deployment. This helps provide valuable feedback during the process of configuration changes and ensures any non-compliant changes are caught early before they might be rejected during application.

### <a name='ConfigSync'></a>Config Sync

#### <a name='ConfigSyncOverviewhttps:cloud.google.comanthos-config-managementdocsconfig-sync-overview'></a>[Config Sync Overview](https://cloud.google.com/anthos-config-management/docs/config-sync-overview)

Config Sync is a GitOps service offered as a part of Anthos. Config Sync is built on an open source core and lets cluster operators and platform administrators deploy configurations from a source of truth. The service has the flexibility to support one or many clusters and any number of repositories per cluster or namespace. The clusters can be in a hybrid or multi-cloud environment.

Config Sync continuously reconciles the state of Config Controller with files stored in one or more Git repositories. This GitOps strategy lets you manage and deploy common configurations with a process that is auditable, transactional, reviewable, and version-controlled.

Getting started with [Config Sync](https://cloud.google.com/anthos-config-management/docs/tutorials/config-sync-multi-repo)

#### <a name='ToolsandUtilities'></a>Tools and Utilities

[Nomos](https://cloud.google.com/anthos-config-management/docs/how-to/nomos-command)
[Kubectl](https://cloud.google.com/anthos-config-management/docs/how-to/configure-config-sync-kubectl)

### <a name='kpt'></a>kpt

[KPT Readme.md](https://github.com/GoogleContainerTools/kpt/#readme) is an open source project used to hydrate yaml, get packages, apply functions, search and replace in yaml manifests. kpt is a package centric toolchain that enables a WYSIWYG configuration authoring and authoring enabling tool.

kpt [installation](https://kpt.dev/installation/)

kpt [functions catalog](https://catalog.kpt.dev/?id=curated-functions-catalog)

**Important:**  Please note that kpt is available through the gcloud components, however you may wish to use a specific version, or newer version than that is provided by the gcloud SDK.

### <a name='DNS'></a>DNS

#### <a name='DNSOverviewhttps:cloud.google.comdnsdocsdns-overview'></a>[DNS Overview](https://cloud.google.com/dns/docs/dns-overview)

High level overview of some of the Google Cloud resource types to accelerate the design and deployment of your *workload*.

#### <a name='GeneralDNSknowledge'></a>General DNS knowledge

- DNS server types *(Authoritative, Recursive)*
- Zones *(Public, Private, Delegated, Split brain/horizon)*
- Records *(A, AAAA, Cname, Mx, NS, PTR, SOA)*

#### <a name='GCPDNSKeyTerms'></a>GCP DNS Key Terms

[Key Terms](https://cloud.google.com/dns/docs/key-terms)

#### <a name='GoogleCloudSpecificDocumentation'></a>Google Cloud Specific Documentation

[Cloud DNS Overview](https://cloud.google.com/dns/docs/overview/)

[Cloud Domains Overview](https://cloud.google.com/domains/docs/overview)

#### <a name='DNSZones'></a>DNS Zones

#### <a name='InternalDNS'></a>Internal DNS

Internal DNS names are names that Google Cloud creates automatically. Available in a single VPC by default, resolved by the metadata server (169.254.169.254). Google Cloud automatically creates, updates, and removes these DNS records. Please see [Internal DNS zone documentation](https://cloud.google.com/compute/docs/internal-dns).

#### <a name='PrivateZones'></a>Private Zones

Managed custom domain names for virtual machines and other GCP resources without exposing the DNS data to the public internet. Please see [Private Zone Documentation](https://cloud.google.com/dns/docs/zones#create-private-zone).

Unless you have specified an alternative name server in an outbound server policy, Google Cloud first attempts to find a record in a private zone (or forwarding zone or peering zone) authorized for your VPC network before it looks for the record in a public zone.

##### Options

- [Forward Queries to another server](https://cloud.google.com/dns/docs/zones/forwarding-zones)
- [DNS Peering](https://cloud.google.com/dns/docs/zones/peering-zones)
- [Managed Reverse Lookup Zone](https://cloud.google.com/dns/docs/zones/managed-reverse-lookup-zones)

#### <a name='PublicExternalDNSZone'></a>Public, External DNS Zone

Create DNS records in public zones to publish your service on the internet. Please see [Public Zone Documentation](https://cloud.google.com/dns/docs/dns-overview#public_zone).

##### Anycast Resolution
>>
>> Google DNS has some fame associated to it for it's resiliency and global availability properties, with publicly available resolvers such as:
>>> 8.8.8.8
>>> 8.8.8.4

These resolvers are *anycast addresses* with requests being routed to the nearest location advertising the address.

#### <a name='CrossProjectBinding'></a>Cross Project Binding

Cross project binding is used to manage private DNS zones that service another project in the same organization.

[Cross project binding Overview](https://cloud.google.com/dns/docs/zones/zones-overview#cross-project_binding)
[Creating a Zone with cross project binding](https://cloud.google.com/dns/docs/zones/cross-project-binding)

### <a name='CloudArmor'></a>Cloud Armor

#### <a name='CloudArmorOverviewhttps:cloud.google.comarmordocscloud-armor-overview'></a>[Cloud Armor Overview](https://cloud.google.com/armor/docs/cloud-armor-overview)

Google Cloud Armor is a DDoS and application defense service the helps protect your *service*. Cloud Armor is tightly coupled with the Global HTTPS(S) Cloud Load Balancer. Cloud Armor protects your network services, typically behind a load balancer from DOS and other web based attacks. Enforcement is managed at the edge Point of Presence (POP), as close to the source traffic as possible. Traffic can be filtered and defended against at layer 7 of the OSI model.

##### Cloud Armour Encompasses

- Policies/Rules
- Allowlist / Denylist IP's

##### Cloud Armor Policies & Ordering

- Rules (allow) [Priority 1]
- Rules (Deny) [Priority 2]

##### How it works

- Create a policy with +1 rules
  - Supply IP range(s) to apply rule to
  - On rule/IP match: Allow/Deny traffic
    - Overlap rules with different priorities
  - Apply policy to Targets
    - Target = Load balanced backend services

##### Example Policy Scopes

- Create multiple rules, overlapping to create finer grained rules.
- SQL Injection Prevention (CSS), Geo based access control.
- Allows edge traffic blocking.

#### <a name='CloudArmorDocumentationofInterest'></a>Cloud Armor Documentation of Interest

[Use cases](https://cloud.google.com/armor/docs/common-use-cases)
[Security Policies](https://cloud.google.com/armor/docs/configure-security-policies#https-load-balancer)
[Rules Language Reference](https://cloud.google.com/armor/docs/rules-language-reference)
[Cloud Armor Standard VS Managed Protection Plus](https://cloud.google.com/armor/docs/managed-protection-overview#standard_versus_plus)

### <a name='CloudLoadBalancing'></a>Cloud Load Balancing

#### <a name='CloudLoadBalancingOverviewhttps:cloud.google.comload-balancingdocsload-balancing-overview'></a>[Cloud Load Balancing Overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview)

A load balancer distributes user traffic across multiple instances of your applications. By spreading the load, load balancing reduces the risk that your applications experience performance issues.
Cloud Load Balancing is a fully distributed, software-defined managed service. It isn't hardware-based, so you don't need to manage a physical load-balancing infrastructure.

Google Cloud offers the following load-balancing features:

- Single anycast IP address
- Software-defined load balancing
- Seamless autoscaling
- Layer 4 and Layer 7 load balancing
- External and internal load balancing
- Global and regional load balancing
- Advanced feature support

### <a name='FirewallRulesCreation'></a>Firewall Rules Creation

#### <a name='VPCFirewallRulesOverviewhttps:cloud.google.comvpcdocsfirewalls'></a>[VPC Firewall Rules Overview](https://cloud.google.com/vpc/docs/firewalls)

Virtual Private Cloud (VPC) firewall rules apply to a given project and network.
VPC firewall rules let you allow or deny connections to or from virtual machine (VM) instances in your VPC network. Enabled VPC firewall rules are always enforced, protecting your instances regardless of their configuration and operating system, even if they have not started up.

### <a name='VPCServiceControls'></a>VPC Service Controls

#### <a name='VPCServiceControlsOverviewhttps:cloud.google.comvpc-service-controlsdocsoverview'></a>[VPC Service Controls Overview](https://cloud.google.com/vpc-service-controls/docs/overview)

#### <a name='ImportantTerminologyhttps:cloud.google.comvpc-service-controlsdocsoverviewterminology'></a>*Important* [Terminology](https://cloud.google.com/vpc-service-controls/docs/overview#terminology)

A service perimeter creates a security boundary around Google Cloud resources. You can configure a perimeter to control communications from virtual machines (VMs) to a Google Cloud service (API), and between Google Cloud services. A perimeter allows free communication within the perimeter but, by default, blocks communication to Google Cloud services across the perimeter. The perimeter does not block access to any third-party API or services in the internet. [Reference doc.](https://cloud.google.com/vpc-service-controls/docs/overview#isolate)

#### <a name='QuickStart'></a>Quick Start

[Learn](https://cloud.google.com/vpc-service-controls/docs/set-up-service-perimeter) how to set up a service perimeter using VPC Service Controls in the Google Cloud console.

#### <a name='Generalinformation'></a>General information

VPC service controls allow blocking or restriction of api services at the Project or VPC network level

- Protects against data exfiltration
- Create a perimeter
  - Apply to a Project
  - Add services to restrict such as the storage API

#### <a name='AccessContextLevelshttps:cloud.google.comvpc-service-controlsdocsuse-access-levels'></a>[Access Context Levels](https://cloud.google.com/vpc-service-controls/docs/use-access-levels)

### <a name='RemoteAccesswithIAP'></a>Remote Access with IAP

#### <a name='IAPaccessintoawindowsVM'></a>IAP access into a windows VM

IAP Desktop is a Windows application that lets you manage multiple Remote Desktop connections to Windows VM instances. IAP Desktop connects to VM instances by using Identity-Aware Proxy TCP forwarding and does not require VM instances to have a public IP address.
Before you connect by using IAP Desktop, make sure that the following prerequisites are met:

- You've configured your VPC to allow IAP traffic to your VM instance.
- You've downloaded and installed IAP Desktop on your local computer.

#### <a name='IAPTCPForwardingOverviewhttps:cloud.google.comiapdocstcp-forwarding-overview'></a>[IAP TCP Forwarding Overview](https://cloud.google.com/iap/docs/tcp-forwarding-overview)

IAP's TCP forwarding feature lets you control who can access administrative services like SSH and RDP on your backends from the public internet. The TCP forwarding feature prevents these services from being openly exposed to the internet. Instead, requests to your services must pass authentication and authorization checks before they get to their target resource.
