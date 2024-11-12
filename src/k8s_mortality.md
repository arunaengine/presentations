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

Overall a proper planning, configuration, and implementation can prevent these failures. For ultra-long running workloads, addressing these avoidable failures can dramatically improve reliability and reduce the operational overhead of managing these critical computations.

<!-- - Misconfigured resource requests and limits
- Inappropriate pod disruption budgets
- Suboptimal node affinity rules
- Inadequate liveness and readiness probe configurations
- Poor handling of application state and checkpoints -->

<!-- The good news is that these failures can be prevented through proper planning, configuration, and implementation. For ultra-long running workloads, addressing these avoidable failures can dramatically improve reliability and reduce the operational overhead of managing these critical computations. -->

#### Unavoidable Failures: The Forces of Nature

As already stated, unavoidable situations where a node faces critical resource pressure, leading to the eviction of pods, can happen.

But some pod failures are simply beyond our control – they're an inherent part of running distributed systems at scale. These failures include:

- Hardware Failures: From disk failures to memory errors, hardware issues can lead to pod termination.
- Infrastructure Maintenance: Necessary system updates or hardware replacements can lead to temporary pod disruptions.
- Network Partitions: Temporary or extended connectivity issues can isolate pods, leading to communication failures.
- Power-Related Incidents: From brief outages to complete data center failures, power issues can abruptly terminate pod operations.

<!-- - Hardware failures: From disk failures to memory errors
- Infrastructure maintenance: Required system updates or hardware replacements
- Network partitions: Temporary or extended connectivity issues
- Power-related incidents: From brief outages to complete data center failures
- System resource exhaustion: Host-level issues affecting all containers -->

While we can't prevent these failures, we can prepare for them. The key is not to fight their inevitability but to design systems that can gracefully handle such disruptions. This is particularly crucial for ultra-long running workloads, where each failure can mean days or weeks of lost computation.

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

### Strategies for Kubernetes Configuration

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

By understanding and accounting for these various termination scenarios, organizations can significantly improve the reliability of their ultra-long running workloads on Kubernetes. The goal isn't to eliminate pod mortality entirely – that's neither possible nor desirable in a distributed system. Instead, the focus should be on making pod terminations predictable, manageable, and minimally disruptive to critical computations.

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


### TODO:

- Node affinity
- Pod priority
- Argo workflows ?
- Exotic strategies:
- - CRIU
- - VMs / Kata containers ?


### Container Necromancy: CRIU to the Rescue

In the world of containerization, one of the trickiest tasks is dealing with ultra-long-running workloads that need to be paused, moved, or restored without losing state. Enter CRIU—Checkpoint/Restore in Userspace—a powerful tool that lets us "freeze" running processes, capturing their state (including memory and open file descriptors) and storing it on disk. Later, this state can be used to "resurrect" the process, restoring it to exactly where it left off. This technique can be especially useful when dealing with stateful workloads or when migrating processes between machines without interruption.

To leverage CRIU effectively, you’ll need a custom container setup with CRIU support (let’s call it crik), which wraps your workload and facilitates checkpointing and restoration. The process starts by checkpointing: CRIU takes a snapshot of the process, recording crucial details like its memory state, open file descriptors, and other critical attributes to persistent storage. When it's time to restore, CRIU uses these saved snapshots to rebuild the process state, including the file descriptors and other essentials, allowing the application to resume as if nothing happened.

However, the process remains manual and comes with some significant limitations when using current runtime support in CRI-O or containerd 2.0. One of the biggest challenges is that there is no standardized restore API, making this approach more complex to integrate seamlessly. Key technical hurdles include PID and TTY restoration, as well as dealing with any side effects from processes that were running at checkpoint time. For these reasons, CRIU is best used as a last resort when other native checkpointing options are unavailable.

### Summary