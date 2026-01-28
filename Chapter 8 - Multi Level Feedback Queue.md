**Multi-Level Feedback Queue:**
- Described by Corbato in 1962
	- Turing Award
- Twofold problem
	- Optimize turnaround time
	- System feel responsive to user
		- Minimize response time
- Crux: How to schedule without perfect knowledge

**Basic Rules:**
- Number of distinct queues each assigned a different priority level
	- Uses priorities to decide which job should run at a given time --> higher priority chosen to run
	- Priority (A) > Priority (B) := A runs B doesn't (R1)
	- Priority (A) = Priority (B) := A & B run in RR (R2)
- Key lies to how scheduler sets to priority
	- Varies priority by observed behavior
	- Predict future behavior
- Lower priority jobs may never be run
	- Need to change priority over time

**Changing Priority**
- Allotment: Amount of time a job can spend at a given priority level before the scheduler reduces its priority
	- Assume allotment is equal to a single time slice
- When job enter system, placed at highest priority (R3)
- If job uses up allotment while running, it's priority is reduced, moves down one queue (R4a)
- If a job gives up the CPU before allotment is up, it stays at the same priority level (allotment reset) (R4b)
- Single long running job
	- Moves down queue as expected
- Short job while long running
	- Put at highest priority
	- Moved down slowly
- I/O
	- Keep at the same level as it would be fast jobs
- Problems
	- Starvation: Too many interactive jobs will combine to consume all CPU time
		- Long running jobs never receive CPU time
	- Game The Scheduler: Giving you more than the fair share of resources
		- I/O operation before allotment finishes to reset allotment
	- Program may change behavior over time


**Priority Boost**
- Boost priority of all jobs in the system
- After some time period S, move all the jobs in the system to the topmost queue. (R5) 
	- Process are guaranteed not to starve: Sitting at top queue, job will share the CPU with other high priority jobs in round robin fashion and eventually receive service 
	- Interactive job is treated properly once it reaches priority
- No good value for S realistically

**Better Accounting**
- Don't forget allotment after using CPU time (especially in the case of I/O jobs)
- Once a job uses up its time allotment at a given level (regardless of how many times it has given up the CPU), its priority is reduced (i.e., it moves down one queue). (R4)

**Tuning**:
- Various challenges regarding finding the best parameters
- Most use short time slices for high priority queues
- Longer time slices for low priority queue
- As job moves down priority, typically gets more CPU time per turn

Different operating systems implement MLFQ differently:

- **Solaris (TS scheduler)** uses configurable tables that define priorities, time slices, and priority boosts. Defaults include many queues (e.g., ~60), time slices ranging from ~20 ms to a few hundred ms, and periodic priority boosts (about once per second).
    
- **FreeBSD** calculates priorities using formulas based on recent CPU usage, which decays over time to naturally boost priority for waiting jobs.

[[Chapter 9 - Proportional Share]]
