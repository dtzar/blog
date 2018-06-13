# Setting up a Jenkins pipeline on top of Kubernetes

Prerequisites:

- An existing 1.9+ kubernetes cluster deployed with kubectl access.  This could be [ACS-Engine](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/deploy.md) or [AKS](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster).
- Helm 2.8+ installed on the client and matching version on the K8s cluster

This is the workflow to follow:

1. Install Jenkins on Kubernetes
1. Configure Jenkins to utilize your Jenkinsfile pipeline from a GitHub repo
1. Execute the pipeline

## Install Jenkins on Kubernetes

1. Ensure your local helm repo cache is up to date: `helm repo update`

1. Deploy Jenkins on top of your K8s cluster using the values.yaml file found under this post `./code/values.yaml`.

    `helm install --name k8sjenkins -f ./code/values.yaml stable/jenkins --version 0.16.1`

    This configuration will create a new service of type Loadbalancer which will expose Jenkins directly with a public IP address.

1. Login to the cluster from the output of the helm chart deployment.

    ```shell
    printf $(kubectl get secret --namespace default k8sjenkins-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);ech
    o
    export SERVICE_IP=$(kubectl get svc --namespace default k8sjenkins-jenkins --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
    echo http://$SERVICE_IP:8080/login
    ```

    Now visit the URL above and login to Jenkins.  
    The username from the template is `jenkinsadmin`.

## Configure Jenkins to utilize your Jenkinsfile pipeline from a GitHub repo
