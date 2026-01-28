#Quiz_2 #Virtualization #Test_1 #Scheduling

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
- Good for when assumed all jobs run at the same time (Turnaround Time is the time all jobs run for)
- Poor when jobs run at different times - especially if one of the job takes long time
	- **Convoy Effect:** Number of relatively-short potential consumers of a resource get queued behind a heavyweight resource consumer

**SJF - Shortest Job First** - Non Preemptive
Solves the Convoy Effect issue with FIFO
- Runs the shortest job first
- Best for our assumption that all jobs arrive at the same time 
- Poor for when jobs arrive at different times
	- Forced to wait until previous job has completed

**Shortest Time to Completion First (STCF)** - Preemptive
Relax assumption 3 - jobs run to completion
- Preempt jobs based on whichever jobs have least time remaining. Least remaining time is continued to be processed

**Response Time**: New metric -  Time of Response = Time of First Run/Schedule - Time of Arrival
- STCF not good for this metric
- Waiting to get response 

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



Cost of Context-Switching does not solely arise from OS actions but also from other programs running and building up cache before switching to a different program

**Incorporating I/O - No to assumption 4**
- CPU blocked during I/O - scheduler needs to make decision 
- Allow for overlap by completing job with CPU and I/O running as sub jobs and then completing next job 

[[Chapter 8 - Multi Level Feedback Queue]]
