# Module 0: Creating AKS cluster

The following guide is based upon the repos from [lastcoolnameleft](https://github.com/lastcoolnameleft/kubernetes-workshop/blob/master/create-aks-cluster.md) and [Azure Kubernetes Hackfest](https://github.com/Azure/kubernetes-hackfest/tree/master/labs/create-aks-cluster#readme).

* * *

**Goal:** Create AKS cluster.

> This workshop uses AKS cluster with Linux containers. To create a Windows Server container on an AKS cluster, consider exploring [AKS documents](https://docs.microsoft.com/en-us/azure/aks/windows-container-cli). This cluster deployment utilizes Azure CLI v2.x from your local terminal or via Azure Cloud Shell. Instructions for installing Azure CLI can be found [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

\[If you already have AKS cluster, make sure the network plugin is "azure", then you can skip this module and go to [module 2](/Applications/Joplin.app/Contents/Resources/modules/joining-aks-to-calico-cloud.md "../modules/joining-aks-to-calico-cloud.md")

## Prerequisite Tasks

Follow the prequisite steps if you need to verify your Azure subscription and Service Principal otherwise proceed to step 1.

- Ensure you are using the correct Azure subscription you want to deploy AKS to.
    
	```
	# View subscriptions
	az account list   
 
  # Verify selected subscription
  az account show
  ```
    
    ```
  # Set correct subscription (if needed)
  az account set --subscription <subscription_id>
  
  # Verify correct subscription is now set
  az account show
  ```
    
- Create Azure Service Principal to use through the labs

	```bash
	az ad sp create-for-rbac --skip-assignment
	```

- This will return the following. !!!IMPORTANT!!! - Please copy this information down as you'll need it for labs going forward.

	```bash
	"appId": "7248f250-0000-0000-0000-dbdeb8400d85",
	"displayName": "azure-cli-2017-10-15-02-20-15",
	"name": "http://azure-cli-2017-10-15-02-20-15",
	"password": "77851d2c-0000-0000-0000-cb3ebc97975a",
	"tenant": "72f988bf-0000-0000-0000-2d7cd011db47"
	```

- Set the values from above as variables **(replace <appid><password>with your values)</password></appid>**.

> **Warning:** Several of the following steps have you echo values to your .bashrc file. This is done so that you can get those values back if your session reconnects. You will want to remember to clean these up at the end of the training, in particular if you're running on your own, or your company's, subscription.

DON'T MESS THIS STEP UP. REPLACE THE VALUES IN BRACKETS!!!

```bash
# Persist for Later Sessions in Case of Timeout
APPID=<appId>
echo export APPID=$APPID >> ~/.bashrc
CLIENTSECRET=<password>
echo export CLIENTSECRET=$CLIENTSECRET >> ~/.bashrc
```

## Steps

1.  Create a unique identifier suffix for resources to be created in this lab.
    
	```bash
    UNIQUE_SUFFIX=$USER$RANDOM
    # Remove Underscores and Dashes (Not Allowed in AKS and ACR Names)
    UNIQUE_SUFFIX="${UNIQUE_SUFFIX//_}"
    UNIQUE_SUFFIX="${UNIQUE_SUFFIX//-}"
    # Check Unique Suffix Value (Should be No Underscores or Dashes)
    echo $UNIQUE_SUFFIX
    # Persist for Later Sessions in Case of Timeout
    echo export UNIQUE_SUFFIX=$UNIQUE_SUFFIX >> ~/.bashrc
	```
    
    **_ Note this value as it will be used in the next couple labs. _**
	
2. Create an Azure Resource Group in your chosen region. We will use East US in this example.

   ```bash
   # Set Resource Group Name using the unique suffix
   RGNAME=aks-rg-$UNIQUE_SUFFIX
   # Persist for Later Sessions in Case of Timeout
   echo export RGNAME=$RGNAME >> ~/.bashrc
   # Set Region (Location)
   LOCATION=eastus
   # Persist for Later Sessions in Case of Timeout
   echo export LOCATION=eastus >> ~/.bashrc
   # Create Resource Group
   az group create -n $RGNAME -l $LOCATION
   ```
    
3.  Create your AKS cluster in the resource group created in step 2 with 3 nodes. We will check for a recent version of kubnernetes before proceeding. You will use the Service Principal information from the prerequisite tasks.
    
    Use Unique CLUSTERNAME
    
    ```bash
    # Set AKS Cluster Name
    CLUSTERNAME=aks${UNIQUE_SUFFIX}
    # Look at AKS Cluster Name for Future Reference
    echo $CLUSTERNAME
    # Persist for Later Sessions in Case of Timeout
    echo export CLUSTERNAME=aks${UNIQUE_SUFFIX} >> ~/.bashrc
    ```
    
    Get available kubernetes versions for the region. You will likely see more recent versions in your lab.
    
    ```bash
    az aks get-versions -l $LOCATION --output table
    ```
    ```
    KubernetesVersion    Upgrades
    -------------------  -----------------------
    1.21.2               None available
    1.21.1               1.21.2
    1.20.9               1.21.1, 1.21.2
    1.20.7               1.20.9, 1.21.1, 1.21.2
    1.19.13              1.20.7, 1.20.9
    1.19.11              1.19.13, 1.20.7, 1.20.9
    ```
    
     For this lab we'll use 1.21.1
    
    ```bash
    K8SVERSION=1.21.1
    echo export K8SVERSION=1.21.1 >> ~/.bashrc
    ```
    
    > The below command can take 10-20 minutes to run as it is creating the AKS cluster. Please be PATIENT and grab a coffee/tea/kombucha...
    
    ```bash
    # Create AKS Cluster - it is important to set the network-plugin as azure in order to connec to Calico Cloud
    az aks create -n $CLUSTERNAME -g $RGNAME \
    --kubernetes-version $K8SVERSION \
    --enable-managed-identity \
    --node-count 3 \
    --network-plugin azure \
    --no-wait
    
    ```

    
4.  Verify your cluster status. The `ProvisioningState` should be `Succeeded`
    
    ```bash
    az aks list -o table -g $RGNAME
    ```
    
    ```bash
    Name                 Location    ResourceGroup      KubernetesVersion    ProvisioningState    Fqdn
    -------------------  ----------  -----------------  -------------------  -------------------  ----------------------------------------------------------------
    jessie-aks-workshop  eastus      aks-rg-jessie1023  1.21.1               Succeeded            jessie-aks-aks-rg-jessie102-03cfb8-23b9dfec.hcp.eastus.azmk8s.io
    ```
    
5.  Get the Kubernetes config files for your new AKS cluster
    
    ```bash
    az aks get-credentials -n $CLUSTERNAME -g $RGNAME
    ```
    
6.  Verify you have API access to your new AKS cluster
    
    > Note: It can take 5 minutes for your nodes to appear and be in READY state. You can run `watch kubectl get nodes` to monitor status. Otherwise, the cluster is ready when the output is similar to the following:
    
	```bash
	kubectl get nodes
	```
	```
    NAME                                STATUS   ROLES   AGE   VERSION
    aks-nodepool1-36555681-vmss000000   Ready    agent   47m   v1.21.1
    aks-nodepool1-36555681-vmss000001   Ready    agent   47m   v1.21.1
    aks-nodepool1-36555681-vmss000002   Ready    agent   47m   v1.21.1
	```

	To see more details about your cluster:
	```bash
	kubectl cluster-info
	```

7. Download this repo into your environment:

    ```bash
    git clone https://github.com/tigera-solutions/calicocloud-aks-workshop.git
    ```
    
    ```bash
    cd ./calicocloud-aks-workshop
    ```

8. *[Optional]*  Install `calicoctl` CLI for use in later labs

    The easiest way to retrieve captured `*.pcap` files is to use [calicoctl](https://docs.tigera.io/maintenance/clis/calicoctl/) CLI. The following binary installations are available:

    a) CloudShell
    ```bash    
    # download and configure calicoctl
    curl -o calicoctl -O -L https://downloads.tigera.io/ee/binaries/v3.10.0/calicoctl

    chmod +x calicoctl
    
    # verify calicoctl is running 
    ./calicoctl version
    ```

    
    b) Linux
    >Tip: Consider navigating to a location that’s in your PATH. For example, /usr/local/bin/
    ```bash    
    # download and configure calicoctl
    curl -o calicoctl -O -L https://downloads.tigera.io/ee/binaries/v3.10.0/calicoctl
    chmod +x calicoctl
    
    # verify calicoctl is running 
    calicoctl version
    ```

    c) MacOS
    >Tip: Consider navigating to a location that’s in your PATH. For example, /usr/local/bin/
    ```bash    
    # download and configure calicoctl
    curl -o calicoctl -O -L  https://downloads.tigera.io/ee/binaries/v3.10.0/calicoctl-darwin-amd64

    chmod +x calicoctl
    
    # verify calicoctl is running 
    calicoctl version
    ```
    Note: If you are faced with `cannot be opened because the developer cannot be verified` error when using `calicoctl` for the first time. go to `Applicaitons` \> `System Prefences` \> `Security & Privacy` in the `General` tab at the bottom of the window click `Allow anyway`.  
    Note: If the location of calicoctl is not already in your PATH, move the file to one that is or add its location to your PATH. This will allow you to invoke it without having to prepend its location.

    d) Windows - using powershell command to download the calicoctl binary  
    >Tip: Consider runing powershell as administraor and navigating to a location that’s in your PATH. For example, C:\Windows.
    
    ```pwsh
    Invoke-WebRequest -Uri "https://downloads.tigera.io/ee/binaries/v3.9.0/calicoctl-windows-amd64.exe" -OutFile "kubectl-calico.exe"
    ```
    
   



--- 
## Next steps

You should now have a Kubernetes cluster running with 3 nodes. You do not see the master servers for the cluster because these are managed by Microsoft. The Control Plane services which manage the Kubernetes cluster such as scheduling, API access, configuration data store and object controllers are all provided as services to the nodes.
<br>    

    
[Next -> Module 1](../modules/joining-aks-to-calico-cloud.md)
