Virtualization to use for Kubernetes Cluster on Home Servers

# Context

The goal is to effectively utilize three old servers, each with 64GB of memory and 4 cores, for self-hosting applications using Kubernetes. An evaluation of virtualization technologies is required to determine the best solution for deploying and managing Kubernetes in a home lab environment.
Decision Drivers

Hardware Utilization: Efficient use of older hardware capabilities.
Cost: Preference for free or cost-effective solutions.
Ease of Setup and Management: User-friendly, with robust management tools.
Flexibility and Scalability: Ability to adapt and scale with the evolving needs of home server applications.

## Considered Options

### Option 1: Proxmox

Description: An open-source virtualization management solution that supports VM and container-based virtualization via LXC and has robust storage and network management capabilities.

Pros:
+ Completely free and open-source, with a strong community for support.
+ Intuitive web-based management interface simplifies complex processes.
+ Supports LXC for lightweight containerization and full KVM for virtua+ machines.
+ High compatibility with various Linux distributions, enhancin+ flexibility.
+ Can easily deploy Kubernetes either on VMs or directly insid+ containers.

Cons:
- Slightly steeper learning curve for complete beginners compared t- solutions like Minikube.

### Option 2: VMware ESXi with Tanzu

Description: VMware’s server virtualization platform with integrated Kubernetes management through Tanzu.

Pros:
+ High stability and extensive features.
+ Integrated Kubernetes support.

Cons:
- High cost due to licensing.
- Potentially excessive resource overhead for older hardware.

### Option 3: KVM (Kernel-based Virtual Machine) with Minikube

Description: A free and open-source virtualization solution built into Linux, paired with Minikube for Kubernetes.

Pros:

+ Minimal overhead and cost.
+ Direct integration with the Linux kernel.

Cons:

- More manual setup and lacks an integrated management interface.
- Less intuitive for those not familiar with Linux administration.

### Option 4: Docker Desktop with Kubernetes

Description: Provides a Docker-based environment with an integrated Kubernetes option.

Pros:
+ Simple to set up and use for Docker and Kubernetes.

Cons:
- Not suitable for larger or more complex deployments.
- Recent changes in licensing and performance issues on older hardware.

### Decision Outcome

Chosen Option: Proxmox. The decision was made due to its balance of features, cost-effectiveness, and robust management capabilities. Proxmox provides a comprehensive platform for both container and virtual machine management, making it ideal for a versatile and scalable home server setup.

### Status

Accepted

### Consequences

Proxmox will require some initial learning and setup, but resources and community forums provide ample support.
The flexibility of Proxmox allows for both VM-based and container-based Kubernetes deployments, catering to different needs and optimizations.
Proxmox's comprehensive toolset will enable detailed resource monitoring and management, maximizing the performance of older server hardware.