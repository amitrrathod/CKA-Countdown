Kubernetes Pods and Self-Healing Essentials

You will learn how Kubernetes manages its smallest unit: the Pod. We will cover how to create them, 
what happens when they fail, and how to bring them back to life. You will also see how Kubernetes detects health issues and fixes them automatically.

1 Pod Fundamentals
1 concept
2 hands-on
1 quiz

Learn what a Pod is and how to run your first container in the cluster.

- What is a Pod?
- Pod Architecture Quiz
- Deploy Your First Pod
- Inspect Pod Details

2 Handling Failure States
1 concept
2 hands-on
1 quiz

Identify common Pod errors and learn how to troubleshoot 'down' pods.

- Common Pod Failure States
- Fix a Broken Image
- Troubleshooting Logic
- Debug a Crashing App

3 Discovery and Self-Healing
1 concept
2 hands-on
1 quiz

Use Liveness Probes to let Kubernetes automatically restart unhealthy Pods.

- Introduction to Liveness Probes
- Configure a Liveness Probe
- Watch Auto-Healing in Action
- Self-Healing Best Practices

The Pod: Kubernetes' Atomic Unit
Linux containers share a network namespace and filesystem only if you manually configure cgroups and namespaces. If you run a web server and a log scraper as separate containers, they cannot communicate via localhost. You have to manage their lifecycles separately, which leads to "zombie" processes when one container dies and its helper keeps running without a purpose.

The Pod solves this by grouping containers into a single execution context. It acts as a wrapper that shares the Network and UTS namespaces and allows shared Volumes.

• Shared IP: All containers in a Pod share the same network interface and IP address.
• IPC Namespace: Containers can communicate using SystemV IPC or POSIX message queues.
• Atomic Scheduling: Kubernetes schedules the entire Pod onto a single Node; it never splits containers across different hosts.

**Inside the Pod Specification **
When you define a Pod, you are configuring the Pause container, also known as the sandbox container. This container holds the namespaces open even if your application containers crash.

```
apiVersion: v1
kind: Pod
metadata:
  name: ghost-stack
spec:
  containers:
  - name: ghost
    image: ghost:latest
    ports:
    - containerPort: 2368
  - name: logger
    image: fluentd:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
  volumes:
  - name: shared-logs
    emptyDir: {}
```

In this setup, the logger container accesses files from ghost via the shared-logs volume. Because they share the network namespace, the logger can also reach the ghost API at 127.0.0.1:2368. If the Node runs out of memory, the Kubelet kills the entire Pod unit based on its Quality of Service (QoS) class, which keeps your multi-container logic in sync.

Key Takeaway: Pods provide the shared Linux namespaces (Network, IPC, UTS) required for tightly coupled containers to function as a single logical host.

---

You have a Pod running a Python web app and a Redis sidecar. The web app is configured to connect to Redis at '127.0.0.1:6379', but your team is worried the connection will fail because they are two different containers. Why does this setup work in a Kubernetes Pod?

B) The containers share the same Network namespace, allowing them to communicate over localhost.
D) Containers in a Pod use the IPC namespace to bridge different IP addresses together.
A) Kubernetes automatically creates a ClusterIP service to link containers within the same Pod.
C) The Kubelet acts as a proxy to route traffic between container ports on the same Node.

Correct Answer - “B) The containers share the same Network namespace, allowing them to communicate over localhost.”
This is exactly how it works. Since they share the same network stack and IP, they see each other on localhost just like processes on a single VM.

---


