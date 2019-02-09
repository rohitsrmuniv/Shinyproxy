# Containerized ShinyProxy in a Kubernetes cluster

In this example, ShinyProxy will run inside a Kubernetes cluster. Shiny containers will also be spawned
in the same cluster. To make the application accessible outside the cluster, a NodePort service is created.

## How to run

1. Build the docker image for kube-proxy-sidecar first and push it to container registry:

	`sudo docker build -t ${registry_name}.azurecr.io/kube-proxy-sidecar:0.1.0 kube-proxy-sidecar/`
	
	`sudo docker push ${registry_name}.azurecr.io/kube-proxy-sidecar:0.1.0`

2. Create a new namespace as "shiny" in your kubernetes cluster. We will do deployments, service and authorization in the same namespace.

	`kubectl create ns shiny`

3. Go to folder shinyproxy-application and replace ${SecretForDockerRegistry} and ${registry_name} in application.yml with your docker container registry secrets and it's name. On top I created Tom and Jon login users as admin and Guest user as non-admin user
	
	`sudo docker build -t ${registry_name}.azurecr.io/shinyproxy-application:latest shinyproxy-application/`
	`sudo docker push ${registry_name}.azurecr.io/shinyproxy-application:latest`

4. Run the below command to do a deployment which will create the pod having two containers 'shinyproxy-application' and 'kube-proxy-sidecar' in the namespace shiny:

	`kubectl create -f sp-deployment.yaml`

5. Run the below command to grant full privileges to the 'default' service account which runs the above pod in namespace shiny:

	`kubectl create -f sp-authorization.yaml`

6. Run the below command to expose the shinyproxy service as LoadBalancer which will assign a public ip and make the service available on port 80. If you don't want public ip for the service you can always use type as NodePort with port assign at nodePort (https://github.com/openanalytics/shinyproxy-config-examples/blob/master/03-containerized-kubernetes/sp-service.yaml) :
	
	`kubectl create -f sp-service.yaml`

7. Once the service and pod is up and running, you can click on External Endpoints or use kube port-forward pod command