ReplicationController is one of the important features in Kubernetes that manages the lifecycle of Pods. Its main responsibility is to make sure that the desired number of Pod replicas are always running.

It helps you easily create multiple Pods and ensures that this number is always maintained.

If a Pod crashes or is deleted, the ReplicationController automatically replaces it with a new one.

ReplicationControllers work closely with labels to identify and manage Pods.

Even if you set the replica count to 1, it guarantees that at least one Pod of your application is always available.

✅ Super Crisp Version (30–40 sec for interviews)

“A ReplicationController in Kubernetes ensures that the desired number of Pods are always running. For example, if I set replicas to 3, it will create 3 Pods and if one fails, it immediately replaces it. It uses labels to identify which Pods to manage. Even with just 1 replica, it makes sure at least one Pod is always running, which ensures high availability. Today, ReplicaSets and Deployments are more commonly used, but the concept started with ReplicationController.”

✅ReplicationController: 
ReplicationController was the first way in Kubernetes to ensure Pods are always running, but it only supported equality-based selectors. ReplicaSet replaced it by adding set-based selectors. In practice, we don’t usually use either RC or RS directly — instead, we use Deployments, which manage ReplicaSets internally and provide powerful features like rolling updates, rollbacks, and easy scaling. So, in production, Deployments are the standard.”


✅ ReplicaSet:

“A ReplicaSet in Kubernetes makes sure the desired number of Pods are always running, similar to ReplicationController. The key difference is that ReplicaSet supports set-based label selectors, which makes it more flexible. For example, I can tell it to manage Pods with labels app in (frontend, backend). In modern Kubernetes, we usually don’t create ReplicaSets directly — instead, Deployments manage ReplicaSets for us and add features like rolling updates and rollbacks.”


✅ Deployment in Kubernetes:

A Deployment is the standard way to run and manage applications in Kubernetes. It manages ReplicaSets, which in turn manage Pods. The advantage of a Deployment is that it supports rolling updates, rollbacks, and scaling. For example, if I want to update my app to a new version, the Deployment gradually replaces old Pods with new ones to avoid downtime. If something fails, it can rollback automatically. That’s why Deployments are widely used in production instead of using ReplicaSets 
directly.”


👉
Let’s say a new version of your application is available and you need to deploy it to your Kubernetes cluster. In this case, would you use a ReplicaSet or a Deployment, and why?”

👉 Answer:
“I would prefer using a Deployment instead of directly using a ReplicaSet.

A ReplicaSet only ensures that a certain number of Pods are running, but it doesn’t manage version updates.

A Deployment is built on top of ReplicaSets and provides rollout functionality. That means I can deploy a new version of my application safely with features like rolling updates, rollbacks, and version history.

For example, if a new version of my app (v2) is available, I just update the Deployment YAML with the new image. Kubernetes creates a new ReplicaSet for v2, gradually replaces Pods from the old ReplicaSet (v1) with the new ones, and ensures zero downtime. If something goes wrong, I can roll back easily.

So in real-world projects, we almost always use Deployments, because they give us version control, safe rollouts, and rollback options — things that ReplicaSets alone cannot do.”

✅ Interview Tip:
If the interviewer asks you, “But ReplicaSets can also keep Pods running, so why not use them?”, you can confidently say:
“ReplicaSets are mainly used internally by Deployments. We rarely use them directly in production because they don’t support version rollouts or rollbacks


👉
“Suppose after deploying a new version, the application functionality is not working as expected. How can you roll back to the previous version, and will this process cause any downtime?”

👉 Answer:
“If the new Deployment version doesn’t work as expected, I can roll back to the previous version using Kubernetes’ rollout feature.

I would run:

kubectl rollout undo deployment <deployment-name> -n <namespace>


This command reverts the Deployment back to the last working ReplicaSet (previous version of the Pods).

About downtime:

If my Deployment is configured with a RollingUpdate strategy (which is the default), Kubernetes will gradually replace the bad Pods with the old version, so there’s no downtime.

If I had used a Recreate strategy, then all Pods of the new version would terminate before the old ones come back, which would cause downtime.

So in real-world setups, we prefer RollingUpdate so that rollback is also smooth and users don’t see outages.”

✅ Interview Tip:
If asked “How does Kubernetes remember the old version?” → You can say:
“Kubernetes Deployments keep a history of ReplicaSets. Each time we update the Deployment, it creates a new ReplicaSet. During rollback, Kubernetes simply switches back to the earlier ReplicaSet.”

Perfect 👍 Rolling updates & rollbacks are hot interview topics because they directly connect to real-world production deployments. Let’s create scenario-based interview questions with detailed, easy-to-understand answers.


🔹 Rolling Updates


Q1. You have a Deployment running version v1 of your app. Now you want to upgrade to version v2 without downtime. How will you do it in Kubernetes?

👉 Answer:

In Kubernetes, Deployments support rolling updates.

I would update the Deployment image:

# kubectl set image deployment myapp myapp=app:v2


Kubernetes will create a new ReplicaSet for v2 Pods.

It will gradually replace old v1 Pods with new v2 Pods (one by one or based on maxUnavailable / maxSurge values).

During the process, users can still access the app, so there’s no downtime.


Q2. During a rolling update, what happens if half the new Pods fail to start?

👉 Answer:

Kubernetes rollout will pause automatically if new Pods are not becoming healthy.

The old Pods will still be running and serving traffic.

I can check status with:

# kubectl rollout status deployment myapp

I can either fix the issue (like wrong image/config) and continue rollout, or perform a rollback to restore the previous version.


Q3. How can you control the speed of a rolling update?

👉 Answer:

By using maxUnavailable and maxSurge in the Deployment spec.

Example:

strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 2


This means: at most 1 Pod can be unavailable during the rollout, and 2 extra Pods can be created temporarily.

These values let me fine-tune rollout speed vs. stability depending on traffic load.


Q4. If you set strategy type to Recreate, how is it different from rolling update?

👉 Answer:

With Recreate: all old Pods are deleted first, then new Pods are created. → This causes downtime.

With RollingUpdate: old Pods are replaced gradually while keeping service available → zero downtime.

In production, we almost always use RollingUpdate.


Q5. Can a rolling update cause downtime if misconfigured?

👉 Answer:

Yes ✅. Example:

If maxUnavailable=100%, then Kubernetes may delete all Pods before creating new ones → downtime.

If readiness/liveness probes are not configured correctly, traffic may go to half-ready Pods, causing failures.

So, correct probe configuration and rolling strategy tuning are critical.


🔹 Rollbacks

Q6. You performed a rolling update, but the new version has bugs. How can you roll back quickly?

👉 Answer:

Kubernetes stores revision history of Deployments.

To roll back to the previous stable version, I just run:

# kubectl rollout undo deployment myapp


If I want to roll back to a specific version:

kubectl rollout undo deployment myapp --to-revision=2


The rollback itself is also a rolling process → no downtime.


Q7. What happens if you roll back a Deployment but the rollback also fails?

👉 Answer:

If rollback Pods fail, Kubernetes will again pause the rollout.

The system will not delete the old stable Pods until the new Pods are ready.

So, traffic keeps flowing to the last working version.

As a DevOps engineer, I’d troubleshoot using kubectl describe pod and kubectl logs.


Q8. Can you prevent rollbacks from happening automatically?

👉 Answer:

Yes. By setting:

spec:
  revisionHistoryLimit: 0


This tells Kubernetes not to keep old ReplicaSets.

But in practice, we usually keep a history (default is 10 revisions), so we can rollback if needed.


Q9. If you delete the old ReplicaSet after a rollout, can you still roll back?

👉 Answer:

No ❌. Rollback requires the old ReplicaSet.

If you delete it, Kubernetes has no version to go back to.

That’s why we usually keep some history of ReplicaSets (using revisionHistoryLimit).



Q10. Suppose rollback was successful, but you don’t want such bugs happening again. What measures would you take?

👉 Answer:

Add proper readiness and liveness probes so that broken Pods don’t get traffic.

Use staging/testing environments before production rollouts.

Enable canary deployments or blue-green deployments for safer upgrades.

Automate monitoring alerts to detect issues quickly.



❓ Why use Deployment when we already have ReplicaSet?

“ReplicaSet ensures Pods are always running, but it doesn’t support rolling updates or rollbacks. Deployment builds on top of ReplicaSet and adds these advanced features. In production, we always use Deployments instead of directly using ReplicaSets, because Deployments provide version control, gradual rollouts, rollbacks, and easier management of application lifecycle. ReplicaSet is mostly used internally by Deployments.”


🚀 Types of Kubernetes Deployments

1️⃣ Recreate Deployment

How it works: K8s deletes all old Pods first, then creates new ones.

Downtime: Yes (because there’s a gap between old Pods shutting down and new Pods starting).

When to use:

When downtime is acceptable.

When your app cannot handle multiple versions running at the same time.

Example:

strategy:
  type: Recreate


⚠️ Drawback → Your app will be offline during rollout.


2️⃣ Rolling Update (Default)

How it works: K8s gradually replaces old Pods with new Pods (e.g., one by one).

Downtime: No (if configured properly).

When to use:

Most production apps.

When zero downtime is required.

Example:

strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1       # Extra pods allowed above desired replicas
    maxUnavailable: 1 # How many old pods can be taken down at a time


✅ Safest and most common strategy.

3️⃣ Blue-Green Deployment (not native, but can be implemented with Services + Deployments)

How it works:

Two environments run: Blue (current) and Green (new).

Traffic is switched from Blue → Green once Green is ready.

Downtime: No.

Rollback: Instant (just switch back traffic).

When to use:

Critical apps where rollback must be instant.

You want a staging-like environment before releasing.

Example:

Blue = deployment-v1

Green = deployment-v2

Service points to either one.

4️⃣ Canary Deployment

How it works: Gradually release new version to a small subset of users (e.g., 5% traffic) → monitor → then increase rollout.

Downtime: No.

Rollback: Easy (just remove canary).

When to use:

When you want to test new features safely.

Useful for A/B testing.

Example:

deployment-v1 (90% replicas)

deployment-v2 (10% replicas)

Service load balances across both.

📝 Interview-Friendly Answer

“Kubernetes supports different deployment strategies. The simplest is Recreate, where all old Pods are killed and new ones created, but this causes downtime. The default is RollingUpdate, which gradually replaces Pods with no downtime. On top of that, we can implement advanced strategies like Blue-Green, where we maintain two environments and switch traffic instantly, and Canary deployments, where a small percentage of traffic is routed to the new version before full rollout. In production, RollingUpdate is most common, while Blue-Green and Canary are used when we want safer releases or instant rollbacks.


 ✅DaemonSet
 
DaemonSet in Kubernetes ensures that one copy of a Pod runs on every node in the cluster. It’s used for system-level services like log collectors, monitoring agents, or networking components that need to run everywhere. For example, if I deploy a DaemonSet with a monitoring agent, it will automatically start one agent Pod on every node, and if a new node joins the cluster, the DaemonSet creates a Pod there too. This is different from Deployments, which focus on running a desired number of replicas across nodes, not necessarily one per node.”

✅Example Interview Answer

“Let’s say I want to monitor the health of every Kubernetes node. If I run Node Exporter as a Deployment, Kubernetes might schedule Pods on only 2–3 nodes, leaving the rest unmonitored. Instead, I use a DaemonSet, which guarantees that one Node Exporter Pod runs on each node. This way, no matter how many nodes I have, every node has exactly one monitoring agent. Prometheus then scrapes these Pods to get full cluster metrics.”

❓ Why use DaemonSet when we already have ReplicaSet / ReplicationController / Pod?

“A ReplicaSet or RC only ensures a number of Pods are running, but it doesn’t control where they run. If I set replicas=5, all Pods could still end up on the same node. A DaemonSet is designed for node-level workloads — it guarantees one Pod runs on every node in the cluster. That’s why DaemonSets are used for things like monitoring agents, log collectors, or network plugins, where you need one agent per node. ReplicaSet/RC are for scaling applications, DaemonSet is for per-node system services.”


🔹 ReplicationController (RC)

Q1. Your application is running in Pods managed by a ReplicationController. Suddenly, one Pod crashes. What will the RC do?
👉 ReplicationController always checks if the number of Pods running matches the number you asked for. For example, if you set replicas=3, it expects 3 Pods running all the time.

If one Pod crashes, RC immediately notices that only 2 Pods are left.

It then creates a new Pod to bring the count back to 3.

This is why RC ensures high availability — your application never goes below the required count.

Q2. If you manually delete a Pod created by an RC, what happens?
👉 When you delete a Pod (let’s say via kubectl delete pod <pod-name>), Kubernetes removes it.

The RC detects the missing Pod and automatically creates a new Pod with the same specification.

This means you don’t have to manually restart applications if they are deleted by mistake.

Example: if you had 4 replicas, and you deleted 1, RC will quickly recreate 1 to keep the replica count = 4.

Q3. Can a ReplicationController manage Pods without labels?
👉 No ❌. RC uses labels and selectors to know which Pods belong to it.

If a Pod has no labels, RC doesn’t “see” it.

Labels are like “tags” or “IDs” that connect Pods with their controllers.

Example: If RC has a selector app=web, but your Pod has no app=web label, RC will ignore it.

Q4. If your RC is configured with replicas=3 and someone manually scales it down to 1 Pod, what happens?
👉 RC always follows the latest configuration.

If you change it with kubectl scale rc myapp --replicas=1, RC stops managing 3 Pods and only keeps 1.

The other 2 Pods will be removed.

So RC doesn’t argue with you, it just enforces the replica count you set.

Q5. Why don’t we use ReplicationController much in production anymore?
👉 RC is considered old/legacy.

It only supports equality-based selectors (app=web).

It doesn’t support advanced rolling updates or rollbacks.

That’s why in modern Kubernetes, we use ReplicaSets and Deployments.

🔹 ReplicaSet (RS)

Q1. If you want to run 5 Pods of the same application, how would a ReplicaSet ensure they are always maintained?
👉 You define a ReplicaSet with replicas: 5.

Kubernetes will create 5 Pods.

If 1 Pod crashes, RS creates 1 new Pod.

If 2 Pods are deleted manually, RS creates 2 new Pods.

If someone adds an extra Pod with matching labels, RS may actually reduce new Pod creation (because it already sees the total = 5).

So RS guarantees the correct number of Pods based on your definition.

Q2. What happens if a Pod managed by a ReplicaSet is modified manually (like labels are changed)?
👉 Imagine RS is looking for Pods with app=web.

If you change one Pod’s label from app=web → app=db, RS will stop managing it.

RS will then notice only 4 Pods match its selector instead of 5, and it creates a new one.

Result: the Pod with changed label is left unmanaged, and RS creates a fresh Pod to maintain its desired state.

Q3. You create a ReplicaSet with matchLabels: app=web. If another Pod with label app=web is created manually, will the RS manage it?
👉 Yes ✅.

RS doesn’t care who created the Pod. If the labels match its selector, it counts that Pod as part of its set.

So if RS wants 3 Pods, and you manually create 1 with app=web, RS will think “okay, I already have 1, I’ll only create 2 more.”

Q4. Can a ReplicaSet directly help in version upgrades of your application?
👉 No ❌.

If you change the Pod template in the RS (like updating the image), all existing Pods are deleted and replaced with new ones immediately.

This means downtime (because old Pods stop before new Pods are fully ready).

That’s why we don’t use ReplicaSet alone for upgrades. Instead, we use Deployment because it does rolling updates.

Q5. In what scenarios would you use a ReplicaSet directly?
👉 Very rarely in production.

Sometimes when you want a fixed number of Pods running with no need for rollouts.

Mostly, RS is used internally by Deployments, not directly by users.

🔹 Deployment

Q1. You deployed an app with a Deployment (replicas=3). A new version is released. How will you roll out the update with zero downtime?
👉 With Deployment, you just update the image:

kubectl set image deployment myapp myapp=nginx:2.0


Kubernetes creates a new ReplicaSet with the new version.

Then it starts new Pods gradually, while removing old Pods one by one.

During this time, both versions run together until the update finishes.

Result: zero downtime because your app never goes completely offline.

Q2. Suppose after upgrading your Deployment, the app is not working. How can you roll back?
👉 You can simply run:

kubectl rollout undo deployment myapp


This reverts to the previous ReplicaSet (the old version).

Kubernetes also deletes the bad ReplicaSet.

Since rollback also happens in a rolling manner, users don’t see downtime.

Q3. During a rollout, some Pods are stuck in CrashLoopBackOff. How do you troubleshoot?
👉 Steps you might take:

Check rollout status:

kubectl rollout status deployment myapp


→ This shows if rollout is stuck.

Describe Pod:

kubectl describe pod <pod-name>


→ Look for events like “image not found” or “CrashLoopBackOff”.

Logs:

kubectl logs <pod-name>


→ Check application errors.

After fixing the error (e.g., typo in image name, config issue), reapply the Deployment.

Q4. What’s the difference between Recreate and RollingUpdate strategies?
👉

Recreate: Deletes all old Pods first, then creates new Pods. App will be down until new Pods are ready → downtime ❌.

RollingUpdate (default): Slowly replaces Pods one by one, keeping service available the whole time → no downtime ✅.

Q5. If you delete a Deployment, what happens to ReplicaSets and Pods?
👉 Deployment is the “parent” object. If you delete it:

The ReplicaSets created by it are also deleted.

The Pods belonging to those ReplicaSets are also deleted.

Everything under that Deployment is cleaned up.

Q6. If HPA is scaling your Deployment up to 10 Pods, what role does Deployment and RS play?
👉

HPA decides the number of Pods needed (e.g., based on CPU).

It updates the Deployment to replicas=10.

Deployment updates its underlying ReplicaSet.

RS ensures exactly 10 Pods are running.
So HPA, Deployment, and RS all work together.

Q7. Why use Deployment instead of ReplicaSet directly?
👉 Deployment is like a smart manager sitting on top of ReplicaSet.

RS = maintains Pod count only.

Deployment = maintains Pod count plus:

Rollouts

Rollbacks

Version history

Zero downtime upgrades
That’s why in real projects, Deployment is always used.

