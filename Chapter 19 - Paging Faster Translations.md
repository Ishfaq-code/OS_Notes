- Paging as the core mechanism to support virtual memory can lead to high performance overheads
	- Chopping address space into small fixed-sized units requires large amount of mapping information 
	- Needs extra lookup as stored in physical memory

**Crux:** How to speed up address translations 

Translation-Lookaside Buffer: Part of chip's MMU and is simply hardware cache of popular virtual to physical address translation
- Address translation cache
- Check TLB before looking up


**TLB Basic Algorithm**
Assumes:
- Linear Page table (array based)
- Hardware managed TLB
Process:
- Extract VPN: Hardware extracts the VPN from the virtual address
- Check TLB: Hardware checks if the TLB contains a translation for the VPN
	- TLB Hit:
		- Extract page frame number from the TLB entry
		- Concatenate PFN with the offset of the original address
		- Form physical address
		- Perform protection checks
		- Access memory using computed physical address
		- Fast, no need to access the page table, TLB located near CPU Core, designed for high speed, low overhead
	- TLB Miss:
		- Hardware access the page table memory
		- Looks up VPN to find translation
		- Verifies that virtual memory reference
			- Valid
			- Accessible
		- Updates TLB with new translation
		- Retries the instruction
		- On retry the translation now found in TLB
		- Memory access proceeds normally
		- Costly
			- Extra memory reference to page table
			- Complex page tables may require multiple memory references
			- Memory accesses are significantly more expensive than most CPU instructions
			- Frequent TLB misses noticeably slow program execution

**Aside: Caching**
- Caching one of the most fundamental performance techniques in computer systems, one used to make "common case fast"
- Take advantage of locality in instruction and data references
	- Temporal locality
		- Item has been recently accessed
	- Spatial locality

### CISC vs. RISC (1980s Architecture Debate)

- In the 1980s, computer architects debated **CISC (Complex Instruction Set Computing)** vs. **RISC (Reduced Instruction Set Computing)**
- **RISC leaders**: David Patterson (Berkeley), John Hennessy (Stanford), and early contributor John Cocke.

### CISC
- Large number of complex, powerful instructions.
- Instructions act as **high-level primitives** (e.g., a single instruction to copy a string).
- Goal:
    - Make assembly easier to use
    - Produce more compact code


### RISC
- Small set of simple, uniform instructions.
- Designed mainly as a **compiler target**.
- Philosophy:
    - Remove hardware complexity (especially microcode)
    - Keep instructions simple and fast
    - Optimize for high performance



### What Happened?
- Early RISC chips were significantly faster and gained popularity.
- Companies like MIPS and Sun emerged from the movement.
- Over time, CISC manufacturers (like Intel) adopted RISC techniques:
    - Breaking complex instructions into simpler micro-instructions
    - Using pipelining and other performance optimizations
- Increased transistor counts helped CISC remain competitive.

**Accessing an array**
- TLB hit  onto addresses that are next to each other
- Lower amounts of misses on larger page sizes
- Better ratio for loops as we already have them in cache
	- Temporal locality

**Who Handles TLB Miss**
- Hardware
	- Olden days hardware had complex instruction sets
		- CISC
		- Hardware has to know exactly where the page tables are located in memory
			- Through page table base regtister
		- Know their exact format
			- On miss, hardware would walk page table
			- Find correct page table entry
			- Extract desired translation
			- Update TLB with translation
			- Retry instruction
		- Intel X86
			- multi-level page table
- Software
	- Modern
		- RISC - Reduced instruction set computers
		- Software managed TLB
		- TLB Miss, hardware raises exception
			- Pauses current instruction stream
			- Trap handler
				- Code within OS for TLB misses
				- Lookup  translation, update TLB
				- Return from trap
					- Needs to different than a system call
					- Resume execution at the instruction after the trap into OS
					- Before resume execution at instruction that caused the trap
				- Needs to be extra careful running to code to not cause infinite chain of TLB misses to occur
					- TLB miss handlers in physical memory
					- Reserve entries in TLB for permanently to address translations
					- Flexibility
						- OS can use any data structure to implement the page table

**TLB Content**
- Might have have 32, 64, 128 entries
	- Fully associative
- Any given translation can be anywhere in the TLB
	- Hardware will search entire TLB in parallel to find the desired translation
- Entry
	- VPN | PFN | other bits
- Both VPN & PFN are present in each entry as a translation could end up in any of these locations
- Other bits
	- Valid
	- Protection



**TLB Issue: Context Switch**
- New issues arise when switching between processes
- TLB contains virtual to physical translations only valid for currently running process
	- Not meaningful for other process
	- Hardware needs to ensure processes don't use wrong TLB
- **Crux:** How to manage TLB Contents on a Context Switch
	- Flush TLB on context switches
		- Explicit and privileged hardware instructions
		- Hardware managed TLB
		- Flash enacted when the page-table base register is changed
		- Each time process run it must incur TLB misses as it touches its data and code pages
		- If context switch happens too often, cost may be too high
	- To reduce overhead, some systems add hardware support to enable sharing of TLB across switches
		- Address Space Identifier field in the TLB
			- Process identifier
			- Usually has fewer bits



**Replacement Policy**
- New entry to TLB means we have to replace the old one
- **Crux:** How to Design TLB Replacement Policy?
	- LRU - Least Recently Used Entry
		- Takes advantage of locality in the memory reference stream
	- Random Policy
		- Evicts at random
		- Useful due to ability to avoid corner case behaviors
		- LRU behaves unreasonably when program loops over n+1 pages



MIPSR400 supports a 32 bit address space with 4KB Pages
20 bit VPN, 12 bit offset

[[Chapter 20 - Smaller Tables]]