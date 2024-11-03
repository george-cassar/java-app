
# Java Application CI/CD Deployment on OpenShift using ArgoCD and BuildConfig

This project demonstrates an automated CI/CD pipeline setup using ArgoCD for GitOps-based deployment and OpenShift's BuildConfig for building the Docker image on OpenShift. The application is deployed in two environments: `bog-dev` and `bog-prd`. All configurations, including namespaces, image build, and routes, are managed via Git without the use of webhooks.

## Project Structure

```
java-app/
├── app-code/                  # Application source code and Dockerfile
│   ├── src/
│   ├── Dockerfile
│   └── pom.xml
└── k8s-manifests/             # Kubernetes manifests for ArgoCD deployment
    ├── base/                  # Base configuration used across environments
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   ├── imagestream.yaml
    │   └── kustomization.yaml
    └── overlays/              # Environment-specific overlays
        ├── bog-dev/           # Dev overlay with namespace and image configs
        │   ├── kustomization.yaml
        │   ├── namespace.yaml
        │   ├── buildconfig.yaml
        │   └── route.yaml
        └── bog-prd/           # Prod overlay with namespace and image configs
            ├── kustomization.yaml
            ├── namespace.yaml
        │   ├── buildconfig.yaml
            └── route.yaml
    └── argocd-applications/   # ArgoCD applications for each environment
        ├── argocd-application-dev.yaml
        └── argocd-application-prd.yaml
```

## Step-by-Step Setup Instructions

### Step 1: Push Code and Manifests to Git Repository

1. Fork or clone this repository.
2. Push all application code and Kubernetes manifests to your own Git repository: [https://github.com/george-cassar/java-app.git](https://github.com/george-cassar/java-app.git).

### Step 2: OpenShift Project Setup with ArgoCD

#### ArgoCD will create and manage the `bog-dev` and `bog-prd` namespaces automatically.

### Step 3: Configure BuildConfig for Image Build

1. The BuildConfig in `pipeline/buildconfig.yaml` is used to build and push the Docker image.
2. This file does not use webhooks and allows manual or configuration-based triggering.

### Step 4: Configure Kubernetes Manifests with Kustomize

1. Define common resources (`deployment.yaml`, `service.yaml`, and `imagestream.yaml`) in the `base` directory.
2. Create environment-specific overlays in `overlays/bog-dev` and `overlays/bog-prd` for each namespace and route.

### Step 5: Add Namespace Manifests in Overlays

The `namespace.yaml` files in each overlay ensure namespaces are created automatically by ArgoCD if they don't already exist.

### Step 6: Configure Kustomize Overlays

In each overlay, set up Kustomize to dynamically reference the correct namespace and image path.

### Step 7: Set Up ArgoCD Applications for Each Environment

In `argocd-applications/`, the ArgoCD applications for `bog-dev` and `bog-prd` are defined to monitor and deploy the changes.

### Step 8: Apply ArgoCD Applications

Apply the ArgoCD applications to your OpenShift cluster:

```bash
oc apply -f k8s-manifests/argocd-application-dev.yaml
oc apply -f k8s-manifests/argocd-application-prd.yaml
```

### Step 9: Trigger the Build Manually

Since no webhooks are used, trigger the build manually:

```bash
oc start-build java-app-build -n bog-dev
oc start-build java-app-build -n bog-prd
```

ArgoCD will automatically detect any updates in the Git repository and sync them to the OpenShift environment.

### Summary

This setup automates building and deploying the application to multiple environments using ArgoCD for GitOps-based deployment and OpenShift's BuildConfig for the CI build process.
