---
title: Common Deep Neural Network Components and Techniques
date: 2017-09-13 13:21:00
tags:
---

<a id="org44ef66e"></a>

# Deep Convolutional Neural Network Components and Techniques

<a id="org9482df0"></a>

There are three main components of neural networks; score function which maps individual features to classes, loss function which quantifies agreement between prediction of class and the ground-truth label and, the final third component is the formulation and solution of a problem through optimisation i.e., minimising loss function with respect to parameters of score function.


<a id="org094ecf2"></a>

## Score Function

Given set of features in the form of pixels colours and containing them images $\boldsymbol{x}\_{i} \in \mathcal{R}^{D}$, where each i<sup>th</sup> image $\boldsymbol{x}\_{i}$ with $i \in \{1, \ldots, N\}$ has also associated class to it $y\_{i}$ and $y\_{i} \in \{1, \ldots, M\}$, i.e., we have $N$ labeled images each having $D$ dimensions, and all images are associated with $M$ distinct classes. A score function going from each image's pixels to a number of class predictions is defined as $\boldsymbol{f}: \mathcal{R}^{D} \to \mathcal{R}^{M}$. The elementary example of such function is a $\boldsymbol{f}(\boldsymbol{x}\_{i}; \boldsymbol{W}, \boldsymbol{b}) = \boldsymbol{W} \boldsymbol{x}\_{i} + \boldsymbol{b}$, where $\boldsymbol{W} \in \mathcal{R}^{M \times D}$ and $\boldsymbol{b} \in \mathcal{R}^{M}$. Thus, single matrix product, simultaneously evaluates $M$ classifiers, where each classifier is a row in matrix $\boldsymbol{W}$. If we also attach an activation function to a single row multiplication, we effectively get what is called linear discriminant or perceptron $h(\boldsymbol{w};\boldsymbol{x}) = \sigma\left(\boldsymbol{w}^{T}\boldsymbol{x}\right) = \sigma\left( w\_{0} + \sum\_{i=1}^{m} w\_{i}x\_{i} \right)$.


<a id="org13d0f42"></a>

## Loss function

Once we have a scoring function, we need to have some way of measuring error of it's output predictions. Assuming $\sigma(z) = z$, the exemplary instance of such loss function would then take a form of $E(\boldsymbol{w}) = \sum\_{i=1}^{m} (y\_{i} - h(\boldsymbol{w};\boldsymbol{x}\_{i}))^{2} = \sum\_{i=1}^{m} (y\_{i} - \boldsymbol{w}^{T}\boldsymbol{x})^{2}$. The more practical example would be to consider Multiclass Support Vector Machine (SVM). The main idea behind SVM is to make a loss function require that correct class gives higher score than than incorrect one, by a fixed margin $\Delta$. Given a score for k<sup>th</sup> class, $\boldsymbol{s}\_{k} = \boldsymbol{f}(\boldsymbol{x}\_{i}; \boldsymbol{W}, \boldsymbol{b})\_{k}$. The multiclass SVM for a single image takes the following form $L\_{i} = \sum\_{k \neq y\_{i}} \max (0, \boldsymbol{s}\_{k} - \boldsymbol{s}\_{y\_{i}} + \Delta)$, and if we consider only linear score functions, this would be $L\_{i} = \sum\_{k \neq y\_{i}} \max (0, \boldsymbol{w}\_{k}^{T} \boldsymbol{x}\_{i} - \boldsymbol{w}\_{y\_{i}}^{T} \boldsymbol{x}\_{i} + \Delta)$. Thus we will accumulate loss, if the score for correct class is not larger than for the incorrect one, by at least $\Delta$.


<a id="org5eaf41b"></a>

## Gradient Descent and Backpropagation

If the parameters of $\boldsymbol{W}$ are set so that, most of the predictions for all $\boldsymbol{x}\_{i}$ examples agree with ground truths $y\_{i}$, the accumulated loss would be very low. Thus our next task is to find the optimal parameters of $\boldsymbol{W}$.

We can define gradient descent by basing on the observation that if multi-variable loss function $E(\boldsymbol{w})$ is differentiable in a neighbourhood of a point $\boldsymbol{w}\_{t}$, then the value of a loss function will decrease fastest if we move in a direction of the negative gradient of the $E$ at $\boldsymbol{w}\_{t}$, i.e., $-\nabla E\left(\boldsymbol{w}\_{t}\right)$. Thus, in order to update weights we can use the following result,

$$
\boldsymbol{w}\_{t+1} = \boldsymbol{w}\_{t} - \eta\frac{\partial E(\boldsymbol{w})}{\partial \boldsymbol{w}}\bigg|\_{\boldsymbol{w}\_{t}}
$$

this equation can be also obtained by rearranging derivative of a function, $f(x + h) = f(x) - h \frac{\partial f }{\partial x}$. The step size $\eta$ also called learning rate is one of the most important hyperparameters in training. There are various tuning strategies devised for it, which we will describe in further sections of this report. Note also, that currently there are kinks in our loss function due to max operation. This seemingly makes it non-differentiable, however we can still use existing subgradients.

The loss functions we concern ourselves with, are defined over high-dimensional spaces i.e., the matrix $\boldsymbol{W}$ of weights corresponds to single point in that space, thus we need to realise efficient strategies for computing derivatives in this large space. Therefore, we can choose to take only a subset of the training samples and form the batches for evaluation of a loss function as it would not be possible to compute it using entire dataset. The examples from a dataset are typically correlated thus it is not a significant obstacle having to compute gradient against only a batch. The extreme case of using one single sample for computation is called stochatic gradient descent (SGD). Because we can use parallelised computation it is much less computationally efficient to compute using just one example as in SGD. Altogether, although batch size is hyperparameter it is not common to cross-validate it and the main constraint here is the capacity of GPU's memory.


<a id="org89af85f"></a>

## Deep Convolutional Neural Networks

Deep convolutional neural networks are formed with three types of layers: convolutional layer, pooling layer and fully-connected layer which has been described in section [1](#org9482df0). The main drawbacks of using fully-connected layers on their own is that they do not scale well and are prone to overfitting. Extending neural network architecture with convolutional layers mitigates scalability issue, by extracting features from input images and thus providing basis to the full-connected layers which are trained with unique set of these discerned features. In the following two sections, we detail the additional two layers types. 


<a id="org0d91e73"></a>

### Convolution

Convolutional layers are formed with set of dimensionally small parametrised filters. During forward pass each filter is being moved across width and height of input image volume. Each such operation generates 2 dimensional activation map, representing response of filter of which parameters are being trained during backward pass, in order to activate on recognition of specific visual feature. The idea then is to have multiple layers of filtering volumes, where each layer learns features of increasing complexity. In our demonstrative setting, the outer layer learns generic features and patterns such as lines and edges. The next layer learns compositions of these i.e., corners and curves; going deeper in hierarchy, we learn simple shapes and so forth. The lowest-level convolutional layers provide activation maps for each instance of an object.


<a id="org083de9f"></a>

### Pooling

In order to reduce the amount of involved parameters and thus also computational cost, we insert pooling layers in-between convolutional layers. It's task is to reduce spatial size of representations by downsampling the input volumes.
