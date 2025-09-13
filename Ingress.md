🔹 What is Ingress in Kubernetes?

👉 By default, Pods are only accessible inside the cluster. If you want to access them from outside, you usually expose them with a Service (NodePort / LoadBalancer).
But in large applications, managing many services with external IPs is complex.

That’s where Ingress comes in.

👉 Ingress is a Kubernetes object that manages external access (HTTP/HTTPS) to services inside the cluster.
It acts like a smart router or entry point for your cluster traffic.


🔹 How does it work?

You create an Ingress resource (YAML) → defines routing rules.

You need an Ingress Controller (e.g., NGINX, HAProxy, AWS ALB Ingress Controller) → actually enforces those rules.

Traffic → Ingress Controller → Routes to correct Service → Routes to correct Pod.


<img width="755" height="768" alt="image" src="https://github.com/user-attachments/assets/60c77bf7-c4ae-49ce-ae9d-daa29c10e402" />


👉 Here:

Requests to myapp.com/api go to api-service.

Requests to myapp.com/web go to web-service.

🔹 Real-Time Example from Projects

In one project, we had multiple microservices (login, payment, orders). Instead of exposing each one separately with a LoadBalancer, we used an NGINX Ingress Controller.

shop.com/login → login service

shop.com/payment → payment service

shop.com/orders → order service

This reduced cost, simplified management, and allowed us to add SSL easily.

✅ Simple Interview Line:
“Ingress is like a traffic controller in Kubernetes. It routes external HTTP/HTTPS traffic to the right services inside the cluster, based on rules like URL paths or domains. It makes applications more secure, scalable, and cost-efficient.”


🔹 Scenario-Based Interview Questions on Ingress
Q1. You have 5 microservices running in Kubernetes. If you expose each service with a LoadBalancer, what’s the issue, and how can Ingress help?

✅ Answer:
If I create a LoadBalancer for each service, it will increase cloud costs and complicate DNS management. Instead, I can use Ingress as a single entry point. With path-based or host-based rules, I can route traffic to the correct service.
For example:

shop.com/cart → cart service

shop.com/payment → payment service

shop.com/orders → orders service

Q2. Suppose your application needs HTTPS. How will you configure it using Ingress?

✅ Answer:
I will create a TLS secret containing the SSL certificate and key, and then reference it in the Ingress YAML under tls.
Example:

<img width="358" height="173" alt="image" src="https://github.com/user-attachments/assets/9f09cb67-2472-4f70-a3f4-acb4996d9111" />


This way, the Ingress Controller handles SSL termination, so I don’t need to configure HTTPS separately in every Pod.


Q3. If a user says some routes are not working (e.g., /login is accessible but /orders is not), how will you troubleshoot?

✅ Answer:

Check Ingress rules (kubectl describe ingress myapp-ingress) – maybe the path or host is misconfigured.

Verify the backend Service exists and is working (kubectl get svc).

Ensure the Pods for that service are running and passing readiness probes.

Look at Ingress Controller logs (e.g., NGINX logs) for errors.


Q4. What happens if you create an Ingress object but don’t have an Ingress Controller?

✅ Answer:
Nothing will happen. The Ingress resource is only a set of rules. Without an Ingress Controller (like NGINX, HAProxy, AWS ALB), those rules are not enforced, so external traffic won’t work.


Q5. How is Ingress different from a Service of type LoadBalancer?

✅ Answer:

LoadBalancer Service → Creates an external IP for one service only.

Ingress → Uses one LoadBalancer (via Controller) to route traffic to multiple services using rules.
👉 Ingress is cheaper and more flexible than creating multiple LoadBalancers.


Q6. Imagine you have two applications running in different namespaces (team1 and team2), but both need to be accessed at app1.company.com and app2.company.com. Can you do this with Ingress?

✅ Answer:
Yes, using host-based routing in Ingress rules:

app1.company.com → Service in team1 namespace

app2.company.com → Service in team2 namespace
I’ll need to ensure the Ingress Controller is configured to watch multiple namespaces.


Q7. In your project, how did you use Ingress and what benefits did you get?

✅ Answer:
In my project, we used NGINX Ingress Controller to manage external traffic for multiple microservices.

Reduced cost: instead of 10+ LoadBalancers, we had just 1.

Easier SSL management: TLS termination handled in Ingress.

Centralized traffic control: We could add new routes without changing DNS or cloud load balancer.

✅ Simple Interview Summary:
“Ingress acts like a smart traffic router for Kubernetes. It saves cost, simplifies management, enables HTTPS, and gives us flexibility to handle multiple services through a single entry point.”
