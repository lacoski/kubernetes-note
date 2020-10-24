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




Readiness Checks


Deployment Strategies
- Recreate Strategy
- RollingUpdate Strategy

rollouts

DaemonSets
DaemonSets are used to deploy system daemons such as log collectors
and monitoring agents, which typically must run on every node. DaemonSets share
similar functionality with ReplicaSets; both create Pods that are expected to be longrunning services and ensure that the desired state and the observed state of the clus‐
ter match.

RollingUpdate

Jobs

ConfigMaps and Secrets

Role-Based Access Control

Integrating Storage Solutions and Kubernetes

StatefulSets

• A persistent volume to manage the lifespan of the on-disk storage independently
from the lifespan of the running MySQL application
• A MySQL Pod that will run the MySQL application
• A service that will expose this Pod to other containers in the cluster

singleton Pod

https://kubernetes.io/docs/concepts/services-networking/service/

Service resources
In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them (sometimes this pattern is called a micro-service). The set of Pods targeted by a Service is usually determined by a selector. To learn about other ways to define Service endpoints, see Services without selectors.

Lastly, the user-space proxy installs iptables rules which capture traffic to the Service's clusterIP (which is virtual) and port. The rules redirect that traffic to the proxy port which proxies the backend Pod.

By default, kube-proxy in userspace mode chooses a backend via a round-robin algorithm.

https://kubernetes.io/docs/concepts/services-networking/service/#proxy-mode-ipvs

https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types