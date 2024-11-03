# KubeCon + CloudNativeCon NA 2024

## Poster: Accepting mortality: Strategies for ultra-long running stateful workloads in K8s

["Pods are mortal"](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/) - this frequently cited credo from the Kubernetes documentation has long been used to argue against running stateful workloads on Kubernetes. The ephemeral nature of pods, while perfect for resilient microservices, was supposedly unsuitable for workloads that required persistence and long-term stability.

But times have changed. Modern Kubernetes has evolved far beyond its initial focus on stateless applications. Today, running databases and other stateful workloads on K8s is not just possible – [it's becoming the norm](https://dok.community/data-on-kubernetes-2022-report/). Through redundant setups and reliable persistent volumes, the "mortality" of pods has transformed from a limitation into a manageable design consideration.

This evolution hasn't stopped at traditional stateful services. Recent years have seen Kubernetes emerging as a compelling alternative to traditional High-Performance Computing (HPC) and High-Throughput Computing (HTC) environments. Tools like Argo Workflows and Apache Airflow have made it seamless to orchestrate compute jobs on Kubernetes. For many computational workloads, pod mortality is a non-issue – if a job fails, it can simply be rescheduled.

However, a new challenge emerges as organizations port legacy applications and intensive computations from traditional HPC systems to Kubernetes. These workloads, which we term "ultra-long running," can run for days or even weeks. At these timescales, the implications of pod mortality become far more severe, raising important questions about how we design for reliability at the extreme end of the runtime spectrum.

### What Makes a Workload "Ultra-Long Running"?
Before diving deeper into the challenges and solutions, it's crucial to understand what we mean by "ultra-long running" workloads. The definition isn't as straightforward as setting a specific duration threshold – it's more nuanced and context-dependent.

In practice, what constitutes an ultra-long running workload varies significantly across different domains and use cases. For teams typically dealing with compuations that complete in seconds or minutes, a job running for several hours might qualify as ultra-long running. Conversely, in scientific computing or data analytics, where jobs commonly run for days, the threshold might be weeks or months.

A practical approach is to consider a workload ultra-long running when its duration exceeds typical workloads in your environment by one to two orders of magnitude. However, runtime isn't the only factor. The criticality and impact of failure must also be considered. Take, for example, a medical imaging analysis job processing a cancer patient's data – even if it runs for just a few days, its importance and the inability to delay results might classify it as ultra-long running from a reliability perspective. 

This highlights that the classification of ultra-long running workloads depends not just on duration, but on a combination of runtime, resource investment, and business or scientific impact. When a computation represents weeks of progress, or when results are tied to critical decisions or deadlines, the stakes of pod mortality become significantly higher.

### Understanding Pod Mortality: The Inevitable and the Preventable

With a clear understanding of ultra-long running workloads, we need to examine why pods die in the first place. Pod failures fall into two distinct categories: unavoidable and avoidable. Understanding this distinction is crucial for developing effective strategies to enhance reliability for these critical workloads.

#### Unavoidable Failures: The Forces of Nature
Some pod failures are simply beyond our control – they're an inherent part of running distributed systems at scale. These unavoidable failures include:

- Hardware failures: From disk failures to memory errors
- Infrastructure maintenance: Required system updates or hardware replacements
- Network partitions: Temporary or extended connectivity issues
- Power-related incidents: From brief outages to complete data center failures
- System resource exhaustion: Host-level issues affecting all containers

While we can't prevent these failures, we can prepare for them. The key is not to fight their inevitability but to design systems that can gracefully handle such disruptions. This is particularly crucial for ultra-long running workloads, where each failure can mean days or weeks of lost computation.

#### Avoidable Failures: The Human Factor

The larger category – and the one where we have the most control – involves avoidable failures. These stem from configuration issues, design oversights, or misalignment between tools and requirements. Common sources include:

- Misconfigured resource requests and limits
- Inappropriate pod disruption budgets
- Suboptimal node affinity rules
- Inadequate liveness and readiness probe configurations
- Poor handling of application state and checkpoints

The good news is that these failures can be prevented through proper planning, configuration, and implementation. For ultra-long running workloads, addressing these avoidable failures can dramatically improve reliability and reduce the operational overhead of managing these critical computations.