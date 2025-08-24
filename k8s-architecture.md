Kubernetes architecture is broadly divided into two parts: the Control Plane (also called the Master) and the Worker Nodes. 

The Control Plane acts as the brain of the cluster, managing and scheduling workloads, while the Worker Nodes are responsible for running the actual application workloads.

1.control plane components : etcd , kube-sheduler , kube-controller-manager , kube api

The etcd stores all cluster metadata and configuration in key-value pairs, acting as the cluster’s database. 

The kube-scheduler decides on which node a pod should run based on resource availability,

The kube-controller-manager supervises and ensures that the cluster’s desired state matches the current state, automatically handling failures and recreations of pods or nodes when necessary. 

The kube-api server serves as the frontend of the Kubernetes control plane, receiving all user requests, whether from CLI commands, dashboards, or automation tools, and passes them for processing.

2. Worker Nodes – Where the actual applications run
Each worker node runs containers and has components to manage them:
•	Kubelet – Talks to the control plane and makes sure containers are running as instructed.
•	Kube-proxy – Handles networking so that containers and services can communicate within and outside the cluster.
•	Container Runtime – A Container Runtime is the software in Kubernetes that actually pulls the container image, creates the container, and runs it.



Pod Lifecycle
-------------

When I create a Pod, the API Server validates it and stores the details in etcd. The Scheduler then assigns the Pod to a suitable node, where the Kubelet works with the container runtime to start the container. Throughout its life, the Pod’s status is updated in etcd until it’s terminated."


You submit a Pod definition –
I prepare a YAML file with details like the container image, ports, and labels.
I send it to the Kubernetes API Server using kubectl apply -f pod.yaml.
Think of the API Server as the front desk of Kubernetes — everything goes through it.

API Server stores it in etcd –
The API Server validates my request and saves the Pod information in etcd, which is Kubernetes’ internal database that always keeps track of the cluster’s state.

Scheduler assigns it to a node –
The Scheduler looks at the Pod, checks resource availability on all nodes, and picks the most suitable one for it.
At this stage, the Pod still isn’t running — it just knows where it will run.

Kubelet starts the container –
On the chosen node, the Kubelet (an agent running on every node) sees the new Pod assigned to it.
It talks to the container runtime (like Docker or containerd) to pull the image and start the container.

Pod status is continuously updated –
As the Pod moves through its states — Pending → Running → Succeeded/Failed — the API Server updates this status in etcd.
This way, Kubernetes always knows exactly what’s happening with the Pod.
