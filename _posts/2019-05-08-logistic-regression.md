---
title: "Logistic Regression Notes"
date: 2019-05-08
permalink: /notes/2019/05/08/logistic-regression
--- 

## Theory
We want to classify a set of inputs. But, rather than explicit classification, we want to provide a probability according to the sigmoid function:
```
y = 1/(1+e^(-z))
```
![alt_text](https://cdn-images-1.medium.com/max/1600/1*sOtpVYq2Msjxz51XMn1QSA.png)

Here, we will make `z = XW` where `X` is an nxd array of inputs and `W` is an dx1 array of weights. Hence, `z` and `y` will both be nx1 arrays.

We compute the set of weights `W` by minimizing the cross entropy cost function:
```
J_theta = -sum_i=0_n y_i * ln(y_hat_i)

# Here, y_i is an explicit classification (either 0 or 1) and y_hat_i is the probability that a given input is 1 
# (e.g. y = [0, 0, 1], y_hat = [0.3, 0.3, 0.4] => J_theta = -(0 + 0 + 1*ln(0.4) = -ln(0.4))).
```

We minimize this by taking the derivative of the function with respect to `W`:
```
d(J_theta)/d(W_j) = -1/y*dy/dW_j = 
```
and applying backwards propagation. Essentially, we update the edge weights according to the rule, penalizing incorrect classes (j) and encouraging correct ones (y):
```
W = W - alpha*d(J_theta)/d(W)
```


#### Advantages
- Obtain probabilities instead of explicit classifier
-
#### Disadvantages
- Still a simple classifier, highly sensisitive to outliers

#### Implementation
```python
>>> X = np.array([
... [1, 2],
... [3, 4],
... [5, 6],
... [7, 8],
... [9, 10]])
>>> X.shape
(5, 2)
>>> y = np.array([0, 0, 0, 1, 1])
>>> y.shape
(5,)
>>> from sklearn.model_selection import train_test_split
>>> X_train, X_test, y_train, y_test = train_test_split(X, y)
>>> lm = LogisticRegression().fit(X_train, y_train)
>>> lm.score(X_test, y_test)
1.0
>>> lm.coef_
array([[ 0.27272919, -0.15845028]])
>>> lm.intercept_
array([-0.43117947])
```

