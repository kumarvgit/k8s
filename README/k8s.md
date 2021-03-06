# Concepts
## cluster = master + nodes
## node = container or vm or physical computer
## All receives command via master
## minikube is to setup k8s in local development environments
## minikube creates vm and kubectl manages it
## node is a vm created by minikube
## each node communicates to outside world via kube proxy only and decides object it redirects to.

# terminologies
## apiVersion - v1 or apps/v1 gives us access to a set of kind (i.e. objects) which can be used
## k8s creates object - a type of container
## kind represents the type of object we want to make
### Pod - container
### Service - networking
#### ClusterIP
#### NodePort - expose a port to outside world - good for developer, with some exceptions since one of reason is it is always assigned between 30000 to 32767
#### LoadBalancer
#### Ingress


## pod is grouping of container with a common purpose i.e. must be running together as tightly coupled
## we can not run a container in k8s rather it runs inside a Pod
## "containerPort" the port which is exposed

## LabeledSelectorSystem - it uses the key value defined in metadata to gain access to e.g. Service pod will define a selector in spec and it will refer to medata data having key value defined as as label in metadata

#### Pod object
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  label:
    component: web <<<<<<< 1
spec:
  containers:
    - name: client
      image: kumarvdocker/multi-client
      ports:
        - containerPort: 3000

#### Service object
apiVersion: v1
kind: Service
metadata:
  name: client-node-ports
spec:
  type: NodePort
  ports:
    - ports:  3050
      targetPort: 3000
      nodePort: 31515
  selector:
    component: web --> This needs to be same as - 1 and it can be any thing like abc: def but at the same time label should also be the same

# Service
## targetPort it the exact port of selector i.e. container port
## port - this is for other service if the are trying to connect to the target port selector
## nodePort - ranging between always 30000 - 32767 - this port is the port via which we will connect to targetPort from outside world
### if nodePort is optional if it is not assigned then a random value between 30000 - 32767 is assigned by kube

# kubectl deployment
## change configuration of an object "kubectl apply -f <FILENAME>"
## get all kind of objects
### PODS - kubectl get pods
### SERVICES - kubectl get services
#### NAME         READY   STATUS    RESTARTS   AGE
#### client-pod   1/1     Running   0          63s
##### here 1/1 means 1 pod is running out of 1 pod expected
##### restart means if there has been any restart as k8s will do it automatically

## kubectl get services
#### workstation@workstation:~/Data/git/k8s/simplek8s$ kubectl get services
#### NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
#### client-node-ports   NodePort    10.106.209.187   <none>        3050:31515/TCP   5m28s
#### kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          126m
##### ports other objects connecting to it: nodePort (the one which is exposed)

## anything inside vm can not be accessed via localhost instead it is accessed by ip address since minikube is creating a vm and hence has its ip on its own
## get minikube ip - 'minikube ip'

## master is responsible for maintaining the number of pods specified and the program is "kube-apiserver"
## it always watch for how many are running and how many are expexted, if running is lessthan expected it is going to rerun it
## when we do apply kube-apiserver maintains a table of name and running container and expected number of pods
## each vm inside kube is running a copy of docker running  and when expected count of image is down it is going to reach out to docker hub (if image is not presetn in local image cache) and get a copy of image and start it.
## master uses polling mechanism and is alwasys running
## we interact with master but not to node via kubectl
## master is also running inside a vm, by default master takes control of in which vm/node the image gets deployed in otherwise we can do certain configuration and  specify the same also

# Minikube Setup on Linux

These instructions should be valid for Debian / Ubuntu / Mint Linux distributions. Your experience may vary if using an RHEL / Arch / Other distribution or non desktop distro like Ubuntu server, or lightweight distros which may omit many expected tools.
Install VirtualBox:

Find your Linux distribution and download the .deb package, using a graphical installer here should be sufficient. If you use a package manager like apt to install from your terminal, you will likely get a fairly out of date version.

https://www.virtualbox.org/wiki/Linux_Downloads

After installing, check your installation to make sure it worked:

VBoxManage —version


As an alternative you can use (or maybe you have to use) KVM instead of VirtualBox. Here are some great instructions that can be found in this post (Thanks to Nick L. for sharing):

https://computingforgeeks.com/install-kvm-centos-rhel-ubuntu-debian-sles-arch/


Install Kubectl

In your terminal run the following:

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl


Check your Installation:

kubectl version


See also official docs:
https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux


Install Minikube

In your terminal run the following:

curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube

sudo install minikube /usr/local/bin


Check your installation:

minikube version


Start Minikube:

minikube start


See also official docs:

https://kubernetes.io/docs/tasks/tools/install-minikube/

## the desired state is given by a yml file using "kubectl apply -f <File-Name>"


# imperitive vs declarative deployment
## declarative  is controlled by master, it is a guidance to master and master decides how to make it happen
## imperitive is saying exactly what needs to be done


#Updating a config
## declarative way
### when kubectl apply command is issued then master looks for a specific "name" and "kind" to decide whether to update or add a container any other change is fine
## get containers running inside of a pod - "kubectl describe <TYPE> <NAME-of-pod>"

## in pods we can not change the containerPort, if we chabge the containerPort and apply I ma going it get an error
## there is better approach to Pods it's called Deployment
## Pods (containers)
### Runs single set of closly related containers
### Good for development purpose
### Rarely used in production

## Deployment (creates identical pods)
### Runs a set of identical pods
### Deployment itself is going to monitor state of pods
### Good for dev and prod
### Deployment --> Contains pod template --> pod contains containers
### Deployment makes a requests k8s api-service to create pods for it
### matchLabels gets the handle of created pod by using the selector property
## why we use service: when a pod gets created it gets an IP address and that ip address is updated or maintained in environment and here Service makes use of selector to redirect the traffic to
## configured is something like created

# Applying a newer version of image
## delete pod so kubectl will recreate it - not a good solution as so many things can go wrong
## reference version in deployment file - adds an extra step
## imperitive command to be used
#### Steps involved
##### tag image with image version number
##### push the image to docker hub
##### tell kubectl to apply changes
###### kubectl set image <OBJECT>/<OBJECT-NAME>  <CONTAINER-NAME>=<IMAGE-NAME>

# Access docker server present inside of k8s
## configure docker-client installed of local system to talk to docker-server running inside of the k8s
## eval $(minikube docker-env) then run "docker ps" to access container inside of k8s
### above commands simply exports certain variables
#### export DOCKER_TLS_VERIFY="1"
#### export DOCKER_HOST="tcp://192.168.49.2:2376"
#### export DOCKER_CERT_PATH="/home/workstation/.minikube/certs"
#### export MINIKUBE_ACTIVE_DOCKERD="minikube"
#### 
#### # To point your shell to minikube's docker-daemon, run:
#### # eval $(minikube -p minikube docker-env)
## get access to logs directly from kube ctl
### kubectl get pods
### kubectl logs <POD-ID>
### kubectl exect -it <POD-ID> sh

# ClusterIP
## Exposes a port to other object in cluster but not to outside parworld unlike NodePort

# Persistence Volume Claim (or simply PVC)
## Volume: it is an object which exist at pod level it dies with the pod - this is a kubernates volume
## persistence volume: it is a volume created outside of pod so if a pod crashes then a new instance attaches itself to persistence volume and carries on with it.
## Persistence volume claim - This is a configuration which says there are various options to create a persistence volume in hard drive
## access modes PVC
### ReadWriteOnce - can be used by single node
### ReadOnlyMany - Multiple node can read from this
### ReadWriteMany - read and write by many nodes
## Persistence volumes are of two types:
### Statically provisioned - These are the ones which are readily available
### Dynamically provisioned - These are the ones which are dynamically provisioned on demand by the pod
## Get Storage volumes 'kubectl get storageclass'
## 'kubectl describe storageclass'
## if we do not specify the sorage class then default is going to be picked
## 'kubectl get pvc' get all pvc i.e. which are available for claim
## 'kubectl get pv' get all persistent volumes
## each in cluster connects to other service using 'ClusterIP' name in metadata
## env is injected as array of key and value pair inside of container

## adding secure environment variables and needs to be added in each environment e.g. for cloud
### done by Secrets object and using a imperitive command
### syntax "kubectl create secret <TYPE-OF-SECRET>  <SECRET-NAME> <SOURCE> <KEY=VALUE>"
#### TYPE-OF-SECRET
##### generic - general purpose
##### tls
##### docker-registry - when we need to pull images from custom registry instead of docker hub
#### SOURCE
##### can be a file
##### --from-literal - from command line
##### KEY=VALUE is an array i.e. we can put multiple key and value in same go and also this would add an extra step while accessing it via secretKeyRef in valueFrom in configuration file
### e.g. commandline "kubectl create secret generic pgpassword --from-literal PGPASSWORD=12345asdf"
## delete a secret : "kubectl delete secret pgpassword"

# Ingress Service
## LoadBalance - Legacy way to get traffic in cluster
### Load balancer exposes to one deployment kind (or one set of pods referred in deployment) i.e. if we have two service getting exposed to outside world it is going to be two load balancer
### There are different types of Ingress Service
#### e.g. Nginx Ingresss
## a controller is anything which helps in achieving a desired state of cluster e.g. deployments, since it specifically says i need x number of pods running
