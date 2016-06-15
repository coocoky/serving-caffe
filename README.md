#TensorFlow Serving + Caffe

__This is a fork of Tensorflow Serving (TFS), extended with support for the
[Caffe](http://caffe.berkeleyvision.org/) deep learning framework.
For more information about Tensorflow Serving, switch to the `master` branch,
or visit the Tensorflow Serving [website](https://tensorflow.github.io/serving/).__

---

## Summary

TensorFlow Serving is an open-source software library for serving
machine learning models. It deals with the *inference* aspect of machine
learning, taking models after *training* and managing their lifetimes, providing
clients with versioned access via a high-performance, reference-counted lookup
table.

## Setup, Build & test

Caffe has been integrated in to TFS build, and as such you should follow the
[TFS installation guide](https://github.com/tensorflow/serving/blob/master/tensorflow_serving/g3doc/setup.md)
first. At a minimum you need to install bazel and configure Tensorflow
(`cd tensorflow; ./configure`).

Next, you will need to install the Caffe prerequisites on your system. For a comprehensive guide, see the [Caffe Installation guide](http://caffe.berkeleyvision.org/installation.html#prerequisites). At a minimum, you
will need the following:

- `CMake >= 2.8.12`
- `Boost >= 1.55`
- `OpenBLAS`
- `gflags`
- `glog`
- `lmdb`

__Note:__ installing protobuf is not required; the build will adopt protobuf from the `tensorflow/google/protobuf` submodule.

To validate the Caffe build, run the following bazel command. This will retrieve Caffe
from Github and build Caffe in cpu mode:

    > bazel build //external:caffe

### GPU Support (optional, linux only)

The Caffe build adopts the CUDA configuration from the Tensorflow build, and as such
will use the version (and location) of cudnn, and the standard cuda libraries you specified when you configured Tensorflow. You can validate this configuration by building Caffe with CUDA:

    > bazel build -c opt --config=cuda //external:caffe

For more information on installing the CUDA libraries and configuring Tensorflow, read
the Tensorflow setup guide [here](https://www.tensorflow.org/versions/r0.8/get_started/os_setup.html#optional-install-cuda-gpus-on-linux).

### Tests

To run tests related to the Caffe servable implementation, run:

    > bazel test tensorflow_serving/servables/caffe/...


## Examples

The `mnist_inference` example service implementations have been altered in this fork to run with the Caffe runtime when built with the `--define=runtime=caffe` option; making it easy to see the required changes to serve with Caffe rather than Tensorflow. Typically less than 10 lines of code need altering.

### MNIST handwriting recognition

#### 1. Download the pre-trained caffe model:

    > bazel run //tensorflow_serving/servables/caffe/example:mnist_caffe_fetch -- \
        --version 1 /tmp/mnist_export_caffe

There's nothing special about this pretrained model, and it can be re-generated by following Caffe's LeNet MNIST Tutorial [here](https://github.com/BVLC/caffe/tree/master/examples/mnist).

The contents of any pretrained model must include `deploy.prototxt` `weights.caffemodel` files, which as you could imagine, contain the deployable model definition and a single training snapshot.

#### 2. Build and run an MNIST service:

Select one of the two mnist services to build and run. Ideally, you should be familiar with the TFS mnist tutorials ([Basic](tensorflow_serving/g3doc/serving_basic.md), [Advanced](tensorflow_serving/g3doc/serving_advanced.md)) before serving Caffe models with TFS. Equivalent instructions for serving the mnist examples with caffe follow:

##### 2.a Basic service

    > bazel build -c opt --define=runtime=caffe //tensorflow_serving/example:mnist_inference
    > ./bazel-bin/tensorflow_serving/example/mnist_inference --port=9000 /tmp/mnist_export_caffe/00000001/

##### 2.b Advanced service

    > bazel build -c opt --define=runtime=caffe //tensorflow_serving/example:mnist_inference_2
    > ./bazel-bin/tensorflow_serving/example/mnist_inference_2 --port=9000 /tmp/mnist_export_caffe

#### 3. Build and run the client

    > bazel build //tensorflow_serving/example:mnist_client
    > bazel-bin/tensorflow_serving/example/mnist_client \
        --num_tests=1000 --server=localhost:9000 --concurrency=10
    ....
    Inference error rate: 1.2%
    Request error rate: 0.0%
    Avg. Throughput: 197.192047438 reqs/s
    Request Latency (percentiles):
      50th ....... 46ms
      90th ....... 62ms
      99th ....... 83ms


## FAQ

### How do I use my own Fork of Caffe?

If you intend to use a fork of Caffe which contains (for example) custom layers, you can alter the `WORKSPACE` file to point to the file/git location of this fork.

__Note:__ To ensure any custom layer is picked up by the build, you currently will need to alter the `caffe.BUILD` to include the name of that layer in the variable `CAFFE_WELL_KNOWN_LAYERS`.


## Misc. Development notes

- The Caffe Servable is implemented in `serving/servables/caffe` and is based on the Tensorflow servable.

- To be able to reuse as much of the TFS infastructure as possible (e.g. batching), and to be able to create server frontends which can be switched to/from Caffe and Tensorflow with minimum effort, the core Caffe servable, the `CaffeServingSession`, derives from the abstract `tensorflow::serving::ServingSession` class, essentially encapsulating the Caffe model as though it were a Tensorflow one.

---

__(C) Copyright IBM Corp. / Google Inc. 2016. All Rights Reserved.__