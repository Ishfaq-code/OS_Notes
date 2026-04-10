- **Crux:** How to use semaphores
- **Semaphore**: An object with an integer value that we can manipulate with two routines
	- POSIX: `sem_wait()` & `sem_post()`
- Initial value of the semaphore determines its behavior
	- Must be initialized
	- 2nd value of the initialized semaphore means it is shared between threads in the same process
		- Different values can make it show the synchronization is different between processes
- `sem_wait()`
	- Either return right away
		- Value of semaphore was one or higher when we called `sem_wait()`
	- Suspend execution waiting for a subsequent post
	- Multiple threads may be called
		- All queued waiting to be woken
- `sem_post()`
	- Does not wait for some particular condition
	- Increments the value of the semaphore and then if there is a thread waiting to be woken, wakes them up
- Value of the semaphore when negative is equal to the number of waiting threads

**Binary Semaphores (Locks)**
- Initial value should be 1
- Scenario with Two Threads
	- First thread calls `sem_wait()` and decrements the value of semaphore changing it 0
		- Only wait if the value of the semaphore is not greater or equal to 0
		- Value is 0 so return the calling thread and continue
		- Thread can enter critical section
			- If no other thread acquires the lock while it's in the CS, Semaphore restores to 1 when `sem_post()` is called
			- If it holds the lock and another thread tries to enter
				- `sem_wait()` is called and Semaphore becomes -1
				- `sem_post()` needs to run for the Semaphore to go back to 0 and wake the waiting thread to acquire the lock
- Binary semaphore because lock only has 2 states


**Semaphores for Ordering**
- Order events in a concurrent program
	- One thread waiting for something to happen and another thread making that something happen and then signaling when it happened
	- Parent - Child ordering
		- Parent calls `sem_wait()`
		- Child calls `sem_post()`
	- Semaphore starts at 0
		- Case 1: Parent creates child but the child has not run
			- Parent calls `sem_wait()` before child can call `sem_post()`
			- Parent needs to wait for child to run
			- Only way for this to happen if semaphore is less than 0
		- Case 2: Child runs to completion before parent calls `sem_wait()`
			- Child calls `sem_post()` first so semaphore is incremented to 1
			- Parent set value to 0 and is able to return from `sem_wait()` without waiting

**Setting the Value of a Sempahore**
- Perry Kivlowitz
	- Consider the number of resources you are willing to give away immedeatly

Producer/Consumer
- First Attempt
	- Use of Two Semaphores
	- Empty and full
	- Threads indicate when a buffer entry has been emptied or filled respectively
		- Producer waits for buffer to become empty in order to put data into it
		- Consumer waits for buffer to become filled
	- Only 1 buffer example
		- Consumer runs first
			- `sem_wait(full)`
				- `full` was initialized to 0 so it will decrement the value to -1
				- Block the consumer and wait for another thread to call `sem_post()` on full
			- `sem_post(empty)`
				- Empty initialized to 1 at the beginning (`MAX` technically depending on how big the buffer is)
					- Immediately can do operation
					- Decrement `empty`
					- Call `sem_post(full)` when buffer is full to signal and increment the  `full` semaphore
						- Two things could happen
							- Producer keeps running but will hit `empty` value being 0 and block
							- Consumer interrupts producer and runs and consume the full buffer
		- More than 1 buffer causes an issue
			- Race conditions, data may be over written
- Mutual Exclusion solution
	- Locks using binary semaphores
		- Deadlock
			- 2 threads 1 P, 1 C
			- C runs first
				- Acquires lock and blocks as the buffer is empty
				- Holds lock
			- P runs
				- Tries to first run `sem_wait()`
				- Lock already held by C
				- Producer now waiting too
- Working solution
	- Locking and unlocking after checking full and empty on the semaphores
		- Avoids lock being held by different things



**Reader Writer Locks**
- Different data structure access require different kind of locking
	- Inserts change the state of the list while lookups only read
	- As long as we can ensure no inserts are going on, we can do many concurrent lookups
- Reader-Writer lock
	- `writelock` to lock critical section for writing , semaphore only single writer can acquire the lock and enter the critical section
	- Reader first acquires lock and increments `readers` variable to track how many readers are currently inside the data structure
		- Acquires both reader and writer lock when the first reader acquires lock by calling `sem_wait()` on the `writelock`
		- Many readers are allowed to acquire the read lock but to write, all readers must be finished
			- Last one to exit critical section calls `sem_post` on write lock and enables writer to acquire lock
		- Not fair, readers can starve writers
		- Add overhead, do not speed up performance

**Hill's Law: Simple and Dumb Can Be Better**
- Simple and dumb approach is the best one
	- Simple spin lock works best


**Dining Philosophers**
- Solved by Dijkstra
- Five philosophers sitting around a table
- Each pair of philosophers there is a single fork
- Philosophers can think and don't need fork and times where they eat
- In order to eat philosopher needs fork in both hands
- Downey's solution with left(p) for left fork and right(p) for right fork
- Broken solution:
	- Initialize each semaphore to 1
	- Each philosopher knows its own number 
	- Grab locks on each fork
	- When done eating release the fork
	- Deadlock
		- If each P grabs fork on their left before one grabs it on their right, they are all holdig left forever
- Solution: Breaking the dependency
	- Philosopher 4 grabs fork in a different order than others 
	- No case of deadlock

**Thread Throttling**
- Limit number of threads doing something, use a semaphore to control it
