# Artifact Registry

## Overview

Google Cloud Artifact Registry serves as a fundamental solution for management of Docker images and deployment of applications to Google Kubernetes Engine (GKE). Artifact Registry integrates with GKE providing a centralized repository for storing and distributing container images. In addition to storage of images, Artifact Registry provides image versioning and advanced container vulnerability scanning.

## Provisioning

Artifact registry is a tier 4 resource that can be provisioned using the Config Connector [Artifact Registry Repository](https://cloud.google.com/config-connector/docs/reference/resource-docs/artifactregistry/artifactregistryrepository) resource. A typical use case for a container image registry can be provisioned using:

```yml
apiVersion: artifactregistry.cnrm.cloud.google.com/v1beta1
kind: ArtifactRegistryRepository
metadata:
  name: artifactregistryrepository-sample
spec:
  format: DOCKER
  location: northamerica-northeast1
```

References:
[ArtifactRegistryRepository Config Connector Documentation](https://cloud.google.com/config-connector/docs/reference/resource-docs/artifactregistry/artifactregistryrepository)

## Version Management

A repository can contain many container images, and these images can have different versions. To identify a specific version of an image, you can specify the image digest or tag.

**Digest**
An image digest is an automatically generated hash of the image index or image manifest. Each image digest is a unique identifier for an image version and cannot be changed. The digest is the sha256 hash value of the image contents.

**Tag**
An image tag is a label and is often a human-readable string such as `v1.1` or `development`. A tag can only point to only one version of an image. In Artifact Registry, you can configure a Docker repository to allow mutable tags or enforce immutable tags.

- **Mutable Tags**: A tag points to only one version of an image, but the specific digest that it references can change.

A common approach is to tag images with a version identifier, such as `v1.1` at build time. When the build pushes multiple versions of the image to the registry with the same `v1.1` tag, the tag references the digest of the last version pushed to the registry. Although mutable tags provide a convenient way to label versions, they have a disadvantage in that a tag does not uniquely specify a particular version of an image.

- **Immutable Tags**: In the repository, a tag always points to the same image digest.

If a Artifact Registry repository is configured for immutable tags, the following actions are not permitted:
- Delete a tagged image. Deleting untagged images is still permitted.
- Remove a tag from an image.
- Push an image with a tag that is already used by another version of the image in the repository.

In production scenarios, it maybe be desirable to enforce immutable tags to provide strict specificity to the deployed artifacts.

To enable immutable tags on an Artifact Registry, use the `dockerConfig.immutableTags` field:

```yml
apiVersion: artifactregistry.cnrm.cloud.google.com/v1beta1
kind: ArtifactRegistryRepository
metadata:
  name: artifactregistryrepository-sample
spec:
  format: DOCKER
  location: northamerica-northeast1
  dockerConfig:
    immutableTags: true
```

References:
[Repository and image names](https://cloud.google.com/artifact-registry/docs/docker/names)

## Pushing Container Images

1. Authenticate to the repository:

```bash
gcloud auth configure-docker us-east1-docker.pkg.dev
```

2. Tag the local image with the repository name:

```bash
docker tag image-name northamerica-northeast1-docker.pkg.dev/project-id/artifactregistryrepository-sample/image-name:tag
```

3. Push the tagged image to Artifact Registry:

```bash
docker push northamerica-northeast1-docker.pkg.dev/project-id/artifactregistryrepository-sample/image-name:tag
```

References:
[Push and pull images](https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling)


## Application Image Deployment

To deploy an application to a Kubernetes cluster, simply specify the reference to the container image in the `image` field of the `containers` object in the manifest for the Kubernetes workload (e.g. `Deployment`, `StatefulSet`, `DeamonSet`, `CronJob`).  Below is a simple Deployment example:

```yml
piVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  labels:
    app: sample-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-app
        image: northamerica-northeast1-docker.pkg.dev/project-id/artifactregistryrepository-sample/image-name:tag
        ports:
        - containerPort: 80
```

References:
[Deployments Kubernetes Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

## Cluster Access Management

Each node in a GKE cluster has an Identity and Access Management (IAM) Service Account associated with it.  This is a custom Service Account provisioned specifically for the cluster nodes. The Service Account has been granted the necessary permissions to access Artifact Registry repositories in the application project.  This enables the GKE cluster to pull images from the artifact registry repository and run containerized applications.

## Vulnerability Scanning

Artifact Analysis provides vulnerability information for the container images in Artifact Registry.  This capability is automatically enabled when the GKE application project is provisioned and configured.  No further actions are required by the application teams.

To view vulnerabilities for an image:

```bash
gcloud artifacts docker images list --show-occurrences northamerica-northeast1-docker.pkg.dev/project-id/artifactregistryrepository-sample/image-name
```

To view vulnerabilities for an image tag:

```bash
gcloud artifacts docker images describe --show-package-vulnerability northamerica-northeast1-docker.pkg.dev/project-id/artifactregistryrepository-sample/image-name
```

Referencees:
[Scan images for OS vulnerabilities automatically](https://cloud.google.com/artifact-analysis/docs/os-scanning-automatically)