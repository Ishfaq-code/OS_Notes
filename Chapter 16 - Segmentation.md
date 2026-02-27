Base and Bound approach is wasteful due the free space it leaves out
- Small address space
- Quite hard to run a program when the entire address space doesn't fit into memory

Crux: How to support a large address space

Generalized Base/Bounds
- Segmentation: Instead of having one base and bound pair, base and bound pair per logical segment of address space
	- Segment is a contiguous portion of the address space of a particular length
	- 3 logically different segments
		- Heap, Code, Stack
		- Segmentation allows the OS to place each one of those segments in different parts of physical memory and thus avoid filling physical memory with unused virtual spaced
		- Only used memory is allocated physical memory and thus large address spaces with large amounts of unused address space can be accommodated
	- MMU is required to support segmentation
		- 3 base and bound register pair
		- Each bound register holds size of segment
			- Tells hardware how many bytes are valid in this segment
	- Segmentation Fault: Arises from a memory access on a segmented machine to an illegal address

Which Segment are we Referring To:
- Uses segment registers during translation
	- Explicit Approach: Chop up address space into segments based on the top few bits of the virtual address
		- Was used in VAX/VMS system
		- If top 2 bits 00, virtual address in code segment and thus uses code base
		- If top 2 bits 01, heap space, use heap base
		- Take first 2 bits to find segment
		- Next 12 bits for offset
		- Add base register to offset, hardware gets to final physical address
			- Check if offset is less than bounds as well
		- To fully utilize segment, some systems put code in the same segment as heap and only use one bit to select segment
		- May limit use of virtual address space
			- Each segment is limited to a maximum size
			- Bad if growing stack or Heap
	- Implicit Approach: Hardware determines the segment by noticing how the address was formed
		- If address was generated from program counter, address within code segment
		- If generated from stack or base pointer, must be stack address
		- Anything else must be heap

Stack
- Stack grows backwards (to lower address)
	- Hardware needs to know which way segment grows (could use a bit, 1 for growing positive)
	- If segments can grow in negative direction, translations need to be done differently
		- Subtract maximum segment size from offset
		- Add negative offset to base
		- Bound check done by taking the absolute value of the negative offset

Support for Sharing
- To save memory, useful to share certain memory segments between address spaces
	- Code sharing is common and still used in systems to this day
- Need to enforce protection bits - support from hardware
	- Add few extra bits per segment to indicate whether or not a program can read or write a segment or perhaps execute code
	- By setting a code segment to read-only same code can be shared across multiple process without harming other process while each process thinks it's still in it's own private memory
	- Need to now check also whether a particular access is permissible 


Fine-grained vs. Coarse-grained Segmentation
- Coarse Grained: Chops address space up into relatively large coarse chunks
- Fine Grained: Large number of smaller chunks
	- Early systems such as Multics
	- Required further hardware support with a segment table
		- Support the creation of very large amounts of segments
		- Use segments more flexibly 
		- Burroughs B500 supported thousands of segments and expected a compiler to chop code and data into separate segments
		- Idea was OS could better learn about which segments are in use and which are not 


OS Support:
- Context Switching
	- Save segment registers and restore them in each process own virtual address space before letting process run again
- Growth and Shrink of process
	- Free space can be found if there is enough available in the heap
	- System call to grow the heap (Unix `sbrk()`) 
	- OS then provides more space updating the segment size register to the new bigger size and informing the library of the success
	- Library can allocate space for new object and return successfully
	- OS could reject the request if no more physical memory is available
- Managing Free Space in physical memory
	- Problem due to different sized segments
	- Physical memory quickly becomes little holes of free space making it harder to allocate new space - External Fragmentation
		- Compact physical memory by rearranging the existing segments
			- Stop whichever process are running
			- Copy data to one contiguous region of memory
			- Change segment register values to point to new physical locations
			- Expensive - copying segments is memory intensive and uses a fair amount of processor time
		- Use free list management algorithm that tries to keep large extents of memory available for allocation
			- Best Fit: Keeps a list of free spaces and returns the one closes in size that satisfies the desired allocation to the request

[[Chapter 17 - Free Space Management]]