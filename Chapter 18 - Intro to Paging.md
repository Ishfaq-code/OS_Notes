**Paging**: Chop up space into fixed sized pieces
- Different from segmentation which uses variable sized pieces
- Introduced in Atlas
- Instead of splitting process address space into some number of variable sized logical segment
	- Divide into fixed sized units called a page
	- Physical memory viewed as an array of fixed sized slots called page frames
		- Each contain a single virtual memory page
 **Crux:** How to virtualize memory with Pages

**Overview**
- Flexibility
	- Fully developed paging approach, the system will be able to support the abstraction of an address space. effectively regardless of how a process uses address space
	- Won't make assumption about direction the heap and stack grow and how they are used
- Simplicity
	- Easy to find pages
	- OS keeps a free list of all free pages for this and grabs first four pages
- To record where each virtual page of the address space is placed in physical memory, the operating system keeps a per process data structure known as a page table
	- Stores address translation for each of the virtual pages of the address space
	- Per process structure
		- Each process data maps to different physical page
- To translate virtual address that the process generated we have to split it into two components
	- Virtual Page Number
		- 2 bits
	- Offset within a page
		- 4 bits
	- Page size is 16 bytes in a 64 byte address space 
		- We can select 4 pages and the top 2 bits of the address do that
		- Remaining 4 bits tell us which byte of the page we are interested in
			- The offset in this case
- When process generates a virtual address space, the OS and hardware must combine to translate into meaningful physical address
	- Address '21' becomes 010101
		- First 2 bits -> 01 virtual page 1
		- Last 4 bits -> 0101, 5th byte
		- 5th of virtual page 1
- PFN: Physical Frame Number
	- Translate virtual address by replacing VPN with PFN and then issue the load to physical memory
	- Only VPN translated, offset stays the same as it just tell us which byte within the page we want

**Page Table Storage**
- Store page table for each process in memory somewhere
	- They can get very large
	- Assume for now they live in physical memory which OS manages

**Contents of Page Table**
- Just a DS to map virtual addresses to physical address
- Simplest form - Linear Page Table
	- Array
	- OS indexes the array by the virtual page number and looks up page table entry at the index in order to find physical frame number
- Contents of PTE - number of different bits
	- A valid bit is common to indicate whether the particular translation is valid 
		- Program running there is Code at 1 end and Heap at the other
		- All unused spaced in between marked invalid
		- Valid bit marks all the unused pages in the address space as invalid
			- Remove the need to allocate physical frames for those pages and save memory
	- Protection bits indicating whether the page could be read from, written to or executed from
	- Present bit indicates whether the page is in physical memory or on disk
	- Dirty bit indicates whether the page has been modified since it was brough into memory
	- Reference bit is used to track whether a page has been accessed and is useful in determining which pages are popular

**Paging Also Too Slow**
- Page Tables are problem
	- They can be too large
	- They can slow down memory access
		- `movl 21 %eax`
			- Translate virtual address into physical address
			- Fetch the page table entry 
			- Extract the physical frame number
			- Combine PFN with offset
			- Fetch actual data from physical memory
				- Extra memory access to perform translation
- Hardware to PTE
	- Page Table Base Register contains the physical address of the start of the page table
		- Extract VPN
			- `VPN = (VirtualAddress & VPN_MASK) >> SHIFT`
		- Compute Address of PTE
			- `PTEAddr = PageTableBaseRegister + (VPN * sizeof(PTE))`
		- Extract Physical Frame Number
		- Extract offset from virtual address space
			- `offset = VirtualAddress & OFFSET_MASK`
		- Construct physical address
			- `PhysAddr = (PFN << SHIFT) | offset`



[[Chapter 19 - Paging Faster Translations]]