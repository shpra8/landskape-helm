# Landskape Helm Repository

This repository contains the **Helm chart** for deploying the **Landskape** application on Kubernetes.

Use this chart to deploy Landskape into your environment by providing environment-specific values and creating the required secrets and PVC.

---

## Repository Structure

```text
.
├── Chart.yaml              # Helm chart metadata
├── templates/              # Kubernetes manifests
│   ├── deployment.yaml     # Main Landskape deployment
│   ├── service.yaml        # Service exposing the application
│   ├── networkpolicy.yaml  # Network policies
│   ├── pvc.yaml            # PersistentVolumeClaim for session storage
│   ├── ikeadns.yaml        # DNS configuration
│   └── values/
│       └── value.example.yaml  # Example values file
└── README.MD               # This file
```

- **Templates location:** `templates/`
- **Example values file:** `templates/values/value.example.yaml`

---

## Quick Start

1. Copy the example values file and fill it in for your environment:

   ```bash
   cp templates/values/value.example.yaml values.yaml
   ```

2. Update `values.yaml` with your environment-specific settings.

3. Create the required Kubernetes secret (see [Secret Configuration](#secret-configuration)).

4. Install the Helm chart:

   ```bash
   helm install landskape . -f values.yaml -n <your-namespace>
   ```

---

## Values Configuration

Edit `values.yaml` (copied from `templates/values/value.example.yaml`) to match your environment.

Key fields you must fill in:

| Field | Description | Example |
|-------|-------------|---------|
| `env` | Deployment environment name | `dev`, `staging`, `prod` |
| `image.repo` | Docker image repository URL | `gcr.io/project/landskape` |
| `image.tag` | Image tag or Git SHA | `v1.2.3` or `abc1234` |
| `envSecretName` | Name of the Kubernetes secret holding app env vars | `landskape-secrets` |
| `imagePullSecrets` | Name of the image pull secret | `regcred` |
| `gcp.serviceAccountName` | Full GCP service account email | `orbit-sa@project.iam.gserviceaccount.com` |
| `gcp.projectID` | GCP project ID | `my-gcp-project` |

> Fill all placeholders (`<...>`) in `values.yaml` before deploying.

---

## Secret Configuration

Create a Kubernetes secret with the environment variables required by Landskape.

### Required Secret Content

Create the secret using the following placeholder values. Fill in the blanks according to your environment before applying.

```env
NEXT_PUBLIC_BYPASS_AUTH=false

# Azure AD / Microsoft Entra ID
AZURE_AD_CLIENT_ID=
AZURE_AD_CLIENT_SECRET=
AZURE_AD_TENANT_ID=

# NextAuth Configuration
NEXTAUTH_URL=<deployed_url>
NEXTAUTH_SECRET=<generate with 'openssl rand -base64 32'>

ORBIT_ENV=<env>

INSYNC_ENABLED=false
SKAPE_ENABLED=false
MCP_ENABLED=true
```

### Create the Secret

You can create the secret using a literal command or from an `.env` file.

#### Option A: From an `.env` file

1. Save the filled values above into a file named `secrets.env`.
2. Create the secret:

   ```bash
   kubectl create secret generic landskape-secrets \
     --from-env-file=secrets.env \
     -n <your-namespace>
   ```

#### Option B: Using kubectl literals

```bash
kubectl create secret generic landskape-secrets \
  --from-literal=NEXT_PUBLIC_BYPASS_AUTH=false \
  --from-literal=AZURE_AD_CLIENT_ID=<your-client-id> \
  --from-literal=AZURE_AD_CLIENT_SECRET=<your-client-secret> \
  --from-literal=AZURE_AD_TENANT_ID=<your-tenant-id> \
  --from-literal=NEXTAUTH_URL=<deployed_url> \
  --from-literal=NEXTAUTH_SECRET=$(openssl rand -base64 32) \
  --from-literal=ORBIT_ENV=<env> \
  --from-literal=INSYNC_ENABLED=false \
  --from-literal=SKAPE_ENABLED=false \
  --from-literal=MCP_ENABLED=true \
  -n <your-namespace>
```

> Replace `landskape-secrets` with the value you set for `envSecretName` in `values.yaml`.

---

## PersistentVolumeClaim (PVC)

This chart optionally creates a **PersistentVolumeClaim** for storing per-user kubeconfig and gcloud sessions.

PVC configuration is controlled under `orbitSessions.pvc` in `values.yaml`:

```yaml
orbitSessions:
  mountPath: /landskape-sessions
  pvc:
    enabled: true
    accessMode: ReadWriteMany
    storage: 3Gi
    storageClassName: netapp-nas
```

| Field | Description |
|-------|-------------|
| `enabled` | Set to `true` to create the PVC |
| `accessMode` | Access mode for the volume (`ReadWriteOnce` or `ReadWriteMany`) |
| `storage` | Requested storage size |
| `storageClassName` | Storage class available in your cluster |

> Ensure the storage class specified exists in your cluster before deploying.

---

## Deployment Checklist

Use this checklist before deploying:

- [ ] Copied `templates/values/value.example.yaml` to `values.yaml`
- [ ] Filled in all `<...>` placeholders in `values.yaml`
- [ ] Created the Kubernetes secret with all required environment variables
- [ ] Verified the PVC storage class exists in the target cluster
- [ ] Chosen the correct namespace for deployment
- [ ] Installed the chart with `helm install` or `helm upgrade`

---

## Upgrading

To upgrade an existing deployment after changing values:

```bash
helm upgrade landskape . -f values.yaml -n <your-namespace>
```

---

## Uninstalling

To remove the deployment:

```bash
helm uninstall landskape -n <your-namespace>
```

> The PVC may be retained depending on your cluster's reclaim policy. Delete it manually if needed.
