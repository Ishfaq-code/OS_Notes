Each job obtains a certain percentage of CPU

**Lottery Scheduling:**
- Hold a lottery to see which process should get to run next
- Processes that should run more often should be given more chances to win

**Crux:** How to share the CPU proportionally? 

**Tickets:** Share of resources a process should receive
- Percent of tickets that a process has represents its share of the system resource in question
- Lotter Scheduling achieves this probabilistically - not deterministically
	- Hold lottery ever time slice

Ticket Process:
- Scheduler knows how many total tickets there are
- Picks winning ticket from 0 to n-1 tickets
- Uses randomness 

## 1. Ticket Currency

### Definition
- **Ticket currency** allows users to distribute tickets among their own jobs using a _private currency_.
- The operating system automatically converts these tickets into a **global currency** so scheduling remains fair across users.
### Why It’s Useful
- Users can manage priorities **within their own jobs** without affecting other users.
- The system still enforces fairness globally.

## 2. Ticket Transfer
### Definition

- **Ticket transfer** allows a process to temporarily give its tickets to another process.
### Primary Use Case
- **Client/server systems**
    - Client requests work from a server
    - Client transfers tickets to server
    - Server runs faster while serving the client
    - Tickets are returned when work is complet
### Benefit
- Improves performance for client requests
- Prevents servers from becoming CPU bottlenecks

## 3. Ticket Inflation
### Definition
- **Ticket inflation** allows a process to temporarily increase or decrease the number of tickets it owns.
### When It Makes Sense
- Only in **trusted environments**
- Processes cooperate rather than compete 

### Why It’s Dangerous in Untrusted Systems
- A greedy process could inflate tickets excessively
- Would monopolize CPU time

### Legitimate Use
- A process that needs more CPU time can:
    - Increase its ticket count
    - Signal urgency to the scheduler
    - Avoid communicating explicitly with other processes