**Crux:** How to integration I/O into Systems


**System Architecture**
- Single CPU attached to the main memory via a memory bus
- Some devices connected to the system via a general I/O bus
	- Many modern systems would be PCI
	- Graphics and some other higher performance I/O devices might be found here
- Lower down is Peripheral bus
	- SCSI, SATA USA
	- Connect slow devices to system such as mice and keyboard
- Why need a hierarchical structure?
	- Physics and Cost
	- Faster a bus is , the shorter it must be 
		- High performance memory bus does not have much room to plug devices and such into it
	- Engineering a bus for high performance is costly
		- Components that demand high performance are near the CPU
		- Lower performance components are further way 
			- Benefits of placing disks and other slow devices on the peripheral bus is manifold - you can place large number of devices on it
- Intel Z270 Chipset
	- ![[Pasted image 20260425151650.png]]
	- CPU connect most closely to memory system, high performance connection to the graphics card
	- CPU connection to an I/O chip via Intel's proprietary DMI and the rest of the devices connect to the chip via a number of different interconnects.
	- One or more hard drives connect the system via eSATA interface
	- Below I/O chip are number of USB connections which in this description enable a keyboard and mouse to be attached to the computer
	- Network interface connected via PCIe (peripheral Component Interconnect Express)
		- NVMs are also connected here



**Canonical Device**
Device has two important components:
- Hardware interface it presents to the rest of the systems
- Internal Structure
	- Responsible for implementing the abstraction the device presents to the system
	- Very simple devices will have one or few hardware chips to implement their functionality
	- More complex devices will include a simple CPU, some general purpose memory and modern RAID controllers might consist of hundreds of thousands of lines of firmware to implement functionality


**Canonical Protocol**
- Device interface is comprised of three register
	- Status register to read the currant status of the device
	- Command register to tell the device to perform a certain task
	- Data register to pass data to the device or get data from the device
	- By reading and writing these registers, the OS can control device behaviors
- Protocol has 4 steps:
	- OS waits until the device to ready to receive a command by repeatedly reading the status register - Polling the device
	- OS sends some data down to the data register
		- When main CPU is involved with data movement, we refer to it as programmed I/O
	- OS writes a command to the command register
		- Lets device know that data is present and it should begin working on the command
	- OS waits for the device to finish again by polling in a loop waiting to see if it has finished
- Basic Protocol is simple and working
	- Inefficiencies and inconveniences involved
		- Polling is inefficient - wastes great deal of CPU time just waiting for the device to complete its activity instead of switching to another ready process thus better utilizing the CPU 
		- **Crux:** How to avoid costs of Polling

**Lowering CPU Overheads with Interrupts**
- Instead of polling the device repeatedly, OS can issue a request, put the calling process to sleep, context switch to another task
- When device is finished with operation, it will raise an hardware interrupt causing CPU to jump into the OS at predetermined Interrupt service Routine or Interrupt Handler 
	- The handler is just a piece of operating system code that will finish the request and wake the process waiting for the I/O, which can then proceed as desired
	- Allow for overlap of computation and I/O
- If the device is fast, still best to poll; if it's slow, interrupts/overlaps are best
- If speed of device is not known, best to us a hybrid that polls for little while and then if not finished, uses interrupts
	- Two-phased approach
- Not to use interrupts in Networks
	- When huge stream of incoming packets each generate an interrupt, it is possible for the OS to livelock
		- OS finds itself only processing the interrupts and never allowing a user-level process to run and service the requests.
- Coalescing - Interrupt based optimization
	- Device raises an interrupt first, waits for a bit before delivering the interrupt to the CPU
	- While waiting, other requests can be coalesced into a single interrupt delivery, thus lowering the overhead of interrupt processing

**More Efficient Data Movement with DMA**
- Large chunk of data is transferred to a device so CPU is over burdened
- **Crux: How to Lower PIO Overheads**
	- Direct Memory Access (DMA)
		- A very specific device within a system that can orchestrate transfers between devices and main memory without much CPU intervention
		- To transfer data, OS would program DMA engine by telling it 
			- where the data lives in memory
			- how much data to copy 
			- which device to send it to
		- OS is then done with transfer and can proceed with other work
		- When DMA is complete, DMA controller raises an interrupt and OS known its complete

**Methods of Device Interaction**
- **Crux:** How to Communicate with Devices
	- Oldest Method (used by IBM mainframes): explicit I/O Instructions
		- x86 `in` and `out` instructions can be used to communicate with devices
			- Caller specifies a register with the data in it and specific port which names the devices
		- Privileged instructions
		- OS controls devices and the OS is only entity allowed to directly communicate with them
	- Memory Mapped I/O
		- Hardware makes device registers available as if they were memory locations
			- OS issues a load to read
			- Store to write the address 
			- The hardware then routes the load/store to the device instead of main memory

**Fitting into the OS: The Device Driver**
- **Crux: Build a Device Natural OS**
	- Problem solved through abstraction
		- Lowest level, a piece of software in the OS knows in detail how a device works - Device Driver
		- Linux Software Organization
			- File system is oblivious to specifics of which disk class it is using
			- Simply issues block read and write requests to general block layer which routes to them appropriate device driver 
				- Handles details of issuing specific requests
			- Raw interface to devices
				- Enables special application to directly read and write blocks without file abstraction
				- Most systems provide this type of interface to support low-level storage management systems
			- Downsides
				- Special capabilities may go unused for using generic interface

**Case Study**
- IDE disk present simple interface to system consisting four types of register
	- Control
	- Command
	- Status
	- Error
- Registers are available by reading or writing to specific I/O addresses
- Basic Protocl
	- Wait for drive to be ready. Read Status Register (0x1F7) until drive is READY and not BUSY.
	- Write parameters to command registers. Write the sector count, logical block address (LBA) of the sectors to be accessed, and drive number (master=0x00 or slave=0x10, as IDE permits just two drives) to command registers (0x1F2-0x1F6).
	- Start the I/O. Write READ|WRITE command to command register (0x1F7).
	- Data transfer (for writes): Wait until drive status is READY and DRQ (drive request for data); write data to data port
	- Handle interrupts. In the simplest case, handle an interrupt for each sector transferred; more complex approaches allow batching and thus one final interrupt when the entire transfer is complete.
	- Error handling. After each operation, read the status register. If the ERROR bit is on, read the error register for details.

**Historical Notes:**
- UNIVAC early 1950s had some form of interrupt vectoring 
- Debate between invention of DMA
	- DYSEAC or IBM Sage