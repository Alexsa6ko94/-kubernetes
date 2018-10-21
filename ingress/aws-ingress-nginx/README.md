### AWS ingress-ngingx deploy and configure:

Prerequisites:
  - You have a working cluster on AWS. If you dont please chech here /https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html/
  - kubect is configured to talk to your cluster. If no chech here /https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html/
  - Please note that only the user that was used to create the cluster have permission to create or modify Amazon EKS resources. In order to give some other user credits to do so, edit the ConfigMap as the cluster creator's user and add that user to the ConfigMap. More info here /https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html/

1. Create the desired namespace that will be used for the ingress:
  - nginx-ingress-namespace.yaml
  - TODO: `kubectl apply -f https://raw.gitcontent.com/Alexsa6ko94/kuberentes-abk8s/ingress/nginx-ingress-namespace.yaml`
2. Create Deployment and Service for the Nginx default backend:
  - nginx-ingress-default-backend.yaml
  - nginx-ingress-default-backend-service.yaml
3. Create Deployment and Service for the Nginx controller:
  - nginx-ingress-controller.yaml
  - nginx-ingress-controller-service.yaml
4.  Create Deployment and Service for some app to test:
  - testapp-deployment.yaml
  - testapp-service.yaml
5. Create your own signed TLS Secret - this for the test only
6. Create the Ingress resource:
  - testapp-ingress-rules.yaml
7. Obtain the controller/LoadBalancer ExternalIP:
  - kubectl get svc --all-namespaces
8. Send a request to the LoadBalancer to check if it is working: 
  - curl -ikL <LoadBalancer IP>
	
### Debugging the nginx controller notes:

1. Inspecting nginx.conf in the nginx-ingress-controller:
  1.1. Find the pod name:
	- kubectl get po --all-namespaces
  1.2. Execute the command against that pod:
	- kubectl exec -it <pod_name> -- cat /etc/nginx/nginx.conf 
  1.3. Chech the proxy_pass redirections:
	- kubectl exec -it <pod_name> -- cat /etc/nginx/nginx.conf | grep proxy_pass
		- Result: proxy_pass <upstream>;
	- kubectl exec -it <pod_name> -- cat /etc/nginx/nginx.conf | grep upstream <upstream>
  1.4. Chech if the upstream definition is pointing to the right servers:
	- kubectl describe svc/<service_name>
	- See the field 'Endpoints' and compare them with the server entries int the <upstream> definition