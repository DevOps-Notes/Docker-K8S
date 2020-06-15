# Kubernetes (K8s)

* **Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications.**
* **Kubernetes Features:**
  * Service discovery and load balancing
  No need to modify your application to use an unfamiliar service discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.
  * Service Topology
  Routing of service traffic based upon cluster topology.
  * Storage orchestration
  Automatically mount the storage system of your choice, whether from local storage, a public cloud provider such as GCP or AWS, or a network storage system such as NFS, iSCSI, Gluster, Ceph, Cinder, or Flocker.
  * Self-healing
  Restarts containers that fail, replaces and reschedules containers when nodes die, kills containers that don't respond to your user-defined health check, and doesn't advertise them to clients until they are ready to serve.
  * Automated rollouts and rollbacks
  Kubernetes progressively rolls out changes to your application or its configuration, while monitoring application health to ensure it doesn't kill all your instances at the same time. If something goes wrong, Kubernetes will rollback the change for you. Take advantage of a growing ecosystem of deployment solutions.
  * Secret and configuration management
  Deploy and update secrets and application configuration without rebuilding your image and without exposing secrets in your stack configuration.
  * Automatic bin packing
  Automatically places containers based on their resource requirements and other constraints, while not sacrificing availability. Mix critical and best-effort workloads in order to drive up utilization and save even more resources.
  * Batch execution
  In addition to services, Kubernetes can manage your batch and CI workloads, replacing containers that fail, if desired.
  * IPv4/IPv6 dual-stack
  Allocation of IPv4 and IPv6 addresses to Pods and Services
  * Horizontal scaling
  Scale your application up and down with a simple command, with a UI, or automatically based on CPU usage.

* **Kubernetes Architecture has the following main components:**

  * **Master nodes**

    It is the entry point for all administrative tasks which is responsible for managing the Kubernetes cluster. There can be more than one master node in the cluster to check for fault tolerance. More than one master node puts the system in a High Availability mode, in which one of them will be the main node which we perform all the tasks.

    For managing the cluster state, it uses etcd in which all the master nodes connect to it.

    * API server:
    Performs all the administrative tasks through the API server within the master node.
    In this REST commands are sent to the API server which validates and processes the requests.
    After requesting, the resulting state of the cluster is stored in the distributed key-value store.

    * Scheduler:
    The scheduler schedules the tasks to slave nodes. It stores the resource usage information for each slave node.
    It schedules the work in the form of Pods and Services.
    Before scheduling the task, the scheduler also takes into account the quality of the service requirements, data locality, affinity, anti-affinity, etc.

    * Controller manager:
    Also known as controllers.
    It is a daemon which regulates the Kubernetes cluster which manages the different non-terminating control loops.
    It also performs lifecycle functions such as namespace creation and lifecycle, event garbage collection, terminated-pod garbage collection, cascading-deletion garbage collection, node garbage collection, etc.
    Basically, a controller watches the desired state of the objects it manages and watches their current state through the API server. If the current state of the objects it manages does not meet the desired state, then the control loop takes corrective steps to make sure that the current state is the same as the desired state.

  * **Worker/Slave nodes (formerly minions)**

    It is a physical server or you can say a VM which runs the applications using Pods (a pod scheduling unit) which is controlled by the master node. On a physical server (worker/slave node), pods are scheduled. For accessing the applications from the external world, we connect to nodes. 

    * Container runtime:
    To run and manage a container’s lifecycle, we need a container runtime on the worker node.
    Sometimes, Docker is also referred to as a container runtime, but to be precise, Docker is a platform which uses containers as a container runtime.

    * Kubelet:
    It is an agent which communicates with the Master node and executes on nodes or the worker nodes. It gets the Pod specifications through the API server and executes the containers associated with the Pod and ensures that the containers described in those Pod are running and healthy.

    * Kube-proxy:
    Kube-proxy runs on each node to deal with individual host sub-netting and ensure that the services are available to external parties.
    It serves as a network proxy and a load balancer for a service on a single worker node and manages the network routing for TCP and UDP packets.
    It is the network proxy which runs on each worker node and listens to the API server for each Service endpoint creation/deletion.
    For each Service endpoint, kube-proxy sets up the routes so that it can reach to it.

    * Pods:
    **A pod is one or more containers that logically go together. Pods run on nodes. Pods run together as a logical unit. So they have the same shared content. They all share the same IP address but can reach other Pods via localhost, as well as shared storage. Pods don’t need to all run on the same machine as containers can span more than one machine. One node can run multiple pods.**

  * **Distributed key-value store(etcd)**

    etcd is a distributed key-value store which stores the cluster state.
    It can be part of the Kubernetes Master, or, it can be configured externally.

* **Core concepts**

  * **Controllers**

    In robotics and automation, a control loop is a non-terminating loop that regulates the state of a system.

    Here is one example of a control loop: a thermostat in a room.

    When you set the temperature, that's telling the thermostat about your desired state. The actual room temperature is the current state. The thermostat acts to bring the current state closer to the desired state, by turning equipment on or off.

    In Kubernetes, controllers are control loops that watch the state of your cluster, then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state.

    * ReplicationController
    A replication controller ensures that a specified number of replicas of a pod are running at all times. If pods exit or are deleted, the replication controller acts to instantiate more up to the defined number. Likewise, if there are more running than desired, it deletes as many as necessary to match the defined amount.

    * ReplicaSet
    Similar to a replication controller, a replica set ensures that a specified number of pod replicas are running at any given time. The difference between a replica set and a replication controller is that a replica set supports set-based selector requirements whereas a replication controller only supports equality-based selector requirements.

    * Deployments
    Deployments are intended to replace Replication Controllers. They provide the same replication functions (through Replica Sets) and also the ability to rollout changes and roll them back if necessary.
    By using a Deployment to manage a ReplicaSet, you can easily rollback to an earlier Deployment revision. A Deployment allows you to create a newer revision of a ReplicaSet and then migrate existing pods from a previous ReplicaSet into the new revision. The Deployment can then manage the cleanup of older unused ReplicaSets.

    * StatefulSets
    StatefulSet is the workload API object used to manage stateful applications.
    If you want to use storage volumes to provide persistence for your workload, you can use a StatefulSet as part of the solution. Although individual Pods in a StatefulSet are susceptible to failure, the persistent Pod identifiers make it easier to match existing volumes to the new Pods that replace any that have failed.

    * Daemon Set
    A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.
    Some typical uses of a DaemonSet are:
    running a cluster storage daemon on every node
    running a logs collection daemon on every node
    running a node monitoring daemon on every node

    * Jobs
    A job is similar to a replication controller, in that its purpose is to create pods for specified reasons. The difference is that replication controllers are designed for pods that will be continuously running, whereas jobs are for one-time pods. A job tracks any successful completions and when the specified amount of completions have been reached, the job itself is completed.

    * CronJob
    A CronJob creates Jobs on a repeating schedule.
    One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.

  * **Service**
    An abstract way to expose an application running on a set of Pods as a network service.
    With Kubernetes you don't need to modify your application to use an unfamiliar service discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.