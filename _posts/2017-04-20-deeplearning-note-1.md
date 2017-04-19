---
layout: post
title:  "Deep Learning Note 1"
date:   2017-04-19 16:16:01 -0600
categories: jekyll update
---

Courses from _Udacity_ _Deep_ _Learning_

### Quiz: Softmax
#### Note: Your softmax(x) function should return a NumPy array of the same shape as x.
#### For example, given a list or one-dimensional array (which is interpreted as a column vector representing a single sample), like:
> #### scores = [1.0, 2.0, 3.0] It should return a one-dimensional array of the same length, i.e. 3 elements:
> #### print softmax(scores) [ 0.09003057 0.24472847 0.66524096] Given a 2-dimensional array where each column represents a sample, like:
> #### scores = np.array([[1, 2, 3, 6], [2, 4, 5, 6], [3, 8, 7, 6]]) It should return a 2-dimensional array of the same shape, (3, 4):
> #### [[ 0.09003057 0.00242826 0.01587624 0.33333333] [ 0.24472847 0.01794253 0.11731043 0.33333333] [ 0.66524096 0.97962921 0.86681333 0.33333333]] The probabilities for each sample (column) must sum to 1. Feel free to test your function with these inputs.

```python
"""Softmax."""

scores = [3.0, 1.0, 0.2]

import numpy as np

def softmax(x):
    """Compute softmax values for each sets of scores in x."""
    
    return np.exp(x) / np.sum(np.exp(x),axis=0)
    #pass  # TODO: Compute and return softmax(x)

print(softmax(scores))

# Plot softmax curves
import matplotlib.pyplot as plt
x = np.arange(-2.0, 6.0, 0.1)
scores = np.vstack([x, np.ones_like(x), 0.2 * np.ones_like(x)])

plt.plot(x, softmax(scores).T, linewidth=2)
plt.show()
```

___________________________________________

### Quiz: One-Hot Encoding Quiz
|Letter|Prob|Prob|Prob|Prob|
| ---- | -- | -- | -- | -- |
|a     |0   |0   |0   |(1) |
|b     |0   |0   |1   |0   |
|c     |(1) |0   |0   |0   |
|d     |0   |1   |0   |0   |
|(c)   |0.7 |0.0 |0.1 |0.2 |

___________________________________________

### Cross Entropy
#### S = 0.7,0.2,0.1 L = 1,0,0 D(S,L) = -Sum(Li*Log(Si))
#### Multinomial Logistical Classification input X -> Linear Model (wx+b) -> Logit -> Softmax -> Cross Entropy -> 1-Hot Encoding
#### Loss = 1/N(Sum(Multinomial Logistical Classification))
#### to Lower Loss -> Grading Descent

```python
# Numerical Stability
#

original = 1000000000.0

for i in range(0,1000000):
    original += 0.000001
    
print(original - 1000000000.0)

original = 1

for i in range(0,1000000):
    original += 0.000001
    
print(original - 1)
```
> 0.95367431640625
>
> 0.9999999999177334

#### Loss = 1/N(Sum(D(S(wx+b),L)))
#### x -> Make mean = 0 ex. (pixel-128)/128
#### w -> 1 sigma (variance)
#### b = 0
