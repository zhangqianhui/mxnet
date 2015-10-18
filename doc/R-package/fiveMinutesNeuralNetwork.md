Neural Network with MXNet in Five Minutes
=============================================

This is the first tutorial for new users of the R package `mxnet`. You will learn to construct a neural network to do regression in 5 minutes. 

We will show you how to do classification and regression tasks respectively. The data we use comes from the package `mlbench`.

## Classification

First of all, let us load in the data and preprocess it:


```r
require(mlbench)
```

```
## Loading required package: mlbench
```

```r
require(mxnet)
```

```
## Loading required package: mxnet
## Loading required package: methods
```

```r
data(Sonar, package="mlbench")

Sonar[,61] = as.numeric(Sonar[,61])-1
train.ind = c(1:50, 100:150)
train.x = data.matrix(Sonar[train.ind, 1:60])
train.y = Sonar[train.ind, 61]
test.x = data.matrix(Sonar[-train.ind, 1:60])
test.y = Sonar[-train.ind, 61]
```

The next step is to define the structure of the neural network.


```r
# Define the input data
data <- mx.symbol.Variable("data")
# A fully connected hidden layer 
# data: input source
# name: fc1
# num_hidden: number of neurons in this hidden layer
fc1 <- mx.symbol.FullyConnected(data, name="fc1", num_hidden=20)

# An activation function
# fc1: input source
# name: relu1
# act_type: type for the activation function
act1 <- mx.symbol.Activation(fc1, name="tanh1", act_type="tanh")
fc2 <- mx.symbol.FullyConnected(act1, name="fc2", num_hidden=2)

# Softmax function for the output layer
softmax <- mx.symbol.Softmax(fc2, name="sm")
```

According to the comments in the code, you can see the meaning of each function and its arguments. They can be easily modified according to your need.

Before we start to train the model, we can specify where to run our program:


```r
device.cpu = mx.cpu()
```

Here we choose to run it on CPU.

After the network configuration, we can start the training process:


```r
mx.set.seed(0)
model <- mx.model.FeedForward.create(softmax, X=train.x, y=train.y,
                                     ctx=device.cpu, num.round=20, array.batch.size=15,
                                     learning.rate=0.07, momentum=0.9, eval.metric=mx.metric.accuracy,
                                     epoch.end.callback=mx.callback.log.train.metric(100))
```

```
## Start training with 1 devices
## [1] Train-accuracy=0.5
## [2] Train-accuracy=0.514285714285714
## [3] Train-accuracy=0.514285714285714
## [4] Train-accuracy=0.514285714285714
## [5] Train-accuracy=0.514285714285714
## [6] Train-accuracy=0.609523809523809
## [7] Train-accuracy=0.676190476190476
## [8] Train-accuracy=0.695238095238095
## [9] Train-accuracy=0.723809523809524
## [10] Train-accuracy=0.780952380952381
## [11] Train-accuracy=0.8
## [12] Train-accuracy=0.761904761904762
## [13] Train-accuracy=0.742857142857143
## [14] Train-accuracy=0.761904761904762
## [15] Train-accuracy=0.847619047619047
## [16] Train-accuracy=0.857142857142857
## [17] Train-accuracy=0.857142857142857
## [18] Train-accuracy=0.828571428571429
## [19] Train-accuracy=0.838095238095238
## [20] Train-accuracy=0.857142857142857
```

Note that `mx.set.seed` is the correct function to control the random process in `mxnet`. You can see the accuracy in each round during training. It is also easy to make prediction and evaluate


```r
preds = predict(model, test.x)
pred.label = max.col(preds)-1
table(pred.label, test.y)
```

```
##           test.y
## pred.label  0  1
##          0 24 14
##          1 36 33
```

## Regression

Again, let us preprocess the data first.


```r
data(BostonHousing, package="mlbench")

train.ind = seq(1, 506, 3)
train.x = data.matrix(BostonHousing[train.ind, -14])
train.y = BostonHousing[train.ind, 14]
test.x = data.matrix(BostonHousing[-train.ind, -14])
test.y = BostonHousing[-train.ind, 14]
```

We can configure a similar network as what we have done above. The only difference is in the output activation:


```r
# Define the input data
data <- mx.symbol.Variable("data")
# A fully connected hidden layer 
# data: input source
# name: fc1
# num_hidden: number of neurons in this hidden layer
fc1 <- mx.symbol.FullyConnected(data, name="fc1", num_hidden=20)

# An activation function
# fc1: input source
# name: relu1
# act_type: type for the activation function
act1 <- mx.symbol.Activation(fc1, name="tanh1", act_type="tanh")
fc2 <- mx.symbol.FullyConnected(act1, name="fc2", num_hidden=1)

# Softmax function for the output layer
lro <- mx.symbol.LinearRegressionOutput(fc2, name="lro")
```

What we changed is mainly the last function, this enables the new network to optimize for squared loss. We can now train on this simple data set.


```r
mx.set.seed(0)
model <- mx.model.FeedForward.create(lro, X=train.x, y=train.y,
                                     ctx=device.cpu, num.round=5, array.batch.size=10,
                                     learning.rate=0.1, momentum=0.9, eval.metric=mx.metric.rmse,
                                     epoch.end.callback=mx.callback.log.train.metric(100))
```

```
## Start training with 1 devices
## [1] Train-rmse=20.8877275599495
## [2] Train-rmse=12.8786644532322
## [3] Train-rmse=10.3635559222185
## [4] Train-rmse=10.5605206622052
## [5] Train-rmse=10.2502398389275
```

It is also easy to make prediction and evaluate


```r
preds = predict(model, test.x)
sqrt(mean((preds-test.y)^2))
```

```
## [1] 9.49181
```

Currently we have two pre-defined metrics "accuracy" and "rmse". One might wonder how to customize the evaluation metric. `mxnet` provides the interface for users to define their own metric of interests:


```r
demo.metric.mae <- mx.metric.custom("mae", function(label, pred) {
  res <- mean(abs(label-pred))
  return(res)
})
```

This is an example for mean absolute error. We can simply plug it in the training function:


```r
mx.set.seed(0)
model <- mx.model.FeedForward.create(lro, X=train.x, y=train.y,
                                     ctx=device.cpu, num.round=5, array.batch.size=10,
                                     learning.rate=0.1, momentum=0.9, eval.metric=demo.metric.mae,
                                     epoch.end.callback=mx.callback.log.train.metric(100))
```

```
## Start training with 1 devices
## [1] Train-mae=19.3546375619262
## [2] Train-mae=10.5938747770646
## [3] Train-mae=8.51244305161869
## [4] Train-mae=8.41277845326592
## [5] Train-mae=8.23570416674895
```

Congratulations! Now you have learnt the basic for using `mxnet`.

