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
