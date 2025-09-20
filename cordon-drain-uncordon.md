# Kubernetes Cordon and Drain

In Kubernetes, cordon and drain are mechanisms used to prepare a node for maintenance or removal without disrupting workloads.
These processes ensure that workloads are safely rescheduled to other nodes in the cluster.

🔹 What are Cordon, Uncordon, Drain?

kubectl cordon <node> → Marks the node as unschedulable.

→ No new pods will be scheduled, but existing pods keep running.


kubectl uncordon <node> → Removes the unschedulable mark.

→ Node becomes available again for scheduling.


kubectl drain <node> → Evicts all pods from the node safely.

→ Typically used before node maintenance (upgrade, scale down).
→ Respects PodDisruptionBudgets (PDBs), DaemonSets, and respects replication.

# 🔹 When I used these in projects (Real Scenarios)

# 1. Node Maintenance / Upgrade

We had to patch worker nodes with security updates (e.g., kernel or OS patches).

Steps:

kubectl cordon node1 → Stop scheduling new pods there.


kubectl drain node1 --ignore-daemonsets --delete-emptydir-data → Move workloads to other healthy nodes.


Perform node patch/reboot.


kubectl uncordon node1 → Bring it back to cluster for scheduling.

Benefit: Zero downtime, workloads automatically rescheduled by Kubernetes.

# 2. Cluster Autoscaling / Scaling Down

When AWS EKS cluster autoscaler wanted to remove underutilized nodes:

It first runs a drain on the node so pods get rescheduled.

Then the node gets terminated.

Benefit: Avoid pod loss during scale-down.

# 3. Node Troubleshooting

If a node was misbehaving (frequent pod crashes, networking issues), I cordoned it to prevent new pods landing on a bad node.

Later drained it and replaced the node.

Benefit: Prevents workloads being scheduled on unhealthy hardware/VM.

# 4. Special Deployments (Blue/Green, Canary)

Sometimes for testing, we cordon specific nodes so that critical workloads do not land there, allowing us to test with less risk.


# 🔹 Why it’s helpful

Prevents disruption during upgrades/patching.

Gives controlled rescheduling of workloads.

Ensures high availability — workloads are moved before shutting down nodes.

Works hand-in-hand with PDBs (PodDisruptionBudgets) to avoid taking down too many replicas.


# 🔹 Interview Style Statement

👉 “Yes, I worked with cordon, uncordon, and drain commands while handling Kubernetes node maintenance and upgrades.
For example, when upgrading worker nodes in EKS, I used cordon to stop new pods from scheduling, then drain to safely evict existing pods, and after maintenance I used uncordon to bring the node back.
This helped ensure zero downtime and smooth rolling upgrades. I also applied cordon when a node was unhealthy to prevent scheduling on it, and used drain during autoscaling and safe node removal.”


# kubectl drain node1 --ignore-daemonsets --delete-emptydir-data

# Why you need to ingonre daemonset while perfoming drain on node

You must ignore DaemonSet pods when draining a node because the DaemonSet controller will immediately reschedule them, even if the node is marked as unschedulable. Draining a node involves evicting all non-DaemonSet pods, so you use the --ignore-daemonsets flag with the kubectl drain command to tell Kubernetes to proceed despite the presence of these critical, node-specific pods.  
Why DaemonSet pods are special
Node-specific deployments: DaemonSets are designed to run one pod on every node in the cluster (or a subset specified by a node selector). 
Critical services: They often manage essential services like node exporters, log collection agents, or storage daemons that are necessary for the node's functionality. 
Controller behavior: When a DaemonSet pod is terminated or the node is marked as unschedulable, the DaemonSet controller will automatically create a new pod for that node. 
How kubectl drain interacts with DaemonSets
Default behavior: Without the --ignore-daemonsets flag, kubectl drain will stop because it detects DaemonSet pods that it cannot delete, as they would be immediately replaced. 
Successful draining: By specifying --ignore-daemonsets, you are acknowledging that these pods will be managed by their controller and not deleted by the drain command. The drain command then proceeds to cordon the node and evict other user-managed pods. 
In summary
Use the --ignore-daemonsets flag to allow kubectl drain to successfully cordon the node for maintenance, knowing that the DaemonSet controller will ensure the required system-level pods remain running on the node. 
