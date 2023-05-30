# How to complete some Health Checks

## 2023-03-14 | Version: 0.01

### Overview

Some different methods and locations to help assist with health checks and troubleshooting for both the landing zone infrastructure and client environment.

---

## Index

Infrastructure

- [Config Sync](#config-sync)
- [Kubernetes Config Controller (KCC)](#kubernetes-config-controller)
- [gcloud](#gcloud)
- [How to set alerts](#setting-alerts)

Core Environment

- [Central logging project's Logs Explorer](#central-logging-projects-logs-explorer)

Client Environment

- [GCP Logging tip/tricks](#check-the-logs-by-using-the-gui)

---

## Infrastructure

### Config Sync

General

"nomos" is a tool to help troubleshoot Config Sync issues.

How to install nomos

``` gcloud
gcloud components install nomos
```

Check on the overall status of the Config Sync

``` nomos
nomos status
```

This will repeat the nomos status command every 5 seconds

``` nomos
nomos status --poll=5s
```

Checking status by using the GUI

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Select the project that is running the kubernetes cluster
3. Hamburger menu > Kubernetes Engine
4. Left menu expand "Config & Policy"
5. Click on "Config"
6. Take a look at the "Dashboard" & "Packages" pages.

Further reading can be found at the Google site for [Troubleshooting Config Sync](https://cloud.google.com/anthos-config-management/docs/how-to/troubleshooting-config-sync)

---

### Kubernetes Config Controller

Check to see if KCC is running

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

example:
kubectl describe folder.resourcemanager.cnrm.cloud.google.com/tests -n hierarchy
```

Additional kubectl commands can be found at [https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

Take a look at the Kubernetes Object Browser in the GUI

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Select the project that is running the kubernetes project
3. Hamburger menu > Kubernetes Engine
4. Click on "Object Browser"
5. Expand the different Object names and click on the links related to the resource you want further details on.

Check the logs by using the GUI

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Select the project that is running the kubernetes cluster
3. Hamburger menu > Kubernetes Engine
4. Left menu expand click on "Clusters"
5. Click on Name of the cluster
6. Take a look at the "Details", "Nodes", "Storage" & "Logs" pages from the sub menu.

OR

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Select the project that is running the kubernetes cluster
3. Hamburger menu > Logging > Logs Explorer

---

### gcloud

List only the log files that contain log entries

``` gcloud
gcloud logging logs list
```

---

### Setting Alerts

How to set alerts so you don't miss important events
> TODO - which alerts should be set?
---

## Core Environment

### Central logging project's Logs Explorer

How to view the logs located in the central logging project.  From this central logging project's Logs Explorer, you'll be able to see log buckets.

   1. [https://console.cloud.google.com](https://console.cloud.google.com)
   2. Select the central logging project
   3. Hamburger menu > Logging
   4. Left menu take a look though "Logs Explorer"
   5. At the top click on "Refine Scope"
   6. Select "Scope by storage"
   7. Select the bucket
   8. Click Apply

Types of log buckets you should see are below.

Experimentation Organization

- Log bucket for the security logs (Cloud Audit, Access Transparency logs, and Data Access Logs)
- Log bucket for platform and component logs for resources under the tests folder
- Log bucket for client platform and component logs for resources under the clients folder

Dev, Pre-prod and Prod Organizations

- Log bucket for the security logs (Cloud Audit, Access Transparency logs, and Data Access Logs)
- Log bucket for platform and component logs for resources under the services and services-infrastructure folders
- Log bucket for client platform and component logs for resources under the clientN/applications-infrastructure folder
- Log bucket for client platform and component logs for resources under the clientN/auto and clientN/applications

---

## Client Environment

### Check the logs by using the GUI

1. [https://console.cloud.google.com](https://console.cloud.google.com)
2. Select the project that you wish to explore
3. Hamburger menu > Logging
4. Left menu take a look though "Logs Explorer", "Logs Dashboard"
