


❓ Interview Question:

“What kind of dynamic storage provisioning have you used in real projects? NFS or Cloud storage classes? Why?”

✅ Strong Answer:

“In real-world Kubernetes setups, the choice depends on the environment:

On-premises clusters: We mostly use NFS dynamic provisioning with an external provisioner. The reason is NFS supports ReadWriteMany access mode, so multiple Pods across different nodes can share the same data. This is useful for shared workloads like logging, monitoring agents, or content management systems.

Cloud environments (AWS, GCP, Azure): Here, we prefer the native storage classes like AWS EBS/EFS, GCP Persistent Disks/Filestore, or Azure Disk/File. These are fully managed, highly available, and integrate tightly with the cloud provider, so we don’t have to manage the storage backend ourselves.

Trend in the industry: In cloud projects, cloud-native storage classes are more popular today because of simplicity and built-in reliability. But in on-prem setups, NFS is still very common and reliable.

So, I’ve used both approaches depending on the project – NFS for on-prem, cloud-native storage classes for cloud. This gives flexibility and ensures portability of workloads.”

👉 With this explanation, you show that:

You know NFS and why it’s used.

You know cloud-native alternatives and why they’re trending.

You show adaptability based on the project environment.



❓ Scenario Question:

“Imagine your application runs multiple Pods across different nodes, and all Pods need to read and write to the same shared storage location. Which storage option would you choose in Kubernetes, and why?”

✅ Answer (Simple & Strong):

“In this case, I would choose a storage option that supports ReadWriteMany (RWX) access mode, because multiple Pods across nodes need to access the same data.

If it’s an on-premises cluster, I would use NFS with dynamic provisioning. The NFS external provisioner can automatically create PersistentVolumes when a PVC is made, and all Pods can share the same storage.

If it’s a cloud cluster, I would prefer something like AWS EFS, Azure File, or GCP Filestore because these cloud storage classes also support RWX and are managed services, so I don’t have to worry about maintaining the NFS server myself.

Using block storage like AWS EBS or GCP PD wouldn’t work here because they support only ReadWriteOnce (RWO), which means only one Pod/node could use it at a time. So the best fit is NFS or cloud file storage for shared storage needs.”

👉 This shows the interviewer that you:

Understand access modes (RWO vs RWX).

Can differentiate on-prem vs cloud setups.

Think practically about real-world storage use cases.




🔹 1. Scenario Question:

“Your developer team says: ‘We need 5GB storage for our application. We don’t care which disk it comes from.’ As a DevOps engineer, how do you handle this in Kubernetes?”

✅ Answer:
I won’t expose the details of storage (like NFS or EBS) directly to developers. Instead, I’ll ask them to create a PVC (PersistentVolumeClaim) for 5GB. Kubernetes will automatically match it with a suitable PV if static PVs exist, or dynamically create a PV using a StorageClass. This way, developers only request storage, and I manage the backend.

🔹 2. Scenario Question:

“Your PVC is stuck in Pending state. What could be the possible reasons?”

✅ Answer:

No available PV matches the PVC’s size, access mode, or StorageClass.

The StorageClass used in PVC doesn’t exist or is misconfigured.

Dynamic provisioner (like NFS provisioner or AWS EBS plugin) is not running or failing.
So, I’ll first check the PVC events and then verify the PV/StorageClass configuration.

🔹 3. Scenario Question:

“You deleted a PVC by mistake, but the PV is still in Released state. How do you recover the data?”

✅ Answer:
When PVC is deleted, PV status changes to Released. The data is still on the storage backend, but the PV cannot be bound to a new PVC automatically. To recover:

Manually create a new PVC with the same storage class, size, and access mode.

Or patch the existing PV (spec.claimRef field) to bind it to the new PVC.

Then, mount that PVC to the Pod to access the old data.

🔹 4. Scenario Question:

“You have an application that requires multiple Pods across nodes to share the same logs directory. Which storage solution would you choose?”

✅ Answer:
I would choose a storage solution that supports ReadWriteMany (RWX), like:

NFS (on-prem) with dynamic provisioning.

AWS EFS, GCP Filestore, or Azure Files (cloud).
These allow multiple Pods on different nodes to share the same storage, unlike block storage (EBS/PD) which only supports ReadWriteOnce.

🔹 5. Scenario Question:

“Your PV has a ReclaimPolicy set to Delete. What happens when PVC is deleted?”

✅ Answer:
If ReclaimPolicy is Delete, the underlying storage (like EBS volume, NFS subdirectory) is also deleted automatically. This is useful for temporary data.
If we want to keep data safe, we should set the ReclaimPolicy to Retain or Recycle (though recycle is deprecated).

🔹 6. Scenario Question:

“Why do we need a StorageClass for dynamic volumes?”

✅ Answer:
StorageClass acts as a blueprint that tells Kubernetes how to create PVs automatically. Without a StorageClass, we’d have to manually create PVs (static provisioning). With StorageClass, Kubernetes dynamically creates a PV whenever a PVC is made, saving time and reducing human errors.
