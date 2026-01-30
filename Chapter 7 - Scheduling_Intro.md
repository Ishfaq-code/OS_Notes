#Quiz_2 #Virtualization #Test_1 #Scheduling

Origins predate computer systems - taken from field of operations management

**The Crux:** How to develop a scheduling policy
- Key Assumptions
- Important Metrics
- Basic Approaches

**Workload:** Number of simplifying assumptions about the processes running in the system, collectively called the workload
- The more you know about the workload, the more fine tuned your policy will be
- Critical to building the policy
- Unrealistic standards to start off with
- Relax them as we go to develop a **fully operational scheduling discipline**


Process are sometimes called **jobs**
We make the following assumptions about the **jobs**:
- Each job runs for the same amount of time
- All jobs arrive at the same time
- Once started, each job runs to completion
- All jobs only use the CPU (i.e., they perform no I/O)
- The run-time of each job is known (very unrealistic)

**Scheduling Metric:** Measure the scheduling
- Single metric for now - **Turnaround Time** = Completion Time - Arrival time (arrived at system)
	- From previous assumption, all jobs arrive at the same time so Arrival time =0, thus Turnaround Time = Completion Time
	- Performance metric
- Performance and Farness at odds in a job 

**FIFO** - Non Preemptive
Most basic algorithm for now is FIFO
- Clearly simple and easy to implement 
- Good for when assumed all jobs run at the same time (Turnaround Time is the time all jobs run for)
- Poor when jobs run at different times - especially if one of the job takes long time
	- **Convoy Effect:** Number of relatively-short potential consumers of a resource get queued behind a heavyweight resource consumer
		- Large cart in a grocery store line

Preemptive vs Non-preemptive Schedulers
- Old days of batch computing, number of non-preemptive schedulers were developed
	- Run each job to competition before thinking about running new jobs
- All modern schedulers are preemptive
	- Quite willing to stop one process from running in order to run another
	- Schedulers can perform context switching

**SJF - Shortest Job First** - Non Preemptive
Perceived turnaround time per customer/job matter
Solves the Convoy Effect issue with FIFO
- Stole from operations research
- Runs the shortest job first
- Best for our assumption that all jobs arrive at the same time 
- Poor for when jobs arrive at different times
	- Forced to wait until previous job has completed
	- If a job starts it has to finish
		- If A arrives at 0 and runs 100 seconds and B and C both arrive at 10 running 10 seconds, even thought B and C are shorter, A has to run first before B and C can run run

**Shortest Time to Completion First (STCF)** - Preemptive Shortest Job First
Relax assumption 3 - jobs run to completion
- Preempt jobs based on whichever jobs have least time remaining. Least remaining time is continued to be processed

**Response Time**: New metric -  Time of Response = Time of First Run/Schedule - Time of Arrival
- Need due to time shared machine as interactive performance is needed
- STCF not good for this metric
- Waiting to get response 


**Amortization**: Incurring cost left often, total cost is reduced
- Commonly used in systems when there is a fixed cost to some operation

**Round Robin**
- Runs a job for a time slice (scheduling quantum) and then switches to the next job to run in queue
- Repeat this until jobs are finished
- Sometimes called time slicing
- Time Slices must be a multiple of timer-interrupt period
- Shorter time slices enable better response time 
	- Too short will prove to be problematic as cost of context switching will dominate performance
- One of the worst policies for turn around time 
- Fair policy
	- Fair policies bad for turnaround time

- Context switching is **not just** about the OS saving and restoring CPU registers.
- While a program runs, it builds up **useful information inside the CPU**, including:
    - **CPU caches** → recently used data and instructions
    - **TLB (Translation Lookaside Buffer)** → recent memory address translations
    - **Branch predictors** → guesses about which code path will run next
    - Other **on-chip hardware state**
- When the OS switches to a different process:
    - Much of this cached state is **lost or becomes useless**
    - The CPU must **reload new data and predictions** for the new process
- This causes:
    - **Cache misses**
    - **TLB misses**
    - **Poor branch prediction at first**
- As a result:
    - The new process runs **slower for a short time**
    - This slowdown is called **context switch overhead**
- Therefore:
    - Context switches can have a **noticeable performance cost**, even if register saving is fast

Cost of Context-Switching does not solely arise from OS actions but also from other programs running and building up cache before switching to a different program

**Incorporating I/O - No to assumption 4**
- CPU blocked during I/O - scheduler needs to make decision 
	- During I/O, scheduler should probably move onto another
- Allow for overlap by completing job with CPU and I/O running as sub jobs and then completing next job 

[[Chapter 8 - Multi Level Feedback Queue]]
