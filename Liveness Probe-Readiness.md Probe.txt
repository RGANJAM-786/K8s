🔹Liveness Probe
-----------------
“Imagine a Pod is running an application, but due to issues like a memory leak, high CPU usage, or a deadlock, the app stops responding.
The liveness probe keeps checking the container’s health. If it fails, Kubernetes assumes the app is stuck and automatically restarts the container to recover.”


🔹Readiness Probe
---------------
“Sometimes an application inside a Pod is running, but it’s not yet ready to handle requests — for example, it might still be loading configuration files or waiting for a database connection.
The readiness probe checks if the container is ready to serve traffic. If it fails, Kubernetes keeps the Pod running but temporarily removes it from the Service load balancer until it becomes ready again.”


🔹 Liveness Probe Scenarios
1. Stuck Application

Scenario: Your application is running, but due to a memory leak, it has stopped responding. The Pod status shows Running, but no requests are served.

Question: Which probe helps in this case?

Answer: Liveness probe — It detects the app is stuck and restarts the container.

2. Wrong Probe Path

Scenario: You configured a liveness probe to check /health, but your app serves health checks on /status. The Pod keeps restarting in a loop.

Question: Why is this happening?

Answer: The probe is failing because the path is wrong, so Kubernetes thinks the container is unhealthy and restarts it repeatedly.

3. Application Startup Delay

Scenario: Your app takes 2 minutes to start, but you configured liveness probe with initialDelaySeconds: 10. The Pod never stabilizes.

Question: What’s the problem, and how do you fix it?

Answer: The liveness probe is checking too early, before the app is ready, causing restart loops. Fix by increasing initialDelaySeconds.

🔹 Readiness Probe Scenarios
4. Database Connection Not Ready

Scenario: Your app needs to connect to a database before serving traffic. The container is running, but the DB is still initializing.

Question: Which probe helps prevent sending traffic too early?

Answer: Readiness probe — It ensures the Pod won’t get traffic until the DB connection is ready.

5. Temporary Backend Failure

Scenario: Your app temporarily loses DB connection. The container is still alive, but requests fail.

Question: What happens if readiness probe fails during this time?

Answer: Kubernetes keeps the Pod running but removes it from Service endpoints until it’s healthy again.

6. Rolling Deployment

Scenario: During a rolling update, some Pods are still starting up. If they receive traffic too soon, users face errors.

Question: How do readiness probes help in this situation?

Answer: Readiness probes keep new Pods out of load balancer rotation until they’re fully ready, ensuring zero downtime deployments.

🔹 Combo Question (Liveness vs Readiness)

Scenario: Your app runs fine at first but later gets stuck due to a deadlock. At the same time, during startup, it takes time to connect to DB.

Question: Which probe helps during startup, and which one helps when the app hangs later?

Answer: Readiness probe ensures traffic isn’t sent until the DB is ready. Liveness probe restarts the container when it gets stuck later.



Example
=======
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebappdep
  namespace: prod
spec:
  replicas: 2
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: javawebapp
  template:
    metadata:
      name: javawebapp
      labels:
        app: javawebapp
    spec:
      containers:
      - name: javawebapp
        image: kkeducation123456/maven-web-app:1.2
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /maven-web-application  # Adjust based on your application's health check endpoint [maven-web-application]
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /maven-web-application  # Adjust based on your application's readiness check endpoint [maven-web-application]
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 5
          failureThreshold: 3

---
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
  namespace: prod
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
  - port: 80
    targetPort: 8080
    

==========================




        livenessProbe:
          httpGet:
            path: /health  # Adjust based on your application's health check endpoint
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready  # Adjust based on your application's readiness check endpoint
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 5
          failureThreshold: 3
