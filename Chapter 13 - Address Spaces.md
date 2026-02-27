Early Systems
- OS set of routines/libraries that sat in memory
- One running process sat in physical memory
	- Used the rest of memory
- Memory then made of OS + 1 Process (Current Process)

Multithreading & Time Sharing
- Multiprogramming because machines were expensive and people wanted to share machines more effectively 
	- Multiple process and CPU would switch between them
	- Effective utilization of CPU
		- Increases efficiency when machines were hundreds of thousands or even millions 
- Time Sharing born because programmers noticed the limitations of batch computing where there long program-debug cycles
	- Notion of interactivity became more important as users might be concurrently using a machine waiting for a timely response from their currently executing task
- Bad implementation of time sharing:
	- One process gets full memory access
	- Stop the process and save all its states to a disk (including all physical memory)
	- Load other process into memory
	- Way too slow
		- Particularly as memory grows
		- Saving and restoring register-level state is fast
		- Saving entire contents of memory to disk is brutally non performant
		- Solution: Leave process in memory while switching between them
			- Process sit in memory in queue while one is running waiting to run
				- Easier context switch
			- Protection becomes an issue as concurrent process in memory may read or write to some other process

The Address Space
- Address Space: Running program's view of the memory in the system
	- Easy to use abstraction of the physical memory
- Contains all of the memory state of running program 
	- Code is static so it can be placed at the top of the address space as it doesn't need to grow
		- Placed at 0 to first 1k memory
	- Heap placed on top next and stack on the bottom
		- Placed as such as they grow in opposite direction
		- Heap grows downwards
		- Starts at 1KB memory
- Abstraction of physical memory
	- Loaded at some arbitrary physical address
	- OS virtualizing the memory 
		- Running program thinks it is loaded into memory at a particular address and has a potentially very large address space, 32 - 64 bits
		- The reality is different 
			- If process A tries to perform a load at address 0 (virtual address) , OS needs to ensure with some hardware support the load doesn't actually go to physical address 0 but rather physical address 320kb (where A is in the memory)

Goals
- Transparency
	- Virtual memory should be implemented in a way that is invisible to the running program 
	- Program not aware of the fact that memory is virtualized
		- Acts like it has it's own private physical memory
- Efficiency
	- Efficient in terms of time and space
		- Don't make programs run slowly
		- Not using too much memory for structures needed for virtualization
		- Needs to rely on TLB
- Protection
	- Protect process from each other and OS from process
	- Property isolation among processes

Microkernels: Wailing off pieces of the OS from other pieces of the OS

**Crux:** How to Virtualize Memory

[[Chapter 15 - Address Translation]]

