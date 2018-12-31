# Implementing NGINX from NGINX as an Ingress Controller in VMware Cloud PKS
An ingress exposes routes from outside the cluster to services inside the cluster. 
The procedure in this document describes how to install the NGINX ingress controller that is maintained by NGINX. 

**[[And another paragraph that briefly describes the "Why?" - typical use cases for this product.]]**

*If you want to use the NGINX community version of the ingress controller deployed for high availability in multiple availability zones on a production Smart Cluster, click here.* [[[ZZZZZZZZZZZZ xref here]]]


### Versions
This implementation uses the following versions:  

| Software    | Version  |           | notes |
| :------     | ---:     |     ---:  | :---  |
| NGINX       | 1.0.2    | 0.23.0    |       |
| Kubernetes* | 1.11.5-1 | 1.10.11-1 | running in a Smart Cluster |
| Helm        | 2.8.0    | 2.8.0     |       |


### Before You Begin
This procedure requires that you have a VMware Cloud Services organization with access to the VMware Cloud PKS service.

The procedure outlined in this document assumes that you have the following tools installed locally.

- kubectl - the Kubernetes command line interface
- Helm - the Kubernetes package manager
- Git

And that you have done the following.

1. Create a Smart Cluster.
2. Connect to the cluster with kubectl.
3. Create a local work directory into which you will clone the project and update its configuration.

For more information on these steps, see the [VMware Cloud PKS documentation](https://docs.vmware.com/en/VMware-Kubernetes-Engine/index.html).

<br>

---
## Steps
This procedure shows how to create a NGINX ingress controller deployment on your Smart Cluster using a Helm chart. 

When you install the controller, you must modify the default settings. At a minimum, you must update ```config.entries``` and probably specify more than one replica of the controller.

Use the following steps to download the project, modify the ```values.yaml``` file, and deploy the controller.

**Note:** *If you want to use the NGINX community version of the ingress controller deployed for high availability in multiple availability zones on a production Smart Cluster, click here.* [[[ZZZZZZZZZZZZ xref here]]]

1. Clone the project into your local work directory.
   ``` 
   git clone https://github.com/nginxinc/kubernetes-ingress
   ```  
   This creates a directory called ```kubernetes-ingress``` that contains the project.

2. Change to the helm-chart directory.
   ```
   cd kubernetes-ingress/deployments/helm-chart
   ```
4. Open the ```values.yaml``` file in a text editor.

3. Locate the ```config.entries``` section. It looks something like this.
```
. . . 
  config:
    entries: {}
. . . 
```

4. Replace the empty brackets with the following entries.
```
. . . 
  config:
    entries:
      server-names-hash-bucket-size: "256"
      server-names-hash-max-size: "2048"
. . . 
```

5. Locate ```replicaCount``` and update its value to ```3```.
```
. . . 
  replicaCount: 3
. . . 
```

Typically you want more than one replicas of the controller.

6. Save and close the ```values.yaml``` file.

7. Install the Helm chart.  
   Use the following command to create a deployment of the ingress controller, replacing ```mynginx-controller-release``` with the release name you want to give it.
   ```
   helm install . --name mynginx-controller-release
   ```
   This creates a deployment called ```nginx-ingress``` in your Smart Cluster.

8. Retrieve the ingress address.
   Run the following command to get the ingress address.
   ```
   kubectl describe svc nginx-ingress | grep 'LoadBalancer Ingress'
   ```
   The resulting output looks something like this.
```
LoadBalancer Ingress: nginx-ingress.cluster-example-ad9291a0209fd11e8bd8902eba45d8e2-820524523.vke-user.com
```
Record this address for future use. You need it to configure your ingress resource.    

<!--
---
## Delete Your XYZ Deployment (optional)
To uninstall XYZ, use the following command.
```
kubectl delete -f yaml/zzzproductzzz.yaml
```  
-->

<br>

---
## What's Next
You now have an ingress controller installed in your Smart Cluster. Now you can configure an ingress resource that allows your Smart Cluster to use the controller.

<!--
---
## Troubleshooting
(solutions to common issues)  
-->

<br>

---
## Resources
- For more information about the NGINX ingress controller from NGINX, see <https://github.com/nginxinc/kubernetes-ingress/blob/master/README.md>.
- For more information about Kubernetes, see <https://kubernetes.io/docs/concepts>.  

<br>

---
## About Ecosystem Solutions for VMware Cloud PKS
These solutions explain how to install and use some common ecosystem tools on VMware Cloud PKS. For more information, including approved usage, contributions, and other ecosystem solutions, see the [README.MD](../README.MD) file at the top of the repository.