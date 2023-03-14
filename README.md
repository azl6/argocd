# Installing ArgoCD

Sources on K8s 1.26.1:

Installing Helm: https://helm.sh/docs/intro/install/ <br>
Installing ArgoCD: https://artifacthub.io/packages/helm/argo/argo-cd/3.26.1?modal=install

```bash
sudo yum install -y git
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install <NAME> argo/argo-cd --version 3.26.1 --create-namespace --namespace argocd
```

# Getting the initial password

```bash
k get secrets -n argocd
NAME                          TYPE                 DATA   AGE
argocd-initial-admin-secret   Opaque               1      10m # This is the secret containing the password!
# ...

k get secret argocd-initial-admin-secret -n argocd -o yaml
apiVersion: v1
data:
  password: Qk9lektCMVEzT0JlYlNKaw== # Password encoded in Base64. Please decode this value to get the password.
kind: Secret
 #...
 
echo "Qk9lektCMVEzT0JlYlNKaw==" | base64 --decode; echo
BOezKB1Q3OBebSJk # Password decoded! Use this to login in ArgoCD.
```

# Accessing the ArgoCD interface

The method I found was to change the `svc/argo-argocd-server` to NodePort, and add the **nodePort: 31111 to HTTP** and **nodePort: 32222** to HTTPS with `kubectl edit`. 

Then, we can access the IP of any node on ports 31111 and 32222 and we'll see the ArgoCD login-page. 

**Login:** admin <br>
**Password** Please use the password generated in the previous step to login.

![image](https://user-images.githubusercontent.com/80921933/224505122-d7e65162-cbe8-4ed3-a307-a30f62e12e14.png)


# Logging in ArgoCD via CLI

First, we need to download the Argocd CLI. Please, use the following link for reference: https://argo-cd.readthedocs.io/en/stable/cli_installation/

After that, we can login:

```bash
argocd login <SERVER>
```
Note: \<SERVER> is \<IP>:\<PORT>

# What is an ArgoCD application

An application in ArgoCD defines a **source** (VCS and manifests) and **destination** (cluster and namespace) to deploy Kubernetes resources

![image](https://user-images.githubusercontent.com/80921933/221785258-f443d948-b130-4baa-834a-57c2d140a522.png)

# What is an ArgoCD project

A project is a grouping of applications. We can define constraints for a project, such as "which clusters can we deploy to" and "which resources can we deploy"

![image](https://user-images.githubusercontent.com/80921933/221785717-03cd80b1-45e5-4096-9302-41faaac55f08.png)

# ArgoCD components

![image](https://user-images.githubusercontent.com/80921933/221786529-04daea41-c0f9-4db2-8001-f93dc668400a.png)

![image](https://user-images.githubusercontent.com/80921933/221786683-9f979e9e-460f-4d2f-874e-6466f6553d7a.png)

![image](https://user-images.githubusercontent.com/80921933/221786837-dc7606d6-43b4-4616-8a22-e7e9e015d9f8.png)

![image](https://user-images.githubusercontent.com/80921933/221787115-727b51d8-8229-47bc-af0b-d8a688f26989.png)

![image](https://user-images.githubusercontent.com/80921933/221787283-412d2671-a9b4-459a-b20c-189800b42924.png)

# Creating an Application via CLI

![image](https://user-images.githubusercontent.com/80921933/224506624-3a9ce61c-be0f-489e-b03f-840d48662fcb.png)

# Creating an Application via YAML file

We can define ArgoCD Applications via YAML files. They represent a specification of the **source** (where the manifests are stores) and **destination** (in which cluster/namespace the manifests will be applied to)

**application.yaml**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: staticsite
  namespace: argocd # App needs to be deployed in the argocd namespace
spec: 
  destination: 
    namespace: staticsite # Namespace for the app
    server: "https://kubernetes.default.svc" # Cluster the app will be deployed on
  project: default 
  source: 
    path: ./ # Path in repository where the manifests for app are stored, in this case, root
    repoURL: "https://github.com/azl6/manifests-for-argocd" # Repository with manifests
    targetRevision: main # Branch
  syncPolicy:
    syncOptions:
      - CreateNamespace=true # Will create namespace staticsite if it doesn't exist
```

**Important info:** The `application.spec.destination.server` field can be retrieved by the command `argocd cluster list`. In general, https://kubernetes.default.svc will represent the cluster where ArgoCD is installed.

After running `kubectl apply -f application.yaml -n argocd`, we need to synchronize the cluster and the repository manually.

The ArgoCD UI will show our resources. We just need to click in "Synchronize" so ArgoCD will deploy the resources to our K8s cluster:

![image](https://user-images.githubusercontent.com/80921933/224509717-e50a906f-a3e1-4060-a403-d3bbb9663df1.png)

# Detection of resources in sub directories

Suppose we have defined a **source** for ArgoCD in a Git repository, however, it has subdirectores with other resources, and we also want these resources to be recognized by ArgoCD.

We can use the `recurse: true` option of **application.spec.source.directory**:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: recursivechange
  namespace: argocd
spec:
  project: default
  source:
    directory:
      recurse: true # Configuration to discover resources in sub-directories
    repoURL: https://github.com/azl6/argocd-example-apps
    path: guestbook-with-sub-directories
    targetRevision: master
  destination:
    namespace: recursive
    server: https://kubernetes.default.svc
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```

# Creating an AppProject via YAML

When we have a project, we can launch a bunch of **Application** resources inside of it.

They also can be used to define constraints of what can be used as source/destination, which resources we can deploy, and which namespaces we can deploy on.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: demo-project
  namespace: argocd
spec:
  description: "Demo Project"
  sourceRepos: ################## Defines accepted sources
    - '*'
  destinations:
    - namespace: '*' ############ Defines accepted namespaces
      server: '*' ############### Defines accepted clusters
  clusterResourceWhitelist:
    - group: '*' ################ Defines...
      kind: '*' ################# Defines what kind of resources we can deploy
  namespaceResourceWhitelist:
    - group: '*' ################ Defines...
      kind: '*' ################# Defines...
```

After creating the project, we can reference it in the next applications that we create, with the **application.spec.project** key.

# Project Roles and JWT tokens

**Project Roles** can be created inside the **AppProject** resource to provide JWT tokens with defined permissions.

Example of an **appproject.spec.roles**

![image](https://user-images.githubusercontent.com/80921933/224521198-9a8bed86-4a0e-4816-a707-2fa2a62cc684.png)

To generate a token of this role, we use:

```bash
argocd proj role create-token PROJECT ROLE-NAME
```

To use the token in CLI, we pass the `--auth-token` flag

```bash
argocd cluster list --auth-token TOKEN
```

This is useful, for example, we we want to allow only the **sync**, but not the delete, get, etc... That way, when someone retrieves a token, it won't have access to the **delete** API call using that token.

# Using a private git repository as the source for manifests

Watch the Section 6 of the course.

We can use **SSH** or **Generated Key** to confirm that we can access that repository.

# Automated Sync

We can define an ArgoCD application with automated sync enabled.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: staticsite
  namespace: argocd
spec:
  destination:
    namespace: staticsite
    server: "https://kubernetes.default.svc"
  project: automated-sync
  source:
    path: app2
    repoURL: "https://github.com/azl6/manifests-for-argocd"
    targetRevision: main
  syncPolicy:
    automated: {} # Declaring Automated Sync. ArgoCD will compare source with destination every 3 minutes by default, and update cluster if needed.
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true
```

# Automated Pruning

By default, ArgoCD will not delete resources that got deleted in the Github repository (**Pruning is not enabled by default!**)

To enable this behaviour, we can set the flag `prune` to `true` in **application.spec.syncPolicy.automated.prune**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: staticsite
  namespace: argocd
spec:
  destination:
    namespace: staticsite
    server: "https://kubernetes.default.svc"
  project: automated-sync
  source:
    path: app2
    repoURL: "https://github.com/azl6/manifests-for-argocd"
    targetRevision: main
  syncPolicy:
    automated: 
      prune: true # Enabling prune when manifests get deleted from Github
```

We can also enable Prune in the web-UI when doing manual sync

![image](https://user-images.githubusercontent.com/80921933/225110824-8de676b5-4d12-466e-80ca-be018420e670.png)

# Automated Self Healing

By default, ArgoCD will **not** trigger sync when the cluster current state deviates from desired state (e.g by a manual change)

To correct the cluster state when a manual change happens, we can enable **automated self-healing**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: staticsite
  namespace: argocd
spec:
  destination:
    namespace: staticsite
    server: "https://kubernetes.default.svc"
  project: automated-sync
  source:
    path: app2
    repoURL: "https://github.com/azl6/manifests-for-argocd"
    targetRevision: main
  syncPolicy:
    automated: 
      selfHeal: true # Cluster will self-heal when a manual change happens to the application
```

