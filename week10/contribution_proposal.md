# Contribution Proposal — Argo CD

## 1. Project

### Project Overview

**Argo CD** is an open-source continuous delivery tool for Kubernetes based on the GitOps model.

The project uses Git repositories as the source of truth for defining the desired state of applications and infrastructure. Argo CD continuously compares the desired state stored in Git with the actual state running inside Kubernetes and provides mechanisms for synchronizing the two.

This makes Argo CD particularly relevant to areas such as:

- Kubernetes
- Continuous Delivery (CD)
- GitOps
- Infrastructure automation
- Cloud-native infrastructure
- DevOps

### Who Uses It

Argo CD is primarily used by organizations and engineering teams operating Kubernetes environments and adopting GitOps-based deployment workflows.

Some organizations that use ArgoCD are **Tesla**, **Adobe**, **Red Hat**, **Black Rock** and a lot more (which shows the importance of this tool to GitOps related work).

### Maintainers

Argo CD is an open-source project under the Cloud Native Computing Foundation (CNCF).

Argo CD sits under the broader **Argoproj** umbrella (CNCF graduated project, alongside Workflows, Events, Rollouts), governed by a shared `GOVERNANCE.md` and `membership.md`.

**Roles / maintainers**
- **Member** – active contributor, sponsored by 2 approvers/leads.
- **Reviewer** – reviews PRs for quality/correctness in a codebase area; defined by a `reviewers` entry in an OWNERS file.
- **Approver** – can approve merges, holistically judging compatibility, API/flag conventions, performance; defined by an `approvers` OWNERS entry. Approvers also help subproject leads triage issues and with project management.
- **Lead** – sets direction/roadmap for a subproject (e.g., Argo CD has its own lead(s)), responsible for driving the project, publishing a roadmap, organizing community meetings, publishing meeting notes, and reporting on status. Leads also escalate reviewer/maintainer concerns to the broader maintainers group and work with other subproject leads on overall project health.

For governance purposes, **reviewers, approvers, and leads all count as "maintainers."** Current CD-specific maintainers include people like Leonardo Luz Almeida and Keith Chong (Approvers - CD), Zach Aller (Reviewer - CD), and others, with affiliations spanning Intuit, Red Hat, and independent contributors.

**Decision-making process**
- Most issues are resolved by consensus; if consensus fails, they go to a vote, with all decisions recorded publicly on GitHub or in meeting notes.
- Regular votes need a simple majority with at least 30% maintainer quorum, cast as +1/-1 on the relevant PR, open for one week.
- Each maintainer has one binding vote; no single organization may hold more than 40% of eligible votes (forcing over-represented companies to designate which maintainers vote).
- Governance changes themselves require a 2/3 super majority vote, also with 30% quorum, but open for two weeks instead of one.

## 2. Why This Project?

I chose Argo CD because I want to develop a deeper understanding of DevOps, particularly **GitOps**.

I currently understand Git and several DevOps concepts, but I want to move beyond simply using DevOps tools and develop a deeper understanding of how production-grade infrastructure tooling is designed and implemented.

Argo CD provides an opportunity to study GitOps through a real, widely adopted project rather than learning the concept only theoretically.

By studying and contributing to Argo CD, I expect to develop a better understanding of areas including:

- GitOps architecture and principles
- Kubernetes
- Continuous Delivery
- Reconciliation and desired-state systems
- Distributed systems
- Production-grade Go codebases
- Open-source project architecture
- Designing and proposing changes through RFCs
- Collaborating with maintainers of a large open-source project

This project also aligns with my career direction because I want to gain more practical experience with DevOps and cloud-native infrastructure.

---

## 3. Target Issue / RFC

At the beginning of the project, I will **not commit to a specific implementation immediately**.

My first objective will be to understand the Argo CD architecture, contribution process, existing roadmap, and recurring problems discussed by maintainers and contributors.

I will investigate:

- Existing RFCs and proposals
- Open issues
- Project roadmap items
- Maintainer discussions
- Recurring architectural or usability problems
- Areas where maintainers are actively requesting contributions

I will also participate in the project's communication channels, including Slack and office hours where appropriate, to understand which problems would make meaningful contribution targets.

The intended outcome of this research phase is to identify a problem substantial enough to require design work rather than selecting an arbitrary issue only for the purpose of submitting a pull request.

### Selected RFC / Problem

**Issue/RFC:** Allow cluster-specific QPS and Burst limit configuration for target clusters

**Link:** https://github.com/argoproj/argo-cd/issues/29001

**Problem:**
Argo CD currently configures Kubernetes REST client rate limiting using global QPS and Burst values that apply to every managed destination cluster. This becomes problematic in multi-cluster environments because different clusters may have very different capabilities and workloads. Small or edge clusters can be overloaded by aggressive global limits, while large clusters may be unnecessarily slowed down by conservative values.

The proposed change is to allow QPS and Burst limits to be configured per cluster while preserving the existing global configuration as a fallback. The issue proposes two possible designs: storing the values as annotations on the Argo CD Cluster Secret, or adding them as first-class fields inside the cluster configuration data.

**Why this problem matters:**
In large multi-cluster deployments, a single global rate limit creates a "lowest common denominator" problem. Setting the limits low enough to protect weaker clusters reduces reconciliation and synchronization performance for larger clusters, while setting them high enough for large clusters can cause API server throttling, increased latency, or instability on smaller clusters. Per-cluster configuration would therefore improve both scalability and operational stability.

**Why I selected it:**
I selected this issue because it involves several areas that match my learning goals for this project: Kubernetes API interaction, Argo CD's multi-cluster architecture, configuration design, scalability, and GitOps infrastructure. It also requires more than simply implementing a small isolated fix; the design must first determine how cluster-specific configuration should be represented while maintaining backward compatibility with the existing global configuration.

The issue is also explicitly marked as requiring a proposal and is classified under scalability and cluster management, which makes it a good candidate for practicing the full contribution process I want to experience: understanding the existing architecture, discussing design alternatives with maintainers, proposing a solution, implementing it, and iterating through review.

---

## 4. 12-Week Milestone Plan

The 12 weeks will be divided into six broad phases:

1. Codebase and architecture research
2. Maintainer discussions
3. Problem/RFC discovery
4. Proposal and design
5. Implementation and iteration
6. Pull request and review

### Weeks 1–2 — Understand the Project

The first two weeks will focus primarily on understanding Argo CD rather than modifying it.

Objectives:

- Build Argo CD locally.
- Learn how to run its development environment.
- Read the contribution documentation.
- Understand the repository structure.
- Identify the major components of Argo CD.
- Trace several important execution paths through the codebase.
- Study the architecture documentation.
- Read previous RFCs/proposals to understand how architectural changes are discussed.
- Study the project's issue and pull-request workflow.

**Deliverable:**
A personal architecture map and notes describing the major Argo CD components and how they interact.

---

### Weeks 3–4 — Maintainer Discussions and Problem Discovery

After gaining an initial understanding of the architecture, I will begin interacting more actively with the Argo CD community.

Activities will include:

- Participating in the Argo CD Slack community.
- Attending office hours or community meetings where possible.
- Asking maintainers about areas needing architectural or implementation work.
- Studying open RFCs and roadmap items.
- Investigating recurring issues in the issue tracker.
- Evaluating candidate problems based on technical value, scope, and feasibility within the remaining time.

**Deliverable:**
A shortlist of approximately **3** candidate contribution ideas, including their expected scope, difficulty, and value to the project.

---

### Weeks 5–6 — Select and Develop the Proposal

During this phase, I will select one contribution target based on the previous research and feedback from maintainers.

The goal will be to understand the problem thoroughly before implementing a solution.

Activities:

- Reproduce or investigate the selected problem.
- Trace the relevant code paths.
- Study previous attempts or related pull requests.
- Discuss possible solutions with maintainers.
- Identify architectural constraints.
- Write an initial RFC/design proposal if the change requires one.
- Incorporate early maintainer feedback.

**Deliverable:**
A concrete RFC/design proposal or an agreed implementation plan for the selected contribution.

---

### Weeks 7–8 — Initial Implementation

Once the proposed direction has received sufficient feedback, I will begin implementation.

Activities:

- Implement the core change.
- Add or modify tests.
- Run the project's existing test suites.
- Validate the implementation against the agreed design.
- Document implementation decisions.
- Regularly communicate progress and blockers with maintainers.

The implementation will intentionally be iterative rather than attempting to produce a complete solution before receiving feedback.

**Deliverable:**
A working initial implementation of the proposed change.

---

### Weeks 9–10 — Iterate With Maintainers

The implementation will be presented to maintainers and contributors for technical feedback.

Potential work during this phase includes:

- Addressing architectural concerns.
- Refactoring the implementation.
- Handling edge cases discovered during testing.
- Improving test coverage.
- Updating documentation.
- Revising the RFC/design if implementation reveals incorrect assumptions.
- Performing additional validation where necessary.

**Deliverable:**
A contribution considered technically ready for formal upstream review.

---

### Weeks 11–12 — Pull Request and Review

The final phase will focus on upstreaming the contribution.

Activities:

- Prepare the final commits.
- Ensure commits and documentation follow Argo CD contribution conventions.
- Submit the pull request.
- Clearly document the problem and proposed solution.
- Link the relevant RFC/issues.
- Respond to review comments.
- Make requested changes.
- Continue iterating until the PR is accepted or a clear next step is established.

**Final deliverable:**
An upstream Argo CD pull request representing the result of the 12-week contribution effort.

**PR:** [A link will be provided when I finish this week.]

---

## 5. Risks

### Maintainer Availability

One major risk is that maintainers may be unavailable or slow to respond.

Because the project depends on feedback during problem selection, design, implementation, and review, delayed communication could block progress.

**Mitigation:**
Avoiding depending entirely on one maintainer. Use public project communication channels, community meetings, Slack, issues, and RFC discussions whenever possible.

---

### Insufficient Documentation

Some internal components may have incomplete or outdated documentation.

This could increase the amount of time required to understand the architecture and relevant code paths.

**Mitigation:**
Using multiple sources of information, including:

- Source code
- Git history
- Existing tests
- Previous pull requests
- Existing RFCs
- Issues
- Maintainer discussions

When documentation is insufficient, understanding the implementation directly from the source code will become part of the research process.

---

### Poor Problem Selection

The selected contribution may turn out to be too large, too small, already being implemented, or unsuitable for the project's current direction.

This is particularly dangerous because selecting the wrong problem early could waste several weeks.

**Mitigation:**
Delaying committing to a major implementation until the architecture has been studied and the proposed problem has been discussed with maintainers.

---

### Incorrect Ordering of Work

Another risk is performing work in the wrong order—for example, implementing a solution before understanding the architecture or before receiving feedback about whether maintainers would accept the design.

**Mitigation:**
Follow the sequence:

**Understand → Discuss → Select → Design → Validate → Implement → Review**

Implementation should begin only after sufficient research and early maintainer feedback.

---

### Proposal Rejection

Even after research and discussion, maintainers may reject the proposed design or request substantial changes.

This is a normal risk when contributing architectural changes to an established open-source project.

**Mitigation:**
Seeking feedback early and keep proposals small enough that they can be changed before significant implementation effort has been invested.

---

## 6. Mentorship Required

I will need mentorship primarily in areas where project-specific experience and architectural judgment are more valuable than information available from documentation.

In particular, I expect to need mentorship with:

### Argo CD Architecture

I may need guidance understanding why certain architectural decisions were made and identifying which components are appropriate extension or modification points.

### Selecting an Appropriate Contribution

I will need feedback when evaluating potential RFCs or issues to ensure that the selected problem:

- provides meaningful technical depth,
- is valuable to the project,
- has a realistic scope for the available time, and
- aligns with the direction maintainers want the project to take.

### RFC and Design Review

Because part of the goal is learning how changes are designed in a large open-source project, I will need feedback on:

- defining the problem clearly,
- evaluating alternative solutions,
- understanding architectural trade-offs,
- controlling scope, and
- communicating the proposal effectively to maintainers.

### Fallback: if no mentor was found, anyone who has a good experience with Distributed Systems will be good

---

## Expected Outcome

The primary objective of this project is not simply to produce a pull request.

The objective is to experience the complete process of contributing a meaningful change to a mature open-source infrastructure project:

**understanding the system → identifying a real problem → discussing it with maintainers → designing a solution → implementing it → receiving feedback → iterating → upstreaming the contribution**

By the end of the 12 weeks, I expect to have a substantially deeper understanding of GitOps, Argo CD, Kubernetes-oriented infrastructure tooling, and the engineering process behind contributing non-trivial changes to a mature open-source project.
