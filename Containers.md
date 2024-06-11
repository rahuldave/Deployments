# Containers

### History of Virtualization and Containerization

#### Virtualization

**1960s - Early Concepts:**

- **IBM CP-40 and CP-67:** IBM's research in the 1960s led to the development of the CP-40 and later CP-67, which introduced the concept of virtualization on mainframe computers. These systems allowed multiple users to share the same hardware by creating separate virtual machines (VMs).

**1990s - Commercial Virtualization:**

- **VMware (1998):** VMware was founded and became a pioneer in x86 virtualization. In 1999, VMware introduced VMware Workstation, allowing multiple operating systems to run on a single physical machine. This was a significant advancement for testing and development environments.

**2000s - Widespread Adoption:**

- **VMware ESX (2001):** VMware released ESX Server, a bare-metal hypervisor that provided robust performance and efficiency for enterprise virtualization.
- **Microsoft Hyper-V (2008):** Microsoft entered the virtualization market with Hyper-V, integrated into Windows Server, making virtualization more accessible to Windows-based environments.

**Virtualization Concepts:**

- Hypervisors:

   

  Software that creates and manages virtual machines. Two types:

  - **Type 1 (Bare-metal):** Runs directly on hardware (e.g., VMware ESXi, Microsoft Hyper-V).
  - **Type 2 (Hosted):** Runs on top of a host operating system (e.g., VMware Workstation, VirtualBox).

#### Containerization

**2000s - Early Developments:**

- **Chroot (1979):** Unix introduced `chroot`, a system call that changes the root directory for a process, providing a basic form of isolation.
- **Solaris Containers (2005):** Solaris introduced containers (zones) that provided isolated environments for running applications.

**2008 - 2013 - Linux Container Advancements:**

- **Linux Containers (LXC):** Linux kernel features like namespaces and control groups (cgroups) were utilized to create lightweight virtual environments. LXC provided a userspace interface for container management.
- **Docker (2013):** Docker revolutionized containerization by introducing an easy-to-use platform for creating, deploying, and managing containers. Docker provided tools for building container images, a registry for sharing them, and a runtime for running containers.

**2014 - Present - Ecosystem Growth:**

- **CoreOS (2014):** Introduced rkt (Rocket) as an alternative container runtime to Docker, emphasizing security.
- **Kubernetes (2014):** Google open-sourced Kubernetes, a container orchestration platform that automated deployment, scaling, and management of containerized applications. It became the de facto standard for container orchestration.
- **Open Container Initiative (OCI) (2015):** Founded by Docker and other industry leaders to establish open standards for container formats and runtimes.
- **Docker Inc. and Moby (2017):** Docker refocused on its commercial products, while the open-source components were reorganized under the Moby project.

### Why Containerization is Better than Virtualization

**Efficiency and Performance:**

- **Lower Overhead:** Containers share the host OS kernel, leading to lower overhead compared to VMs that each require a full OS instance.
- **Faster Startup:** Containers can start and stop in seconds, whereas VMs take minutes to boot up.

**Resource Utilization:**

- **Higher Density:** More containers can be run on a single host compared to VMs, improving resource utilization.
- **Scalability:** Containers are lightweight and can be easily scaled up or down, making them ideal for microservices architectures.

**Portability and Consistency:**

- **Write Once, Run Anywhere:** Container images bundle applications with their dependencies, ensuring consistent behavior across different environments.
- **Isolation:** Containers provide strong isolation between applications, reducing conflicts and making it easier to deploy multiple applications on the same host.

**DevOps and CI/CD:**

- **Simplified Deployment:** Containers streamline the deployment process, allowing developers to package applications and dependencies together.
- **Integration with CI/CD:** Tools like Docker integrate well with CI/CD pipelines, automating testing, deployment, and rollback processes.

**Ecosystem and Tooling:**

- **Rich Ecosystem:** The container ecosystem includes robust tools for orchestration (Kubernetes), monitoring (Prometheus), logging (ELK stack), and networking (CNI plugins).
- **Community and Support:** A large and active community supports continuous improvement and innovation in container technologies.

### Conclusion

Virtualization laid the groundwork for efficient resource utilization and application isolation, but containerization has taken these concepts further by offering lightweight, portable, and highly efficient environments. The development of containerization tools like Docker and orchestration platforms like Kubernetes has driven widespread adoption, making containerization a cornerstone of modern application deployment and management.



## Containerization: How does it work?



Containerization on Linux leverages several kernel features to provide isolation, resource management, and efficient application deployment. The core technologies behind containerization are namespaces, cgroups (control groups), and filesystem layers. Here’s a detailed look at how these components work together to achieve containerization:

### Linux Namespaces

Namespaces are a feature of the Linux kernel that provide isolation of global system resources between different processes. Each namespace wraps a global system resource in an abstraction that makes it appear to the processes within the namespace as if they have their own isolated instance of that resource. There are several types of namespaces:

1. **PID Namespace:**
   - Isolates process IDs. Processes in different PID namespaces can have the same PID without conflicting.
   - Allows a container to have its own PID 1 (init process), enabling independent process trees.
2. **NET Namespace:**
   - Isolates network resources such as network interfaces, IP addresses, routing tables, and ports.
   - Each container can have its own network stack, including virtual network interfaces, IP addresses, and routing rules.
3. **MNT Namespace:**
   - Isolates mount points. Each container can have its own filesystem mount points, separate from the host and other containers.
   - Provides the ability to create private mounts, making it possible to control what parts of the filesystem are accessible to the container.
4. **UTS Namespace:**
   - Isolates hostname and domain name. Each container can have its own hostname and NIS domain name.
5. **IPC Namespace:**
   - Isolates inter-process communication resources, such as shared memory segments, message queues, and semaphores.
   - Prevents IPC mechanisms from being shared between containers and the host.
6. **USER Namespace:**
   - Isolates user and group IDs. Allows containers to have a different set of user IDs and group IDs, providing user namespace remapping for improved security.
   - Enables non-root users to run containers with root privileges inside the container without being root on the host.
7. **CGROUP Namespace:**
   - Isolates the view of the control groups (cgroups). Each container can only see and manipulate its own cgroups, providing resource accounting and limiting.

### Control Groups (cgroups)

Cgroups are a Linux kernel feature that limits, accounts for, and isolates the resource usage (CPU, memory, disk I/O, network, etc.) of a collection of processes. Cgroups provide the ability to manage resources effectively for containers:

1. **CPU Resources:**
   - Cgroups can limit the CPU usage of a container by specifying the maximum amount of CPU time that can be allocated.
   - Cgroups can also prioritize CPU access for different containers using shares and quotas.
2. **Memory Resources:**
   - Cgroups can limit the memory usage of a container, ensuring that no container can consume all the host memory and affect other containers or the host system.
   - Memory limits include the ability to control both physical memory and swap usage.
3. **Block I/O Resources:**
   - Cgroups can limit the block I/O (disk I/O) operations, ensuring fair access to disk resources and preventing a single container from monopolizing disk I/O.
4. **Network Resources:**
   - Cgroups can control the network bandwidth allocated to each container, ensuring fair distribution of network resources.

### Filesystem Layers

Containers typically use a layered filesystem to manage their files and directories. The most common implementation is the UnionFS (union filesystem), which allows multiple filesystems to be layered together:

1. **Read-Only Layers:**
   - The base image of a container is composed of multiple read-only layers. These layers are immutable and shared among containers, reducing disk usage and improving efficiency.
2. **Read-Write Layer:**
   - When a container is started, a read-write layer is added on top of the read-only layers. All changes made by the container (new files, modifications, deletions) are written to this layer.
3. **OverlayFS:**
   - OverlayFS is a modern union filesystem that is widely used in Docker. It efficiently combines the read-only image layers with the writable container layer, providing a unified view of the filesystem.

### How Containerization Works Together

When you start a container:

1. **Namespaces:** Docker creates a set of namespaces for the container, isolating its processes, network, mounts, IPC, UTS, and user IDs from the host and other containers.
2. **Cgroups:** Docker assigns the container to specific cgroups, limiting its resource usage according to the specified policies.
3. **Filesystem:** Docker constructs the container's filesystem from the base image layers and adds a writable layer on top. The container sees a unified view of its filesystem, isolated from the host and other containers.
4. **Network:** Docker sets up a network namespace for the container, assigning it virtual network interfaces and configuring its networking stack. This allows containers to communicate with each other and the host through virtual networks.
5. **Execution:** Docker starts the container's main process in the isolated environment created by namespaces, cgroups, and the layered filesystem. The process runs as if it has its own isolated machine.

### Advantages of Containerization Over Virtualization

1. **Efficiency:**
   - Containers share the host OS kernel, leading to less overhead compared to VMs, which each require a full OS instance.
   - Faster startup times and reduced resource usage.
2. **Portability:**
   - Containers encapsulate the application and its dependencies, ensuring consistent behavior across different environments.
   - "Write once, run anywhere" approach simplifies deployment and scaling.
3. **Density:**
   - Higher density of containers on a single host compared to VMs, leading to better resource utilization and cost savings.
4. **Isolation:**
   - Provides strong isolation between applications, reducing conflicts and improving security.

By leveraging Linux namespaces, cgroups, and layered filesystems, containerization provides a lightweight, efficient, and secure way to deploy and manage applications, offering significant advantages over traditional virtualization methods.

