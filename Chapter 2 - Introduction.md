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

**Virtualizing the CPU:** Turning single CPU into a seemingly infinite number of CPUs and allowing many programs to seemingly run at once 

Memory accessed each instruction phase

Each process running has own private virtual address space which OS maps onto physical memory of the machine

**Atomically:** Values execute all at once

OS needs to be **persistent** so values are not lost or are not volatile  
- OS does not create a private virtualized disk for each application

OS provides standard and simple way to access devices through its system calls, thus OS is called standard library. 

Abstraction to make use easier

Overheads - Extra time and space to use features

**Protection:** Bad behavior of one process/program/OS does not harm the other
- Need to isolate process from one another

OS started out as a set of libraries at the beginning 
- Batch processing, number of jobs set up and run in batch by operator

File System for privacy
- Don't want to allow read from anywhere

[[Chapter 4 - The Abstraction - Process]]

