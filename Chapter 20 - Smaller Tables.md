**Crux:** How to make page tables smaller?

**Aside** Multiple Page sizes
- Many archetectures (MIPS, SPARC, x86-64) support multiple page sizes

**Simple Solution**
- Use bigger pages
- Big pages lead to waste within each pages
	- Internal fragmentation
	- Applications end up allocating pages but only using little bits and pieces of each and memory quickly fills up with these overly-large pages
- Most systems use relatively small page sizes in the common case

**Hybrid Approach: Paging and Segments**
- Multics Virtual Memory System
	- Combining paging and segmentation in order to reduce the memory overhead of page tables
	- Most of the page table is unused, full of invalid entry
		- Hybrid Approach: Instead of having a single page table for entire address space of the process, have one per logical segments
			- Base register points to the physical address of the page table of that segment
			- Bound registers used to indicate the end of page tables
			- On TLB miss, hardware uses the segment bits to determine which base and bounds pair to use
			- Bound Register per segment
				- Each register holds the value of the maximum valid page in the segment
		- Segmentation quite flexible
		- Hybrid causes external fragmentation to arise
			- Finding free space in memory becomes more complicated

**Multi Level Page Table**
- Linear page table into a tree
- Many modern systems adopt it (x86, BOH10)
- Chop up page table into page sized units
	- If entire page of page table entries is invalid, don't allocate that page of the page table at all
	- To see validity of page, use page directory
		- Can tell you page of the page table, or the entire page of the page table contains no valid pages
		- Simple two-level table contains one entry per page of the page table
			- Consists of page directory entries
				- Valid bit and Page Frame Number
				- Valid bit is slightly different
					- If PDE is valid, it means that at least one of the pages of the page table that the entry point to is valid, at least one PTE on page pointed to by the PDE , the valid bit in the PTE is set to 1
					- If PDE not valid, rest of PDE is not defined
- Only allocates page-table space in the proportion to the amount of address space you are using
	- Generally compact and supports sparse address space 
- Each portion of the page table fits neatly within a page making it easier to manage memory
	- OS can simply grab the next free page when it needs to allocate or grow a page table
- Level of indirection with the use of page directory which points to pieces of the page table
	- Allows us to place page-table pages wherever we would like in physical memory
- On TLB miss, two loads from memory will be required to get the right translation information from page table
	- Time-space trade off
	- Smaller table more time
- Complexity
	- More involved than linear look up

**Inverted Page Tables**:
- Single page table that has an entry for each physical page of the system
	- Each entry tells us which process is using this page, and which virtual page of that process maps to this physical page

Some systems put page tables in kernel virtual memory to allow system system to swap some of these page tables to disk when memory pressure gets too tight

[[Chapter 21 - Mechanisms]]