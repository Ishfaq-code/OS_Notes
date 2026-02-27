When memory is free, memory pressure forces the OS to start paging out pages to make room for actively-used pages
- Deciding which page to evict is encapsulated within the replacement policy of the OS
- **Crux:** How to Decide Which Page to Evict?

**Cache Management**:
- Main memory holds some subset of all the pages in the system
	- Can be viewed as the Cache for virtual memory pages
	- Goal for this cache is to minimize the number of cache misses to minimize the number of time we have to fetch a page from disk
		- Maximizing cache hits
	- Average Memory Access Time = Tm + (Pmiss * Td )
		- Tm cost of accessing memory
		- Td cost of accesing disk
		- Pmiss probability of not finding the data in the cache
		- Pay the cost of accessing the data in memory
			- When you miss you pay additional


**Optimal Replacement Policy**
- Best possible replacement policy
	- Developed by Belady `B66` `MIN`
	- Fewest number of misses overall
	- Replace the page that will be accessed furthest in the future
- Cold Start Miss: Missing at start (nothing in cache)
- Capacity Miss: Occurs because the cache ran out of space and had to evict an item to bring a new item into cache
- Conflict Miss: Limits on where it can be placed in the hardware
- Can't build for general operating systems


**Simple Policy: FIFO**
- Pages placed in a queue when they enter the system 
	- Replacement evicts the page on the tail of the queue
- Simple to implement
- Belady's Anomaly: Cache hit rate decreases from larger cache size
	- Happens with FIFO
	- Not with LRU as it is stact proportional
		- Cache of size N+1 includes cache of size N

Random Policy:
- Randomly picks a place to replace under memory pressure
- Better than FIFO

**LRU:**
- FIFO or Random may kick out an important page
- Use frequency of page use
	- If page is used many times shouldn't be replaced
	- More recently a page has been accessed, the more likely it will be accessed again
- Family of principle of locality
	- Use history to figure out what pages are important

**Workload Examples**:
- Evaluates replacement policies under simplified workload
- Without locality policy choice barely matter
- 80-20 Workload
	- 100 Unique Pages
	- 80% of accesses go to 20% of pages (hot pages)
	- 20% of accesses go to remaining 80% (cold pages)
	- Optimal Policy Best
	- LRU better than FIFO and Random