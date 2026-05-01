- Solid State Storage
	- No mechanical or moving parts like hard drives
	- Simply built out of transistors, much like memory and processors
	- Retains information despite power loss
- Flash
	- NAND Based Flash
	- Fujio Masuoka in 1980s
	- To write to a chunk of it, you have to erase a bigger chunk
	- Writing too often to a page will cause it wear out
- **Crux: How To Build a Flash Based SSD**

**Storing a Single Bit**
- Designated to store one or more bits in a single transistor
	- Level of charge stored mapped to a binary value
- Single level cell flash (SLC), only single bit is stored (0, 1)
- Multi Level Cell (MLC) two bit are encoded into different levels of charge (00, 01, 10,11)
	- Low
	- Somewhat low
	- Somewhat high
	- high
- Triple Level Cell 3 bits
- SLC chips achieve higher performance and are more expensive


**From Bits to Banks/Planes**
- Flash chips organized into banks or planes which consist of a large number of cells
- Bank is accessed in two different sized units
	- Blocks (erase blocks) typically size of 128 kb or 256 kb
	- Pages, few KB in size (4 KB)
	- Large number of blocks or pages within each bank

**Basic Flash Operations**
- Read (a page)
	- Client of the flash chip can read any page by specifying read command and appropriate page number to the device
	- Quite fast, 10s of microseconds regardless of location on the device and location of previous request
	- Random Access Device
- Erase (a block)
	- Before writing to a page within a flash, erase entire block the page lies within
	- Destroys contents of the block
	- Takes few milliseconds to complete
- Program (a page)
	- Once block has been erases, program command used to change some of the 1s within a page to 0s and write desired contents of page to flash
	- Takes around 100s of microseconds on modern chips
- Each page has a state associated with it
	- Start in invalid state
	- Erasing block sets to erased
	- Programming ap page sets it to valid 
	- Read does not affect states
	- Once a page has been programmed, only way to change content is to erase entire block

**Flash Performance And Reliability**
- Flash chip are pure silicon and in that sense have fewer reliability issue
- Primary concern is wear out
	- Erase and Programming accrues extra charge
	- Overtime builds and gets difficult to differentiate between a 0 and 1
	- MLC based block have 10,000 P/E cycle
	- SLC based are longer with 100,000 P/E cycle
- Disturbance
	- When accessing a particular page within a flash, possible that some bits get flipped in neighboring pages
		- Known as read or program disturbs

**From Raw Flash to Flash Based SSDs**
- Standard storage interface is simple block based one
	- Purpose of flash based SSD is to provide standard block interface atop raw flash chips inside
- Internally SSD contains number of flash chips
- Flash Translation Layer (FTL) provides essential function to control logic to satisfy client reads and writes turning them into internal flash operations
	- Takes read and write requests on logical blocks and turns them into low level read , erase and program commands on the underlaying physical books and physical pages
	- Utilize multiple flash chips in parallel for excellent performance
	- Write amplification for performance as well, defined as total write traffic issued to flash chips by FTL divided by total write traffic issued by client to SSD
		- High write amplification leads to low performance
	- To avoid wear outs, FTL should spread write across blocks of the flash as evenly as possible ensuring all blocks of device wear out at roughly the same time
		- Wear leveling
	- To avoid disturbance FTL will program pages within erased block in order from low page to high page


**FTL Organization: A Bad Approach**
- Direct Mapped
	- Read Logical Page N is mapped directly to Physical page N
	- Write first needs to erase entire block on page N, then FTL programs old page to new one
- Write is costly, read, erase and program entire blocks
- Bad reliability as data is often overwritten

**Log Structured FTL**
