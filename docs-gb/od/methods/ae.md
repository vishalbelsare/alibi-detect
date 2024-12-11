# Auto-Encoder

## Overview

The Auto-Encoder (AE) outlier detector is first trained on a batch of unlabeled, but normal (inlier) data. Unsupervised training is desireable since labeled data is often scarce. The AE detector tries to reconstruct the input it receives. If the input data cannot be reconstructed well, the reconstruction error is high and the data can be flagged as an outlier. The reconstruction error is  measured as the mean squared error (MSE) between the input and the reconstructed instance.

## Usage

### Initialize

Parameters:

* `threshold`: threshold value above which the instance is flagged as an outlier.

* `encoder_net`: `tf.keras.Sequential` instance containing the encoder network. Example:

```python
encoder_net = tf.keras.Sequential(
  [
      InputLayer(input_shape=(32, 32, 3)),
      Conv2D(64, 4, strides=2, padding='same', activation=tf.nn.relu),
      Conv2D(128, 4, strides=2, padding='same', activation=tf.nn.relu),
      Conv2D(512, 4, strides=2, padding='same', activation=tf.nn.relu)
  ])
```

* `decoder_net`: `tf.keras.Sequential` instance containing the decoder network. Example:

```python
decoder_net = tf.keras.Sequential(
  [
      InputLayer(input_shape=(1024,)),
      Dense(4*4*128),
      Reshape(target_shape=(4, 4, 128)),
      Conv2DTranspose(256, 4, strides=2, padding='same', activation=tf.nn.relu),
      Conv2DTranspose(64, 4, strides=2, padding='same', activation=tf.nn.relu),
      Conv2DTranspose(3, 4, strides=2, padding='same', activation='sigmoid')
  ])
```

* `ae`: instead of using a separate encoder and decoder, the AE can also be passed as a `tf.keras.Model`.

* `data_type`: can specify data type added to metadata. E.g. *'tabular'* or *'image'*.

Initialized outlier detector example:

```python
from alibi_detect.od import OutlierAE

od = OutlierAE(threshold=0.1,
               encoder_net=encoder_net,
               decoder_net=decoder_net)
```

### Fit

We then need to train the outlier detector. The following parameters can be specified:

* `X`: training batch as a numpy array of preferably normal data.

* `loss_fn`: loss function used for training. Defaults to the Mean Squared Error loss.

* `optimizer`: optimizer used for training. Defaults to [Adam](https://arxiv.org/abs/1412.6980) with learning rate 1e-3.

* `epochs`: number of training epochs.

* `batch_size`: batch size used during training.

* `verbose`: boolean whether to print training progress.

* `log_metric`: additional metrics whose progress will be displayed if verbose equals True.


```python
od.fit(X_train, epochs=50)
```

It is often hard to find a good threshold value. If we have a batch of normal and outlier data and we know approximately the percentage of normal data in the batch, we can infer a suitable threshold:

```python
od.infer_threshold(X, threshold_perc=95)
```

### Detect

We detect outliers by simply calling `predict` on a batch of instances `X`. Detection can be customized via the following parameters:

* `outlier_type`: either *'instance'* or *'feature'*. If the outlier type equals *'instance'*, the outlier score at the instance level will be used to classify the instance as an outlier or not. If *'feature'* is selected, outlier detection happens at the feature level (e.g. by pixel in images).

* `outlier_perc`: percentage of the sorted (descending) feature level outlier scores. We might for instance want to flag an image as an outlier if at least 20% of the pixel values are on average above the threshold. In this case, we set `outlier_perc` to 20. The default value is 100 (using all the features).

* `return_feature_score`: boolean whether to return the feature level outlier scores.

* `return_instance_score`: boolean whether to return the instance level outlier scores.

The prediction takes the form of a dictionary with `meta` and `data` keys. `meta` contains the detector's metadata while `data` is also a dictionary which contains the actual predictions stored in the following keys:

* `is_outlier`: boolean whether instances or features are above the threshold and therefore outliers. If `outlier_type` equals *'instance'*, then the array is of shape *(batch size,)*. If it equals *'feature'*, then the array is of shape *(batch size, instance shape)*.

* `feature_score`: contains feature level scores if `return_feature_score` equals True.

* `instance_score`: contains instance level scores if `return_instance_score` equals True.


```python
preds = od.predict(X,
                   outlier_type='instance',
                   outlier_perc=75,
                   return_feature_score=True,
                   return_instance_score=True)
```

## Examples

### Image

[Outlier detection on CIFAR10](../../examples/od_ae_cifar10.md)
