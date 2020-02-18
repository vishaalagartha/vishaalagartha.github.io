---
title: "Linear Regression Notes"
date: 2019-05-03
permalink: /notes/2019/05/03/linear-regression
--- 

#### Theory
We compute `y=XW` where `y` is the predictions (an nx1 array), `X` is the inputs (an nxm array), and `W` is the set of weights (an 1xm array). Here, we have `n` inputs and `m` features.

We compute the set of weights `W` by minimizing the least squares cost function:
```
J_theta = 1/2 sum_i=0_n (x_i*w_i-y_i)**2
``` 

We minimize this by taking the derivative of the function with respect to `W`:
```
d(J_theta)/d(W) = X(y_pred-y_i)
```
and applying backwards propagation. Essentially, we update the edge weights according to the rule:
```
W = W - alpha*d(J_theta)/d(W)
```
until we achieve a happy set of weights.

Linear regression is also nice because we have direct formulas for the equation as well:
```
m = (sum_i=1_to_n(x_i-X_bar)(y_i-Y_bar)) / sum_i=1_to_n (x_i-X_bar)^2
b = Y_bar - m*X_bar
```

#### Advantages
- Simple
- Easy to interpret and visualize

#### Disadvantages
- Only works well for linearly separable functions 

#### Implementation
```python
import numpy as np
from sklearn.linear_model import LinearRegression
X = np.array([
[1, 1],
[2, 3],
[4, 5]])
# we have 3 inputs with 2 features to create a 3x2 array
W = np.array([[
[1],
[2]])
# we have 2 feature weights to create a 2x1 array
y = np.dot(X, W) + 3 
# y = 1*x_0 + 2*x_1 + 3
lm = LinearRegression().fit(X, y)
```

We have now trained model. Here are some additional functions to extract parameters, get the score, etc.
```python
>>> lm.score(X, y)
1.0
>>> lm.coef_
array([[1., 2.]])
>>> lm.intercept_
array([3.])
>>> lm.predict(np.array([[3, 4]]))
array([[14.]]) # 1*3+2*4+3 = 3+8+3 = 14
```

