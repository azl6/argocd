# What is an ArgoCD application

An application in ArgoCD defines a **source** (VCS and manifests) and **destination** (cluster and namespace) to deploy Kubernetes resources

![image](https://user-images.githubusercontent.com/80921933/221785258-f443d948-b130-4baa-834a-57c2d140a522.png)

# What is an ArgoCD project

A project is a grouping of applications. We can define constraints for a project, such as "which clusters can we deploy to" and "which resources can we deploy"

![image](https://user-images.githubusercontent.com/80921933/221785717-03cd80b1-45e5-4096-9302-41faaac55f08.png)
