
# Virtual Kublet Example on Azure

Setting up an AKS clust on Azure and configuring the Virutal Kublet ACI connector

Create a new resource group for the AKS cluster

```bash
az group create -n shayne-kublet -l eastus
```

Create the AKS cluster
```bash
az aks create -n shayne-kublet -g shayne-kublet --node-count=1 --generate-ssh-keys
```

Configure `kubectl` to connect to the AKS cluster
```bash
az aks get-credentials -n shayne-kublet -g shayne-kublet
```

Initialize Helm on the cluster for installing the connector
```bash
helm init
```

Install the ACI Connector for the cluster
```bash
az aks install-connector -g shayne-kublet -n shayne-kublet --connector-name shayneaciconnector
```

Validate the ACI connector is installed and initiated by running `kubectl get nodes`

```bash
NAME                       STATUS    ROLES     AGE       VERSION
aci-connector              Ready     <none>    9m        v1.6.6
aks-nodepool1-26023942-0   Ready     agent     12m       v1.7.7
```

Create the sample ASP.NET Core Razor Pages app using the yml file

```bash
kubectl create -f sample-dotnet-aci.yml
```
Running `kubectl -o wide` shows the ACI Connector pod as well as the sample app container pod

```bash
NAME                                                READY     STATUS    RESTARTS   AGE       IP             NODE
aci-hellorazor-116105062-54bzl                      1/1       Running   0          1m        52.168.55.44   aci-connector
shayneaciconnector-aci-connector-1640834989-86nn0   1/1       Running   0          51m       10.244.0.9     aks-nodepool1-26023942-0
```

See the list of containers in ACI using `az container list -o table`

```bash
Name                            ResourceGroup    ProvisioningState    Image
------------------------------  ---------------  ------------------  -------------------
aci-hellorazor-116105062-54bzl  shayne-kublet    Succeeded            spboyer/hellorazor

 IP:ports          CPU/Memory       OsType    Location
 ----------------  ---------------  --------  ----------
 52.168.55.44:80   1.0 core/1.5 gb  Linux     eastus
```

Scale the deployment and look at the containers in ACI

```bash
kubectl scale --replicas=3 deployment/aci-hellorazor
```

`az container list -o table`

```bash
Name                            ResourceGroup    ProvisioningState    Image
------------------------------  ---------------  ------------------  -------------------
aci-hellorazor-116105062-54bzl  shayne-kublet    Succeeded            spboyer/hellorazor
aci-hellorazor-116105062-71pcj  shayne-kublet    Succeeded            spboyer/hellorazor
aci-hellorazor-116105062-g5lzv  shayne-kublet    Succeeded            spboyer/hellorazor
```

You can remove the connector using `az aks az aks remove-connector`

```bash
az aks remove-connector --resource-group shayne-kublet --name shayne-kublet --connector-name shayneaciconnector
```

