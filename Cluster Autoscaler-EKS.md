🔹 What is Cluster Autoscaler?

Cluster Autoscaler (CA) is a Kubernetes component that automatically adjusts the number of nodes in your cluster.

If Pods cannot be scheduled (due to insufficient CPU/Memory), CA adds new nodes.

If nodes are underutilized (and their Pods can move elsewhere), CA removes them.


👉 Difference from HPA:

HPA scales Pods (application level).

CA scales Nodes (infrastructure level).

🔹 How I Implemented It in My Project

Cloud Provider Setup

We were running Kubernetes on AWS (EKS).

Installed Cluster Autoscaler using Helm chart (can also use official YAML from GitHub).

Deployed it into kube-system namespace.

IAM/Permissions (AWS case)

Created IAM role for Cluster Autoscaler to interact with Auto Scaling Groups.

Attached policy allowing CA to scale worker nodes.

Deployment
Example config snippet:

command:
  - ./cluster-autoscaler
  - --cloud-provider=aws
  - --nodes=2:10:my-node-group   # min:2, max:10 nodes
  - --scale-down-unneeded-time=10m


This means:

Minimum 2 nodes, maximum 10 nodes.

If a node is unused for 10 minutes, CA will remove it.

Testing

Deployed a workload that requested more CPU/Memory than current cluster could handle.

Verified that new nodes got added automatically.

🔹 Issues I Faced & Troubleshooting

Pods stuck in Pending even when CA was running

Cause: Sometimes Pod resource requests were too high for any node type available.

Fix: Adjusted Pod requests/limits OR added a larger node type to node group.

Cluster Autoscaler not scaling down

Cause: Some Pods had local storage or PodDisruptionBudgets (PDB) preventing eviction.

Fix: Checked kubectl describe node and logs of CA to see which Pods were blocking scale-down. Tuned PDBs.

CA Pod crashing

Cause: Missing IAM permissions in AWS.

Fix: Updated IAM role with correct policies (autoscaling:DescribeAutoScalingGroups, autoscaling:SetDesiredCapacity, etc.).

Slow scaling

Cause: Default scale-down-unneeded-time=10m was too high.

Fix: Tuned parameters like --scale-down-unneeded-time=3m and --scale-down-delay-after-add=5m.

🔹 Simple Interview Answer

“Cluster Autoscaler helps Kubernetes automatically add or remove nodes based on workload needs. I implemented it in AWS EKS by deploying the official Cluster Autoscaler in the kube-system namespace and connecting it with our node groups.

In practice, we faced issues like Pods stuck in Pending because their requests didn’t fit on any available node, and scale-down not happening due to PodDisruptionBudgets. To troubleshoot, I checked the Cluster Autoscaler logs, adjusted resource requests, tuned PDBs, and updated IAM roles when permissions were missing. Overall, it helped us save costs by scaling down unused nodes and ensured applications always had enough capacity.”
