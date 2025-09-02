ConfigMaps and Secrets in Kubernetes

🔹 ConfigMap

Used to store non-confidential configuration data (like environment variables, config files, URLs, app settings).

Example: database host, app mode (dev, prod).

🔹 Secret

Used to store sensitive data (like passwords, API keys, tokens, certificates).

Data inside a Secret is base64 encoded (not fully encrypted, but safer than ConfigMap).

Real-world Example

Imagine you’re deploying an application that connects to a database.

The database hostname is not sensitive → goes into a ConfigMap.

The database password is sensitive → goes into a Secret.

👍 ConfigMap does not provide secrecy or encryption. If the data you want to store are confidential, use a Secret rather than a ConfigMap, or use additional (third party) tools to keep your data private.


📌 when will be the pod status will be create container congig error

In Kubernetes, a Pod goes into CreateContainerConfigError when Kubernetes fails to generate the container’s runtime configuration before actually starting it.

👉 This usually happens when something is wrong with the configuration of the Pod, not with the image or runtime itself.

Here are the common causes you can mention in an interview:

👍 ConfigMap or Secret not found

Example: You reference a ConfigMap/Secret in your Pod spec, but it doesn’t exist in the namespace.

Result → Pod cannot build container config.

👍 Wrong environment variables from ConfigMap/Secret

Example: You refer to a key inside a ConfigMap that doesn’t exist.

👍 Volume issues

Example: Pod mounts a PVC, but no matching PV is bound.

Example: Wrong subPath or invalid mountPath configuration.

👍 Invalid Pod spec

Example: Bad syntax in YAML (wrong imagePullPolicy, invalid field name, etc.).

📌 Interview-style answer:
“In Kubernetes, a Pod shows CreateContainerConfigError when it fails to prepare the container’s configuration. This often happens when a referenced ConfigMap, Secret, or volume is missing, or when the Pod spec has an invalid configuration. For example, if I mount a ConfigMap that doesn’t exist, the Pod will stay in CreateContainerConfigError until I fix it.”


:

🔹 ConfigMap Scenarios

Scenario:
You created a Pod that reads environment variables from a ConfigMap, but the Pod is stuck in CreateContainerConfigError.

Question: How would you troubleshoot this issue?

Expected Answer: Check if the ConfigMap exists in the same namespace, verify the keys inside it match what the Pod spec is referencing, and run kubectl describe pod to see the exact error.

Scenario:
You updated a ConfigMap that your Deployment is using. But the running Pods didn’t pick up the new values.

Question: Why didn’t the Pods update automatically, and how can you make them use the new values?

Expected Answer: ConfigMaps are not automatically updated in existing Pods. You need to restart the Pods (redeploy) so they can load the new ConfigMap values.

Scenario:
You mounted a ConfigMap as a volume into a Pod, but inside the container, you don’t see the expected file.

Question: What could be the possible reasons?

Expected Answer: Either the ConfigMap doesn’t exist, the key doesn’t exist, wrong mount path is used, or there was a YAML indentation mistake.

🔹 Secrets Scenarios

Scenario:

You want to store a database password for your application. Would you use a ConfigMap or a Secret? Why?

Expected Answer: Use a Secret because it stores data in base64-encoded format, designed for sensitive information, while ConfigMaps are for non-confidential data.

Scenario:

Your Pod is failing to start because it’s referencing a Secret. You checked and the Secret exists.

Question: What could still be wrong?

Expected Answer: The key inside the Secret might not match what the Pod spec expects, or the Secret could be in a different namespace.

Scenario:

You mounted a Secret as a volume, but when you check inside the container, the file permissions are too restrictive and the app can’t read it.

Question: How would you fix this?

Expected Answer: Adjust file permissions by using defaultMode in the volume spec, or configure the app to run with proper permissions.

🔹 Mixed ConfigMap & Secret Scenarios

Scenario:

Your application requires both a non-sensitive config file and a sensitive password.

Question: How would you design this in Kubernetes?

Expected Answer: Store the config file in a ConfigMap and the password in a Secret, then mount both into the Pod either as environment variables or volumes.

📌 Pro tip for interviews: Whenever you answer, add how to debug (kubectl describe pod, kubectl get cm, kubectl get secret, checking namespace). That makes your answer more practical.
