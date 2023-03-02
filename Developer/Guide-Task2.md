## Google Cloud Developer Resources

High level overview of some of the Google Cloud resource types to accelerate the design and deployment of your *workload*  

### Public and Private DNS Zones

#### General DNS knowledge

- DNS server types *(Authoritative, Recursive)*
- Zones *(Public, Private, Delegated, Split brain/horizon)*
- Records *(A, AAAA, Cname, Mx, NS, PTR, SOA)*

Google provides a very good [High level document](https://cloud.google.com/dns/docs/dns-overview) on these subjects.  

#### Public, External DNS
Google DNS has some fame associated to it for it's resiliency and global availability properties, with publically available resolvers such as:  

- 8.8.8.8
- 8.8.8.4

These resolvers are *anycast addresses* with requests being routed to the nearest location advertising the address.

#### Google Cloud Specific Documentation

[Cloud DNS Overview](https://cloud.google.com/dns/docs/overview/)

[Cloud Domains Overview](https://cloud.google.com/domains/docs/overview)  




### Cloud Armor


### Vpc Service Controls


### kpt
