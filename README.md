# Installing ArgoCD

Sources on K8s 1.26.1:

Installing Helm: https://helm.sh/docs/intro/install/ <br>
Installing ArgoCD: https://artifacthub.io/packages/helm/argo/argo-cd/3.26.1?modal=install

```bash
sudo yum install -y git
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install <NAME> argo/argo-cd --version 3.26.1
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
