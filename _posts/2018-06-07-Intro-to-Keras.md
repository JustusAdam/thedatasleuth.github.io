---
layout: post
title: The Writing's on the Wall, Part 1 (Intro to Keras)
categories: Neural-Network Keras Kaggle
highlight_code: true
published: False
---

*Visions of the future are never completely true, but with the right key, some can be more truthful than others.  Here's an introduction to the neural network library, Keras, which I used to predict handwritten numbers.*

<!--more-->

>*"Oneiroi are beyond our unravelling --who can be sure what tale they tell? Not all that men look for comes to pass. Two gates there are that give passage to fleeting Oneiroi; one is made of horn, one of ivory. The Oneiroi that pass through sawn ivory are deceitful, bearing a message that will not be fulfilled; those that come out through polished horn have truth behind them, to be accomplished for men who see them." Homer, Odyssey 19. 562 ff (Shewring translation).*

For thousands of years, we've tried to prophesize what the future will hold.  We hope that whatever visions come to us are truthful, but those are only confirmed when the significance of the event has already passed.  As in the passage above, Odysseus' men were similarly concerned with false visions of the future.  

As artificial intelligence continues to be researched and integrated into our daily lives, we should be as concerned as Odysseus' men with which visions of the future are true and which are false.

A neural network model approximates the way our actual brains work.  Information passes from one layer of neurons to the next, and the information gained from those interactions informs the final prediction.  Keras is a deep neural network library that makes use of TensorFlow on the backend to allow for easier and faster implementation.  Credited as its primary author, François Chollet, a Google enigeer, explained that the origin of the Keras name comes from the Greek word κέρας for horn.  It references imagery from the Odyssey where men are deceived by dream spirits who arrive through gates of ivory but are led to truth by spirits who arrive through gates of horn.  

Neural networks, in a single line, attempt to iteratively train a set (or sets) of weights that, when used together, return the most accurate predictions for a set of inputs. Just like many of our past models, the model is trained using a loss function, which our model will attempt to minimize over iterations. Remember that a loss function is some function that takes in our predictions and the actual values and returns some sort of aggregate value that shows how accurate (or not) we were.

Neural networks do this by establishing sets of neurons (known as hidden layers) that take in some sort of input(s), apply a weight, and pass that output onward. As we feed more data into the network, it adjusts those weights based on the output of the loss function, until we have highly trained and specific weights.







I applied Keras to the MSINT handwriting database made available by Kaggle to predict numbers based on their handwritten equivalent.  Below is a walkthrough of the steps I took to make predictions.


## Prepping the Canvas

To start, I read the training set into pandas, identifying 42,000 rows and 785 columns of data, inclusive of the target column, 'label.'  With the exception of the target column, all of the rest of the columns were pixel features of the images.  

Should I maybe talk about how to convert images to pixels here?


Because Keras requires a numpy array (as opposed to a pandas dataframe), I transformed the X set into a numpy matrix and then normalized the dataset by dividing each value by the maximum number of pixels (255).  I also transformed the target (y) values into a one-hot encoded matrix given that this dataset is a multi-classification problem.

Before feeding the data into the neural network model, I train-test-split my data for training purposes.

## Adding Nuance

I initialized the model by making it Sequential() and adding layers to it.  Within a neural network, there are different layers that can be added to improve model performance.  First, I added a dense layer which connected all of the neurons.  Then, I added a "hidden" dropout layer.  A hidden layer does not mean that it is not present or isn't working - it just means that it is not the initial input layer or the final output layer, as seen in the image below.

![neuralnet.png](/static/img/neuralnet.png)

A neural network can have any number of hidden layers with any number of neurons.  However, within each layer, each neuron will transform the data in different (and difficult to quantify) ways based on the assigned and change in weights within that neuron.

I decided to set my dropout layer to 0.5, which means that 50% of the neurons would be randomly "turned off" from training.  I did this to prevent the model from being overfit to the training set and to allow for some flexibility when faced with new data.

```
model = Sequential()
model.add(Dense(X_train.shape[1], input_shape=(784,), activation='relu'))
model.add(Dropout(.5))
model.add(Dense(y_train.shape[1], activation='softmax'))
```

Next, I activated the training set with 'relu' (Rectified Linear Unit) which only activates neurons whose output will be positive.  For the target, I activated with 'softmax' to essentially normalize the inputs within a scale of 0 to 1 to create a probability for an input falling within one of the target classes.

## Knowing Your Audience

The next step to modeling a neural network is to compile the model.  I chose the 'adam' optimizer.  (NEED TO EXPLAIN MORE HERE)  

For this particular muticlassification problem, I set the loss metric to 'categorical_crossentropy.'

## Failing Better

When training the model, a user can specify how many epochs used to train the model.  An epoch is a "roundtrip" for information to be passed forwards and backwards through the network.  During forward propagation, data is passed from the initial input layer through the hidden layers and finally to the output layer during which weights, biases, and activation functions are applied to the neurons.  During back propagation, the data is then passed back from the output towards the input layer, measuring the loss produced from the weights assigned to the neurons and, using gradient descent, changes the weights accordingly in order to make more accurate predictions over a user-defined number of iterations.  The amount of epochs to use during training can be identified when either the accuracy or the loss of the predictions stabilizes at an acceptable value.

To illustrate this point, I plotted a summary chart of the four runs I conducted, tweaking the number of epochs and the dropout percentage with each new run.  

![keras-summary.png](/static/img/keras-summary.png)

The top left plot shows the loss from both the training and the cross-validation (test) during which I trained 50 epochs without a dropout layer.  Notice how the training loss dramatically reduces within the first ten epochs but continues to improve only slightly as the training continues to the 50th epoch.  The loss from the test set, however, increases with the number of epochs with a wider amount of variance - a clear sign of overfitting.  The top right plot shows a similar training run but with an added dropout layer of 50%.  Both the training and test sets appear to be more stable and the loss from the test set is reduced.  In my third run, illustrated by the bottom left plot, I increased the dropout percentage to 75% while still running 50 epochs.  Again, the test loss amount is reduced.  Finally, in the fourth run, noticing that in the last three runs there was a shift in performance between the tenth and twentieth epoch, I decided to set my training epochs to 15 while keeping the dropout rate at 75%.  In this run, the test set loss hued closely to the loss of the training set.  Now I had a model that was trained up on the dataset while remaining flexible enough to predict on unseen data.

## Next Steps