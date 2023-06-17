# DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM

- To simplify the process of deploying JFrog Artifactory into Kubernetes, the recommended approach is to utilize Helm, a package manager for Kubernetes. You can begin by searching for the official Helm chart for Artifactory on Artifact Hub

![saa](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/197d973b-2d30-4610-88a6-45b6dc347a0a)

- Click on the install menu on the right to see the installation commands.

- Add the jfrog remote repository on your laptop/computer . `helm repo add jfrog https://charts.jfrog.io` .Create a namespace called tools where all the tools for DevOps will be deployed. `kubectl create ns tools`

- Update the helm repo index on your laptop/computer `helm repo update`

- Install artifactory `helm install my-artifactory jfrog/artifactory --version 107.33.12 -n tools`

![cu1](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/7d0efc97-c7ea-483f-834f-e2ddff76cc48)

![cu2](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/b92d1bc9-f5ea-43f2-845b-09044f452917)


- After a period of time, when the Artifactory Helm chart is deployed, you will notice the presence of several pods. The pods will consist of the Artifactory software itself, a PostgreSQL database, and an Nginx proxy. The Nginx proxy is responsible for configuring routes to the various capabilities offered by Artifactory. Here's an example of what you might observe when inspecting the pods:

![daa](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/24e12d2e-f604-43b8-82e1-28609da5b72b)

- Each of the deployed application have their respective services. This is how you will be able to reach either of them.

![aqa](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/099b058a-d15f-4ed5-96fe-34e60cb1b051)

- It's important to note that the Nginx Proxy within the Artifactory deployment is configured with a service type of LoadBalancer. This means that in order to access Artifactory, you'll need to go through the service associated with the Nginx proxy, which is created as a load balancer in your cloud provider.

- Copy the URL and paste in the browser.

![cu3](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/fe558d25-f714-4361-bdc5-90182fa5d3a1)

- The default username is admin

- The default password is password

![cu4](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/79f90dd1-d0ce-411f-ad9e-a3fddbf86166)


- Setting the service type to LoadBalancer initially is a convenient method for exposing applications externally in Kubernetes. However, provisioning individual load balancers for each application can become costly and complex, especially as the number of deployed applications increases.

- To address this, a recommended approach is to utilize Kubernetes Ingress along with an Ingress Controller. By deploying an Ingress Controller, you can leverage its capabilities to manage routing and load balancing for multiple applications.

- One significant advantage of using an Ingress Controller is the ability to use a single load balancer for multiple applications. This means that both Artifactory and other tools can utilize the same load balancer, reducing cloud costs and the burden of managing numerous load balancers.

- An Ingress is an API object in Kubernetes that facilitates external access to services within the cluster. It acts as a gateway, providing capabilities such as load balancing, SSL termination, and name-based virtual hosting. With Ingress, you can define HTTP and HTTPS routes from outside the cluster to direct traffic to specific services within the cluster. The routing of traffic is controlled by the rules specified in the Ingress resource. In essence, Ingress enables the management of external access and traffic routing for services in a Kubernetes cluster.

Deploying the YAML configuration for an Ingress resource alone will not be sufficient for it to function correctly. The presence of an Ingress controller is essential for the Ingress resource to work as expected.

## Ingress controller
    
   -  Unlike other controllers, such as the Node Controller, Replica Controller, Deployment Controller, Job Controller, or Cloud Controller, which are    automatically started with the cluster as part of the kube-controller-manager, Ingress controllers do not start automatically. 

  - Therefore, to enable the functionality of the Ingress resource, you need to separately deploy and configure an Ingress controller in the cluster. The Ingress    controller is responsible for interpreting and implementing the rules defined in the Ingress resource, handling traffic routing, load balancing, and other        related tasks.

## Deploy Nginx Ingress Controller
   
   - In this project, we will be deploying and utilizing the Nginx Ingress Controller, which is often the default choice for Kubernetes projects. It is a reliable     and user-friendly option.As the Nginx Ingress Controller is maintained by Kubernetes, it is recommended to follow the official installation guide provided by     Kubernetes for this purpose. While there may be ready-to-use charts available on artifacthub.io, in this scenario, it is advisable to rely on the official         guide for consistency and reliability.

  - Using the Helm approach, according to the official guide;
     - Install Nginx Ingress Controller in the ingress-nginx namespace:
       
       ```
         helm upgrade --install ingress-nginx ingress-nginx \
         --repo https://kubernetes.github.io/ingress-nginx \
         --namespace ingress-nginx --create-namespace 
         
       ```
         
      - A few pods should start in the ingress-nginx namespace: `kubectl get pods --namespace=ingress-nginx`
      
      - After a while, they should all be running. The following command will wait for the ingress controller pod to be up, running, and ready:

       ```
       kubectl wait --namespace ingress-nginx \
         --for=condition=ready pod \
         --selector=app.kubernetes.io/component=controller \
         --timeout=120s
      ```
    
    - Check to see the created load balancer in AWS: `kubectl get service -n ingress-nginx`
    
    - The ingress-nginx-controller service that was created is of the type LoadBalancer. That will be the load balancer to be used by all applications which             require external access, and is using this ingress controller.
    
    - Check the IngressClass that identifies this ingress controller. `kubectl get ingressclass -n ingress-nginx`
      
![sa1](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/03782e67-d20f-43e2-b33c-73753bb26292)

![cu6](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/1c8c7628-556f-4e21-9f50-8c8da7821658)

   ## Deploy Artifactory Ingress
   
   - Now, it is time to configure the ingress so that we can route traffic to the Artifactory internal service, through the ingress controller’s load balancer.
```
 apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: artifactory
spec:
  ingressClassName: nginx
  rules:
  - host: "tooling.artifactory.sandbox.svc.gbaderobusola.click"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-artifactory
            port:
              number: 8082 
 ```

![cu9](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/2fdbf4c2-3164-43a9-b509-77e9061cd9fb)


  - The spec section,the configuration selects the ingress controller using the ingressClassName

  - Run the command `kubectl apply -f p.25.yaml -n tools`
  
  ![cu8](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/9d48c02d-a950-430a-84d6-cf125615cd7a)

## Configure DNS

- If anyone were to visit the tool, it would be very inconvenient sharing the long load balancer address. Ideally, you would create a DNS record that is human readable and can direct request to the balancer. This is exactly what has been configured in the ingress object - host: "tooling.artifactory.sandbox.svc.gbaderobusola.click" but without a DNS record, there is no way that host address can reach the load balancer.The sandbox.svc.gbaderobusola.click part of the domain is the configured HOSTED ZONE in AWS. So you will need to configure Hosted Zone in AWS console or as part of your infrastructure as code using terraform.

  ### Create Route53 record
     
     - (AWS Alias) In the create record section, type in the record name, and toggle the alias button to enable an alias. An alias is of the A DNS record type            which basically routes directly to the load balancer. In the choose endpoint bar, select Alias to Application and Classic Load Balancer.
     - Select the region and the load balancer required. You will not need to type in the load balancer, as it will already populate.

![cu11](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/80ec42f7-7c8d-4ff0-8af4-4bb79723a1a8)


 ## Visiting the application from the browser
 
   - So far, we now have an application running in Kubernetes that is also accessible externally. That means if you navigate to https://tooling.artifactory.sandbox.svc.gbaderobusola.click/ , it should load up the artifactory application.Using Chrome browser will show something like the below. It shows that the site is indeed reachable, but insecure. This is because Chrome browsers do not load insecure sites by default. It is insecure because it either does not have a trusted TLS/SSL certificate, or it doesn’t have any at all.

![cu12](https://github.com/busolagbadero/DEPLOYING-AND-PACKAGING-APPLICATIONS-INTO-KUBERNETES-WITH-HELM/assets/94229949/2547490d-1fdd-49de-bf28-ea4356621e86)


   


       
