---
title: "Print Dataframe"
date: 2021-11-09T17:29:50+01:00
draft: true
---

This is a nicely way of printing a Pandas DataFrame:

```python
from tabulate import tabulate
print(tabulate(df, headers='keys', tablefmt='fancy_grid'))
```