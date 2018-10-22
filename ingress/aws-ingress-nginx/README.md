### AWS EKS ingress-nginx deploy and configure:
More information about the ingress [here](https://kubernetes.io/docs/concepts/services-networking/ingress/)

Prerequisites:
  - You have a working cluster on AWS. If you dont please chech [here](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)
  - kubectl is configured to talk to your cluster. If no chech [here](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html)
  - Please note that only the user that was used to create the cluster have permission to create or modify Amazon EKS resources. In order to give some other user credits to do so, edit the ConfigMap as the cluster creator's user and add that user to the ConfigMap. More info [here](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)

1. Create the `nginx-ingress` namespace:
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/nginx-ingress-namespace.yaml```
2. Grant access to the ingress controller using RBAC:
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/nginx-ingress-controller-rbac.yaml```
3. Create Deployment and Service for the Nginx default backend:
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/nginx-ingress-default-backend.yaml```
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/nginx-ingress-default-backend-service.yaml```
  - If you want to create a custom backend container check [here](https://github.com/kubernetes/ingress-nginx/tree/master/images/404-server)
4. Create Deployment and Service for the Nginx controller:
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/nginx-ingress-controller.yaml```
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/nginx-ingress-controller-service.yaml```
4.  Create Deployment and Service for echoserver app to test:
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/echoserver/echoserver-namespace.yaml```
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/echoserver/echoserver-deployment.yaml```
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/echoserver/echoserver-service.yaml```
5. Create your own signed TLS Secret - this for the test only
  - ```openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=nginxsvc/O=nginxsvc"```
  - ```kubectl create secret tls tls-secret --key tls.key --cert tls.crt```
6. Obtain the `Controller/LoadBalancer` ExternalIP:
  - ```kubectl get svc --all-namespaces```
  - Or open the `Load Balancers` manager in AWS's UI
7. Create a CNAME record in Route53, so your domain to point to the ELB's DNS:
  - TODO: add a picture in the AWS console
8. Create the Ingress resource:
  - ```kubectl apply -f https://raw.githubusercontent.com/Alexsa6ko94/kubernetes-abk8s/master/ingress/aws-ingress-nginx/echoserver/echoserver-ingress-rule.yaml```
9. Send a request to the LoadBalancer to check if it is working: 
  - ```curl -ikL <LoadBalancer IP>```
	
### Debugging the nginx controller itself:

- Inspecting nginx.conf in the nginx-ingress-controller:
  - Find the pod name:
    - ```kubectl get po --all-namespaces```
  - Execute the command against that pod:
    - ```kubectl exec -it <pod_name> -- cat /etc/nginx/nginx.conf```
  - Chech the proxy_pass redirections:
    - ```kubectl exec -it <pod_name> -- cat /etc/nginx/nginx.conf | grep proxy_pass```
      - `Result: proxy_pass <upstream>;`
    - ```kubectl exec -it <pod_name> -- cat /etc/nginx/nginx.conf | grep upstream <upstream>```
  - Chech if the upstream definition is pointing to the right servers:
    - ```kubectl describe svc/<service_name>```
    - See the field `Endpoints` and compare them with the server entries int the <upstream> definition
