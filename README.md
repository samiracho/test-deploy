This repository has the configuration files needed to build, test and deploy the project https://github.com/guipal/apisamplejava.git
The following parameters can be set in the Jenkins job:
- repository: project repository url
- branch:  project branch
- image_tag: Docker image tag
- profile: environment where the Docker image will be deployed
- action: Build & Deploy / Only build / Only deploy

Since the CI process is under Jenkins, it should be straight forward to automate this process even more and trigger it e.g when there is changes in a branch, with a commit message, peroiodically e.t.c 

This project contains the following files:
# Jenkinsfile
+ Groovy script that is executed by Jenkins and does the following actions.
    + Test code.
    + Build the project. 
    + Build the Docker image. 
    + Test the Docker Image.
    + Push the Docker Image to the registry.
    + Create the kubernetes namespace using the *profile* name
    + Deploy in Kubernetes.

# Dockerfile
+ Simple Dockerfile that creates an image with the project and starts it.

# goss.yaml & goss_way.yaml
+ Configuration files to run the tests over the Docker image before pushing it to the registry.

# K8sfile
+ Kubernetes yaml template file to deploy the project in Kubernetes.