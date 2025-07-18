# FluxCD Metabase GitOps Demo

This repository demonstrates a GitOps workflow for deploying [Metabase](https://www.metabase.com/) on Kubernetes using [FluxCD](https://fluxcd.io/), [Kustomize](https://kustomize.io/), and the [Delivery Hero Metabase Helm chart](https://artifacthub.io/packages/helm/deliveryhero/metabase).

---

## Repository Structure

```
.
├── apps/
│   ├── base/
│   │   └── metabase/
│   │       ├── kustomization.yaml
│   │       ├── namespace.yaml
│   │       ├── repository.yaml
│   │       └── release.yaml
│   └── staging/
│       ├── kustomization.yaml
│       └── metabase/
│           ├── kustomization.yaml
│           └── metabase-values.yaml
├── clusters/
│   └── staging/
│       ├── apps.yaml
│       ├── infrastructure.yaml
│       ├── flux-system/
│       │   ├── gotk-components.yaml
│       │   ├── gotk-sync.yaml
│       │   └── kustomization.yaml
│       └── image-automation/
│           ├── imagepolicy-metabase.yaml
│           └── imageupdateautomation.yaml
├── infrastructure/
│   ├── configs/
│   │   ├── cluster-issuers.yaml
│   │   └── kustomization.yaml
│   └── controllers/
│       ├── cert-manager.yaml
│       ├── ingress-nginx.yaml
│       └── kustomization.yaml
└── script/
    └── validate_sh
```

---

## Key Components

### 1. **Metabase Deployment (HelmRelease)**
- Uses the [Delivery Hero Metabase Helm chart](https://artifacthub.io/packages/helm/deliveryhero/metabase).
- Chart version: `0.14.3`
- Values are managed via overlays for different environments (e.g., `staging`).

### 2. **Kustomize Structure**
- `apps/base/metabase/`: Base Kustomize and HelmRelease for Metabase.
- `apps/staging/metabase/`: Staging overlay with environment-specific values.
- `apps/staging/kustomization.yaml`: Includes the `metabase` overlay.

### 3. **FluxCD Kustomizations**
- `clusters/staging/apps.yaml`: Reconciles the `apps/staging` directory.
- `clusters/staging/infrastructure.yaml`: Reconciles the `infrastructure` directory.
- `clusters/staging/flux-system/`: Contains Flux bootstrap and sync manifests.

### 4. **Infrastructure**
- `infrastructure/controllers/`: Manifests for controllers like cert-manager and ingress-nginx.
- `infrastructure/configs/`: Cluster-wide configs like ClusterIssuer for cert-manager.

### 5. **Image Automation**
- `clusters/staging/image-automation/`: Contains `ImagePolicy` and `ImageUpdateAutomation` for automated image updates.

### 6. **Validation Script**
- `script/validate_sh`: Bash script to validate all manifests and overlays using `kubeconform`, `kustomize`, and `yq`.

---

## How to Use

### Prerequisites

- Kubernetes cluster
- FluxCD installed and bootstrapped
- [Helm](https://helm.sh/), [kustomize](https://kustomize.io/), [yq](https://github.com/mikefarah/yq), [kubeconform](https://github.com/yannh/kubeconform) (for validation)
- Cluster has internet access to `https://charts.deliveryhero.io/`

### 1. **Validate Manifests**

```sh
./script/validate_sh
```

### 2. **Deploy with FluxCD**

- Push your changes to the Git repository.
- Flux will automatically reconcile the manifests and deploy Metabase.

### 3. **Check Status**

```sh
kubectl get helmrepository -n metabase
kubectl get helmrelease -n metabase
kubectl get pods -n metabase
```

### 4. **Port Forward to Access Metabase**

```sh
kubectl port-forward svc/metabase -n metabase 8000:80
```
Then open [http://localhost:8000](http://localhost:8000) in your browser.

---

## Customization

- **Change Metabase chart version:** Edit `version:` in `apps/base/metabase/release.yaml` and `apps/staging/metabase/metabase-values.yaml`.
- **Override values for staging:** Edit `apps/staging/metabase/metabase-values.yaml`.
- **Add more environments:** Copy the `staging` overlay and adjust as needed.

---

## Troubleshooting

- If you see errors about chart versions, run:
  ```sh
  helm repo add deliveryhero https://charts.deliveryhero.io/
  helm repo update
  helm search repo deliveryhero/metabase
  ```
  and use a version that is actually available.

- If resources are not created, ensure your kustomization files reference all necessary resources and overlays.

---

## References

- [Delivery Hero Metabase Helm Chart on ArtifactHub](https://artifacthub.io/packages/helm/deliveryhero/metabase)
- [FluxCD Documentation](https://fluxcd.io/docs/)
- [Kustomize Documentation](https://kubectl.docs.kubernetes.io/references/kustomize/)
- [Metabase Documentation](https://www.metabase.com/docs/)

---

Let me know if you want to include more details or environment-specific instructions! 