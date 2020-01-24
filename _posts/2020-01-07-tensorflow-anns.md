---
title: "Tensorflow 2.0 Artificial Neural Network Cheat Sheet"
date: 2020-01-07
permalink: /notes/2020/01/07/tensorflow-anns
tags:
  - python
  - notebook
--- 

## Data Preprocessing

### Train Test Split

Training Data - data that our model **fits** on

Test Data - data that our model will test or **validate** itself on

Typically, we want our training data to be ~0.7 or more of our total data and the validation data is the rest.

Code:
```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)
```

### Data Normalization

We want all the data in the dataset to be on a normal scale, so we must normalize the data.

**NOTE:** do not normalize on the test data since we technically *don't have* it. We only want to fit on the training data to prevent data leakage.

Code:
```python
from sklearn.preprocessing import MinMaxScaler
scaler = MinMaxScaler()

scaler.fit(X_train) # only fit to training data

# but transform both training and test data
X_train = scaler.transform(X_train)
X_test = scaler.transform(X_test)
```

## Model Creation

Now we want to create a model. A model in an ANN consists of multiple Dense layers followed by one final output layer.

Code:
```
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense,Dropout

model = Sequential()
```

The number of layers and the number of neurons in each layer is pretty arbitrary. But, typically the first layer should have some correspondence to the number of inputs and we should scale the number of neurons gradually.

Usually, we use the rectified linear activation function or `'relu'` for the hidden layers.

Final Layer:

The final layer's neurons depends on the problem:
  * If binary classification: use 1 (it either is or isn't the class)
  * If multiclass classification: use the number of classes (the output will provide a probability for each class)
  * If regression: use 1
The final layer's function depends on the problem.
  * If binary classification: use `sigmoid`
  * If multiclass classification: use `softmax`
  * If regression: we don't need one



Code:
```
model.add(Dense(78, activation='relu'))
model.add(Dropout(0.2))

model.add(Dense(39, activation='relu'))
model.add(Dropout(0.2))

model.add(Dense(19, activation='relu'))
model.add(Dropout(0.2))

model.add(Dense(1, activation='<activation function>'))
```

Compiling the model:
When compiling, we want to provide the loss function, any additional metrics to calculate, and the optimizer.

Optimizer: usually `adam`

Additional metrics: usually `accuracy` since loss is an arbitrary value

The loss function depends on the problem:
  * If binary classification: use `binary_crossentropy`
  * If multiclass classification: use `categorical_crossentropy`
  * If regression: use `mse` (mean squared error)

Code:
```
model.compile(loss='<loss function>', metrics=['accuracy'], 
              optimizer='adam')
```

### Training the model
When training our model, we want to provide the training data, test data, and number of epochs.

An epoch is how many times we feed the training data through the network. Hence, more epochs means a longer training time, but a more rigorous fit.

Code:
```
model.fit(x_train, y_train, epochs=10, 
          validation_data=(x_test, y_test))
```

## Model Evaluation
Now let's plot our losses and accuracy, which is accessible via `model.history.history`.

Code:
```
losses = pd.DataFrame(model.history.history)
losses[['loss', 'val_loss']].plot()
losses[['accuracy', 'val_accuracy']].plot()
```
For a classification problem, we want to make a classification report and confusion matrix.

```
predictions = model.predict_classes(X_test)
from sklearn.metrics import classification_report,confusion_matrix
print(classification_report(y_test,predictions))
print(confusion_matrix(y_test,predictions))
```

For a regression problem, we want to simply plot our predictions and errors.

```
import seaborn as sns
predictions = model.predict(X_test)
mean_absolute_error(y_test,predictions)
np.sqrt(mean_squared_error(y_test,predictions))
plt.scatter(y_test,predictions)
plt.plot(y_test,y_test,'r')
errors = y_test.values.reshape(6480, 1) - predictions
sns.distplot(errors)
```
