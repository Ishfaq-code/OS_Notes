- Main form of persistent data storage in computer systems
- **Crux: How to Store and Access Data on Disk**

**Interface**
- Consists of large number of sectors (512-byte blocks) each of which can be read or written
	- Numbered 0 to n-1 on a disk with n sectors
	- Array of sectors
	- 0 to n-1 is the address space of the disk
- Multi-sector operations are possible
	- Many file systems will read or write 4KB at a time
	- When updating the disk, the only guarantee drive manufacturers make is that a single 512 byte write is atomic
		- If an untimely power loss occurs, only portion of larger write maybe complete (torn write)
- Schlosser and Ganger "unwritten contract" of disk drives
		- One can usually assume that accessing two blocks near one another within the drive's address space will be faster than accessing two blocks that are far apart
- Accessing blocks in a contiguous chunk is the fastest access mode (faster than random access pattern)


**Basic Geometry**
- Platter: Circular hard surface on which data is stored persistently by inducing magnetic charges to it
	- A disk may have one or more platter
	- Each platter has two sides called the surface
	- Made of hard materials, ex: Aluminium
	- Coated with a thin magnetic layer that enables the drive to persistently store bits even when the drive is powered off
- Platters are all bound together around a spindle
	- Connected to a motor that spins the platters around
	- Rater of rotation is measured in RPM
		- Typical modern values are in 7200 ton 15000 RPM range
		- Interested in a single rotation
- Data is encoded in each surface of the concentric circles of sectors
	- Called a track
		- Single surface contains many thousands and thousands of tracks tightly packed together
	- To read and write from surface mechanisms are needed
		- Accomplished by disk head
			- One such head per surface of the drive
			- Attached top a disk arm - moves across the surface to position the head over the desired track


**Simple Disk Drive**
- Single Track Latency: Rotational Delay
	- Wait for desired sector to rotate under the disk head
		- Happens often enough in modern drives and is important component of I/O service time
			- Called Rotational delay
			- If full rotational delay is `R` then disk has to incur a delay of about `R/2` to wait for 0 to come under read/write head (if we start at 6 on 12 sector disk)
				- Worst case would be to request sector 5 by a single track, full rotational delay
- Multiple Tracks: Seek Time
	- Modern disks have millions of tracks
		- Sets of sectors used (multiple tracks)
		- Goes from innermost to outermost
		- Disk first moves arm to the correct track in a process called seek
			- One of the most costly operation along with rotations
			- Has many phases
				- Acceleration phase as the disk arm gets moving
				- Coasting as arm is moving at full speed
				- Deceleration as arm slows down
				- Settling as head is positioned over correct track
					- Settling time is significant as drive must be certain to find right track
		- Final phase of I/O after disk finds correct sector: Transfer
			- Data is read from or written to the surface
- Other details
	- Many drives employ some kind of track skew to make sure sequential reads can be properly serviced even when crossing track boundaries
	- Sectors are skewed like this because when switching from one track to another, the disk needs time to reposition the head
		- Without such skew, head would be moved to the next track but desired next block would have already rotated under the head
	- Outer tracks tend to have more sectors than inner track
		- More room out there
		- Multi zoned disk drives, disk organized into multiple zones
	- Important part of modern disk drive is cache
		- Called track buffer historically
		- Small amount of memory (8-16 MB) which drive can hold data read or written to disk
		- Allows drive to quickly respond to subsequent requests on the same track
	- Drive has choice on writes
		- Acknowledge the write has completed when it has put data in its memory (cache)
			- Called write back
			- Sometimes make drive appear faster
			- Can be dangerous as can be written to disk in certain order of correctness
		- Acknowledge after has been written to disk
			- Write through

**I/O Time**
- `I/O Time = Seek Time + Rotation Time + Transfer Time`
- `Rate of I/O = Size of Transfer/ I/O Time`
- Cheetah 15k.5 - High performance SCSI drive
	- High performance is marketed to spin as fast as possible
	- Deliver lower seek times
	- Transfer data quickly
- Barracuda - Built for capacity
	- Cost per byte is the most important aspect
	- Drives are slower but pack as many bits as possible
- Experimentation shows sequential is faster than random workload
	- Random: Reads to random locations on the disk
		- Common in database management systems
	- Sequential: Reads a large number of sectors consecutively from the disk without jumping around
- Large performance different between high end performance drives, and low end capacity drive

Average Seek distance on a disk over all possible seeks is 1/3 the full distance

**Disk Scheduling**
- OS plays role in deciding the order of I/O issued to the disk
	- Given a set of I/O requests, the disk scheduler examines the requests and decides which one to schedule next
	- Need to make a good guess for how long a job will take
		- Using seek and possible rotational delay of a request, the disk scheduler can know how long each request will take and pick the shortest one
		- Followings SJF principle
- Shortest Seek Time First
	- Early disk scheduling approach
	- Order queue of I/O requests by track
		- Picks requests on the nearest track to complete first
	- Not a panacea
		- Drive is not gematrically available to OS - appears as array of block
			- OS can implement nearest block first
		- Starvation
			- Steady stream of request to one track will ignore the requests to other track in pure SSTF approach
			- **Crux: How to handle disk starvation**
- Elevator (SCAN or C-Scan)
	- Originally called SCAN
	- Moves back and forth across the disk servicing requests in order across the tracks
		- Single pass across the disk a sweep
		- If a request comes for a block on a track that has already been serviced on a track that has already been serviced on a sweep of the disk
			- Not handled immediately and queued until next sweep
	- F-SCAN, Coffman et al
		- Freezes queue to be serviced when it is doing a sweep
		- Places requests that come in during the sweep into a queue to be serviced later
		- Avoid starvation of faraway requests by delaying servicing late arriving requests
	- C-SCAN
		- Circular SCAN
		- Only sweeps from outer to inner and resets the outer track to begin again
			- More fair to inner and outer tracks
			- SCAN favors middle tracks tracks
	- Referred as elevator as it behaves like an elevator going back and forth
	- SCAN and SSTF ignore rotation
		- **Crux: How to Account for Disk Rotation Costs**
- SPTF: Shortest Positioning Time First
	- For times when rotation is faster than seek time
- Other Scheduling Issues
	- Where the disk scheduling is performed on modern systems
	- Older systems OS did all the scheduling
	- Modern systems, disks can accommodate multiple outstand requests and have sophisticated intern schedulers themselves
		- OS scheduler picks what it thinks are the best requests and issues them all to disk 
		- Disk uses internal knowledge and head position along with detailed track layout information to service said requests
	- I/O Merging
		- Merging close requests into a block
		- Important as it reduces number of requests sent to the disk and lowers over head
	- How long the system should wait before issuing an I/O disk 
		- Naive: Immediately after having a single I/O request
			- Work conserving
		- Research shows best to wait for a bit
			- A better request may arrive at the disk and efficiency could be increased



**Livney's Law**
- It always depends