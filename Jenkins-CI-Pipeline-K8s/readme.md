# Setting up a Jenkins pipeline on top of Kubernetes

1. Ensure your local helm repo cache is up to date: `helm repo update`
1. Deploy Jenkins on top of your K8s cluster using the values.yaml file found under this post `./code/values.yaml`.
`helm install --name k8sjenkins -f values.yaml stable/jenkins --version 0.9.0`

1. Login to the cluster from the output of the helm chart deployment.

```shell
printf $(kubectl get secret --namespace default k8sjenkins-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);ech
o

```