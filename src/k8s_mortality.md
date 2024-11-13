# KubeCon + CloudNativeCon NA 2024

## Poster: Accepting mortality: Strategies for ultra-long running stateful workloads in K8s

### Table of Contents

1. [Introduction](#introduction)
2. [What Makes a Workload "Ultra-Long Running"?](#what-makes-a-workload-ultra-long-running)
3. [Understanding Pod Mortality: The Inevitable and the Preventable](#understanding-pod-mortality-the-inevitable-and-the-preventable)
4. [Selecting the Right Tool](#selecting-the-right-tool)
5. [Strategies for Kubernetes Configuration](#strategies-for-kubernetes-configuration)
6. [A Practical Example](#a-practical-example)
7. [Container Necromancy: CRIU to the Rescue!](#container-necromancy-criu-to-the-rescue)
8. [Summary](#summary)

### Introduction

["Pods are mortal"](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/) - this frequently cited credo from the Kubernetes documentation has long been used to argue against running stateful workloads on Kubernetes. The ephemeral nature of pods, while perfect for resilient microservices, was supposedly unsuitable for workloads that required persistence and long-term stability.

But times have changed. Modern Kubernetes has evolved far beyond its initial focus on stateless applications. Today, running databases and other stateful workloads on K8s is not just possible – [it's becoming the norm](https://dok.community/data-on-kubernetes-2022-report/). Through redundant setups and reliable persistent volumes, the "mortality" of pods has transformed from a limitation into a manageable design consideration.

This evolution hasn't stopped at traditional stateful services. Recent years have seen Kubernetes emerging as a compelling alternative to traditional High-Performance Computing (HPC) and High-Throughput Computing (HTC) environments. Tools like Argo Workflows and Apache Airflow have made it seamless to orchestrate compute jobs on Kubernetes. For many computational workloads, pod mortality is a non-issue – if a job fails, it can simply be rescheduled.

However, a new challenge emerges as organizations port legacy applications and intensive computations from traditional HPC systems to Kubernetes. These workloads, which we term "ultra-long running," can run for days or even weeks. At these timescales, the implications of pod mortality become far more severe, raising important questions about how we design for reliability at the extreme end of the runtime spectrum.

### What Makes a Workload "Ultra-Long Running"?
Before diving deeper into the challenges and solutions, it's crucial to understand what we mean by "ultra-long running" workloads. The definition isn't as straightforward as setting a specific duration threshold – it's more nuanced and context-dependent.

In practice, what constitutes an ultra-long running workload varies significantly across different domains and use cases. For teams typically dealing with computations that complete in seconds or minutes, a job running for several hours might qualify as ultra-long running. Conversely, in scientific computing or data analytics, where jobs commonly run for days, the threshold might be weeks or months.

A practical approach is to consider a workload ultra-long running when its duration exceeds typical workloads in your environment by one to two orders of magnitude. However, runtime isn't the only factor. The criticality and impact of failure must also be considered. Take, for example, a medical imaging analysis job processing a cancer patient's data – even if it runs for just a few days, its importance and the inability to delay results might also be an important factor where mortality becomes a critical problem. 

This highlights that the classification of ultra-long running workloads depends not just on duration, but on a combination of runtime, resource investment, and business or scientific impact. When a computation represents weeks of progress, or when results are tied to critical decisions or deadlines, the stakes of pod mortality become significantly higher.

### Understanding Pod Mortality: The Inevitable and the Preventable

With a clear understanding of ultra-long running workloads, we need to examine why pods die in the first place. Pod failures fall into two distinct categories: avoidable and unavoidable. Understanding this distinction is crucial for developing effective strategies to enhance reliability for these critical workloads.

#### Avoidable Failures: The Human Factor

The larger category – and the one where we have the most control – involves avoidable failures. These stem from configuration issues, design oversights, or misalignment between tools and requirements. Common sources include:

- Preemption: Higher-priority pods can preempt lower-priority ones, leading to their termination.
- API-Initiated Eviction: Kubernetes may evict pods based on API decisions, such as during scaling events or policy enforcement.
- Misconfigured Resource Requests and Limits: Setting inappropriate resource allocations can lead to pod instability or inefficient resource usage.
- Node Pressure: Resource exhaustion at the node level, such as CPU or memory shortages, can force the eviction of pods. Although this can become an unavoidable failure, it is often preventable through proper resource management.
- Configuration issues on a workload level: A wrong parameters or malformed input data, can also cause failures during execution.

Overall a proper planning, configuration, and implementation can prevent these failures. For ultra-long running workloads, addressing these avoidable failures can dramatically improve reliability and reduce the operational overhead of managing these critical computations.

#### Unavoidable Failures: The Forces of Nature

As already stated, unavoidable situations where a node faces critical resource pressure, leading to the eviction of pods, can happen.

But some pod failures are simply beyond our control – they're an inherent part of running distributed systems at scale. These failures include:

- Hardware Failures: From disk failures to memory errors, hardware issues can lead to pod termination.
- Infrastructure Maintenance: Necessary system updates or hardware replacements can lead to temporary pod disruptions.
- Network Partitions: Temporary or extended connectivity issues can isolate pods, leading to communication failures.
- Power-Related Incidents: From brief outages to complete data center failures, power issues can abruptly terminate pod operations.

While we can't prevent these failures, we can prepare for them. The key is not to fight their inevitability but to design systems that can gracefully handle such disruptions. This is particularly crucial for ultra-long running workloads, where each failure can mean days or weeks of lost computation.

## Strategies for Success

### Selecting the Right Tool

While it might seem obvious, tool selection is often neglected in favor of focusing on infrastructure configuration and deployment strategies. Yet this foundational decision can make or break your success when running workloads that span days or weeks on Kubernetes. The right tool isn't just about features or performance metrics – it's about finding technology that aligns with the unique demands of ultra-long running workloads. Let's explore three critical characteristics that separate the ideal tools from the merely adequate ones.

#### Built-in State Management

Robust state management serves as the foundation for any reliable stateful workload. While it's possible to implement checkpointing externally, tools that provide this functionality out of the box offer significant advantages.

A well-designed state management system should automatically save the program's current state at configurable intervals without significant performance impact. The best tools compress these checkpoints efficiently and maintain multiple versions, allowing you to fall back to previous states if corruption occurs. Look for tools that make checkpointing transparent to your code while still giving you control over the frequency and storage location of checkpoints. Ideally these should have the option to automatically pick up existing checkpoints when restarted and start from the latest valid checkpoint. 

#### Resource Efficiency and Predictability

When workloads extend over days or weeks, resource efficiency becomes critical to success. Memory management stands out as particularly crucial, as RAM constraints and out-of-memory events frequently cause pod failures in Kubernetes. Tools exhibiting erratic memory patterns or unexpected usage spikes pose significant risks, often forcing teams to dramatically overprovision resources as a precaution.  The optimal solution should demonstrate consistent resource utilization and support configurable internal memory boundaries that respect defined limits.

Consider runtime efficiency through a practical lens – a tool that completes tasks in three days with steady resource consumption often proves more valuable than one promising faster completion but demanding unpredictable resource spikes. 
If runtime flexibility is possible, tools that allow adjustments between precision and performance should be evaluated to find a balance that suits your needs. Ultimately, shorter runtimes are beneficial, but only when they maintain steady resource consumption and operational reliability over extended periods.

#### Kubernetes Integration

Choose tools that handle pod lifecycle events gracefully and support Kubernetes features like pod disruption budgets. Good integration includes proper termination handling, accurate health probes, and comprehensive monitoring capabilities. The tool should provide clear documentation for Kubernetes deployment and maintain stability over extended running periods. Remember that for ultra-long running workloads, operational reliability typically matters more than raw performance.

While selecting the right tool is crucial, even the most suitable tool requires proper Kubernetes configuration to perform optimally. In the next strategy, we'll explore essential Kubernetes configurations that help minimize pod mortality and maximize reliability. We'll examine how to set up resource quotas, configure pod disruption budgets, and implement other critical settings that create a robust environment for ultra-long running workloads.

### Kubernetes Configuration Best Practices

Effective workload management in Kubernetes begins with strategic priority configuration. Through a well-designed priority system, you can safeguard critical workloads' resource availability while maintaining overall cluster health. While the [Kubernetes documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/) provides comprehensive examples, here's a practical overview.

#### Understanding PriorityClass
PriorityClass is a cluster-wide resource that defines the importance of Pods relative to other Pods. This becomes crucial in two key scenarios:

- Resource Contention: When the cluster is under pressure, Kubernetes uses priority to determine which Pods should be evicted first
- Scheduling Decisions: Higher priority Pods get preferential treatment during scheduling

Here's an example of a high-priority configuration:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "High priority class for critical production workloads"
preemptionPolicy: PreemptLowerPriority  # Optional: explicitly state preemption behavior
```

##### Key Fields Explained:

- `value`: Determines the relative priority (higher numbers mean higher priority)
- `globalDefault`: When true, automatically assigns this priority to Pods without a specified PriorityClass
- `preemptionPolicy`: Controls whether Pods with this priority can preempt lower-priority Pods

##### Implementing Priority in Workloads
To associate a Pod with a PriorityClass, add the `priorityClassName` field to the Pod specification:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-app
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: your-app:latest
```

#### Managing Planned Disruptions with PodDisruptionBudgets

While PriorityClasses handle unexpected resource constraints, [PodDisruptionBudgets (PDBs)](https://kubernetes.io/docs/tasks/run-application/configure-pdb/) help manage planned maintenance activities. PDBs ensure that a specified number of pods remain available. For long-running jobs where continuity is critical, you can configure strict protection like this:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: no-disruption-allowed
spec:
  minAvailable: 100%
  selector:
    matchLabels:
      app: critical-long-running-job
```

This will add a pod disruption budget to all pods that match a specific label.

#### Configuring Resources and Requests

Resource management in Kubernetes is often misunderstood. While many guides suggest setting both requests and limits for all resources, the reality is more nuanced. The key to effective resource management lies in understanding how different resources behave and how your workloads consume them. Let's dive into how you can make better decisions about resource configuration.

#### The Foundation

At its core, Kubernetes uses resource requests and limits to manage how pods consume cluster resources. Requests represent the guaranteed resources for a pod, while limits set the maximum. This system helps Kubernetes make intelligent scheduling decisions and determines the order of pod eviction when nodes are under pressure.

#### A Tale of Two Resources

CPU and memory behave fundamentally differently in Kubernetes, and treating them the same way can lead to suboptimal performance. Let's explore why.
CPU is a compressible resource. When a container needs more CPU than is available, it simply has to wait. Think of it like a queue at a coffee shop – if there are more customers than baristas, people wait their turn. This means we can safely overprovision CPU without causing catastrophic failures. In fact, CPU throttling often causes more problems than it solves, which is why we recommend against setting CPU limits for most workloads.

Memory, on the other hand, tells a different story. Unlike CPU, memory can't be compressed or throttled. When a container hits its memory limit, it's terminated – imagine a cup that overflows when it's too full. This abrupt behavior means we need to be especially thoughtful about how we handle memory limits.

#### Quotas for Long Runners

Resource quotas for Long-running workloads need to be treated with particular care. These are the marathon runners of your cluster – they need to maintain their pace over long periods and handle occasional sprints.
For CPU, we've found that setting requests at 1.5 times the expected usage provides a comfortable buffer for traffic spikes. We intentionally omit CPU limits for these services. This might seem counterintuitive, but it actually improves overall performance by allowing services to use available CPU when they need it without artificial throttling.
Memory configuration for long-runners follows a similar pattern. We set memory requests at 1.5 times the expected base usage, but here's the key insight: we don't set memory limits. This might make some administrators nervous, but it's a calculated decision. Long-running workloads often have complex memory patterns, and setting limits can lead to unexpected terminations that impact pod mortality.

#### Short-Running Pods: The Sprinters

Short-running pods and batch processes have different needs. These are your cluster's sprinters – they need resources for a brief, often predictable period. For these workloads, we can be more precise with CPU requests, matching them closely to expected usage. The workload's predictable nature means we can be more confident in our resource estimates.

Memory configuration for short-running jobs takes an interesting approach. We actually set memory requests lower than we expect to need – about 80% of the expected usage. This intentional underprovision means these pods are more likely to be evicted when memory runs tight, which is exactly what we want for short-running jobs that can be rescheduled.

We do set memory limits for these jobs, targeting a value that covers 90-95% of executions. This prevents runaway jobs while allowing for most normal variations in memory usage.

### Example in Practice

Consider this configuration for a long-running service:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: long-runner
spec:
  containers:
  - name: app
    image: podimage
    resources:
      requests:
        cpu: "1500m"      # 1.5x expected usage
        memory: "1500Mi"  # 1.5x expected base memory
```

Notice the absence of limits – this is intentional. Now compare it to a short-running job:

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: short-runner
spec:
  containers:
  - name: app
    image: podimage
    resources:
      requests:
        cpu: "500m"       # Matched to expected usage
        memory: "400Mi"   # 80% of expected usage
      limits:
        memory: "512Mi"   # Covers 95% of executions
```

#### The Path Forward

Effective resource management isn't a set-and-forget task. It requires ongoing monitoring and adjustment. Watch how your workloads behave, analyze their resource usage patterns, and adjust your configurations accordingly. Pay attention to pod eviction events and out-of-memory kills – they're valuable signals that your resource configuration might need tweaking.

Remember that these guidelines are starting points, not hard rules. Your specific workloads might have different needs, and that's okay. The key is understanding the principles behind these decisions and applying them thoughtfully to your unique situation.

By treating CPU and memory differently, and by distinguishing between long-running services and short-running jobs, you can create a more stable and efficient Kubernetes cluster that better serves your applications' needs.

### The Art of Pod Placement

While resource management controls how much of the cluster your workloads can consume, affinity rules control where these workloads run. These placement strategies are crucial for both performance and reliability. Let's explore how to use node affinity and pod anti-affinity to implement sophisticated scheduling strategies.

#### Node Affinity: Choosing the Right Neighborhood

Node affinity is like choosing the right neighborhood for your home. Sometimes you want to live near specific amenities, and sometimes you want to avoid certain areas altogether. In Kubernetes terms, this means selecting nodes with specific characteristics for your workloads.

Consider a machine learning workload that requires GPU acceleration. Rather than hoping it lands on the right node, we can explicitly specify this requirement:


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: highmem-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: mem-type
            operator: In
            values:
            - highmem
  containers:
  - name: highmem-container
    image: highmem-workload
```

But node affinity isn't just about hardware requirements. We often use it to ensure workloads land on nodes with specific characteristics. For instance, you might want to ensure your production workloads only run on nodes with premium storage or specific networking capabilities:

```yaml
nodeAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
  - weight: 100
    preference:
      matchExpressions:
      - key: storage-type
        operator: In
        values:
        - premium-ssd
```

Notice we used preferred instead of required here. This tells Kubernetes "try your best to place this pod on nodes with premium storage, but if you can't, that's okay." This flexibility can be crucial for maintaining service availability during cluster changes.

#### Pod Anti-Affinity: Maintaining Safe Distances

While node affinity helps us choose where to run our pods, pod anti-affinity helps us keep them apart. This is crucial for high availability – you don't want all instances of your critical service running on the same node.

Think of pod anti-affinity like planning store locations: you generally don't want all your stores in the same neighborhood. Here's how we might configure a web service to spread across nodes:

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: long-runner
spec:
  replicas: 1
  template:
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - long-runner
            topologyKey: "kubernetes.io/hostname"
```

This configuration ensures that no two pods of our web-server will run on the same node. But sometimes this strict requirement can be counterproductive. For instance, during a rolling update or when scaling up rapidly, you might want to allow temporary co-location of pods. In such cases, we can use preferred anti-affinity:

```yaml

podAntiAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
  - weight: 100
    podAffinityTerm:
      labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - web-server
      topologyKey: "kubernetes.io/hostname"
```

<!-- 
Properly configured resource requests and limits are essential for cluster stability. Requests represent the guaranteed resources for a pod, while limits set the maximum. This helps Kubernetes make better scheduling decisions and has an effect on eviction order when NodePressure occurs.

It is important to know that CPU and Memory are different topics. While overprovisioning CPU and using more CPU than available cores is possible. Exceeding the memory limits is a hard limit and the node will evict pods wenn overall memory runs low. This leads to some crucial design decisions for our long running pods and other shorter pods. We do not want to have a CPU resource limit. This often leads to unnecessary throttling and often results in an overall lower performance. In terms of CPU requests this depends on the workload. For the shorter running jobs we want to ideally choose the exact CPU we are expecting the workload to use. For our long runners that often have a significant variability we want to give them a 1.5x overprovision to buffer some CPU spikes.

Memory on the otherhand is a different picture, exceeding memory limits will result in a pod kill. To prevent this kill option we do also not specify any limits for our long runners. For our short running pods we do want to have limits that ideally should cover 90-95% of all executions. 
Requests are also a different story, we want our short running pods to exceed their memory requests on a regular basis. This gives them a higher chance of getting evicted when memory runs low. So in our tests we choose to give our long runners a 1.5x overprovision and our short runners a ~20% underprovision. 


Resource management requires careful consideration. Set resource requests based on actual usage patterns, and think carefully about memory limits, especially for workloads prone to usage spikes. Node affinity rules ensure proper workload distribution across your cluster.
Monitoring plays a vital role in preventing failures. Implement robust monitoring to detect potential issues before they cause disruption, and maintain proper redundancy and backup strategies. Distributing workloads across failure domains adds an extra layer of protection against infrastructure issues.

Understanding pod mortality in Kubernetes requires a deep dive into the various mechanisms that can terminate a pod. While some terminations are intentional and controlled, others occur as emergency responses to system stress. Let's explore the core reasons for pod termination and how to mitigate them effectively.

Preemption serves as Kubernetes' method for ensuring critical workloads get the resources they need. The scheduler may terminate lower-priority pods to make room for those deemed more important. Fortunately, this common cause of pod mortality has a straightforward solution: assigning appropriate PriorityClass values to your ultra-long running workloads. By setting higher priority values, you can protect these crucial computations from preemption by less critical workloads.

Node pressure eviction represents a more challenging scenario. When a kubelet detects resource exhaustion at the node level, it initiates emergency procedures to protect the node's stability. These pressures typically manifest in three ways: memory exhaustion (MemoryPressure), storage constraints (DiskPressure), or process limitations (PIDPressure). What makes node pressure particularly troublesome is that kubelet will bypass PodDisruptionBudget configurations in these scenarios – the need to protect node stability takes precedence over workload preservation.

Understanding the eviction selection process becomes crucial when node pressure occurs. Kubernetes follows a specific hierarchy: first targeting pods that exceed their resource requests, then considering pod priority levels, and finally examining resource usage relative to requests. This selection process emphasizes the importance of accurate resource requests and for ultra-long running workloads.

API-initiated evictions present a more controlled scenario. These administrative actions, while potentially disruptive, can be effectively managed through properly configured PodDisruptionBudgets. By setting appropriate budgets, you can ensure that manual interventions and cluster maintenance activities don't compromise your critical workloads.

Resource limit violations, particularly memory limits, constitute another common cause of pod termination. When a pod exceeds its configured memory limits, Kubernetes terminates it to protect the node's stability. This highlights the delicate balance required when setting resource limits – they must be high enough to accommodate legitimate usage patterns while preventing runaway resource consumption.

Hardware failures round out the primary causes of pod mortality. While impossible to prevent entirely, their impact can be mitigated through proper redundancy and backup strategies. This includes distributing workloads across failure domains and maintaining regular checkpoints of computation progress.

The key to managing these various mortality factors lies in a comprehensive configuration strategy. This involves:

- Setting appropriate resource requests and limits that reflect actual usage patterns
- Implementing PodDisruptionBudgets to protect against voluntary disruptions
- Configuring PriorityClasses to prevent preemption of critical workloads
- Establishing node affinity rules to ensure proper workload distribution
- Implementing robust monitoring to detect potential issues before they cause termination

By understanding and accounting for these various termination scenarios, organizations can significantly improve the reliability of their ultra-long running workloads on Kubernetes. The goal isn't to eliminate pod mortality entirely – that's neither possible nor desirable in a distributed system. Instead, the focus should be on making pod terminations predictable, manageable, and minimally disruptive to critical computations. -->

### A practical example

#### Our workload(s)
In our setup we are running a diverse set of workflows from computational biology. These include primarily the analysis and processing of DNA and RNA sequences from various sources. Most of these workflows consist of a combination of long-running steps and (sometimes in parallel) shorter running steps in between. Our goal is to make the most efficient use of the available hardware while maximizing the success rate of our long running workloads. Some of these applications, large metagenomic assemblies for example can take weeks or months to complete on high performance hardware using 100+ CPU cores and up-to terabytes of RAM.

It is important to note that these assembly jobs do not always use the full resources and often have long spans that take either a lot of CPU resources or Memory.

#### Workload configuration: Request and Limits

Through extensive measurement we have a quite good estimate of the average and minimum / maximum resource usage of small and large jobs. As a baseline we configure all workloads to use the average CPU / RAM as requests, due to the memory spikes we do give our ultra-long-running pods around 1.5x memory above average to decrease the likelihood of eviction when MemoryPressure occurs.

The long-running workloads have no resource limits. CPU limits do have a significant runtime impact. Memory limits while in theory possible we do want to give these jobs the option to consume almost all memory of the node. All shorter / smaller jobs do not have limits.

Before we dive into specific strategies let us first explore the knobs and dials someone can turn to modify the default pod behavior. Five reasons exist why a pod in your kubernetes environment can get killed: Preemption, Node pressure eviction, API-initiated Eviction, Exceeding resource limits and external factors (hardware failures, external sigkill signals etc.).

Preemption is the selective eviction of pods by the kubernetes scheduler to make room for pods with a higher priority, thus a very low hanging fruit is assigning your long running jobs a high priority. This can be done via priority classes.

Node pressure eviction occurs when kubelet detects some resource exhaustion on the hardware level, these eviction signals can be memory (MemoryPressure), file system related (DiskPressure) or pid related (PIDPressure). It is important to note that kubelet does not respect your configured PodDisruptionBudget and will kill your pod to remove the exhaustion state.

Pod selection at eviction: The pods that exceed their resource requests are evicted first, afterwards the pods are evicted sorted by pod priority, lastly by total resource usage relative to requests.

The third reason for mortality is api-initiated eviction. Someone initiated the eviction, these can effectively be removed via pod disruption budgets.

The fourth reason is exceeding the configured memory limits. If a pod exceeds its memory limits it will get killed.

The last reason are hardware failures, as said in the previous sections you cannot effectively guarantee that they will not occur.

Before addressing more technical topics how we can configure a kubernetes cluster to be as reliable as possible let me first talk about an obvious yet sometimes unexpected topic: Choosing the correct tool. For most tasks multiple options exist, all with their specific benefit and drawbacks. When a decision must be made between these options you can factor in some important characteristics for being better suited for a long running environment.

1. Choose a tool with built-in checkpoints:
   Built-in checkpointing and persistence to disk can make it way easier to recover from an unavoidable failure. While there are strategies to make programs without built-in checkpoints checkpointable (more on that later), it should only be treated as last resort and not the preferred way to make checkpoint of your program.

2. Choose a tool with lower runtime. Sounds obvious but sometimes there are other factors that play into role here like the precision of the results etc. Always ask yourself: Is it really needed to have the highest precision ? Or does a more broader workflow with a little bit less precision but a drastically decreased runtime also work ?

3. Prefer tools that use less RAM and or have a predictable RAM allocation. One of the hardest things to manage for ultra-long-running workloads is RAM. Especially if the RAM has some high spikes during the runtime it gets very frustrating and sometime not manageable in a cost and time effective way because crazy overprovisioning would be needed to cope with the RAM requirements. The fact is: RAM cannot be shared, so it is crucial to think about this in your considerations. The overall guideline should be: The less RAM usage and the more consistent the RAM usage the better.

### Container Necromancy: CRIU to the Rescue

In the world of containerization, one of the trickiest tasks is dealing with ultra-long-running workloads that need to be paused, moved, or restored without losing state. Enter CRIU—Checkpoint/Restore in Userspace—a powerful tool that lets us "freeze" running processes, capturing their state (including memory and open file descriptors) and storing it on disk. Later, this state can be used to "resurrect" the process, restoring it to exactly where it left off. This technique can be especially useful when dealing with stateful workloads or when migrating processes between machines without interruption.

To leverage CRIU effectively, you’ll need a custom container setup with CRIU support (let’s call it crik), which wraps your workload and facilitates checkpointing and restoration. The process starts by checkpointing: CRIU takes a snapshot of the process, recording crucial details like its memory state, open file descriptors, and other critical attributes to persistent storage. When it's time to restore, CRIU uses these saved snapshots to rebuild the process state, including the file descriptors and other essentials, allowing the application to resume as if nothing happened.

However, the process remains manual and comes with some significant limitations when using current runtime support in CRI-O or containerd 2.0. One of the biggest challenges is that there is no standardized restore API, making this approach more complex to integrate seamlessly. Key technical hurdles include PID and TTY restoration, as well as dealing with any side effects from processes that were running at checkpoint time. For these reasons, CRIU is best used as a last resort when other native checkpointing options are unavailable.

### Summary