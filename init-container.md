❓ Interviewer: Can you explain what Init Containers are, and how they differ from Liveness and Readiness probes?
✅ Candidate Answer (Clear & Simple):

"An Init Container is a special container in Kubernetes that always runs before the main application container starts. Its job is usually to do setup tasks, like checking if dependencies are available, copying configuration files, or running database migrations. It only runs once, and the Pod won’t start the main container until the Init Container finishes successfully.

On the other hand, Liveness and Readiness probes are health checks for containers after they start running.

A Liveness probe checks if the container is still working properly. If it fails, Kubernetes restarts the container.

A Readiness probe checks if the container is ready to handle traffic. If it fails, Kubernetes temporarily removes the Pod from the Service endpoints until it becomes ready again.

So, the main difference is:

Init Container = one-time setup before app starts.

Liveness & Readiness probes = continuous health checks after app starts."

💡 Example for extra clarity (if interviewer asks for scenario):

"In one of my projects, I used an Init Container to make sure a database schema migration was applied before starting the application container. For the same app, I also used a Liveness probe to restart the app if it ever got stuck due to a memory leak, and a Readiness probe to make sure the app only started receiving traffic after it had fully loaded all configurations. This combination ensured reliability and smooth deployments."

👉 This way, you show clear difference, real-world usage, and your experience.


🔹 Scenario-Based Interview Questions on Init Containers & Probes
1.

Q: Suppose your application needs a configuration file from an external Git repo before starting. How would you handle this in Kubernetes?
✅ A: I’d use an Init Container that clones the repo or fetches the file, places it in a shared volume, and then the main app container uses that volume.

2.

Q: Your Pod is running, but your application is stuck due to a memory leak. How would Kubernetes detect and fix this automatically?
✅ A: I’d configure a Liveness Probe. If the probe fails (e.g., HTTP check doesn’t respond), Kubernetes will restart the container automatically.

3.

Q: Imagine your app takes 2 minutes to fully start because it loads large data into memory. How would you make sure Kubernetes doesn’t send traffic too early?
✅ A: I’d configure a Readiness Probe with an appropriate initialDelaySeconds and conditions so the Pod only starts receiving traffic when it’s ready.

4.

Q: If an Init Container fails, what happens to the Pod?
✅ A: The Pod won’t start the main containers. Kubernetes retries the Init Container until it succeeds. This ensures setup tasks are completed before the app runs.

5.

Q: Can a Pod have multiple Init Containers? If yes, how do they run?
✅ A: Yes, a Pod can have multiple Init Containers. They always run sequentially, one after another. Each must succeed before the next one starts.

6.

Q: In your project, did you ever combine Init Containers with Probes? Can you explain a real scenario?
✅ A: Yes. Example:

Used an Init Container to wait until a database service became available.

Used a Readiness Probe to delay traffic until the app connected successfully to the DB.

Used a Liveness Probe to restart the app if it got stuck due to long-running queries.
