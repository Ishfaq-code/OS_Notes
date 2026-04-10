- Much of the early works regarding bugs focused on deadlock
	- Recently going into more common concurrency bugs
**Crux:** How to handle common concurrency bugs

**Types of Bugs**
- Study by Lu et al
	- Focuses on four major open source applications
		- MySQL
		- Apache
		- Mozilla
		- Open Office
	- Examine the concurrency bugs that have been found and fixed in each of these code bases 
	- 105 total bugs
		- 74 non deadlocks
		- 31 deadlock

**Non-Deadlock Bugs**
- Atomicity violation bugs
	- Desired serializability among multiple memory accesses is violated (definition by Lu et al)
	- 1 thread checks for a non null value while the other sets the value to null
		- Interruptions can happen which violate atomic operation
		- Atomicity Assumption on the non-NULL check
		- Can be solved easily by using a lock 
- Order violation bugs
	- Thread assumes something done by another thread
		- We can't assume that one thread runs before another
	- Desired order between two memory accesses is flipped 
	- Solution can done by using a condition variable
- 97% of these bugs fall between atomicity bugs or order violation bugs

**32.3 Deadlock Bugs**
- Thread holding a lock waiting for another infinitely
	- Presence of a cycle in the graph
- **Crux How to Deal With Deadlock**
- Reason
	- Complex dependencies arise between components
	- Nature of Encapsulation
		- Modularity does not match well with locking 
		- Jula et all point out some interfaces invite you to deadlock
- Conditions
	- Mutual Exclusion: Thread claim exclusive control of resources that they require
	- Hold and wait: Threads hold resources allocated to them while waiting for additional resources 
	- No preemption: Resources cannot be forcibly remove from threads that are holding them
	- Circular wait: Exists a circular chain of threads such that each thread holds one or more resources that are being requested by the next thread in the chain
	- If any of these four conditions are not met, dead cannot occur
		- Explore techniques to prevent deadlock
	- Prevention
		- Circular Wait
			- Write locking code such that you never induce a circular wait 
				- Provide total ordering of lock acquisition
				- Partial ordering can also be useful
					- Memory mapping code in Linux
		- Hold and wait
			- Acquire all locks at once
			- Need to know beforehand which locks need to be acquired
			- Decreases concurrency as all locks being acquired at same time rather than at 'correct times'
		- No Preemption
			- Grab lock only when it's available
				- Introduces live lock where threads continuously checking and trying to grab the lock
					- Using trylock to solve- Thread tries to grab lock, backs out and starts over
						- Complicated - modularity makes restarting harding
						- Program must carefully manage all steps that happened before the failure
						- Need to clean up resources
						- Doesn't truly preempt locks - threads voluntarily give up lock
		- Mutual Exclusion
			- Herlihy with idea of data structures without any locks (lock free and wait free)
				- Powerful hardware instructions to build data structures in a manner that does not require explicit locking
				- Compare and Swap
		- Via Scheduling
			- Deadlock avoidance
				- Requires global knowledge of which locks various threads might grab during their execution, schedule threads in a way that non dead lock occurs
		- Delete and Recover
			- Allow deadlock to occur and take action when it odes
			- Deadlock detection in DBS 
				- Run periodically building a resource graph and checking it for cycles
					- Event of a cycle system restarts


**Don't Always Do It Perfectly (Tom West**
- Soul of a new machine
- "Not everything worth doing is worth doing well"

