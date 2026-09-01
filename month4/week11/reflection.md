For the past weeks what I did was simply randomly exploring the Woodpecker codebase, I managed to map files/functions to their specific usages. After that I tried to build a testing environment for my work that uses woodpecker instances inside Minikube cluster on my machine, I used ArgoCD manifest files to setup the kubernetes to run the woodpecker workers and I tests some workflows in debugging mode to understand how it works.

Now I have a good understanding of the architecture of the server/workers and they follow a master/slave architecture and workers are what execute the workflows and server is mostly about control and data collection/visualization.

To the next week I'll checkout any good issues and try to trace back the code path so I can understand where the last contributions are focused, and if I find any good features that will help making Codeberg a better forge (it's where Wooopecker is used) I'll prioritize them.
