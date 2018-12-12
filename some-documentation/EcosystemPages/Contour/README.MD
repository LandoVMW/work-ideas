# Implementing Contour as an Ingress in VMware Cloud PKS
Contour is an ingress controller for Kubernetes from Heptio.   

Configure ingress in your Smart Cluster to allow inbound connections to access apps and services.  

### Versions
This implementation uses the following versions:  

| Software | Version |
| :------ | ---: |
| Contour      | 0.7.0 |
| Kubernetes | 1.10.2-80 |


### Before You Begin
This procedure requires that you have a VMware Cloud Services organization with access to the VMware Cloud PKS service.

The procedure outlined in this document assumes that you have the following tools installed locally.
 - cURL - command-line tool for transferring data
 - kubectl - the Kubernetes command line interface

And that you have done the following.  
 1. Create a Smart Cluster. 
 2. Connect to the cluster with kubectl.  

For more information on these steps, see the [VMware Cloud PKS documentation](https://docs.vmware.com/en/VMware-Kubernetes-Engine/index.html).


---
## Steps
This procedure shows how to create a Contour ingress controller deployment on your Smart Cluster using the YAML spec from Heptio.  

You have two options for how to perform this.
 - [Install using the default values](#install-with-default-values)
 - [Install after modifying default settings](#install-after-modifying-default-settings)


### Install with default values
Install the controller directly from the YAML spec on j.hept.io without modification. 
Use the following command to install the RBAC-enabled Contour ingress controller with default settings.  
```
kubectl apply -f https://j.hept.io/contour-deployment-rbac
```


### Install after modifying default settings
If you want to change the default values, for example to change the number of replicas, use the following steps to download the settings to a YAML file that you can update locally and use to install the controller.

#### Step 1: Download the default settings defined for the controller to a local YAML file using the following command.
   ```
   curl https://raw.githubusercontent.com/heptio/contour/master/deployment/render/deployment-rbac.yaml > contour-deployment-rbac.yaml
   ```  
   This creates a file called contour-deployment-rbac.yaml that contains the deployment settings defined for the controller.  
#### Step 2: Modify the ```contour-deployment-rbac.yaml``` file.
  1. Open the ```contour-deployment-rbac.yaml``` file in a text editor.
  2. Locate the ```spec.replicas``` section for the deployment, and update the value of ```replicaCount``` to ```3```. 
```
. . . 
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: contour
  name: contour
  namespace: heptio-contour
spec:
  selector:
    matchLabels:
      app: contour
  replicas: 3
. . . 
```

  3. For high availability deployment across availability zones in a production Smart Cluster, change the ```topologyKey``` in the ```affinity``` block to use ```zone``` rather than ```hostname```. For example:
```
. . . 
 affinity:
   podAntiAffinity:
     preferredDuringSchedulingIgnoredDuringExecution:
     - weight: 100
       podAffinityTerm:
       labelSelector:
         matchLabels:
           app: contour
       topologyKey: failure-domain.beta.kubernetes.io/zone
. . . 
```

  4. Save and close the ```contour-deployment-rbac.yaml``` file.
#### Step 3: Install the controller.
```
kubectl apply -f contour-deployment-rbac.yaml
```
#### Step 4: Retrieve the ingress address.
Run the following command to get the ingress address.
```
kubectl describe svc contour -n heptio-contour | grep 'LoadBalancer Ingress'
```
The resulting output looks something like this.
```
LoadBalancer Ingress: contour.cluster-example-ad9291a0209fd11e8bd8902eba45d8e2-820524523.vke-user.com
```
Record this address for future use. You need it to configure your ingress resource.  


---
## What's Next
This procedure created a deployment called ```contour``` running the ingress controller in a new namespace called ```heptio-contour``` in your Smart Cluster.  

After installing the ingress controller, configure an ingress resource that allows your Smart Cluster to use the controller.


---
## Resources
For more information about Heptio, go to <https://heptio.com/>.  
For more information about the Contour controller, see <https://github.com/heptio/contour>.  

---
## About Ecosystem Solutions for VMware Cloud PKS
(link back to the top-level page of the 'ecosystem solutions' repository for information about license / user agreement / other legalese / contributions / etc)
