# Kubernetes!!!

- Mainly leverages Docker engine and Docker store
- Open source platform designed to automate, deploy, scale, and operate application containers

#### Multi-host container scheduling
- Done with the kube-scheduler
- Assigns pods to nodes at runtime
- Kubernetes master can be deployed in a highly available environment
	- Multi region deployments available

#### Scalability
- Supports 5,000 node clusters
- 150,000 total pods
- Max of 100 pods per node
- Pods can be horizontally scaled via API

#### Registration
- Nodes register themselves with master seamlessly

#### Service discovery
- Includes service discovery out of the box
- Automatic detection of services and endpoints via DNS or environment variables

#### Persistent Storage
- Pods can use persistent storage volumes to store data across restarts and crashes

#### Maintenance
- Features are backwards compatible 
- APIs are versioned

#### Logging and Monitoring
- Application monitoring built in
	- TCP, HTTP or container execution health check
- Node health check
	- Failures monitored by node controller
- Kubernetes status
	- Add-ons: Heapster and cAdvisor

#### Secret management
- Sensitive data is a first class citizen
- Mounted as data volumes or environment variables 
- Specific to namespace (are not shared across all applications)

# Architecture
![[Pasted image 20220719114925.png]]

#### Master Node
Responsible for management of Kubernetes cluster
- The API Server
	- Allows for interaction with Kubernetes API
- Scheduler
	- Watches created pods that don't have a design yet and designs it to run on a specific node
- Controller Manager
	- Runs controllers that run tasks in a cluster

#### etcd
- Simple distributed key value store
- Kubernetes uses this as its database and stores its cluster data here

#### kubectl
- Contains kubeconfig
	- Container server information and authentication information to access API

#### Worker Nodes
- Communication to worker nodes is handled by `kubelet` process
- Agent that communicates with the API server to see if pods have been designed
- Executes pod containers
- Mounts and runs pod volumes and secrets
- Aware of pod and node states and replies to master

#### kube-proxy
- Network proxy and load balancer on the node
- Handles routing for TCP and UDP and performs connection forwarding

#### Docker Pod
- Structures containers in pods
- Pods are the smallest unit for deployment in Kubernetes
- Pods share storage, Linux name space, and IP addresses among other things
	- Share resources that are always scheduled together
- Kubelet checks on the pods for status and health
- Kubeproxy handles routing to the pods and load balancing
- Has unique network IP


## Labels
Labels are key/value pairs that are attacked to objects like pods, services, deployments. Labels are for users of Kubernetes to identify attributes for objects.

Example:
`"release": "stable", "release": "canary"`
`"environment": "dev", "environment": "qa", "environment": "production"`
`"tier": "frontend", "tier": "backend", "tier": "cache"`

### Selectors
1. Equality based
	1. `=` : Two labels or values should be equal
	2. `!=` : Two values should not be equal
2. Set-based
	1. `IN` - A value should be inside a set of defined values
	2. `NOTIN` - A value should not be in a set of defined values
	3. `EXISTS` - Determines whether a label exists or not

Usually used by kubectl 

### Namespaces
- Great for enterprises
- Allows teams to access resources with accountability
- Great way to divide resources between users
- Provides scope for names - must be unique in the namespace
- `Default`  namespace created when you launch Kubernetes
- Objects are placed in `default` namespace at the start
- Newer applications install their resources in a different namespace
	- To try and not interfere with your infrastructure

### Kubelet
- Communicates with API server to see if pods were assigned to the nodes
- Executes pod containers via container engine
- Mounts and runs pod volumes and secrets
- Executes health checks to identify pod/node status

- Podspec: YAML file that describes the pod
- kubelet takes the podspecs that are defined and provides it to the kube-apiserver and ensures that the containers described in the podspecs are running and healthy
- kubelet only manages containers that were made through the API server - not any contianer on the node

### Kube-proxy
- Process that runs on all worker nodes
- Reflects services as defined on each node, and can do simple network streams or round-robin forwarding across a set of backends
- Service cluster IPs and ports are currently found through Docker --link compatible environment vatiables specifying ports opened by the service proxy

kube-proxy has 3 modes
1. User space mode
2. IPTables mode
3. Ipvs mode (alpha feature)

These modes are important because:
- Services defined against hte API server: kube-proxy watches the API server for additional services
- For each new service, kube-proxy opens a randomly chosen port on the local node
- Connections made to the chosen port are proxied to one of the corresponding back-end pods
