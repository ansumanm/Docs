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

