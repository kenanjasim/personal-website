+++
title = 'Certified Kubernetes Administrator Notes'
date = "2024-01-16"
author = "Kenan Jasim"
categories = ["notes"]
tags = ["kubernetes", "notes"]
+++


A couple of years ago now, I did my Certified Kubernetes Administrator exam, I created a set of notes containing everything I needed to know for the exam. I have decided to share these notes in the hope that they will help others.

# Section 1: Core Concepts

## Cluster Components

Kubernetes allows you to think of any number of servers as a single virtual machine. It abstracts the complexities of the infrastructure and how it connects. It hosts your apps in containers. Consists of nodes which host containers. Worker nodes will host the apps as containers and master nodes will manage plan and schedule nodes (control plane)

## Control Plane Components

### etcd 

Distributed key-value database which kubernetes uses to store it’s objects. Cluster forms with an elected leader and the followers Before any value is written to the cluster, they agree on that value then persist it, this is known as **consensus**. They coordinate using the RAFT protocol

### API Server
The Kubernetes API exposes an API endpoint which allows users and cluster components to communicate with each other. It lets you query and manipulate the state of API objects in etcd, ensuring data is valid when retrieved or placed in the database. The API handles authentication through RBAC.

### Controller Manager
The controller manager contains all the control loops Kubernetes needs to function. A controller is a control loop that watches the shared state of the cluster through the apiserver and makes changes attempting to move the current state towards the desired state. For example you have the as the endpoint controller, service account controller, replication controller, node controller

### Scheduler
A scheduler watches for newly created pds which are not yet scheduled to a node. It then becomes responsible 
for finding the best node to schedule that pod on. The default scheduler filters nodes according to scheduling requirements. The scheduler is then left with feasible nodes to schedule the pods onto. The scheduler then assigns a score to the feasible nodes based on scheduling policies and picks the one with the highest ranking 
* **Policies** allow you to configure predicates for filtering and priorities for scoring
* **Profiles** allow you to configure plugins that implement different stages such as: QueueSort, Filter, Score, Bind, Reserve, Permit

## Node Components

### kubelet
Primary node agent which runs on each node. It will register the node with the API server and is responsible for taking in PodSpecs (through the API server, static pod, http endpoint or http requests) and ensuring that the containers defined in those PodSpecs are healthy and running by communicating with the container runtime.

### Kube Proxy
Network proxy which implements part of the services concept. They maintain network roles on the nodes which allow network communication to and from Pods. It utilises the OS packet filtering layer if there is one (iptables) or it forwards the packets themself.

### Container Runtime
Software which is responsible for running containers. This can be containerd, `CRI-O` or `Docker`.

## Pods 	 

Pods are the smallest addressable units for kubernetes. It is a group of one or more containers which shares storage and network resources and specs on how the containers should be run.

To create pods via the kubernetes client, run:

```bash
kubectl run [name] --image={image} # run directly
kubectl run [name] --image={image} -o=yaml --dry-run=client # Get YAML definition
```

You can edit a Pod’s image using the `kubectl edit` command. Pods can take time to delete. To delete a pod quickly then run:

```bash
kubectl delete pod [name] --grace-period=0 --force
```

## Replica Sets

A ReplicaSet’s purpose is to maintain a stable set of replica Pods. It’s often used to guarantee the availability of a specified number of identical pods. They have a selector which defines which pods it will acquire. ReplicaSets create and delete Pods as needed to reach the specified number of replicas.  They create these pods using the specified Pod specification. 

Replica Sets are linked to Pods via the `metadata.OwnerReferences` field. This field specifies what resource the Pod is owned by. It will acquire any pods that match the selector and whose OwnerReferences are empty.

Any change that is made to a ReplicaSet only takes effect after the pods have been deleted.

## Deployments

A Kubernetes Deployment provides  declarative updates for Pods and ReplicaSets. The Deployment Controller changes the actual state to the desired state at a controlled rate. 

### Create a Deployment
To create deployments via the kubernetes client, run:

```bash
kubectl create deployment --image=nginx --replicas=3 nginx
```

## Namespaces  	 

A Namespace provides a mechanism for isolating groups of resources in a single cluster. They allow you to separate resources between teams/projects
To create a namespace via the kubernetes client, run:

```bash
kubectl create namespace [name]
```

To access a resource from another namespace use: `[resource-name].[namespace].[type].cluster.local`

## Services

In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them. The set of Pods targeted by a Service is usually determined by a selector.

### Cluster IP Service

A ClusterIP Service assigns an IP address the serviceCIDR, an address range that is different from nodes and Pods. Exists only for load balancing purposes and for the routing rules that are written to disk by iptables. Packets match the rule for the service and are rewritten to go where they need to.

```bash
kubectl expose [resource] --port=80 --target-port=8080
```

### NodePort Service
Similar to the ClusterIP except it has an additional port in the range of of 30000 and 32767 that used to reach the Service from outside the cluster. The port is exposed from all nodes. 

```bash
kubectl expose [resource] --type=NodePort --port=80  #Random Nodeport
```

### LoadBalancer Service

Builds on top of the node port service, has an IP and a port like Cluster IP and an external port like NodePort but when you request a service of type LoadBalancer a real load balancer is automatically attached to the port. Every cloud provider attaches its own load balancer. 

### Service Discovery

When you create a service an entry is added to the DNS entries in the cluster. There’s a component in the control plane of your Kubernetes cluster whose only job is to watch for new and old Service and add their entry to the DNS.

* Entries follow `name-of-the-service.the-namespace.svc.cluster.local`
* If the service is in the same namespace you can drop most of that and just use the name. If not you could use `name-of-the-service.the-namespace`

 
# Section 2: Scheduling   		 

## Manual Scheduling   	
Setting the `spec.nodeName` will bind the Pod to the selected Node. If this can't happen then the scheduler will look for the best node to schedule a pod in.

## Labels and Selectors 	

You can decorate pods with key value pairs, these can be used for a multitude of reasons. They can be used for filtering pod output in the kubectl command with the `--selector` flag.  Adding a selector to the services, deployments and Replica sets  will allow it to point to a logical set of pods with the same label.

## Taints and Tolerations 	
Taints allow a node to repel a set of pods. Tollerations applied to pods allow (but not require) them to schedule on that node. They ensure that pods cannot be scheduled on inappropriate nodes.
The Taint effect decides what happens to pods which do not tolerate the taint:
* `NoSchedule` means that pods should not be scheduled on the node
* `PreferNoSchedule` means the scheduler should try to avoid scheduling on that node, but can if it has to
* `NoExecute` means new pods won’t be scheduled and existing pods should be evicted if they do not tolerate the taint.

To taint a node run:

```bash
kubectl taint nodes <node name> key1=value1: taint-effect
kubectl taint nodes node1 key1=value1:NoSchedule    # Apply a taint
kubectl taint nodes node1 key1=value1:NoSchedule-   # Remove a taint
```

A Tolleration can be added to a pod by placing the key, value and taint effect in the `spec.tolerations` section of the Pod yaml file.

## Affinity and Anti-Affinity

Affinities are a more capable version of node selectors. They allow more complex operations such as Exists, NotIn and NotExists and allows you to use anti-affinities which define where a pod should not be scheduled.

There are two types of affinity available: 
* `requiredDuringSchedulingIgnoredDuringExecution` - The affinity must be adhered to or the pod can't be scheduled
* `preferedDuringSchedulingIgnoredDuringExecution` - The affinity is preferred but the pod may be scheduled elsewhere if it can't be met.

For **nodes** you must first label the node. This can be done imperatively using the `kubectl label node` command. From there you are able to alter the spec.affinity field of a Pod template. 

Like nodes, **Pods** also have affinities, they work in the same ways as node affinities and are used to tell the scheduler to schedule pods based on other Pods’ labels.

Pods can have anti-affinities, which makes sure a Pod is not scheduled where other pods match the labels written. This can be used for instance to only have one pod of the same type run on each node

## Resource Limits 

The Scheduler takes into account the resources of a pod before scheduling. If no node can take a pod then the pod is pending with a relevant error. You can limit these. If you go over CPU it will try to throttle, if over RAM then it will be terminated. CPU goes as low and 1m. 1 CPU is 1 vCPU.

Kubernetes assumes that each pod has a resource request of 0.5cpu, 256 mB of RAM. These can be changed in the definition file.  If the kubelet has to eject pods, it assumes that any pod without requests has gone over their quota and ejects them first

## DaemonSets

Similar to ReplicaSets but runs one copy of a pod on each node. Even when a new node is added or deleted. Used for monitoring solutions and log viewers. `kube-proxy` is put as a daemonset.

## Static Pods
The kubelet of each node is able to function on its own.  You can configure it to read Pod definition files from a static file path, by default this is `/etc/kubernetes/manifests` (this can be changed in kubelet service file). 

Each Node’s kubelet will check this directory periodically and make any missing Pods, modify existing Pods, delete Pods whose manifests are no longer there and ensure all existing manifest pods are still running. 

The Kubernetes API considers these pods part of the cluster and will create a mirror object so it can still be seen by the user

## Multiple Schedulers
Kubernetes is highly extensible. You are able to deploy your own scheduler as either the default scheduler or alongside the default scheduler. You have to create a `ConfigMap` which contains a config file as specified to  to differentiate it from the `kube-scheduler` which takes the name `default-scheduler`. 

The `--leader-elect` flag is used to choose a leader to lead scheduling activities, this is used when there are multiple copies of the scheduler running on multiple nodes (high availability). You should set this as false on the custom scheduler or set lock object will set the name to differentiate it during election

# Section 3: Logging and Monitoring 

## Monitor Cluster Components

It is often necessary to know Node, Resource and Pod metrics. Kubernetes doesn’t natively support this and so you need to deploy one yourself. The metric server (replacing heapster) can be deployed to provide metric data to the API. One metric server per cluster. It receives from each Pod and puts it in memory. The kubelet contains a container Advisor (cAdvisor) this will take the performance metrics from the container and expose it via the API

To deploy the metric server, clone from GitHub and deploy using `kubectl apply -f`  then you can get the info by running:
```bash
kubectl top [resource] [name]
kubectl top nodes      # Get node metrics
kubectl top pod nginx  # Get specific pod metrics
```

## Application Logs
You can collect the logs of a container simply. You can either go through the container runtime, for example `crictl logs` for docker containers. Or you could go through the kubernetes api and run `kubectl logs`. If the pod contains more than one container then you must specify the container name as well as the pod name

# Section 4: Application Lifecycle Monitoring 

## Rolling Updates and Rollbacks 
You are able to perform rolling upgrades and rollbacks on Deployments, ReplicaSets and Daemonsets. This allows a simple mechanism to change images of a deployment without causing any downtime. 

The replica set of the initial image gets set, incrementally, to 0 while the other replica set spawns its pods. The pods are created and health checked while the old ones are being deleted. It ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 75% up and that only a certain number of Pods are created above the desired number of Pods. By default 125% 

You can change the image with 
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 #Perform Update
kubectl rollout status deployment/nginx-deployment # See Rollout Status
```

Deployments can also have the `spec.strategy` type  Recreate which, unlike RollingUpdate, will bring all the old pods down before starting new ones. 

### Rollback
You are able to rollback any rolling updates which have been done. By default, all of the Deployment's rollout history is kept in the system so that you can rollback anytime you want
  
```bash
kubectl rollout history deployment/nginx-deployment # Get rollout history
kubectl rollout history deployment/nginx-deployment --revision=2 # Specific revision
kubectl rollout undo deployment/nginx-deployment # Rollback to previous
kubectl rollout undo deployment/nginx-deployment --to-revision=2 # Specific rollback
```

## Commands and Arguments

You are able to define commands and arguments to pass into a kubernetes pod. The spec.containers.command field is written as a list starting with the entrypoint (binary to run) and any arguments passed to it. You can split the arguments into their own separate field spec.containers.args or keep them placed in the command list. 

The command field in kubernetes overwrites the command which may be specified in the container image. To run a command in shell, you can use:

```yaml
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]
```

## Environment Variables

You are able to pass environment variables into a kubernetes pod. These environment variables can be read by the individual containers. You can pass environment variables from:

* **name-value pairs** - These come in the from of name value pairs
* **ConfigMaps** - Values from config maps can be passed as environment variables
* **Secrets** - Decoded values from secrets can also be passed into pods via environment variables

You can use the whole secret or configMap in the environment by using envFrom in place of env and referring to the secret or ConfigMap.

## ConfigMaps 
ConfigMaps are used to store non-confidential data in key-value pairs. Pods are able to use config maps in environment variables, command line args or in the form of configuration files.
You can create a config map using kubectl: 

```bash
kubectl create configmap <name> --from-literal=<key>=<value>
kubectl create configmap <name> --from-file=filename
kubectl create configmap <name> --from-env-file=filename
```

## Secrets
Secrets are used to store sensitive data such as passwords, tokens or keys. Secrets much like ConfigMaps can be used as files, environment variables or are used by kubelet when pulling images from private registries.

A secret is only sent to a node if a pod on that node requires it. The kubelet stores the secret into a tmpfs so that the secret is not written to disk storage. Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well. 

You can create a secret using kubectl: 

```bash
echo -n 'value' | base64 (--decode for decoding)
kubectl create secret generic [#SAME AS CONFIG MAP]
Kubectl create secret [type] [options]
```

## Multi Container Pods
Pods are designed to support multiple containers which form a cohesive unit. The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster. 

The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.

## Init Containers
At times you may want to run a process that runs to completion in a container. For example a process that pulls code or binary from a repository that will be used by the main application.Or a process that waits  for an external service or database to be up before the actual application starts.

You can configure multiple initContainers, in that case each init container is run one at a time in sequential order.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.


# Section 5: Cluster Maintenance   		 

## OS Upgrades     	 
When upgrading the operating system of a node, the node will be inaccessible for a while. Because of this the pods running on that node must be scheduled on other nodes in the cluster. 
* You can remove the pods from a node so that the workloads will be evenly distributed across  the other remaining nodes: `kubectl drain <node name>`
* Then the node will become unschedulable. Once the update has finished and the node is back online, the node can be marked as schedulable by running `kubectl uncordon <node name>`
* To simply make the node unschedulable, without evicting pods,  run `kubectl cordon <node name>`

You can only delete a pod which is managed by a ReplicaSet. Any pod which has been scheduled outside of this and is forcefully drained will be lost forever.

## Cluster Upgrade Process
Components can be at any version the same or lower than the release version of the API. `kubectl` can be one above the same or below. 

The `kubeadm` binary has a way of upgrading, you can run `kubeadm upgrade` plan to plan the upgrade versions and `kubeadm upgrade` apply to apply that upgrade. You can also upgrade each component on its own

You should upgrade master first then worker node.  Use apt to update `kubeadm` first upgrade the node then upgrade `kubelet` to the right version

## Backup and Restore Methods   	
Backing up the resources, etcd database and persistent volumes is an important task. It is essential that, if something goes wrong, the cluster can be salvaged. A primitive way to upgrade is to collect all the resources into one YAML file and store this somewhere

```bash
kubectl get all --all-namespaces -o yaml > all-services.yaml
```

### etcd Backup

The etcd database is separately able to be backed up. A snapshot of the etcd database can be taken and stored for when it is needed. When it comes time to restore the ectd database, a change to the etcd pod volume mount can be made to point to the data-dir provided to the ectd backup command. 

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/snapshot-pre-boot.db 
```

You must pass the endpoint, cacert, cert and key flags into the ectd command so that it is able to properly communicate with the etcd database. You can then restore from this snapshot using the `snapshot restore` command and setting the `data-dir` flag to specify the directory to store the etcd data 

# Section 6: Security

## Security Primitives
Kubernetes uses users,  certificates, external authentication with LDAP and service accounts for access to the API server. Once gaining access to the API server, what a user can do is determined by their role.

All Kubernetes components are  secured by TLS certificates. Network policies are used to secure communication between hosts.

### Certificate Details
It may be important to view the details of a certificate which has been generated either by the user or by the cluster provisioning tool. You are able to decode a certificate by running:

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

## Certificate API  

### Users
Kubernetes has two classes of users: normal users and service accounts. Kubernetes does not have objects which represent users and assumes there is some cluster independent way of storing users (keys, keystores or plain text).

Any user which presents a valid certificate signed by the CA is considered authenticated. Kubernetes determines the username from the CN field (CN=ted).

### Certificates
Kubernetes heavily uses certificate authentication. Most components use their own certificate pairs for authentication.

### Client Generation
To generate client certificate and keys, you must create a certificate signing request. Adding group user `SYSTEM:MASTERS` for admin privileges.

```bash
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=ted /O=system:masters" -out admin.csr
```

### Certificate API
Kubernetes allows you to sign the Certificate signing requests created above to add the users to the cluster. You need to first create a CertificateSigningRequest object. NOTE: the csr must be base64 encoded.

```bash
cat file.csr | base64 -w 0
apiVersion: certificates.k8s.io/v1beta1 kind: CertificateSigningRequest
metadata:
 name: jane
spec:
 groups:
 - system:authenticated
 usages:
 - digital signature
 - key encipherment
 - server auth
 request:
  {BASE 64 ENCODING GOES HERE}
```

You can approve the request by running:

```bash
kubectl certificate approve {csr}
```

You then can collect the certificate by running:

```bash
kubectl get csr {name} -o yaml
```

then decoding the certificate with `echo {certificate} | base64 —decode`

The signing is handled by the `kube-controller-manager`.

## Kubeconfig
You are able to specify the server, client-key and certificate and ca as flags. Instead you are able to move these configurations into a file called a kubeconfig. The default file can be found at `~/.kube/config` or you can specify with the kubeconfig flag

The KubeConfig contains 3 sections: 
1. **Clusters** - Information regarding the cluster you wish to access such as name, certificate authority and api server URL
2. **Users** - The user which can access the cluster, this often contains the name, client cert and key
3. **Contexts** - Combination of user and certificate. You can switch contexts using kubectl so you don't have to keep multiple KubeConfigs. You can specify a default context by adding a current context field.

```bash
kubectl config view --kubeconfig={Specify} # View a kubeconfig
kubectl config use-context {context} # Switch Context
```

## Role Based Access Control
Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organisation.

The RBAC API declares four kinds of Kubernetes objects: Role, ClusterRole, RoleBinding and ClusterRoleBinding.

### Roles
Roles are namespaced objects which represent a set of additive permissions. You can create a role by:

```bash
kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods
```

### Role Binding
A role binding grants the permissions from a role to a user or set of users. It holds a list of users, groups, or service accounts, and a reference to the role being granted. You can create a role binding by:

```bash
kubectl create rolebinding admin --role=admin --user=user1 --user=user2 --group=group1 --service-account=svact
```

### Cluster Roles
Cluster roles are roles which apply on the cluster level rather than on a namespace level.  You can create a cluster role by running:
  
```bash
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods
```

### Cluster Role Binding
Cluster role bindings are the same as role bindings, but they bind cluster role permissions to a user or set of users:

```bash
kubectl create clusterrolebinding cluster-admin --clusterrole=cluster-admin --user=user1 --user=user2 --group=group1
```

### Check Access
To check if you have permissions to do something then you can run:

```bash
kubectl auth can-i <command>
```

As an admin you can impersonate a user you can use the same command with the `--as <user>` flag.

### Service Accounts
A service account provides an identity for pods to authenticate with the API server. Assigning roles to service accounts allow you to manage the access pods have to the API.

```bash
kubectl create serviceaccount NAME
```

## Image Security   	

All images are by default pulled from DockerHub. You can provide the name of the repository in the same way as pulling from docker (username/image:version). For Kubernetes if the name of the user is the same as the app you can just provide the name, for example just nginx rather than nginx/nginx. You can also specify another registry in the form registry.io/path/to/app

### Private Repo
To use an image from a private repository. You have to provide the full path to the private repo. You pass the credentials into a kubernetes secret object of type docker-registry. In the pod definition file, you must specify the secret name in the `spec.imagePullSecrets`.

## Security Contexts   	
A security context defines privilege and access control settings for a Pod or Container. You are able to specify security in the container level and pod level. If both are specified then it will take the container security. The security context settings include:
* Linux Capabilities
* UserID and GroupID
* Privilege Escalation

## Network Policy

NetworkPolicies are an application-centric construct which allow you to specify  specify a set of ingress and egress rules defining how a pod is allowed to communicate with various network "entities". The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers: Pods, Namespaces and IP blocks

There are two sorts of isolation for a pod: isolation for egress, and isolation for ingress. By default, a pod is non-isolated for egress and ingress. They are enforced by the type of network solution that is used, flannel does not support it but Kube-router, Calico, Romana, Wave-net do. No error, it just ignores them if they fail.

# Section 7: Storage 

## Pre-Requisites

### Docker Storage
Docker stores all its data in `var/lib/docker`. Files in the image layer are read only and the files in the container layer are read and write. If you want to edit a file from the image layer it is copied to the container layer. This is called copy on write

You can mount a folder from the host to a docker container and point a folder in that container to it or a volume from the var/lib/docker/volumes folder. Docker’s default **volume driver** plugin is local and it helps to mount a volume to the docker host. Can specify the driver (such as AUFS, ZFS) using `--volume-driver`

### Container Storage Interface
Universal standard for storage, defines a set of remote procedure calls which can be called by the orchestrator. When a volume is needed it should run a create volume call and pass all the information that it needed (specified by the driver).

## Persistent Volumes 

Normally when a pod is created, any files which change within it are deleted when the pod is deleted. You are able to attach a volume to help persist the data, this could be a hostPath pointing to a directory or file on the host machine. This can be difficult as that folder has to exist on the node where the pod is running.

**Persistent volumes** provide a large pool of data and allows users to make claims on part of the data that they want for their pods. You can use any storage solution (same as normal volumes) including cloud storage.

Kubernetes binds a persistent volume to a **PersistentVolumeClaim** based on sufficient capacity, such as access mode, volume mode and storage class. 
* You can specify a label for the volume and selectors for the claims.
* The claim must have the same access modes 
  * **ReadWriteOnce** means the volume can be mounted as read-write by a single node. 
  * **ReadOnlyMany** means the volume can be mounted as read-only by many nodes.
  * **ReadWriteMany** means the volume can be mounted as read-write by many nodes.
  * **ReadWriteOncePod** means the volume can be mounted as read-write by a single Pod. 
* You can get a big volume being used up by a smaller one as it is one to one mapping so no other pod can use it.
* When you delete a PVC the volume is either **deleted**, **recycled** (data deleted before being made available again) or **retained** (default). You can set it by setting the persistentVolumeReclaimPolicy.

You can then use the persistent volume inside of a pod, mounting it in a similar fashion to a normal Volume.

## Storage Classes
Allows dynamic provisioning in a cloud for storage. These classes can be used in volumes and persistent volumes instead of HostPath for instance. Binding can be immediate(default) or can be `WaitForFirstConsumer` (This will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created.)

# Section 8: Networking	 

## Networking Pre-Requisites 

### Switching
Switching is the act of sending packets between interfaces on a network switch. The Linux software equivalent of a switch is a bridge. Packets are not by default sent between interfaces in Linux, you can change this by enabling IP forwarding

### Routing
Routing refers to how network packets are routed so they can reach their destination. For example if the packet is destined to a machine on the local network, then the packet can be easily sent to the local machine. However if the IP is unrecognised, the IP should be sent  to the default gateway (often a router). The router consults the routing table to forward the packet to the relevant machine

### Domain Name System
Domain Name System (DNS) allows domain names (www.google.com) to be resolved into IP addresses (8.8.8.8). The DNS server will resolve the domain name to the correct IP of google.

On Linux you are able to do basic DNS resolution using the `/etc/hosts` file. It uses name resolution to substitute the name specified with the relevant IP address. This does cause issues as this is only local to the machine. It does not copy over to other machines on the network

A DNS server effectively stores a hosts file on a server, and every time you wish to resolve a name your machine will check the server and get the IP in return. In Linux you can set the DNS server in the `/etc/resolv.conf` file.

You can add multiple DNS Nameservers in the resolv file and this will allow a local server which contains resolutions in the local network to be queried first then a public DNS server such as google to be queried next which contains the public internet. The etc hosts file takes precedence, change this in `/etc/switch.conf`

#### Domain Names
Domain names are separated by dots. The top level is the right most part of the name such as com and org are specifying the intent of the website. The middle name is the name of the site. subdomains such as www and maps and blogs help further divide stuff. You can have caches in DNS.

#### DNS Records
There are multiple record types in a DNS server. These define what is being stored by the DNS server.
* A name records map names to IPv4 IP addresses
* AAAA records map names to IPv6 addresses
* CNAME records maps one name to another

### Namespaces
Used to implement network isolation generally in container runtimes, The network namespace isolates the arp table, routing and interfaces in the container from the hosts. Containers can see its own processes which it has created but no processes created by or on the host. This is the process namespace.

#### Useful Commands

```bash
ip netns add [name] # Create Network Namespace
ip netns exec [name] [command] # Execute in the namespace

ip link add br0 type bridge # Create bridge
ip link add veth-1 type veth peer name veth-2 # Create veth pair
ip link set veth-1 netns ns1 # Assign veth pair end to namespace
ip link set veth-2 master br0 # Assign other end to bridge
ip a add # Add IP address
ip link set veth-1 up # Bring Veth Pair Up
iptables -t nat -A POSTROUTING -s [IP OF VETH NETWORK] -j MASQURADE # Enable access to outside world (must add default route to container
iptables -t nat -A PREROUTING --dport [port on host] --to-destination [port to access] -j DNAT # Allow public access
```  

### Docker Networking
Docker networking is slightly different from other container networking. When you run a docker container you can specify one of:
* `--network none` -  no interfaces, no access to outside or other network
* `--network host` -  bound to the host networking
* `--network bridge` - internal private network (172.17.0.1/24) bound to a bridge (like the network namespaces)

Docker creates a bridge network in docker and a docker0 interface. They have a similar network namespace technique as before. IP is assigned as 172.16.0.1/24. docker inspect allows you to find the namespace which was created. It creates an veth pair between the host and the container mastered by the docker0 bridge

You can map ports on docker using the `-p` flag which adds an iptables rule to the host linking from the containers open port to the host port which was specified.

### Container Network Interfaces
rkt, mesos, docker and other container runtimes are all solving the same issue in very similar ways. CNI is a standard that describes how plugins should be developed and how CRIs should invoke them.
* It aims to provide a plugin orientated networking solution for containers.
* It consists of specifications and libraries for writing networking plugins for Linux containers.
* It only deals with network connectivity and garbage collection of resources when containers are deleted.
There are two central definitions, namely a container and a network.
* **Container** - Synonymous with a network namespace, single addressable unit
* **Network** - Uniquely addressable group of entities that can communicate with one and another.

CRIs are responsible for: creating the network namespace, identifying the network the container must attach to, invoking the CNI plugin with the ADD to add a container and DEL to delete the container and provide a JSON format of the network configuration

The network plugin should support command line arguments (ADD,DEL,CHECK), support parameters, container id, network ns etc, Must manage IP address assignment to PODs.  Must return results in specific format. CNI comes with default plugins: bridge, vlan, ipvlan, macvlan, windows, dhcp and host-local plugins

**Calico** - Network plugin for CNI, which manages a flat layer 3 network, assigning each workload a fully relatable IP address. For any overlay requirements it can implement IP to IP tunnelling or can use flannel.

#### Docker
Docker does not support CNI plugins, instead opting for their own slightly different CNM standards. To use CNI plugins on a docker container you need to create the container which the none network type and invoke the CNI plugin directly

### Cluster Networking
A kubernetes network should be a flat network that spans several nodes.
* **Containers can talk to any other container in the network**, and there’s no translation of addresses in the process — i.e. no NAT is involved
* **Nodes in the cluster can talk to any other container in the network and vice-versa**. Even in this case, there’s no translation of addresses — i.e. no NAT
* **A container’s IP address is always the same**, independently if seen from another container or itself.
* **Each node has separate IPs and MAC addresses**. 

## CNI In Kubernetes

### Pod Networking
Kubernetes has no built-in solution for pod networking. Kubernetes however expects any network plugin ensure that  each pod should:
* have a unique IP address,
* be able to communicate with every other pod on the same node,
* be able to communicate with every other pod on other nodes without NAT.
This can be achieved manually:
* Create a bridge on each host (add IP address and bring up interface)
* Attach the network namespaces of containers to a bridge on each host (via veth pairs). 
* Bring the interfaces up and add addresses.
* Add routing to the hosts to ensure multi nodes can communicate. or add to a router.

This is effectively a CNI ADD command. Kubelet is configured to look at the cni-config-path (`/etc/cni/net.d/` by default) when creating pods, which has been configured as a flag, it will find a config file here telling it which CNI plugin to run (if multiple, use the first in alphabetical order). Kubelet will look for the binary in the cni-bin-dir flag (`/opt/cni/bin` by default) and run it with the add command providing the container id and namespace as arguments. 

### IP Address Management (IPAM)
It is the responsibility of the CNI plugin to assign the IP addresses to the pods. Kubernetes doesn't care how it is done  as long as Pods have IPs and that they are unique. 

CNI comes with two built-in plugins (host local or dhcp) to help with this. The type which is used is defined in the cni config file. 
### Weaveworks Plugin
Once you have multiple kubernetes nodes, the manual addition of bridges and veth pairs simply will not scale. WeaveWorks is one of many CNI plugins which can be used to solve the networking issue. 

Weaveworks deploys an agent on each node, The agents communicate with each other and store a topology of the cluster so they know the IPs and amount of nodes, pods etc. WeaveWorks creates their own bridge and assigns their own IPs to them, it makes sure that the pods get the right route and can communicate with other pods.

Weave intercepts packets going to a different node and it encapsulates the packet before sending it to the other node, the agent on the other node can then collect the packet, decapsulate it and route it to the right place. 

Weave can be deployed as pods or as a service. If you already have a cluster setup then you can use a simple kubectl command and it will deploy all the necessary components including the agents which are deployed as a Daemonset.

## Service Networking
Pods very rarely communicate with each-other, often applications communicate with each other with the use of services. Services are not bound to a single node, they are available on every node meaning that a pod on any node can successfully communicate with a service.  

Pods are able to access services either via the IP address of the service or the name of the service. Services are a virtual object, the IP is not bound to any interface and only used as a reference for the rules.

Each node runs an application called kube-proxy, which watches for changes from the kube-api server. When a new service is created it is responsible for assigning an ip from a predefined range (defined in the api server, shouldnt overlap with pod range) and is responsible for setting the rules which route the service’s IP:port to the pod ip.

The `kube-proxy` can use userspace (listens and routes itself), `iptables` or `ipvs` rules and this is set by the `--proxy-mode` flag (iptables by default). To view the rules created by the kube-proxy run:

```bash
iptables -L -t nat | grep 'name-of-service'
```

## DNS and CoreDNS

### DNS in Kubernetes
Kubernetes deploys a built-in DNS cluster if using `kubeadm`. When a service is created, the DNS server will map the name of the service to the ip address of the service, meaning any pod in the cluster can now access the service by name (if the same namespace) or by name.namespace (if in a different namespace). 

Services are grouped together as sub domains of the namespace  then grouped together further in a subdomain defining their type (svc) and then further grouped together into a root domain for the whole cluster (by `default cluster.local`).

Records for pods are not created by default, this can be enabled in the DNS config file. The name of the dns record is the IP of the pod but with the dots replaced with dashes. The name follows the service convention, with type set to pod rather than svc.

### CoreDNS
Post 1.12 a CoreDNS server is deployed as a Deployment (2 replicas) on the Kubernetes cluster. Kubernetes defines a config file for coredns in a config map mounted to the `/etc/coredns/Corefile` file of the pod. This file defines plugins for the DNS server. The kubernetes plugin allows it to work with the kubernetes cluster. 

CoreDNS is exposed using a service, the IP of which is written to the pod’s `/etc/resolv.conf` file by the kubelet, the file also has a search entry allowing for services to be looked up by just their names/name.namespace.

## Ingres Networking 
Normally when you deploy an application into kubernetes you tend to deploy the deployment and some services like ClusterIP or Nodeport. You can even use load balancers in cloud environments.

This is difficult to scale because of the proxy servers needed to ensure that all routes still work. That is where ingress comes in.

Ingress deploys a supported controller and resources, A controler effectively acts as an intelligent proxy server. routing traffic to services. You can create a controller with nginx for example. The Ingress resources can then be used. 

The default backend is where things get routed by default. You can set the http path to do for example /api. or you can use the hosts which allows different domains or subdomains. If you don't specify a host it assumes *

Ingress needs to be in the same namespace as the service, can create multiple ingress resources spanning namespaces:

```bash
kubectl create ingress <ingress-name> --rule="host/path=service:port"
```

# Section 9: Kubernetes Cluster Installation and Design

## Design a Kubernetes Cluster
There are many different tools to deploy Kubernetes clusters for multiple use cases, for example:
* Minikube or Kubeadm for testing/learning purposes 
* Kubeadm can be used for development purposes. 
* For production you need Highly available with multiple master nodes with multiple worker nodes also. This can be done using Kubeadm or cloud providers offerings.

For on prem, the kubeadm tool is very useful, in addition cloud providers have relevant tools should you need to use a public cloud. You can use physical or virtual machines. A minimum of 4 nodes for production is preferred. The best practice is to have a dedicated master node. Etcd can be separated from the master node also and placed on its own server for better redundancy.

### Choosing Infrastructure
Kubernetes can be deployed in many ways. You can go from binaries or you can use a tool. Kubeadm is only supported on linux. Minikube (provisions VMs) can be used for a pain free experience while kubeadm(requires VMs provisions) allows more versatility
* **Turnkey Tools** - Provision VMs yourself and use a tool to deploy Kubernetes (e.g OpenShift, Cloud Foundry Container Runtime and Vagrant)
* **Hosted Tools** - Kubernetes as a service (e.g Google Container Engine (GKE), EKS (Amazon))
  
### Highly Available Kubernetes Cluster
When you lose a master the workers can run on their own but you have no access to the cluster through the api server etc.

High availability provides redundancies for all the components in your cluster, by duplicating components across nodes, so that if one node goes down the cluster wont go down (master, workers etc)

The api-server works on one request at a time, so multiple api server instances can be working at the same time. Should use a load balancer for the master nodes so you can point the kubectl to the IP of the load balancer and have the traffic balanced across the nodes.

The controller manager and scheduler, run in parallel so they may duplicate actions. They run in an active standby method. They elect a leader, on startup, they  try to gain a lock by updating their info in an endpoint file. The first one to do it gets the lease. This is checked as defined by flags.

### ETCD in High Availability
Etcd can be stacked in the master (Easier to manage but if one node goes down then loses a member). The etcd server can be separated from the master nodes (additional server). API server needs to point to both the instances.

When data is written or read, etcd makes sure that the data is consistent on all instances. ectd does not process the writes on all nodes. They elect a leader and the elected leader is responsible for  processing the writes and ensures that the data is distributed to other nodes.

A write is complete if it can be written on the majority of nodes. If a node is offline when write is done then as long as the etcd cluster achieves quorum (N/2 + 1) then the write can be complete. If a node comes back online then the write is propagated to it.

ectd uses RAFT protocol. RAFT uses a random timer to initiate requests from ectd members to become a leader. Once a node is the leader they periodically send info saying that they are still assuming the role. If a node goes down another node assumes the role of the leader.

Even numbers of etcd members means that the cluster is more likely to fail if there is network segmentation so odd numbers of etcd members have better chances to stay alive. 5 over 6 etc

## Instal using Kubeadm
Kubeadm is a tool provided by kubernetes to help set up a kubernetes cluster following the best practices.
* You must first provision the machines needed for the cluster you are building (single node, 1 master 2 workers etc)
* Install  a Container Runtime Interface (docker, containerd etc)
* Install the kubeadm, kubelet and kubectl binaries on all nodes.
* Initialise the master node kubeadm init <args>
* Ensure network prerequisites are met (cni plugin)
* Join worker nodes using the join command specified by the kubeadm tool

# Section 10: Troubleshooting   		 
## Application Failure  
Check the application is accessible on the IP of the application, check service and endpoints which it has grabbed. Check if the pod is running and if it has crashed and restarted. Check the logs of the pod to see why it is failing. Check any dependant services such as databases or apis.

## Control Plane Falure   	
Check status of the nodes and pods on the cluster. You should also check the controlplane pods or services on the master nodes. Check the logs of the control plane components

## Worker Node Failure
Check the status of the nodes of the pods. Check the description if the node is failed. If the statuses are unknown then it may have crashed. You need to check the status of the nodes. Check CPU, memory and disk space. Check kubelet and kubelet certs.

## Troubleshoot Network
Check the CNI binary directory and the network plugin that is being used. You can also check if DNS resolves by deploying a test pod and execing into it. Check kube proxy and its logs if service rules are not being applied

# Section 11: JSONPATH Queries		
JSONpath is useful where there are 100s of nodes and 1000s of pods you need to query and you will be using kubectl for these objects. JSON path is supported to help with filtering the queries.

You must first get the path to the field by running kubectl with the output set as json. Then you can form the query `-o=jsonpath '{.items[0].spec.containers[0].image}’` (You can use wildcard instead of the number in a list to list all the objects). You can also insert a new line with `{"\n"}` and a tab with `{"\t"}`

You can use loops and ranges:

```JSON
{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}
```

You can use the `--sort-by` flag in kubectl and pass in the jsonpath and this will sort by the item specified. You can also add custom columns to kubectl by using the json path of the fields you want:

```bash
kubectl get nodes -o=custom-columns=NODE:.metadata.name, CPU:.status.capacity.cpu
```




