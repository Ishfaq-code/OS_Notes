A lock is a variable that holds the state of the lock at any given time
- Available, unlocked or free
- Acquired or locked
- One thread holds the lock and is in CS
- Calling `lock()` tries to acquire the lock
	- If no other thread holds it, successfully acquires and enters the critical section
		- Thread called the owner of the lock
		- If another thread calls the lock on same lock variable, it will not return while the lock is held by another thread
		- Other threads are prevented from entering the critical section while the first thread holds it
- Calling `unlock()`, the lock is available and free again
	- If no other threads are waiting for the lock it is kept free
	- If there are other waiting threads, one of them will eventually notice this change and acquire the lock
- Locks help give control of scheduling back to programmer as they can decide on the fact that only 1 thread can be in the CS

**Pthread Locks**
- Mutex used by POSXI library for a lock
	- Mutual exclusion between threads 
	- If 1 thread in the CS it prevents other from entering 
- Coarse-grained locking: 1 big lock used any time any CS is accessed
- Fine Grained: Different locks for different critical section to improve concurrency

**Building A Lock**
Crux: How to build a lock
- Hardware privatives to built

**Evaluating Locks**
- Mutual Exclusion
	- Prevent multiple threads from entering the critical section
- Fairness
	- Each thread contending the lock gets a fair shot at acquiring it once it is free
- Performance
	- Time overhead added by using locks
		- Single threaded CPU with no contention, how does overhead apply
		- Multiple threads where there are performance concerns

**Controlling Interrupts**
- One of the earliest solutions for mutual exclusion
	- Disabling interrupts for critical sections
	- Invented for single processor systems
	- Uses some kind of special hardware instruction
- Ensure the code inside the critical section will not be interrupted
- Positive
	- Simple
		- Without interruption a thread can be sure that the code it executes will execute and no other thread will interfere with it
- Negative
	- Threads need to perform privileged instruction
		- Trusting an arbitrary program 
		- Greedy program could call `lock()` at the beginning of its execution and monopolize the processor
		- Malicious program could call `lock()` and go into an endless loop
			- OS never regains control of the system, need to restart
	- Requires too much trust on the programs
	- Approach does not work for multiprocessor
		- Threads are are able to run on other CPUs despite interrupts being disabled
	- Turning off interrupts for extended periods of time can lead to interrupts becoming lost
- Still some use
	- OS itself uses interrupt masking to guarantee atomicity when accessing its own data structures
		- Makes sense deep inside the OS


**Failed Attempt: Just using Loads/Stores**
- Single flag variable
	- Variable flag to indicate whether some thread has possession of a lock
	- Thread enters the critical section and call lock
		- Tests to see if flag is equal to not
			- If not, set the flag to 1 to indicate current thread holds the flag
	- After finishing work in the critical section, thread calls `unlock()` and sets flag to 0 to indicate it is not being held
	- If another thread call `lock()` when the first thread is in the CS, it will spin-wait in the while loop for the thread to call `unlock` and clear the flag
	- Correctness Problem:
		- With bad timing and interleaving, both threads can set the flag to 1 at the same time , failing mutual exclusion
	- Performance Problem
		- Endlessly waits using spin-waiting
			- Wastes time waiting for another thread to release a lock
			- Waste is exceptionally high on a uniprocessor where the thread that the waiter is waiting for cannot even run


**Working Spin Locks with Test and Set**
- Earliest multiprocessor systems such as Burroughs B5000 in the early1960s had such support
- Test and Set
	- Test old value value while simultaneously setting the memory location to a new value
		- Thread call `lock` when no other thread holds it, so flag is 0
		- Test-and-Set is called by the thread which returns the old value of the flag (0)
			- Testing it and finding it out to be 0 will make sure it is not spinning endlessly in the while loop
			- Also will atomically set the value of the flag to 1 to indicate it has acquired the lock
			- Unlocking will simply set the flag back to 0
		- If another thread is already in the critical section
			- Calling `TestAndSet` will return the value of the flag to be 1 and atomically set it to 1
			- The thread will then spin until the old value is returned as 0
		- Making both `get` and `set` atomic, only 1 thread can acquire the lock
		- Spin Lock: Simply spins using CPU cycles until the lock becomes avaliable
			- Needs preemptive scheduling (interrupt CPU by a timer to try run a different thread)

**Evaluating Spin Locks**
- Correctness: Yes due to mutual exclusion
- Fairness: No fairness guarantee, threads may spin forever due to contention
- Performance: 
	- Single CPU performance overhead can be heavy
		- Scheduler might get to a locked thread after going through all threads so thread needs to wait for N-1 thread to be tested
	- Works reasonably well for multiple processors 


**Compare And Swap**
- Test whether the value at the address specified by `ptr` is equal to `expected`
	- If so update `ptr` to new value
	- If not, do nothing
	- Return original value at the memory location
- Identical to  `TestAndSet` if we just build a spin lock


**Load-Linked and Store Conditional**
- MIPS architecture
	- Used to build locks and other concurrent structures
- Load Linked operates like a typical load instruction
	- Fetch a value from memory and place it in a register
- Store conditional only succeeds if no intervening store to address has taken place
	- In case of success, store-conventional returns 1 and updates ptr to value
	- If it fails it returns 0 and value is not updated
- `lock()`
	- Thread spins waiting for flag to be set to 0
	- Once 0, thread tries to acquire the lock via store-conditional
		- On success, atomically change flag value to 1 and enter CS
		- Failure can occur when before attempting to do the store conditional, another thread interrupts and enters the critical section

**Fetch and Add**
- Hardware privative that atomically increments a value while returning the old value at a particular address
- Ticket Lock by Mellor Crummy & Scott
	- Ticket and turn variable in combination to build a lock
	- When a thread wishes to acquire a lock, it does an atomic fetch and add on the ticket value
	- That value is considered the thread's turn
	- Globally shared `lock-->turn` is then used to determine which thread's turn it is to enter the critical section
	- Unlock is accomplished by incrementing turn to be the same value as the next waiting thread's turn
	- Guaranteed to enter as we are using a queue through the ticket system

**Too Much Spinning**
- Single processor bad
	- Spends entire time spinning and not doing anything (checks value while spinning)
	- N-1 time slices wasted for N threads
- **Crux: How to Avoid Spinning**

**Simple Approach: Yielding**
- Giving up the CPU to another thread
	- Thread can be in one of three states: running, ready, blocked
	- Yield is simply a system call that moves the caller from the running state to the ready state and thus promotes another thread to running 
	- Yielding a thread essentially deschedules itself
	- Cost of context switching while yielding is costly
		- Many threads yield again and again when they find that the CS is being used
	- Does not address starvation
		- Thread may get caught in an endless yield loop while other thread repeatably use the CS

**Reasons to avoid Spinning: Priority Inversion**
- T2 has higher priority than T1 but is blocked
	- T1 is CS holding lock
	- T2 gets rescheduled and is unblocked
	- T1 is descheduled 
	- T1 holds lock so T2 is spinning forever
- Changes priority around
- More generally, a higher-priority thread waiting for a lower-priority thread can temporarily boost the lower thread’s priority, thus enabling it to run and overcoming the inversion, a technique known as priority inheritance.
- Could ensure all threads have same priority


**Queues: Sleeping Instead of Spinning**
- Scheduler can make bad choice which can lead to starvation or spin locking
	- Potential waste and no prevention of starvation
- Use support provided by Solaris
	- `park()` put thread to sleep
	- `unpark(threadID)` to wake a particular thread designated by `threadID`
		- Can be used in tandem to build a lock that puts a caller to sleep if it tries to acquire a held lock and wakes it when the lock is free
		- Threads spin when other threads are releasing or acquiring the lock
			- Does not completely get rid of spinning
			- Time spent spinning is limited
		- When a thread can't acquire the lock, it is added to the queue
		- Does not set flag to 0 when another thread is woken
			- When a thread is woken, it is returning from park
			- Does not hold the guard at that point
			- Cannot try to set flag to 1
			- Just passes lock to next thread
		- Wakeup/Waiting Race: Thread about to sleep and and a thread about to wake up at the same time could course race condition
			- Solaris solves by system call `setpark()`
				- Thread can indicate it is about to park
				- If thread happens to be interrupted and another thread calls unpark before park is actually called, subsequent park returns immediately instead of sleeping


**Different OS, Different Support**
- Linex provides futex, similar to Solaris but more kernel support, similar to Solaris interface
	- Each futex has associated with a specific physical memory location as well as a pre-futex queue
	- `futex_wait` puts calling thread to sleep assuming value at address is equal to expected
		- If not returns immedeatly
	- `futex_wake(address)` wake one thread that is waiting in the queue

**Two Phase Locks**
- Linux approach has the flavor of an old approach that has been used on and off for years going at least as far back to Dham locks in the early 1960s, is the two phase locks
	- Realizes spinning can be useful especially if the lock is about to be released
	- First phase lock spins for a while hoping it can acquire the lock
	- If lock is not acquired during the first spin phase, a second phase is entered where caller is put to sleep and only woken up when the lock becomes free later