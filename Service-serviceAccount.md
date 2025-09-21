# 🔹 Service Object in Kubernetes

A Service is an abstraction in Kubernetes that provides stable networking for Pods.

Since Pods are ephemeral (they can die and restart with new IPs), a Service ensures that we always have a fixed DNS name or IP to access them.

Services can also load balance traffic across multiple Pods of the same application.


👉 Types of Services:


ClusterIP – Default. Exposes the Service inside the cluster only.

NodePort – Exposes the Service on a static port on each Node so it can be accessed from outside.

LoadBalancer – Integrates with Cloud Provider’s load balancer (e.g., AWS ELB, GCP LB).

Headless Service – Doesn’t do load balancing, used often with StatefulSets to talk to Pods directly.


⚡ How I used in project:


For our microservices-based application, we used ClusterIP to enable communication between services internally.

For external traffic, we exposed critical apps using LoadBalancer Service behind Ingress.

For our database StatefulSet, we used a Headless Service to make each replica discoverable individually.


# 🔹 Service Account in Kubernetes


A Service Account (SA) is an identity used by Pods to interact with the Kubernetes API.

By default, every Pod gets a default service account, but often we need custom service accounts with specific permissions.

Permissions are granted using RBAC (Roles/ClusterRoles + RoleBindings/ClusterRoleBindings).

👉 Example:

If a Pod needs to read Secrets or list ConfigMaps, I don’t want to give it cluster-admin rights.

Instead, I create a custom ServiceAccount, bind it with only the required permissions, and assign it to the Pod spec under serviceAccountName.

⚡ How I used in project:

For our CI/CD pipeline (Jenkins running inside cluster), we created a ServiceAccount with limited RBAC so Jenkins could deploy applications but not delete system resources.

For monitoring tools like Prometheus, we created a ServiceAccount that had permission to list/watch Pods, Nodes, and Endpoints to scrape metrics securely.

✅ Simple Difference:

Service = Networking for Pods (who can talk to the Pod).

Service Account = Identity for Pods (what the Pod itself can do inside the cluster).


📝 Answer:

“In Kubernetes, a Service is used to expose Pods and provide stable networking. Since Pods keep changing IPs, a Service makes sure we have a fixed way to reach them, and it can also load balance traffic across multiple Pods.

On the other hand, a Service Account is like an identity for Pods. If a Pod needs to talk to the Kubernetes API or access resources like ConfigMaps or Secrets, it uses a Service Account. We can give it only the permissions it needs using RBAC.

For example, in my project, I used Services to expose my microservices and databases, and I used Service Accounts to make sure apps like Jenkins or Prometheus could securely interact with the cluster without giving them unnecessary access.”


# scenario-based interview questions

# 🔹 Service (Networking)


Q1. You deployed a web application with 3 replicas. Each Pod has a different IP. How will users consistently access the app without worrying about Pod IP changes?

👉 Answer:
“In Kubernetes, Pod IPs keep changing if Pods restart. To solve this, we use a Service. A Service gives a fixed DNS name and virtual IP that stays the same, no matter how many times Pods restart. The Service automatically routes traffic to the right Pod. So users don’t need to worry about Pod IPs.”

Q2. You deployed a database as a Pod, and multiple applications need to access it within the cluster. How will you make sure all apps can connect reliably?

👉 Answer:
“I’ll create a ClusterIP Service for the database. That way, all applications can connect using a single stable DNS name, like mysql-service.default.svc.cluster.local. Even if the DB Pod restarts with a new IP, the Service always routes to it.”

Q3. Your service should be accessible from the internet, but only for specific ports. How would you configure it?

👉 Answer:
“I’ll use either a NodePort or a LoadBalancer Service. With NodePort, the service is exposed on each Node’s IP at a static port. With LoadBalancer, a cloud provider creates an external load balancer. I can also restrict ports or firewall rules so that only required ports are open.”

Q4. You want traffic to be distributed evenly across all Pods of your backend service. How would Kubernetes achieve this?

👉 Answer:
“Kubernetes Service automatically does load balancing using kube-proxy. When requests come to the Service, they are evenly distributed across all healthy Pods behind it.”


# 🔹 Service Account (Security & Access)


Q1. You have a Pod running a monitoring tool that needs to read metrics from the Kubernetes API. How will you give it access without exposing full cluster-admin rights?

👉 Answer:
“I’ll create a Service Account for the monitoring Pod, then assign it a Role/RoleBinding with only read permissions on metrics resources. That way, it has just enough access, not full admin rights.”

Q2. Your Dev team wants to deploy apps but should not be able to delete Pods. How would you configure this using Service Accounts?

👉 Answer:
“I’ll create a Role that allows get, list, and create Pods, but not delete. Then I’ll bind this Role to the Dev team’s Service Account. This ensures they can deploy apps but cannot delete running Pods.”

Q3. You integrated Jenkins with Kubernetes to deploy apps. Jenkins needs to run kubectl commands inside the cluster. How will you give Jenkins secure access?

👉 Answer:
“I’ll create a dedicated Service Account for Jenkins, and attach only required RBAC rules (like managing Deployments and Services). Jenkins will use this Service Account’s token to authenticate with the Kubernetes API securely.”

Q4. You want to audit which Pods are making API requests. How can Service Accounts help here?

👉 Answer:
“Each Pod in Kubernetes runs with a Service Account token. Whenever that Pod talks to the API server, the API server logs show which Service Account made the request. This way, we can easily audit and track actions.”


✅ So in simple words:

Service = Stable networking & load balancing for Pods.

Service Account = Secure identity to control Pod/API access.
