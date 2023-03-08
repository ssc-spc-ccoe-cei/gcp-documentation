## Google Cloud Developer Resources - DNS

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

#### Public, External DNS Zone

Create DNS records in public zones to publish your service on the internet. Please see [Public Zone Documentation](https://cloud.google.com/dns/docs/dns-overview#public_zone). 


##### Anycast Resolution
>> Google DNS has some fame associated to it for it's resiliency and global availability properties, with publicly available resolvers such as:  
>>>  8.8.8.8  
>>>  8.8.8.4

These resolvers are *anycast addresses* with requests being routed to the nearest location advertising the address.


### Cross Project Binding

https://cloud.google.com/dns/docs/zones/zones-overview#cross-project_binding





#### 




### Cloud Armor


### Vpc Service Controls


### kpt
