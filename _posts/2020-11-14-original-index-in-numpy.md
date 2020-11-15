---
layout: posts
tags: ["python", "numpy"]
title: "Searching for original index in numpy"
published: true
---
A common pattern found doing classification:
Given dataset `X`, label `Y`, and a class `c`, operate on `X[Y=c]`and extract a few data points.

For example, the following example sort all data points in class 0 and find out the two smallest and two biggest:

```python
# finds the two smallest samples and two largest samples of that class
X_class = X[Y=0]
args = np.argsort(X_class)[[0,1,-2,-1]]
sample = X_class[args]
```
Now what if I need to know where these 4 data points are in the original dataset `X`? Notice it is not `args`, as `args` stores the positions in `X_class` rather than `X`.

The trick is to do another indexing:
```python
ori_args = np.argwhere(Y == 0).squeeze()
```
and `ori_args[args]` gives you the four original position of the data points in `X`.
