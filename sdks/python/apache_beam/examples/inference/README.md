<!--
    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.
-->

# Example RunInference API pipelines

This module contains example pipelines that use the Beam RunInference
API. <!---TODO: Add link to full documentation on Beam website when it's published.-->

## Prerequisites

You must have `apache-beam>=2.40.0` or greater installed in order to run these pipelines,
because the `apache_beam.examples.inference` module was added in that release.
```
pip install apache-beam==2.40.0
```

### PyTorch dependencies

The following installation requirements are for the files used in these examples.

The RunInference API supports the PyTorch framework. To use PyTorch locally, first install `torch`.
```
pip install torch==1.10.0
```

If you are using pretrained models from Pytorch's `torchvision.models` [subpackage](https://pytorch.org/vision/0.12/models.html#models-and-pre-trained-weights), you also need to install `torchvision`.
```
pip install torchvision
```

If you are using pretrained models from Hugging Face's `transformers` [package](https://huggingface.co/docs/transformers/index), you also need to install `transformers`.
```
pip install transformers
```

For installation of the `torch` dependency on a distributed runner such as Dataflow, refer to the
[PyPI dependency instructions](https://beam.apache.org/documentation/sdks/python-pipeline-dependencies/#pypi-dependencies).


### TensorRT dependencies

The RunInference API supports TensorRT SDK for high-performance deep learning inference with NVIDIA GPUs.
To use TensorRT locally, we suggest an environment with TensorRT >= 8.0.1. Install TensorRT as per the
[TensorRT Install Guide](https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html). You
will need to make sure the Python bindings for TensorRT are also installed correctly, these are available by installing the python3-libnvinfer and python3-libnvinfer-dev packages on your TensorRT download.

If you would like to use Docker, you can use an NGC image like:
```
docker pull nvcr.io/nvidia/tensorrt:22.04-py3
```
as an existing container base to [build custom Apache Beam container](https://beam.apache.org/documentation/runtime/environments/#modify-existing-base-image).

### Additional resources
For more information, see the
[Machine Learning](/documentation/sdks/python-machine-learning) and the
[RunInference transform](/documentation/transforms/python/elementwise/runinference) documentation.

---
## Image classification

[`pytorch_image_classification.py`](./pytorch_image_classification.py) contains an implementation for a RunInference pipeline that performs image classification using the `mobilenet_v2` architecture.

The pipeline reads the images, performs basic preprocessing, passes the images to the PyTorch implementation of RunInference, and then writes the predictions to a text file.

### Dataset and model for image classification

To use this transform, you need a dataset and model for image classification.

1. Create a directory named `IMAGES_DIR`. Create or download images and put them in this directory. The directory is not required if image names in the input file `IMAGE_FILE_NAMES.txt` you create in step 2 have absolute paths.
One popular dataset is from [ImageNet](https://www.image-net.org/). Follow their instructions to download the images.
2. Create a file named `IMAGE_FILE_NAMES.txt` that contains the absolute paths of each of the images in `IMAGES_DIR` that you want to use to run image classification. The path to the file can be different types of URIs such as your local file system, an AWS S3 bucket, or a GCP Cloud Storage bucket. For example:
```
/absolute/path/to/image1.jpg
/absolute/path/to/image2.jpg
```
3. Download the [mobilenet_v2](https://pytorch.org/vision/stable/_modules/torchvision/models/mobilenetv2.html) model from Pytorch's repository of pretrained models. This model requires the torchvision library. To download this model, run the following commands from a Python shell:
```
import torch
from torchvision.models import mobilenet_v2
model = mobilenet_v2(pretrained=True)
torch.save(model.state_dict(), 'mobilenet_v2.pth') # You can replace mobilenet_v2.pth with your preferred file name for your model state dictionary.
```

### Running `pytorch_image_classification.py`

To run the image classification pipeline locally, use the following command:
```sh
python -m apache_beam.examples.inference.pytorch_image_classification \
  --input IMAGE_FILE_NAMES \
  --images_dir IMAGES_DIR \
  --output OUTPUT \
  --model_state_dict_path MODEL_STATE_DICT
```
`images_dir` is only needed if your `IMAGE_FILE_NAMES.txt` file contains relative paths (they will be relative from `IMAGES_DIR`).

For example, if you've followed the naming conventions recommended above:
```sh
python -m apache_beam.examples.inference.pytorch_image_classification \
  --input IMAGE_FILE_NAMES.txt \
  --output predictions.csv \
  --model_state_dict_path mobilenet_v2.pth
```
This writes the output to the `predictions.csv` with contents like:
```
/absolute/path/to/image1.jpg;1
/absolute/path/to/image2.jpg;333
...
```

Each image path is paired with a value representing the Imagenet class that returned the highest confidence score out of Imagenet's 1000 classes.

---
## Image segmentation

[`pytorch_image_segmentation.py`](./pytorch_image_segmentation.py) contains an implementation for a RunInference pipeline that performs image segementation using the `maskrcnn_resnet50_fpn` architecture.

The pipeline reads images, performs basic preprocessing, passes the images to the PyTorch implementation of RunInference, and then writes predictions to a text file.

### Dataset and model for image segmentation

To use this transform, you need a dataset and model for image segmentation.

1. Create a directory named `IMAGES_DIR`. Create or download images and put them in this directory. The directory is not required if image names in the input file `IMAGE_FILE_NAMES.txt` you create in step 2 have absolute paths.
A popular dataset is from [Coco](https://cocodataset.org/#home). Follow their instructions to download the images.
2. Create a file named `IMAGE_FILE_NAMES.txt` that contains the absolute paths of each of the images in `IMAGES_DIR` that you want to use to run image segmentation. The path to the file can be different types of URIs such as your local file system, an AWS S3 bucket, or a GCP Cloud Storage bucket. For example:
```
/absolute/path/to/image1.jpg
/absolute/path/to/image2.jpg
```
3. Download the [maskrcnn_resnet50_fpn](https://pytorch.org/vision/0.12/models.html#id70) model from Pytorch's repository of pretrained models. This model requires the torchvision library. To download this model, run the following commands from a Python shell:
```
import torch
from torchvision.models.detection import maskrcnn_resnet50_fpn
model = maskrcnn_resnet50_fpn(pretrained=True)
torch.save(model.state_dict(), 'maskrcnn_resnet50_fpn.pth') # You can replace maskrcnn_resnet50_fpn.pth with your preferred file name for your model state dictionary.
```
4. Note the path to the `OUTPUT` file. This file is used by the pipeline to write the predictions.

### Running `pytorch_image_segmentation.py`

To run the image segmentation pipeline locally, use the following command:
```sh
python -m apache_beam.examples.inference.pytorch_image_segmentation \
  --input IMAGE_FILE_NAMES \
  --images_dir IMAGES_DIR \
  --output OUTPUT \
  --model_state_dict_path MODEL_STATE_DICT
```
`images_dir` is only needed if your `IMAGE_FILE_NAMES.txt` file contains relative paths (they will be relative from `IMAGES_DIR`).

For example, if you've followed the naming conventions recommended above:
```sh
python -m apache_beam.examples.inference.pytorch_image_segmentation \
  --input IMAGE_FILE_NAMES.txt \
  --output predictions.csv \
  --model_state_dict_path maskrcnn_resnet50_fpn.pth
```
This writes the output to the `predictions.csv` with contents like:
```
/absolute/path/to/image1.jpg;['parking meter', 'bottle', 'person', 'traffic light', 'traffic light', 'traffic light']
/absolute/path/to/image2.jpg;['bottle', 'person', 'person']
...
```
Each line has data separated by a semicolon ";". The first item is the file name. The second item is a list of predicted instances.

---
## Object Detection

[`tensorrt_object_detection.py`](./tensorrt_object_detection.py) contains an implementation for a RunInference pipeline that performs object detection using [Tensorflow Object Detection's](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf2_detection_zoo.md) SSD MobileNet v2 320x320 architecture.

The pipeline reads the images, performs basic preprocessing, passes them to the TensorRT implementation of RunInference, and then writes the predictions to a text file.

### Dataset and model for image classification

You will need to create or download images, and place them into your `IMAGES_DIR` directory. Popular dataset for such task is [COCO dataset](https://cocodataset.org/#home). COCO validation dataset can be obtained [here](http://images.cocodataset.org/zips/val2017.zip).
- **Required**: A path to a file called `IMAGE_FILE_NAMES` that contains the absolute paths of each of the images in `IMAGES_DIR` on which you want to run image segmentation. Paths can be different types of URIs such as your local file system, a AWS S3 bucket or GCP Cloud Storage bucket. For example:
```
/absolute/path/to/000000000139.jpg
/absolute/path/to/000000289594.jpg
```
- **Required**: A path to a file called `TRT_ENGINE` that contains the pre-built TensorRT engine from SSD MobileNet v2 320x320 model. You will need to [follow instructions](https://github.com/NVIDIA/TensorRT/tree/main/samples/python/tensorflow_object_detection_api) on how to download and convert this SSD model into TensorRT engine. At [Create ONNX Graph](https://github.com/NVIDIA/TensorRT/tree/main/samples/python/tensorflow_object_detection_api#create-onnx-graph) step, keep batch size at 1. As soon as you are done with [Build TensorRT Engine](https://github.com/NVIDIA/TensorRT/tree/main/samples/python/tensorflow_object_detection_api#build-tensorrt-engine) step. You can use resulted engine as `TRT_ENGINE` input. In addition, make sure that environment you use for TensorRT engine creation is the same environment you use to run TensorRT inference. It is related not only to TensorRT version, but also to a specific GPU used. Read more about it [here](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#compatibility-serialized-engines).

- **Required**: A path to a file called `OUTPUT`, to which the pipeline will write the predictions.
- **Optional**: `IMAGES_DIR`, which is the path to the directory where images are stored. Not required if image names in the input file `IMAGE_FILE_NAMES` have absolute paths.

### Running `tensorrt_object_detection.py`

To run the image classification pipeline locally, use the following command:
```sh
python -m apache_beam.examples.inference.tensorrt_object_detection \
  --input IMAGE_FILE_NAMES \
  --images_dir IMAGES_DIR \
  --output OUTPUT \
  --engine_path TRT_ENGINE
```
For example:
```sh
python -m apache_beam.examples.inference.tensorrt_object_detection \
  --input image_file_names.txt \
  --output predictions.csv \
  --engine_path ssd_mobilenet_v2_320x320_coco17_tpu-8.trt
```
This writes the output to the `predictions.csv` with contents like:
```
/absolute/path/to/000000000139.jpg;[{'ymin': '217.31875205039978'	 'xmin': '295.93122482299805'	 'ymax': '315.90323209762573'	 'xmax': '357.8959655761719'	 'score': '0.72342616'	 'class': 'chair'}	 {'ymin': '166.81788557767868'.....

/absolute/path/to/000000289594.jpg;[{'ymin': '227.25109100341797'	 'xmin': '331.7402381300926'	 'ymax': '476.88533782958984'	 'xmax': '402.2928895354271'	 'score': '0.77217317'	 'class': 'person'}	 {'ymin': '231.8712615966797'	 'xmin': '292.8590789437294'.....
...
```
Each line has data separated by a semicolon ";". The first item is the file name. The second item is a list of dictionaries, where each dictionary corresponds with a single detection. A detection contains: box coordinates (ymin, xmin, ymax, xmax); score; and class.

---
## Language modeling

[`pytorch_language_modeling.py`](./pytorch_language_modeling.py) contains an implementation for a RunInference pipeline that performs masked language modeling (that is, decoding a masked token in a sentence) using the `BertForMaskedLM` architecture from Hugging Face.

The pipeline reads sentences, performs basic preprocessing to convert the last word into a `[MASK]` token, passes the masked sentence to the PyTorch implementation of RunInference, and then writes the predictions to a text file.

### Dataset and model for language modeling

To use this transform, you need a dataset and model for language modeling.

1. Download the [BertForMaskedLM](https://huggingface.co/docs/transformers/model_doc/bert#transformers.BertForMaskedLM) model from Hugging Face's repository of pretrained models. You must already have `transformers` installed, then from a Python shell run:
```
import torch
from transformers import BertForMaskedLM
model = BertForMaskedLM.from_pretrained('bert-base-uncased', return_dict=True)
torch.save(model.state_dict(), 'BertForMaskedLM.pth') # You can replace BertForMaskedLM.pth with your preferred file name for your model state dictionary.
```
2. (Optional) Create a file named `SENTENCES.txt` that contains sentences to feed into the model. The content of the file should be similar to the following example:
```
The capital of France is Paris .
He looked up and saw the sun and stars .
...
```

### Running `pytorch_language_modeling.py`

To run the language modeling pipeline locally, use the following command:
```sh
python -m apache_beam.examples.inference.pytorch_language_modeling \
  --input SENTENCES \
  --output OUTPUT \
  --model_state_dict_path MODEL_STATE_DICT
```
The `input` argument is optional. If none is provided, it will run the pipeline with some
example sentences.

For example, if you've followed the naming conventions recommended above:
```sh
python -m apache_beam.examples.inference.pytorch_language_modeling \
  --input SENTENCES.txt \
  --output predictions.csv \
  --model_state_dict_path BertForMaskedLM.pth
```
Or, using the default example sentences:
```sh
python -m apache_beam.examples.inference.pytorch_language_modeling \
  --output predictions.csv \
  --model_state_dict_path BertForMaskedLM.pth
```

This writes the output to the `predictions.csv` with contents like:
```
The capital of France is Paris .;paris
He looked up and saw the sun and stars .;moon
...
```
Each line has data separated by a semicolon ";".
The first item is the input sentence. The model masks the last word and tries to predict it;
the second item is the word that the model predicts for the mask.

---
## MNIST digit classification
[`sklearn_mnist_classification.py`](./sklearn_mnist_classification.py) contains an implementation for a RunInference pipeline that performs image classification on handwritten digits from the [MNIST](https://en.wikipedia.org/wiki/MNIST_database) database.

The pipeline reads rows of pixels corresponding to a digit, performs basic preprocessing, passes the pixels to the Scikit-learn implementation of RunInference, and then writes the predictions to a text file.

### Dataset and model for language modeling

To use this transform, you need a dataset and model for language modeling.

1. Create a file named `INPUT.csv` that contains labels and pixels to feed into the model. Each row should have comma-separated elements. The first element is the label. All other elements are pixel values. The csv should not have column headers. The content of the file should be similar to the following example:
```
1,0,0,0...
0,0,0,0...
1,0,0,0...
4,0,0,0...
...
```
2. Create a file named `MODEL_PATH` that contains the pickled file of a scikit-learn model trained on MNIST data. Please refer to this scikit-learn [model persistence documentation](https://scikit-learn.org/stable/model_persistence.html) on how to serialize models.


### Running `sklearn_mnist_classification.py`

To run the MNIST classification pipeline locally, use the following command:
```sh
python -m apache_beam.examples.inference.sklearn_mnist_classification.py \
  --input INPUT \
  --output OUTPUT \
  --model_path MODEL_PATH
```
For example:
```sh
python -m apache_beam.examples.inference.sklearn_mnist_classification.py \
  --input INPUT.csv \
  --output predictions.txt \
  --model_path mnist_model_svm.pickle
```

This writes the output to the `predictions.txt` with contents like:
```
1,1
4,9
7,1
0,0
...
```
Each line has data separated by a comma ",". The first item is the actual label of the digit. The second item is the predicted label of the digit.

### Running `sklearn_japanese_housing_regression.py`

#### Getting the data:
Data for this example can be found at:
https://www.kaggle.com/datasets/nishiodens/japan-real-estate-transaction-prices

#### Models:
Prebuilt sklearn pipelines are hosted at:
https://storage.cloud.google.com/apache-beam-ml/models/japanese_housing/

Note: This example uses more than one model. Since not all features in an example are populated, a different model will be chosen based on available data.

For example, an example without distance to the nearest station will use a model that doesn't rely on that data.

#### Running the Pipeline
To run locally, use the following command:
```sh
python -m apache_beam.examples.inference.sklearn_japanese_housing_regression.py \
  --input_file INPUT \
  --output OUTPUT \
  --model_path MODEL_PATH
```
For example:
```sh
python -m apache_beam.examples.inference.sklearn_japanese_housing_regression.py \
  --input_file housing_examples.csv \
  --output predictions.txt \
  --model_path https://storage.cloud.google.com/apache-beam-ml/models/japanese_housing/
```

This writes the output to the `predictions.txt` with contents like:
```
True Price 40000000.0, Predicted Price 34645912.039208
True Price 34000000.0, Predicted Price 28648634.135857
True Price 31000000.0, Predicted Price 25654277.256461
...
```

