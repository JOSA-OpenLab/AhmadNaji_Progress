## [Task1](https://github.com/TanStack/query/issues/10907#issuecomment-4687981033)
## [Task2](https://github.com/woodpecker-ci/woodpecker/issues/6708#issuecomment-4716855168)

## Task3
I started by reproducing an issue I found in a prettier-plugin [Issue](https://github.com/trivago/prettier-plugin-sort-imports/issues/394), I made the appropriate setup where I installed the plugin source code and make it a global library so the reproducing code can reference my locall version while bisecting instead of npm repo version, I started the bisect starting with version V5 since the submitter mentioned it was broken since version V6, doing manual bisect I've found that c42a081c45934daae541f1fde71984905acf0204 is the first bad commit, and the interesting part I found is that this commit implemented the feature wrong in the first place and it's still wrong since then, there is no commit that solved the issue according to my investigation, the problem is in file src/utils/assemble-updated-code.ts and I proposed a fix in this [PR](https://github.com/trivago/prettier-plugin-sort-imports/pull/411).

## Task4
The journal entry is in the other file in this directory


