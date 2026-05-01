- Running different operating systems on a machine at the same time
- IBM introduces virtual machine monitor (hypervisor) as a solution
	- Sits between one or more operating systems and the hardware and gives illusion to each running OS that it controls the machine
	- Behind scene monitor is what is actually in control of the hardware, must multiplex running OSes across physical resources of machine
	- VMM serves as an operating system for operating systems but at much lower level
		- OS must still think it is interacting with physical hardware
		- Transparency major goal of VMMs
	- **CRUX: How To Virtualized The Machine Underneath The OS**

**Motivation**
- Consolidate multiple OSes into fewer hardware platforms and lower costs and ease of administration
	- Run services on different machines running different operating systems
- Users want to run other OS while keeping native applications 
- Testing and debugging code on many different platform
- Resurgence in virtualization beginning in earnest the mid-late 1990s led by a group of researchers at Stanford headed by Prof Mendel Rosen Blum
	- Work on Disco - VMM for MIPS processor early effort revive VMM
	- Led group to founding of VMware
		- Market leader in virtualization technology

**Virtualizing The CPU**
- Running VM on top of VMM
	- Basic technique of limited direct execution
		- Jump address to first instruction and let OS begin running
	- VMM must perform machine switch between virtual machines
		- Save entire machine state of one OS and restore the machine state of the to be run VM before jumping to PC
			- Maybe within the OS itself
			- Or a process that is simply running on the OS
	- Privileged Operation
		- OS cannot be allowed to perform privileged instruction because then it controls the machine rather than VMM beneath it
		- VMM must intercept attempts to perform privileged operation and retain control of machine 
		- Executes a trap instruction with arguments carefully on stack
			- VMM has a installed trap handler that will first get executed in kernel mode
			- VMM does not know how to handle system call as it does not know details of each system call 
			- Knows where OS's trap handler is
				- Tried to install its own trap handlers  when the OS did so it was try to do something privileged and trapped into VMM. VMM recorded necessary information
			- Using this it can jump to OS's trap handler and let OS handle system call as it should
			- OS return from trap bounces into VMM, so VMM returns from trap and returns control user and puts the machine back in user mode
		- What mode OS should be running in
			- Not kernel as it gives too much privilege
			- In Disco, use of MIPS hardware supervisor mode
				- Not fully execute privileged instructions but has more memory than when in user mode 
				- Can use extra memory for data structure
				- On hardware that doesn't have such a mode, one has to run OS in user mode and use memory protection to protect OS data structures appropriately
					- When switching into OS, monitor would have to make the memory of the OS data structures available to the OS via page-table protections
					- When switching back to the running application, ability to read and write the kernel would be removed


**Virtualizing Memory**
- Add another layer of virtualization so multiple OSes can share actual physical memory of machine
	- Makes 'physical' memory a virtualization on top of what VMM refers to as machine memory
		- Each OS maps virtual to physical address via per process page table
		- VMM maps resulting physical mappings to underlying machine address via its per OS page tables 
- When process makes a virtual memory reference and misses in TLB, VMM TLB miss handler runs as VMM is the true privileged owner of the machine
	- VMM uses OS trap handler as it knows the location, OS returns trap handler after translation
	- VMM installs its desired VPN to MFN mapping
	- System then gets back to user level code
- VMM must track physical to machine mappings for each VM it is running
	- Per machine page tables need to be consulted in the VMM TLB miss handler in order to determine which machine a particular physical page maps to
- To reduce cost Disco added VMM-level "software TLB"
	- Data structure records every virtual to physical mapping that it sees the OS try to install
	- On TLB miss, VMM consults its software TLB to see if it has seen virtual to physical mapping before  and what the VMM's desired virtual to machine mapping should be
		- If it finds the translation, simply installs it directly into hardware TLB and skips back and forth control flow

**Information Gap**
- Lack of knowledge between VMM and OS
	- Can lead to inefficienceies
	- When OS has nothing else to run will go into an idle loop just spinning and waiting for the next interrupt to occur
		- Makes sense if OS is in charge of whole machine
		- In VMM we can use resources effectively by letting one OS idle loop and one do work
	- VMM must zero pages it gives to OS for security
		- Zeroed twice due to security purposes
		- Disco solution (not good): Changed OS (IRIX) to not zero pages that it knew had been zeroed by underlying VMM


**Types of Virtualization**
- Type 1 - Bare Metal
	- Runs directly on hardware without a host OS
	- Provides direct hardware access through a hyperviosr
	- Offers better performance and security
- Type 2 - Hosted
	- Runs as an application on top of a host OS
	- Hardware access is mediated through the OS
	- Slower than type 1 but easier to use

**Paravirtualization**
- Represents middle ground between full virtualization and emulation
- Modifies guest OS to communicatee directly with hypervisor
	- Both type 1 and type 2 hypervisors
- More efficient than full emulation but requires modified guests

**Performance Hierarchy (Fastest to Slowest)**
- Virtualization with direct hardware access (Type 1)
	- Data Center: VMware ESXi running multiple  enterprise servers on bare metal
	- Cloud Providers: Microsoft Hyper-V powering Azure's infrastructure
- Paravirtualization
	- Cloud Services: AWS using xen based paravirtualization for EC2
	- Linux Systems: KVM/QEMU implementing paravirtualization for improved performance
	- Enterprise Virtualization: Xen hypervisor managing resource-intensive workloads
- Hosted Virtualization (type 2)
	- Development Environments: VirtualBox running test environments on a developer's laptop
	- Software Testing: VMware workstation running multiple OS versions for compatibility testing
	- Home Labs: Running windows and Linux simultaneously on a personal computer 
- Full emulation
	- Must translate each instruction between architectures

**Virtualization vs Emulation**
- Virtualization partitions existing hardware resources and allows direct hardware access 
- Emulation completely mimics one system's hardware on another, requiring full translation of instructions 
- Virtualization is generally faster since it doesn't need to translate between architectures