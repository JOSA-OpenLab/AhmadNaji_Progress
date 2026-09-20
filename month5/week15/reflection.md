For the first week of month5 and a bit of the second week, I worked on implementing individual workflow cancellation in Woodpecker CI. The goal was to allow users to cancel a specific pending or running workflow without stopping the rest of the pipeline.

What initially appeared to be a relatively small feature quickly became an exercise in understanding how state is managed across the entire pipeline lifecycle. Implementing cancellation required me to trace the feature through multiple layers of the system, including the API, pipeline logic, queue, datastore, runtime, frontend, and backend execution mechanisms.

One of the most important parts of the implementation was handling the distinction between pending and running workflows. A pending workflow can often be transitioned directly into a canceled state, while a running workflow requires communicating with the execution layer, stopping the associated task, allowing cleanup to happen, and eventually transitioning it into its terminal state. I also had to consider workflows that had not yet been inserted into the queue, which showed me that the visible workflow state does not necessarily describe everything happening internally.

The more challenging part was concurrency. Cancellation can happen at almost the same time as a workflow starts or finishes, so simply updating its state would introduce race conditions. I had to reason about conditional datastore updates and lifecycle locking to ensure that cancellation, startup, and completion could not incorrectly overwrite each other's state. This made the feature less about adding a "cancel" operation and more about defining a safe state transition within an already concurrent system.

Dependency handling introduced another important dimension. Canceling one workflow should not automatically terminate unrelated workflows in the pipeline, while dependent workflows still need to respect their configured success and failure conditions. I also had to make sure these dependency states could be reconstructed correctly after a server restart rather than relying only on transient in-memory state.

The implementation eventually touched almost every relevant layer of the feature: the API endpoint and permission checks, pipeline state transitions, queue behavior, datastore operations, frontend controls, backend cancellation, documentation, OpenAPI definitions, and tests.

The main thing I took away from this work is that implementing a feature in an established system is often much more about understanding invariants than writing the feature itself. "Cancel this workflow" sounds like a simple operation from the user's perspective, but internally it crosses concurrency boundaries, persistent state, scheduling, dependency resolution, and backend execution.

Working through the feature gave me a significantly clearer mental model of how Woodpecker coordinates pipelines and workflows. It also reinforced why understanding an existing architecture before modifying it matters: the difficult part was not making cancellation happen, but making sure cancellation behaved correctly regardless of when it happened and without violating the assumptions made by the rest of the system.

For the next week I'm planning on going through the iterations of the review process until I get my PR accepted.

[The PR link](https://github.com/woodpecker-ci/woodpecker/pull/7165)

[The Issue link](https://github.com/woodpecker-ci/woodpecker/issues/7156)
