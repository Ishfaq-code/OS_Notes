- When using disk, we sometimes wish it to be faster
	- I/O operations are slow and can bottle neck entire system
- Sometimes need disk to be larger
	- More data being put online
- More reliable
- **Crux: How to make a large, fast, reliable, Disk?**'
- RAID: A technique to use multiple disks in concert to build a faster, bigger and more reliable disk system
	- Introduced in late 1980s by group of researchers at UC Berkely
		- Professor David Patterson and Randy Katz, then student Garth Gibson
	- Many researchers cam to the basic idea of using multiple disks to build a better storage system
	- Externally looks like a disk
		- A group of blocks one can read or read
	- Internally complex
		- Consisting of multiple disks, memory and one or more processors to manage the system
	- Hardware RAID much like computer system specialized for the task of managing a group of disks
	- Advantage
		- Performance
			- Multiple disks in parallel can greatly speed up I/O times
		- Capacity
		- Reliability
			- Adds some form of redundancy to tolerate the risk of losing a disk and keep operating if nothing were wrong 
	- Provides advantages transparently
		- OS does need modification
		- Improves deployability of RAIDs 
			- No need to worry about software compatibility


**Interface and RAID Internals**
- RAID looks like a big, fast and reliable disk
- Presents itself as a linear array of blocks 
- When file system issues a logical I/O request to RAID
	- Internally calculate which disk to access in order to complete the request
	- Issue one or more physical I/Os to do so
		- Depend on RAID level
- Often build with a separate hardware box with a standard connection to a host 
	- Internally complex consisting of a microcontroller that runs firware to direct operation of RAID, volatile memory such as DRAM to buffer data blocks as they are read and written and some cases none volatile memory to buffer writes safely and perhaps even specialized logic to perform parity calculations

**Fault Model**
- Fail Stop
	- Disk can be in a exactly one or two states
		- Working or failed
		- Work disk, all blocks can be written or read
		- Failed, we assume permanently lost
		- Assumes fault detection
			- When a disk has failed, we assume it is easily detected 
			- Do not worry about complex "silent" failures such as disk corruption
			- Do not have to have to worry about single block becoming inaccessible
			- Consider more complex disk faults later

**How To Evaluate Raid**
- Capacity
	- Given N disks each with B blocks, how much useful capacity is available to clients of the RAID
		- N * B
		- If mirroring used (N * B ) / 2
- Reliability
	- How many disk faults can the given design tolerate
- Performance
	- Depends on workload presented to the disk array
- RAID Levels: Pioneering work of Patternson, Gibson and Katz at Berkely

**RAID Level 0: Stripping**
- Not a level at all - no redundancy
- Serves as upper bound on performance and capacity 
- Spread the blocks of the array across the disks in a round robin fashion
	- Extract most parallelism from array when requests are made for contiguous chunk
	- Call blocks in the same row stripes, blocks 0,1,2 and 3 are in same stripe as above
	- Place multiple blocks per chunk
- Chunk Size
	- Mostly affects the performance of the array
		- Small chunk size implies that many files will get striped across many disks
		- Increasing parallelism of reads and writes to single file
		- Positioning time to access blocks across multiple disks increases
			- Determined by maximum of the positioning times of the requests across all drives
		- Big chunk size reduces parallelism
			- Requires multiple concurrent requests to achieve high throughput
			- Reduce positioning time
		- Best chunk size is hard to find
			- Great deal of knowledge of workload needs to presented
- Raid 0 Analysis
	- Capacity: N * B blocks of useful capacity
	- Reliability: Any disk failure will lead to data loss
	- Performance: All disks are utilized, often in parallel
- Evaluating Raid Performance
	- Single Request Latency
		- Latency of a single I/O request to a RAID is useful as it reveals how much parallelism can exist during a single logical I/O operation
	- Steady State Throughput
		- Total bandwidth of many concurrent requests
	- Sequential Workloads perform better than Random
- Raid 0 Performance
	- Latency: Identical to that of a single disk
		- Simply redirects requests to one of it's disks 
	- Steady State
		- Full bandwidth of the system
		- N (number of disks) * S (sequential bandwidth of a single disk)


**RAID Level 1: Mirroring**
- Make more than one copy of each block in the system
	- Each copy placed on a separate disk
	- Tolerate disk failures
	- Assume each logical bloc, the RAID keeps two physical copies of it
	- Can read Either Copy
- Analysis
	- Capacity
		- Expensive: (N * B) / 2
	- Reliability
		- Does well
		- Can tolerate failure of any one disk
		- Tolerate up to N/2 failures depending on the case
	- Performance
		- Latency
			- Same latency as a single disk
			- All it does is direct to read one of it's copies
			- Write is different
				- Two physical writes are required before it's done
				- Happen in parallel
				- Roughly equivalent 
				- Logical write must wait for both physical writes
					- Worst case seek and rotational delay of two requests, slightly higher than write of a single disk
		- Steady State Throughput 
			- Sequential Workload
				- Maximum bandwidth (N/2 * S): Half the peak bandwidth
			- Random Reads are beast case
				- Full possible bandwidth N/2 * R MB/s
				- Writes are N/2 * R
					- Each logical write must turn into two physical writes and all the disk will be in use, client only perceives use of half the bandwidth 


**RAID Level 4: Saving Space With Parity**
- Parity based approach attempt to use less capacity and overcome huge space penalty of mirrored systems
- Cost of performance
- Each stripe of data, a single parity block that stores  the redundant information for that stipe of blocks
- Use XOR to compute parity
	- Given set of bits, XOR of all bits return 0  if there are even number of 1's and 1 if odd
	- If a column is lost, read all values and reconstruct the right answer
	- Perform bitwise XOR across each bit of the data blocks, put the result of each bitwise XOR into corresponding bit slot in the parity block
- RAID 4 Analysis
	- Capacity: (N-1) * B
	- Reliability
		- Tolerates 1 disk failure and no more
		- If more than 1 disk is lost, no way to reconstruct data
	- Performance:
		- Steady State Throughput
			- Sequential read utilizes all of the disks except for parity disk: (N - 1)* S MB/S
			- Random Reads: (N - 1) * R MB/s
		- In Write we must correctly reflect parity values
			- Additive Parity
				- Read in all of the other data blocks in the stripe in parallel and XOR with new block
				- Results in new parity
				- Scales with Number of reads, inefficient in large RAID systems
			- Subtractive Parity
				- Read the old data and column and old parity
				- Compare old data and new data
					- If they are the same we know the parity bit also remains the same
					- If they are different, parity bit mist flip
				- Perform calculation over all bits in the block
					- Most cases new block will be different old block
			- (R/2)MB/s under random small writes
		- Latency is equivalent to that of a single disk
	- Small write problem: Modifying small amount of data requires large amount of I/O operations


**RAID Level 5: Rotating Parity**
- Rotates parity blocks across drives - addresses small write problem
- Analysis
	- Capacity and Fault Tolerance are identical to RAID 4
		- Same with latency
	- Random Read performs little better as we utilize all disks 
	- Random write improve noticeably 
		- Parallelism across requests
		- Keep all disks evenly busy
		- Small write bandwidth (N/4)R mb/s
			- Each RAID 5 write still generates 4 total I/O operations
- Replaced RAID 4 everywhere except
	- Systems where only large writes are performed


**Other RAID Issues**
- RAID 6 tolerate multiple disk faults
- Sometimes RAID has hot spare sitting around to fill in for filled disk
- Techniques for latent sector errors and block corruption
- Can be build as a software layer