---
layout: post
title:  "DenseNet and Deconvolution."
date:   2020-02-21 14:34:33 -0700
categories: update
---

**A Sparse-View CT Reconstruction Method Based on Combination of DenseNet and Deconvolution**

Didn't get good results.

I am trying to reimplement the paper below in keras

paper: https://ieeexplore.ieee.org/document/8331861
github: https://github.com/zzc623/DD_Net


```python
from google.colab import drive
drive.mount('/content/drive')
```

    Go to this URL in a browser: https://accounts.google.com/o/oauth2/auth?client_id=947318989803-6bn6qk8qdgf4n4g3pfee6491hc0brc4i.apps.googleusercontent.com&redirect_uri=urn%3aietf%3awg%3aoauth%3a2.0%3aoob&response_type=code&scope=email%20https%3a%2f%2fwww.googleapis.com%2fauth%2fdocs.test%20https%3a%2f%2fwww.googleapis.com%2fauth%2fdrive%20https%3a%2f%2fwww.googleapis.com%2fauth%2fdrive.photos.readonly%20https%3a%2f%2fwww.googleapis.com%2fauth%2fpeopleapi.readonly

    Enter your authorization code:
    ··········
    Mounted at /content/drive


paper: https://ieeexplore.ieee.org/document/8331861
github: https://github.com/zzc623/DD_Net


```python
import os
import keras
import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.image as mpimg
from keras.layers import Conv2D, MaxPooling2D, Input, Dense, UpSampling2D
from keras.layers import BatchNormalization, Activation, Dropout, Subtract
from keras.models import Model
from glob import glob
from keras.layers.convolutional import Conv2DTranspose
from keras.layers import concatenate
```

    Using TensorFlow backend.



<p style="color: red;">
The default version of TensorFlow in Colab will soon switch to TensorFlow 2.x.<br>
We recommend you <a href="https://www.tensorflow.org/guide/migrate" target="_blank">upgrade</a> now
or ensure your notebook will continue to use TensorFlow 1.x via the <code>%tensorflow_version 1.x</code> magic:
<a href="https://colab.research.google.com/notebooks/tensorflow_version.ipynb" target="_blank">more info</a>.</p>




```python
# %tensorflow_version 1.x
# import tensorflow as tf
print(tf.__version__)
```

    1.15.0


If we have weights we canload them: loaded_model.load_weights("model.h5")



```python
# print(len(os.listdir('/content/drive/My Drive/Colab Notebooks/CT_data/sparseview_60/train')))
# print(len(os.listdir('/content/drive/My Drive/Colab Notebooks/CT_data/ndct/train')))
# # 3600
# # 3600
```


```python
# print(len(os.listdir('/content/drive/My Drive/Colab Notebooks/CT_data/sparseview_60/test/')))
# print(len(os.listdir('/content/drive/My Drive/Colab Notebooks/CT_data/ndct/test/')))
# # # 354
# # # 354
```


```python
ndct = sorted(glob('/content/drive/My Drive/Colab Notebooks/CT_data/ndct/train/*'))
ldct = sorted(glob('/content/drive/My Drive/Colab Notebooks/CT_data/sparseview_60/train/*'))

ndct_test = sorted(glob('/content/drive/My Drive/Colab Notebooks/CT_data/ndct/test/*'))
ldct_test = sorted(glob('/content/drive/My Drive/Colab Notebooks/CT_data/sparseview_60/test/*'))

print(len(ndct))
print(len(ldct))
print(len(ndct_test))
print(len(ldct_test))
```

    3600
    3600
    354
    354


The formulas below will be used to calculate the quality of the reconstruction. Higher PSNR generally indicates high quality of reconstruction.


```python
def cal_psnr(im1, im2):
    # assert pixel value range is 0-255 and type is uint8
    mse = ((im1.astype(np.float) - im2.astype(np.float)) ** 2).mean()
    maxval = np.amax(im1)
    psnr = 10 * np.log10(maxval ** 2 / mse)
    return psnr

def tf_psnr(im1, im2):
    # assert pixel value range is 0-1
    #mse = tf.losses.mean_squared_error(labels=im2 * 255.0, predictions=im1 * 255.0)
    mse = tf.compat.v1.losses.mean_squared_error(labels=im2 * 255.0, predictions=im1 * 255.0)
    return 10.0 * (tf.log(255.0 ** 2 / mse) / tf.log(10.0))
```

Using less data: #for i in range(0, 600). Have 3600 in all.
Processing 3600 images takes aprox. 20 minutes to run. But once we create .npy aray we don't have to rerun this code in the future and we will have .npy form of our data. Colab has 11GB RAM limit.


```python
ndct_imgs_train = []
# for i in range(0, len(ndct)):                                                                                                                                      
for i in range(0, 600):
    f = open(ndct[i],'rb')
    a = np.fromfile(f, np.float32)
    ndct_imgs_train.append(a)
    f.close()
print("len(ndct_imgs_train)....: ",len(ndct_imgs_train))
#len(ndct_imgs_train)....:  3600                                                                                                                                                         
```


Using different range to use less data to train #for i in range(0, 600). In all have 3600 images. It takes aprox. 20 minutes to process all 3600. But once we create .npy aray we don't have to rerun this code in the future and we will have .npy form of our data.


```python
ldct_imgs_train = []
# for i in range(0, len(ldct)):
for i in range(0, 600):
    f = open(ldct[i],'rb')
    a = np.fromfile(f, np.float32)
    ldct_imgs_train.append(a)
    f.close()
print("len(ldct_imgs_train)....: ",len(ldct_imgs_train))
```

only using 100 images to test


```python
ndct_imgs_test = []
# for i in range(0, len(ndct_test)):
for i in range(0, 100):
    f = open(ndct_test[i],'rb')
    a = np.fromfile(f, np.float32)
    ndct_imgs_test.append(a)
    f.close()
print("len(ndct_imgs_test)....: ",len(ndct_imgs_test))

```

only using 100 images to test


```python
# load the image
ldct_imgs_test = []
# for i in range(0, len(ldct_test)):
for i in range(0, 100):
    f = open(ldct_test[i],'rb')
    a = np.fromfile(f, np.float32)
    ldct_imgs_test.append(a)
    f.close()
print("len(ldct_imgs_test)....: ",len(ldct_imgs_test))

```

Must reshape images to train


```python
ldct_train = np.asarray(ldct_imgs_train)
ndct_train = np.asarray(ndct_imgs_train)

ldct_train = ldct_train.reshape(600,512,512,1)
ndct_train = ndct_train.reshape(600,512,512,1)

ldct_test = np.asarray(ldct_imgs_test)
ndct_test = np.asarray(ndct_imgs_test)

# ldct_test = ldct_test.reshape(len(ldct_imgs_test),512,512,1)
# ndct_test = ndct_test.reshape(len(ldct_imgs_test),512,512,1)

ldct_test = ldct_test.reshape(100,512,512,1)
ndct_test = ndct_test.reshape(100,512,512,1)

print(ldct_train.shape)
print(ndct_train.shape)
print(ldct_test.shape)
print(ndct_test.shape)

```


```python
# np.save('sparseview_60_train_600', ldct_train) # save the file as "sparseview_60_train.npy"
# np.save('ndct_train_600', ndct_train) # save the file as "ndct_train.npy"

# np.save('sparseview_60_test_100', ldct_test) # save the file as "sparseview_60_test.npy"
# np.save('ndct_test_100', ndct_test) # save the file as "ndct_test.npy"


np.save('/content/drive/My Drive/Colab Notebooks/dd_net/sparseview_60_train_600', ldct_train) # save the file as "sparseview_60_train.npy"
np.save('/content/drive/My Drive/Colab Notebooks/dd_net/ndct_train_600', ndct_train) # save the file as "ndct_train.npy"

np.save('/content/drive/My Drive/Colab Notebooks/dd_net/sparseview_60_test_100', ldct_test) # save the file as "sparseview_60_test.npy"
np.save('/content/drive/My Drive/Colab Notebooks/dd_net/ndct_test_100', ndct_test) # save the file as "ndct_test.npy"
```


```python
sparseview_60_train = np.load('/content/drive/My Drive/Colab Notebooks/dd_net/sparseview_60_train_600.npy') # loads saved array into variable sparseview_60_train.
ndct_train = np.load('/content/drive/My Drive/Colab Notebooks/dd_net/ndct_train_600.npy') # loads saved array into variable ndct_train.
sparseview_60_test = np.load('/content/drive/My Drive/Colab Notebooks/dd_net/sparseview_60_test_100.npy') # loads saved array into variable sparseview_60_test.
ndct_test = np.load('/content/drive/My Drive/Colab Notebooks/dd_net/ndct_test_100.npy') # loads saved array into variable ndct_test.

# sparseview_60_train = np.load('sparseview_60_train_600.npy') # loads saved array into variable sparseview_60_train.
# ndct_train = np.load('ndct_train_600.npy') # loads saved array into variable ndct_train.
# sparseview_60_test = np.load('sparseview_60_test_100.npy') # loads saved array into variable sparseview_60_test.
# ndct_test = np.load('ndct_test_100.npy') # loads saved array into variable ndct_test.

```


```python
def denseblock(input):
  # - L1
  d2_1 = BatchNormalization()(input)
  d2_1 = Activation('relu')(d2_1)
  d2_1 = Conv2D(16, (5, 5), padding='same', use_bias=False, strides=(1, 1))(d2_1)
  d2_1 = concatenate([input, d2_1])

  # - L2
  d2_2 = BatchNormalization()(d2_1)
  d2_2 = Activation('relu')(d2_2)
  d2_2= Conv2D(16, (5, 5), padding='same', use_bias=False, strides=(1, 1))(d2_2)
  d2_2 = concatenate([input, d2_1, d2_2])

  # - L3
  d2_3 = BatchNormalization()(d2_2)
  d2_3 = Activation('relu')(d2_3)
  d2_3 = Conv2D(16, (5, 5), padding='same', use_bias=False, strides=(1, 1))(d2_3)
  d2_3 = concatenate([input, d2_1, d2_2, d2_3])

  # - L4
  d2_4 = Conv2D(16, (5, 5), padding='same', use_bias=False, strides=(1, 1))(d2_3)
  d2_4 = BatchNormalization()(d2_4)
  d2_4 = Activation('relu')(d2_4)
  d2_4= Conv2D(16, (5, 5), padding='same', use_bias=False, strides=(1, 1))(d2_4)
  d2_4 = concatenate([input, d2_1, d2_2, d2_3, d2_4])
  return d2_4
```


```python
inputs = Input((None, None,1))
# ---A1 Layer-----------------------
a1 = Conv2D(16, (7, 7), padding='same', use_bias=False, strides=(1, 1))(inputs)
a1 = MaxPooling2D((2, 2)) (a1)
# images 256 X 256
d1 = denseblock(a1)

a1 = BatchNormalization()(d1)
a1 = Activation('relu')(a1)
a1 = Conv2D(16, (1, 1), strides=(1, 1)) (a1)

# ----A2 Layer---------------------
a2 = MaxPooling2D((2, 2)) (a1)
# images 128 X 128
d2 = denseblock(a2)

a2 = BatchNormalization()(d2)
a2 = Activation('relu')(a2)
a2 = Conv2D(16, (1, 1), strides=(1, 1)) (a2)

# # ----A3 Layer----------------------
a3 = MaxPooling2D((2, 2)) (a2)
# images 64 X 64
d3 = denseblock(a3)

a3 = BatchNormalization()(d3)
a3 = Activation('relu')(a3)
a3 = Conv2D(16, (1, 1), strides=(1, 1)) (a3)

# ----A4 Layer----------------------
a4 = MaxPooling2D((2, 2)) (a3)
# images 32 X 3
d4 = denseblock(a4)

a4 = BatchNormalization()(d4)
a4 = Activation('relu')(a4)
a4 = Conv2D(16, (1, 1), strides=(1, 1)) (a4)

# #----B1 Layer-----------------------
b1 = UpSampling2D((2, 2)) (a4)
# images 64 X 64
b1 = concatenate([b1, a3])

b1 = Conv2DTranspose(16, (5, 5), padding='same', strides=(1, 1)) (b1)
b1 = Activation('relu')(b1)
b1 = BatchNormalization()(b1)

b1 = Conv2DTranspose(16, (1, 1), padding='same', strides=(1, 1)) (b1)
b1 = Activation('relu')(b1)
b1 = BatchNormalization()(b1)

# #----B2 Layer-----------------------
b2 = UpSampling2D((2, 2)) (b1)
# images 128 X 128
b2 = concatenate([b2, a2])

b2 = Conv2DTranspose(16, (5, 5), padding='same', strides=(1, 1)) (b2)
b2 = Activation('relu')(b2)
b2 = BatchNormalization()(b2)

b2 = Conv2DTranspose(16, (1, 1), padding='same', strides=(1, 1)) (b2)
b2 = Activation('relu')(b2)
b2 = BatchNormalization()(b2)

#----B3 Layer------------------------
b3 = UpSampling2D((2, 2)) (b2)
# images 256 X 256
b3 = concatenate([b3, a1])

b3 = Conv2DTranspose(16, (5, 5), padding='same', strides=(1, 1)) (b3)
b3 = Activation('relu')(b3)
b3 = BatchNormalization()(b3)

b3 = Conv2DTranspose(16, (1, 1), padding='same', strides=(1, 1)) (b3)
b3 = Activation('relu')(b3)
b3 = BatchNormalization()(b3)

#----B4 Layer-------------------------
b4 = UpSampling2D((2, 2)) (b3)
# images 512 X 512
b4 = concatenate([b4, inputs])

b4 = Conv2DTranspose(16, (5, 5),padding='same', strides=(1, 1)) (b4)
b4 = Activation('relu')(b4)
output_img = Conv2DTranspose(1, (1, 1), strides=(1, 1)) (b4)
# output_img = Activation('relu')(b4)
# ------ end B4 layer

subtracted = Subtract()([inputs, output_img])

ddnet_model = Model(inputs=[inputs], outputs=[subtracted])
ddnet_model.compile(optimizer='adam', loss='mse', metrics=[tf_psnr])

```

DDNet model


```python
ddnet_model.summary()
```

    Model: "model_2"
    __________________________________________________________________________________________________
    Layer (type)                    Output Shape         Param #     Connected to                     
    ==================================================================================================
    input_2 (InputLayer)            (None, None, None, 1 0                                            
    __________________________________________________________________________________________________
    conv2d_26 (Conv2D)              (None, None, None, 1 784         input_2[0][0]                    
    __________________________________________________________________________________________________
    max_pooling2d_5 (MaxPooling2D)  (None, None, None, 1 0           conv2d_26[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_27 (BatchNo (None, None, None, 1 64          max_pooling2d_5[0][0]            
    __________________________________________________________________________________________________
    activation_28 (Activation)      (None, None, None, 1 0           batch_normalization_27[0][0]     
    __________________________________________________________________________________________________
    conv2d_27 (Conv2D)              (None, None, None, 1 6400        activation_28[0][0]              
    __________________________________________________________________________________________________
    concatenate_21 (Concatenate)    (None, None, None, 3 0           max_pooling2d_5[0][0]            
                                                                     conv2d_27[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_28 (BatchNo (None, None, None, 3 128         concatenate_21[0][0]             
    __________________________________________________________________________________________________
    activation_29 (Activation)      (None, None, None, 3 0           batch_normalization_28[0][0]     
    __________________________________________________________________________________________________
    conv2d_28 (Conv2D)              (None, None, None, 1 12800       activation_29[0][0]              
    __________________________________________________________________________________________________
    concatenate_22 (Concatenate)    (None, None, None, 6 0           max_pooling2d_5[0][0]            
                                                                     concatenate_21[0][0]             
                                                                     conv2d_28[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_29 (BatchNo (None, None, None, 6 256         concatenate_22[0][0]             
    __________________________________________________________________________________________________
    activation_30 (Activation)      (None, None, None, 6 0           batch_normalization_29[0][0]     
    __________________________________________________________________________________________________
    conv2d_29 (Conv2D)              (None, None, None, 1 25600       activation_30[0][0]              
    __________________________________________________________________________________________________
    concatenate_23 (Concatenate)    (None, None, None, 1 0           max_pooling2d_5[0][0]            
                                                                     concatenate_21[0][0]             
                                                                     concatenate_22[0][0]             
                                                                     conv2d_29[0][0]                  
    __________________________________________________________________________________________________
    conv2d_30 (Conv2D)              (None, None, None, 1 51200       concatenate_23[0][0]             
    __________________________________________________________________________________________________
    batch_normalization_30 (BatchNo (None, None, None, 1 64          conv2d_30[0][0]                  
    __________________________________________________________________________________________________
    activation_31 (Activation)      (None, None, None, 1 0           batch_normalization_30[0][0]     
    __________________________________________________________________________________________________
    conv2d_31 (Conv2D)              (None, None, None, 1 6400        activation_31[0][0]              
    __________________________________________________________________________________________________
    concatenate_24 (Concatenate)    (None, None, None, 2 0           max_pooling2d_5[0][0]            
                                                                     concatenate_21[0][0]             
                                                                     concatenate_22[0][0]             
                                                                     concatenate_23[0][0]             
                                                                     conv2d_31[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_31 (BatchNo (None, None, None, 2 1024        concatenate_24[0][0]             
    __________________________________________________________________________________________________
    activation_32 (Activation)      (None, None, None, 2 0           batch_normalization_31[0][0]     
    __________________________________________________________________________________________________
    conv2d_32 (Conv2D)              (None, None, None, 1 4112        activation_32[0][0]              
    __________________________________________________________________________________________________
    max_pooling2d_6 (MaxPooling2D)  (None, None, None, 1 0           conv2d_32[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_32 (BatchNo (None, None, None, 1 64          max_pooling2d_6[0][0]            
    __________________________________________________________________________________________________
    activation_33 (Activation)      (None, None, None, 1 0           batch_normalization_32[0][0]     
    __________________________________________________________________________________________________
    conv2d_33 (Conv2D)              (None, None, None, 1 6400        activation_33[0][0]              
    __________________________________________________________________________________________________
    concatenate_25 (Concatenate)    (None, None, None, 3 0           max_pooling2d_6[0][0]            
                                                                     conv2d_33[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_33 (BatchNo (None, None, None, 3 128         concatenate_25[0][0]             
    __________________________________________________________________________________________________
    activation_34 (Activation)      (None, None, None, 3 0           batch_normalization_33[0][0]     
    __________________________________________________________________________________________________
    conv2d_34 (Conv2D)              (None, None, None, 1 12800       activation_34[0][0]              
    __________________________________________________________________________________________________
    concatenate_26 (Concatenate)    (None, None, None, 6 0           max_pooling2d_6[0][0]            
                                                                     concatenate_25[0][0]             
                                                                     conv2d_34[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_34 (BatchNo (None, None, None, 6 256         concatenate_26[0][0]             
    __________________________________________________________________________________________________
    activation_35 (Activation)      (None, None, None, 6 0           batch_normalization_34[0][0]     
    __________________________________________________________________________________________________
    conv2d_35 (Conv2D)              (None, None, None, 1 25600       activation_35[0][0]              
    __________________________________________________________________________________________________
    concatenate_27 (Concatenate)    (None, None, None, 1 0           max_pooling2d_6[0][0]            
                                                                     concatenate_25[0][0]             
                                                                     concatenate_26[0][0]             
                                                                     conv2d_35[0][0]                  
    __________________________________________________________________________________________________
    conv2d_36 (Conv2D)              (None, None, None, 1 51200       concatenate_27[0][0]             
    __________________________________________________________________________________________________
    batch_normalization_35 (BatchNo (None, None, None, 1 64          conv2d_36[0][0]                  
    __________________________________________________________________________________________________
    activation_36 (Activation)      (None, None, None, 1 0           batch_normalization_35[0][0]     
    __________________________________________________________________________________________________
    conv2d_37 (Conv2D)              (None, None, None, 1 6400        activation_36[0][0]              
    __________________________________________________________________________________________________
    concatenate_28 (Concatenate)    (None, None, None, 2 0           max_pooling2d_6[0][0]            
                                                                     concatenate_25[0][0]             
                                                                     concatenate_26[0][0]             
                                                                     concatenate_27[0][0]             
                                                                     conv2d_37[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_36 (BatchNo (None, None, None, 2 1024        concatenate_28[0][0]             
    __________________________________________________________________________________________________
    activation_37 (Activation)      (None, None, None, 2 0           batch_normalization_36[0][0]     
    __________________________________________________________________________________________________
    conv2d_38 (Conv2D)              (None, None, None, 1 4112        activation_37[0][0]              
    __________________________________________________________________________________________________
    max_pooling2d_7 (MaxPooling2D)  (None, None, None, 1 0           conv2d_38[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_37 (BatchNo (None, None, None, 1 64          max_pooling2d_7[0][0]            
    __________________________________________________________________________________________________
    activation_38 (Activation)      (None, None, None, 1 0           batch_normalization_37[0][0]     
    __________________________________________________________________________________________________
    conv2d_39 (Conv2D)              (None, None, None, 1 6400        activation_38[0][0]              
    __________________________________________________________________________________________________
    concatenate_29 (Concatenate)    (None, None, None, 3 0           max_pooling2d_7[0][0]            
                                                                     conv2d_39[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_38 (BatchNo (None, None, None, 3 128         concatenate_29[0][0]             
    __________________________________________________________________________________________________
    activation_39 (Activation)      (None, None, None, 3 0           batch_normalization_38[0][0]     
    __________________________________________________________________________________________________
    conv2d_40 (Conv2D)              (None, None, None, 1 12800       activation_39[0][0]              
    __________________________________________________________________________________________________
    concatenate_30 (Concatenate)    (None, None, None, 6 0           max_pooling2d_7[0][0]            
                                                                     concatenate_29[0][0]             
                                                                     conv2d_40[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_39 (BatchNo (None, None, None, 6 256         concatenate_30[0][0]             
    __________________________________________________________________________________________________
    activation_40 (Activation)      (None, None, None, 6 0           batch_normalization_39[0][0]     
    __________________________________________________________________________________________________
    conv2d_41 (Conv2D)              (None, None, None, 1 25600       activation_40[0][0]              
    __________________________________________________________________________________________________
    concatenate_31 (Concatenate)    (None, None, None, 1 0           max_pooling2d_7[0][0]            
                                                                     concatenate_29[0][0]             
                                                                     concatenate_30[0][0]             
                                                                     conv2d_41[0][0]                  
    __________________________________________________________________________________________________
    conv2d_42 (Conv2D)              (None, None, None, 1 51200       concatenate_31[0][0]             
    __________________________________________________________________________________________________
    batch_normalization_40 (BatchNo (None, None, None, 1 64          conv2d_42[0][0]                  
    __________________________________________________________________________________________________
    activation_41 (Activation)      (None, None, None, 1 0           batch_normalization_40[0][0]     
    __________________________________________________________________________________________________
    conv2d_43 (Conv2D)              (None, None, None, 1 6400        activation_41[0][0]              
    __________________________________________________________________________________________________
    concatenate_32 (Concatenate)    (None, None, None, 2 0           max_pooling2d_7[0][0]            
                                                                     concatenate_29[0][0]             
                                                                     concatenate_30[0][0]             
                                                                     concatenate_31[0][0]             
                                                                     conv2d_43[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_41 (BatchNo (None, None, None, 2 1024        concatenate_32[0][0]             
    __________________________________________________________________________________________________
    activation_42 (Activation)      (None, None, None, 2 0           batch_normalization_41[0][0]     
    __________________________________________________________________________________________________
    conv2d_44 (Conv2D)              (None, None, None, 1 4112        activation_42[0][0]              
    __________________________________________________________________________________________________
    max_pooling2d_8 (MaxPooling2D)  (None, None, None, 1 0           conv2d_44[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_42 (BatchNo (None, None, None, 1 64          max_pooling2d_8[0][0]            
    __________________________________________________________________________________________________
    activation_43 (Activation)      (None, None, None, 1 0           batch_normalization_42[0][0]     
    __________________________________________________________________________________________________
    conv2d_45 (Conv2D)              (None, None, None, 1 6400        activation_43[0][0]              
    __________________________________________________________________________________________________
    concatenate_33 (Concatenate)    (None, None, None, 3 0           max_pooling2d_8[0][0]            
                                                                     conv2d_45[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_43 (BatchNo (None, None, None, 3 128         concatenate_33[0][0]             
    __________________________________________________________________________________________________
    activation_44 (Activation)      (None, None, None, 3 0           batch_normalization_43[0][0]     
    __________________________________________________________________________________________________
    conv2d_46 (Conv2D)              (None, None, None, 1 12800       activation_44[0][0]              
    __________________________________________________________________________________________________
    concatenate_34 (Concatenate)    (None, None, None, 6 0           max_pooling2d_8[0][0]            
                                                                     concatenate_33[0][0]             
                                                                     conv2d_46[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_44 (BatchNo (None, None, None, 6 256         concatenate_34[0][0]             
    __________________________________________________________________________________________________
    activation_45 (Activation)      (None, None, None, 6 0           batch_normalization_44[0][0]     
    __________________________________________________________________________________________________
    conv2d_47 (Conv2D)              (None, None, None, 1 25600       activation_45[0][0]              
    __________________________________________________________________________________________________
    concatenate_35 (Concatenate)    (None, None, None, 1 0           max_pooling2d_8[0][0]            
                                                                     concatenate_33[0][0]             
                                                                     concatenate_34[0][0]             
                                                                     conv2d_47[0][0]                  
    __________________________________________________________________________________________________
    conv2d_48 (Conv2D)              (None, None, None, 1 51200       concatenate_35[0][0]             
    __________________________________________________________________________________________________
    batch_normalization_45 (BatchNo (None, None, None, 1 64          conv2d_48[0][0]                  
    __________________________________________________________________________________________________
    activation_46 (Activation)      (None, None, None, 1 0           batch_normalization_45[0][0]     
    __________________________________________________________________________________________________
    conv2d_49 (Conv2D)              (None, None, None, 1 6400        activation_46[0][0]              
    __________________________________________________________________________________________________
    concatenate_36 (Concatenate)    (None, None, None, 2 0           max_pooling2d_8[0][0]            
                                                                     concatenate_33[0][0]             
                                                                     concatenate_34[0][0]             
                                                                     concatenate_35[0][0]             
                                                                     conv2d_49[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_46 (BatchNo (None, None, None, 2 1024        concatenate_36[0][0]             
    __________________________________________________________________________________________________
    activation_47 (Activation)      (None, None, None, 2 0           batch_normalization_46[0][0]     
    __________________________________________________________________________________________________
    conv2d_50 (Conv2D)              (None, None, None, 1 4112        activation_47[0][0]              
    __________________________________________________________________________________________________
    up_sampling2d_5 (UpSampling2D)  (None, None, None, 1 0           conv2d_50[0][0]                  
    __________________________________________________________________________________________________
    concatenate_37 (Concatenate)    (None, None, None, 3 0           up_sampling2d_5[0][0]            
                                                                     conv2d_44[0][0]                  
    __________________________________________________________________________________________________
    conv2d_transpose_9 (Conv2DTrans (None, None, None, 1 12816       concatenate_37[0][0]             
    __________________________________________________________________________________________________
    activation_48 (Activation)      (None, None, None, 1 0           conv2d_transpose_9[0][0]         
    __________________________________________________________________________________________________
    batch_normalization_47 (BatchNo (None, None, None, 1 64          activation_48[0][0]              
    __________________________________________________________________________________________________
    conv2d_transpose_10 (Conv2DTran (None, None, None, 1 272         batch_normalization_47[0][0]     
    __________________________________________________________________________________________________
    activation_49 (Activation)      (None, None, None, 1 0           conv2d_transpose_10[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_48 (BatchNo (None, None, None, 1 64          activation_49[0][0]              
    __________________________________________________________________________________________________
    up_sampling2d_6 (UpSampling2D)  (None, None, None, 1 0           batch_normalization_48[0][0]     
    __________________________________________________________________________________________________
    concatenate_38 (Concatenate)    (None, None, None, 3 0           up_sampling2d_6[0][0]            
                                                                     conv2d_38[0][0]                  
    __________________________________________________________________________________________________
    conv2d_transpose_11 (Conv2DTran (None, None, None, 1 12816       concatenate_38[0][0]             
    __________________________________________________________________________________________________
    activation_50 (Activation)      (None, None, None, 1 0           conv2d_transpose_11[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_49 (BatchNo (None, None, None, 1 64          activation_50[0][0]              
    __________________________________________________________________________________________________
    conv2d_transpose_12 (Conv2DTran (None, None, None, 1 272         batch_normalization_49[0][0]     
    __________________________________________________________________________________________________
    activation_51 (Activation)      (None, None, None, 1 0           conv2d_transpose_12[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_50 (BatchNo (None, None, None, 1 64          activation_51[0][0]              
    __________________________________________________________________________________________________
    up_sampling2d_7 (UpSampling2D)  (None, None, None, 1 0           batch_normalization_50[0][0]     
    __________________________________________________________________________________________________
    concatenate_39 (Concatenate)    (None, None, None, 3 0           up_sampling2d_7[0][0]            
                                                                     conv2d_32[0][0]                  
    __________________________________________________________________________________________________
    conv2d_transpose_13 (Conv2DTran (None, None, None, 1 12816       concatenate_39[0][0]             
    __________________________________________________________________________________________________
    activation_52 (Activation)      (None, None, None, 1 0           conv2d_transpose_13[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_51 (BatchNo (None, None, None, 1 64          activation_52[0][0]              
    __________________________________________________________________________________________________
    conv2d_transpose_14 (Conv2DTran (None, None, None, 1 272         batch_normalization_51[0][0]     
    __________________________________________________________________________________________________
    activation_53 (Activation)      (None, None, None, 1 0           conv2d_transpose_14[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_52 (BatchNo (None, None, None, 1 64          activation_53[0][0]              
    __________________________________________________________________________________________________
    up_sampling2d_8 (UpSampling2D)  (None, None, None, 1 0           batch_normalization_52[0][0]     
    __________________________________________________________________________________________________
    concatenate_40 (Concatenate)    (None, None, None, 1 0           up_sampling2d_8[0][0]            
                                                                     input_2[0][0]                    
    __________________________________________________________________________________________________
    conv2d_transpose_15 (Conv2DTran (None, None, None, 1 6816        concatenate_40[0][0]             
    __________________________________________________________________________________________________
    activation_54 (Activation)      (None, None, None, 1 0           conv2d_transpose_15[0][0]        
    __________________________________________________________________________________________________
    conv2d_transpose_16 (Conv2DTran (None, None, None, 1 17          activation_54[0][0]              
    __________________________________________________________________________________________________
    subtract_2 (Subtract)           (None, None, None, 1 0           input_2[0][0]                    
                                                                     conv2d_transpose_16[0][0]        
    ==================================================================================================
    Total params: 479,457
    Trainable params: 476,193
    Non-trainable params: 3,264
    __________________________________________________________________________________________________



```python
history=ddnet_model.fit(sparseview_60_train, ndct_train, validation_split=0.1, batch_size=10, epochs=40)

```

    Train on 540 samples, validate on 60 samples
    Epoch 1/40
    540/540 [==============================] - 19s 35ms/step - loss: 0.0011 - tf_psnr: 29.8691 - val_loss: 0.0019 - val_tf_psnr: 27.2116
    Epoch 2/40
    540/540 [==============================] - 19s 35ms/step - loss: 0.0011 - tf_psnr: 29.7986 - val_loss: 0.0017 - val_tf_psnr: 27.6557
    Epoch 3/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.9521e-04 - tf_psnr: 31.0269 - val_loss: 0.0010 - val_tf_psnr: 29.9420
    Epoch 4/40
    540/540 [==============================] - 19s 35ms/step - loss: 8.5768e-04 - tf_psnr: 30.7284 - val_loss: 7.3941e-04 - val_tf_psnr: 31.3242
    Epoch 5/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.7062e-04 - tf_psnr: 31.1725 - val_loss: 7.0704e-04 - val_tf_psnr: 31.5166
    Epoch 6/40
    540/540 [==============================] - 19s 35ms/step - loss: 8.8858e-04 - tf_psnr: 30.6390 - val_loss: 7.9605e-04 - val_tf_psnr: 30.9996
    Epoch 7/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.2811e-04 - tf_psnr: 31.4067 - val_loss: 6.8186e-04 - val_tf_psnr: 31.6725
    Epoch 8/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.9759e-04 - tf_psnr: 31.5844 - val_loss: 8.0168e-04 - val_tf_psnr: 30.9784
    Epoch 9/40
    540/540 [==============================] - 19s 35ms/step - loss: 8.2150e-04 - tf_psnr: 30.9244 - val_loss: 0.0016 - val_tf_psnr: 28.1129
    Epoch 10/40
    540/540 [==============================] - 19s 35ms/step - loss: 8.3867e-04 - tf_psnr: 30.8462 - val_loss: 7.0182e-04 - val_tf_psnr: 31.5452
    Epoch 11/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.6414e-04 - tf_psnr: 31.8029 - val_loss: 6.0881e-04 - val_tf_psnr: 32.1663
    Epoch 12/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.9797e-04 - tf_psnr: 31.5900 - val_loss: 8.1600e-04 - val_tf_psnr: 30.8983
    Epoch 13/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.3342e-04 - tf_psnr: 31.3820 - val_loss: 6.0540e-04 - val_tf_psnr: 32.1898
    Epoch 14/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.1870e-04 - tf_psnr: 31.5521 - val_loss: 8.0864e-04 - val_tf_psnr: 30.9299
    Epoch 15/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.8621e-04 - tf_psnr: 31.6766 - val_loss: 5.9929e-04 - val_tf_psnr: 32.2365
    Epoch 16/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.2689e-04 - tf_psnr: 31.5204 - val_loss: 8.9305e-04 - val_tf_psnr: 30.5157
    Epoch 17/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.5388e-04 - tf_psnr: 31.3420 - val_loss: 6.9770e-04 - val_tf_psnr: 31.5760
    Epoch 18/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.7351e-04 - tf_psnr: 31.8044 - val_loss: 6.8552e-04 - val_tf_psnr: 31.6556
    Epoch 19/40
    540/540 [==============================] - 19s 35ms/step - loss: 0.0038 - tf_psnr: 27.8579 - val_loss: 0.1067 - val_tf_psnr: 9.7576
    Epoch 20/40
    540/540 [==============================] - 19s 35ms/step - loss: 0.0100 - tf_psnr: 23.9637 - val_loss: 7.0196 - val_tf_psnr: -8.4257
    Epoch 21/40
    540/540 [==============================] - 19s 35ms/step - loss: 0.0011 - tf_psnr: 29.5019 - val_loss: 0.0052 - val_tf_psnr: 24.7202
    Epoch 22/40
    540/540 [==============================] - 19s 35ms/step - loss: 9.1100e-04 - tf_psnr: 30.4118 - val_loss: 0.0020 - val_tf_psnr: 27.4877
    Epoch 23/40
    540/540 [==============================] - 19s 35ms/step - loss: 8.4341e-04 - tf_psnr: 30.7515 - val_loss: 8.7340e-04 - val_tf_psnr: 30.6866
    Epoch 24/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.9827e-04 - tf_psnr: 30.9913 - val_loss: 8.9695e-04 - val_tf_psnr: 30.4866
    Epoch 25/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.7264e-04 - tf_psnr: 31.1276 - val_loss: 6.9286e-04 - val_tf_psnr: 31.6229
    Epoch 26/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.3970e-04 - tf_psnr: 31.3195 - val_loss: 6.8380e-04 - val_tf_psnr: 31.6801
    Epoch 27/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.3843e-04 - tf_psnr: 31.3281 - val_loss: 7.0879e-04 - val_tf_psnr: 31.5225
    Epoch 28/40
    540/540 [==============================] - 19s 35ms/step - loss: 7.0559e-04 - tf_psnr: 31.5243 - val_loss: 6.3572e-04 - val_tf_psnr: 32.0017
    Epoch 29/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.9338e-04 - tf_psnr: 31.6012 - val_loss: 6.1666e-04 - val_tf_psnr: 32.1169
    Epoch 30/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.7773e-04 - tf_psnr: 31.6996 - val_loss: 6.1554e-04 - val_tf_psnr: 32.1326
    Epoch 31/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.5468e-04 - tf_psnr: 31.8491 - val_loss: 6.1902e-04 - val_tf_psnr: 32.1035
    Epoch 32/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.4296e-04 - tf_psnr: 31.9295 - val_loss: 5.8644e-04 - val_tf_psnr: 32.3353
    Epoch 33/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.2382e-04 - tf_psnr: 32.0584 - val_loss: 5.7848e-04 - val_tf_psnr: 32.3948
    Epoch 34/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.3895e-04 - tf_psnr: 31.9555 - val_loss: 7.9203e-04 - val_tf_psnr: 31.0150
    Epoch 35/40
    540/540 [==============================] - 19s 35ms/step - loss: 6.1542e-04 - tf_psnr: 32.1189 - val_loss: 5.4819e-04 - val_tf_psnr: 32.6290
    Epoch 36/40
    540/540 [==============================] - 19s 35ms/step - loss: 5.9707e-04 - tf_psnr: 32.2535 - val_loss: 5.3602e-04 - val_tf_psnr: 32.7234
    Epoch 37/40
    540/540 [==============================] - 19s 35ms/step - loss: 5.8027e-04 - tf_psnr: 32.3750 - val_loss: 5.1697e-04 - val_tf_psnr: 32.8870
    Epoch 38/40
    540/540 [==============================] - 19s 35ms/step - loss: 5.6443e-04 - tf_psnr: 32.4924 - val_loss: 5.1814e-04 - val_tf_psnr: 32.8796
    Epoch 39/40
    540/540 [==============================] - 19s 35ms/step - loss: 5.7418e-04 - tf_psnr: 32.4189 - val_loss: 5.5014e-04 - val_tf_psnr: 32.6038
    Epoch 40/40
    540/540 [==============================] - 19s 35ms/step - loss: 5.4728e-04 - tf_psnr: 32.6273 - val_loss: 5.1227e-04 - val_tf_psnr: 32.9183


Save weights for the future reuse.


```python
ddnet_model.save_weights("/content/drive/My Drive/Colab Notebooks/dd_net/model.h5")
```

Plotting PSNR values to see the trend.


```python
import matplotlib.pyplot as plt
acc = history.history['tf_psnr']
val_acc = history.history['val_tf_psnr']
loss = history.history['loss']
val_loss = history.history['val_loss']

epochs = range(len(acc))

plt.plot(epochs, acc, 'r', label='Training accuracy')
plt.plot(epochs, val_acc, 'b', label='Validation accuracy')
plt.title('Training and validation accuracy')
plt.legend(loc=0)
plt.figure()
plt.show()
```


![png](output_31_0.png)



    <Figure size 432x288 with 0 Axes>



```python
reconstructed = ddnet_model.predict(sparseview_60_test)
psnr = cal_psnr(ndct_test, reconstructed)
print("psnr 40 epochs.....",psnr)
```

    psnr 40 epochs..... 29.894562583454164


psnr: 32.38




```python
from PIL import Image

a = reconstructed[0].reshape(512, 512)
scalef = np.amax(a)
a = np.clip(255 * a/scalef, 0, 255).astype('uint8')
#result = Image.fromarray((a * 255).astype(np.uint8))                                                                                                
result = Image.fromarray((a).astype(np.uint8))
# result.save('unet_15_600_0.png')
result.save('/content/drive/My Drive/Colab Notebooks/dd_net/reconstructed_ddnet_0.png')

a = reconstructed[99].reshape(512, 512)
scalef = np.amax(a)
a = np.clip(255 * a/scalef, 0, 255).astype('uint8')
#result = Image.fromarray((a * 255).astype(np.uint8))                                                                                                
result = Image.fromarray((a).astype(np.uint8))
# result.save('unet_15_600_99.png')
result.save('/content/drive/My Drive/Colab Notebooks/dd_net/reconstructed_ddnet_99.png')
```


```python
a = sparseview_60_test[0].reshape(512, 512)
scalef = np.amax(a)
a = np.clip(255 * a/scalef, 0, 255).astype('uint8')
#result = Image.fromarray((a * 255).astype(np.uint8))                                                                                                
result = Image.fromarray((a).astype(np.uint8))
# result.save('unet_15_600_0.png')
result.save('/content/drive/My Drive/Colab Notebooks/dd_net/sparseview_60_test_ddnet_0.png')

a = sparseview_60_test[99].reshape(512, 512)
scalef = np.amax(a)
a = np.clip(255 * a/scalef, 0, 255).astype('uint8')
#result = Image.fromarray((a * 255).astype(np.uint8))                                                                                                
result = Image.fromarray((a).astype(np.uint8))
# result.save('unet_15_600_99.png')
result.save('/content/drive/My Drive/Colab Notebooks/dd_net/sparseview_60_test_ddnet_99.png')
```


```python

```


```python
# Plot training & validation accuracy values
plt.plot(history.history['tf_psnr'])
plt.plot(history.history['val_tf_psnr'])
plt.title('Model accuracy')
plt.ylabel('Accuracy')
plt.xlabel('Epoch')
plt.legend(['Train', 'Test'], loc='upper left')
plt.show()

# Plot training & validation loss values
plt.plot(history.history['loss'])
plt.plot(history.history['val_loss'])
plt.title('Model loss')
plt.ylabel('Loss')
plt.xlabel('Epoch')
plt.legend(['Train', 'Test'], loc='upper left')
plt.show()
```


![png](/2020-02-21-ddnet/output_37_0.png)



![png](/2020-02-21-ddnet/output_37_1.png)



```python
img=mpimg.imread('/content/drive/My Drive/Colab Notebooks/dd_net/sparseview_60_test_ddnet_0.png')
imgplot = plt.imshow(img)
plt.show()
img=mpimg.imread('/content/drive/My Drive/Colab Notebooks/dd_net/reconstructed_ddnet_0.png')
imgplot = plt.imshow(img)
plt.show()
```


![png](/2020-02-21-ddnet/output_38_0.png)



![png](/2020-02-21-ddnet/output_38_1.png)



```python
img=mpimg.imread('/content/drive/My Drive/Colab Notebooks/dd_net/sparseview_60_test_ddnet_99.png')
imgplot = plt.imshow(img)
plt.show()
img=mpimg.imread('/content/drive/My Drive/Colab Notebooks/dd_net/reconstructed_ddnet_99.png')
imgplot = plt.imshow(img)
plt.show()
```


![png](/2020-02-21-ddnet/output_39_0.png)



![png](/2020-02-21-ddnet/output_39_1.png)
