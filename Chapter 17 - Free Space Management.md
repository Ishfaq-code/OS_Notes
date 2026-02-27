Free space management becomes difficult when free space you are managing consists of variable sized units.
- Arises in user level memory allocation
- OS managing physical memory using segmentation

**External Fragmentation:** Free space gets chopped into little pieces of different sizes and is thus fragmented
- Subsequent requests may fail because their is no single contiguous space that can satisfy the request, even if the total amount of free space exceeds the size of request

**Crux:** How to Manage Free Space

**Assumptions**
- When freeing memory using `free(void *ptr)` the user does not specify the size, the library must figure it out
- **Free List:** Data used to mange free space in the heap
	- Contains references to all of the free chunks of space in the managed region of memory 
	- Not necessarily a list but just needs to track free space
- Primary concerned about external fragmentation
- **Internal Fragmentation**: Hands out chunks of memory bigger than that requested, any unasked for chunk in that space is called Internal fragmentation
- Once memory is handed out to a client it cannot be relocated to another memory location
	- No compaction of free space is possible
- Compaction can be used in the OS to deal with fragmentation when implementing segmentation.
- Allocator manages a contiguous region of bytes
	- Region is a single fixed size of bytes

**Low Level Mechanisms**"
- **Splitting and Coalescing**
	- Splitting: Find a free chunk of memory that can satisfy the request and split it into two
		- First chunk it will return to the caller
		- Second chunk will remain on the list 
		- Happens when requesting memory less than the size of a chunk
	- **Coalesce:** When returning a free chunk in memory look carefully at the addresses of the chunk you are returning as well as the nearby chunks of free space
		- If newly freed space sites next to one or more existing free space chunks, merge them into a single free chunk
- **Tracking Size of Allocated Regions**
	**Header Blocks and Determining Allocation Size**
	- `free(void *ptr)` does **not** take a size parameter
		- The allocator must determine the size of the memory region being freed using only the pointer
	- The malloc library assumes it can quickly determine the size of the allocated block from metadata
	**Header Block**
	- Allocators store **extra information** in a header block
		- Header is kept in memory **just before** the memory chunk handed to the user
	- Purpose of the header
		- Determine size of allocated region during `free()`
		- Support integrity checks
		- Speed up deallocation
	**Example Allocation**
	- User calls `ptr = malloc(20);`
	- Allocated block consists of:
		- Header block
		- 20 bytes of user-accessible memory
	- `ptr` points to the start of the **user memory**, not the header
	**Header Contents**
	- Minimum information stored
		- Size of the allocated region
	- May also store
		- Pointers for faster free-list management
		- Magic number for integrity checking
		- Other allocator-specific metadata
- **Embedding A Free List**
	- Initially free list treated as a conceptual structure
		- List describing which chunks of memory are free
	- Free list must be stored inside the free list itself
		- Cannot call malloc
		- Allocator manage memory using the memory it controls
	- Initially contains
		- 1 Free chunk
		- Size = Total Heap - Header size
		- Heap memory acquired using mmap()
		- Free list node placed inside the heap itself
		- `head` points to start of heap
	- To accommodate a request:
		- Find a chunk that is large enough to accommodate the request 
			- Chunk will be then split into 2
				- One big enough to service the request
				- Remaining free chunk


**Growing Heap**
- What to do if heap runs out of memory
	- Simplest option to fail
		- Return NULL
- Most start with small sized heap and request memory from OS when out of memory`
	- Make some kind of system call to grow the heap and allocate new chunks from there

**Basic Strategies for Managing Free Space**
- Ideal allocator is both fast and minimizes fragmentation
- Best Fit
	- Search through the free list and find chunks of free memory that are as big or bigger than requested size
	- Return the one that is smallest in that group of candidates
	- Reduce wasted space
		- Naive implementation pay a heavy performance penalty when performing exhaustive search for the correct free block
- **Worst Fit**
	- Opposite of best
	- Find the largest chunk and return the requested amount
	- Keep the remaining large chunk on the free list
	- Costly as full search of free list can be required
		- Excess fragmentation
- First Fit
	- Finds the first block that is big enough and returns the requested amount to the user
	- Any remaining free space is kept for subsequent free requests
	- Advantage of speed
		- No exhaustive searching
	- Pollutes beginning of free list with small object
		- Use address based ordering
		- Keep list ordered by the address of the free list, coalescing becomes easier
- Next List
	- Keeps an extra pointer to the location within the list where one was looking list
	- Spread the searches for free space throughout the list more uniformly
		- Avoid splintering the beginning of the list

**Other Approaches**
- Segregated Lists
	- Maintain separate free lists for specific object sizes
	- Popular sized allocations get their own dedicated lists
	- All other allocation requests go to a general purpose allocator
	- Why help
		- Reduce fragmentation for frequently requested sizes
		- Faster allocation and free
			- No need to search a long free list
			- Requests of the right size are handled immediately
		- Complications
			- Decide between how much memory to allocate to specialized size-specific pool
				- How much to leave for allocators 
			- Pool sizing decision can waste memory or hurt performances
- Slab Allocator
	- Jeff Bonwick, Solaris
	- Well known elegant implementation of segregated lists
	- Designed for kernel memory allocation
	- Kernel boot
		- Allocate object caches for frequently used kernel objects
			- Locks
			- I/O
			- DS
		- Each object cache
			- Segregated free list for a fixed size object
			- Serves allocation and frees very quickly
- Interaction with General Allocator
	- When object cache runs low on memory
		- Requests one or more slabs from general allocator
		- Slabs allocated in
			- Multiples of the page size
			- Sizes proportional to the object being cached
		- When all objects in a slab have referenced counts of zero
			- Slab can be returned to the general allocator
			- Often triggered when VM system needs memories
- Pre initalized object
	- Freed objects are kept initialized in the ache
	- Avoid repeated
		- Object initialization
		- Object destruction
	- Reduces overhead significantly
- Buddy allocation
	- Designed to make coalescing simple and efficient 
	- Free memory treated as a single block of size 2<sup>n</sup>
	- Memory recursively split into halves during allocation
	- Process
		- Recursively divide free blocks by 2
		- Stop when
			- Block large enough to satisfy request
			- Further splitting would make blocks too small
			- Allocated blocks always power of size 2
			- Leads to internal fragmentation
				- User gets more memory than requested
	- Freeing and coalescing
		- When freed:
			- Allocator check if its buddy block is also free
			- If
				- Coalesces them into a larger block
			- Repeats recursively up the tree
			- Stops when
				- Buddy in use
				- or entire memory space is reassembled
	- Buddy blocks are easy to find
		- Address differ by exactly 1 bit
		- The differing bit depends on the tree level
	- Property makes coalescing fast and simple


[[Chapter 18 - Intro to Paging]]