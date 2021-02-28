
---
layout: post
title: MNIST Digits with CNN
---

# MNIST Data Set with a Convolutional Neural Network
###Georg Zhelev
####Code Adapted from: Kaggle 

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.image as mpimg
import seaborn as sns
%matplotlib inline

np.random.seed(2)

from sklearn.model_selection import train_test_split
from sklearn.metrics import confusion_matrix
import itertools

from keras.utils.np_utils import to_categorical # convert to one-hot-encoding
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten, Conv2D, MaxPool2D
from keras.optimizers import RMSprop
from keras.preprocessing.image import ImageDataGenerator
from keras.callbacks import ReduceLROnPlateau


sns.set(style='white', context='notebook', palette='deep')
```

# 2. Data preparation
## 2.1 Load data


```python
from google.colab import drive

drive.mount('/content/drive')
```

    Drive already mounted at /content/drive; to attempt to forcibly remount, call drive.mount("/content/drive", force_remount=True).
    


```python
# Load the data
train = pd.read_csv("/content/drive/MyDrive/Colab Notebooks/Econ/Digits/train.csv")
test = pd.read_csv("/content/drive/MyDrive/Colab Notebooks/Econ/Digits/test.csv")
```

### Observe Data


```python
train.head()
```






```python
train.shape
```




    (42000, 785)




```python
test.shape
```




    (28000, 784)




```python
np.add(train.shape,test.shape)[0]
```




    70000



### Extract Labels


```python
Y_train = train["label"]
print(Y_train.shape)
Y_train.head(10)
```

    (42000,)
    




    0    1
    1    0
    2    1
    3    4
    4    0
    5    0
    6    7
    7    3
    8    5
    9    3
    Name: label, dtype: int64



### Extract Features


```python
# Drop 'label' column
X_train = train.drop(labels = ["label"],axis = 1) 
X_train.shape

```




    (42000, 784)



### Remove Big Data set from Disk


```python
# free some space
del train 

```

### Graph Distribution of Labels


```python

g = sns.countplot(Y_train)

```

    /usr/local/lib/python3.6/dist-packages/seaborn/_decorators.py:43: FutureWarning: Pass the following variable as a keyword arg: x. From version 0.12, the only valid positional argument will be `data`, and passing other arguments without an explicit keyword will result in an error or misinterpretation.
      FutureWarning
    


![png](https://gzhelev2020.github.io/images/notebook_16_1.png)


### Show Distribution of Lables



```

# Als Code formatiert

```






```python

Y_train.value_counts()
```




    1    4684
    7    4401
    3    4351
    9    4188
    2    4177
    6    4137
    0    4132
    4    4072
    8    4063
    5    3795
    Name: label, dtype: int64



## 2.2 Check for null and missing values


```python
X_train.isnull().any().describe()
```




    count       784
    unique        1
    top       False
    freq        784
    dtype: object




```python
test.isnull().any().describe()
```




    count       784
    unique        1
    top       False
    freq        784
    dtype: object



## 2.3 Normalization


```python
X_train.describe()
```





```python
# # Normalize the data
# X_train = X_train / 255.0
# test = test / 255.0
```


```python
X_train.shape
```




    (42000, 784)




```python
Y_train.shape
```




    (42000,)




```python
test.shape
```




    (28000, 784)




```python
X_train.describe()
```



## 2.3 Reshape


```python
# Reshape image in 3 dimensions (height = 28px, width = 28px , canal = 1)
X_train = X_train.values.reshape(-1,28,28,1)
test = test.values.reshape(-1,28,28,1)
```


```python
print(X_train.shape)

print(test.shape)
```

    (42000, 28, 28, 1)
    (28000, 28, 28, 1)
    

## 2.5 Label encoding


```python
Y_train.shape
```




    (42000,)




```python
Y_train[0]
```




    1




```python
# Encode labels to one hot vectors (ex : 2 -> [0,0,1,0,0,0,0,0,0,0])
Y_train = to_categorical(Y_train, num_classes = 10)
```


```python
Y_train[0]
```




    array([0., 1., 0., 0., 0., 0., 0., 0., 0., 0.], dtype=float32)




```python
Y_train.shape
```




    (42000, 10)



## 2.6 Split training and valdiation set 


```python
# Set the random seed
random_seed = 2
```


```python
# Split the train and the validation set for the fitting
X_train, X_val, Y_train, Y_val = train_test_split(X_train, Y_train, test_size = 0.1, random_state=random_seed)
```

We can get a better sense for one of these examples by visualising the image and looking at the label.


```python
# Some examples
g = plt.imshow(X_train[0][:,:,0])
```


![png](notebook_files/notebook_42_0.png)


# 3. CNN
## 3.1 Define the model


```python
# Set the CNN model 

model = Sequential()

model.add(Conv2D(filters = 32, kernel_size = (5,5),padding = 'Same', 
                 activation ='relu', input_shape = (28,28,1)))
model.add(MaxPool2D(pool_size=(2,2)))

model.add(Flatten())
model.add(Dense(64, activation = "relu"))
model.add(Dropout(0.5))
model.add(Dense(10, activation = "softmax"))
```


```python


model.summary()
```

    Model: "sequential_16"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    conv2d_26 (Conv2D)           (None, 28, 28, 32)        832       
    _________________________________________________________________
    max_pooling2d_21 (MaxPooling (None, 14, 14, 32)        0         
    _________________________________________________________________
    flatten_16 (Flatten)         (None, 6272)              0         
    _________________________________________________________________
    dense_32 (Dense)             (None, 64)                401472    
    _________________________________________________________________
    dropout_25 (Dropout)         (None, 64)                0         
    _________________________________________________________________
    dense_33 (Dense)             (None, 10)                650       
    =================================================================
    Total params: 402,954
    Trainable params: 402,954
    Non-trainable params: 0
    _________________________________________________________________
    


```python
# Define the optimizer
#optimizer = RMSprop(lr=0.001, rho=0.9, epsilon=1e-08, decay=0.0)

from keras.optimizers import Adam
optimizer = Adam(learning_rate=0.01)

```


```python
# Compile the model
model.compile(optimizer = optimizer , loss = "categorical_crossentropy", metrics=["accuracy"])
```


```python
epochs = 5 # Turn epochs to 30 to get 0.9967 accuracy
#only one epoch for fast run time
batch_size = 86
```


```python
# Fit without data augmentation
history = model.fit(X_train, Y_train, batch_size = batch_size, epochs = epochs, 
          validation_data = (X_val, Y_val), verbose = 1)
```

    Epoch 1/5
    440/440 [==============================] - 2s 4ms/step - loss: 9.6224 - accuracy: 0.6398 - val_loss: 0.3336 - val_accuracy: 0.8955
    Epoch 2/5
    440/440 [==============================] - 2s 4ms/step - loss: 0.4818 - accuracy: 0.8571 - val_loss: 0.2993 - val_accuracy: 0.9150
    Epoch 3/5
    440/440 [==============================] - 2s 4ms/step - loss: 0.4275 - accuracy: 0.8732 - val_loss: 0.2401 - val_accuracy: 0.9321
    Epoch 4/5
    440/440 [==============================] - 2s 4ms/step - loss: 0.4129 - accuracy: 0.8758 - val_loss: 0.2642 - val_accuracy: 0.9324
    Epoch 5/5
    440/440 [==============================] - 2s 4ms/step - loss: 0.4240 - accuracy: 0.8789 - val_loss: 0.2800 - val_accuracy: 0.9171
    

#### we get a validation accuracy of around 92 %

# 4. Evaluate the model
## 4.1 Training and validation curves


```python
# Plot the loss and accuracy curves for training and validation 
fig, ax = plt.subplots(2,1)
ax[0].plot(history.history['loss'], color='b', label="Training loss")
ax[0].plot(history.history['val_loss'], color='r', label="validation loss")
legend = ax[0].legend(loc='best', shadow=True)

ax[1].plot(history.history['accuracy'], color='b', label="Training accuracy")
ax[1].plot(history.history['val_accuracy'], color='r',label="Validation accuracy")
legend = ax[1].legend(loc='best', shadow=True)
```


![png](notebook_files/notebook_52_0.png)


## 4.2 Confusion matrix


```python
# Look at confusion matrix 

def plot_confusion_matrix(cm, classes,
                          normalize=False,
                          title='Confusion matrix',
                          cmap=plt.cm.Blues):
    """
    This function prints and plots the confusion matrix.
    Normalization can be applied by setting `normalize=True`.
    """
    plt.imshow(cm, interpolation='nearest', cmap=cmap)
    plt.title(title)
    plt.colorbar()
    tick_marks = np.arange(len(classes))
    plt.xticks(tick_marks, classes, rotation=45)
    plt.yticks(tick_marks, classes)

    if normalize:
        cm = cm.astype('float') / cm.sum(axis=1)[:, np.newaxis]

    thresh = cm.max() / 2.
    for i, j in itertools.product(range(cm.shape[0]), range(cm.shape[1])):
        plt.text(j, i, cm[i, j],
                 horizontalalignment="center",
                 color="white" if cm[i, j] > thresh else "black")

    plt.tight_layout()
    plt.ylabel('True label')
    plt.xlabel('Predicted label')

# Predict the values from the validation dataset
Y_pred = model.predict(X_val)
# Convert predictions classes to one hot vectors 
Y_pred_classes = np.argmax(Y_pred,axis = 1) 
# Convert validation observations to one hot vectors
Y_true = np.argmax(Y_val,axis = 1) 
# compute the confusion matrix
confusion_mtx = confusion_matrix(Y_true, Y_pred_classes) 
# plot the confusion matrix
plot_confusion_matrix(confusion_mtx, classes = range(10)) 
```


![png](notebook_files/notebook_54_0.png)


Here we can see that our CNN performs very well on all digits with few errors considering the size of the validation set (4 200 images).

However, it seems that our CNN has some little troubles with the 4 digits, hey are misclassified as 9. Sometime it is very difficult to catch the difference between 4 and 9 when curves are smooth.

Let's investigate for errors. 

I want to see the most important errors . For that purpose i need to get the difference between the probabilities of real value and the predicted ones in the results.


```python
# Display some error results 

# Errors are difference between predicted labels and true labels
errors = (Y_pred_classes - Y_true != 0)

Y_pred_classes_errors = Y_pred_classes[errors]
Y_pred_errors = Y_pred[errors]
Y_true_errors = Y_true[errors]
X_val_errors = X_val[errors]

def display_errors(errors_index,img_errors,pred_errors, obs_errors):
    """ This function shows 6 images with their predicted and real labels"""
    n = 0
    nrows = 2
    ncols = 3
    fig, ax = plt.subplots(nrows,ncols,sharex=True,sharey=True)
    for row in range(nrows):
        for col in range(ncols):
            error = errors_index[n]
            ax[row,col].imshow((img_errors[error]).reshape((28,28)))
            ax[row,col].set_title("Predicted label :{}\nTrue label :{}".format(pred_errors[error],obs_errors[error]))
            n += 1

# Probabilities of the wrong predicted numbers
Y_pred_errors_prob = np.max(Y_pred_errors,axis = 1)

# Predicted probabilities of the true values in the error set
true_prob_errors = np.diagonal(np.take(Y_pred_errors, Y_true_errors, axis=1))

# Difference between the probability of the predicted label and the true label
delta_pred_true_errors = Y_pred_errors_prob - true_prob_errors

# Sorted list of the delta prob errors
sorted_dela_errors = np.argsort(delta_pred_true_errors)

# Top 6 errors 
most_important_errors = sorted_dela_errors[-6:]

# Show the top 6 errors
display_errors(most_important_errors, X_val_errors, Y_pred_classes_errors, Y_true_errors)
```


![png](notebook_files/notebook_57_0.png)


The most important errors are also the most intrigous. 

For those six case, the model is not ridiculous. Some of these errors can also be made by humans, especially for one the 9 that is very close to a 4. The last 9 is also very misleading, it seems for me that is a 0.


```python
# predict results
results = model.predict(test)

# select the index with the maximum probability
results = np.argmax(results,axis = 1)

results = pd.Series(results,name="Label")
```


```python
submission = pd.concat([pd.Series(range(1,28001),name = "ImageId"),results],axis = 1)

submission.to_csv("cnn_mnist_datagen.csv",index=False)
from google.colab import files

files.download("cnn_mnist_datagen.csv")
```


    <IPython.core.display.Javascript object>



    <IPython.core.display.Javascript object>
