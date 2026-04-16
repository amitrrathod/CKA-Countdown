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

---

Deploy Your First Pod

Objective
In Kubernetes, a Pod is the smallest deployable unit. When you run a container, Kubernetes wraps it in a Pod to provide a shared network namespace and IP address. In this task, you will deploy a standalone Nginx web server as a Pod and explore its details to see how Kubernetes assigns it a unique cluster IP.

Commands
Create a new Pod named 'nginx-pod' using the lightweight Nginx Alpine image.
```kubectl run nginx-pod --image=nginx:alpine```

Check the status of your Pod. Wait until the 'STATUS' column shows 'Running'.
```kubectl get pods```

View extended details to find the internal IP address assigned to the Pod by the cluster.
```kubectl get pod nginx-pod -o wide```

Examine the Pod's lifecycle events and configuration details, including the container ID and image hash.
``` kubectl describe pod nginx-pod ```

---

---
Inspect Pod Details

Objective
A developer reported that the new 'monitor-app' pod is failing to start. Your goal is to investigate the failure using Kubernetes inspection tools. You need to identify why the container is crashing and check the internal logs to see the application's last error message before it stopped.

Commands
Create the pod using the provided manifest file.
``` kubectl apply -f monitor-pod.yaml ```

Check the status of the pod to see if it is running or crashing.
```kubectl get pods```

Inspect the pod events and container state to see why it restarted.
```kubectl describe pod monitor-app```

View the application logs to find the specific error message that caused the crash.
```kubectl logs monitor-app```
---

---
Kubernetes Pod Failure States

Your application code crashes on startup or your image registry requires credentials you forgot to provide. When these issues occur, the Kubelet on the node updates the Pod's status.phase and status.containerStatuses fields to reflect the failure.

Kubernetes uses these states to signal specific lifecycle bottlenecks within the PodStatus API object.

• ImagePullBackOff: The Kubelet cannot fetch the container image. This usually stems from a 401 Unauthorized error, a 404 Not Found for the tag, or hitting registry rate limits.
• CrashLoopBackOff: The container starts but exits with a non-zero exit code or terminates immediately. Kubernetes applies an exponential backoff delay (10s, 20s, 40s...) before restarting the container to prevent resource exhaustion.
• Pending: The Scheduler cannot place the Pod on a node. This happens when no node meets the resourceRequests (CPU/Memory) or nodeSelector constraints defined in the spec.
To diagnose these, you must inspect the Events and ExitCode fields.
---

```
# Get the high-level status
kubectl get pods

# Inspect the 'Events' section for the exact error message
kubectl describe pod <pod-name>
```

Look for these specific technical indicators in the output:

• Exit Code 1: General error; often an unhandled exception in your application code.
• Exit Code 137: The container received a SIGKILL. This typically means the OOM (Out Of Memory) Killer terminated the process because it exceeded its resources.limits.memory.
• Reason: ImagePullBackOff: Check the Events log. If you see pull access denied, verify your imagePullSecrets.
If a Pod stays Pending, check the Events for FailedScheduling. It will explicitly list how many nodes had insufficient CPU or memory.

Key Takeaway: Pod failures are reported via the PodStatus API; use kubectl describe to find the specific Exit Code or Scheduler event causing the stall.





---

