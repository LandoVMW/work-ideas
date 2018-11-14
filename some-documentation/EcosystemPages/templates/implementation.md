# Implementing XYZ as an ABC in VKE
Introductory paragraph that describes what you're going to do. Introductory paragraph that describes what you're going to do. Introductory paragraph that describes what you're going to do. Introductory paragraph that describes what you're going to do. 

### Before You Begin
[[reminder about the prereqs for this implementation. this includes simple vke procedures that are described in the VKE documentation. ]]
1. Create a Smart Cluster in privileged mode. 
2. Connect to the cluster with kubectl.
3. Initialize Helm, which will be used for installation.

For more information on these steps, see the VKE documentation at <https://docs.vmware.com/en/VMware-Kubernetes-Engine/index.html>.

- You will also need to download and install XYZ locally, which you can get from ((here)).

---
## Steps
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
5. Deploy XYZ to your cluster.
```
xyz install --flag12 --flag34
```

---
## Delete Your XYZ Deployment
To uninstall XYZ, use the following command.
```
kubectl delete -f yaml/zzzproductzzz.yaml
```

---
## What's Next
Summary of what you now have and what you can do with it.



