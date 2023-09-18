# GKE Deployment 

[**Link to Demo**](https://163gc.sharepoint.com/:v:/r/sites/SSCSPC-CTODPT-GCP/Shared%20Documents/Videos/demo%20-%20gke%20package%20deployment-20230822_150526-Meeting%20Recording.mp4?csf=1&web=1&e=bYRk7H)

- [GKE Deployment](#gke-deployment)
  - [Background Information](#background-information)
  - [Steps](#steps)
                   



## Background Information

No PR process, pushes are done in main. Root-sync and repo-sync are configured to observe HEAD instead of specific tags. When pushed to the observed branch (usually main), the new commit will be applied automatically. 

Part of the resource that needs to be deployed is a Google Managed Certificate. SSL certificate attached to the external load balancer for https for the web service. The custom resource definition (crd) for the resource isn’t released yet at the beginning so it is in alpha state. Loaded the alpha resource inside config controller so we can deploy a Google managed certificate. The only issue with the package is the Google Managed Certificate.

At the end there will be a namespace ready for the app team to publish their manifest files/application. Different projects can live on the same gke cluster just different namespaces.

**Tier2** repository will include: **Client-landing-zone package**, **Projects each with a client-project-setup** and two packages which will be added which are the **crds** and **gke-setup**

### Required Information 

•	You have a host project where the network is. Tier34 repository must be ready and project must be created for the client. This can be done using the client-setup package

•	You have a service project you deploy for an app team.

•	2 services accounts. Tier3 service account and Tier4 services account.

•	Deployment of a cluster will be done in the service project.

•	The project Id should not be application specific, it should be for an “app team”.

**Tier1** repository which should include: **Core-landing-zone** package and **Client-setup package**

For more information on each package, consult the README.md for each of those packages. 

## Steps

### GKE Package setup in tier2

1.	Run a <<nomos status –contexts gke_${PROJECT_ID}_northamerica-northeast1_krmapihost-${CLUSTER}>> to see what the sync status is on the resources in the cluster.

2.	In the console, go to the project where the config controller cluster is built.

3.	The certificate will be stuck on “in Progress” – that is normal

4.	Go into source-base of the project you are working in

5.	Add a package: cd tier2/configcontroller/source-base/projects/projectname/

    - run: kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/gke/configconnector/gke-setup@0.2.0

    - to find the version head over to: https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit/blob/main/solutions/gke/configconnector/gke-setup/CHANGELOG.md

recall if you forget, the official step is to define the following 

export TIER=

export REPO_URI=

export PKG_PATH=

export VERSION = located in the package CHANGELOG.md, use ‘main’ if not available

export LOCAL_DEST_DIRECTORY=

if you do not remember how to add a package, the information can be found in ssc-spc-ccoe-cei/gcp-documentation/Landing Zone Operations/Changing.md

6.	Customize the setters

    - In your folder explorer in VSCode, go to source-customization/dev/projects/projectname

    - In source-customization, create an empty “gke-setup” folder

    - Copy the setters from the gke-setup package into the gke-setup folder created above


**Information for the setters.yaml in “gke-setup”**:
Some information can be found in the 

tier2/configcontroller/source customization/dev/client-landing-zone

AND/OR

tier1/clients/ssc-spc/client-setup/setters.yaml

-	**org-id:** in the console, click on the top left where you can navigate between projects. Look for the org name your project is in and copy the ID on the right side of the name.  

-	**gke-monitoring-group:** team email address of the platform admin. 

-	**client-name:** value used inside the client landing zone

-	**client-management-project-id:** Same as what is in the tier2/configcontroller/source-customization/dev/clients/ssc-spc/client-setup


-	**host-project-id:** same as tier2/configcontroller/source-customization/dev/client-landing-zone/setters.yaml

-	**Project-id:** ?

-	**Project-number:** In the console, make sure you are in the project you are working in. Go to the dashboard and you’ll see the Project number there. 

7.	cd to the root of the repository up to tier2/

8.	run "kpt-hydrate or bash tools/scripts/kpt/hydrate.sh"

9.	run a "git status"

10.	Review the changes in VSCode’s source control. In VSCode, on the left hand side, click on source-control. 

11.	run a "git add ."

12.	In source-control on VScode, in the commit section type in: feat: adding gke-setup package to “project name” and click on the arrow point down beside the commit button and click on commit and push. 

13.	Run "git status to see if the branch is up to date"

14.	Run a "nomos status –contexts gke_${PROJECT_ID}_northamerica-northeast1_krmapihost-${CLUSTER}"

15.	The “client name”-t2 should be shown as pending because the the rootsync is applying the changes. 

16.	In the VsCode file explorer go to Tier2/configcontroller/deploy/dev/projects/gke-setup/services.yaml, in there you will see all the setters that were customized.  

17.	 In the console, you should see that the certificates won’t be working in Kubernetes Engine. 

### Gke-defaults package setup in tier3/configcontroller

Deploys resources for the gateway. The gateway controller in Kubernetes is the implementation of the gateway API. When you are building a gateway in the controller, you are deploying a global external load balancer. The load balancer will require an IP from Google Cloud to be reserved.  

1.	cd tier3/configcontroller/source-base

2.	Run a "kpt pkg get 
https://github.com/GoogleCloudPlatform /pubsec-declarative-toolkit.git/solutions/gke/configconnector/gke-defaults@0.2.0"

3.	Create a new empty folder in tier3/source-customization called “gke-default”. 

4.	Under gke-default, create a “gateway-setup” folder, a “dns” folder under that and a “ssl-certificate” folder under that. 

5.	Copy the setters,yaml from under the “gke-defaults” folder and paste it under the “gke-defaults folder you created in source-customization.  

6.	Copy the setters.yaml from under the “gateway-setup” folder and paste it under the “gateway-setup” folder you created in source-customization. 

7.	Copy the setters.yaml from under the “dns” folder and paste it under the “dns” folder you created in source-customization

8.	Copy the setters.yaml from under the “ssl-cetificate” folder and paste it under the “ssl-certificate” folder you created in source-customization

**Information for the setters.yaml in “gke-defaults”**

-	**client-name:** “client name you are working to create the project for”

-	**team-gkeviewer:** group that will having gke logging and monitoring viewer permissions

-	**host-project-id: ?**


-	**project-id: ?**

**Information for the setters.yaml in “gateway-setup”**

-	**gateway-name:** name of the gateway resource

-	**project-id:** project id that was created by the client-project-setup 

-	**dns-project-id:** Project that has the public dns zone. Usually the client host project that was created by 

-	**metadata-name:** Kubernetes resource name (needs to be unique)

-	**spec-name:** The DNSRecordset.spec.name

-	**client-name:** kpt run will override this as the parent package

-	**project-id:**  kpt run will override this as the parent package

**Information for the setters.yaml in “ssl-certificate”**

-	**certificate-id:** the unique identifier for the resource certificate

-	**certificate-name:** a resource name for the certificate (create it yourself)

-	**domains:** whenever you create a certificate you can have 100 domains. It is a list. Avoid the ending dot. 

9.	Run a "kpt hydrate"

10.	Run a "git status" and review changes

11.	Run a "git add."

12.	Run "git commit -m ‘feat: adding tier3 configcontroller gke-defaults package’"

13.	git push 

### Cloud Armor Policies 

Lives in your loadbalancer where you have your front-end which is your Public IP which lives in the points of presence in Google’s network. The front end service lives inside a Google Cloud datacentre (in our case it is Montreal). You attach the Cloud-Armor Policy to the backend service. In other words, Cloud Armor is the Web Application Firewall solution. 

1. Go inside tier3/source-base and paste the cloud armor-policies folder

2. Copy the setters.yaml

3. Create the cloud-armor-policies folder in source customization

4. Paste the setters.yaml in the cloud-armor-policies folder

**information for the setters.yaml in "cloud-armor-policies setters"**

-	**Project-id:** the project id that was created by the client-project-setup

-	**Workload-name:** workload


5.	Run a “nomos status –contexts gke_${PROJECT_ID}_northamerica-northeast1_krmapihost-${cluster}"

6.	You will see that the dnsrecordset is stuck InProgress. Go under tier3/configcontroller/gke-defaults/gateway-setup/dns/dns.yaml. Put everything in comments for now because it won’t be able to deploy successfully. 

7.	Run a "kpt hydrate"

8.	Run "nslookup" on the dns record in command prompt

9.	In the console go to Security/Certificate Manager

10.	Inside the certificate details go to the blue-certificate to see what is the status on its provisioning. 

11.	Run git status to see the changes on the dns. It should be deleted from the deploy folder. In addition the security-policy.yaml is being added at the same time. 

12.	Usually we will do 2 separate commits. In the source-control section, we will remove all the changes except for the dns.yaml commits. 

13.	In the textbox type: fix: disabling dns record. Click on commit. 

14.	Add the other changes and in the textbox type: feat: adding cloud armor policy. Click on commit

15.	Run a "git push"

### SSL-Policies Package (tier3 - configcontroller)

**SSL Policies Package:** Attach it to your front end. Define what are the cifers you’ll allow for communication with your backend servers. The front end terminates the ssl communication. Therefore, when a client tried to establish communication with your servers, it negotiates through cifers. The web server(load balancer in the front end), it defines a list of cifers it allows for communication. CCCS defines the “allow” list. 

1.	Go to tier3 source base and paste the ssl-policies package

2.	Go to the console and confirm that the Cloud Armor Policy has been provisioned. Network Security/Cloud Armor Policy and make sure the security-policy is there. 

3.	In VSCode, create the ssl-policies folder and paste the setters.yaml from the ssl-policies in source-base. 

**Information for the setters in "ssl policies"**

-	**project-id:** the the project ID that was created by the client-project-setup 

4.	Run the "hydrate.kpt"

5.	Run a "git status"

6.	Run "git add." , "git commit -m ‘feat: adding ssl policies package", and "git push"

### GKE-cluster-autopilot

1.	Add the gke-cluster-autopilot package to source-base in tier3

2.	Under source-customization
/dev create a folder called gke-cluster-autopilot

3.	Copy the setters.yaml to the gke-cluster-autopilot folder created above. 

**Information for the setters in "gke-cluster-autopilot**

-	Project-id: The project id that was created by the client-project-setup. 

-	Host-project-id: the client network host project id

-	Host-project-vpc: VPC to deploy the subnet – K8S resource name

Inside our code when we are deploying our subnet, we are referencing the vpc resource. Adds a Kubernetes metadata name. We need that value in the host-project.vpn

To find it, go into tier2/configcontroller/deploy/dev/client-landing-zone/client-folder/standard/application-infrastructure/host-project/network/vpc.yaml

You will find the name under the ComputeNetwork

-	**Cluster-name:** name of the GKE cluster

-	**location:** the region where to deploy the GKE cluster: northamerica-northeast1

-	**masterIpv4CidrBlock:** 172.16.0.32/28 the master control plane cidr for kind ContainerCluster. Make sure you’re not reusing a range that is already used by another cluster. Must be unique.

There is a document describing how the Cloud Operations team is going to be using its IP ranges inside the landing zone. It is IP access management. 

-	**masterIpv4Range:** 172.16.0.48/28 same as masterIpv4CidrBlock but can be a list. 

-	**master-authorized-networks-cidr:** the bastion Ip. We are building the list of ip that are allowed to connect to the cluster. You do not deploy a bastion host per cluster. One bastion host to many clusters. Right now there is an issue with routing for the bastion hosts per region. 

- **cidrBlock:** 10.1.1.5.32

**Subnet IP ranges:** 
-	subnet-primary-cird: 10.1.40.0/24
-	subnet-services-cidr: 10.1.41.0.24
-	subnet-pod-cidr: 240.1.0.0/20

Have to have a discussion with the app team. Size the network according to the outcomes of the discussion. Everytime you assign an IP range to a cluster. You need to do some calculations. Default is /24. You get around 32 or 64 clusters per client.

-	**podIpv4Range:** 240.1.0.0/21. The pod range for kind ComputeFirewall (same as subnet-pod-cidr but using list)

-	**primaryIpv4Range:** 10.1.32.0/24. The primary range for kind ComputeFirewall (same as subnet-primary-cidr but using list)

**firewall policies priority** (cannot overlap with any existing policy id)

-	gke-to-azdo-priority: 2006

-	gke-to-github-priority: 2007

-	gke-to-docker-priority: 2008

For the variables above, the policies are deployed at the standard application infrastructure. In the org where the project is, in the folder hierarchy you created, under the client folder, click on the application-infrastructure folder. In Netork Security, click on Firewall policies on the sidebar and there click on the firewall policy located under the "Firewall policies located in this node". 

There you will see the list of firewall rules inside the policy. GKE has the three rules created above. Everytime a cluster is created, the firewall policies are automatically created. 

The Platform Admin approves the firewall policies. The GKE firewall policies are tier3 resources. Any change in tier3 is approved by the SSC security admins and the platform admins. 

The Priority for the firewall policies are not strict but must be agreed upon by the team. The numbers used in the demo start at 2000. 

**repo-url:** the cluster will be observing one folder/repo and branch. It would be the tier34 repo. 

**repo-branch:** leave as main

**repo-dir:** csync/tier3/kubernetes/"x-fleet-id"/deploy/dev

4. Run "kpt-hydrate"

5. Run "git status"

6. Run "git add ."

7. Run "git commit -m "feat: adding gke-cluster-autopilot package" 

8. Run "git push"

Using the repository the client will have the necessary privileges in tier4 to put their manifest yaml files. They will not need to request approval from the platform administrator. Deploying an application is not a platofrm admin or security admin role. If the application needs to talk to the outside world or there needs to be firewall rules, that is done in tier3. The platform admins will review/approve the firewall rules in that tier for the client. 

In the config controller cluster, it has deployed the tier1 landing zone, 1 client setup package for the client level resources, 3 service projects that are attached to the host project, 3 onboarded gke clusters. 

### Anthos Config Management 

1. In the console navigate inside the host project

2. On the hamburger menu, navigate to Compute Engine/VM Instance. You will see the bastion host. 

3. Click on SSH and it will let you into the VM

4. Run nomos status inside the bastion. It will show you the status of all of the clusters inside the output. It will also show you the repo syncs. 

5. In the Google Cloud console, in the hamburger menu, click on the Kubernetes Engine and click on clusters. On the right hand side of the cluster, click on the three dots and click on "Connect". There you will copy the command and paste it into your VSCode to connect to the cluster.

6. Run "kubectl configget-contexts" to see which cluster you are on. 

7. Run "Kubectl get nodes".  This will show you the nodes in the cluster. 

The cluster is building the worker pool. By deploying the Anthos config management feature, there are pods that have been deployed on the cluster. It is building the worker pool to support the Anthos config management pods and same thing for policy controller. 

Next in the bastion host shell, we will create the git creds secret so that the cluster we are connected to can read the Azure DevOps repository. 

8. We will need the username which is the name of the Azure DevOps organization and the token. We will add those inside the bastion host shell command prompt and export them. 

9. Inside the bastion host sehll prompt we will add the 

```shell
kubectl create secret generic git-creds --namespace="config-management-system" --from-literal=username =${USERNAME} --from-literal=token=${TOKEN}
```
10. Run "nomos status"

The cluster needs to build the nodes so it can run the Anthos Config Management pods so that all of the root-sync can work. 

The root-sync as configured inside the gke cluster autopilot package, is observing the tier34 repo, the csync/tier3/kubernetes. This is the first time the team works with a Kubernetes layer inside the repo-structure. 

Similar to what is in the configcontroller. The cluster first observes the csync so that we can use the root-sync package to manage the version we wantt o have deployed to our cluster. 

11. In the repo structure in VSCode, navigate to tier3/kubernetes/source-customization/root-sync-git/setters.yaml

12. under id: change project-id-12345 to the project-id of the project you created.

13. Run "kpt hydrate"

14. Run "git add ."

15. Run "git status"

16. Run "git commit -m 'fix: csync tier3 kubernetes roots-sync'

17. Run "git push"

### Cluster-defaults package (tier3 kubernetes)

1. cd into the "tier34 repo"/tier3/kubernetes/"project"/source-base

2. Run "kpt pkg get https://github.com/GoogleCloudPlatform/pubsec-declarative-toolkit.git/solutions/gke/kubernetes/cluster-defaults@0.2.0"

Consult the README.md for package details

3. Copy the setters.yaml file

4. Under source-customization create a new folder called "cluster-defaults" and under that folder, create a folder called "gateway"

5. Move the setters.yaml for the gateway from source-base inside the gateway folder in source-customization you just created. 

6. Under the gateway folder in source-customization, create another folder called backendpolicy. In there, paste the setters.yaml from the backendpolicy in the source-base. 

**Information for the setters in "gateway**

**gateway-name**: name of gateway to be created

**ssl-policy**: go inside tier3/configcontroller/deploy/dev/ssl-policies and look inside the ssl-policy-tls-1.2.yaml. Should be the same as what is currently shown in the setters.yaml. 


**ssl-cert**: name of certificate

**Information for the setters in "backendpolicy**

**workload-name**: name of cloud-armor

**service-name**: same as the workload-name

service-name is the name of the service resource we deploy in the lower tiers. The app deployment, the service, the http route of the resource we are expecting to build inside tier4. The app team is fully responsible for the resources. The |cloud-armor policy needs to be attached to a service. The service needs to be defined. If the service is not ready the resource won't be able to reconcile. 

If all works out, the connection between cloud-armor and the backend service will be made and you will have protection. 

7. Run "kpt hydrate"

8. Run "git add."

9. Run "git commit -m 'feat: adding cluster-defaults package'

10. Run "git push"

under tier3/kubernetes/dlxmu-consol-helloweb/source-base the namespace defaults creates a namespace for an application team for the team to deploy their application. 

11. Wait for the cluster to reconcile

## The End

Consult the README.md for each package for more information on the resources/workloads.










