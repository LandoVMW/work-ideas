# Implementing NGINX from NGINX *for High Availability* in VMware Cloud PKS
An ingress exposes routes from outside the cluster to services inside the cluster. 
The procedure in this document describes how to install the NGINX ingress controller that is maintained by NGINX for high availability. 

**[[And another paragraph that briefly describes the "Why?" - typical use cases for this product.]]**


### Versions
This implementation uses the following versions:  

| Software    | Version  |           | notes |
| :------     | ---:     |     ---:  | :---  |
| NGINX       | 1.0.2    | 0.23.0    |       |
| Kubernetes* | 1.11.5-1 | 1.10.11-1 | running in a Smart Cluster |


### Before You Begin
This procedure requires that you have a VMware Cloud Services organization with access to the VMware Cloud PKS service.

The procedure outlined in this document assumes that you have the following tools installed locally.

- kubectl - the Kubernetes command line interface
- Git

And that you have done the following.

1. Create a *production* Smart Cluster.
2. Connect to the cluster with kubectl.
3. Create a local work directory into which you will clone the project and update its configuration.

For more information on these steps, see the [VMware Cloud PKS documentation](https://docs.vmware.com/en/VMware-Kubernetes-Engine/index.html).

<br>

---
## Steps
This procedure shows how to create a NGINX ingress controller deployment in multiple availability zones on your Smart Cluster. 

When you install the controller, you must modify the default settings. At a minimum, you must update ```config.entries``` and probably specify more than one replica of the controller.

Use the following steps to download the project, modify the ```values.yaml``` file, and deploy the controller.

1. Clone the project into your local work directory.
   ``` 
   git clone https://github.com/nginxinc/kubernetes-ingress
   ```  
   This creates a directory called ```kubernetes-ingress``` that contains the project.

2. Change to the ```deployment``` directory.
   ```
   cd kubernetes-ingress/deployments/deployment
   ```
3. Open the ```nginx-ingress.yaml``` file in a text editor.

4. Locate ```replicaCount``` and update its value to ```3```.
```
. . . 
  replicaCount: 3
. . . 
```

Typically you want more than one replicas of the controller.

5. Locate the ```.spec.template.spec``` section, where you will insert an ```affinity``` section prior to ```serviceAccountName```, as shown below.
```
spec:
  . . . 
  template:
    . . . 
    spec:
      ## Insert affinity section here
      serviceAccountName: nginx-ingress
      containers:
      . . . 
```

6. Insert the ```affinity``` section shown below, prior to ```serviceAccountName```.
```
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx-ingress
            topologyKey:  failure-domain.beta.kubernetes.io/zone
```

7. Save and close the ```nginx-ingress.yaml``` file.

8. Follow the instructions in the [installation.md](https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/installation.md) file on GitHub to create a deployment. 

9. Retrieve the ingress address.
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