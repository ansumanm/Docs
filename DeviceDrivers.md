# Table of contents
1. [System bootup process](system-bootup-process)
2. [Hardware detection process](hardware-detection-process)
   
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
    - ***Dynamic Hardware Detection:*** Tools like *lsusb*, *lspci*, *lsblk* can be used to list USB devices, PCI devices and block devices respectively.These tools read from
      various system interfacecs to present current hardware information.
