---
layout: default
---
## Architecture
![图片pic1]({{ "/assets/images/arch.png" | absolute_url }}) 
 
Core layer: the core function of Kubernetes, providing API to build high-level applications externally, and providing plug-in application execution environment internally  
Application layer: deployment (stateless applications, stateful applications, batch tasks, cluster applications, etc.) and routing (service discovery, DNS resolution, etc.), Service Mesh (partially located in the application layer)  

Management layer: system metrics (such as infrastructure, container and network metrics), automation (such as automatic scaling, dynamic Provision, etc.) and policy management (RBAC, Quota, PSP, NetworkPolicy, etc.), Service Mesh (partially located in the management layer)  

Interface layer: kubectl command line tool, client SDK and cluster federation  

Ecosystem: The ecosystem of large container cluster management and scheduling above the interface layer can be divided into two categories  

Outside of Kubernetes: logs, monitoring, configuration management, CI/CD, Workflow, FaaS, OTS applications, ChatOps, GitOps, SecOps, etc. 
 
Inside Kubernetes: CRI, CNI, CSI, mirror warehouse, Cloud Provider, configuration and management of the cluster itself, etc.
## API
For cloud computing systems, the system API is actually in the dominance of system design. As mentioned earlier in this article, every time a new function is supported by the Kubernetes cluster system, a new technology will be introduced, and the corresponding API object will be newly introduced to support the Function management operation.  
1.All APIs should be declarative  
  2.API objects are complementary and composable  
  3.High-level APIs are designed based on operational intent  
  4.The low-level API is designed according to the control needs of the high-level API    
  5.Try to avoid simple encapsulation and do not have internal hiding mechanisms that are not explicitly known by external APIs  
  6.API operation complexity is proportional to the number of objects  
  7.API object state cannot depend on network connection state  
  8.Try to avoid making the operation mechanism dependent on the global state, because it is very difficult to ensure the synchronization of the global state in a distributed system  
## Etcd
Etcd is a very important component in the Kubernetes cluster, which is used to save all the network configuration and object status information of the cluster.

Etcd uses the Raft consensus algorithm to achieve, is a distributed consistent KV storage, mainly used for shared configuration and service discovery
## Interface

![图片pic1]({{ "/assets/images/interface.png" | absolute_url }}) 

#### CRI
container runtime interface, providing computing resources

The interface between the container and the image service is defined in CRI. Since the container runtime and the image life cycle are isolated from each other, two services need to be defined.
#### CNI
Container network interface, providing network resources

CNI consists of a set of specifications and libraries for configuring the network interface of Linux containers, and also includes some plug-ins. CNI only cares about network allocation when the container is created, and releases network resources when the container is deleted.
#### CSI
Container storage interface, providing storage resources

CSI attempts to establish an industry-standard interface specification. With the help of the CSI Container Orchestration System (CO), any storage system can be exposed to its own container workload.

# Pod
Pod represents the process running in the cluster  
Pods encapsulate application containers (in some cases, several containers), storage, independent network IP, and policy options for managing how containers run. Pod represents a unit of deployment: an instance of an application in kubernetes, which may be composed of one or more containers to share resources.
![图片pic1]({{ "/assets/images/pod.png" | absolute_url }})   
Pod can share two kinds of resources: network and storage   
Network  
Each Pod will be assigned a unique IP address. All containers in the Pod share network space, including IP addresses and ports. Pod containers can use localhost to communicate with each other. When the container in the Pod communicates with the outside world, it must allocate shared network resources (for example, use the host's port mapping).  
Storage  
You can specify multiple shared volumes for a Pod. All containers in the Pod can access the shared volume. Volume can also be used to persist storage resources in the Pod to prevent file loss after the container restarts.
#### Init container
Init containers are very similar to ordinary containers, except for the following two points:

The Init container always runs until it completes successfully.
Each Init container must complete successfully before the next Init container starts.  
They can contain and run utilities, but for security reasons, it is not recommended to include these utilities in the application container image.  
They can include installation using tools and customized code, but they cannot appear in the application image. For example, it is not necessary to create another image from FROM, just use tools like sed, awk, python, or dig during the installation process.  
Application images can separate the roles of creation and deployment without the need to join them to build a single image.  
The Init container uses Linux Namespace, so it has a different view of the file system than the application container. Therefore, they can have access to Secret, but the application container cannot.  
They must be run before the application container is started, and the application containers are run in parallel, so the Init container can provide a simple way to block or delay the start of the application container until a set of prerequisites is met.  

#### Pause container
Pause container, also known as Infra container.  
For example, there is now a Pod, which contains a container A and a container B, the two of them will share the Network Namespace. The solution in Kubernetes is this: it will create a small Infra container in each Pod to share the Network Namespace of the entire Pod.  
Infra container is a very small image, about 700KB, is a container written in C language and always in a "suspended" state. With such an Infra container, all other containers will be added to the Network Namespace of the Infra container through Join Namespace.

#### Pod life cycle
![图片pic1]({{ "/assets/images/life.png" | absolute_url }})   

Pending: The Pod has been accepted by the Kubernetes system, but one or more container images have not yet been created. The waiting time includes the time to schedule the Pod and the time to download the image via the network, which may take some time.  
Running: The Pod has been bound to a node, and all the containers in the Pod have been created. At least one container is running, or is in a startup or restart state.  
Successful (Succeeded): All containers in the Pod are successfully terminated and will not be restarted.  
Failed: All containers in the Pod have been terminated, and at least one container was terminated due to failure. That is, the container exits with a non-zero status or is terminated by the system.  
Unknown: Unable to obtain the status of the Pod for some reason, usually because the communication with the host where the Pod is located fails.  


[back](./)