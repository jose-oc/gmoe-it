---
title: "Search Within Files"
date: 2021-11-09T17:50:31+01:00
draft: true
---

Useful commands: look for a text in files.

## Finding text

To find a text in certain files but skipping looking into certain directories and file types:

```shell
find . -type f \( -name "*.json" -or -name "*.py" \) ! \( -name "*._test.py" -or -path "*/.git/*" -or -path "*/node_modules/*" -or -path "*/.virtualenvs/*" -or -path "*/.venv/*" \) -exec grep --color -HinI 'text I want to find' {} +
```

In this example I look into json and python files but avoiding test python files and skipping git, node packages and python virtual environment directories.
