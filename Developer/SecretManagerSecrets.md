# Overview

Storing sensitive data outside the Kubernetes cluster reduces the risk of unauthorized access to the data. External secret store providers such as [Google Secret Manager](https://cloud.google.com/secret-manager) provide a dedicated service for external storage of sensitive data.

Applications deployed to Google Kubernetes Engine (GKE) clusters can access secrets stored in Secret Manager using Workload Identity.

Each node in a GKE cluster has an Identity and Access Management (IAM) Service Account associated with it. This is a custom Service Account provisioned specifically for the cluster nodes. The Service Account has been granted the necessary permissions to access Google Secret Manager services in the application project. This enables the applications deployed to the GKE cluster to access secrets.

References:
[Secrets Manager Documentation](https://cloud.google.com/secret-manager/docs/overview)

# Creating a secret

To create a secret from a file:

```
gcloud secrets create my-secret --data-file=/tmp/secret --replication-policy=user-managed --locations=northamerica-northeast1
```

To create a secret with value `s3c3t`

```
printf "s3cr3t" | gcloud secrets create my-secret --data-file=- --replication-policy=user-managed --locations=northamerica-northeast1
```

References:
[`gcloud secrets` CLI reference](https://cloud.google.com/sdk/gcloud/reference/secrets)

# Accessing a secret in applications

To access a secret within an application, the application must programmatically retrieve the secret from Google Cloud Secret Manager.  For implementation samples in various programming languages, refer to [the official documentation](https://cloud.google.com/secret-manager/docs/reference/libraries)

References:
[Access secrets stored outside GKE clusters using Workload Identity](https://cloud.google.com/kubernetes-engine/docs/tutorials/workload-identity-secrets)