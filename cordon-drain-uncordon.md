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


# Why DaemonSet pods are special
Node-specific deployments: DaemonSets are designed to run one pod on every node in the cluster (or a subset specified by a node selector). 


Critical services: They often manage essential services like node exporters, log collection agents, or storage daemons that are necessary for the node's functionality. 

Controller behavior: When a DaemonSet pod is terminated or the node is marked as unschedulable, the DaemonSet controller will automatically create a new pod for that node. 


# How kubectl drain interacts with DaemonSets

Default behavior: Without the --ignore-daemonsets flag, kubectl drain will stop because it detects DaemonSet pods that it cannot delete, as they would be immediately replaced. 


Successful draining: By specifying --ignore-daemonsets, you are acknowledging that these pods will be managed by their controller and not deleted by the drain command. The drain command then proceeds to cordon the node and evict other user-managed pods. 



In summary
Use the --ignore-daemonsets flag to allow kubectl drain to successfully cordon the node for maintenance, knowing that the DaemonSet controller will ensure the required system-level pods remain running on the node. 



# Scenario 1: Routine node maintenance

✅Question: What is the first command you would run to begin the maintenance on node-1? Explain what this command does.

✅ Answer: kubectl cordon node-1.

✅ Explanation: This command marks node-1 as unschedulable. This prevents the Kubernetes scheduler from placing any new pods on this node while allowing existing pods to continue running without interruption. It is the recommended first step for any node maintenance to ensure a graceful transition.


✅ Question: After cordoning the node, what command would you run to prepare node-1 for a safe shutdown? What happens to the DaemonSet pods during this process?

✅ Answer: kubectl drain node-1 --ignore-daemonsets.

✅ Explanation: This command safely evicts all regular pods from node-1 to other nodes in the cluster. It includes the --ignore-daemonsets flag to prevent issues with DaemonSet pods. DaemonSet pods are intentionally ignored because they are designed to run on every node and would be immediately replaced, causing the drain command to hang indefinitely.


✅Question: During the draining process, you see a message: "pod has a local storage volume, will not continue draining." What does this mean, and how would you handle it?


✅ Answer: This message indicates that a pod on the node is using a local volume, such as an emptyDir. Draining a node would lead to the loss of this data, so Kubernetes blocks the operation by default as a safety measure.
Handling: You would need to use the --delete-local-data flag to force the deletion of these pods and their data. This should only be done if the data is non-critical or has been backed up. kubectl drain node-1 --ignore-daemonsets --delete-local-data.


✅ Question: The drain command completes successfully, and you have rebooted node-1. What is the final step to make the node available for scheduling new pods again?

✅Answer: kubectl uncordon node-1.


✅Explanation: This command removes the unschedulable flag that was set by the cordon or drain command. After this, the Kubernetes scheduler can once again place new pods on node-1. 



# Scenario 2: Troubleshooting a failing drain operation

✅ Question: Why is the drain command failing when it encounters DaemonSet pods? What specific option should you use to fix this?

✅Answer: The drain command fails by default with DaemonSet pods because deleting them is pointless; the DaemonSet controller will simply recreate them immediately on the same node, leading to a deadlock.


✅ Solution: You should use the --ignore-daemonsets flag to tell the drain command to skip evicting these pods.


✅ Question: After adding the correct flag, the drain command finishes, but you see some pods still running on the node. You investigate and find these are singleton pods not managed by a ReplicaSet. What command do you need to use to forcefully remove them?


✅ Answer: To remove singleton pods not managed by a controller, you must use the --force flag. The command would be kubectl drain node-4 --ignore-daemonsets --force.


✅ Question: Explain the potential risks of using a command that forcefully removes pods. When is it acceptable to use it?


✅Answer:
Risks: Using --force can interrupt an application's graceful shutdown and potentially lead to data corruption for applications that rely on graceful termination. For StatefulSet pods, it is especially dangerous because it can violate the "at most one" semantic, potentially causing data loss.
When to use: It is acceptable for singleton pods without persistent data, as they will not be recreated. It can also be necessary for removing StatefulSet pods from a permanently failed node, but this must be done with extreme caution and a full understanding of the application's data consistency model. 


# Scenario 3: High-availability application with Pod Disruption Budget

✅ Question: You run kubectl drain node-5 --ignore-daemonsets. What impact might the PDB have on this command's execution?

✅ Answer: A PDB defines the minimum number of available replicas for an application. If draining the node would cause the number of available replicas to drop below this budget, the kubectl drain command will block and refuse to proceed. The drain command will respect the PDB and not evict the pod until enough replicas are available elsewhere.


✅ Question: The drain operation completes, and you begin your maintenance. After a few minutes, an application developer reports that the application is experiencing performance issues and is running with only four pods. Why did this happen, and what does it tell you about the PDB's configuration?


✅ Answer: The PDB was likely configured to allow only one replica to be unavailable (maxUnavailable: 1). When the drain command started, it safely evicted one pod. It then waited for the new pod to become ready on a different node before evicting the next one, but the maintenance was started while the application was still at its minimum replica count. The PDB is working as intended, but it highlights the need to complete the entire drain-and-uncordon cycle before moving to other maintenance tasks.


✅ Question: The PDB is configured correctly. How does a properly configured PDB prevent a full-scale outage while draining a node?


✅Answer: A properly configured PDB prevents a full-scale outage by ensuring that a minimum number of replicas remain available during a voluntary disruption like a drain. It forces the drain command to proceed slowly, one pod at a time (or according to the configured maxUnavailable), waiting for replacement pods to become healthy before evicting another. This controlled approach prevents the simultaneous loss of too many application pods, maintaining service availability. 


# Scenario 4: Isolating a problematic node

✅Question: What is the difference between running kubectl cordon node-6 versus immediately running kubectl drain node-6? In this scenario, which command would you start with and why?


✅ Answer:
kubectl cordon marks a node as unschedulable, but it does not remove any existing pods.
kubectl drain first cordons the node and then evicts all non-DaemonSet pods.
Which to use: You should start with kubectl cordon node-6 in this scenario. Cordoning the node prevents new pods from being scheduled to the unstable node, which immediately stops things from getting worse. It provides a period for investigation before you decide whether to evict all pods with a drain.


✅ Question: The drain operation is successful. You then run kubectl get pods -o wide and still see some pods from the kube-system namespace on node-6. Is this expected behavior? If so, why?

✅Answer: Yes, this is expected behavior. The kube-system pods still running are almost certainly DaemonSet pods (like kube-proxy, CNI plugin pods, or logging agents). You used the --ignore-daemonsets flag to prevent the drain from getting stuck, so these pods were intentionally not evicted.


✅Question: After your investigation, you decide that node-6 is no longer viable and needs to be permanently removed from the cluster. What steps would you take to complete this task?


✅Answer:
Ensure all maintenance work on the node is completed and the node is fully drained of all non-DaemonSet pods.
Run kubectl delete node node-6. This command removes the node's entry from the Kubernetes API, and the cluster's control plane will no longer attempt to manage it.
Delete the underlying virtual machine or cloud instance that hosts node-6. 


# Scenario 5: Automation and scripting

✅ Question: What is the logical order of the kubectl commands you would include in your script for a single node?


✅ Answer: The logical order is:

kubectl cordon <node-name>: Prevent new pods from being scheduled.


kubectl drain <node-name> --ignore-daemonsets --delete-local-data --force: Evict all existing pods gracefully, handling DaemonSets, local data, and singleton pods


Perform Maintenance Action (e.g., SSH and reboot).


kubectl uncordon <node-name>: Mark the node as schedulable again.



✅ Question: How would you handle potential failures in the drain operation (e.g., due to local storage pods)? Would you want the script to stop or continue?


✅Answer: You should configure the script to check the exit code of the drain command. If it returns an error, the script should stop and alert an operator. Forcing the drain with --delete-local-data or --force is a destructive operation that should be deliberately chosen by a human operator rather than a default script action, especially in a production environment.


✅ Question: To ensure the script is robust, how would you verify that all pods have been successfully evicted from the node before proceeding with the reboot?


✅ Answer: The kubectl drain command will return with a zero exit code only if it is successful. However, a more robust check would be to use a loop that polls for pods. For example, use a command like kubectl get pods -A -o wide --field-selector spec.nodeName=<node-name> and wait until the output contains only DaemonSet pods.


✅ Question: After the node is back online and uncordoned, how would you verify that new pods are being scheduled on it again?


✅Answer: You would run kubectl get pods -o wide --field-selector spec.nodeName=<node-name> and check for new application pods being created. Additionally, you could check the kubectl get nodes output to confirm that the SchedulingDisabled status has been removed.
