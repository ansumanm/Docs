# Table of contents
1. [System bootup process](system-bootup-process)
2. [Hardware detection process](hardware-detection-process)
3. [*/proc* file system](#proc-file-system)
4. [*/sys* file system](#sys-file-system)
   
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
