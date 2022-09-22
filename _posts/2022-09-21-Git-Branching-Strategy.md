---
layout: post
title: "Git Branching Strategy Best Practices"
author: "Sivakumar Vunnam"
excerpt: "In this article we are going to cover Git Branching Strategy Best Practices,What is Git branching,Why do we need branching in Git,What are the advantages of branching in Git,Commands used in Git branching.
."
keywords: Git, Github, DevOps
draft: false
---
What is Git branching?
---------------
```Git branching is working on branches by pulling the codebase from the master as per requirement in the project without affecting the master codebase. The branch is a short-lived branch which is created for a particular feature or related work and can be deleted once the work is completed and merged back to the master branch.```

Why do we need branching in Git?
---------------
```Branching is necessary in a large project as multiple developers work on the same project from different machines and each may have to work on the related issue assigned to the```

What are the advantages of branching in Git?
---------------
```Developer can work on specific feature or as per the need in the project without affecting the codebase Multiple developers can work on the same codebase simultaneously Easy to work and detect bugs before merging to the master branch Can fix the problem when the need arises Git Branching Strategy Best Practices There can be many branches at various levels of working in order to get to a stable level.```

For example

```Master branch – Is the main branch which has the source code
Develop branch – Is the branch which developers work on
Feature branch – Is the branch which developer work on for assigned/particular feature
Release branch – Is the branch which has the code ready for release
Hotfix branch – Is the branch where a developer has to quickly fix an issue immediate Commands used in Git branching
```

`git branch <branch_name>` | creates a new branch

`git merge <branch_name>` | merges a branch

`git checkout <branch_name>` | To switch between branches

How does Git Branching Strategy works? Let’s understand with a diagram
---------------

```The simple way of understanding like from which branch will be the code branched off. As shown in the figure the develop branch and the hotfix branch work from the codebase which is on the master branch also known as main branch.

The release and the feature work on the code from the develop branch. In either case the main codebase is not affected if any failure happens. At anytime if there is any failure it can be rolled back and since the codebase is unaffected the developers can rework on it.```

Git Branching Strategy Best Practices 1
---------------
The flow diagram of branching off into different branches as per requirement.

Git Branching Strategy Best Practices 2
---------------
Conclusion:
---------------
In this article we have covered Git Branching Strategy Best Practices,What is Git branching,Why do we need branching in Git,What are the advantages of branching in Git,Commands used in Git branching.
