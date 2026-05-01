- vsfs - Very Simple File System
	- Simplified version of a typical UNIX file system 
	- Pure software
		- No hardware features
		- Flexibility 
			- AFS - Andrew File System
			- ZFS (Sun's Zettabyte File System)
	- **Crux: How to Implement a Simple File System**


**Overall Organization**
- Divide disk into blocks
	- Simple file systems just use one block size 
	- Common size of 4 KB
	- Each block of size 4 KB
	- Addressed from 0 -  (N - 1)
- Need to store data in blocks to build file system
	- Most of the space is user data
		- Called the data region
		- Reserve a fixed portion of these blocks for this 
	- Reserve space on the disk for inodes as well
		- Call this portion the inode table
			- Holds an array of on-disk inodes
		- Inodes are not that big, typically of size 128-256 bytes
		- Number of inodes represents the maximum number of files we can have in our file system 
			- Same file system built on a larger disk could simply allocate a larger inode table and accommodate more files
	- Allocation structures used to to keep track if blocks are free or allocated 
		- Requisite element in file system
		- Many structures possible
			- Free list to point to first free block, which then points to next free block and so forth
			- Bitmap - each bit used to indicate whether corresponding object/block is free or in use
	- Reserve superblock that contains information about particular file system
		- How many inodes, data blocks
		- Where indoe table and data blocks are in file system
		- Magic number to identify the file system type
		- When mounting the file system, OS reads the superblock first initialize various parameters and then attach the volume to the file system tree 

**File Organization: The Inode**
- Inode short for index node, historical term used in Unix
	- Reflects the nodes were originally stored in an array, which the system index into access a particular node
- In vsfs, given an i number (implicitly referred to number), calculate where on disk the corresponding inode is location
	- First calculate offset into the inode region ((i-number) * sizeof(inode))
	- Add it to the start address
	- Issue a read to sector as disk consist large number of sector (result * 1024/ 512)
	- `blk = (inumber * sizeof(inode_t)) / blockSize; 
	- `sector = ((blk * blockSize) + inodeStartAddr) / sectorSize;`
- Inode has file metadata
- Limited approach to how indoe refers to data blocks
	- Direct pointers inside indoe
	- Refers to one disk block that belongs to file
	- Limited as file may be bigger than block size * number of direct pointers
- Multi Level Index
	- Different structure within i-nodes to support bigger files
		- Indirect pointer
			- Points to block that contains more pointers, each pointing to user data
		- Inode may have a fixed number of direct pointer and a single indirect pointer
		- If file grows, indirect block allocated and inode's indirect pointer points to it
		- To support larger files, just add another indirect pointer to point another indirect pointer (double indirect pointer)
			- Add as many indirect pointers
			- Multi level index approach to pointing to file blocks
		- Used by Linu ext2 and ext3, NetApp's WAFL, original unix system
		- Other systems such as ext4 use extents
			- Disk pointer + length 
			- Instead of needing a pointer to each block just needs pointer and length
			- Allow more than extent as single extent may have trouble finding a contiguous chunk on-disk free space
		- Imbalanced design as most files are small


**Directory Organization**
- Just contains a list of pairs (entry name, inode number)
- For each file or directory in a directory, there is a string and a number in data blocks of the directory
- Each string may also have a length
- Each directory has two extra empties (dot and dot dot)
- Deleting an entry can leave an empty space in the middle of a directory and there should be  a way to mark it
	- Record length used
	- New entry may reuse an old, bigger entry and have extra space within
- FS treat directories as special type of file
	- Directory has inode in the inode table
	- Directory has data blocks pointed to
- XFS stories directories in B-tree form making file create operations faster than systems with simple lists

**Free Space Management**
- File system needs to track which inodes and data blocks are free and which are not

**Access Paths: Reading and Writing**
- Reading a File From Disk
	- File System first needs to find the inode for the file bar to obtain some basic information about the file
	- File system must traverse the pathname and locate the desired inode
		- All traversals begin in the root directory
			- Read inode of root directory
			- Must i-number to read inode
				- Found in the in parent directory
				- Root has no parent so the i-number is well known by FS
					- Most UNIX system it's 2
	- Once inode is read, the FS can look inside to find pointers to data blocks 
		- Use this to find the entry
		- Use inode here again
	- Recursively traverse the pathname until inode is found
	- Finally read inode of the found into memory
		- Does a final permissions check, allocates a file descriptor for process in per process the open file table and returns it to the user
	- Once open, system can issue `read` system call
	- Amount of I/O genereted by the open is proportional to the length of the pathname
- Writing a file to disk
	- File opened as the read process
	- Issue `write system call`
		- Allocate a block
		- Each write not only has to write data to disk but has to decide which block to allocate to the file and update other structures of the disk accordingly
			- Each write generally generates 5 I/O
			- Worse for creating files as it needs to allocate inode, bitmap AND space within directory containing new file
			- **Crux: How To Reduce File System I/O Costs**
	- Close file

**Caching and Buffering**
- Each FS introduced a fixed-size cache to hold popular blocks
	- Strategies such as LRU and different variants would decide which blocks to keep in cache
	- Allocate at boot time to be roughly 10% of total memory
		- Static partitioning of memory can be wasteful
			- File system may not need to use all 10%
	- Modern systems employ dynamic partitioning
		- Integrate virtual memory pages and file system pages into unified page cache
- Write Buffering as caching not too useful (need to write to disk every time for persistence )
	- Delaying writes, file system can batch some updates into smaller set of I/Os
	- System can schedule subsequent I/Os and increase performance
	- Some writes are avoided all together by delaying
		- Writing and deleting instantaneously 