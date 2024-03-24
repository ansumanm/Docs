# Kernel-based Virtual Machine (KVM)
KVM is a popular open-source virtualization solution for Linux that turns the kernel into a hypervisor.

## Managing VMs
-  **virt-install:** Creates new VMs. It allows specifying various parameters like the name, disk, memory, network interface and more during the VM creation process.
```bash
virt-install --name=vm_name --vcpus=2 --memory=2048 --cdrom=/path/to/install.iso --disk size=20G,format=qcow2 --os-type=linux --os-variant=generic
```

- **virsh start <vm name>:** Starts a specified VM.
```bash
virsh start vm_name
```

- **virsh shutdown <vm name>:** Gracefully shutdown a specified VM.
```bash
virsh shutdown vm_name
```

- **virsh reboot <vm name>:** Reboots a specified VM.
```bash
virsh reboot vm_name
```

- **virsh destroy <vm name>:** Forcibly stop a specified VM. This is equivalent to unplugging the power cord.
```bash
virsh destroy vm_name
```

- **virsh undefine <vm name>:** Deletes a VM definition. This does not delete the VM's storage volumes.
```bash
virsh undefine vm_name
```

- **virsh list [--all]:** Lists all running VMs. The --all flag includes stopped VMs in the list.

## Managing Disk Images
```bash
# Create
qemu-img create -f qcow2 /path/to/image.qcow2 20G

# Resize
qemu-img resize /path/to/image.qcow2 +10G

# Clone
virt-clone --original vm_name --name new_vm_name --auto-clone
```

## Networking
```bash
# List all defined networks
virsh net-list --all

# Starts a specified network
virsh net-start default

# Stops a specified network
virsh net-stop default

# Delete a network definition
virsh net-undefine default
```

## Monitoring and Troubleshooting
```bash
# Displays information about a specified VM
virsh dominfo vm_name

# Lists all block devices for a specified VM
virsh domblklist vm_name

# Show the VNC display port for a specified VM, allowing remote access to the VM's console.
virsh vncdisplay vm_name
```

# KVM + QEMU
## KVM's Role:
-  **Hardware-assisted virtualization:**
KVM turns the Linux kernel into a hypervisor. It leverages CPU extensions designed for virtualiztion (Intel VT-x or AMD-V) to run VMs("guests") directly on the hardware, which significantly improves the performance by allowing most guest code to execute unmodified at near-native speed.
-  **Kernel Integration:**
Being part of the linux kernel, KVM benefits from kernel features such as scheduling, memory management and security applying these directly to VM management.

## QEMU's Role:
-  **Device Emulation:**
QEMU is capable of emulating a wide range of hardware peripherals (network cards, disk controllers, GPUs, etc..) making it possible to create virtual hardware environments for VMs that mimic real systems.
-  **User-space Components:**
Operating in user space, QEMU interacts with KVM to launch  VMs and provides emulation layer for devices not handled by KVM or for which direct hardware access isn't possible or desired.

## How they work together
### VM Creation and Execution
QEMU initiates the creation of a VM, setting up the virtual hardware environment. It then utilizes the KVM to create and manage VM's processes, leveraging KVM's integration with Linux kernel for efficient execution.

### CPU Virtualizaion
For CPU-intensive tasks, KVM directly executes guest code on the host CPU, using hardware virtualization features to ensure isolation and performance. This direct execution path significantly reduces overhead compared to full emulation.

### Device I/O
For device I/O operations, QEMU emulates the virtual hardware devices of a VM. When a guest OS attempts to interact with a hardware device, QEMU intercepts these calls and emulates the expected behavior of the hardware, providing the quest with a realistic operating environment.

### Memory Management
KVM manages memory of VMs through Linux kernel, using standard memory management techniques and additional features like huge pages for performance. QEMu interacts with KVM to allocate, map and access the VM's memory space.

### Network emulation and bridging:
QEMU can emulate network interfaces for VMs and also supports bridged networking, allowing VMs to be connected to real network segments transparently.

### Graphics and Input/Output:
QEMU provides virtual graphics adapters and input/output devices for VMs, allowing users to interact with their VMs using standard peripherals like keyboards, mice and monitoprs.
