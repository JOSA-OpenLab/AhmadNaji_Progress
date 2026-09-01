---

## Task3: [PR](https://github.com/woodpecker-ci/woodpecker/pull/6017)

**issue**: the test file got rewritten three times for no reason

Going through the 13-commit refactor of `server/queue`, one thing jumped out: `fifo_test.go` gets restructured *three separate times* in three consecutive commits (08, 09, 10), each with a different opinion on "the right shape."

**What happened:**
- Commit 08 groups tests into `t.Run` subtests, each with its own fresh queue.
- Commit 09 throws that away for a shared `setupTestQueue(t)` helper, with tests now sharing one queue instance across subtests.
- Commit 10 bolts "edge case" comments onto commit 09's structure, including gems like `// Document current behavior - should either error or be idempotent`.

**Why it's premature:** Sharing a queue instance across test cases sounds like reasonable DRY-ing up, but this queue is concurrent and stateful — goroutines, channels, sleeps tied to `processTimeInterval`. Sharing it means each test now has to manually verify "the queue is actually clean before I start" instead of just getting a guaranteed-fresh one. That's a worse trade: you saved a few lines of `NewMemoryQueue(ctx)` calls and bought yourself test interdependence in the one domain (concurrency) where that's most dangerous to debug later.

The "should either error or be idempotent" comments are the tell. When your assertions are hedged like that, it usually means the refactor made the contract you're testing less clear, not more.

**Simpler alternative:** Just keep commit 08's shape — fresh queue per `t.Run`, no shared helper — and stop. It was already fine.

**What would actually justify the shared helper:** If `NewMemoryQueue` were expensive to construct — real connections, big fixtures, slow I/O — paying setup cost once would make sense. It's not; it's a struct with a couple of in-memory lists and maps. So there's no real cost being amortized here, just complexity being moved around.

**Bottom line:** this is a smaller issue than the nil-pointer bug in the Bitbucket forge, but worth a Conventional Comments nit — the test restructuring churn doesn't earn its keep and the original grouped-subtest version was simpler and safer.
