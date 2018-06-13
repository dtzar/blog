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

1. (Optional) Assign a public DNS name for the Jenkins service

    This will enable jenkins to have a DNS name configuration required for the proper operation of many Jenkins features like email notifications, PR status update, and environment variables such as BUILD_URL.

    In order to accomplish this for running k8s on Azure, modify the `./code/values.yaml` file from this repo as follows:

    - Uncomment line 44 and place the contents inside the brackets of line 43.  It should look no something like this: `ServiceAnnotations: {service.beta.kubernetes.io/azure-dns-label-name: "jenkinsk8s4azureyo"}`
    - change the `jenkinsk8s4azureyo` to be a globally unique value per datacenter region
    - Uncomment line 46 and replace the base name with whatever the value was changed to above.  Replace the datacenter region `eastus` in the example with whatever the region of where the Azure kubernetes cluster is deployed. i.e. `westus`

1. Deploy Jenkins on top of your K8s cluster

    Specify the modified values.yaml file after `-f` in the below command from the previous step OR just use the default from this repo if skipping the above step.

    `helm install --name k8sjenkins -f https://raw.githubusercontent.com/dtzar/blog/master/Jenkins-CI-Pipeline-K8s/code/values.yaml stable/jenkins --version 0.16.1`

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
