# Implementing NGINX from the Kubernetes Community as an Ingress Controller in VMware Cloud PKS
An ingress exposes routes from outside the cluster to services inside the cluster. 
The procedure in this document describes how to install the NGINX ingress controller that is maintained by the Kubernetes community. 

**[[And another paragraph that briefly describes the "Why?" - typical use cases for this product.]]**

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

And that you have done the following.

1. Create a Smart Cluster.
2. Connect to the cluster with kubectl.

The latest versions of this controller require a Smart Cluster created with privileged mode enabled. If you want to run this controller without privileged mode, use an earlier version of the controller.


For more information on these steps, see the [VMware Cloud PKS documentation](https://docs.vmware.com/en/VMware-Kubernetes-Engine/index.html).

---
## Steps
This procedure shows how to create a NGINX ingress controller deployment on your Smart Cluster using a Helm chart. 

When you install the controller, you must modify the default settings. At a minimum, you must enable RBAC and probably specify more than one replica of the controller.

You have two options for how to perform this.
 - [Install using command-line flags](#install-with-default-values) to specify the settings you want to change.
 - [Install after downloading and modifying default settings](#install-after-downloading-and-modifying-default-settings).  

Then you can [retrieve the ingress address](#retrieve-the-ingress-address).


### Install using command-line flags
Use the following command to install the NGINX ingress controller with 3 replicas and RBAC enabled, replacing *k8s-nginx-ingress* with the name you want to give the deployment. The latest stable version of this Helm chart requires privileged mode enabled in your Smart Cluster.
```
helm install stable/nginx-ingress --name k8s-nginx-ingress --set controller.replicaCount=3 --set rbac.create=true
```
To install an earlier version of the controller that does not require privileged mode, use the ```version``` flag. For example:
```
helm install stable/nginx-ingress --name k8s-nginx-ingress --set controller.replicaCount=3 --set rbac.create=true --version 0.23.0
```


### Install after downloading and modifying default settings
If you want to change the default values, for example to change the number of replicas, use the following steps to download the settings to a YAML file that you can update locally and use to install the controller.

1. Download the default values defined for the Helm chart using the following command.
```
helm inspect values stable/nginx-ingress > nginx-ingress-values.yaml
```  
This creates a file called ```nginx-ingress-values.yaml``` that contains the default values defined for the Helm chart.

2. Open the ```nginx-ingress-values.yaml``` file in a text editor.

3. Locate the ```rbac.create``` section, and make sure that the value for ```create``` is ```true```.
```
 . . . 
rbac:
  create: true
. . .
```

4. Locate the ```controller.replicaCount``` section, and update the value of ```replicaCount``` to ```3```.
```
. . . 
controller:
. . . 
  replicaCount: 3
. . . 
```
Typically you want more than one replicas of the controller.

5. You can optionally add a ```podAntiAffinity``` spec to the ```affinity``` section to make sure the ingress controller pods are distributed across different nodes.  
Locate the ```affinity``` section, and replace the empty brackets as shown below.
```
. . . 
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - nginx-ingress
          - key: component
            operator: In
            values:
            - controller
        topologyKey: kubernetes.io/hostname
. . . 
```

6. For high availability deployment across availability zones in a production Smart Cluster, change the ```topologyKey``` in the ```affinity``` block to use ```zone``` rather than ```hostname```. For example:
```
. . . 
  affinity:
    podAntiAffinity:
      - labelSelector:
. . . 
        topologyKey: failure-domain.beta.kubernetes.io/zone
. . . 
```

7. Save and close the ```nginx-ingress-values.yaml``` file.

8. Install the Helm chart.  
   Use the following command to install the NGINX ingress controller, replacing ```k8s-nginx-ingress``` with the name you want to give the deployment.
   ```
   helm install stable/nginx-ingress --name k8s-nginx-ingress -f nginx-ingress-values.yaml
   ```
   This creates two deployments called ```k8s-nginx-ingress-controller``` and ```k8s-nginx-ingress-default-backend``` in your Smart Cluster.

### Retrieve the ingress address
Run the following command to get the ingress address.
```
kubectl describe svc k8s-nginx-ingress-controller | grep 'LoadBalancer Ingress'
```
The resulting output looks something like this.
```
LoadBalancer Ingress: k8s-nginx-ingress-controller.cluster-example-ad9291a0209fd11e8bd8902eba45d8e2-820524523.vke-user.com
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
---
## What's Next
You now have an ingress controller installed in your Smart Cluster. Now you can configure an ingress resource that allows your Smart Cluster to use the controller.

<!--
---
## Troubleshooting
(solutions to common issues)  
-->
---
## Resources
- For more information about the NGINX ingress controller from Kubernetes, see <https://kubernetes.github.io/ingress-nginx>.
- For more information about Kubernetes, see <https://kubernetes.io/docs/concepts>.  

---
## About Ecosystem Solutions for VMware Cloud PKS
These solutions explain how to install and use some common ecosystem tools on VMware Cloud PKS. For more information, including approved usage, contributions, and other ecosystem solutions, see the [README.MD](../README.MD) file at the top of the repository.