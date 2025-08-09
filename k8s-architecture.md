Kubernetes architecture is broadly divided into two parts: the Control Plane (also called the Master) and the Worker Nodes. 
The Control Plane acts as the brain of the cluster, managing and scheduling workloads, while the Worker Nodes are responsible for running the actual application workloads.
The etcd stores all cluster metadata and configuration in key-value pairs, acting as the cluster’s database. 
The kube-scheduler decides on which node a workload (pod) should run based on resource availability,
 while the kube-controller-manager supervises and ensures that the cluster’s desired state matches the current state, automatically handling failures and recreations of pods or nodes when necessary. 
The kube-api server serves as the frontend of the Kubernetes control plane, receiving all user requests, whether from CLI commands, dashboards, or automation tools, and passes them for processing.
2. Worker Nodes – Where the actual applications run
Each worker node runs containers and has components to manage them:
•	Kubelet – Talks to the control plane and makes sure containers are running as instructed.
•	Kube-proxy – Handles networking so that containers and services can communicate within and outside the cluster.
•	Container Runtime – A Container Runtime is the software in Kubernetes that actually pulls the container image, creates the container, and runs it.
