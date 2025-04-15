https://drive.google.com/drive/folders/143DtdyFFToeSkwvxlVE_16QQTQV6a1Yg

#Lecture-18
#Parallelism
## Parallelism
- Running multiple things at same time
	- **In Functional Programming:**
		- Everything is pure functions
		- Call expressions can be eval'd in lazy/any order and in parallel
		- #D/Referential-Transparency: An implementation in which call expressions can be replaced with their evaluation value (and vice versa) without changing the program and/or its structure
		- Functional programming has this, no side effects makes things easy
		- Implementable using some #MapReduce
	- Mutable shared state makes non-purely-functional programming implementations of parallelism a bit more difficult
#MapReduce/Parallelism-Evaluation-Model
- This is a method purely to implement parallelism in **functional programming** where there is no mutability weirdness
- Two phases
- In order to run things in parallel:
	- **Map Phase:**
		- Take in iterator over inputs ie text lines
		- Spit out >= 0 key-value pairs per input detected
		- Do this mapping multiple times for multiple different input values
		- With multiple of these dicts, group them together, form groups of key-value pairs that have the same key
	- **Reduce Phase:**
		- Mush all of the elements in a single group together, ie summing the values for multiples of a given key

In languages with mutability, need to do something different

#Parallelism/Python
- Two main mechanisms to achieve parallelism in python
	- #Parallelism/Python/Threads:
		- Simply run programs in tandem with no safety rails
		- Same interpreter for two threads, data shared between them
		- One thread run at a time, but interpreter rapidly and arbitrarily switches between them
		- Out-of-interpreter/external operations may run in the background unaffected
	- #Parallelism/Python/Processes:
		- Run programs in separate interpreters
		- Data generally not shared between interpreters
			- Shared state must be explicitly communicated between interpreters
		- Programs actually run in parallel at hardware level, as they are being run on separate interpreter
#Parallelism/Python/Threads Ex:
```
from threading import Thread, current_thread

def thread_hello():
	other = Thread(target=thread_say_hello, args=())
	other.start()
	thread_say_hello()

def thread_say_hello():
	print('hello from', current_thread().name)

'''
thread_say_hello = function that new thread should run
put any args in the args=()
.start() to start the thread
print output will be unordered because parallelism things
'''
```
#Parallelism/Python/Processes Ex:
```
from multiprocessing import Process, current_process

def process_hello():
	other = Process(target=process_say_hello, args=())
	other.start()
	process_say_hello()

def process_say_hello():
	print('hello from', current_process().name)

'''
Essentially identical to thread, but you're starting a process instead of a thread with .start()
'''
```

These options however can cause dataraces if they are not dealt with

#Parallelism/Atomic-Operation
## Atomic Operations
- #D/Atomic-Operation: Any operation that takes effect instantaneously
- Most operations are not atomic, can be interrupted by another thread
- Things usually made from several atomic operations
- #D/Race-Conditions: Any situation in which multiple threads concurrently access the same data, and at least one thread mutates the data in some way
- #D/Synchronization: The management of shared data in parallelism when mutation of the data is possible
	- Can't under-synchronize without underprotecting
	- Can't over-synchronize without limiting parallelism
	- #D/Deadlock: A situation in which different threads indefinitely wait for each other to finish, aka a circular dependency
- Some data structures are atomic by nature, ie queues
- Other programs can be synchronized by using locks
	- #D/Locks: A datatype meant to ensure that at any given time, only **one** thread/process has access to a given piece of data
		- Once a thread **acquires** a lock, they alone have access to the data, and they can modify it as they please
		- Upon finishing computation on a piece of data, the thread **releases** the lock, so that other programs may access and acquire the lock for their own computations
		- Python has a Lock class to implement this
		- Essentially a fancy content manager

#E/Parallelism/Web-Crawler
- **Web Crawler Example**
	- Program to systematically browse the internet
		- ie. to validate website link, recursively check all links hosted by same site, many such uses
	- **Parallel Crawler:**
		- Set up a queue of URLs that still need to be processed
		- Make a shared set of URLs that have already been seen so that the web crawler avoids them
		- Just make sure the queue is synchronized

```
# This would look something like this

def put_url(url):
	"""Queue the given URL."""
	queue.put(url)

def get_url():
	"""Retrieve a URL."""
	return queue.get()

def already_seen(url):
	"""Check if a URL has already been seen."""
	with seen_lock:
		if url in seen:
			return True
		seen.add(url)
		return False
```

#E/Parallelism/Particle-Simulation
- **Particle Simulation Example**
	- Bunch of particles interacting
	- One big set of particle
	- Subsets of big set of particles divided among threads or processes
		- Forces computed from particle positions
		- **shared data = particle positions**
		- Simulation discretized into timestamps
	- **Computation Steps:**
		1. Read particle positions (read shared data)
		2. Update particle accelerations (access non-shared data)
		3. Update particle velocities (access non-shared data)
		4. Update positions of own particles (write shared data)
		- Concurrent reads are ok, and writes are always to different locations (timestamps), so dataraces avoided if timestamps implemented properly
		- How do we implement timestamps though?

#Parallelism/Barriers
#D/Barriers
**Solution 1 to Dataraces: Barriers**
- Break program into **non-overlapping phases**
	- Ensure non-overlapping phases by putting **barriers** between phases
	- Actual synchronization mechanism (from threading import Barrier) to divide phases

#Parallelism/Message-Passing
#D/Message-Passing
**Solution 2 to Dataraces: Message Passing**
- Explicitly pass state from the thread/process that owns it/has the lock for it to a thread that needs to use that state
	- ie in each timestamp as in particle example, each particle makes copy of own particles, send this to others, receive what others send

#Parallelism/Summary
## Summary
- Parallelism is good for performance
- Parallelism easy to implement w/ functional programming
- Mutability of data adds an extra layer of complication to implementation
	- Not all operations are an #Parallelism/Atomic-Operation, so need to make sure there is some level of #D/Synchronization or else there will be #D/Race-Conditions and hard-to-catch bugs
		- Essentially, you must control the access patterns/restrictions on access by multiple threads to ensure computation works as intended
		- Threads cannot be allowed to just modify and read state as they please in most conditions, unless we truly don't care
		- Fix using things like #D/Message-Passing, #D/Barriers, or #D/Locks on specific pieces of data
			- With #D/Locks, make sure not to cause #D/Deadlock conditions by allowing for conditions where things are indefinitely waiting for other things to finish

Keaton Hedrick
Apr 14 2025