**Crux: ** How to wait for a condition
- Thread should wait for some condition before proceeding
- Spinning until condition becomes true

**Definition and Routines**
- Condition variable is an explicit queue that threads can put themselves on when some state of execution is not as desired
	- Some other thread changing state can then wake one or more of those waiting threads and all them to continue
	- Signaling the condition 
	- Dijkstra's use of 'private semaphores'
	- Condition variable named by Hoare in his work on monitors
	- Declare condition variable by `pthread_cond_t c;`, declares `c` as a condition variable
	- Two behaviors
		- Wait - Put itself to sleep
			- Takes mutex as parameter and assumes the mutex is locked
			- Responsible to release the lock and put the thread to sleep
			- After thread wakes up, it must re-acquire the lock before returning to the caller
				- Prevent race conditions
		- Signal - executed when a thread has changed something in the program and needs to wake a sleeping thread waiting on a condition
			- Always hold lock while signaling
	- `done` variable is important as it is what signals the parent thread to a wake up from it's wait
	- Race condition happens without a lock
		- Parent is interrupted by child before running wait
		- Parent never goes to sleep so no signal function is called


**Producer/Consumer Problem**
- Also known as the bounded buffer problem first proposed by Dijkstra
	- Led them to create the Semaphore
- Producers generate data items and place them in a buffer
- Consumers grab items from the buffer and consume them in some way
	- HTTP Work queue
- Bounded buffer is used when you pip the output of one program into another
	- `grep foo file.txt | wc -1`
	- Concurrently writes lines from `file.txt` with the string `foo` in the standard output 
		- UNIX shell redirects the output to what is called a UNIX pipe
		- Other end of this pipe is connected to the standard input of the process `wc` which simply counts the number of lines in the input stream and prints out the results
			- Grep process in the producer
			- wc process in consumer
- Bounded buffer is a shared resource, so synchronized access is required
	- Single integer example
		- `put` routine assumes the buffer is empty and simply puts a value into the shared buffer and marks it full by setting count to 1
		- `get` routine does the opposite, setting the buffer to empty, count to 0 and returning the value
		- Need to write routines that know when it is OK to access the buffer to either put data into it or get data out of it
			- Condition: Only put data into buffer when count is 0
				- Only get data from buffer when count is
				- Will be done by producer and consumer threads
	- Race condition
- Broken Solution
	-  Simple lock doesn't work
	- Single condition variable and associated lock mutex
		- Signaling logic
			- Producer waits for buffer to be empty
			- Consumer waits for buffer to be full
		- Single producer and consumer, the logic works
		- Breaks if 1+ threads for either producer or consumer
			- Consumer threads might get a signal to get something from the buffer when another consumer thread already made it empty
				- After Tc1 woke and but before it ran, the state of the buffer changed (Tc2 took the value)
				- **Mesa Semantics:** Interpretation of what a signal means
				- **Hoare Semantics:** Stronger guarantee that the woken thread will run immediately upon being woken
- Better Solution Broken Solution
	- Change if condition to while
		- Rechecking state of shared variable
		- If buffer is empty, consumer goes back to sleep
		- Always use while for mesa semantics
	- Other problem
		- Consumer doesn't know which thread to wake because of only 1 signal variable 
			- 1 producer thread and 1 consumer thread sleeping
			- Consumer might wake up on an empty buffer (as previous consumer just consumed the value)
			- All three threads sleep
- Single Buffer Producer/Consumer Solution:
	- Use of two condition variables
		- Signal which state should wake up when the state of the system changes
		- Empty and Fill signals
		- To add more concurrency, simply add more buffer slots


**Covering Conditions**
- Lampson and Redell implemented Mesa Semantics
- Wake all threads that can be woken instead of randomly using signal (`broadcast`)
	- Can be a negative performance impactive
	- All threads wake up and check condition before going back to sleep
	- Covering conditions as it covers all conditions 


[[Chapter 31 - Semaphores]]