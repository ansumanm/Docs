# Table of contents
1. [System bootup process](system-bootup-process)
2. [Hardware detection process](hardware-detection-process)
3. [*/proc* file system](#proc-file-system)
4. [*/sys* file system](#sys-file-system)
5. [Process virtual memory map](#process-virtual-memory-map)
6. [Kernel space](#kernel-space)
7. [OS scheduling](#os-scheduling)
8. [Linux system calls](#linux-system-calls)
9. [Linux device driver](#linux-device-driver)
   
# System bootup process
  - **Power-on and Initial Startup:**
    - When a computer is powered on, the CPU is reset and starts executing instructions from a predefined memory address. For traditional BIOS systems,
      the address is in the firmware
    - The firmware initializes the hardware components necessary for booting and performs a series of diagnostic tests known as the Power-On Self Test.
  - **Firmware Handoff:**
    - After POST, the BIOS looks for bootable devices in a predefined order set in the BIOS settings. It reads the first sector (512 bytes) of the bootable device,
      known as the Master Boot Record (MBR), into memory and transfers control to the code found there.
  - **Bootloader Execution:**
    - The primary bootloader, has a very limited role: to load a secondary, more complex bootloader that resides somewhere on the disk. This secondary bootloader is what
      users typically interact with (GRUB, LILO etc..)
    - The secondary bootloader displays a meny allowing the user to select the operating system or kernel version to boot. It then loads the selected kernel into memory,
      along with any initial RAM disk thats necessary for booting.
  - **Kernel Initialization:**
    - Once the kernel is loaded into memory by the bootloader, the bootloader executes a jump instruction to the starting address of the kernel, effectively transferring
      control to the kernel.
    - The kernel then initializes itself, detects and configures hardware, mounts the root filesystem, and starts the init process, which completes the boot process by
      starting system services and bringing the system up to a usable state.

# Hardware detection process
  - **Hardware Enumeration:**
    -  ***BIOS/UEFI:*** Initially, the kernel retreives basic hardware information from the BIOS,. This information includes data about the CPU, memory and other essential devices.
    -  ***Device Trees(for some architectures):*** On systems like ARM, device trees are used to describe hardware components and their configuration.The kernel uses this
        static description to understand what devices are present.
    - ***PCI Enumeration:*** The kernel scans the PCI(Peripheral Component Interconnect) bus to discover devices such as graphics cards, network cards and storage
      controllers. For each detected PCI device, the kernel reads its identifiers.
  - **Device Initialization and Driver Binding:**
    - ***Loading Device Drivers:*** Using the hardware identifiers collected during enumeration, the kernel matches devices with the appropriate drivers -- software components
      that knwo how to interact with and control the hardware.
    - ***udev and Hotplugging:*** The *udev* daemon in user space plays a crucial role in managing device nodes in the */dev* directory. When new hardware is detected, either
      at boot or when hot-plugged, *udev* is responsible for creating device nodes and initiating any user-space actions required to configure the device.
    - ***Modprobe and Kernel Modules:*** For hardware that requires drivers not built into the kernel, the `modprobe` utility automatically loads the necessary kernel modules.
      Modules can be loaded and unloaded dynamically, allowing for flexible hardware support.
  - **System initialization scripts and services:**
    - After the kernel has initialized the hardware, the system initialization scripts and services managed by system and service managers like *systemd* or *SysVinit* may
      perform additional configurations and start user-space services that require hardware access.
  - **Dynamic Detection with Tools and Daemons:**
    - ***Dynamic Hardware Detection:*** Tools like *lsusb*, *lspci*, *lsblk* can be used to list USB devices, PCI devices and block devices respectively.These tools read from various system interfacecs to present current hardware information.

# */proc* file system

The /proc filesystem in Linux is a virtual filesystem that provides a window into the kernel's view of the system. It doesn't contain real files but rather, virtual files that allow users and applications to access kernel internals. Many of these virtual files contain information about the system hardware and the current status of various components. 

| **File/Directory**          | **Description**                                                                 |
|-----------------------------|---------------------------------------------------------------------------------|
| `/proc/cpuinfo`             | Contains detailed information about the CPU(s), such as model, cores, and speed.|
| `/proc/meminfo`             | Provides details about the system's memory usage, including total and free memory.|
| `/proc/interrupts`          | Lists the number of interrupts per CPU per I/O device.                          |
| `/proc/diskstats`           | Offers detailed statistics for each disk device, including read/write operations.|
| `/proc/device-tree/`        | Shows the device tree structure (mainly on systems like ARM), describing hardware components. Not available on all systems.|
| `/proc/bus/pci`             | Contains information about PCI buses and devices on the system.                 |
| `/proc/bus/usb`             | Provides information on USB buses and devices. This might be more commonly accessed through `/sys` or `lsusb` in modern systems.|
| `/proc/loadavg`             | Shows the system load average, indicating the system's activity level.          |
| `/proc/net/dev`             | Contains statistics about network interfaces, including performance data.       |


# */sys* file system
The */sys* filesystem in Linux, also known as *sysfs*, provides a structured and dynamic interface to the kernel's device model. Itt is designed to offer information about devices and drivers fromt eh system's perspective, making it easier for user space tools and applications to interact with and manage hardware devices.

## Key Features and Uses of */sys*:
### Device Information and Management
*/sys* provides a wealth of information about devices connected to the system. It organizes devices by their type and bus, making it easier to find detailed information about specific hardware components, such as USB devices, PCI devices and network interfaces.

### Driver Binding and Unbinding:
It allows dynamic binding and unbinding of drivers to devices, which can be particularly useful for troubleshooting or for systems where devices need to be reconfigured on-the-fly.

### Hardware Configuration:
Through */sys*, users can modify certain hardware parameters, enabling or disabling features of devices as supported by their drivers. This can include toggling power management features, changing LED states, or adjusting performance settings.

### Hotplug support:
The filesystem plays a crucial role in supporting the hotplug mechanism, allowing the system to detect and configure hardware as it's physically added or removed.

## Structure of */sys*:
### ***/sys/devices***
This directory contains a hierarchy reporesenting all devices and buses in the system. Devices are organized according to their pyysical or logical connection in the system,
providing a comprehensive view of all hardware.

### ***/sys/class***:
Devices are grouped by class(type) here, providing a more intuitive way to access devices based on their functionality (e.g., */sys/class/net* for network interfaces, */sys/class/block* for storage devices).

### ***/sys/block***:
Lists block devices and allows access to attributes specific to storage operations.

### ***/sys/bus***:
Shows the various bus types and supports actions like scanning for new devices or directly interacting with bus drivers and devices.

# */proc* vs */sys*
- ## For Kernel and Process Info:
  Prefer */proc* for accessing information related to the kernel's view of processes, or for adjusting certain kernel parameters that affects the system's operation but not directly tied to specific devices or drivers.
- ## For Hardware and Device Info:
  Use */sys* for tasks that involve direct interaction with hardware devices or when you need to read or modify device attributes. Sysfs's structured layout and device-centric approach make it ideal for managing hardware components and their drivers.

# Process virtual memory map
Below is a simplified representation for a generic 32-bit Linux process. Note that actual sizes and specific addresses can vary based on system configuration, architecture, and the process itself.

| Segment             | Address Range                   | Description                                                                 |
|---------------------|---------------------------------|-----------------------------------------------------------------------------|
| Kernel Space        | 0xC0000000 - 0xFFFFFFFF (upper 1GB) | Reserved for the kernel; inaccessible from user-space processes.            |
| Stack               | Variable, grows downward        | Stores local variables, function parameters, return addresses, and control data. Located near the top of the user space. |
| Memory Mappings     | Variable                        | Includes shared libraries, memory-mapped files, etc. Can be located between heap and stack. |
| Heap                | Variable, grows upward          | Dynamically allocated memory (e.g., via `malloc`) during runtime. Starts after BSS/data segments and grows towards higher addresses. |
| BSS Segment         | Variable                        | Uninitialized data initialized to zero by the kernel at start time.          |
| Data Segment        | Variable                        | Initialized global and static variables.                                    |
| Text Segment        | 0x08048000 (example start)      | Executable instructions of the process. Typically read-only.                |
| Reserved            | 0x00000000 - 0x08047FFF         | Reserved by the system, including the null pointer dereference protection.  |

This table simplifies the layout and doesn't account for various system-specific or architecture-specific differences, such as the use of Address Space Layout Randomization (ASLR) which randomizes the starting addresses of some segments to mitigate certain types of security vulnerabilities. Also, in 64-bit systems, the address space and the kernel/user space split will differ significantly due to the much larger addressable memory space.

## Kernel Space
The kernel space in the user process map is inaccessible to the process. It is mapped into every process's address space to facilitate efficient transitions from user mode to kernel mode. It contains the following:
-   ### Kernel code and data
  This includes the executable binary code of the kernel, global variables, kernel functions and kernel modules.
-   ### System call handlers
  The entry point for system calls made by the processes. These handlers facilitate the execution of the kernel services on behalf of user processes.
-   ### Device drivers
  Code that manages hardware devices.
-   ### Memory management structures
  Structures such as page tables, which the kernel uses to manage both physical and virtual memory of the system.
-   ### Kernel stacks
  Each process has a kernel-mode stack in the kernel space. This stack is used when the process executes in kernel mode, separate from the user-mode stack.
-   ### Process and Scheduler Information
  Information about processes and threads, including task structs, scheduling queues and other metadata that the kernel scheduler uses to manage process execution.
-   ### File system cache and buffers
  Buffers and caches for file systems to improve the performance of disk operations.
-   ### Network Buffers and structures
  Similar to file system buffers but for network operations.
-   ### Inter-process Communication Mechanisms
  Kernel space structures for IPC mechanisms like pipes, message queues, semaphores and shared memory.
-   ### Security and access control data
  Data structures that enforce security policies, including user credentials associated with processes, access rights and crypt keys for secure communication.

### Kernel Stack
The kernel stack is specifically used when the process operates in kernel mode, such as during system calls, interrupts and exceptions.This separation ensures that kernel operations have a dedicated, protected space for execution independent of the user-space environment.

#### Key Aspects of the Kernel stack
- **Dedicated Space:** Each process or thread has its own kernel stack, which is relatively small compared to the user-space stack. The stack is used exclusively for operations that occur while the process in in kernel mode.
- **Security and Stability:** By isolating the kernel stack from user space, the operating system protects sensitive kernel data and operations from user-space errors or malicious activities, enhancing system security and stability.
- **Context Switching:** The kernel stack plays a crucial role during context switches between processes. When the scheduler decides to switch the CPU from running one process to another, it uses the kernel stack to store the state of the process being switched out(if it's in kernel mode) and to restore the state of the process being switched in.

#### When is the Kernel Stack used?
- **System calls:** When a user process requests a service from the kernel via a system call, the transition from user mode to kernel mode involves switching to use the process's kernel stack. The kernel stack stores the parameters of the system call, return address and local variables needed by the kernel to service the request.
- **Interrupt handling:** Upon receiving a hardware interrupt, the CPU switches to kernel mode to execute the interrupt service routine (ISR). The ISR uses the kernel stack of the current process to store its execution context and any temporary data.
- **Exceptions and Faults:** When an exception -- like page fault, division by zero -- occurs while a process is executing, the handling of this exception by the kernel uses the kernel stack to maintain the context of the operation and to manage recovery or error reporting.
- **Kernel threads:** which are thread of execution entirely within kernel space use kernel stacks for their operations, independent of any user process.

#### Characteristics
- **Size:** Kernel stacks are typically fixed in size and are relatively small. For instance, on Linux, the default size might be a few KB which is usually sufficient for kernel mode operation, but requires careful management to avoid stack overflow.
- **Allocation:** The kernel stack is allocated when the process or thread is created and dellocated when it terminates. The allocation is managed by the kernel's memory management subsystem.
  
# OS scheduling
A hardware timer interrupt is commonly used. This timer interrupt allows the operating system's scheduler to perform regular and timely assessments of the running processes and make decisions about context switching, process prioritization, and load balancing among CPUs in multicore systems.

## Hardware Timer Interrupt
### Function:
-   Generates interrupts at regular, predefined intervals. When the operating system initializes, it configures this timer to interrupt the CPU after a specific period- this period is open referred to as a ***tick***.
### Usage in scheduling:
-   Each time a timer interrupt occurs, the ISR associated with the timer is executed. This mechanism is central to implementing time-sharing systems, allowing multiple processes to run "simulatneously" by giving each a slice of CPU time.
### Premptive Multitasking:
In preemptive multitasking environments, the timer interrupt enables the OS to preempt the currently running process if its allocated t ime slice has expired, ensuring that no single procecss monopolizes the CPU.
### Platform specific implementations:
#### x86 Architecture:
Uses the Programmable Interval Timer (PIT), Local APIC timers or High Precision Event Timer (HPET) for generating periodic interrupts.
#### ARM Architecture:
Uses system timers provided by the ARM architecture.
### Modern Enhancements
#### Tickless kernels:
Modern operating systems have been moving towards *tickless* or *dynamic ticking* kernels, which reduce or eliminate unnecessary timer interrupts during periods of inactivity. This approach conserves power and reduces interrupt overhead, which is particularly beneficial in mobile and embedded systems.
#### High-Resolution Timers:
Operating systems also use high-resolution timers to support applications that require precise timing beyond the basic scheduling tick. These are typically implemented using different hardware timers that can be programmed to generate interrupts at very fine-grained intervals.

# Linux System Calls

Here's a table summarizing some fundamental Linux system calls and their purposes. System calls provide the primary interface between user-space applications and the kernel, allowing programs to request various services, such as file operations, process management, and communication.

| System Call    | Description                                                                                                                                                 |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `open()`       | Opens a file or device, returning a file descriptor that can be used in subsequent operations like `read()`, `write()`, and `close()`.                      |
| `read()`       | Reads data from an open file or device into a buffer. The amount of data read is specified by the caller.                                                  |
| `write()`      | Writes data from a buffer to an open file or device. The amount of data written is specified by the caller.                                                |
| `close()`      | Closes an open file descriptor, freeing up resources associated with it.                                                                                    |
| `fork()`       | Creates a new process by duplicating the calling process. The new process is referred to as the child process.                                             |
| `exec()`       | Replaces the current process image with a new process image, specified by a file path. This family of functions includes `execl()`, `execp()`, and others.|
| `wait()`       | Waits for a child process to change state, usually to finish execution. It's used for process synchronization.                                             |
| `exit()`       | Terminates the calling process and returns an exit status to the operating system.                                                                         |
| `socket()`     | Creates a socket descriptor, specifying the communication domain (e.g., IPv4, IPv6), type (e.g., stream, datagram), and protocol.                          |
| `bind()`       | Binds a socket to a local address and port number, necessary for servers to receive connections on a specified port.                                       |
| `listen()`     | Marks a socket as a passive socket that will be used to accept incoming connection requests using `accept()`.                                              |
| `accept()`     | Blocks the caller until a connection request arrives on a listening socket, then returns a new socket descriptor for the connection.                       |
| `connect()`    | Initiates a connection on a socket, used by clients to establish a connection with a server.                                                               |
| `send()`       | Sends data on a connected socket. It's part of a set of functions, including `sendto()` and `sendmsg()`, for various use cases.                            |
| `recv()`       | Receives data from a connected socket. Similar to `send()`, there are related functions like `recvfrom()` and `recvmsg()` for different contexts.           |
| `mmap()`       | Maps files or devices into memory, providing a method for applications to access files or devices as if they were in memory.                               |
| `munmap()`     | Unmaps a previously mapped memory area, typically used after finishing with the memory area provided by `mmap()`.                                          |
| `ioctl()`      | Provides device-specific input/output operations that don’t fit the regular system call semantics, like controlling non-data operations of a device.        |

These system calls enable fundamental operations necessary for the execution and management of processes, handling of files and devices, and network communication in Linux-based systems.

# Thread-safe function
Making a function thread-safe means ensuring that it can be safely called from multiple threads simultaneously withoug causing data corruption or inconsistent results.
-   **Avoid shared state:** Use only local variables withing the function.
-   **Immutable shared state:** If shared data does not change, multiple threads can safely read it withouh synchronization.
-   **Synchronization mechanisms:**
   - ***Mutexes and Locks:*** Use mutexes to synchronize access to shared resources. A thread acquires the mutex before accessing the shared resource and releases it afterwards.
   - ***Reader-Writer Locks:*** If your function frequently reads but infrequently writes to a shared resource, consider using reader-writer locks. They allow multiple readers simultaneous access but require exclusive access for writing.
   - ***Atomic operations:*** For simple data  types, use atomic operations to read, write, increment or compare-and-swap without needing explicit locks.
   - ***Condition Variables:*** Use condition variables in conjuction with mutexes for scenarios where a thread needs to wait for a certain condition before proceeding.
-   **Thread-local Storage**
     Use TLS to provide each thread with its own instance of a variable. TLS variables are global but are instantiated separately for each thread.
    Ex- __thread int thread_local_variable;
-   **Reentrancy**
     - ***Make functions reentrant:***  A reentrant function does not hold an static or global non-constant data, does not return a pointer to a static data and only calls reentrant functions.

# Pthread synchronization

Here's a table summarizing the different pthread features used for thread synchronization, along with a brief description of each:

| Feature                   | Description                                                                                                                                   |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| Mutex (pthread_mutex_t)   | Mutual exclusion object that prevents multiple threads from executing critical sections of code that access shared data concurrently.         |
| Recursive Mutex           | A special type of mutex that can be locked multiple times by the same thread without causing a deadlock.                                      |
| Spinlock (pthread_spinlock_t) | A lock that causes a thread trying to acquire it to simply wait in a loop ("spin") while repeatedly checking if the lock is available.        |
| Read-Write Lock (pthread_rwlock_t) | Allows concurrent access for read-only operations, while write operations require exclusive access.                                          |
| Condition Variable (pthread_cond_t) | Allows threads to suspend execution and relinquish the processor until some condition is true. Typically used with a mutex.                 |
| Barrier (pthread_barrier_t) | A synchronization mechanism that blocks a set of threads until all threads have reached a certain point in their execution.                   |
| Semaphore (POSIX sem_t)    | Not a pthread feature per se, but POSIX semaphores can be used in pthreads-based applications for counting-based synchronization.            |

Each of these synchronization features serves different use cases and scenarios in multithreaded programming, providing a robust set of tools for managing concurrent execution and access to shared resources in pthreads-based applications.

# Linux device driver
## Driver and Device Registration
### PCI registration
The driver must register itself with the PCI subsystem, providing a table of device IDs it supports.Functions for probing (initializing a device upon detection) and removing (cleanup when a device is removed or the driver is unloaded) are specified here.
### Network Device Registration
Within the probe function, after successful PCI-specific initialization, a network device instance (**struct net_device**) is allocated and registered with the networking subsystem. The structure presents the Ethernet interface to the kernel.

## Initialization and Configuration
### PCI device initialization:
In the probe function, the driver enables the PCI device, requests the necessary I/O and memory regions, sets up DMA parameters, and maps the device's registers into the kernel's address space.
### Network Device Setup:
The network device's operations (**netdev_ops**) are defined and assigned. These operations include open, close, start_xmit(transmission), change_mtu ..etc

## Packet Transmission
### Start Transmission (**start_xmit** function)
This function is called by the networking subsystem when there's a packet ready to be sent. The driver must take the packet ( a **skb** or socket buffer), possibly map it for DMA, and initiate the transmission process according to the hardware's requirements.
### Transmissin Complete
Often handled in an interrupt service routine that responds to transmission complete interrupts from the hardware. The driver must clear the necessary flag, unmap the DMA and inform the kernel of the completeion.

## Packet Reception
### Reception process
Typically the hardware interrupts the CPU when packets are received. The driver's ISR must then retreive the packets from the device, possibly involving DMA unmap operation, and pass them up to the networking stack.
### Passing the packets to the stack
Received packets are wrapped into the **skb** structures and handed to the kernel using netif_rx() or similar functions. The driver must also manage receive buffers and notify the hardware to be ready for more packets.

## Interrupt Handling
### Interrupt registration
The driver registers an ISR with the kernel to handle hardware interrupts. The ISR will deal with both transmission and receiption notifications along with any errors and status interrupts from the device.

## Teardown and cleanup
### Device shutdown
Operations to stop the device's network activity, usually implemented in the network device's **stop** function. This includes disabling the interrupts, stopping DMA and putting the hardware into a low-powered state if needed.
### Driver unloading
When the driver is removed, it must unregister the network device, release requested resources (I/O memory. DMA buffers etc), unmap device memory and perform any additionah hardware specific cleanup.

## Supporting structures and functions
### Private data structures
Drivers often define a structure to hold device specific information. This is attached to the **net_device** structure via a **priv* field.
