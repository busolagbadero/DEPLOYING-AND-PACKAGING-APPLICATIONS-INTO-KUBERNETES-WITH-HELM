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
