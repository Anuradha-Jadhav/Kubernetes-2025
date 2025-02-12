# Kubernetes Components: Master & Worker Nodes

## Master Node Components

1. **kube-apiserver** - Exposes Kubernetes API; acts as a control plane interface.  
2. **etcd** - Distributed key-value store for cluster configuration and state.  
3. **kube-scheduler** - Assigns Pods to nodes based on resource availability and constraints.  
4. **kube-controller-manager** - Runs controllers (Node, Replication, Endpoint, etc.) to maintain cluster state.  
5. **cloud-controller-manager** - Manages cloud provider-specific resources (if using a cloud environment).  

## Worker Node Components

1. **kubelet** - Manages containers on the node and communicates with the API server.  
2. **kube-proxy** - Handles network communication and load balancing within the cluster.  
3. **Container Runtime (Docker/Containerd/CRI-O)** - Runs containerized applications inside Pods.  

## Kubernetes Architecture Diagram

```
 +------------------------------------------------------+
 |                   Master Node                        |
 |------------------------------------------------------|
 |  +--------------+   +------------------+            |
 |  | kube-apiserver |←→| etcd (DB)        |            |
 |  +--------------+   +------------------+            |
 |         ↓                  ↑                         |
 |  +-----------------------+                          |
 |  | kube-scheduler        | Assigns Pods            |
 |  +-----------------------+                          |
 |         ↓                                          |
 |  +----------------------------------+              |
 |  | kube-controller-manager          | Manages     |
 |  | - Node Controller                 | Cluster    |
 |  | - Replication Controller           | State     |
 |  | - Endpoint Controller               |         |
 |  +----------------------------------+              |
 +------------------------------------------------------+

 +------------------------------------------------------+
 |                   Worker Nodes (Multiple)            |
 |------------------------------------------------------|
 |  +------------+   +-----------------------+         |
 |  | kubelet    |←→ | Container Runtime      |        |
 |  |            |   | (Docker/Containerd/CRI-O)|      |
 |  +------------+   +-----------------------+         |
 |        ↓                                          |
 |  +------------+  +----------------------+        |
 |  | kube-proxy |←→| Pod (App Container)  |        |
 |  +------------+  +----------------------+        |
 +------------------------------------------------------+
```

## Explanation
- **Master Node** controls the cluster using `kube-apiserver`, `etcd`, `kube-scheduler`, and controllers.  
- **Worker Nodes** run workloads (Pods) and communicate with the Master using `kubelet`.  
- **kube-proxy** ensures network communication between services.  
- **Container Runtime** runs application containers inside Pods.  

---

