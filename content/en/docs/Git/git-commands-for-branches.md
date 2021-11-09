---
title: "Git branches"
date: 2021-11-09T17:50:31+01:00
draft: true
---

Useful commands.

# Useful git commands

## Branch name

I want to name branches with a code to identify the Jira ticket and a description. 
To write the description quickly but avoid using blank spaces in the git branch name, as they aren't allowed, I use the `tr` shell command to replace spaces with dashes.

```sh
git checkout -b PROJECT-123_$(echo "Make process events topic configurable via env variable" | tr ' ' -)
```
