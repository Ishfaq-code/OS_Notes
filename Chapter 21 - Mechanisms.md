- We want to support many concurrently running large address spaces
	- Additional level in the memory hierarchy
	- Crux: How to Go Beyond Physical Memory
		- Convenience and ease of use
			- Large address space you don't have to worry about if there is enough room in memory for your program's data structures
			- Just write the program naturally allocate memory as needed
				- Powerful illusion by the OS
			- Older System: Memory overlays which required programmers to manually move pieces of code or data in and out of memory
			- Swap space allows OS to support the illusion of a large virtual memory for multiple concurrently running program



**Swap Space**
- Reserve space on the disk for moving pages back and forth
	- Swap pages out of memory to it and swap paged into memory from it
	- Assume OS can read and write to swap space in page sized length
	- Need to remember disk address of a given page
	- OS needs to remember the disk address of a given page
- Size of swap space is important and determines the maximum number of memory pages that can be in use by a system at a given time
	- Allows the system to pretend that memory is larger than it actually is


**The Present Bit**
- Hardware managed TLB
	- Need to add machinery for how hardware looks in the PTW as it may find the page is not present in physical memory
		- Use of Present bit in each PTE
		- If present bit is set to 1, it means page is present in the physical memory
		- If it is set as 0, page is not in the memory but rather on the disk somewhere
		- Act of accessing a page that is not in physical memory is called a page fault


**The Page Fault**
- Page Fault Handler responsible
	- Almost all software perform page faults using software
	- OS needs to swap page from disk to service a page fault
		- How will OS know
		- Information stored in page table
		- OS looks in PTE to find address and issues the request to disk to fetch the page into memory
			- Disk I/O completes the OS will update the page table to mark the page as present, update PFN field of the page table entry to record the in memory location of the newly fetched page and retry the instruction
				- Next attempt may be a TLB miss which would be serviced and update the TLB with the translation
				- Last restart would find the translation in the TLB and proceed to fetch the desired data or instruction from memory at the translated address
			- Process will be blocked
				- OS can run other process


**Memory is Full**
- Page out 1 or more pages and make room for new pages
- Process of picking a page to kick or replace is known as the page replacement policy
	- Wrong decision can cause a program to run at disk-like speeds instead of memory-like speeds


**Page Fault Control Flow**
- Page both present and valid
	- TLB miss handler grabs PFN from PTE retry the instruction to do TLB hit
- Page not present in physical memory
	- Run page fault handler
- Invalid page
	- Due to bug?
	- Hardware traps it
	- OS Trap handler to terminate forces
- Servicing page faults
	- Find physical frame for the soon to be faulted page to reside within
		- No page? Replacement algorithm to run and  kick some pages out of memory thus freeing them for use here
		- Read page from swap space
		- OS Updates page table and retries instruction
			- TLB Miss
			- TLB hit

[[Chapter 22 - Policies]]