Make sure you have the Azure CLI installed on your local machine. If not, you can install it by following the instructions on the Azure CLI documentation website (https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).

Open the terminal or command prompt and log in to your Azure account by running the following command:


az login

Create a resource group to hold your ACR and AKS cluster by running the following command:

az group create --name sample-rg --location "west europe"


Create an ACR instance in your resource group by running the following command:

az acr create --resource-group sample-rg --name oursbankacr --sku Basic

Create an AKS cluster in your resource group by running the following command:

az aks create --resource-group sample-rg --name my-aks --node-count 3 --generate-ssh-keys --enable-managed-identity


Attach the ACR instance to the AKS cluster by running the following command:


az aks update --resource-group sample-rg --name my-aks --attach-acr oursbankacr


Verify that the ACR instance is attached to the AKS cluster by running the following command:



az aks show --resource-group sample-rg --name my-aks


This should display the details of the AKS cluster, including the ACR instance that is attached to it.





Import an image to ACR
------------------------

az acr import  -n oursbankacr --source docker.io/library/nginx:latest --image nginx:v1

get credentials
----------------

az aks get-credentials -g sample-rg -n my-aks


Pods
-------

In Kubernetes, a pod is the smallest deployable unit in the cluster. It is a logical host for one or more containers. A pod represents a running process on the cluster, and it is a logical host for one or more containers.

Here is an example of a pod configuration file that runs a single container:

Copy code
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: nginx:1.16
    ports:
    - containerPort: 80

This pod configuration will create a pod named "my-pod" with a single container running the nginx version 1.16 image. The container will expose port 80.

You can create this pod in your cluster by using the kubectl command-line tool:

Copy code
kubectl apply -f my-pod.yaml
This will create the pod in the cluster and start the container. You can then use the kubectl get pods command to see a list of all pods running in the cluster.


Replica Sets:
--------------

In Kubernetes, a ReplicaSet is a controller that ensures a specified number of replicas of a pod are running at any given time. If there are too few replicas running, the ReplicaSet will create new ones. If there are too many, it will delete excess replicas.

Here is an example ReplicaSet configuration that ensures there are always 3 replicas of a pod running:

Copy code
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:1.16
        ports:
        - containerPort: 80

This ReplicaSet configuration will create 3 replicas of the pod defined in the template field. The selector field specifies the label selector that the ReplicaSet will use to determine which pods belong to it.

You can create this ReplicaSet in your cluster by using the kubectl command-line tool:

Copy code
kubectl apply -f my-replicaset.yaml
This will create the ReplicaSet in the cluster and start 3 replicas of the pod. You can then use the kubectl get replicasets command to see a list of all ReplicaSets in the cluster.


Daemon Sets
------------

In Kubernetes, a DaemonSet is a controller that ensures a copy of a pod is running on every node in the cluster, or on a subset of nodes as specified by a label selector. This is useful for running pods that provide cluster-level services, such as log collection or monitoring.

Here is an example DaemonSet configuration that runs a single copy of a pod on each node in the cluster:


apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:1.16
        ports:
        - containerPort: 80
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule


This DaemonSet configuration will create a pod on each node in the cluster, using the pod template defined in the template field. The tolerations field specifies that the pod should not be scheduled on the master node, which is useful if you don't want to run the pod on the master node for some reason.

You can create this DaemonSet in your cluster by using the kubectl command-line tool:

kubectl apply -f my-daemonset.yaml
This will create the DaemonSet in the cluster and start a copy of the pod on each node. You can then use the kubectl get daemonsets command to see a list of all DaemonSets in the cluster.


Deployments
------------

In Kubernetes, a Deployment is a resource that manages a set of replicas of a pod. It provides declarative updates for the replicas, allowing you to specify the desired state of the replicas and the Deployment will ensure that the actual state matches the desired state.

Here is an example Deployment configuration that runs 3 replicas of a pod:

Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:1.16
        ports:
        - containerPort: 80

This Deployment configuration will create 3 replicas of the pod defined in the template field, using the label selector specified in the selector field.

You can create this Deployment in your cluster by using the kubectl command-line tool:

Copy code
kubectl apply -f my-deployment.yaml
This will create the Deployment in the cluster and start 3 replicas of the pod. You can then use the kubectl get deployments command to see a list of all Deployments in the cluster.

You can also use the kubectl rollout command to perform rolling updates on the Deployment. For example, you can use kubectl rollout restart to restart the replicas in the Deployment, or kubectl rollout undo to roll back to a previous revision of the Deployment.


Kubectl rollout examples:
--------------------------

Here is a list of common kubectl rollout commands that you can use to manage deployments in Kubernetes:

kubectl rollout status: Display the rollout status of a deployment.
kubectl rollout history: Display the history of a deployment.
kubectl rollout pause: Pause a deployment.
kubectl rollout resume: Resume a paused deployment.
kubectl rollout restart: Restart the pods in a deployment.
kubectl rollout undo: Roll back a deployment to a previous revision.
kubectl rollout cancel: Cancel a in-progress deployment.
kubectl rollout history revision: Display the details of a specific revision of a deployment.
Here is an example of using the kubectl rollout restart command to restart the pods in a deployment:


kubectl rollout restart deployment/my-deployment
This will perform a rolling restart of the pods in the my-deployment deployment. The pods will be restarted one at a time, so that there is always at least one available to serve traffic during the rollout.

You can use the --dry-run flag to perform a dry run of the rollout, which will show you the actions that would be taken without actually making any changes. For example:


kubectl rollout restart deployment/my-deployment --dry-run
This will show you the actions that would be taken to restart the my-deployment deployment, without actually making any changes.



Services
-----------


In Kubernetes, a Service is a resource that defines a set of pods and a policy for accessing them. It acts as an abstraction layer over a set of pods, allowing you to access the pods by a stable DNS name and IP address, regardless of which pods are actually running at any given time.

Here is an example Service configuration that exposes a Deployment as a Service:

Copy code
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort

This Service configuration will create a Service that exposes the pods with the app: my-app label as a Service. The Service will listen on port 80 and forward traffic to the target pods on port 80. The type field specifies that the Service should be of type NodePort, which means that it will be exposed on a static port on each node in the cluster.

You can create this Service in your cluster by using the kubectl command-line tool:

Copy code
kubectl apply -f my-service.yaml
This will create the Service in the cluster and expose it on a static port on each node in the cluster. You can then use the kubectl get services command to see a list of all Services in the cluster.

You can access the Service by its DNS name or IP address. The DNS name of the Service is of the form <service-name>.<namespace>.svc.cluster.local, and the IP address is an internal cluster IP address that is only accessible within the cluster.

Powershell Script to create AKS & ACR
-------------------------------------

Here is a PowerShell script that creates an Azure Kubernetes Service (AKS) cluster and an Azure Container Registry (ACR) instance, and then attaches the ACR instance to the AKS cluster:

Copy code
# Replace these values with your own resource names and resource group name
$aksName = "my-aks-cluster"
$acrName = "my-acr"
$resourceGroupName = "my-resource-group"

# Create the resource group
New-AzResourceGroup -Name $resourceGroupName -Location "eastus"

# Create the AKS cluster
$aksCluster = New-AzAksCluster -ResourceGroupName $resourceGroupName -Name $aksName -Location "eastus"

# Create the ACR instance
New-AzContainerRegistry -ResourceGroupName $resourceGroupName -Name $acrName -Location "eastus"

# Get the ID of the ACR instance
$acr = Get-AzContainerRegistry -ResourceGroupName $resourceGroupName -Name $acrName
$acrId = $acr.Id

# Get the AKS cluster resource ID
$aksClusterId = (Get-AzAksCluster -ResourceGroupName $resourceGroupName -Name $aksName).ResourceId

# Create a role assignment to give the AKS cluster access to the ACR instance
New-AzRoleAssignment -ObjectId $aksCluster.ServicePrincipalProfile.ClientId `
                     -RoleDefinitionName Reader `
                     -Scope $acrId

# Update the AKS cluster to use the ACR instance
$aksCluster.AddOnProfiles.KubeletConfiguration["acr"] = @{"registry"=$acrId}
Update-AzAksCluster -ResourceId $aksClusterId -AddOnProfiles $aksCluster.AddOnProfiles
This script will create an AKS cluster and an ACR instance in the eastus region, and then attach the ACR instance to the AKS cluster by creating a role assignment and updating the AKS cluster configuration.

You can then use the kubectl command-line tool to authenticate to the ACR instance from the AKS cluster and pull images from the registry. For example:

Copy code
# Get the ACR login server and password
$acrLoginServer = (Get-AzContainerRegistry -Name $acrName -ResourceGroupName $resourceGroupName).LoginServer
$acrPassword = (Get-AzContainerRegistryCredential -Name $acrName -ResourceGroupName $resourceGroupName).Passwords[0].Value

# Authenticate to the ACR instance
kubectl create secret docker-registry acr-auth --docker-server $acrLoginServer --docker-username $acrName --docker-password $acrPassword

# Use the ACR instance in a deployment
kubectl apply -f deployment.yaml
This will create a secret in the cluster that contains the credentials for the ACR instance, and then use the secret to authenticate to the ACR instance when pulling images for the deployment.

K8s imperative commands
-----------------------

Here are some of the imperative commands in Kubernetes:

kubectl create: create a new resource
kubectl apply: apply changes to a resource
kubectl delete: delete a resource
kubectl get: retrieve a resource
kubectl describe: get detailed information about a resource
kubectl edit: edit a resource in-place
kubectl exec: execute a command in a container
kubectl logs: view the logs of a container
kubectl port-forward: forward a local port to a port on a pod
kubectl cp: copy files to or from a container
kubectl scale: scale the number of replicas of a resource
kubectl rollout: manage the rollout of a resource
kubectl top: display resource usage for pods or nodes
kubectl cordon: mark a node as unschedulable
kubectl drain: drain a node in preparation for maintenance
kubectl uncordon: mark a node as schedulable
kubectl taint: add, modify, or remove a taint on a node



Different Objects Examples:
---------------------------

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: dev
    app: frontend
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
      - containerPort: 80
        name: nginx-port


---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: nodePort
  selector:
    env: dev
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

---

---

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc


---

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    run: nginx
  name: nginx-replicaset
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      name: nginx
      labels:
        run: nginx
    spec:
      containers:
      - image: laxmanae/nginx:v1
        name: nginx


---


apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        name: nginx

---

Scheduling Topics
----------------

Manual Scheduling with nodeName

---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
     name: nginx
  nodeName: aks-nodepool1-26732023-vmss000000

---

labels error below

---

apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: front-end
   template:
     metadata:
       labels:
        tier: nginx
     spec:
       containers:
       - name: nginx
         image: nginx


---

Taints and Tolerations

---

NoSchedule—The pod will not get scheduled to the node without a matching toleration.

NoExecute—This will immediately evict all the pods without the matching toleration from the node.

PerferNoSchedule—This is a softer version of NoSchedule where the controller will not try to schedule a pod with the tainted node. However, it is not a strict requirement.


command taint & untaint
-----------------------
kubectl taint nodes node01 gpu=true:NoSchedule

kubectl taint nodes minikube-m02 gpu:NoSchedule-



apiVersion: v1
kind: Pod
metadata:
  name: nginx-test
  labels:
    env: test-env
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "cpu"
    operator: "Equal"
    value: "high"
    effect: "NoSchedule"

conditions
----------


tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  value: "NoExecute"
  tolerationSeconds: 3600


Daemon Sets
-----------
apiVersion: apps/v1
kind: DaemonSet
metadata:
 name: fluentd-daemon
spec:
 selector:
   matchLabels:
 name: fluentd-daemon
 template:
   metadata:
 labels:
   name: fluentd-daemon
   spec:
 containers:
   - image: nginx
     name: fluentd-daemon



volumes
-----------

apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: my-app
    image: nginx
    ports:
    - containerPort: 8080
    imagePullPolicy: Always
    volumeMounts:
    - name: my-volume
      mountPath: /app
  volumes:
  - name: my-volume
    emptyDir: {} 

---
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: my-app
    image: nginx
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: my-volume
      mountPath: /app
  volumes:
  - name: my-volume
    hostPath:
      path: /mnt/vpath

---

---

Storage Class
---------

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium-retain
provisioner: disk.csi.azure.com
parameters:
  skuName: Premium_LRS
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true


apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile
provisioner: kubernetes.io/azure-file
parameters:
  skuName: Standard_LRS
  location: westeurope
  storageAccount: kubernetesstoragetest



apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium-retain
  resources:
    requests:
      storage: 5Gi



kind: Pod
apiVersion: v1
metadata:
  name: nginx
spec:
  containers:
    - name: myfrontend
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
      volumeMounts:
      - mountPath: "/mnt/azure"
        name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azure-managed-disk


apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  nodeSelector:
    kubernetes.io/hostname: kube-01
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
---


apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-configmap
data:
  sqlname: "testserver"
  sqlusername: "laxmana"
  sqldbname: "testdb"

---




















