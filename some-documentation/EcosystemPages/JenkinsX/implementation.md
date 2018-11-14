# Implementing Jenkins-X as CI/CD Pipeline in VMware Cloud PKS
This procedure explains how to configure and install **Jenkins-X** on a new Smart Cluster.

### Before You Begin
The procedure outlined in this document assumes that you have the following tools installed locally.

- Git - You must also have a Git repository provider to create environment repositories (GitHub, by default).
- kubectl - the Kubernetes command line interface
- Helm - the Kubernetes package manager

And that you have done the following.

1. Create a Smart Cluster with privileged mode enabled.
2. Connect to your Smart Cluster with kubectl.
3. Initialize Helm, which you will use for installation.

For more information on these steps, see the VMware Cloud PKS documentation at <https://docs.vmware.com/en/VMware-Kubernetes-Engine/index.html>.  


---
## Step 1: Set Up Ingress on Your Smart Cluster
1. Download and install the JX tool, which automates the process of deploying Jenkins-X on a kubernetes cluster.  
   Follow the instructions at <https://jenkins-x.io/getting-started/install/>.  
   **Note:** If you have installed or attempted to install Jenkins-X previously, delete all files in the ```~/.jx``` directory to purge settings from any previous installation.

2. Install the Nginx ingress controller from Kubernetes.  
   Make sure kubectl is configured to connect to your Smart Cluster before you start.
   ```
   helm install stable/nginx-ingress -n ingress --set rbac.create=true --version 0.23.0 --namespace kube-system
   ```
   It takes several minutes to install the ingress controller and configure the external endpoint for access. You can check for the external access endpoint by running the following command. The ```-w``` flag tells ```kubectl``` to wait for an update, such as the endpoint being configured. 
   ```
   kubectl get services --namespace kube-system -o wide -w ingress-nginx-ingress-controller
   ```
   After some time, the EXTERNAL-IP property is configured. The value looks something like this: 
   ```
   ingress-nginx-ingress-controller.<cluster-address>.vke-user.com
   ```
   This value is used for the ```external-ip``` parameter when you run the ```jx install``` command.  
   When the EXTERNAL-IP property is configured, press Ctrl-C to cancel the command. 

3. Use kubectl to determine the deployment and service name for the ingress controller.
   ```
   kubectl get deployment -n kube-system
   kubectl get svc -n kube-system
   ```
   Both the deployment and the service should be named ```ingress-nginx-ingress-controller``` by default.  

---
## Step 2: Run the JX Installer

1. Configure Git to use the account that Jenkins will interact with.  
   If you intend to test your installation with the quickstarts, the account should have read/write access. The following commands show how to set the Git properties. Make sure you replace the example values for user name (*\<jx-git-user\>*) and user email (*\<jx-git-user@vmware.com\>*) in these commands.
   ```
   git config --global --add user.name <jx-git-user>
   git config --global --add user.email <jx-git-user@vmware.com>
   ```

2. Run the JX installer with the following parameters.
    - ```provider=kubernetes```  
    JX provides built-in integration for certain providers. To integrate with VMware Cloud PKS, specify a generic kubernetes provider.
    - ```external-ip```  
    Use the address of your ingress controller from step 2. It looks something like this: ```   ingress-nginx-ingress-controller.<cluster-address>.vke-user.com```
    - ```ingress-service=ingress-nginx-ingress-controller```  
    Use the name of the ingress service from step 3.
    - ```ingress-deployment=ingress-nginx-ingress-controller```  
    Use the name of the ingress deployment from step 3.
    - ```ingress-namespace=kube-system```  
    Use the kube-system namespace.
    
    The following is an example of the command to use.  
    Make sure you replace the example value for external IP (*\<external-ip\>*) in this command.
    ```
    jx install --provider=kubernetes --external-ip=<external-ip> --ingress-service=ingress-nginx-ingress-controller --ingress-deployment=ingress-nginx-ingress-controller --ingress-namespace=kube-system
    ```
3. If the installer prompts to resolve the IP address, enter ```y```.
   The prompt looks something like this.
   ```
   The Ingress address  ingress-nginx-ingress-controller.<cluster-address>.vke-user.com is not an IP address. We recommend we try resolve it to a public IP address and use that for the domain to access services externally? (y/N)
   ```
   When you choose yes, JX internally sets up a DNS name using the resolved AWS IP address. For example, if the AWS IP address is ```52.32.77.25```, then it creates the DNS name ```<clustername>.jx.52.32.77.25.nip.io```. This address can be accessed from a browser using the URL ```http://<clustername>.jx.52.32.77.25.nip.io/me/configure```. 

   The installer might also prompt for the following inputs. These can also be provided as arguments for a fully automated deployment.
    - ```git username```  
      username of a Git account
    - ```git API token```  
      API token associated with the Git username
    - ```jenkins API token```  
      API token from Jenkins server that is deployed to validate

---
## What's Next
When the installer completes, Jenkins-X CI/CD pipeline is ready to execute jobs. 

Jenkins-X includes some quickstart guides for trying out the system. You can execute one of the quickstarts such as the following Go lang quickstart.

```
jx create quickstart -l go
```
This creates a new Jenkins job on the Jenkins master. The progress of this job can be tracked through the Jenkins console:
```
http://<clustername>.jx.52.32.77.25.nip.io/
```
The Jenkins job creates a pod which acts as a Jenkins worker for the execution of the job. After the job is completed, the pod is terminated. 




