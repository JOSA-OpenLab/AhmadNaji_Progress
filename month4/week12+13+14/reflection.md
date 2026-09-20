Over the last three weeks, I focused primarily on strengthening my foundations while applying what I learned directly to a Woodpecker codebase.

During the first week, I studied Cornell’s PhD-level compiler course. My goal was not simply to learn compiler implementation, but to develop a deeper understanding of how programming languages and compilers work beneath the abstractions I normally rely on as a software engineer. The course exposed me to more rigorous ways of reasoning about compilation and program execution, and gave me a stronger theoretical foundation for understanding the systems I work with.

For the following two weeks, I shifted my focus toward Go and the Woodpecker CI codebase. I have been reading *Learning Go* to strengthen my understanding of the language itself while simultaneously exploring Woodpecker’s implementation. Rather than studying Go in isolation, I used the codebase as a practical reference for seeing how the language and its patterns are applied in a mature production system.

My exploration of Woodpecker focused particularly on understanding how pipelines and workflows are represented and executed. I traced the mechanisms responsible for moving workflows through their lifecycle and examined how the pipeline runtime interacts with the different backend integrations. This helped me move beyond understanding individual functions and toward building a mental model of how the different layers of the system cooperate.

The main theme across these three weeks has been connecting theory with implementation. The compiler course pushed me toward understanding systems from first principles, while studying Go alongside Woodpecker gave me an opportunity to apply that mindset to a real codebase. Instead of treating the code as something to modify until it works, I have been trying to understand why the system is structured the way it is, what responsibilities belong to each layer, and how changes propagate through the architecture.

I believe this approach is slower initially, but it gives me a much stronger foundation for making meaningful contributions later. The goal is not only to become familiar with Woodpecker or Go, but to improve my ability to enter an unfamiliar system, understand its architecture, and reason confidently about the consequences of the changes I make.

What I learned from those weeks helped me make a good contribution that I will explain in the reflection of the first week of month5.
