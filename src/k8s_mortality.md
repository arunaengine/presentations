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