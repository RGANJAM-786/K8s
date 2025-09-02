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

ConfigMap or Secret not found

Example: You reference a ConfigMap/Secret in your Pod spec, but it doesn’t exist in the namespace.

Result → Pod cannot build container config.

Wrong environment variables from ConfigMap/Secret

Example: You refer to a key inside a ConfigMap that doesn’t exist.

Volume issues

Example: Pod mounts a PVC, but no matching PV is bound.

Example: Wrong subPath or invalid mountPath configuration.

Invalid Pod spec

Example: Bad syntax in YAML (wrong imagePullPolicy, invalid field name, etc.).

📌 Interview-style answer:
“In Kubernetes, a Pod shows CreateContainerConfigError when it fails to prepare the container’s configuration. This often happens when a referenced ConfigMap, Secret, or volume is missing, or when the Pod spec has an invalid configuration. For example, if I mount a ConfigMap that doesn’t exist, the Pod will stay in CreateContainerConfigError until I fix it.”
