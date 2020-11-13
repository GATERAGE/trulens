# Welcome to Netlens!

Library containing attribution and interpretation methods for deep nets.


# Overview

## Backends

The `lens-api` library supports several common machine learning libraries, including Keras, Pytorch, and TensorFlow.

In order to set the backend to the backend of your choice, use the `NETLENS_BACKEND` flag, e.g., to use the Keras backend, the following code could be used before NetLens imports:

```python
import os
os.environ['NETLENS_BACKEND'] = 'keras'
```

## Attributions

### Model Wrappers

In order to support a wide variety of backends with different interfaces for their respective models, NetLens uses its own `Model` class which provides a general model interface to simplify the implementation of the API functions.
A model wrapper class exists for backend's model that converts a model in the respective backend's format to the general NetLens `Model` interface.
The wrappers are found in the `models` module, and any model defined using Keras, Pytorch, or Tensorflow should be wrapped with the appropriate wrapper before being used with the other API functions that require a model -- *all other NetLens functionalities expect models to be an instance of `netlens.models.Model`.*

For example,

```python
wrapped_model = KerasModel(model_defined_via_keras)
```

### Attribution Methods

Attribution methods, in the most general sense, allow us to quantify the contribution of particular variables in a model towards a particular behavior of the model.
In many cases, for example, this may simply measure the effect each input variable has on the output of the network.

Attribution methods extend the `AttributionMethod` class, and many concrete instances are found in the `attribution` module.

Once an attribution method has been instantiated, its main function is its `attributions` method, which takes an `np.Array` of batched instances, where each instance matches the shape of the *input* to the model the attribution method was instantiated with.

See the *method comparison* demo for further information on the different types of attribution methods, their uses, and their relationships with one another.

### Slices, Quantities, and Distributions

In order to obtain a high degree of flexibility in the types of attributions that can be produced, we implement *Internal Influence*, which is parameterized by a *slice*, *quantity of interest*, and *distribution of interest*, explained below.

The *slice* essentially defines a layer to use for internal attributions.
The slice for the `InternalInfluence` method can be specified by an instance of the `Slice` class in the `slices` module.
A `Slice` object specifies two layers: (1) the layer of the variables that we are calculating attribution *for* (e.g., the input layer), and (2) the layer whose output defines our *quantity of interest* (e.g., the output layer, see below for more on quantities of interest).

The *quantity of interest* (QoI) essentially defines the model behavior we would like to explain using attributions.
The QoI is a function of the model's output at some layer.
For example, it may select the confidence score for a particular class.
In its most general form, the QoI can be pecified by an implementation of the `QoI` class in the `quantities` module.
Several common default implementations are provided in this module as well.

The *distribution of interest* (DoI) essentially specifies for which points surrounding each instance the calculated attribution should be valid.
The distribution can be specified via an implementation of the `DoI` class in the `distributions` module, which is a function taking an input instance and producing a list of input points to aggregate attribution over.
A few common default distributions implementing the `DoI` class can be found in the `distributions` module. 

See the *parameterization demo* for further explanations of the purpose of these parameters and examples of their usage.

## Visualizations

In order to interpret the attributions produced by an `AttributionMethod`, a few useful visualizers are provided in the `visualizations` module.
While the interface of each visualizer varies slightly, in general, the visualizers are a function taking an `np.Array` representing the attributions returned from an `AttributionMethod` and producing an image that can be used to interpret the attributions.