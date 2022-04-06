+++
title = "Scheduling"

date = 2022-01-31



[taxonomies]

categories = ["Operating Systems Design"]

tags = ["cs416"]

+++

The way that the operating system decides which processes to run and when is a complicated process known as **scheduling**. This is how it works.

<!-- more -->

We should understand a few vocabulary words before we discuss schedulers in detail.

- job: an execution stream for a certain amount of time (not the same as process!)

- workload: the jobs that must be executed

- latency: time taken for one operation

- throughput: number of total operations

## Metrics

We can design schedulers that are designed to minimize certain metrics. No scheduler is perfect at everything, so we need to prioritize what metrics matter.

- turnaround time: how long does the job take to complete?
  
  - \\( completionTime - arrivalTime \\)

- response time: how long does the job take to start? e.g. game keystrokes
  
  - \\( scheduleTime - arrivalTime \\)

- waiting time: how long does the job wait in the ready queue?
  
  - same as response except when killing procs

- throughput: jobs completed per unit time

- resource utilization: manage small resources well e.g. battery

- overhead: how strenuous is the scheduler itself?

- fairness: how well is CPU time shared between jobs?

## FIFO

FIFO stands for **First In, First Out**, which is a fairly self-explanatory title. A FIFO scheduler does jobs in the order that they arrive. This kind of scheduler is trivial to implement. However, we can run into issues if say, the first job takes a long time to complete. The other jobs that would complete much quicker are waiting even though it would be better if we could just get those out of the way first.

## SJF

SJF stands for **Shortest Job First**, which again is self-explanatory. This is better than FIFO, but relies on the OS having oracle-like knowledge of all the jobs that will come, since a shorter job could come later that the scheduler can't factor into its calculations.

These schedulers only work when you have access to all the jobs at once, which is almost never the case in real-world workloads. We need to use some kind of *preemptive scheduling* which will switch and split jobs as necessary.

## STCF

STCF stands for **Shortest To Completion First**. It is similar to SJF, but it recalculates the job closest to completion every time a new job arrives. This way, if a super short job arrives while a long job is executing, STCF can switch to it and complete it quickly before finishing the long job, which improves completion time. This expands the machinery needed in the scheduler but has much better outcomes.

## RR

RR stands for **Round Robin**. This approach is fairly different than the the previous two schedulers because it does not make any attempt to optimize for the shortest time. Instead, RR optimizes for fairness by executing every single job on the same exact time intervals. So it will run 10 ms of Job A, then 10 ms of Job B, then 10 ms of Job C, etc. in the fairest way possible regardless of the actual length of those jobs.

Preemptive scheduling is pretty cool, but we're still treating all our jobs like they're the same. If we introduce the notion of **priority** in jobs then we can use it for real-time jobs.

## MLFQ

MLFQ stands for **Multi-Level Feedback Queue**. The implementation is similar to RR but it works on multiple priority levels. There are two job types: *interactive* and *batch*. Interactive processes are those like games and text editors that need immediate response and therefore low response time. Batch processes are those like daemons that are lower priority and care more about general turnaround time than response time.

For our implementation, we have multiple priority queues:

```java
if a.priority > b.priority {
    a.run();
} else if a.priority == b.priority {
    rr(a, b);
}
```

MLFQ prioritizes *nice* processes. This is a technical term that means that a process is satistfied with getting a response and is willing to turn over the CPU as needed without the OS needing to force it to.

All jobs begin by having top priority, but if a job takes too long on the RR, then you demote it to a lower priority level. This way, smaller jobs that are contained within RR are executed quickly at top priority.

However, this system is incredibly easy to game. Processes control the jobs that they send to the CPU so you can split up jobs exactly aligned to RR to execute all the jobs at top priority even though you don't actually need it.

## Lottery

The lottery system is fairer, just like a real lottery. Every process gets a certain amount of lottery tickets associated with it, scaling with priority. Whichever process wins the lottery gets to run its job. This way, higher priority jobs have a higher probability of running than lower priority jobs, but it's protected against gaming the system. The number of tickets that a process gets represents its CPU usage.

```c
int counter = 0;
int winner = getrandom(0, total_tickets);
node_t *curr = head;
while curr {
    counter += curr->tickets;
    if (counter > winner) {
        break;
    }
    curr = curr->next;
}
```

## Multiprocessing

Every scheduler we've discussed so far assumes only a single CPU can execute jobs, which was true for a long time. Now, however, multicore CPUs are becoming increasingly common in consumer electronics. How do we schedule jobs among different CPUs?

*Cache affinity* and *coherence* can become issues. CPUs have a cache located next to them different from main memory where commonly used memory is stored in order to speed up operations. Every core has its own caches, so what happens when multiple cores execute the same stream? They have to have cache coherence so that we don't end up with bugs. The way to fix this is to copy caches between cores.

Obviously, this is pretty inefficient. We want to avoid this as much as possible by having each process run on its own CPU thread as much as possible; this is cache affinity. Basic FIFO scheduling is simple but is not good at preserving affinity. Having a scheduler that preserves affinity at the cost of some other inefficiencies can actually cause massive speedups because of the importance of cache affinity. However, this machinery can be complex.

## CFS

The **Completely Fair Scheduler** is the scheduler that Linux actually uses. Instead of using time-slices to manage job usage, the scheduler assigns processes a proportion of the CPU. The fairness is absolute because each thread gets an absolute amount of CPU running. This also fixes multiprocessing issues because the scheduler is based around CPU cores instead of time. However, the switching rate might change a lot because we are no longer looking at time, and as we discussed context switching has its own costs that we might want to minimize. This is a tradeoff made for fairness.

I mentioned niceness earlier. CFS uses a more complex version of niceness which ranges from -20 to 19, with the default of 0, and higher values being worse. There is also a separate priority called *real-time priority* ranging from 0 - 99 which always execute before nice processes, regardless of how nice they are.

In order to calculate RR, we take the target latency e.g. 20 ms and split the time across the tasks equally among process of the same priority. However, because this is based around CPU and not constant time, this can be extremely fast. A tree data structure is used for priority in order to quickly get the highest priority and least time jobs.

**Target Latency** is the minimum amount of time required to get a task at least one turn on the processor. Within this window, every process gets some CPU. **Minimum Granularity** imposes small unfairness in CFS. There is a floor on the timeslice of 1 ms regardless of how much CPU we have. This means that for a large amount of jobs, smaller jobs get unfairly good treatment.
