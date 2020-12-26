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
