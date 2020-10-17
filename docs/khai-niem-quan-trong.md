Có các khái niệm chính như sau trong K8S:
- Pods
- Labels
- Replica Controllers
- Replica Sets
- Deployments
- Services
- Volumes
- Config Maps
- Daemons
- Jobs
- Cron Jobs
- Namespaces
- Quotas and Limits

---

Controller 
Kubernetes usually does not create Pods directly, but manages Pods through Controller. The Controller defines the deployment characteristics of the Pod, such as how many copies are there, and what kind of Node it runs on. In order to meet different business scenarios, Kubernetes provides a variety of Controllers, including Deployment, ReplicaSet, DaemonSet, StatefuleSet, Job, etc. We will discuss them one by one.

Deployment  is the most commonly used Controller. For example, in the previous online tutorials, deployments are deployed by creating Deployments. The Deployment can manage multiple copies of the Pod and ensure that the Pod is running in the desired state.

ReplicaSet  implements the management of multiple copies of Pod. When using Deployment, ReplicaSet is automatically created, which means that Deployment manages multiple copies of Pod through ReplicaSet. We usually don't need to use ReplicaSet directly.

DaemonSet is  used in scenarios where each Node can run at most one Pod copy. As its name suggests, DaemonSet is usually used to run daemons.

StatefuleSet  can ensure that the name of each copy of the Pod remains unchanged throughout its life cycle. Other Controllers do not provide this function. When a Pod fails and needs to be deleted and restarted, the name of the Pod will change. At the same time, StatefuleSet will ensure that the copies are started, updated or deleted in a fixed order.

Job is  used for applications that are deleted at the end of the run. Pods in other Controllers usually run continuously for a long time.

Service 
Deployment can deploy multiple copies. Each Pod has its own IP. How can the outside world access these copies?

Is it through the Pod's IP?
It is important to know that Pods are likely to be destroyed and restarted frequently, their IP will change, and it is not realistic to use IP to access.

The answer is Service.
Kubernetes Service defines a way for the outside world to access a specific set of Pods. Service has its own IP and port, and Service provides load balancing for Pod.

The two tasks of Kubernetes running the container (Pod) and accessing the container (Pod) are performed by the Controller and Service respectively.

Namespace

If multiple users or project groups use the same Kubernetes Cluster, how to separate the Controller, Pod and other resources they created?

The answer is Namespace.
Namespace can logically divide a physical Cluster into multiple virtual Clusters, and each Cluster is a Namespace. Resources in different Namespaces are completely isolated.

Kubernetes creates two Namespaces by default.

590.png

default - If not specified when creating a resource, it will be placed in this Namespace.

kube-system - The system resources created by Kubernetes itself will be placed in this Namespace.

Familiar with these important concepts of k8s, starting from the next section, we will build our own Kubernetes cluster.

