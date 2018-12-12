# Implementing Istio as a Service Mesh on VMware Cloud PKS
Istio provides a service mesh with multiple components that allow you to manage many aspects of complex deployments, including ingress, traffic management, and security.

**[[[ And another paragraph that briefly describes the "Why?" - typical use cases for this product. ]]]**  

This procedure explains how to configure and install Istio on a new Smart Cluster.

For more information about Istio, see <https://istio.io>.

### Versions
This implementation uses the following versions:  

| Software | Version |
| :------ | ---: |
| Istio      | 1.0.2 |
| Kubernetes | 1.10.2-80 |
| Helm | 2.8.0 |

### Before You Begin
This procedure requires that you have a VMware Cloud Services organization with access to the VMware Cloud PKS service.  

The procedure outlined in this document assumes that you have the following tools installed locally.
- cURL - command-line tool for transferring data
- kubectl - the Kubernetes command line interface
- Helm - the Kubernetes package manager

And that you have done the following.
1. Create a Smart Cluster.
2. Connect to your Smart Cluster with kubectl.
3. Initialize Helm, which you will use for installation.

For more information on these steps, see the [VMware Cloud PKS documentation](https://docs.vmware.com/en/VMware-Cloud-PKS/index.html).

---
## Step 1: Download and Install Istio
The Istio download is a compressed directory that contains the YAML files and the ```istioctl``` CLI, along with other tools and samples.
1. Download the compressed installation directory from <https://istio.io>.  
   Note that this implementation is tested with Istio **1.0.2**.
2. Extract the compressed installation directory.
3. Add the Istio ```bin``` directory to your PATH.  
For example, on Linux you can use the following commands.
   ```
   cd <path-to>/istio-1.0.2/bin
   export PATH=$PWD:$PATH
   ```

---
## Step 2: Configure the Istio YAML File
1. In the Istio install directory, create a working directory where you configure your YAML deployment file. For example: 
   ```
   cd <path-to>/istio-1.0.2
   mkdir yaml
   ```
2. Make a copy of the template.
   ```
   helm template install/kubernetes/helm/istio --name istio --namespace istio-system > yaml/istio.yaml
   ```
3. Open the YAML file in a text editor and comment out the ```nodePort``` specifications in the service spec, as shown in the example below. (Hint: search for "nodeport". )   
The node ports defined in the downloaded template cannot be used because they fall outside the allowed node port ranges for a Smart Cluster in VMware Cloud PKS. Remove these specifications and allow VMware Cloud PKS to assign the node ports.

```
. . . 

---
apiVersion: v1
kind: Service
metadata:
  name: istio-ingressgateway
  namespace: istio-system
  annotations:
  labels:
    chart: gateways-1.0.1
    release: RELEASE-NAME
    heritage: Tiller
    app: istio-ingressgateway
    istio: ingressgateway
spec:
  type: LoadBalancer
  selector:
    app: istio-ingressgateway
    istio: ingressgateway
  ports:
    -
      name: http2
      #######################
      # Comment out nodeport
      #
      # nodePort: 31380
      port: 80
      targetPort: 80
    -
      name: https
      #######################
      # Comment out nodeport
      #
      # nodePort: 31390
      port: 443
    -
      name: tcp
      #######################
      # Comment out nodeport
      #
      # nodePort: 31400
      port: 31400
    -
      name: tcp-pilot-grpc-tls
      port: 15011
      targetPort: 15011
    -
      name: tcp-citadel-grpc-tls
      port: 8060
      targetPort: 8060

. . . 

```

---
## Step 3: (optional) Configure Istio Components for High Availability
You can optionally generate multiple replicas and use ```podAntiAffinity``` to configure Istio for high availability on a *production* Smart Cluster.

The default YAML defines a single replica for each of the control plane components (Pilot, Citadel, and Galley). Update the ```deployment``` spec for each of these components as follows:  
1. To make sure there are multiple replicas of each of these components, update the replica count.
2. To make sure they are distributed across multiple nodes, add a ```podAntiAffinity``` spec.
3. To make sure they are distributed across multiple availability zones, use ```hostname``` for the ```topologyKey```.

The example below shows the spec for Pilot. Implement these changes for Citadel and Galley as well. Note that you'll need to adjust the ```values``` spec under both ```- key: istio``` and ```- key: app``` to the name of the component (pilot, citadel, galley). 
```
. . . 

---
# Source: istio/charts/pilot/templates/deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: istio-pilot
  namespace: istio-system
  labels:
    app: istio-pilot
    chart: pilot-1.0.1
    release: istio
    heritage: Tiller
    istio: pilot
  annotations:
    checksum/config-volume: f8da08b6b8c170dde721efd680270b2901e750d4aa186ebb6c22bef5b78a43f9
spec:
  ###############################################
  # Increase replica count to 3
  #
  replicas: 3
  template:
    metadata:
      labels:
        istio: pilot
        app: pilot

      . . . [code snipped] . . .

      volumes:
      - name: config-volume
        configMap:
          name: istio
      - name: istio-certs
        secret:
          secretName: istio.istio-pilot-service-account
          optional: true   
      affinity:
        ###############################################
        # insert podAntiAffinity section starting here
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: istio
                operator: In
                values:
                - pilot
              - key: app
                operator: In
                values:
                - pilot
            topologyKey: kubernetes.io/hostname
        # podAntiAffinity section ends here
        ###############################################
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                - amd64
                - ppc64le
                - s390x

          . . . 

```

---
## Step 4: Deploy and Verify Your Istio Installation
After you have finished configuring your YAML file, you can deploy Istio.

Make sure you have connected kubectl to your Smart Cluster before proceeding. 

1. Use kubectl to install Istio's custom resource definitions.  
```
cd <path-to>/istio-1.0.2
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
```  
Wait a few seconds for the CRDs to be committed in the kube-apiserver before continuing.

2. Create a namespace for Istio.
```
kubectl create namespace istio-system 
```
3. Deploy Istio to your Smart Cluster.
```
kubectl apply -f yaml/istio.yaml
```
4. Verify that the necessary services are deployed.
```
kubectl get svc -n istio-system
```
The resulting output should look something like this.
```
NAME                       TYPE           CLUSTER-IP   EXTERNAL-IP        PORT(S)                                                                                                                  AGE
istio-citadel              ClusterIP      10.0.0.93    <none>             8060/TCP,9093/TCP                                                                                                        38s
istio-egressgateway        ClusterIP      10.0.0.42    <none>             80/TCP,443/TCP                                                                                                           38s
istio-galley               ClusterIP      10.0.0.164   <none>             443/TCP,9093/TCP                                                                                                         38s
istio-ingressgateway       LoadBalancer   10.0.0.156   istio-ingress...   80:30407/TCP,443:30410/TCP,31400:30434/TCP,15011:30442/TCP,8060:30411/TCP,853:30418/TCP,15030:30403/TCP,15031:30419/TCP  38s
istio-pilot                ClusterIP      10.0.0.50    <none>             15010/TCP,15011/TCP,8080/TCP,9093/TCP                                                                                    38s
istio-policy               ClusterIP      10.0.0.32    <none>             9091/TCP,15004/TCP,9093/TCP                                                                                              38s
istio-sidecar-injector     ClusterIP      10.0.0.196   <none>             443/TCP                                                                                                                  38s
istio-statsd-prom-bridge   ClusterIP      10.0.0.105   <none>             9102/TCP,9125/UDP                                                                                                        38s
istio-telemetry            ClusterIP      10.0.0.130   <none>             9091/TCP,15004/TCP,9093/TCP,42422/TCP                                                                                    38s
prometheus                 ClusterIP      10.0.0.189   <none>             9090/TCP                                                                                                                 38s

```

5. Verify that the pods are deployed and the containers are up and running.
```
kubectl get pod -n istio-system
```
The resulting output should look something like this.
```
NAME                                        READY     STATUS      RESTARTS   AGE
istio-citadel-84b7985bf-9xdmt              1/1       Running     0          5m
istio-cleanup-secrets-5j7g                 0/1       Completed   0          5m
istio-egressgateway-8fbdb886-94srn         1/1       Running     0          5m
istio-galley-cf4cb598-chkkm                1/1       Running     0          5m
istio-ingressgateway-774cbcc85-mnpk2       1/1       Running     0          5m
istio-pilot-5d57bb584-hfhrd                2/2       Running     0          5m
istio-policy-7754cb58-jnchf                2/2       Running     0          5m
istio-sidecar-injector-856dc499d-p6s6n     1/1       Running     0          5m
istio-statsd-prom-bridge-64b9d8f6b-qflvm   1/1       Running     0          5m
istio-telemetry-54c4f5cb7-pb65t            2/2       Running     0          5m
prometheus-76dd6d86b-gxsq7                 1/1       Running     0          5m

```
6. You can optionally test your Istio deployment using the bookinfo app, which illustrates many use cases for Istio. For more information, see <https://istio.io/docs/examples/bookinfo/>.

---
## Step 5: (optional) Install Jaeger
Jaegar is a distributed tracing system, used for monitoring microservices-based distributed systems. You can optionally install Jaeger on top of Istio. For more information about Jaeger, see <https://github.com/jaegertracing/jaeger>.

- To install Jaeger after you have installed Istio as described above, use the following command.
```
kubectl create -n istio-system -f https://raw.githubusercontent.com/jaegertracing/jaeger-kubernetes/master/all-in-one/jaeger-all-in-one-template.yml
```
This YAML creates four services and a deployment extension.
```
deployment.extensions "jaeger-deployment" created
service "jaeger-query" created
service "jaeger-collector" created
service "jaeger-agent" created
service "zipkin" created
```

- You can also install Jaeger when you install Istio. When you make a copy of the installation YAML file, add the ```tracing.enabled=true``` flag as shown in the following command. You must then configure the installation YAML file and create the namespace, as described above.
```
helm template install/kubernetes/helm/istio --name istio --namespace istio-system --set tracing.enabled=true > yaml/istio.yaml
```

---
## Step 6: (optional) Delete Your Istio Deployment
To uninstall Istio, run the ```kubectl delete``` command in your Istio install directory.
```
cd <path-to>/istio-1.0.2
kubectl delete -f yaml/istio.yaml
```
---
## What's Next

**[[[ Summary of what you now have and what you can do with it. Perhaps you can point to quickstart deployments provided by the highlighted product. ]]]**  

---
## Troubleshooting
(solutions to common issues)  

---
## Resources
(pointers to other helpful documentation)  

---
## About Ecosystem Solutions for VMware Cloud PKS
These solutions explain how to install and use some common ecosystem tools on VMware Cloud PKS. For more information, including approved usage, contributions, and other ecosystem solutions, see the [README.MD](../README.MD) file at the top of the repository.  
