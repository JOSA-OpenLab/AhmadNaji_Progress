Over the past two weeks,I changed my plans for the feature I was trying to add (because it didn't make sense these days to add it, it has some historical need), I investigated and improved a performance bottleneck in Woodpecker’s workflow queue. My goal was to make a measurable improvement to execution performance while preserving the queue’s existing behavior. This involved understanding the codebase, profiling a suspected bottleneck, implementing an optimization, and validating both its performance and correctness.

## Week 17: Investigation and Profiling

I began by exploring the codebase to understand how workflows move through the server, scheduler, and queue. Rather than immediately changing code that looked inefficient, I focused on finding an execution path where unnecessary work could affect the wider system.

The strongest candidate was dependency checking in the workflow queue. During each scheduling pass, the queue checked every pending task by scanning other queued and running tasks. As the backlog grew, this could produce quadratic work. I also found that tasks without dependencies still performed these scans.

What made this particularly important was that dependency filtering happened while holding the queue mutex. Slow filtering could therefore affect more than CPU usage: it could delay agents polling for work, newly submitted workflows, and completion updates.

I developed benchmarks covering independent tasks, dependency chains, tasks blocked on a running parent, and mixed workloads. Testing different queue sizes helped me understand how the cost scaled rather than relying on a single timing result.

Profiling confirmed the initial hypothesis. Dependency checking accounted for approximately 95% of sampled CPU in the isolated filtering workload and 91% in the concurrent dispatch workload. With 10,000 tasks blocked on a running parent, filtering took around 278 milliseconds per pass—longer than the queue’s 100-millisecond scheduling interval.

The main lesson from this first week was the importance of separating a convincing code-level suspicion from measured evidence. The profiles gave me a clear reason to prioritize this path and a baseline against which to evaluate the change.

## Week 18: Implementation and Validation

In the second week, I replaced the repeated scans with indexed dependency lookups. The optimized implementation skips dependency checking for tasks without dependencies and lazily builds a task-ID index when dependency checks are needed. It then checks dependency IDs against that index and the existing running-task map.

This reduced filtering from repeated quadratic scans to approximately linear work in the number of queued tasks and dependency references.

An important part of the implementation was preserving behavior. Waiting tasks can themselves be dependencies, so the index needed to include them before the queue was partitioned. I added deterministic regression tests covering dependency chains, cycles, missing dependencies, waiting-task membership, and ordering across repeated passes.

The results were substantial. For 10,000 tasks blocked on a running parent, median filtering time fell from approximately 278 milliseconds to 0.935 milliseconds—a speedup of about 298 times. Dependency chains improved by around 150 times, while independent tasks benefited even more because their unnecessary scans were eliminated.

I also measured the effect beyond the isolated function. In the concurrent dispatch workload, p95 dispatch latency fell from approximately 395 milliseconds to 102 milliseconds. Throughput increased from about 10.6 to 39.5 workflows per second in that benchmark. These results showed that the optimization improved scheduling behavior, not just a microbenchmark.

## The Pull Request

You can find the PR here: [PR](https://github.com/woodpecker-ci/woodpecker/pull/7303)

## Overall Reflection

The most valuable outcome was developing a complete, evidence-driven optimization process. I moved from code exploration to a hypothesis, established representative benchmarks, confirmed the bottleneck through profiling, and validated the implementation with regression tests, race detection, linting, and formatting checks.

I finished by documenting the methodology, results, and reproduction commands, then committing the work on a dedicated performance branch.

Overall, these two weeks strengthened my understanding of algorithmic complexity, lock contention, benchmarking, and performance tradeoffs. The key lesson I would carry forward is that a strong optimization should be measurable, behavior-preserving, and reproducible—and its limitations should be documented as clearly as its improvements.
