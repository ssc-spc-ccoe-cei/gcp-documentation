# Google Cloud Developer Resources 

## DNS

High level overview of some of the Google Cloud resource types to accelerate the design and deployment of your *workload*  


### General DNS knowledge

Google provides a very good [High level document](https://cloud.google.com/dns/docs/dns-overview) on these subjects:
- DNS server types *(Authoritative, Recursive)*
- Zones *(Public, Private, Delegated, Split brain/horizon)*
- Records *(A, AAAA, Cname, Mx, NS, PTR, SOA)*

#### GCP DNS Key Terms

[Key Terms](https://cloud.google.com/dns/docs/key-terms)

#### Google Cloud Specific Documentation

[Cloud DNS Overview](https://cloud.google.com/dns/docs/overview/)

[Cloud Domains Overview](https://cloud.google.com/domains/docs/overview)  


#### DNS Zones

#### Internal DNS

Internal DNS names are names that Google Cloud creates automatically. Available in a single VPC by default, resolved by the metadata server (169.254.169.254). Google Cloud automatically creates, updates, and removes these DNS records. Please see [Internal DNS zone documentation](https://cloud.google.com/compute/docs/internal-dns)

#### Private Zones
 
Managed custom domain names for virtual machines and other GCP resources without exposing the DNS data to the public internet. Please see [Private Zone Documentation]()

Unless you have specified an alternative name server in an outbound server policy, Google Cloud first attempts to find a record in a private zone (or forwarding zone or peering zone) authorized for your VPC network before it looks for the record in a public zone.

##### Options

* [Forward Queries to another server](https://cloud.google.com/dns/docs/zones/forwarding-zones)
* [DNS Peering](https://cloud.google.com/dns/docs/zones/peering-zones)
* [Managed Reverse Lookup Zone](https://cloud.google.com/dns/docs/zones/managed-reverse-lookup-zones)




#### Public, External DNS Zone

Create DNS records in public zones to publish your service on the internet. Please see [Public Zone Documentation](https://cloud.google.com/dns/docs/dns-overview#public_zone). 


##### Anycast Resolution
>> Google DNS has some fame associated to it for it's resiliency and global availability properties, with publicly available resolvers such as:  
>>>  8.8.8.8  
>>>  8.8.8.4

These resolvers are *anycast addresses* with requests being routed to the nearest location advertising the address.



#### Cross Project Binding

Cross project binding is used to manage private DNS zones that service another project in the same organization.

https://cloud.google.com/dns/docs/zones/zones-overview#cross-project_binding

https://cloud.google.com/dns/docs/zones/cross-project-binding


### Cloud Armor

#### [Cloud Armor Overview ](https://cloud.google.com/armor/docs/cloud-armor-overview)

Cloud Armor is a security focused product the helps protect your *service*. Cloud Armor is tightly coupled with Cloud Load Balancer. Cloud Armor protects your network services, typically behind a load balancer from DOS and other web based attacks. Enforcement is managed at the edge Point of Presence (POP), as close to the source traffic as possible. Traffic can be filtered and defended against at layer 7 of the OSI model. 


##### Cloud Armour Encompasses:

* Policies/Rules
* Whitelist / Blacklist IP's

##### Cloud Armor Policies & Ordering:
- Rules (allow) [Priority 1]
- Rules (Deny) [Priority 2]

##### How it works:

* Create a policy with +1 rules
    * Supply IP range(s) to apply rule to
    * On rule/IP match: Allow/Deny traffic
      * Overlap rules with different priorities
    * Apply policy to Targets
      * Target = Load balanced backend services  

##### Example Policy Scopes:  

* Create multiple rules, overlapping to create finer grained rules.  
* SQL Injection Prevention (CSS), Geo based access control.  
* Allows edge traffic blocking.  

####  Cloud Armor Documentation of Interest

[Use cases](https://cloud.google.com/armor/docs/common-use-cases)  
[Security Policies](https://cloud.google.com/armor/docs/configure-security-policies#https-load-balancer)


### Vpc Service Controls

VPC service controls allow blocking or restriction of api services within a VPC. 

### kpt@

[KPT](https://kpt.dev/?id=overview) is an open source project used to hydrate yaml, get packages, apply functions, search and replace in yaml manifests.  