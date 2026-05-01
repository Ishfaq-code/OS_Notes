**Crux: How to Manage a Persistent Device**

**Files & Directories**
- File: Liner array of bytes each of which you can read or write
	- Has some kind of low level name - usually a number of some kind
		- User is not aware of this
		- Often referred to as inode number (i-number)
	- Most systems, OS does not know much about the structure of the file
		- Responsibility of the file system to store such data persistently on the disk
- Directory
	- Also has a low level name
	- Contains list of pairs (user-readable name, low-level name)
		- ("foo" , 10)
		- Maps user readable name to low level name
		- Each entry refers to files or other directories
- Directory Tree/Hierarchy: Placing directories within other directories
	- Hierarchy starts at a root directory
		- Use separator or sub sequent sub directories until desired file or directory is named
	- Directories and files can have same names as long as they are in different locations in the file system tree 
	- First part of file name is arbitrary name and second part is type


**Creating Files**
- Accomplished by `open` system call and passing `O_CREAT` flag 
	- `int fd = open("foo", O_CREAT|O_WRONLY|O_TRUNC, S_IRUSR|S_IWUSR);`
		- `O_CREAT`: Creating the file
		- `O_WRONGLY`: If it does not exist, file can be written to
		- `O_TRUNC`: If it already exists, truncate size to 0 bytes and remove any existing content
		- Third parameter specifics permissions, file readable and writable to owner
	- Old way to use `CREAT()` system call, second flag for permission 
	- Returns file descriptor
		- A integer private per process used in UNIX systems to access files
		- Once a file is open, use descriptor to read or write to file
		- Capability  to perform certain operation
		- Pointer to an object analogy
		- Array is kept in the `proc` structure in UNIX to keep track
			- Indexed by file descriptor
			- Keeps tracks of which files are opened per-process basis
	- `strace` tool provides way to see what programs are up to, can trace system calls a program makes

**Reading And Writing Files**
- `prompt> echo hello > foo`
	- The `foo` file contains hello
	- `cat` to use to read
		- `strace` used by Linux (dtruss on Mac, truss on older Unix variants)
			- Traces every system call made by a program while it runs and dump the trace to the screen to see 
		- Open file for reading
			- File descriptor returns 3
				- Each process already has three files opened, stdi, stdo, stderr , represented by 0,1,2
		- After open succeeds, `cat` calls `read()` system call to repeatedly read some bytes from a file
			- `read`
				- First argument: file descriptor telling file system which file to read
				- Second argument: Points to a buffer where the result of the `read()` will be placed
				- Third argument: Size of buffer
		- `write` call to 1
			- Write to stdoutput
		- Tries to read more, but as no more bytes left, returns 0 and program closes
	- Writing is similar
		- File opened for writing
		- `write` system call is called, maybe repeatably for larger files 
		- `close`
		- Use `strace` to trace writes to a file 


**Open File Table**
- Data Structure
- Each process maintains an array of file descriptors
	- Each refer to an entry in system wide open-file table
	- Each entry in table tracks which underlying file the descriptor refers to, current offset, etc

**Reading And Writing Not Sequentially**
- Useful to read and write to a specific offset within a file
	- Use `lseek(int fildes, off_t offset, int whence);`
		- First argument is file descriptor
		- Second argument is offset, positions the file offset to a particular location within the file
		- Third argument determines exactly how seek is performed
		- Does not perform disk seek
		- Simple changes variable in OS memory
	- File for a process opens
	- OS track current offset which determines where the next read or write will begin
		- When read or write of N bytes take place, N is added to current offset, each read or write implicitly updates offset
		- `lseek` explicitly changes offset
		- Offset kept in the file struct



**Shared File Table Entries: `fork()` & `dup()`**
- Entry in open file table is shared in some cases
	- Parent process creates a child process with `fork()`
		- When file table entry is shared, it's reference count is incremented, only when both process close the file, the entry will be removed
		- Occasionally useful
			- Number of processes write to the same output file without any extra coordination
	- `dup()` call
		- Allows process to create a new file descriptor that refers to the same underlying open file as an existing descriptor 
		- Useful when writing to UNIX shell and performing operations like output redirection


**Writing Immediately with `fsync()` **
- `write` tells file system to write data at some point in time
	- File system buffers writes to memory for performance reasons 
	- Only in rare cases data is lost
- Some system require more than buffer
	- DBMS require force writes to disk
	- Use of `fsync(int fd)`
		- Forces all dirty (not written) data to disk 
		- Returns once all writes are complete



`MMAP()` and Persistent Memory
- Memory mapping is alternative way to access persistent data in files
- `MMAP` creates a correspondence between byte offset in a file and virtual address in the calling process
	- Former called backing file and latter its in-memory image



**Renaming Files**
- Use `mv` uses system call `rename(char *old, char*new)`
	- Implemented as an atomic call with respect to system crashed


**Getting Information About Files**
- Expect file system to keep a fair amount of information about each file it is storing
	- Metadata
	- Use `stat` or `fstat` system calls to see metadata
		- Take in pathname or fd and fill in a `stat` struct
		- Each file keeps this type of information in structure call an inode


**Removing Files**
- `unlink` system call to remove a file 
	- Takes name of file to be removed
	- returns 0 upon success 

**Making Directories**
- You can never write to a directory 
- Format of directory is considered file system metadata so file system considers itself responsible for the integrity of directory data
	- Only update a directory indirectly
- `mkdir()` system call for making a directory
	- Creates empty directory
	- Has two entries
		- Itself (dot)
		- Parent (dot-dot)


**Reading Directories**
- `ls`
	- `opendir` : Open a directory
	- `readdir`: Read contents of directory and print it out
	- `closedir`: Close the directory

**Deleting Directories**
- `rmdir()`
	- More dangerous as large amounts of files can be deleted 
	- Directory has to be empty or fail

**Hard Links**
- New way to make entry in the file system using `link()`
	- Takes two argument, old pathname and new one
	- When you a link a new file name to old one, essentially create another way to refer to the same file
	- Simply creates another name in the directory you are creating link to and refers it to the same inode number of the orignal file
		- Not copied in any way but 2 human readable names that refer to the same files
- When creating files:
	- Making a structure (inode) that will track virtually all relevant information about the file including its size, where its blocks are on disk and so forth
	- Linking a human readable name to that file and putting that link in the directory
		- So we need to call `unlink()` to remove a file 
		- Checks the reference count within inode number
			- Tracks how many different files have been linked to particular inode number
		- When unlink called, it removes link between human readable name and inode number


**Symbolic Names**
- Sometimes called a soft link
	- Hard Links are limited
		- Cannot hard link files in other disk partitions (inode number unique in a file system)
- Use same `ln` but `-s` flag
- Symbolic links are a file itself of different type
	- A third type of file (than file and directories)
- Symbolic name formed by holding the pathname of the linked to file as the data of the link file 
	- Longer pathname means longer link file
- Can leave dangling reference 
	- Removing file causes link to point to a pathname that doesn't exist


**Permission Bits and Access Control Lists**
- File system presents a virtual view of a disk transforming it from a bunch of raw blocks into much more user friendly files and directories 
	- Different from process as files are shared between different users and processes
	- Comprehensive set of mechanism for enabling various degrees of sharing 
- Permission Bits
	- Determine for regular file, directory and other entities exactly who can access it and how
		- `rw-r--r--`
		- Owner of the file can do to the file
		- Group can do the file
		- Anyone can do the file
		- Owner of file can change permission using `chmod` (change file mode)
			- Remove ability for anyone except owner: `chmod 600 foo.txt
			- Permission to `rw---------`
- Execute Bit
	- Determines whether a program can be run or not (file)
	- Behaves differently for directories 
		- Enables user, group or everyone to change directories into given directory
- Distributed file system known as AFS include Access Control List
	- More general and power way to represent  who can access a given resource


**Making and Mounting a File System**
- `mkfs` 
	- Give input a device (disk partition) and a file system type (ex ex3) and it writes an empty file system starting with a root directory on the disk partition
	- Needs to be made accessible within the unform file system tree
		- Achieve via `mount` program with `mount()` system
		- Takes an existing directory as a target mount point and pastes a new file system onto the directory tree at that point

