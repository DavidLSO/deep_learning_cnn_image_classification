
# Image Classification
In this project, you'll classify images from the [CIFAR-10 dataset](https://www.cs.toronto.edu/~kriz/cifar.html).  The dataset consists of airplanes, dogs, cats, and other objects. You'll preprocess the images, then train a convolutional neural network on all the samples. The images need to be normalized and the labels need to be one-hot encoded.  You'll get to apply what you learned and build a convolutional, max pooling, dropout, and fully connected layers.  At the end, you'll get to see your neural network's predictions on the sample images.
## Get the Data
Run the following cell to download the [CIFAR-10 dataset for python](https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz).


```python
"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""
from urllib.request import urlretrieve
from os.path import isfile, isdir
from tqdm import tqdm
import problem_unittests as tests
import tarfile

cifar10_dataset_folder_path = 'cifar-10-batches-py'

# Use Floyd's cifar-10 dataset if present
floyd_cifar10_location = '/cifar/cifar-10-python.tar.gz'
if isfile(floyd_cifar10_location):
    tar_gz_path = floyd_cifar10_location
else:
    tar_gz_path = 'cifar-10-python.tar.gz'


class DLProgress(tqdm):
    last_block = 0

    def hook(self, block_num=1, block_size=1, total_size=None):
        self.total = total_size
        self.update((block_num - self.last_block) * block_size)
        self.last_block = block_num


if not isfile(tar_gz_path):
    with DLProgress(unit='B', unit_scale=True, miniters=1, desc='CIFAR-10 Dataset') as pbar:
        urlretrieve(
            'https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz',
            tar_gz_path,
            pbar.hook)

if not isdir(cifar10_dataset_folder_path):
    with tarfile.open(tar_gz_path) as tar:
        tar.extractall()
        tar.close()

tests.test_folder_path(cifar10_dataset_folder_path)

```

    All files found!
    

## Explore the Data
The dataset is broken into batches to prevent your machine from running out of memory.  The CIFAR-10 dataset consists of 5 batches, named `data_batch_1`, `data_batch_2`, etc.. Each batch contains the labels and images that are one of the following:
* airplane
* automobile
* bird
* cat
* deer
* dog
* frog
* horse
* ship
* truck

Understanding a dataset is part of making predictions on the data.  Play around with the code cell below by changing the `batch_id` and `sample_id`. The `batch_id` is the id for a batch (1-5). The `sample_id` is the id for a image and label pair in the batch.

Ask yourself "What are all possible labels?", "What is the range of values for the image data?", "Are the labels in order or random?".  Answers to questions like these will help you preprocess the data and end up with better predictions.


```python
%matplotlib inline
%config InlineBackend.figure_format = 'retina'
import helper
import numpy as np

# Explore the dataset
batch_id = 1
sample_id = 5
helper.display_stats(cifar10_dataset_folder_path, batch_id, sample_id)

```

    
    Stats of batch 1:
    Samples: 10000
    Label Counts: {0: 1005, 1: 974, 2: 1032, 3: 1016, 4: 999, 5: 937, 6: 1030, 7: 1001, 8: 1025, 9: 981}
    First 20 Labels: [6, 9, 9, 4, 1, 1, 2, 7, 8, 3, 4, 7, 7, 2, 9, 9, 9, 3, 2, 6]
    
    Example of Image 5:
    Image - Min Value: 0 Max Value: 252
    Image - Shape: (32, 32, 3)
    Label - Label Id: 1 Name: automobile
    


![png](output_3_1.png)


## Implement Preprocess Functions
### Normalize
In the cell below, implement the `normalize` function to take in image data, `x`, and return it as a normalized Numpy array. The values should be in the range of 0 to 1, inclusive.  The return object should be the same shape as `x`.


```python
def normalize(x):
    """
    Normalize a list of sample image data in the range of 0 to 1
    : x: List of image data.  The image shape is (32, 32, 3)
    : return: Numpy array of normalize data
    """
    # TODO: Implement Function
    # ret = []
    # for image in x:
    #     ret.append(np.divide(image, 255.))
    # return np.asarray(ret) 
    return np.array((x - np.min(x)) / (np.max(x) - np.min(x)))
    # return np.asarray(ret) 


"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""
tests.test_normalize(normalize)

```

    Tests Passed
    

### One-hot encode
Just like the previous code cell, you'll be implementing a function for preprocessing.  This time, you'll implement the `one_hot_encode` function. The input, `x`, are a list of labels.  Implement the function to return the list of labels as One-Hot encoded Numpy array.  The possible values for labels are 0 to 9. The one-hot encoding function should return the same encoding for each value between each call to `one_hot_encode`.  Make sure to save the map of encodings outside the function.

Hint: Don't reinvent the wheel.


```python
def one_hot_encode(x):
    """
    One hot encode a list of sample labels. Return a one-hot encoded vector for each label.
    : x: List of sample Labels
    : return: Numpy array of one-hot encoded labels
    """
    # TODO: Implement Function
    one_hot = np.zeros(shape=(len(x), 10))
    for i in range(len(x)):
        for j in range(10):
            one_hot[i][j] = (j == x[i])
    return one_hot


"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""
tests.test_one_hot_encode(one_hot_encode)
```

    Tests Passed
    

### Randomize Data
As you saw from exploring the data above, the order of the samples are randomized.  It doesn't hurt to randomize it again, but you don't need to for this dataset.

## Preprocess all the data and save it
Running the code cell below will preprocess all the CIFAR-10 data and save it to file. The code below also uses 10% of the training data for validation.


```python
"""
DON'T MODIFY ANYTHING IN THIS CELL
"""
# Preprocess Training, Validation, and Testing Data
helper.preprocess_and_save_data(cifar10_dataset_folder_path, normalize, one_hot_encode)
```

# Check Point
This is your first checkpoint.  If you ever decide to come back to this notebook or have to restart the notebook, you can start from here.  The preprocessed data has been saved to disk.


```python
"""
DON'T MODIFY ANYTHING IN THIS CELL
"""
import pickle
import problem_unittests as tests
import helper

# Load the Preprocessed Validation data
valid_features, valid_labels = pickle.load(open('preprocess_validation.p', mode='rb'))
```

## Build the network
For the neural network, you'll build each layer into a function.  Most of the code you've seen has been outside of functions. To test your code more thoroughly, we require that you put each layer in a function.  This allows us to give you better feedback and test for simple mistakes using our unittests before you submit your project.

>**Note:** If you're finding it hard to dedicate enough time for this course each week, we've provided a small shortcut to this part of the project. In the next couple of problems, you'll have the option to use classes from the [TensorFlow Layers](https://www.tensorflow.org/api_docs/python/tf/layers) or [TensorFlow Layers (contrib)](https://www.tensorflow.org/api_guides/python/contrib.layers) packages to build each layer, except the layers you build in the "Convolutional and Max Pooling Layer" section.  TF Layers is similar to Keras's and TFLearn's abstraction to layers, so it's easy to pickup.

>However, if you would like to get the most out of this course, try to solve all the problems _without_ using anything from the TF Layers packages. You **can** still use classes from other packages that happen to have the same name as ones you find in TF Layers! For example, instead of using the TF Layers version of the `conv2d` class, [tf.layers.conv2d](https://www.tensorflow.org/api_docs/python/tf/layers/conv2d), you would want to use the TF Neural Network version of `conv2d`, [tf.nn.conv2d](https://www.tensorflow.org/api_docs/python/tf/nn/conv2d). 

Let's begin!

### Input
The neural network needs to read the image data, one-hot encoded labels, and dropout keep probability. Implement the following functions
* Implement `neural_net_image_input`
 * Return a [TF Placeholder](https://www.tensorflow.org/api_docs/python/tf/placeholder)
 * Set the shape using `image_shape` with batch size set to `None`.
 * Name the TensorFlow placeholder "x" using the TensorFlow `name` parameter in the [TF Placeholder](https://www.tensorflow.org/api_docs/python/tf/placeholder).
* Implement `neural_net_label_input`
 * Return a [TF Placeholder](https://www.tensorflow.org/api_docs/python/tf/placeholder)
 * Set the shape using `n_classes` with batch size set to `None`.
 * Name the TensorFlow placeholder "y" using the TensorFlow `name` parameter in the [TF Placeholder](https://www.tensorflow.org/api_docs/python/tf/placeholder).
* Implement `neural_net_keep_prob_input`
 * Return a [TF Placeholder](https://www.tensorflow.org/api_docs/python/tf/placeholder) for dropout keep probability.
 * Name the TensorFlow placeholder "keep_prob" using the TensorFlow `name` parameter in the [TF Placeholder](https://www.tensorflow.org/api_docs/python/tf/placeholder).

These names will be used at the end of the project to load your saved model.

Note: `None` for shapes in TensorFlow allow for a dynamic size.


```python
import tensorflow as tf


def neural_net_image_input(image_shape):
    """
    Return a Tensor for a batch of image input
    : image_shape: Shape of the images
    : return: Tensor for image input.
    """
    # TODO: Implement Function
    return tf.placeholder(tf.float32, shape=[None, *image_shape], name="x")


def neural_net_label_input(n_classes):
    """
    Return a Tensor for a batch of label input
    : n_classes: Number of classes
    : return: Tensor for label input.
    """
    # TODO: Implement Function
    return tf.placeholder(tf.float32, shape=[None, n_classes], name='y')


def neural_net_keep_prob_input():
    """
    Return a Tensor for keep probability
    : return: Tensor for keep probability.
    """
    # TODO: Implement Function
    return tf.placeholder(tf.float32, name='keep_prob')    


"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""
tf.reset_default_graph()
tests.test_nn_image_inputs(neural_net_image_input)
tests.test_nn_label_inputs(neural_net_label_input)
tests.test_nn_keep_prob_inputs(neural_net_keep_prob_input)

```

    Image Input Tests Passed.
    Label Input Tests Passed.
    Keep Prob Tests Passed.
    

### Convolution and Max Pooling Layer
Convolution layers have a lot of success with images. For this code cell, you should implement the function `conv2d_maxpool` to apply convolution then max pooling:
* Create the weight and bias using `conv_ksize`, `conv_num_outputs` and the shape of `x_tensor`.
* Apply a convolution to `x_tensor` using weight and `conv_strides`.
 * We recommend you use same padding, but you're welcome to use any padding.
* Add bias
* Add a nonlinear activation to the convolution.
* Apply Max Pooling using `pool_ksize` and `pool_strides`.
 * We recommend you use same padding, but you're welcome to use any padding.

**Note:** You **can't** use [TensorFlow Layers](https://www.tensorflow.org/api_docs/python/tf/layers) or [TensorFlow Layers (contrib)](https://www.tensorflow.org/api_guides/python/contrib.layers) for **this** layer, but you can still use TensorFlow's [Neural Network](https://www.tensorflow.org/api_docs/python/tf/nn) package. You may still use the shortcut option for all the **other** layers.


```python
def conv2d_maxpool(x_tensor, conv_num_outputs, conv_ksize, conv_strides, pool_ksize, pool_strides):
    """
    Apply convolution then max pooling to x_tensor
    :param x_tensor: TensorFlow Tensor
    :param conv_num_outputs: Number of outputs for the convolutional layer
    :param conv_ksize: kernal size 2-D Tuple for the convolutional layer
    :param conv_strides: Stride 2-D Tuple for convolution
    :param pool_ksize: kernal size 2-D Tuple for pool
    :param pool_strides: Stride 2-D Tuple for pool
    : return: A tensor that represents convolution and max pooling of x_tensor
    """
    # TODO: Implement Function
    input_channel_depth = int(x_tensor.get_shape()[3])
    filter_weights = tf.Variable(tf.truncated_normal([*conv_ksize, input_channel_depth, 
                                                      conv_num_outputs], dtype=tf.float32))
    filter_biases = tf.Variable(tf.constant(0, shape=[conv_num_outputs], dtype=tf.float32))
    
    conv1 = tf.nn.conv2d(input=x_tensor, filter=filter_weights, strides=[1, *conv_strides, 1], padding='SAME')
    conv1 += filter_biases
    max_pool = tf.nn.max_pool(conv1, [1, *pool_ksize, 1], strides=[1, *pool_strides, 1], padding='SAME')
    return tf.nn.relu(max_pool, name='relu_tensor')


"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""
tests.test_con_pool(conv2d_maxpool)
```

    Tests Passed
    

### Flatten Layer
Implement the `flatten` function to change the dimension of `x_tensor` from a 4-D tensor to a 2-D tensor.  The output should be the shape (*Batch Size*, *Flattened Image Size*). Shortcut option: you can use classes from the [TensorFlow Layers](https://www.tensorflow.org/api_docs/python/tf/layers) or [TensorFlow Layers (contrib)](https://www.tensorflow.org/api_guides/python/contrib.layers) packages for this layer. For more of a challenge, only use other TensorFlow packages.


```python
def flatten(x_tensor):
    """
    Flatten x_tensor to (Batch Size, Flattened Image Size)
    : x_tensor: A tensor of size (Batch Size, ...), where ... are the image dimensions.
    : return: A tensor of size (Batch Size, Flattened Image Size).
    """
    # TODO: Implement Function
    return tf.contrib.layers.flatten(x_tensor)


"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""
tests.test_flatten(flatten)
```

    Tests Passed
    

### Fully-Connected Layer
Implement the `fully_conn` function to apply a fully connected layer to `x_tensor` with the shape (*Batch Size*, *num_outputs*). Shortcut option: you can use classes from the [TensorFlow Layers](https://www.tensorflow.org/api_docs/python/tf/layers) or [TensorFlow Layers (contrib)](https://www.tensorflow.org/api_guides/python/contrib.layers) packages for this layer. For more of a challenge, only use other TensorFlow packages.


```python
def fully_conn(x_tensor, num_outputs):
    """
    Apply a fully connected layer to x_tensor using weight and bias
    : x_tensor: A 2-D tensor where the first dimension is batch size.
    : num_outputs: The number of output that the new tensor should be.
    : return: A 2-D tensor where the second dimension is num_outputs.
    """
    # TODO: Implement Function
    return tf.contrib.layers.fully_connected(inputs=x_tensor, num_outputs=num_outputs)


"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""
tests.test_fully_conn(fully_conn)
```

    Tests Passed
    

### Output Layer
Implement the `output` function to apply a fully connected layer to `x_tensor` with the shape (*Batch Size*, *num_outputs*). Shortcut option: you can use classes from the [TensorFlow Layers](https://www.tensorflow.org/api_docs/python/tf/layers) or [TensorFlow Layers (contrib)](https://www.tensorflow.org/api_guides/python/contrib.layers) packages for this layer. For more of a challenge, only use other TensorFlow packages.

**Note:** Activation, softmax, or cross entropy should **not** be applied to this.


```python
from numpy.distutils.system_info import x11_info


def output(x_tensor, num_outputs):
    """
    Apply a output layer to x_tensor using weight and bias
    : x_tensor: A 2-D tensor where the first dimension is batch size.
    : num_outputs: The number of output that the new tensor should be.
    : return: A 2-D tensor where the second dimension is num_outputs.
    """
    # TODO: Implement Function
    return tf.layers.dense(inputs=x_tensor, units=num_outputs)


"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""
tests.test_output(output)
```

    Tests Passed
    

### Create Convolutional Model
Implement the function `conv_net` to create a convolutional neural network model. The function takes in a batch of images, `x`, and outputs logits.  Use the layers you created above to create this model:

* Apply 1, 2, or 3 Convolution and Max Pool layers
* Apply a Flatten Layer
* Apply 1, 2, or 3 Fully Connected Layers
* Apply an Output Layer
* Return the output
* Apply [TensorFlow's Dropout](https://www.tensorflow.org/api_docs/python/tf/nn/dropout) to one or more layers in the model using `keep_prob`. 


```python
def conv_net(x, keep_prob):
    """
    Create a convolutional neural network model
    : x: Placeholder tensor that holds image data.
    : keep_prob: Placeholder tensor that hold dropout keep probability.
    : return: Tensor that represents logits
    """
    # TODO: Apply 1, 2, or 3 Convolution and Max Pool layers
    #    Play around with different number of outputs, kernel size and stride
    # Function Definition from Above:
    x_tensor = x
    conv_num_outputs = 64
    conv_ksize = (5, 5)
    conv_strides = (2, 2)
    pool_ksize = (3, 3)
    pool_strides = (2, 2)
    conv2d_layer = conv2d_maxpool(x_tensor, conv_num_outputs, conv_ksize, conv_strides, pool_ksize, pool_strides)
    
    # TODO: Apply a Flatten Layer
    # Function Definition from Above:
    flatten_layer = flatten(conv2d_layer)
    

    # TODO: Apply 1, 2, or 3 Fully Connected Layers
    #    Play around with different number of outputs
    # Function Definition from Above:
    num_outputs = 10
    fully_conn_layer = fully_conn(flatten_layer, 1000)
    fully_conn_layer = tf.nn.dropout(fully_conn_layer, keep_prob)
    
    # TODO: Apply an Output Layer
    #    Set this to the number of classes
    # Function Definition from Above:
    #   output(x_tensor, num_outputs)
    # TODO: return output
    return output(fully_conn_layer, num_outputs)


"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""

##############################
## Build the Neural Network ##
##############################

# Remove previous weights, bias, inputs, etc..
tf.reset_default_graph()

# Inputs
x = neural_net_image_input((32, 32, 3))
y = neural_net_label_input(10)
keep_prob = neural_net_keep_prob_input()

# Model
logits = conv_net(x, keep_prob)

# Name logits Tensor, so that is can be loaded from disk after training
logits = tf.identity(logits, name='logits')

# Loss and Optimizer
cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(logits=logits, labels=y))
optimizer = tf.train.AdamOptimizer().minimize(cost)

# Accuracy
correct_pred = tf.equal(tf.argmax(logits, 1), tf.argmax(y, 1))
accuracy = tf.reduce_mean(tf.cast(correct_pred, tf.float32), name='accuracy')

tests.test_conv_net(conv_net)
```

    Neural Network Built!
    

## Train the Neural Network
### Single Optimization
Implement the function `train_neural_network` to do a single optimization.  The optimization should use `optimizer` to optimize in `session` with a `feed_dict` of the following:
* `x` for image input
* `y` for labels
* `keep_prob` for keep probability for dropout

This function will be called for each batch, so `tf.global_variables_initializer()` has already been called.

Note: Nothing needs to be returned. This function is only optimizing the neural network.


```python
def train_neural_network(session, optimizer, keep_probability, feature_batch, label_batch):
    """
    Optimize the session on a batch of images and labels
    : session: Current TensorFlow session
    : optimizer: TensorFlow optimizer function
    : keep_probability: keep probability
    : feature_batch: Batch of Numpy image data
    : label_batch: Batch of Numpy label data
    """
    # TODO: Implement Function
    session.run(optimizer, feed_dict={x: feature_batch, y: label_batch, keep_prob: keep_probability})
    

"""
DON'T MODIFY ANYTHING IN THIS CELL THAT IS BELOW THIS LINE
"""
tests.test_train_nn(train_neural_network)
```

    Tests Passed
    

### Show Stats
Implement the function `print_stats` to print loss and validation accuracy.  Use the global variables `valid_features` and `valid_labels` to calculate validation accuracy.  Use a keep probability of `1.0` to calculate the loss and validation accuracy.


```python
def print_stats(session, feature_batch, label_batch, cost, accuracy):
    """
    Print information about loss and validation accuracy
    : session: Current TensorFlow session
    : feature_batch: Batch of Numpy image data
    : label_batch: Batch of Numpy label data
    : cost: TensorFlow cost function
    : accuracy: TensorFlow accuracy function
    """
    # TODO: Implement Function
    loss = session.run(cost, feed_dict={x: feature_batch, y: label_batch, keep_prob: 1.0})
    valid_acc = session.run(accuracy, feed_dict={x: valid_features, y: valid_labels, keep_prob: 1.0})
    print('Loss: {:>10.4f} Accuracy: {:.6f}'.format(loss, valid_acc))

```

### Hyperparameters
Tune the following parameters:
* Set `epochs` to the number of iterations until the network stops learning or start overfitting
* Set `batch_size` to the highest number that your machine has memory for.  Most people set them to common sizes of memory:
 * 64
 * 128
 * 256
 * ...
* Set `keep_probability` to the probability of keeping a node using dropout


```python
# TODO: Tune Parameters
epochs = 30
batch_size = 512
keep_probability = 0.8
```

### Train on a Single CIFAR-10 Batch
Instead of training the neural network on all the CIFAR-10 batches of data, let's use a single batch. This should save time while you iterate on the model to get a better accuracy.  Once the final validation accuracy is 50% or greater, run the model on all the data in the next section.


```python
"""
DON'T MODIFY ANYTHING IN THIS CELL
"""
print('Checking the Training on a Single Batch...')
with tf.Session() as sess:
    # Initializing the variables
    sess.run(tf.global_variables_initializer())
    
    # Training cycle
    for epoch in range(epochs):
        batch_i = 1
        for batch_features, batch_labels in helper.load_preprocess_training_batch(batch_i, batch_size):
            train_neural_network(sess, optimizer, keep_probability, batch_features, batch_labels)
        print('Epoch {:>2}, CIFAR-10 Batch {}:  '.format(epoch + 1, batch_i), end='')
        print_stats(sess, batch_features, batch_labels, cost, accuracy)
```

    Checking the Training on a Single Batch...
    Epoch  1, CIFAR-10 Batch 1:  Loss:     4.2471 Accuracy: 0.210000
    Epoch  2, CIFAR-10 Batch 1:  Loss:     2.0152 Accuracy: 0.293200
    Epoch  3, CIFAR-10 Batch 1:  Loss:     1.8660 Accuracy: 0.363200
    Epoch  4, CIFAR-10 Batch 1:  Loss:     1.7256 Accuracy: 0.399600
    Epoch  5, CIFAR-10 Batch 1:  Loss:     1.6314 Accuracy: 0.422400
    Epoch  6, CIFAR-10 Batch 1:  Loss:     1.5434 Accuracy: 0.442600
    Epoch  7, CIFAR-10 Batch 1:  Loss:     1.4608 Accuracy: 0.460400
    Epoch  8, CIFAR-10 Batch 1:  Loss:     1.3799 Accuracy: 0.478800
    Epoch  9, CIFAR-10 Batch 1:  Loss:     1.2976 Accuracy: 0.492600
    Epoch 10, CIFAR-10 Batch 1:  Loss:     1.2198 Accuracy: 0.506200
    Epoch 11, CIFAR-10 Batch 1:  Loss:     1.1562 Accuracy: 0.511600
    Epoch 12, CIFAR-10 Batch 1:  Loss:     1.1068 Accuracy: 0.518000
    Epoch 13, CIFAR-10 Batch 1:  Loss:     1.0565 Accuracy: 0.524400
    Epoch 14, CIFAR-10 Batch 1:  Loss:     1.0012 Accuracy: 0.530600
    Epoch 15, CIFAR-10 Batch 1:  Loss:     0.9613 Accuracy: 0.539000
    Epoch 16, CIFAR-10 Batch 1:  Loss:     0.9136 Accuracy: 0.547200
    Epoch 17, CIFAR-10 Batch 1:  Loss:     0.8769 Accuracy: 0.545800
    Epoch 18, CIFAR-10 Batch 1:  Loss:     0.8420 Accuracy: 0.550200
    Epoch 19, CIFAR-10 Batch 1:  Loss:     0.8082 Accuracy: 0.546800
    Epoch 20, CIFAR-10 Batch 1:  Loss:     0.7668 Accuracy: 0.553400
    Epoch 21, CIFAR-10 Batch 1:  Loss:     0.7456 Accuracy: 0.558000
    Epoch 22, CIFAR-10 Batch 1:  Loss:     0.7081 Accuracy: 0.559000
    Epoch 23, CIFAR-10 Batch 1:  Loss:     0.6813 Accuracy: 0.552200
    Epoch 24, CIFAR-10 Batch 1:  Loss:     0.6578 Accuracy: 0.559800
    Epoch 25, CIFAR-10 Batch 1:  Loss:     0.6429 Accuracy: 0.562800
    Epoch 26, CIFAR-10 Batch 1:  Loss:     0.6207 Accuracy: 0.564000
    Epoch 27, CIFAR-10 Batch 1:  Loss:     0.5977 Accuracy: 0.560200
    Epoch 28, CIFAR-10 Batch 1:  Loss:     0.5704 Accuracy: 0.564800
    Epoch 29, CIFAR-10 Batch 1:  Loss:     0.5648 Accuracy: 0.559600
    Epoch 30, CIFAR-10 Batch 1:  Loss:     0.5363 Accuracy: 0.562800
    

### Fully Train the Model
Now that you got a good accuracy with a single CIFAR-10 batch, try it with all five batches.


```python
"""
DON'T MODIFY ANYTHING IN THIS CELL
"""
save_model_path = './image_classification'

print('Training...')
with tf.Session() as sess:
    # Initializing the variables
    sess.run(tf.global_variables_initializer())
    
    # Training cycle
    for epoch in range(epochs):
        # Loop over all batches
        n_batches = 5
        for batch_i in range(1, n_batches + 1):
            for batch_features, batch_labels in helper.load_preprocess_training_batch(batch_i, batch_size):
                train_neural_network(sess, optimizer, keep_probability, batch_features, batch_labels)
            print('Epoch {:>2}, CIFAR-10 Batch {}:  '.format(epoch + 1, batch_i), end='')
            print_stats(sess, batch_features, batch_labels, cost, accuracy)
            
    # Save Model
    saver = tf.train.Saver()
    save_path = saver.save(sess, save_model_path)
```

    Training...
    Epoch  1, CIFAR-10 Batch 1:  Loss:     6.5434 Accuracy: 0.127200
    Epoch  1, CIFAR-10 Batch 2:  Loss:     1.9689 Accuracy: 0.246000
    Epoch  1, CIFAR-10 Batch 3:  Loss:     1.7488 Accuracy: 0.350000
    Epoch  1, CIFAR-10 Batch 4:  Loss:     1.6143 Accuracy: 0.406600
    Epoch  1, CIFAR-10 Batch 5:  Loss:     1.6596 Accuracy: 0.424400
    Epoch  2, CIFAR-10 Batch 1:  Loss:     1.6434 Accuracy: 0.442800
    Epoch  2, CIFAR-10 Batch 2:  Loss:     1.4792 Accuracy: 0.463800
    Epoch  2, CIFAR-10 Batch 3:  Loss:     1.3479 Accuracy: 0.476000
    Epoch  2, CIFAR-10 Batch 4:  Loss:     1.3143 Accuracy: 0.492000
    Epoch  2, CIFAR-10 Batch 5:  Loss:     1.4050 Accuracy: 0.507800
    Epoch  3, CIFAR-10 Batch 1:  Loss:     1.3846 Accuracy: 0.515200
    Epoch  3, CIFAR-10 Batch 2:  Loss:     1.2881 Accuracy: 0.510400
    Epoch  3, CIFAR-10 Batch 3:  Loss:     1.1859 Accuracy: 0.518400
    Epoch  3, CIFAR-10 Batch 4:  Loss:     1.1446 Accuracy: 0.532000
    Epoch  3, CIFAR-10 Batch 5:  Loss:     1.2411 Accuracy: 0.530400
    Epoch  4, CIFAR-10 Batch 1:  Loss:     1.2600 Accuracy: 0.550400
    Epoch  4, CIFAR-10 Batch 2:  Loss:     1.1957 Accuracy: 0.535000
    Epoch  4, CIFAR-10 Batch 3:  Loss:     1.1022 Accuracy: 0.543200
    Epoch  4, CIFAR-10 Batch 4:  Loss:     1.0442 Accuracy: 0.563200
    Epoch  4, CIFAR-10 Batch 5:  Loss:     1.1387 Accuracy: 0.560600
    Epoch  5, CIFAR-10 Batch 1:  Loss:     1.1763 Accuracy: 0.576400
    Epoch  5, CIFAR-10 Batch 2:  Loss:     1.1229 Accuracy: 0.552800
    Epoch  5, CIFAR-10 Batch 3:  Loss:     1.0440 Accuracy: 0.571800
    Epoch  5, CIFAR-10 Batch 4:  Loss:     0.9700 Accuracy: 0.574000
    Epoch  5, CIFAR-10 Batch 5:  Loss:     1.0512 Accuracy: 0.588600
    Epoch  6, CIFAR-10 Batch 1:  Loss:     1.1002 Accuracy: 0.585200
    Epoch  6, CIFAR-10 Batch 2:  Loss:     1.0845 Accuracy: 0.572600
    Epoch  6, CIFAR-10 Batch 3:  Loss:     0.9780 Accuracy: 0.586800
    Epoch  6, CIFAR-10 Batch 4:  Loss:     0.9188 Accuracy: 0.591200
    Epoch  6, CIFAR-10 Batch 5:  Loss:     1.0008 Accuracy: 0.586600
    Epoch  7, CIFAR-10 Batch 1:  Loss:     1.0359 Accuracy: 0.594600
    Epoch  7, CIFAR-10 Batch 2:  Loss:     0.9879 Accuracy: 0.582400
    Epoch  7, CIFAR-10 Batch 3:  Loss:     0.8888 Accuracy: 0.599400
    Epoch  7, CIFAR-10 Batch 4:  Loss:     0.8726 Accuracy: 0.604600
    Epoch  7, CIFAR-10 Batch 5:  Loss:     0.9699 Accuracy: 0.591400
    Epoch  8, CIFAR-10 Batch 1:  Loss:     0.9915 Accuracy: 0.600600
    Epoch  8, CIFAR-10 Batch 2:  Loss:     0.9699 Accuracy: 0.588800
    Epoch  8, CIFAR-10 Batch 3:  Loss:     0.8911 Accuracy: 0.599000
    Epoch  8, CIFAR-10 Batch 4:  Loss:     0.8341 Accuracy: 0.601800
    Epoch  8, CIFAR-10 Batch 5:  Loss:     0.9102 Accuracy: 0.601800
    Epoch  9, CIFAR-10 Batch 1:  Loss:     0.9653 Accuracy: 0.604400
    Epoch  9, CIFAR-10 Batch 2:  Loss:     0.9526 Accuracy: 0.585400
    Epoch  9, CIFAR-10 Batch 3:  Loss:     0.8421 Accuracy: 0.605800
    Epoch  9, CIFAR-10 Batch 4:  Loss:     0.8111 Accuracy: 0.609800
    Epoch  9, CIFAR-10 Batch 5:  Loss:     0.8676 Accuracy: 0.610200
    Epoch 10, CIFAR-10 Batch 1:  Loss:     0.9324 Accuracy: 0.603000
    Epoch 10, CIFAR-10 Batch 2:  Loss:     0.9186 Accuracy: 0.602200
    Epoch 10, CIFAR-10 Batch 3:  Loss:     0.7992 Accuracy: 0.615200
    Epoch 10, CIFAR-10 Batch 4:  Loss:     0.7814 Accuracy: 0.609600
    Epoch 10, CIFAR-10 Batch 5:  Loss:     0.8583 Accuracy: 0.608000
    Epoch 11, CIFAR-10 Batch 1:  Loss:     0.8907 Accuracy: 0.619600
    Epoch 11, CIFAR-10 Batch 2:  Loss:     0.8784 Accuracy: 0.600400
    Epoch 11, CIFAR-10 Batch 3:  Loss:     0.7809 Accuracy: 0.616800
    Epoch 11, CIFAR-10 Batch 4:  Loss:     0.7268 Accuracy: 0.621600
    Epoch 11, CIFAR-10 Batch 5:  Loss:     0.7851 Accuracy: 0.625200
    Epoch 12, CIFAR-10 Batch 1:  Loss:     0.8648 Accuracy: 0.626600
    Epoch 12, CIFAR-10 Batch 2:  Loss:     0.8447 Accuracy: 0.605000
    Epoch 12, CIFAR-10 Batch 3:  Loss:     0.7586 Accuracy: 0.617000
    Epoch 12, CIFAR-10 Batch 4:  Loss:     0.7448 Accuracy: 0.617000
    Epoch 12, CIFAR-10 Batch 5:  Loss:     0.7878 Accuracy: 0.624200
    Epoch 13, CIFAR-10 Batch 1:  Loss:     0.8448 Accuracy: 0.630600
    Epoch 13, CIFAR-10 Batch 2:  Loss:     0.8201 Accuracy: 0.601200
    Epoch 13, CIFAR-10 Batch 3:  Loss:     0.7377 Accuracy: 0.623000
    Epoch 13, CIFAR-10 Batch 4:  Loss:     0.7213 Accuracy: 0.614800
    Epoch 13, CIFAR-10 Batch 5:  Loss:     0.7710 Accuracy: 0.628400
    Epoch 14, CIFAR-10 Batch 1:  Loss:     0.8338 Accuracy: 0.629000
    Epoch 14, CIFAR-10 Batch 2:  Loss:     0.8110 Accuracy: 0.602800
    Epoch 14, CIFAR-10 Batch 3:  Loss:     0.7199 Accuracy: 0.624200
    Epoch 14, CIFAR-10 Batch 4:  Loss:     0.6573 Accuracy: 0.637800
    Epoch 14, CIFAR-10 Batch 5:  Loss:     0.7278 Accuracy: 0.635200
    Epoch 15, CIFAR-10 Batch 1:  Loss:     0.7864 Accuracy: 0.629400
    Epoch 15, CIFAR-10 Batch 2:  Loss:     0.7768 Accuracy: 0.600000
    Epoch 15, CIFAR-10 Batch 3:  Loss:     0.6791 Accuracy: 0.633800
    Epoch 15, CIFAR-10 Batch 4:  Loss:     0.6481 Accuracy: 0.630600
    Epoch 15, CIFAR-10 Batch 5:  Loss:     0.7010 Accuracy: 0.633200
    Epoch 16, CIFAR-10 Batch 1:  Loss:     0.7673 Accuracy: 0.641000
    Epoch 16, CIFAR-10 Batch 2:  Loss:     0.7708 Accuracy: 0.597600
    Epoch 16, CIFAR-10 Batch 3:  Loss:     0.6924 Accuracy: 0.622600
    Epoch 16, CIFAR-10 Batch 4:  Loss:     0.6782 Accuracy: 0.623600
    Epoch 16, CIFAR-10 Batch 5:  Loss:     0.7311 Accuracy: 0.627400
    Epoch 17, CIFAR-10 Batch 1:  Loss:     0.7712 Accuracy: 0.622000
    Epoch 17, CIFAR-10 Batch 2:  Loss:     0.7277 Accuracy: 0.606000
    Epoch 17, CIFAR-10 Batch 3:  Loss:     0.6568 Accuracy: 0.630400
    Epoch 17, CIFAR-10 Batch 4:  Loss:     0.6275 Accuracy: 0.635600
    Epoch 17, CIFAR-10 Batch 5:  Loss:     0.6568 Accuracy: 0.635800
    Epoch 18, CIFAR-10 Batch 1:  Loss:     0.7605 Accuracy: 0.631000
    Epoch 18, CIFAR-10 Batch 2:  Loss:     0.7214 Accuracy: 0.601000
    Epoch 18, CIFAR-10 Batch 3:  Loss:     0.6424 Accuracy: 0.635200
    Epoch 18, CIFAR-10 Batch 4:  Loss:     0.6219 Accuracy: 0.636200
    Epoch 18, CIFAR-10 Batch 5:  Loss:     0.6258 Accuracy: 0.643200
    Epoch 19, CIFAR-10 Batch 1:  Loss:     0.7212 Accuracy: 0.632800
    Epoch 19, CIFAR-10 Batch 2:  Loss:     0.7284 Accuracy: 0.596200
    Epoch 19, CIFAR-10 Batch 3:  Loss:     0.6236 Accuracy: 0.636000
    Epoch 19, CIFAR-10 Batch 4:  Loss:     0.6041 Accuracy: 0.636600
    Epoch 19, CIFAR-10 Batch 5:  Loss:     0.6318 Accuracy: 0.640200
    Epoch 20, CIFAR-10 Batch 1:  Loss:     0.7188 Accuracy: 0.637800
    Epoch 20, CIFAR-10 Batch 2:  Loss:     0.6761 Accuracy: 0.612400
    Epoch 20, CIFAR-10 Batch 3:  Loss:     0.6294 Accuracy: 0.636800
    Epoch 20, CIFAR-10 Batch 4:  Loss:     0.5708 Accuracy: 0.635600
    Epoch 20, CIFAR-10 Batch 5:  Loss:     0.6210 Accuracy: 0.631800
    Epoch 21, CIFAR-10 Batch 1:  Loss:     0.6969 Accuracy: 0.644600
    Epoch 21, CIFAR-10 Batch 2:  Loss:     0.6530 Accuracy: 0.619400
    Epoch 21, CIFAR-10 Batch 3:  Loss:     0.5840 Accuracy: 0.635400
    Epoch 21, CIFAR-10 Batch 4:  Loss:     0.5626 Accuracy: 0.640800
    Epoch 21, CIFAR-10 Batch 5:  Loss:     0.6005 Accuracy: 0.642000
    Epoch 22, CIFAR-10 Batch 1:  Loss:     0.6888 Accuracy: 0.631200
    Epoch 22, CIFAR-10 Batch 2:  Loss:     0.6646 Accuracy: 0.624800
    Epoch 22, CIFAR-10 Batch 3:  Loss:     0.6013 Accuracy: 0.636200
    Epoch 22, CIFAR-10 Batch 4:  Loss:     0.5542 Accuracy: 0.636400
    Epoch 22, CIFAR-10 Batch 5:  Loss:     0.5973 Accuracy: 0.635600
    Epoch 23, CIFAR-10 Batch 1:  Loss:     0.6826 Accuracy: 0.636600
    Epoch 23, CIFAR-10 Batch 2:  Loss:     0.6165 Accuracy: 0.618600
    Epoch 23, CIFAR-10 Batch 3:  Loss:     0.5737 Accuracy: 0.635000
    Epoch 23, CIFAR-10 Batch 4:  Loss:     0.5434 Accuracy: 0.642200
    Epoch 23, CIFAR-10 Batch 5:  Loss:     0.5518 Accuracy: 0.649200
    Epoch 24, CIFAR-10 Batch 1:  Loss:     0.6363 Accuracy: 0.640800
    Epoch 24, CIFAR-10 Batch 2:  Loss:     0.6168 Accuracy: 0.629000
    Epoch 24, CIFAR-10 Batch 3:  Loss:     0.5549 Accuracy: 0.640000
    Epoch 24, CIFAR-10 Batch 4:  Loss:     0.5317 Accuracy: 0.648400
    Epoch 24, CIFAR-10 Batch 5:  Loss:     0.5158 Accuracy: 0.651000
    Epoch 25, CIFAR-10 Batch 1:  Loss:     0.6413 Accuracy: 0.644400
    Epoch 25, CIFAR-10 Batch 2:  Loss:     0.5692 Accuracy: 0.630200
    Epoch 25, CIFAR-10 Batch 3:  Loss:     0.5226 Accuracy: 0.634400
    Epoch 25, CIFAR-10 Batch 4:  Loss:     0.5308 Accuracy: 0.640800
    Epoch 25, CIFAR-10 Batch 5:  Loss:     0.5221 Accuracy: 0.646800
    Epoch 26, CIFAR-10 Batch 1:  Loss:     0.5999 Accuracy: 0.644200
    Epoch 26, CIFAR-10 Batch 2:  Loss:     0.5926 Accuracy: 0.629800
    Epoch 26, CIFAR-10 Batch 3:  Loss:     0.5151 Accuracy: 0.640200
    Epoch 26, CIFAR-10 Batch 4:  Loss:     0.5414 Accuracy: 0.638400
    Epoch 26, CIFAR-10 Batch 5:  Loss:     0.4832 Accuracy: 0.642400
    Epoch 27, CIFAR-10 Batch 1:  Loss:     0.6161 Accuracy: 0.643400
    Epoch 27, CIFAR-10 Batch 2:  Loss:     0.5439 Accuracy: 0.634800
    Epoch 27, CIFAR-10 Batch 3:  Loss:     0.5081 Accuracy: 0.641200
    Epoch 27, CIFAR-10 Batch 4:  Loss:     0.5201 Accuracy: 0.638200
    Epoch 27, CIFAR-10 Batch 5:  Loss:     0.4774 Accuracy: 0.645800
    Epoch 28, CIFAR-10 Batch 1:  Loss:     0.6055 Accuracy: 0.650800
    Epoch 28, CIFAR-10 Batch 2:  Loss:     0.5674 Accuracy: 0.632200
    Epoch 28, CIFAR-10 Batch 3:  Loss:     0.5133 Accuracy: 0.631600
    Epoch 28, CIFAR-10 Batch 4:  Loss:     0.4827 Accuracy: 0.643200
    Epoch 28, CIFAR-10 Batch 5:  Loss:     0.4355 Accuracy: 0.644600
    Epoch 29, CIFAR-10 Batch 1:  Loss:     0.5991 Accuracy: 0.637400
    Epoch 29, CIFAR-10 Batch 2:  Loss:     0.5373 Accuracy: 0.635400
    Epoch 29, CIFAR-10 Batch 3:  Loss:     0.4676 Accuracy: 0.647200
    Epoch 29, CIFAR-10 Batch 4:  Loss:     0.4807 Accuracy: 0.635000
    Epoch 29, CIFAR-10 Batch 5:  Loss:     0.4621 Accuracy: 0.638800
    Epoch 30, CIFAR-10 Batch 1:  Loss:     0.5536 Accuracy: 0.648400
    Epoch 30, CIFAR-10 Batch 2:  Loss:     0.5118 Accuracy: 0.640400
    Epoch 30, CIFAR-10 Batch 3:  Loss:     0.4551 Accuracy: 0.642400
    Epoch 30, CIFAR-10 Batch 4:  Loss:     0.4755 Accuracy: 0.633200
    Epoch 30, CIFAR-10 Batch 5:  Loss:     0.4146 Accuracy: 0.650000
    

# Checkpoint
The model has been saved to disk.
## Test Model
Test your model against the test dataset.  This will be your final accuracy. You should have an accuracy greater than 50%. If you don't, keep tweaking the model architecture and parameters.


```python
"""
DON'T MODIFY ANYTHING IN THIS CELL
"""
%matplotlib inline
%config InlineBackend.figure_format = 'retina'

import tensorflow as tf
import pickle
import helper
import random

# Set batch size if not already set
try:
    if batch_size:
        pass
except NameError:
    batch_size = 64

save_model_path = './image_classification'
n_samples = 4
top_n_predictions = 3


def test_model():
    """
    Test the saved model against the test dataset
    """

    test_features, test_labels = pickle.load(open('preprocess_test.p', mode='rb'))
    loaded_graph = tf.Graph()

    with tf.Session(graph=loaded_graph) as sess:
        # Load model
        loader = tf.train.import_meta_graph(save_model_path + '.meta')
        loader.restore(sess, save_model_path)

        # Get Tensors from loaded model
        loaded_x = loaded_graph.get_tensor_by_name('x:0')
        loaded_y = loaded_graph.get_tensor_by_name('y:0')
        loaded_keep_prob = loaded_graph.get_tensor_by_name('keep_prob:0')
        loaded_logits = loaded_graph.get_tensor_by_name('logits:0')
        loaded_acc = loaded_graph.get_tensor_by_name('accuracy:0')
        
        # Get accuracy in batches for memory limitations
        test_batch_acc_total = 0
        test_batch_count = 0
        
        for test_feature_batch, test_label_batch in helper.batch_features_labels(test_features, test_labels, batch_size):
            test_batch_acc_total += sess.run(
                loaded_acc,
                feed_dict={loaded_x: test_feature_batch, loaded_y: test_label_batch, loaded_keep_prob: 1.0})
            test_batch_count += 1

        print('Testing Accuracy: {}\n'.format(test_batch_acc_total/test_batch_count))

        # Print Random Samples
        random_test_features, random_test_labels = tuple(zip(*random.sample(list(zip(test_features, test_labels)), n_samples)))
        random_test_predictions = sess.run(
            tf.nn.top_k(tf.nn.softmax(loaded_logits), top_n_predictions),
            feed_dict={loaded_x: random_test_features, loaded_y: random_test_labels, loaded_keep_prob: 1.0})
        helper.display_image_predictions(random_test_features, random_test_labels, random_test_predictions)


test_model()
```

    INFO:tensorflow:Restoring parameters from ./image_classification
    Testing Accuracy: 0.6460707724094391
    
    


![png](output_36_1.png)


## Why 50-80% Accuracy?
You might be wondering why you can't get an accuracy any higher. First things first, 50% isn't bad for a simple CNN.  Pure guessing would get you 10% accuracy. However, you might notice people are getting scores [well above 80%](http://rodrigob.github.io/are_we_there_yet/build/classification_datasets_results.html#43494641522d3130).  That's because we haven't taught you all there is to know about neural networks. We still need to cover a few more techniques.
## Submitting This Project
When submitting this project, make sure to run all the cells before saving the notebook.  Save the notebook file as "dlnd_image_classification.ipynb" and save it as a HTML file under "File" -> "Download as".  Include the "helper.py" and "problem_unittests.py" files in your submission.
