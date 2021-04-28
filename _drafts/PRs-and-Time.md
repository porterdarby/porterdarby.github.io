---
layout: post
title: PRs and Time
tags: git, devops
---

It feels like whenever I get onto a project these days I very often get a similar pile of problems. One of those problems that I often see is the long-running PR, the PR that Time Forgot, the Refactor PR, what have you. Basically, a PR that has been sitting in the list for like 2 months and hasn't been merged yet. Why? Because the project is waiting to merge so that a number of things are on the same page, especially in a multi-repo project (which is a topic for another time).

Whenever this happens, it almost always turns out to be a problem. This is not an issue with the team members, or an issue with the code, but rather just a natural fact about performing PRs. If the PR isn't merged in quickly enough, then it will sit there, causing a split. You'll end up in situations where you have your normal `master` and `dev` branches, but then you also have some sort of `major-refactor` branch. PRs that are made after the `dev`/`major-refactor` split need to target one of the two, or potentially both. If you don't do it right though, merging the refactor in to the core branch can/will have merge conflicts. And everyone just _loves_ merge conflicts.

# How do we not have this happen?
Don't let PRs sit if at all possible. If a PR has been started, that means that you have an upstream branch that has been pushed. If the branch languishes, then it's a problem. Make sure to have your team review any and all PRs quickly, and once it's approved merging quickly. If there is a "wait wait, don't merge it yet" scenario, you need to address that situation immediately. 

# How do we manage this once it happens?
Inevitably, you are going to run into a 