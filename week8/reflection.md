# Task1 -> Woodpecker CI


## Work completed

I profiled the Woodpecker YAML schema linter using Go CPU profiling. The goal was to measure the real bottleneck before changing the code, then measure the same workload again after the optimization.

The focused workload ran the `TestSchema` test suite 20 times:

```bash
go test -tags test -run '^TestSchema$' -count=20 \
  -cpuprofile=/tmp/woodpecker-task1-schema-baseline.pprof \
  ./pipeline/frontend/yaml/linter/schema
```

## Problem found

The initial profile showed that `schema.Lint` spent most of its time compiling the JSON schema. The call path included:

```text
schema.Lint
└── gojsonschema.Validate
    └── gojsonschema.NewSchema / SchemaLoader.Compile
```

The schema definition itself is constant, but `gojsonschema.Validate` compiled it again on every lint call. This repeated work was the main performance bottleneck.

## Change made

I changed the schema linter so that:

1. The schema is compiled only once.
2. The compiled schema is cached for later calls.
3. `sync.Once` protects initialization when multiple pipelines call the linter concurrently.
4. Each later lint call validates the document using the already-compiled schema.

The conceptual flow changed from:

```text
Every Lint call → compile schema → validate document
```

to:

```text
First Lint call → compile schema once
Every Lint call → validate document using cached schema
```

## Before-and-after results

| Measurement | Before | After |
|---|---:|---:|
| Profile duration | 1.47 s | 188.31 ms |
| Total CPU samples | 1.86 s | 220 ms |
| Schema compilation | About 1.18 s cumulative | No longer appeared as a hotspot |
| `schema.Lint` cumulative time | About 1.32 s | About 150 ms |
| Main remaining work | Schema compilation and validation | YAML conversion and validation |

The profile duration decreased by approximately **87%**, and the total CPU samples decreased by approximately **88%** for this repeated test workload.

The CPU sample total can be greater than wall-clock duration because the tests may execute concurrently. The changed build ID is expected because the program was rebuilt after the code change.

## Interpretation

The optimization worked because it removed repeated schema compilation rather than trying to optimize the validation logic itself. After the change, the profile no longer shows schema compilation as the dominant cost.

The remaining profile is mainly occupied by YAML-to-JSON conversion, especially `go-yaml2json.ConvertNode`, and by the actual schema validation. This means a future optimization should be investigated separately and measured before changing it.

The larger percentage shown for some remaining functions after the change does not mean they became slower. The total amount of work became much smaller, so the remaining operations represent a larger percentage of the reduced profile.

## Verification status

- Completed before-and-after CPU profile comparison.
- Confirmed that schema compilation disappears from the main after-profile hotspots.
- Confirmed that document validation still occurs through the compiled schema.
- Remaining checks: run the focused tests with the race detector and run the relevant full test/e2e workload after the change.

## Main lesson

Performance work should follow this cycle:

```text
Measure → identify the dominant cost → make a small change → measure again
```

In this case, profiling showed that caching the compiled schema was a high-impact, low-risk improvement.

# Task2:
For this task I searched for a repo with N+1 problem but I didn't find any to be honest.

# Task3:
To improve my ability to interpret profiles, I studied three real flame graphs published by Brendan Gregg. I deliberately chose graphs with different metrics because a flame graph does not always measure CPU time. The meaning of the width depends on what was sampled: CPU samples indicate code consuming processor time, allocation bytes indicate memory pressure, and off-CPU time indicates time spent blocked or waiting.

When reading a flame graph, I use the following rules:

- Each rectangle represents a stack frame, or function.
- The vertical direction represents call-stack depth.
- The width represents the measured metric, not necessarily elapsed time.
- The horizontal ordering is not a timeline; frames are arranged to merge similar stacks.
- The widest paths are the first places to investigate, but the metric must be understood before calling something a bottleneck.

### 1. CPU flame graph: Bash running under Xen

**Graph:** [CPU flame graph for Bash](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)  
**Explanation:** [Brendan Gregg — CPU Flame Graphs](https://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html)

This graph represents sampled on-CPU stack traces. Therefore, a wide branch means that the processor spent a large proportion of its sampled time executing that path. The graph shows Bash activity flowing through command execution, redirection, system-call entry, and kernel code. A particularly important path includes `xen_hypercall_xen_version`, `check_events`, and `syscall_trace_enter`. The accompanying profile reports `xen_hypercall_xen_version` at about 20% of the samples.

The main lesson is that the visible workload is Bash, but the hottest work is not necessarily Bash's own parsing or business logic. A significant amount of CPU time is being spent in the interaction between Bash, system calls, tracing, and the Xen hypervisor. The likely bottleneck is therefore virtualization or system-call overhead triggered by the Bash workload, rather than one expensive shell function alone.

This graph also demonstrates why a flame graph is more useful than reading thousands of textual stack traces. The wide paths expose the dominant call chains immediately, while the stack depth explains how the application reached the expensive kernel or hypervisor operation.

### 2. Memory flame graph: MySQL allocation volume

**Graph:** [MySQL `malloc()` bytes flame graph](https://www.brendangregg.com/FlameGraphs/mallocbytes_mysqld1.svg)  
**Explanation:** [Brendan Gregg — Memory Flame Graphs](https://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html)

This graph uses the number of allocated bytes as its width metric. It is not showing CPU consumption and it is not simply counting how many times `malloc()` was called. The widest regions show the call paths responsible for the largest total memory allocation during the MySQL benchmark.

The hot allocation path is `JOIN::exec()`, not `JOIN::optimize()`. This is an important distinction. A separate graph that counts allocation calls identifies `st_select_lex::optimize()` and `JOIN::optimize()` as the most frequent callers, but the bytes graph shows that the execution path allocates more memory overall. In other words, many small allocations can dominate the call count, while fewer large allocations dominate the amount of memory.

The bottleneck indicated by this graph is memory-allocation volume during query execution. It does not prove that `JOIN::exec()` contains a memory leak; it only identifies where the benchmark allocated the most bytes. Further investigation would be needed to determine whether the allocations are necessary, excessively large, short-lived, or retained longer than intended.

This example taught me that selecting the wrong metric can lead to the wrong optimization. If the problem is allocation volume or memory pressure, a call-count graph is insufficient. The profiler must measure bytes, surviving allocations, page faults, or another metric that matches the actual symptom.

### 3. Off-CPU flame graph: Linux kernel compilation

**Graph:** [Off-CPU flame graph from a Linux kernel build](https://www.brendangregg.com/blog/images/2016/off-cpu-flame-graph-build.svg)  
**Explanation:** [Brendan Gregg — Linux eBPF Off-CPU Flame Graph](https://www.brendangregg.com/blog/2016-01-20/ebpf-offcpu-flame-graph.html)

This graph measures time when threads were not running on a CPU. That changes how the graph must be interpreted: wide regions represent accumulated blocked time, not processor instructions. The profile was taken during a Linux kernel build and includes thread names such as `as`, `make`, `gcc`, and `sh` at the bottom of the stacks.

The important blocking paths include `vfs_read()`, where assembler and build threads are likely waiting for filesystem reads, and child-process waiting paths involving `gcc` and `sh`. Other threads are also blocked in polling routines while waiting for work or events. These paths are wide because multiple threads can wait concurrently, so their blocked times accumulate. As a result, the total blocked time can exceed the wall-clock duration of the 30-second profile.

The bottleneck here is not necessarily a CPU-saturated function. It is waiting: filesystem I/O and process coordination contribute to the time experienced by the build. A normal CPU flame graph could miss this problem because blocked threads do not appear in on-CPU samples while they are waiting. The off-CPU graph therefore provides the missing half of the performance picture.

### Comparison and conclusions

| Graph | Width represents | What is hot | Bottleneck indicated |
|---|---|---|---|
| Bash CPU profile | On-CPU samples | System-call, tracing, and Xen-related paths | CPU overhead in the application–kernel–hypervisor path |
| MySQL memory profile | Total allocated bytes | `JOIN::exec()` | Allocation volume during query execution |
| Linux kernel build off-CPU profile | Accumulated blocked time | Filesystem reads, child-process waits, and polling | I/O and synchronization latency rather than CPU execution |

The broader lesson is that flame graphs are a visual summary of a specific measurement. The widest rectangle is only meaningful after I know what was measured. CPU, memory, and off-CPU graphs can describe completely different bottlenecks in the same program. Effective performance analysis therefore follows this sequence:

```text
Identify the symptom → choose the matching metric → read the widest paths → verify the cause with another measurement
```

These examples also reinforce the workflow I used in Woodpecker: profile first, identify the dominant cost, make a targeted change, and profile again. A flame graph helps locate the suspected bottleneck, but it does not replace testing or causal verification.

# Task4:

