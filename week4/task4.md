## Chosen Pull Request discussions
[First PR #139850](https://github.com/kubernetes/kubernetes/pull/139850)
[Second PR #139842](https://github.com/kubernetes/kubernetes/pull/139842)
[Third PR #139791](https://github.com/kubernetes/kubernetes/pull/139791)


## How review norms played out across these three PRs

**#139850** (kubelet memory leak fix), **#139842** (kubeadm learner promotion), **#139791** (probe error clarity)

### What gets flagged

The reviewers consistently zeroed in on a few specific things, and none of it was style nitpicking:

**Structural cleanup over correctness alone.** In #139842, ahrtr's first round of comments wasn't about whether the fix worked, it was about *where* the check happened. He pushed the author to check learner status before attempting promotion rather than after a failed attempt, and to consolidate two near-duplicate variables (`promoteResp` and a separate member list) into one shared `memberList`. The code already solved the bug; the reviewer was optimizing for fewer code paths to reason about later.

**Removed TODOs and intentionally-deleted context.** In #139850, HirazawaUi caught that the author's refactor had silently dropped a TODO comment that had been there to flag temporary behavior. That got flagged hard enough that the author had to restore it plus add two new explanatory lines for whoever picks up the TODO next. This wasn't a "nice to have," it was treated as something that had to happen before lgtm.

**Test structure, not just test existence.** #139842 had a unit test from the start, but neolit123 still pushed for it to be restructured into a table-driven test with subtests (`t.Run()`), specifically because that function ("MemberPromote") had a history of recurring bugs and he wanted a structure that would make it easy to bolt on more cases later. The reviewer was thinking about the next five bugs in that function, not just this one.

**Compatibility blast radius.** This is the big one in #139791. The PR started as a behavioral change (named port not found should now fail the probe instead of returning Unknown). aojea immediately flagged that this would silently break currently-passing workloads after a cluster upgrade, and cited a specific historical precedent (a five-year-long feature flag removal) as the cost of getting this wrong. That single comment caused the author to scope the entire PR down to just an error-message fix and drop the behavior change entirely.

**Whether validation already covers the edge case elsewhere.** Late in #139791, SergeyKanzhelev didn't just approve, he asked whether kubelet validates the same thing for static pods (which bypass the API server). The author had to go check `ValidatePodCreate`, test it on an actual cluster, and report back with the exact error string kubelet produces. That's a "prove it" follow-up, not a rubber stamp, even after an lgtm was already on the PR from someone else.

### What doesn't get flagged

- **Missing unit tests, if justified.** In #139850, the author straight up said "I don't have any unit tests for this, here's why" (low value relative to complexity, especially since the touched code has a TODO marking it as temporary). No reviewer pushed back on that. The bar isn't "always add tests," it's "tests should be worth what they cost."
- **Small wording and naming things.** Variable naming suggestions (`memberList` instead of two separate vars) got a thumbs up and an immediate fix, no back and forth, no debate.
- **PR title changes, scope renames.** Both #139850 and #139791 had their titles changed mid-review to reflect a narrower or corrected scope. Nobody treated that as a big deal, it's just bookkeeping once the actual content changes.
- **Self-merges of straightforward bug fixes.** None of these turned into multi-week debates. #139842, despite getting fairly detailed structural feedback, went from open to merged within about a week.

### How disagreements got resolved

The interesting thing here is there wasn't really a "disagreement" in the adversarial sense in any of these threads. The pattern instead looks like this:

1. **Reviewer raises a concern with a concrete reason attached**, often linking to a past PR, a past incident, or an existing related issue (aojea linking the five-year flag removal, ahrtr referencing "network jitter" as a reason to add a retry around `listMembersOnce`).
2. **Author asks a direct, scoped question back** rather than arguing. In #139791, the author flat out asked "do you think this is worth pursuing, or should I scope it down?" instead of defending the original design. That question is what actually unblocked the thread, it turned a values disagreement (should this be a breaking change) into a scoping decision the author could just make.
3. **Scope gets reduced rather than the reviewer being argued out of their position.** This shows up twice. The kubeadm PR didn't get the reviewer to drop the "add network jitter retry" suggestion, the author just implemented it. The probe PR didn't convince aojea that the compatibility risk was fine, the author removed the risky part entirely.
4. **A second or third reviewer sometimes reopens something everyone thought was settled.** This is probably the most distinctive pattern in the data. #139791 had already gotten an lgtm and approve from bitoku, and SergeyKanzhelev still asked a new question (static pod validation) after the fact, which led to another force-push and the lgtm label getting pulled and reapplied. The merge train doesn't stop just because one approver is happy; sig-level approvers seem to feel free to ask their own questions even late in the process.
5. **Maintainers explicitly route work forward instead of just closing the loop.** neolit123's comment in #139842 is a good example: approve the fix, but immediately point out that `MemberPromote` as a whole still needs better test coverage, and that's a separate PR (#139958 ended up being exactly that). Nobody tried to cram the broader fix into the current PR.

### The general shape of it

None of these reviews are about taste. Every substantive comment ties back to one of: future maintainability (TODOs, table-driven tests), blast radius for existing users (the probe compatibility discussion), or closing a logic gap the author hadn't considered (checking state before vs. after, static pod validation). And the way disagreements resolve is less "reviewer wins" or "author wins" and more "scope gets renegotiated until the risk is gone," usually triggered by the author asking a clarifying question rather than pushing back on the substance.

Want me to keep going if you send more PRs, or is three enough of a sample for what you're trying to do with this?
