# Implementing XYZ as an ABC in VMware Cloud PKS
Introductory paragraph that describes what you're going to do. Introductory paragraph that describes what you're going to do. Introductory paragraph that describes what you're going to do. Introductory paragraph that describes what you're going to do. 

And another paragraph that briefly describes the "Why?" - typical use cases for this product.

### Versions
This implementation uses the following versions:  

| Software | Version |
| :------ | ---: |
| Istio      | 1.0.2 |
| Kubernetes | 1.10.2-80 |
| Helm | 2.8.0 |


### Before You Begin
[[reminder about the prereqs for this implementation. this includes simple VMware Cloud PKS procedures that are described in the VMware Cloud PKS documentation. ]]
1. Create a Smart Cluster in privileged mode. 
2. Connect to the cluster with kubectl.
3. Initialize Helm, which will be used for installation.

For more information on these steps, see the [VMware Cloud PKS documentation](https://docs.vmware.com/en/VMware-Kubernetes-Engine/index.html).

- You will also need to download and install XYZ locally, which you can get from ((here)).  

---
## Steps
(Start with a list (TOC) of the major steps, which can be added during editing. Initially just record the actual steps to implementation.)

1. Install an ingress controller for your cluster, and create a resource for it.
2. Determine the deployment and service name for the ingress controller.
   ```
   kubectl get deployment -n kube-system
   kubectl get svc -n kube-system
   ```
3. Configure your XYZ deployment YAML.
    1. Open ```xyzdepl.yaml``` in a text editor.
    2. Update the following lines.
       ```
       lipsem-orem=**pasquilli-magenta**
       ```
    3. Save and exit.
4. Deploy XYZ to your cluster.
   ```
   xyz install --flag12 --flag34
   ```

---
## Delete Your XYZ Deployment (optional)
To uninstall XYZ, use the following command.
```
kubectl delete -f yaml/zzzproductzzz.yaml
```  

---
## What's Next
Summary of what you now have and what you can do with it.
Perhaps you can point to quickstart deployments provided by the highlighted product.  

---
## Troubleshooting
(solutions to common issues)  

---
## Resources
- (pointers to other helpful documentation)
- For more information about Kubernetes, see https://kubernetes.io/docs/concepts/.  

---
## About Ecosystem Solutions for VMware Cloud PKS
These solutions explain how to install and use some common ecosystem tools on VMware Cloud PKS. For more information, including approved usage, contributions, and other ecosystem solutions, see the [README.MD](../README.MD) file at the top of the repository.
