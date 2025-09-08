

RBAC
====


RBAC (Role-Based Access Control) in Kubernetes is a powerful method for regulating access to the Kubernetes API based on the roles of individual users or groups. Here's a breakdown of its key components and how it works:

Key Components

Role: Defines a set of permissions within a specific namespace. It grants access to resources like pods, deployments, etc.

RoleBinding: Grants the permissions defined in a Role to a user or a group within a specific namespace.

ClusterRole: Similar to a Role but applies across the entire cluster. It can be used for non-namespaced resources or for cluster-wide permissions.


ClusterRoleBinding: Grants the permissions defined in a ClusterRole to a user or a group across the entire cluster.


🔹 What is RBAC?

RBAC in Kubernetes is a security mechanism that controls who (user/service) can do what (permissions) on which resources.

It works with four main components:

Role / ClusterRole → Defines permissions. (what actions are allowed)

Role → for a specific namespace.

ClusterRole → across the whole cluster.

RoleBinding / ClusterRoleBinding → Connects a user/group/service account with the Role or ClusterRole.

RoleBinding → in a namespace.

ClusterRoleBinding → cluster-wide.

ServiceAccount → Used by Pods (not humans) to interact with the cluster.

Subjects → The entities (users, groups, or service accounts) to which roles are assigned.



🔹 How I used RBAC in my projects

In real projects, RBAC is crucial because multiple teams and applications share the same cluster.

Team Isolation:

Dev team should only access resources in the dev namespace.

QA team should only work in qa namespace.

Production access is only for admins.
👉 I achieved this using Role + RoleBinding in each namespace.

Application Security:

Some monitoring tools (like Prometheus, Jenkins, ArgoCD) need access to cluster resources.

Instead of giving them admin, I created ServiceAccounts with least privilege roles (only what they need).

Compliance & Auditing:

In one project, security policy required that no user should have cluster-admin rights except very few admins.

We audited ClusterRoleBindings and removed unnecessary permissions.

CI/CD Pipeline Integration:

Jenkins needed to deploy apps into dev and qa.

I created a ServiceAccount jenkins-deployer with RoleBindings in those namespaces → Jenkins could only deploy apps but not touch other namespaces or cluster-wide settings.

🔹 Why is RBAC useful in Kubernetes?

Provides least privilege access → improves security.

Prevents accidental production changes by devs/testers.

Ensures compliance with security rules.

Makes multi-tenant clusters safe and manageable.

✅ Interview Tip (Simple Answer)
"RBAC in Kubernetes controls who can do what on which resources.
In my projects, I used RBAC to restrict teams to their own namespaces, gave CI/CD tools limited permissions via ServiceAccounts, and followed least-privilege principles to protect production resources."


🔹 RBAC Scenario-Based Interview Questions
Q1. A developer says: “I cannot delete a Pod in the dev namespace, but I can view it. What could be wrong?”

👉 Answer:

Likely the Role/RoleBinding given to the developer has only get, list, watch verbs for Pods.

It does not have delete permission.

To fix: update the Role to include delete.

Q2. You want Jenkins to deploy applications only in the qa namespace. How would you configure RBAC?

👉 Answer:

Create a ServiceAccount for Jenkins in qa namespace.

Create a Role that allows managing Deployments, Pods, Services, PVCs.

Bind the Role to the ServiceAccount using a RoleBinding.

This way, Jenkins can only deploy to qa, not touch other namespaces.

Q3. A monitoring tool like Prometheus needs to list and watch Pods across all namespaces. Which RBAC object would you use?

👉 Answer:

Since it needs access across all namespaces, I’d use a ClusterRole with list and watch on Pods.

Then bind it with a ClusterRoleBinding to Prometheus’ ServiceAccount.

Q4. A tester complains: “I cannot create ConfigMaps in the test namespace, but I can create Pods.” What’s the issue?

👉 Answer:

The Role probably allows only Pod actions but not ConfigMaps.

ConfigMaps are a separate resource type → need explicit permissions.

Update the Role to add:

resources: ["pods", "configmaps"]
verbs: ["get", "list", "create", "delete"]

Q5. If you accidentally gave a user cluster-admin role, how would you restrict it?

👉 Answer:

Check ClusterRoleBindings:

kubectl get clusterrolebindings


Identify the user or ServiceAccount bound to cluster-admin.

Remove or edit the binding so they only have namespace-specific Roles.

Always apply least privilege principle.

Q6. Can a RoleBinding in the dev namespace bind a ClusterRole?

👉 Answer:

Yes ✅. A RoleBinding can reference either a Role (namespace-specific) or a ClusterRole (cluster-wide).

But when used in a namespace, it will apply only within that namespace.

Q7. You created a Role and RoleBinding, but the developer still can’t access resources. How do you troubleshoot?

👉 Answer:

Steps I’d take:

Check if the Role has the right resources and verbs.

Verify the RoleBinding references the correct Role name.

Check if the developer is using the correct ServiceAccount/User.

Use kubectl auth can-i command to debug:

kubectl auth can-i delete pods --as=developer -n dev


This shows if the user has permission.

Q8. You want to ensure that devs can only read Secrets, but not create or modify them. How would you enforce that?

👉 Answer:

Create a Role with only get, list, watch on secrets.

Do not include create, update, or delete.

Bind it to the devs’ ServiceAccounts.

Q9. You want to allow all users to list Pods in all namespaces, but only admins can delete them. How would you achieve this?

👉 Answer:

Create a ClusterRole with list on Pods.

Bind it to system:authenticated group so all users can list.

Create another ClusterRole with delete on Pods.

Bind it only to admin ServiceAccounts.

Q10. A Pod needs to access the Kubernetes API to create ConfigMaps. How can you allow that securely?

👉 Answer:

Assign a ServiceAccount to the Pod.

Create a Role with create permission on ConfigMaps.

Bind the Role to the ServiceAccount with a RoleBinding.

This way, the Pod has only the access it needs, nothing more.

✅ Simple Summary for Interviews:
RBAC is about least privilege. You always think:

Who? (user, group, service account)

What can they do? (verbs like get, list, create, delete)

On which resources? (pods, services, secrets, configmaps, etc.)

Where? (namespace or whole cluster)
