---
title: 18 Virtual Machines
toc: false
date: 2017-10-30
---

### 1 Overview

Virtual machine implementations involve several components. 

* **Host**: the underlying hardware system that runs the virtual machines
* **Virtual machine manager**(VMM, or **hypervisor): creates and runs virtual machines by providing an interface that is *identical* to the host
* **Guest**: an operating system.

![system_models_virtual_machine](figures/system_models_virtual_machine.png)

Figure: <small>System models. (a) Nonvirtual machine. (b) Virtual machine.</small>


The implementation of VMMs varies greatly. Options include the following:

#### Type 0

Type 0 hypervisors: hardware-based solutions, provides support for virtual machine creation and management via *firmware*. It is commonly used in mainframe and large to midsized servers.

_The VMM itself is encoded in the firmware_ and loaded at boot time. In turn, it loads the guest images to run in each partition.

A type 0 hypervisor can run multiple guest operating systems (one in each hardware partition). All of those guests, because they are running on raw hardware, can in turn be VMMs. Because of that, each can have its own guest operating systems.


!!! example "Oracle SPARC"
    
    The [Oracle SPARC](https://docs.oracle.com/cd/E38405_01/html/E38406/hypervisorandldoms.html) hypervisor is a small firmware layer that provides a stable virtualized machine architecture to which an operating system can be written.

    ![](figures/type_0_hypervisor_demo.jpg)


#### Type 1 

Type 1 hypervisors: operating-system-like software built to provide virtualization. It is commonly found in company data centers. They are special-purpose operating system that run natively on the hardware, but rather than providing system calls and other interfaces for running programs, they create, run, and manage guest operating systems.

Type 1 hypervisors run in kernel mode, taking advantage of hardware protection. Because they are operating systems, they must also provide CPU scheduling, memory management, I/O management, protect and so on.


!!! example "KVM"

    ![](figures/kvm.png)

    KVM(Kernel-based Virtual Machine)is a full virtualization solution for the AMD64/Intel 64. It consists of two main component[^1]:
    
    * A set of kernel modules (kvm.ko, kvm-intel.ko, and kvm-amd.ko) that provides the core virtualization infrastructure and processor-specific drivers.
    * A user space program (qemu-system-ARCH) that provides emulation for virtual devices and control mechanisms to manage VM Guests (virtual machines).
    
![type_1_hypervisor](figures/type_1_hypervisor.png)


Unmodified OS is running in user mode (or ring 1)
    
* But it thinks it is running in kernel mode (virtual kernel mode)
* privileged instructions trap; sensitive inst-> use VT to trap 
    * Hypervisor is the “real kernel”
    * Upon trap, executes privileged operations 
    * Or emulates what the hardware would do
    
#### Type 2

Type 2 hypervisors: applications that run on standard operating systems but provide VMM features to guest operating systems.

Type 2 hypervisors run on a variety of general-purpose operating systems, and running them requires no changes to the host operating system.


!!! example "VirtualBox"

    To run a Type 2 hypervisor, you need an operating system that will run underneath the hypervisor. For Oracle VM VirtualBox, this means an already running host operating system on an x86-based desktop, laptop, or server. You install Oracle VM VirtualBox on top of that, as shown in figure below. Then you can simultaneously run multiple guest operating systems inside Oracle VM VirtualBox using multiple virtual machines (VMs).

    ![](figures/virtualbox.jpg)


#### Paravirtualization

*Paravirtualization*(准虚拟化) is a technique in which the guest operating system is modified to work in cooperation with the VMM to optimize performance.

Rather than try to trick a guest operating system into believing it has a system to itself, paravirtualization presents the guest with a system that is similar but not identical to the guest’s preferred system. The guest must be modified to run on the paravirtualized virtual hardware. The gain for this extra work is more efficient use of resources and a smaller virtualization layer.

!!! example "VirtualBox Paravirtualization Interface"

    VirtualBox enables the exposure of a paravirtualization interface for more accurate and efficient execution of software within a VM. Three paravirtualization interfaces are provided[^2]:
    
    * **Minimal**: Announces the presence of a virtualized environment. Additionally, reports the TSC and APIC frequency to the guest operating system. This provider is mandatory for running any Mac OS X guests.
    * **KVM**: Presents a Linux KVM hypervisor interface which is recognized by Linux kernels starting with version 2.6.25. VirtualBox's implementation currently supports paravirtualized clocks and SMP spinlocks. This provider is recommended for Linux guests.
    * **Hyper-V**: Presents a Microsoft Hyper-V hypervisor interface which is recognized by Windows 7 and newer operating systems. VirtualBox's implementation currently supports paravirtualized clocks, APIC frequency reporting, guest debugging, guest crash reporting and relaxed timer checks. This provider is recommended for Windows guests.

    ![virtualbox_paravirtualization](figures/virtualbox_paravirtualization.png)


#### Emulation

**Problem**: But what if an application or operating system needs to run on a different CPU? 

**Solution**: Emulation translates all of the source CPU’s instructions so that they are turned into the equivalent instructions of the target CPU. Such an environment is no longer virtualized but rather is fully emulated.

It is useful when the host system has one system architecture and the guest system was compiled for a different architecture.

The major challenge of emulation is performance. Instruction-set emulation may run an order of magnitude slower than native instructions, because it may take ten instructions on the new system to read, parse, and simulate an instruction from the old system.


!!! note "QEMU"
    
    QEMU is a hosted virtual machine monitor: it emulates the machine's processor through dynamic binary translation and provides a set of different hardware and device models for the machine, enabling it to run a variety of guest operating systems. [^11]



#### Application Containment

The goal of *Application Containment* is to segregate applications, manage their performance and resource use, and create an easy way to start, stop, move, and manage them.

*Containers* are much lighter weight than other virtualization methods. That is, they use fewer system resources and are faster to instantiate and destroy, more similar to processes than virtual machines. Containers are also easy to automate and manage, leading to orchestration tools like [docker](Docker.md) and Kubernetes.


!!! note "virtual machine v.s. containers"
    
    https://cs162.eecs.berkeley.edu/static/lectures/26.pdf
    
    * Virtual machines: provide each guest with the illusion of its *own dedicated hardware*
    * Containers: provide each guest with the illusion of its *own dedicated operating system*
        * Via resource isolation (cgroups)
            * Via namespace isolation
            * PID namespace
            * Network namespace
            * Filesystem…
        * With its own binaries, libraries, and dependencies


### 7 Examples

#### VMware Workstation


VMware Workstation is a popular commercial application that abstracts Intel x86 and compatible hardware into isolated virtual machines.At the heart of VMware is the virtualization layer, which abstracts the physical hardware into isolated virtual machines running as guest operating systems. Each virtual machine has its own virtual CPU, memory, disk drives, network interfaces, and so forth.

![vmware_workstation](figures/vmware_workstation.png)


[^1]: Introduction to KVM Virtualization, https://documentation.suse.com/sles/12-SP4/html/SLES-all/cha-kvm-intro.html
[^2]: Paravirtualization Providers, https://docs.oracle.com/en/virtualization/virtualbox/6.1/admin/gimproviders.html
[^11]: https://en.wikipedia.org/wiki/QEMU