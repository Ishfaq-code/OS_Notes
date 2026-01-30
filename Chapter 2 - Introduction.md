#OS #Introduction #Quiz_1 #Test_1 #Virtualization



 Primary goal is to make system easy to use

**Von Neumann Machine**: Processor fetches instruction from memory, decodes it and executes it
- Simple model
- Model needs to do a lot more than just do these steps
- Need to make system easy to use

**Crux of Operating Systems**: How to virtualize resources
- Why: To make the system easier to use

**Operating System**: Software making sure system operates correctly and efficiently
- Allow programs to share memory
- Enabling programs to interact with devices 
- System Operates correctly and efficiently 

**Virtualization**: OS takes physical resource and transforms into more general powerful easy to use virtual form of itself
- OS sometimes referred to as the virtual machine 
- Typical OS has few hundred system calls available to application
	- To run programs 
	- Access memory and devices and other related actions
- Provides standard library to applications
- Virtualization allows many program to run
	- Program needs to access devices concurrently
- OS known as the resource manager
	- Many programs run concurrently 
	- Access their own instructions and data (sharing memory)
	- Access devices (sharing disks )
	- CPU, Memory and Disk resource of the system 

Control-C terminates programs in the foreground in Unix Based Systems

**Virtualizing the CPU:** Turning single CPU into a seemingly infinite number of CPUs and allowing many programs to seemingly run at once  - **The Illusion**

APIs to communicate

**Memory**:
- Array of bytes
	- To read memory need address to be able to access data stored there
	- To write memory, specify the date to be written at given address
- Accessed all the time when a program is running 
- Each instruction of program in memory too
	- Memory accessed each instruction phase
- Each process accesses its own private virtual address space

Each process running has own private virtual address space which OS maps onto physical memory of the machine
- Sometimes called address space
- OS maps onto physical memory of machine
- Memory reference of 1 running program does not affect the address space of other programs 
- Physical memory is a shared resource managed by OS

**Concurrency**:
- CRUX: Host of problems that arise and must be dealt with when many programs are working together
- How to build correct concurrent programs 
- Three instructions to change value
	- One to load value from memory to register
	- One to increment it
	- One to store it back into memory
	- Must happen automictically for this to work 

**Thread:** Function running within the same memory space as other functions with more than one of them active at a time 

**Atomically:** Values execute all at once

- **Persistence** is the OS’s ability to **store data permanently**, even:
    - After power loss
    - After crashes
- **Main issue**:
    - **Main memory (DRAM) is volatile**
    - Data in memory is **lost when power is gone**

- Persistence is provided by **I/O devices**, mainly:
    - **Hard Disk Drives (HDDs)** – traditional, magnetic storage
    - **Solid-State Drives (SSDs)** – faster, flash-based storage
- These devices store **long-lived data** that survives reboots and crashes

- The OS component responsible for persistence is the **file system**
- Responsibilities:
    - Store files **reliably**
    - Store files **efficiently**
    - Manage how data is placed on disk

- CPU & memory:
    - Each process gets a **private, virtualized view**
- Disk:
    - **No private disk per process**
    - Files are **shared across processes**

Programs interact with files using **system calls**, not raw disk access.
- `open()` – opens or creates a file
- `write()` – writes data to a file
- `close()` – closes the file and finishes writing

Behind the scenes, the file system must:
- Decide **where on disk** data goes
- Maintain **metadata structures** (tracking files, locations, sizes)
- Issue **low-level I/O requests** to the device
- Coordinate with **device drivers**

This process is:
- Complex
- Error-prone
- Highly hardware-specific

The OS hides this complexity and provides **simple system calls**.


- The OS provides a **uniform interface** to devices
- Applications:
    - Don’t need to know device details
    - Just use system calls
- Device-specific logic lives in **device drivers**

- Disk operations are **slow**
- To improve performance, file systems:
    - **Delay writes**
    - **Batch multiple writes together**
    - Use **caching and buffering**

Problem:
- What if the system crashes **mid-write**?
Solution:
- File systems use **carefully ordered write protocols**, such as:
    - **Journaling**
    - **Copy-on-Write (CoW)**

These ensure
- The disk can be recovered to a **consistent state**
- Data corruption is minimized

**CRUX:** How to Store Data Persistantly

Main Goals of CPU:
- Provide High Performance
- Minimize Overheads of the OS virtualization
	- Extra time (more instructions)
	- Extra space (in memory or disk)
- Provide protection between applications 
	- Malicious or accidental bad behavior of one does not affect other
	- Isolating processes from one another 
- Reliability - runs non stop
- Energy Efficient
- Security
- Mobility - running on smaller and smaller devices

**History**:
Early OS didn't do much
- Set of libraries of commonly used functions
	- Programmers are not writing low level code
	- OS provides API
- One program runs at a time controlled by human
- Batch processing: number of jobs set up and run in batch by the operator 

**Beyond Libraries: Protection:**
- Privacy goes away when allowing any application to read from anywhere in the disk
	- Implement file system to manage files (library)
- Idea of System Call by Atlas Computing System
	- Add special pair of hardware instructions and hardware state to make the transition into the OS a more formal controlled process
	- System call transfers control into OS while simultaneously raising hardware privilege level
- User Applications run in user mode
	- Hardware restricts what user can do
- System Call initiated through hardware called trap, hardware transfers control to pre-specified trap handler
	- Raises privilege to Kernel Mode 
- When OS done servicing request, reverts to user mode in return to trap instruction 

Classic machines like the PDP family from Digital Equipment made computers hugely more affordable; thus, instead of having one mainframe per large organization, now a smaller collection of people within an organization could likely have their own computer
- Increased developer activity
	- Specifically multiprogramming
	- CPU would load multiple jobs instead of 1 and switch rapidly between them improving CPU utilization 
		- Switching because I/O devices were slow 
		- Having a program wait on CPU while I/O was being services was a waster of time
	- Issues such as memory protection became important
	- Concurrency issues
- Development of UNIX operating system - Ken Thompson & Dennis Ritche at Bell Labs (phone company)
	- Took ideas from different operating systems such as Multics and Tenex and Berkely Time Sharing System but made them simpler and easy to use
	- Inspired a lot by Multics from MIT
	- Unifying principle of building powerful programs that could be connected together to form larger workflows
	- Shell to type commands with pipes to enable meta level programming 
	- grep to find stuff, wc word count
	- Provided compiler for C
	- Early form of OSS
- Enterprising group at Berkely led by Bill Joy made Berkely Systems Distribution
	- Later co-founded Sun Microsystems - SunOS
- AIX from IBM
- HPUX from HP
- IRIX from SGI

Step backward from DoS (Disk Operating System from Microsoft) as didn't think of memory protection

Early versions of Mac took cooperative approach to scheduling and was stuck on infinite loops


[[Chapter 4 - The Abstraction - Process]]

