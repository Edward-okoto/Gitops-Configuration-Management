## Gitops-Configuration-Management

### Advanced Configuration Management in ArgoCD

##### Introduction

This module provides an in-depth look at advanced configuration management in ArdoCD, utilizing tools like Helm and Kustomize, and delving into secrets management and customization of resource management and sync policies.

**Lesson 1 : Managing Configurations with Helm and Kustomize in ArgoCD**

**Integrating Helm with ArgoCD**

Create a **Helm chart** based on the structure you provided and explain each part in detail.  

##### **Helm Chart Structure**
Here’s the Helm chart directory layout:
```plaintext
my-app/
├── Chart.yaml         # Defines metadata about the Helm chart
├── values.yaml        # Stores customizable values for the application
├── templates/         # Holds Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
```

---

##### **Chart.yaml (Helm Chart Metadata)**

This file defines the Helm chart's **metadata**, including its name, version, and description.
```yaml
apiVersion: v2
name: my-app
description: A Helm chart for my application
version: 1.0.0
appVersion: "1.0"
```
✅ **`apiVersion`** → Defines the Helm version. (`v2` is used for modern Helm charts.)  
✅ **`name`** → Name of the Helm chart (`my-app`).  
✅ **`description`** → Brief description of the chart.  
✅ **`version`** → Helm chart version (used for tracking releases).  
✅ **`appVersion`** → Application version being deployed (could match your Docker image version).  

---

##### **values.yaml (Customizable Configuration)**
Defines configurable parameters for the application.
```yaml
replicaCount: 2
image:
  repository: my-app-image
  tag: latest
service:
  type: ClusterIP
  port: 8080
```
✅ **`replicaCount`** → Number of application instances (pods).  
✅ **`image.repository`** → Specifies the container image location.  
✅ **`image.tag`** → Defines the image version (`latest` is often used for development).  
✅ **`service.type`** → Defines how the service is exposed (`ClusterIP`, `NodePort`, `LoadBalancer`).  
✅ **`service.port`** → Port exposed by the application container.  

---

##### **templates/deployment.yaml (Application Deployment)**
Defines how the application is deployed using Kubernetes **Deployments**.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
```
✅ **`{{ .Chart.Name }}`** → Pulls the Helm chart name dynamically (`my-app`).  
✅ **`replicas`** → Uses `values.yaml` to control replica count.  
✅ **`image`** → Uses `values.yaml` for `repository` and `tag`.  
✅ **`containerPort`** → Configures the application port dynamically.  

---

##### **templates/service.yaml (Exposing the Application)**
Defines a Kubernetes **Service** to expose the application.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
  selector:
    app: {{ .Chart.Name }}
```
✅ **`type`** → Uses `values.yaml` to dynamically select the service type.  
✅ **`port`** → Uses `values.yaml` to set the exposed port.  

---

##### **templates/ingress.yaml (Ingress Routing)**
Optional, if you need a **public domain** or **external access**.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Chart.Name }}-ingress
spec:
  rules:
    - host: my-app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ .Chart.Name }}
                port:
                  number: {{ .Values.service.port }}
```
✅ **`host`** → Defines a custom domain (can be adjusted per environment).  
✅ **`service.port`** → Ensures it routes traffic to the correct service.  

---

##### **Initialize Git & Push to Repository**
```bash
git init
git add .
git commit -m "Added Helm chart for my-app"
git remote add origin https://github.com/Edward-okoto/gitops-Application-Deployment.git
git push origin main
```


**Utilize Kustomize in ArgoCD**

---
Creating a **Kustomize base and overlays** 

##### **Directory Structure**

```plaintext
my-app/
├── base/                   # Base Kubernetes resources
│   ├── kustomization.yaml  # Kustomize file defining base manifests
│   ├── deployment.yaml     # Generic deployment spec
│   ├── service.yaml        # Generic service spec
└── overlays/               # Environment-specific modifications
    ├── dev/                
    │   ├── kustomization.yaml  # Dev-specific customization
    │   └── patch.yaml          # Patch to modify base deployment
    ├── prod/                    
    │   ├── kustomization.yaml  # Prod-specific customization
    │   └── patch.yaml          # Patch to modify base deployment
    ├── staging/                 
        ├── kustomization.yaml  # Staging-specific customization
        └── patch.yaml          # Patch to modify base deployment
```

---

##### **Step 1: Create the `base/` Directory**
Define the core Kubernetes manifests that will be **shared across all environments**.

✅ **`base/deployment.yaml`**  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app-image:latest
          ports:
            - containerPort: 8080
```

✅ **`base/service.yaml`**  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 8080
```

✅ **`base/kustomization.yaml`**  
```yaml
resources:
  - deployment.yaml
  - service.yaml
```
This file tells Kustomize to **use** the `deployment.yaml` and `service.yaml` resources.

---

##### **Step 2: Create Environment Overlays**
Each environment (`dev`, `prod`) will **modify the base configurations**.

✅ **`overlays/dev/kustomization.yaml`**  
```yaml
bases:
  - ../../base
patchesStrategicMerge:
  - patch.yaml
```
This **references the base configuration** and applies a patch.

✅ **`overlays/dev/patch.yaml`**  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: my-app
          image: my-app-image:dev
```
This **reduces replicas** and **changes the container image** for Dev.

✅ **`overlays/prod/kustomization.yaml`**  
```yaml
bases:
  - ../../base
patchesStrategicMerge:
  - patch.yaml
```

✅ **`overlays/prod/patch.yaml`**  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: my-app
          image: my-app-image:prod
```
This **increases replicas** and **uses the production image**.

---

##### **Step 3: Apply Kustomize Configuration**
To deploy **Dev environment**, run:
```bash
kubectl apply -k overlays/dev
```
To deploy **Prod environment**, run:
```bash
kubectl apply -k overlays/prod
```

---

##### **Step 4: Push to Git & Integrate with Argo CD**
```bash
git init
git add .
git commit -m "Added Kustomize base and overlays"
git remote add origin https://github.com/Edward-okoto/gitops-Application-Deployment.git
git push origin main
```

To create an **Argo CD application** using Kustomize:
```bash
argocd app create my-app \
  --repo https://github.com/Edward-okoto/gitops-Application-Deployment.git \
  --path overlays/dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
```
Repeat for `overlays/prod`.

---

###### **1️⃣ Argo CD Application for Helm Chart**
Create a YAML file `argocd-my-app-helm.yaml` to tell Argo CD how to deploy your **Helm chart**.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-helm
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/Edward-okoto/gitops-Application-Deployment.git'
    path: my-app       # Points to Helm chart directory
    targetRevision: HEAD
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

✅ **Tells Argo CD to track the Helm chart in the Git repository.**  
✅ **Automates deployment by enabling self-healing and pruning outdated resources.**  
✅ **Uses the Helm `values.yaml` file for customization.**  

---

##### **2️⃣ Argo CD Application for Kustomize (Dev, Prod, Staging)**
Each Kustomize overlay needs its own Argo CD application. Below is an example for **Dev** (`argocd-my-app-dev.yaml`):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/Edward-okoto/gitops-Application-Deployment.git'
    path: my-app/overlays/dev  # Points to Kustomize overlay for Dev
    targetRevision: HEAD
    kustomize: {}  # Enables Kustomize processing
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

✅ **Uses Kustomize overlays to modify the base configuration for Dev.**  
✅ **Automates sync & drift correction.**  
✅ **Deploys to the `dev` namespace.**  

Repeat the same for **Prod and Staging** (`argocd-my-app-prod.yaml` and `argocd-my-app-staging.yaml`):

**`argocd-my-app-prod.yaml`**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/Edward-okoto/gitops-Application-Deployment.git'
    path: my-app/overlays/prod
    targetRevision: HEAD
    kustomize: {}
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**`argocd-my-app-staging.yaml`**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-staging
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/Edward-okoto/gitops-Application-Deployment.git'
    path: my-app/overlays/staging
    targetRevision: HEAD
    kustomize: {}
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: staging
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

##### **3️⃣ Apply Argo CD Applications**
Once defined, apply the applications to Argo CD:
```bash
kubectl apply -f argocd/argocd-my-app-helm.yaml -n argocd
kubectl apply -f argocd/argocd-my-app-dev.yaml -n argocd
kubectl apply -f argocd/argocd-my-app-prod.yaml -n argocd
kubectl apply -f argocd/argocd-my-app-staging.yaml -n argocd
```

---

##### **4️⃣ Sync and Verify**
Sync each application in Argo CD:
```bash
argocd app sync my-app-helm
argocd app sync my-app-dev
argocd app sync my-app-prod
argocd app sync my-app-staging
```
Verify deployment:
```bash
kubectl get pods -n default
kubectl get pods -n dev
kubectl get pods -n prod
kubectl get pods -n staging
```

---


### Lesson 2: **Securely Managing Secrets in Kubernetes & Argo CD**

Managing secrets in Kubernetes and Argo CD is **critical** to ensure sensitive credentials, API keys, and certificates remain **secure** while maintaining automation in GitOps workflows. Below is a **comprehensive guide** on securely handling secrets, including integration with **external secret managers** like HashiCorp Vault, AWS Secrets Manager, and Azure Key Vault.

---

##### **1️⃣ Why Secure Secret Management Is Important**
❌ **Hardcoding secrets in Kubernetes manifests** leads to **exposure risks**.  
❌ **Storing secrets in Git repositories** is unsafe—even in private repositories.  
✅ Best practice: **External Secret Managers + Kubernetes Secret Management Tools**  

---

##### **2️⃣ Built-In Secret Management in Kubernetes**
Kubernetes offers **Secrets** (`apiVersion: v1 kind: Secret`) to store sensitive data in base64-encoded form.

**Example: Standard Kubernetes Secret**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: default
type: Opaque
data:
  password: cGFzc3dvcmQ=  # base64-encoded "password"
```
  **Issues with Default Kubernetes Secrets**:
- Stored in **etcd**, meaning **anyone with cluster access** can read them.
- Not encrypted by default (**base64 is NOT encryption**).
- Requires additional tools for **better security**.

---

##### **3️⃣ Enhancing Security for Kubernetes Secrets**
##### **Best Practices**
✅ **Use Kubernetes Secrets with Encryption**  
✅ **Restrict access with RBAC** (`kubectl get secrets` should be **limited**).  
✅ **Use External Secret Managers for Secure Storage**  

---

##### **4️⃣ External Secret Managers & Kubernetes Integration**
Instead of storing secrets inside Kubernetes, **external secret managers** provide a secure vault.  

##### **Popular External Secret Managers**
✅ **HashiCorp Vault** – Secret lifecycle management with fine-grained access control.  
✅ **AWS Secrets Manager** – Secure storage of AWS credentials with automatic rotation.  
✅ **Azure Key Vault** – Managed encryption, access policies, and secret retrieval.  
✅ **Google Secret Manager** – Fully managed secret storage in GCP.  

---

##### **5️⃣ Using Argo CD with External Secret Managers**
##### **Challenge in GitOps:**  
Argo CD deploys Kubernetes resources from Git repositories, but **secrets should NOT be stored in Git**.

##### **Solution: Using External Secrets Operator**
Integrate **External Secrets Operator** with Argo CD to securely retrieve and inject secrets into Kubernetes.

---

##### **6️⃣ Example: Managing Secrets with AWS Secrets Manager + Argo CD**
Use the **External Secrets Operator** to pull secrets securely from **AWS Secrets Manager** into Kubernetes.

##### **1️⃣ Install External Secrets Operator**
```bash
kubectl apply -f https://github.com/external-secrets/external-secrets/releases/latest/download/crds.yaml
kubectl apply -f https://github.com/external-secrets/external-secrets/releases/latest/download/operator.yaml
```

##### **2️⃣ Define an ExternalSecret Object**
This fetches secrets from AWS Secrets Manager:
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-secret
  namespace: default
spec:
  refreshInterval: "1m"  # Automatically sync secrets every 1 minute
  secretStoreRef:
    name: aws-secrets-store
    kind: SecretStore
  target:
    name: aws-secret  # The name of the Kubernetes Secret to create
    creationPolicy: Owner
  data:
  - secretKey: password  # Key in the Kubernetes Secret
    remoteRef:
      key: my-app-password  # Key in AWS Secrets Manager
```
**Now, Argo CD can reference the Kubernetes Secret (`aws-secret`) without storing credentials in Git!**

---

##### **7️⃣ Securely Managing Helm & Kustomize Secrets in Argo CD**
##### **🔹 Helm-Based Secrets Management**
Helm allows secret injection using **Helm Secrets Plugin** (`helm-secrets`), which integrates with HashiCorp Vault or AWS Secrets Manager.

##### **Example: Securely Managing Helm Secrets**
```bash
helm secrets install my-app ./my-app
```

##### **🔹 Kustomize-Based Secrets Management**
Use **SOPS** (Mozilla's Secrets OPerationS) with Kustomize to manage secrets **encrypted in Git**.

---

##### **Final Takeaways**
✅ **Never store raw secrets in Git repositories.**  
✅ **Use external secret managers to securely retrieve secrets.**  
✅ **Integrate Argo CD with HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault.**  
✅ **Use Kubernetes External Secrets Operator for automated retrieval & rotation.**  
✅ **Leverage encryption tools (Helm Secrets, SOPS) for GitOps security.**  


### Lesson 3: **Customizing Resource Management and Sync Policies in ArgoCD**

Argo CD offers powerful **resource management and sync policies**, allowing fine-grained control over how applications are deployed, updated, and maintained in Kubernetes. Let’s explore how to customize these settings to optimize deployment behavior.

---

#### **1️⃣ Customizing Resource Management in ArgoCD**
Argo CD allows customization of resource handling for different **applications** and **environments**.

##### **🔹 Resource Pruning**
By default, Argo CD **does not delete unmanaged resources** after an application update. To enable **automatic cleanup**, set:
```yaml
spec:
  syncPolicy:
    automated:
      prune: true  # Deletes outdated Kubernetes resources
```
✅ Ensures **obsolete resources** do not accumulate after a sync.  
✅ Prevents **drift** between Git and the actual cluster state.  

##### **🔹 Resource Exclusion (Ignore Certain Kubernetes Objects)**
If certain Kubernetes objects should be **ignored** (e.g., ServiceAccounts, ConfigMaps):
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
spec:
  clusterResourceBlacklist:
    - group: ""
      kind: ConfigMap
    - group: rbac.authorization.k8s.io
      kind: RoleBinding
```
✅ Ensures that **specific resources remain untouched**.  
✅ Useful for **RBAC policies** or **external dependencies** that Argo CD should not modify.  

---

#### **2️⃣ Customizing Sync Policies**
Sync policies define **how and when Argo CD applies updates** to applications.

##### **🔹 Automated Sync Policy**
Argo CD can be configured to **automatically sync** applications whenever Git changes:
```yaml
spec:
  syncPolicy:
    automated:
      selfHeal: true   # Fix drifted resources automatically
      prune: true      # Removes outdated resources
```
✅ **Self-Healing:** Detects and **corrects** drifted resources (manual changes that don't match Git).  
✅ **Pruning:** Deletes **obsolete** Kubernetes objects during sync.  

---

##### **🔹 Manual Sync Policy**
If manual intervention is required before applying changes:
```yaml
spec:
  syncPolicy:
    automated: {}  # Disables automated sync (user must manually trigger)
```
✅ **Best for controlled environments** where **manual approvals** are required.  
✅ Ensures **changes do not deploy automatically** before verification.  

---

##### **🔹 Sync Windows (Restrict Deployment Timing)**
Control **when applications sync** using predefined **time windows**.
```yaml
spec:
  syncWindows:
    - kind: allow
      schedule: "0 3 * * *"  # Only sync at 3 AM daily
      duration: 2h
      applications:
        - my-app
```
✅ Ensures sync happens **only at specific times** (useful for regulated deployments).  
✅ Prevents updates **during critical operation periods**.  

---

#### **3️⃣ Sync Hooks for Advanced Customization**
Argo CD supports **sync hooks**, allowing custom actions **before, during, or after deployments**.

##### **🔹 PreSync Hook (Run Tasks Before Deployment)**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
spec:
  template:
    spec:
      containers:
        - name: migrate
          image: my-db-migration:latest
```
✅ Runs **database migrations** before deploying a new version.  
✅ Ensures **dependencies** are handled before the main sync starts.  

##### **🔹 PostSync Hook (Run Tasks After Deployment)**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: cleanup-temp-files
  annotations:
    argocd.argoproj.io/hook: PostSync
spec:
  template:
    spec:
      containers:
        - name: cleanup
          image: my-cleanup-image:latest
```
✅ Performs **cleanup tasks** after successful deployment.  
✅ Useful for **log cleanup, reporting, or monitoring adjustments**.  

---

##### **Wrapping Up**
✅ **Prune outdated resources** to maintain a clean environment.  
✅ **Exclude specific resources** from Argo CD’s control when needed.  
✅ **Use self-healing sync policies** to automatically fix drift.  
✅ **Set manual sync policies** when approval is needed.  
✅ **Schedule sync windows** to restrict deployment timing.  
✅ **Utilize sync hooks** for pre-sync and post-sync automation.  


