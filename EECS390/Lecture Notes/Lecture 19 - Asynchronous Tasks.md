https://drive.google.com/drive/folders/143DtdyFFToeSkwvxlVE_16QQTQV6a1Yg

**Up to Slide 16 in Lecture**

#Lecture-19
#Asynchronous-Tasks
## Asynchronous Tasks
- #Lecture-18 was about tackling #Parallelism from a resource perspective
	- "Divide program over set of workers"
- Now: algorithm-centric view
	- #D/Asynchronous-Task: Method of implementing mutable-friendly parallelism in which programs are decomposed into an algorithm-centric view, each operation has a specified task to complete
	- Might schedule asynchronous tasks to be assigned to several worker threads

#Asynchronous-Tasks/CPP
**C++ Asynchronous-Tasks:**
```
void foo(int x, int y) {
	cout << (x + y) << endl;
}

int main() {
	async(foo, 3, 4);
	async(foo, 5, 6);
	// foo is function to call
	// specified args come afterwards
	// launches an asynchronous task to compute the function foo with
	// the given input

```
- **Futures**
	- ie "future\<void> f1  = async(foo, 3, 4)"
	- Abstraction for computation result
	- use **.get()** to wait for task to complete and then obtain its return value

#E/Asynchronous-Tasks/Asynchronous-Fibonacci
**Asynchronous Fibonacci Example**
```
long logn async_fib(int n) {
	if (n <= 1)
		return n;
		
	// launch separate task for one recursive call
	future<long long> res1 = async(async_fib, n - 1);
	
	// use existing task for another recursive call
	long long res2 = async_fib(n - 2);
	
	// wait for AND obtain result from other task
	return res2 + res1.get();
}
```
- Have to be careful to make sure actually parallelizing program in first place
	- ie:
#E/Asynchronous-Tasks/Not-Parallel
```
long long async_fib(int n) {
	if (n <= 1)
		return n;
	return async(async_fib, n - 1).get() + async_fib(n - 2);
}

// not actually asynchronous, as the async_fib(n - 2) has to wait for
// the get() to return a value before it is allowed to run
```

#Asynchronous-Tasks/Limiting-Task-Overhead
## Limiting Task Overhead
- Implementations that use a thread pool will always have to deal with the additional overhead of scheduling/dispatching tasks to a thread
- More granularity causes more overhead
- Eventually, overhead can bog down the computation enough to take longer than the actual computation itself
- Must experiment to find the correct level of granularity
	- Could also do amortized analysis

#E/Asynchronous-Tasks/Limited-Fibonacci
**Limited Fibonacci Example**
- Solution to too much overhead: place a threshold and limit on the number of tasks allowed to exist at a given time
```
long long async_fib(int n, int tasks, int max_tasks) {
	if (n <= 1)
		return n;
	if (tasks < max_tasks) {
		future<long long> res1 =
			async(async_fib, n - 1, 2 * tasks, max_tasks);
		long long res2 = async_fib(n - 2, 2 * tasks, max_tasks);
		
		return res2 + res1.get();
	} else {
		return fib(n - 1) + fib(n - 2);
	}
}

// tasks == current number of tasks
// max_tasks == limit on number of tasks
// launches new tasks only if the limit is not currently exceeded
// if launched, doubles the number of tasks "tasks" for the next level\
// if limit reached, do the computation normally, sequentially.
//    ^this portion is the else statement you see
```
- Switching between creating new/more granular tasks and doing the task normally without threads can essentially be a really efficient way to cut back on parallelism overhead
	- Not surefire, but generally works

#E/Asynchronous-Tasks/Asynchronous-Quicksort
**Asynchronous Quicksort Example**
- Can also be a good idea at smaller input sizes to switch to a different algorithm
- For example with Quicksort:
```
void async_quicksort(int *A, size_t size, int thread_count, int max_tasks) {
	if (size <= CUTOFF) {
		std::sort(A, A + size);
		return;
	}
	int pivot = A[size/2];
	std::swap(A[0], A[size/2]);
	size_t pivot_index = partition(A, size);
	if (thread_count < max_tasks) {
		future<void> rec1 = 
					async(async_quicksort, A, pivot_index,
						2*thread_count, max_tasks);
		async_quicksort(A+pivot_index+1, size-pivot_index-1,
						2*thread_count, max_tasks);
		rec1.wait();
	} else {
		quicksort(A, pivot_index);
		quicksort(A+pivot_index+1, size-pivot_index-1)
	}
}
```

**Launch Policy**
- #D/Launch-Policy: Specification of whether a task should execute in a given thread or in a new thread
- Launching new tasks don't make them run in NEW threads by default
	- Might be better to do lazy computation and make computation only occur when the value is needed (ie. using .get())
	- Can specify with std::launch::policy_here to specify lazy vs open another thread and thug it out
	- For example:

#E/Asynchronous-Tasks/Computing-Pi
**Computing π Example**
```
double compute_pi(size_t samples) {
	default_random_engine generator;
	uniform_real_distribution<> dist(-1.0, 1.0);
	size_t count = 0;
	for (size_t i = 0; i < samples; i++) {
		double x = dist(generator);
		double y = dist(generator);
		if (sqrt(x * x + y * y) <= 1.0)
			count++;
	}
	return 4 * count / (double) samples;
}
```

#E/Asynchronous-Tasks/Parallel-Computing-Pi
**Computing π with Parallel Computation Example**
- Accomplishing the same thing using parallel computation over a fixed set of threads
```
double async_compute_pi(size_t samples, size_t num_workers) {
	future<double> *results = new future<double>[num_workers];
	for (size_t i = 0; i < num_workers; i++)
		results[i] = 
			async(std::launch::async, compute_pi, samples/num_workers);
	double total = 0;
	for (size_t i = 0; i < num_workers; i++)
		total += results[i].get();
	delete[] results;
	return total / num_workers;
}

// Each task will be launched on a separate thread
```

**Some Special Things**
- #D/Dependent-Computation: Specification for a task that makes a task dependent on the result of some other task
	- Only a feature in SOME task system implementations
- #D/Task-Graph: Expression of more complicated dependencies in the form of a graph
	- Only a feature in SOME task system implementations, enables you to do things like wait for an entire graph of tasks to execute by simply using a regular .wait() or some equivalent.

#Asynchronous-Tasks/Summary
## Summary
- Another good method of parallelizing programs is using **Asynchronous Tasks**
	- Break a program down into granular operations called "tasks" instead of managing some shared data/resources
		- Going TOO granular can increase overhead, perform amortized or stopwatch analysis to hit some sweet spot
			- This overhead comes from managing/scheduling tasks in using a pool of threads
			- Can remedy this by either not going so granular or using methods such as putting a cap on the number of threads and switching to regular algorithms if there are too many threads (ie as in #E/Asynchronous-Tasks/Limited-Fibonacci, or switching between algorithms depending on input size (ie as in #E/Asynchronous-Tasks/Asynchronous-Quicksort)
	- There is also some speed to be gained between using lazy computation (only doing computation when result is actually needed), as opposed to starting up a new thread and starting computation immediately every time.

Keaton Hedrick
Apr 14 2025
:3