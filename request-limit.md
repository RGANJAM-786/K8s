📝 Interview Answer – Requests & Limits in Kubernetes

In Kubernetes, Requests and Limits are used to control how much CPU and Memory a Pod can use. They help in resource allocation and fair usage across multiple applications.

🔹 Requests

A request is the minimum guaranteed amount of CPU/Memory a container will get.

Kubernetes scheduler uses this to decide on which node to place the Pod.

Example: If you request 500Mi memory, the scheduler will only place the Pod on a node that has at least 500Mi available.

🔹 Limits

A limit is the maximum amount of CPU/Memory a container can use.

If the container tries to use more than the CPU limit, it gets throttled (slowed down).

If it tries to use more than the Memory limit, it will be killed (OOMKilled).

🔹 Real-World Example

Imagine you are deploying a Java-based web application:

resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1"


👉 Explanation:

Requests (512Mi, 500m): The Pod is guaranteed half a CPU and 512Mi memory. Scheduler will only place it on nodes with this much free space.

Limits (1 CPU, 1Gi): The Pod can grow up to 1 CPU and 1Gi memory if available, but not more than that.

If the app tries to use 2Gi memory, it will be OOMKilled.

If it tries to use 2 CPUs, Kubernetes will throttle it to 1 CPU max.

🔹 Why Requests & Limits are Important?

Prevents one application from hogging all resources.

Ensures fair distribution among different workloads.

Helps Kubernetes schedule Pods intelligently.

Protects cluster from crashes due to memory leaks.

✅ Crisp 30-sec Interview Answer

“Requests and Limits in Kubernetes define how much CPU and memory a Pod is guaranteed to get and the maximum it can use. Requests are used by the scheduler to place Pods on suitable nodes, while limits enforce caps. If CPU exceeds limit, it gets throttled, and if memory exceeds limit, the Pod is killed. For example, if I set request=512Mi and limit=1Gi, the Pod always gets 512Mi but cannot use more than 1Gi. This ensures fair resource usage and cluster stability.”


🔥 Scenario-Based Questions on Requests & Limits
Q1.

You have a Pod without any requests or limits. What happens?

👉 Answer:

The Pod can be scheduled on any node, even if the node is heavily loaded.

It has no guaranteed resources, so if other Pods consume resources, this Pod might get starved.

If it consumes too much memory, it can crash other Pods → bad practice in production.

Q2.

Your Pod has:

requests:
  cpu: "500m"
  memory: "256Mi"
limits:
  cpu: "500m"
  memory: "256Mi"


What will happen if the application tries to use 600Mi memory?

👉 Answer:

Request = 256Mi (minimum guaranteed).

Limit = 256Mi (max allowed).

Since it tries to consume more than limit, Kubernetes will kill the Pod with OOMKilled error.

Q3.

Two Pods are scheduled on the same node.

Pod A → request.memory=1Gi, limit.memory=2Gi

Pod B → no requests, no limits.

If both try to consume 2 Gi each, who survives?

👉 Answer:

Pod A is guaranteed 1Gi (because of request).

Pod B has no guarantee → it may be starved or killed first.

Pod A might get killed only if it exceeds 2Gi (its limit).

Q4.

You set only limits (not requests). Example:

limits:
  cpu: "2"
  memory: "1Gi"


What does Kubernetes do?

👉 Answer:

Pod gets no guaranteed resources (scheduler won’t reserve anything).

But container cannot use more than limit.

Scheduler may put it on a crowded node → risk of performance issues.

Q5.

Why is it dangerous to set requests too high?

👉 Answer:

Example: request.cpu=4 in a cluster where most nodes have only 2 CPUs.

Pod will never get scheduled, because scheduler can’t find a node that satisfies request.

It leads to Pending Pods problem.

Q6.

Real-world: You run a monitoring agent (like Prometheus Node Exporter) on every node with request.cpu=50m and limit.cpu=100m. Why small values?

👉 Answer:

Monitoring agents need very little CPU/RAM.

If we don’t set small requests, they might block scheduling of bigger workloads.

If we don’t set limits, they might spike and eat too much CPU.

✅ Pro Interview Tip
Always say:

Requests → Guarantees.

Limits → Maximum cap.

If not set → Pod may starve OR hog resources.

Setting them wrongly → Pod may not schedule or may crash.
