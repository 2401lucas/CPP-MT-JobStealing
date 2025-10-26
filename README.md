# CPP-MT-JobStealing
A lightweight and flexible C++ Thread Pool implementation with Job Stealing support.

# Overview
This threadpool implementation provides a simple and extentable Threadpool that supports parallel task execution using a job-stealing scheduler.

## What Is Job Stealing?

In multithreaded applications workloads can often become unbalanced. For example, if two threads each start with an equal share of work but one thread finishes early due to a conditional early exit, it will remain idle while others continue processing.

Job stealing addresses this by allowing idle threads to "steal" pending tasks from the queues of other threads.
This dynamic redistribution ensures more uniform utilization across all threads and significantly reduces idle time.

The core idea is that each thread maintains its own local queue of tasks, when a thread’s queue becomes empty it attempts to "steal" tasks from another thread’s queue. This can greatly improve performance, particularly when workloads have unpredictable task durations.

## Key Features
- Flexible Design – Users define their own Processor and WorkItem types.
- Custom Execution Logic – Thread behavior is fully customizable.
- Low Overhead – Minimal locking and no unnecessary synchronization for simple use cases.
- Optional Context Tracking – Extend WorkerContext to collect per-thread statistics.
- No Data Race Handling – The implementation assumes workloads are race-free, suitable for isolated or read-only operations.


# Usage Example

When creating a ThreadPool, it requires two user-provided types:

- A WorkItem – Describes a single unit of work (e.g., an index, range, or chunk of data).
- A Processor – Defines the per-thread logic and the shared state for processing work items.

Each worker thread executes Processor::Process(), which takes a WorkItem and a WorkerContext.
The WorkerContext (found in ThreadPool.h) can be extended to track statistics. Currently it only tracks the number of stolen batches.

```
struct WorkItem {
  uint32_t index; // Index into Data
  uint32_t count;
};

struct Processor{
  // Shared context between threads
  Data* data;

  // Per thread execution
  void Process(const WorkItem& item, WorkerContext& ctx) {
  // Logic to modify/read from data[item.index]
  }
};

ThreadPool<WorkItem, Processor> thread_pool(Processor(data));
thread_pool.Start();

// Queueing the workload, subdividing the initial workload can be done automatically or manually
for (uint32_t i = 0; i < workload_size; ++i) {
    thread_pool.SubmitBatch(WorkItem(i, 1));
  
}

  thread_pool.WaitForCompletion();
```


### Performance Notes

- Designed for CPU-bound workloads with many small, independent tasks.

- Scales well up to N threads on systems with minimal contention.

- Batch submission should be used for improved locality and reduced contention.
