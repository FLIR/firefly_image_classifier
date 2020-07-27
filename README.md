# Image classification model library

This repository is based on the TensorFlow-Slim image classification repository
[TF-slim](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/slim)
is a new lightweight high-level API of TensorFlow (`tensorflow.contrib.slim`)
for defining, training and evaluating complex
models. This directory contains
code for training and evaluating several widely used Convolutional Neural
Network (CNN) image classification models using TF-slim.
It contains scripts that will allow
you to train models from scratch or fine-tune them from pre-trained network
weights. It also contains code for downloading standard image datasets,
converting them
to TensorFlow's native TFRecord format and reading them in using TF-Slim's
data reading and queueing utilities. You can easily train any model on any of
these datasets, as we demonstrate below. We've also included a
[jupyter notebook](https://github.com/tensorflow/models/blob/master/research/slim/slim_walkthrough.ipynb),
which provides working examples of how to use TF-Slim for image classification.
For developing or modifying your own models, see also the [main TF-Slim page](https://github.com/tensorflow/tensorflow/tree/r1.13/tensorflow/contrib/slim).

## Contacts

Maintainers of this repository:
This repository is maintained by FLIR-IIS R&D team. 

* Ahmed Sigiuk, Ahmed.Sigiuk@flir.com
* Di Xu, Di.Xu@flir.com
* Douglas Chong, Douglas.Chong@flir.com



## Citation
"TensorFlow-Slim image classification model library"
N. Silberman and S. Guadarrama, 2016.
https://github.com/tensorflow/models/tree/master/research/slim

## Table of contents

<a href="#Install">Installation and setup</a><br>
<a href='#Data'>Preparing the datasets</a><br>
<a href='#Pretrained'>Using pre-trained models</a><br>
<a href='#Training'>Training from scratch</a><br>
<a href='#Tuning'>Fine tuning to a new task</a><br>
<a href='#Eval'>Evaluating performance</a><br>
<a href='#Export'>Exporting Inference Graph</a><br>
<a href='#Troubleshooting'>Troubleshooting and Current Known Issues</a><br>

#Installation
<a id='Install'></a>

In this section, we describe the steps required to install the appropriate Tensorflow prerequisite packages.
This repository uses Tensorflow framework for training (with GPU and CPU support), and was tested on Tensorflow version 1.13.
We provide two options for instaling Tensorflow on your system.
<a href="#Host">Setting up python libraries on a Linux machine</a><br>
<a href="#Docker">Insall Tensorflow using Docker</a><br>

## Setting up python libraries on a Linux machine
<a id='Host'></a>
This section guides you through the steps required to install Tensorflow- 1.13 and Tensorflow-slim libraries on your Linux host machine. 

### Insall Tensorflow using pip
You can use `pip` python package manager to install Tensorflow library on your host machine.

```bash
# Install tensorflow. 
# If you have GPU,
pip install tensorflow-gpu==1.13.2  
# or, for training on CPU
pip install tensorflow

pip install sklearn
```

### Installing latest version of TF-slim

TF-Slim is available as `tf.contrib.slim` via TensorFlow 1.0. To test that your
installation is working, execute the following command; it should run without
raising any errors.

```
python -c "import tensorflow.contrib.slim as slim; eval = slim.evaluation.evaluate_once"
```

### Installing the TF-slim image models library

To use TF-Slim for image classification, you also have to install the [TF-Slim image models library](https://github.com/tensorflow/models/tree/master/research/slim),
which is not part of the core TF library.
To do this, check out the
[tensorflow/models](https://github.com/tensorflow/models/) repository as follows:

```bash
cd $HOME/workspace
git clone https://github.com/tensorflow/models/
```

This will put the TF-Slim image models library in `$HOME/workspace/models/research/slim`.
(It will also create a directory called
[models/inception](https://github.com/tensorflow/models/tree/r1.13/research/inception),
which contains an older version of slim; you can safely ignore this.)

To verify that this has worked, execute the following commands; it should run
without raising any errors.

```
cd $HOME/workspace/models/research/slim
python -c "from nets import cifarnet; mynet = cifarnet.cifarnet"
```

## Insall Tensorflow using Docker
<a id='Docker'></a>
To be added

```bash
# Install tensorflow. 
docker build ...

docker run ...

```

# Preparing the datasets
<a id='Data'></a>

As part of this library, we've included scripts to download several popular
image datasets (listed below) and convert them to slim format.

Dataset | Training Set Size | Testing Set Size | Number of Classes | Comments
:------:|:---------------:|:---------------------:|:-----------:|:-----------:
Flowers|2500 | 2500 | 5 | Various sizes (source: Flickr)
[Cifar10](https://www.cs.toronto.edu/~kriz/cifar.html) | 60k| 10k | 10 |32x32 color
[MNIST](http://yann.lecun.com/exdb/mnist/)| 60k | 10k | 10 | 28x28 gray
[ImageNet](http://www.image-net.org/challenges/LSVRC/2012/)|1.2M| 50k | 1000 | Various sizes
VisualWakeWords|82783 | 40504 | 2 | Various sizes (source: MS COCO)

## Downloading and converting flower dataset to TFRecord format

For each dataset, we'll need to download the raw data and convert it to
TensorFlow's native
[TFRecord](https://www.tensorflow.org/versions/r0.10/api_docs/python/python_io.html#tfrecords-format-details)
format. Each TFRecord contains a
[TF-Example](https://github.com/tensorflow/tensorflow/blob/r0.10/tensorflow/core/example/example.proto)
protocol buffer. Below we demonstrate how to do this for the Flowers dataset.

```shell
$ DATA_DIR=/tmp/data/flowers
$ python download_and_convert_data.py \
    --dataset_name=flowers \
    --dataset_dir="${DATA_DIR}"
```

When the script finishes you will find several TFRecord files created:

```shell
$ ls ${DATA_DIR}
flowers_train-00000-of-00005.tfrecord
...
flowers_train-00004-of-00005.tfrecord
flowers_validation-00000-of-00005.tfrecord
...
flowers_validation-00004-of-00005.tfrecord
labels.txt
```

These represent the training and validation data, sharded over 5 files each.
You will also find the `$DATA_DIR/labels.txt` file which contains the mapping
from integer labels to class names.

You can use the same script to create the mnist, cifar10 and visualwakewords
datasets. However, for ImageNet, you have to follow the instructions
[here](https://github.com/tensorflow/models/blob/master/research/inception/README.md#getting-started).
Note that you first have to sign up for an account at image-net.org. Also, the
download can take several hours, and could use up to 500GB.

## Collect and convert your own dataset

First, you must collect and label a sample of images that you would like to train the model to classify. Then use the `create_and_convert_dataset.py` to convert this dataset to TFRecord format.

### Collect training images.

For each dataset, we'll need to label the dataset into classes by placing the raw image file into directory with matching class name. Please note the following;

* The `train_image_classifier.py` script only supports the following image formats 'jpg', 'jpeg', 'png', and 'bmp'.
* Label the images into classes using the parent directory name.
* Each image most be save into only one folder (representing the class)
* The ground-truth label for each image is taken from the parent directory name.

The diagram below shows the expected folder structure.

```

    dataset-name
    |
    |-- class_1
    |   |
    |   |--image_1.jpg
    |   |--image_2.jpg
    |           :
    |           :
    |-- class_2
    |   |
    |   |--image_1.jpg
    |   |--image_2.jpg
    |           :
    |           :
    |-- class_3
    |   |
    |   |--image_1.jpg
    |   |--image_2.jpg
    |           :
                :
```

​

### Convert custom dataset to TFRecord format

For each dataset, we'll need to label the dataset into classes by placing the raw image file into directory with matching class name.
Below we demonstrate how to do this for the blocks dataset.

```shell
$ IMAGE_INPUT_DIR=/path/to/folder/image_dir
$ TFRECORD_OUTPUT_DIR=/path/to/folder/tfrecord_dir
$ python create_and_convert_dataset.py \
    --dataset_name=blocks \
    --images_dataset_dir="${IMAGE_INPUT_DIR}" \
    --tfrecords_dataset_dir="${TFRECORD_OUTPUT_DIR}" \
    --validation_percentage=10 \
    --test_percentage=10
```

When the script finishes you will find several TFRecord files created:

```shell
$ ls ${DATA_DIR}
blocks_train-00000-of-00005.tfrecord
...
blocks_train-00004-of-00005.tfrecord
blocks_validation-00000-of-00005.tfrecord
...
blocks_validation-00004-of-00005.tfrecord
blocks_test-00000-of-00005.tfrecord
...
blocks_test-00004-of-00005.tfrecord
labels.txt
dataset_config.json
```

These represent the training, validation and test data, sharded over 5 files each.
You will also find the `$DATA_DIR/labels.txt` file which contains the mapping
from integer labels to class names. In addition, you will find the dataset_config.json file which stores some of the dataset attribues.

```shell
$ cat dataset_config.json
{"dataset_name": "blocks", 
"dataset_dir": "path/to/this/directory", 
"class_names": ['A', 'B'], 
"number_of_classes": 16, 
"dataset_split": {
	"validation": 100, 
	"train": 1000,
	"test": 100
	}
}
```

You can use the same script to create any other custom dataset. 

# Pre-trained Models
<a id='Pretrained'></a>

Neural nets work best when they have many parameters, making them powerful
function approximators.
However, this  means they must be trained on very large datasets. Because
training models from scratch can be a very computationally intensive process
requiring days or even weeks, we provide various pre-trained models,
as listed below. These CNNs have been trained on the
[ILSVRC-2012-CLS](http://www.image-net.org/challenges/LSVRC/2012/)
image classification dataset.

In the table below, we list each model, the corresponding
TensorFlow model file, the link to the model checkpoint, and the top 1 and top 5
accuracy (on the imagenet test set).
Note that the VGG and ResNet V1 parameters have been converted from their original
caffe formats
([here](https://github.com/BVLC/caffe/wiki/Model-Zoo#models-used-by-the-vgg-team-in-ilsvrc-2014)
and
[here](https://github.com/KaimingHe/deep-residual-networks)),
whereas the Inception and ResNet V2 parameters have been trained internally at
Google. Also be aware that these accuracies were computed by evaluating using a
single image crop. Some academic papers report higher accuracy by using multiple
crops at multiple scales.

Model | TF-Slim File | Checkpoint | Top-1 Accuracy| Top-5 Accuracy |
:----:|:------------:|:----------:|:-------:|:--------:|
[Inception V1](http://arxiv.org/abs/1409.4842v1)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/inception_v1.py)|[inception_v1_2016_08_28.tar.gz](http://download.tensorflow.org/models/inception_v1_2016_08_28.tar.gz)|69.8|89.6|
[Inception V2](http://arxiv.org/abs/1502.03167)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/inception_v2.py)|[inception_v2_2016_08_28.tar.gz](http://download.tensorflow.org/models/inception_v2_2016_08_28.tar.gz)|73.9|91.8|
[Inception V3](http://arxiv.org/abs/1512.00567)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/inception_v3.py)|[inception_v3_2016_08_28.tar.gz](http://download.tensorflow.org/models/inception_v3_2016_08_28.tar.gz)|78.0|93.9|
[Inception V4](http://arxiv.org/abs/1602.07261)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/inception_v4.py)|[inception_v4_2016_09_09.tar.gz](http://download.tensorflow.org/models/inception_v4_2016_09_09.tar.gz)|80.2|95.2|
[Inception-ResNet-v2](http://arxiv.org/abs/1602.07261)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/inception_resnet_v2.py)|[inception_resnet_v2_2016_08_30.tar.gz](http://download.tensorflow.org/models/inception_resnet_v2_2016_08_30.tar.gz)|80.4|95.3|
[ResNet V1 50](https://arxiv.org/abs/1512.03385)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/resnet_v1.py)|[resnet_v1_50_2016_08_28.tar.gz](http://download.tensorflow.org/models/resnet_v1_50_2016_08_28.tar.gz)|75.2|92.2|
[ResNet V1 101](https://arxiv.org/abs/1512.03385)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/resnet_v1.py)|[resnet_v1_101_2016_08_28.tar.gz](http://download.tensorflow.org/models/resnet_v1_101_2016_08_28.tar.gz)|76.4|92.9|
[ResNet V1 152](https://arxiv.org/abs/1512.03385)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/resnet_v1.py)|[resnet_v1_152_2016_08_28.tar.gz](http://download.tensorflow.org/models/resnet_v1_152_2016_08_28.tar.gz)|76.8|93.2|
[ResNet V2 50](https://arxiv.org/abs/1603.05027)^|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/resnet_v2.py)|[resnet_v2_50_2017_04_14.tar.gz](http://download.tensorflow.org/models/resnet_v2_50_2017_04_14.tar.gz)|75.6|92.8|
[ResNet V2 101](https://arxiv.org/abs/1603.05027)^|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/resnet_v2.py)|[resnet_v2_101_2017_04_14.tar.gz](http://download.tensorflow.org/models/resnet_v2_101_2017_04_14.tar.gz)|77.0|93.7|
[ResNet V2 152](https://arxiv.org/abs/1603.05027)^|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/resnet_v2.py)|[resnet_v2_152_2017_04_14.tar.gz](http://download.tensorflow.org/models/resnet_v2_152_2017_04_14.tar.gz)|77.8|94.1|
[ResNet V2 200](https://arxiv.org/abs/1603.05027)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/resnet_v2.py)|[TBA]()|79.9\*|95.2\*|
[VGG 16](http://arxiv.org/abs/1409.1556.pdf)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/vgg.py)|[vgg_16_2016_08_28.tar.gz](http://download.tensorflow.org/models/vgg_16_2016_08_28.tar.gz)|71.5|89.8|
[VGG 19](http://arxiv.org/abs/1409.1556.pdf)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/vgg.py)|[vgg_19_2016_08_28.tar.gz](http://download.tensorflow.org/models/vgg_19_2016_08_28.tar.gz)|71.1|89.8|
[MobileNet_v1_1.0_224](https://arxiv.org/pdf/1704.04861.pdf)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet_v1.py)|[mobilenet_v1_1.0_224.tgz](http://download.tensorflow.org/models/mobilenet_v1_2018_02_22/mobilenet_v1_1.0_224.tgz)|70.9|89.9|
[MobileNet_v1_0.50_160](https://arxiv.org/pdf/1704.04861.pdf)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet_v1.py)|[mobilenet_v1_0.50_160.tgz](http://download.tensorflow.org/models/mobilenet_v1_2018_02_22/mobilenet_v1_0.5_160.tgz)|59.1|81.9|
[MobileNet_v1_0.25_128](https://arxiv.org/pdf/1704.04861.pdf)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet_v1.py)|[mobilenet_v1_0.25_128.tgz](http://download.tensorflow.org/models/mobilenet_v1_2018_02_22/mobilenet_v1_0.25_128.tgz)|41.5|66.3|
[MobileNet_v2_1.4_224^*](https://arxiv.org/abs/1801.04381)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet/mobilenet_v2.py)| [mobilenet_v2_1.4_224.tgz](https://storage.googleapis.com/mobilenet_v2/checkpoints/mobilenet_v2_1.4_224.tgz) | 74.9 | 92.5|
[MobileNet_v2_1.0_224^*](https://arxiv.org/abs/1801.04381)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet/mobilenet_v2.py)| [mobilenet_v2_1.0_224.tgz](https://storage.googleapis.com/mobilenet_v2/checkpoints/mobilenet_v2_1.0_224.tgz) | 71.9 | 91.0
[NASNet-A_Mobile_224](https://arxiv.org/abs/1707.07012)#|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/nasnet/nasnet.py)|[nasnet-a_mobile_04_10_2017.tar.gz](https://storage.googleapis.com/download.tensorflow.org/models/nasnet-a_mobile_04_10_2017.tar.gz)|74.0|91.6|
[NASNet-A_Large_331](https://arxiv.org/abs/1707.07012)#|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/nasnet/nasnet.py)|[nasnet-a_large_04_10_2017.tar.gz](https://storage.googleapis.com/download.tensorflow.org/models/nasnet-a_large_04_10_2017.tar.gz)|82.7|96.2|
[PNASNet-5_Large_331](https://arxiv.org/abs/1712.00559)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/nasnet/pnasnet.py)|[pnasnet-5_large_2017_12_13.tar.gz](https://storage.googleapis.com/download.tensorflow.org/models/pnasnet-5_large_2017_12_13.tar.gz)|82.9|96.2|
[PNASNet-5_Mobile_224](https://arxiv.org/abs/1712.00559)|[Code](https://github.com/tensorflow/models/blob/master/research/slim/nets/nasnet/pnasnet.py)|[pnasnet-5_mobile_2017_12_13.tar.gz](https://storage.googleapis.com/download.tensorflow.org/models/pnasnet-5_mobile_2017_12_13.tar.gz)|74.2|91.9|

^ ResNet V2 models use Inception pre-processing and input image size of 299 (use
`--preprocessing_name inception --eval_image_size 299` when using
`eval_image_classifier.py`). Performance numbers for ResNet V2 models are
reported on the ImageNet validation set.

(#) More information and details about the NASNet architectures are available at this [README](nets/nasnet/README.md)

All 16 float MobileNet V1 models reported in the [MobileNet Paper](https://arxiv.org/abs/1704.04861) and all
16 quantized [TensorFlow Lite](https://www.tensorflow.org/mobile/tflite/) compatible MobileNet V1 models can be found
[here](https://github.com/tensorflow/models/tree/r1.13/research/slim/nets/mobilenet_v1.md).

(^#) More details on MobileNetV2 models can be found [here](nets/mobilenet/README.md).

(\*): Results quoted from the [paper](https://arxiv.org/abs/1603.05027).

Here is an example of how to download the Inception V3 checkpoint:

```shell
$ CHECKPOINT_DIR=./checkpoints
$ mkdir ${CHECKPOINT_DIR}
$ cd ${CHECKPOINT_DIR}
$ wget http://download.tensorflow.org/models/mobilenet_v1_2018_02_22/mobilenet_v1_1.0_224.tgz
$ tar -xvf mobilenet_v1_1.0_224.tgz
$ rm mobilenet_v1_1.0_224.tgz
```



# Training a model from scratch.
<a id='Training'></a>

We provide an easy way to train a model from scratch using any TF-Slim dataset.
The following example demonstrates how to train Inception V3 using the default
parameters on the ImageNet dataset.

```shell
TFRECORD_OUTPUT_DIR=/path/to/folder/tfrecord_dir
TRAIN_DIR=/output/train_dir  
python train_image_classifier.py \
    --train_dir=${TRAIN_DIR} \
    --dataset_name=blocks \
    --dataset_dir=${TFRECORD_OUTPUT_DIR} \
    --batch_size=64 \
    --dataset_split_name=train \
    --model_name=mobilenet_v1 \
    --train_image_size=224 \
    --max_number_of_steps=1000 \
```

This process may take several days, depending on your hardware setup.
For convenience, we provide a way to train a model on multiple GPUs,
and/or multiple CPUs, either synchrononously or asynchronously.
See [model_deploy](https://github.com/tensorflow/models/blob/master/research/slim/deployment/model_deploy.py)
for details.

### TensorBoard

To visualize the losses and other metrics during training, you can use
[TensorBoard](https://github.com/tensorflow/tensorboard)
by running the command below.

```shell
tensorboard --logdir=${TRAIN_DIR}
```

Once TensorBoard is running, navigate your web browser to http://localhost:6006.

# Fine-tuning a model from an existing checkpoint
<a id='Tuning'></a>

Rather than training from scratch, we'll often want to start from a pre-trained
model and fine-tune it.
To indicate a checkpoint from which to fine-tune, we'll call training with
the `--checkpoint_path` flag and assign it an absolute path to a checkpoint
file.

When fine-tuning a model, we need to be careful about restoring checkpoint
weights. In particular, when we fine-tune a model on a new task with a different
number of output labels, we wont be able restore the final logits (classifier)
layer. For this, we'll use the `--checkpoint_exclude_scopes` flag. This flag
hinders certain variables from being loaded. When fine-tuning on a
classification task using a different number of classes than the trained model,
the new model will have a final 'logits' layer whose dimensions differ from the
pre-trained model. For example, if fine-tuning an ImageNet-trained model on
Flowers, the pre-trained logits layer will have dimensions `[2048 x 1001]` but
our new logits layer will have dimensions `[2048 x 5]`. Consequently, this
flag indicates to TF-Slim to avoid loading these weights from the checkpoint.

Keep in mind that warm-starting from a checkpoint affects the model's weights
only during the initialization of the model. Once a model has started training,
a new checkpoint will be created in `${TRAIN_DIR}`. If the fine-tuning
training is stopped and restarted, this new checkpoint will be the one from which weights are restored and not the `${checkpoint_path}$`. Consequently, the flags `--checkpoint_path` and `--checkpoint_exclude_scopes` are only used during the `0-`th global step (model initialization). 

Typically for fine-tuning you only wants to train a sub-set of layers. The flag `--trainable_scopes` allows you to specify which subset of layers should be trained, and the rest would remain frozen. Where the `--trainable_scopes`  flag expects a string of comma separated variable names with no spaces. In addition you can specify all the trainable variables in a specific layer by only specifying the common name (name_scope) of the variables in that layer, as defined in the graph. For example, if you only want to train all the variables in the logits, Conv2d_13, and Conv2d_12 layers. You would set the `--trainable_scopes` argument as such


```shell

--trainable_scopes=MobilenetV1/Logits,MobilenetV1/Conv2d_13,MobilenetV1/Conv2d_12

```
The training script (`train_image_classifier.py`) will print out a list of all the trainable variable that are defined in the selected model graph `--model_name=mobilenet_v1` . Note that if the specified variable name(s) do not match the name(s) defined in the select model graph, that variable will be ignored with no error messages. Hence it is important to check that the provided variable names are correct, and that all the desired trainable variable have be selected. For the above example where we wanted to train the last three layers of the model (logits, Conv2d_13, and Conv2d_12 layers) you should get the follow trainable variable list as a screen print out when you run the (`train_image_classifier.py`) script.

```shell

######## List of all Trainable Variables ###########
 [<tf.Variable 'MobilenetV1/Logits/Conv2d_1c_1x1/weights:0' shape=(1, 1, 256, 16) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Logits/Conv2d_1c_1x1/biases:0' shape=(16,) dtype=float32_ref>
, <tf.Variable 'MobilenetV1/Conv2d_13_depthwise/depthwise_weights:0' shape=(3, 3, 256, 1) dtype=float32_ref>, <tf.Variable MobilenetV1/Conv2d_13_depthwise/BatchNorm/gamma:0' shape=(256,) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_13_depthwise/BatchNorm/beta:0' shape=(256,) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_13_pointwise/weights:0' shape=(1, 1, 256, 256) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_13_pointwise/BatchNorm/gamma:0' shape=(256,) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_13_pointwise/BatchNorm/beta:0'
shape=(256,) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_12_depthwise/depthwise_weights:0' shape=(3, 3, 128, 1) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_12_depthwise/BatchNorm/gamma:0' shape=(128,) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_12_depthwise/BatchNorm/beta:0' shape=(128,) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_12_pointwise/weights:0' shape=(1, 1, 128, 256) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_12_pointwise/BatchNorm/gamma:0' shape=(256,) dtype=float32_ref>, <tf.Variable 'MobilenetV1/Conv2d_12_pointwise/BatchNorm/beta:0' shape=(256,) dtype=float32_ref>]



```

Below we give an example of mobilenet_v1 that was trained on ImageNet with 1000 class labels, however, now we set `--datasetdir=${DATASET_DIR}`  to point to our custom dataset. Since the dataset is quite small we will only train the last two layers.


```shell

$ CHECKPOINT_PATH=./checkpoints/mobilenet_v1_0.25_224/mobilenet_v1_0.25_224.ckpt
$ python train_image_classifier.py \
    --train_dir=${TRAIN_DIR} \
    --dataset_dir=${DATASET_DIR} \
    --dataset_name=blocks \
    --batch_size=64 \
    --dataset_split_name=train \
    --model_name=mobilenet_v1_025 \
    --preprocessing_name=mobilenet_v1 \
    --train_image_size=224 \
    --max_number_of_steps=1000 \
    --checkpoint_path=${CHECKPOINT_PATH} \
    --checkpoint_exclude_scopes=MobilenetV1/Logits,MobilenetV1/AuxLogits \
    --trainable_scopes=MobilenetV1/Logits,MobilenetV1/AuxLogits \
    --clone_on_cpu=True
```
For training on cpu (with tensorflow package, instead of tensorflow-gpu), set flag `--clone_on_cpu` to `True`. For training on gpu, this flag can be ignored or set to `False`.

We suggest to use a different directory `TRAIN_DIR` is suggested to be in a different directory each time  


# Evaluating performance of a model while training
<a id='Eval'></a>

To evaluate the performance of a model (whether pretrained or your own),
you can use the eval_image_classifier.py script, as shown below.

The script should be run while training and `--checkpoint_path`  should point to the directory where the training job checkpoints are stored.

```shell

$ python eval_image_classifier.py \
    --alsologtostderr \
    --checkpoint_path=${TRAIN_DIR} \
    --dataset_dir=${TFRECORD_OUTPUT_DIR} \
    --dataset_name=blocks \
    --dataset_split_name=validation \
    --model_name=mobilenet_v1 \
    --preprocessing_name=mobilenet_v1 \
    --eval_image_size=224
```

See the [evaluation module example](https://github.com/tensorflow/tensorflow/tree/r1.13/tensorflow/contrib/slim#evaluation-loop)
for an example of how to evaluate a model at multiple checkpoints during or after the training.

# Exporting the Inference Graph
<a id='Export'></a>

Saves out a GraphDef containing the architecture of the model.

To use it with a model name defined by slim, run:

```shell

$ python export_inference_graph.py \
  --alsologtostderr \
  --model_name=mobilenet_v1 
  --dataset_dir=${TFRECORD_OUTPUT_DIR}  
  --output_file=${TRAIN_DIR}/inference_graph_mobilenet_v1.pb --dataset_name=blocks

```

## Freezing the exported Graph
If you then want to use the resulting model with your own or pretrained
checkpoints as part of a mobile model, you can run freeze_graph to get a graph
def with the variables inlined as constants using:

```shell
python freeze_graph.py \
  --input_graph=${TRAIN_DIR}/inference_graph_mobilenet_v1.pb  \
  --input_checkpoint=${TRAIN_DIR}/model.ckpt-1000 \
  --input_binary=true --output_graph=${TRAIN_DIR}/frozen_mobilenet_v1.pb \
  --output_node_names=MobilenetV1/Predictions/Reshape_1
```
[Note: The bazel commands were replaced with a python file. Same arguments were used.]
The output node names will vary depending on the model, but you can inspect and
estimate them using the summarize_graph tool:


# Troubleshooting and Current Known Issues
<a id='Troubleshooting'></a>

## Known issues that need tobe addressed;

* Tensorboard stops updating after `eval_image_classifier.py` script while training script is running.

* Add to readme;
    * Section on custom preprocessing scripts
    * Section on custom model scripts. 


#### The model runs out of CPU memory.

See
[Model Runs out of CPU memory](https://github.com/tensorflow/models/tree/r1.13/research/inception#the-model-runs-out-of-cpu-memory).

#### The model runs out of GPU memory.

See
[Adjusting Memory Demands](https://github.com/tensorflow/models/tree/r1.13/research/inception#adjusting-memory-demands).

#### The model training results in NaN's.

See
[Model Resulting in NaNs](https://github.com/tensorflow/models/tree/r1.13/research/inception#the-model-training-results-in-nans).

#### The ResNet and VGG Models have 1000 classes but the ImageNet dataset has 1001

The ImageNet dataset provided has an empty background class which can be used
to fine-tune the model to other tasks. If you try training or fine-tuning the
VGG or ResNet models using the ImageNet dataset, you might encounter the
following error:

```bash
InvalidArgumentError: Assign requires shapes of both tensors to match. lhs shape= [1001] rhs shape= [1000]
```
This is due to the fact that the VGG and ResNet V1 final layers have only 1000
outputs rather than 1001.

To fix this issue, you can set the `--labels_offset=1` flag. This results in
the ImageNet labels being shifted down by one:


#### I wish to train a model with a different image size.

The preprocessing functions all take `height` and `width` as parameters. You
can change the default values using the following snippet:

```python
image_preprocessing_fn = preprocessing_factory.get_preprocessing(
    preprocessing_name,
    height=MY_NEW_HEIGHT,
    width=MY_NEW_WIDTH,
    is_training=True)
```

#### What hardware specification are these hyper-parameters targeted for?

See
[Hardware Specifications](https://github.com/tensorflow/models/tree/master/research/inception#what-hardware-specification-are-these-hyper-parameters-targeted-for).
