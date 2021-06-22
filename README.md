# ___Keras insightface___
***

# License
  This is the keras implementation of [deepinsight/insightface](https://github.com/deepinsight/insightface), and is released under the MIT License. There is no limitation for both acadmic and commercial usage.

  The training data containing the annotation (and the models trained with these data) are available for non-commercial research purposes only.
# Table of Contents
  <!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

  - [___Keras insightface___](#keras-insightface)
  - [License](#license)
  - [Table of Contents](#table-of-contents)
  - [Current accuracy](#current-accuracy)
  - [Usage](#usage)
  	- [Environment](#environment)
  	- [Beforehand Data Prepare](#beforehand-data-prepare)
  	- [Training scripts](#training-scripts)
  	- [Mixed precision float16](#mixed-precision-float16)
  	- [Learning rate](#learning-rate)
  	- [Other backbones](#other-backbones)
  	- [Optimizers](#optimizers)
  	- [Multi GPU train using horovod or distribute strategy](#multi-gpu-train-using-horovod-or-distribute-strategy)
  - [TFLite model inference time test on ARM64](#tflite-model-inference-time-test-on-arm64)
  - [Sub Center ArcFace](#sub-center-arcface)
  - [Knowledge distillation](#knowledge-distillation)
  - [Evaluating on IJB datasets](#evaluating-on-ijb-datasets)
  - [Related Projects](#related-projects)

  <!-- /TOC -->
***

# Current accuracy
  - [Training results with different backbones / activation / data augmentation / loss function / others](https://www.xmind.net/m/rhrHGE/)
  - Model structures may change due to changing default behavior of building models.
  - `IJBB` and `IJBC` are scored at `TAR@FAR=1e-4`
  - Links in `Model backbone` are `h5` models in Google drive. Links in `Training` are training details.

  | Model backbone | Training | lfw      | cfp_fp   | agedb_30 | IJBB     | IJBC     |
  | -------------- | ----- | -------- | -------- | -------- | -------- | -------- |
  | [Resnet34](https://drive.google.com/file/d/1evH39rCBtdJ_wysv8LFHwT2GrgIYcCP0/view?usp=sharing) | [CASIA, E40](https://github.com/leondgarse/Keras_insightface/discussions/36)  | 0.99417 | 0.95086 | 0.94733 |          |          |
  | [Mobilenet emb256](https://drive.google.com/file/d/1i0B6Hy1clGgfeOYtUXVPNveDEe2DTIBa/view?usp=sharing) | [Emore, E110](https://github.com/leondgarse/Keras_insightface/discussions/15#discussioncomment-286398) | 0.996000 | 0.951714 | 0.959333 | 0.887147 | 0.911745 |
  | [Mobilenet_distill swish](https://drive.google.com/file/d/1yUjCG5rMeVCKTSPbST2F9BrRRlkDPzEA/view?usp=sharing) | [MS1MV3, E50](https://github.com/leondgarse/Keras_insightface/discussions/30) | 0.997333    | 0.969    | 0.975333 | 0.91889   | 0.940328 |
  | [Ghostnet strides 2](https://drive.google.com/file/d/1--ZRa8v3j5KKEcHzwTVMV6F8gsRr0mLB/view?usp=sharing) | [MS1MV3, E49](https://github.com/leondgarse/Keras_insightface/discussions/15#discussioncomment-322997) | 0.997167 | 0.959429 | 0.969333 | 0.912463 | 0.934499 |
  | [Ghostnet strides 1](https://drive.google.com/file/d/18RzKJDcrpdeYlJ70sMsYBOXwg4nh3EbA/view?usp=sharing) | [MS1MV3, E50](https://github.com/leondgarse/Keras_insightface/discussions/15#discussioncomment-583252) | 0.997833 | 0.978286 | 0.980000 | 0.931159 | 0.94943 |
  | [EfficientNetV2 B0](https://drive.google.com/file/d/1PgH87fznclAVI8Fsiu2x5BaY4UUiukNG/view?usp=sharing) | [MS1MV3, E50](https://github.com/leondgarse/Keras_insightface/discussions/15#discussioncomment-790627) | 0.997167 | 0.975286 | 0.975 | 0.937098 | 0.951935 |
  | [Botnet50 relu GDC](https://drive.google.com/file/d/12zD6Lba55WEHAcVAuCinJbaptrxH1pIW/view?usp=sharing) | [MS1MV3, E52](https://github.com/leondgarse/Keras_insightface/discussions/15#discussioncomment-583259) | 0.9985 | 0.980286 | 0.979667 | 0.940019 | 0.95577 |
  | [r50 swish](https://drive.google.com/file/d/1Mb2ZjBHFSQha8y2UOXO6-vxNWdzw815P/view?usp=sharing) | [MS1MV3, E50](https://github.com/leondgarse/Keras_insightface/discussions/15#discussioncomment-824090) | 0.998333 | 0.989571 | 0.984333 | 0.950828 | 0.964463 |
  | [se_r50 swish SD](https://drive.google.com/file/d/16R9weVSlXBgXPq7tduZGI6mDuB1nQ5MC/view?usp=sharing) | [MS1MV3, E50](https://github.com/leondgarse/Keras_insightface/discussions/15#discussioncomment-903227) | 0.9985 | 0.989429 | 0.9845 | 0.954333 | 0.966252 |
  | [Renet101V2 swish](https://drive.google.com/file/d/1joXsSpu22aa-kvnG1lQGNdVGfdPArUXM/view?usp=sharing) | [MS1MV3, E50](https://github.com/leondgarse/Keras_insightface/discussions/15#discussioncomment-790754) | 0.9985 | 0.989143 | 0.9845 | 0.952483 | 0.966406 |
***

# Usage
## Environment
  - **Nvidia CUDA and cudnn** `Tensorflow 2.5.0` now using `cuda==11.2` `cudnn==8.1`
    - [Install cuda-toolkit](https://developer.nvidia.com/cuda-toolkit)
    - [Install cudnn](https://developer.nvidia.com/rdp/cudnn-download)
  - **python and tensorflow version**
    ```py
    # $ ipython
    Python 3.8.5 (default, Sep  4 2020, 07:30:14)
    In [1]: tf.__version__
    Out[1]: '2.5.0'

    In [2]: import tensorflow_addons as tfa
    In [3]: tfa.__version__
    Out[3]: '0.13.0'
    ```
    Or `tf-nightly`
    ```sh
    # conda create -n tf-nightly python==3.7.6
    conda create -n tf-nightly python==3.8.5
    conda activate tf-nightly
    pip install tf-nightly tfa-nightly glob2 pandas tqdm scikit-image scikit-learn ipython
    # Not required
    pip install pip-search icecream opencv-python cupy-cuda112 tensorflow-datasets tabulate mxnet-cu112 torch
    ```
    ```py
    In [1]: tf.__version__
    Out[1]: '2.6.0-dev20210525'
    ```
  - **Default import for ipython**
    ```py
    import os
    import sys
    import pandas as pd
    import numpy as np
    import tensorflow as tf
    from tensorflow import keras

    gpus = tf.config.experimental.list_physical_devices("GPU")
    for gpu in gpus:
        tf.config.experimental.set_memory_growth(gpu, True)
    ```
## Beforehand Data Prepare
  - **Training Data** in this project is downloaded from [Insightface Dataset Zoo](https://github.com/deepinsight/insightface/wiki/Dataset-Zoo)
  - **Evaluating data** is `LFW` `CFP-FP` `AgeDB-30` bin files included in `MS1M-ArcFace` dataset
  - Any other data is also available just in the right format
  - **[prepare_data.py](prepare_data.py)** script, Extract data from mxnet record format to `folders`.
    ```sh
    # Convert `/datasets/faces_emore` to `/datasets/faces_emore_112x112_folders`
    CUDA_VISIBLE_DEVICES='-1' ./prepare_data.py -D /datasets/faces_emore
    # Convert evaluating bin files
    CUDA_VISIBLE_DEVICES='-1' ./prepare_data.py -D /datasets/faces_emore -T lfw.bin cfp_fp.bin agedb_30.bin
    ```
    Executing again will skip `dataset` conversion.
  - **Training dataset Required** is a `folder` including `person folders`, each `person folder` including multi `face images`. Format like
    ```sh
    .               # dataset folder
    ├── 0           # person folder
    │   ├── 100.jpg # face image
    │   ├── 101.jpg # face image
    │   └── 102.jpg # face image
    ├── 1           # person folder
    │   ├── 111.jpg
    │   ├── 112.jpg
    │   └── 113.jpg
    ├── 10
    │   ├── 707.jpg
    │   ├── 708.jpg
    │   └── 709.jpg
    ```
  - **Evaluating bin files** include jpeg image data pairs, and a label indicating if it's a same person, so there are double images than labels
    ```sh
    #    bins   | issame_list
    img_1 img_2 | True
    img_3 img_4 | True
    img_5 img_6 | False
    img_7 img_8 | False
    ```
    Image data in bin files like `CFP-FP` `AgeDB-30` is not compatible with `tf.image.decode_jpeg`, we need to reformat it, which is done by `-T` parameter.
    ```py
    ''' Throw error if not reformated yet '''
    ValueError: Can't convert non-rectangular Python sequence to Tensor.
    ```
## Training scripts
  - **Basic Scripts**
    - [backbones](backbones) basic model implementation of `mobilefacenet` / `mobilenetv3` / `resnest` / `efficientnet` / `botnet` / `ghostnet`. Most of them are copied from `keras.applications` source code and modified. Other backbones like `ResNet101V2` is loaded from `keras.applications` in `train.buildin_models`.
    - [data.py](data.py) loads image data as `tf.dataset` for training. `Triplet` dataset is different from others.
    - [evals.py](evals.py) contains evaluating callback using `bin` files.
    - [losses.py](losses.py) contains `softmax` / `arcface` / `centerloss` / `triplet` loss functions.
    - [myCallbacks.py](myCallbacks.py) contains my other callbacks, like saving model / learning rate adjusting / save history.
    - [models.py](models.py) contains model build related functions, like `buildin_models` / `add_l2_regularizer_2_model` / `replace_ReLU_with_PReLU`.
    - [train.py](train.py) contains a `Train` class. It uses a `scheduler` to connect different `loss` / `optimizer` / `epochs`. The basic function is simply `basic_model` --> `build dataset` --> `add output layer` --> `add callbacks` --> `compile` --> `fit`.
  - **Other Scripts**
    - [IJB_evals.py](IJB_evals.py) evaluates model accuracy using [insightface/evaluation/IJB/](https://github.com/deepinsight/insightface/tree/master/evaluation/IJB) datasets.
    - [data_drop_top_k.py](data_drop_top_k.py) create dataset after trained with [Sub Center ArcFace](#sub-center-arcface) method.
    - [data_distiller.py](data_distiller.py) create dataset for [Knowledge distillation](#knowledge-distillation).
    - [eval_folder.py](eval_folder.py) Run model evaluation on any custom dataset folder, which is in the same format with Training dataset.
    - [plot.py](plot.py) contains a history plot function.
    - [video_test.py](video_test.py) can be used to test model using video camera.
  - **Model** contains two part
    - **Basic model** is layers from `input` to `embedding`.
    - **Model** is `Basic model` + `bottleneck` layer, like `softmax` / `arcface` layer. For triplet training, `Model` == `Basic model`. For combined `loss` training, it may have multiple outputs.
  - **Training example** `train.Train` is mostly functioned as a scheduler.
    ```py
    from tensorflow import keras
    import losses, train, models
    import tensorflow_addons as tfa

    # basic_model = models.buildin_models("ResNet101V2", dropout=0.4, emb_shape=512, output_layer="E")
    basic_model = models.buildin_models("MobileNet", dropout=0, emb_shape=256, output_layer="GDC")
    data_path = '/datasets/faces_emore_112x112_folders'
    eval_paths = ['/datasets/faces_emore/lfw.bin', '/datasets/faces_emore/cfp_fp.bin', '/datasets/faces_emore/agedb_30.bin']

    tt = train.Train(data_path, save_path='keras_mobilenet_emore.h5', eval_paths=eval_paths,
                    basic_model=basic_model, batch_size=512, random_status=0,
                    lr_base=0.1, lr_decay=0.5, lr_decay_steps=16, lr_min=1e-5)
    optimizer = tfa.optimizers.SGDW(learning_rate=0.1, momentum=0.9, weight_decay=5e-5)
    sch = [
      {"loss": losses.ArcfaceLoss(scale=32), "epoch": 1, "optimizer": optimizer},
      {"loss": losses.ArcfaceLoss(scale=64), "epoch": 49},
      # {"loss": losses.ArcfaceLoss(), "epoch": 20, "triplet": 64, "alpha": 0.35},
    ]
    tt.train(sch, 0)
    ```
    Using `tt.train_single_scheduler` can control the behavior more detail.
  - **models.print_buildin_models** is used to print build-in model names in `models.py`, which can be used in `models.buildin_models`.
    ```py
    >>>> buildin_models
    MXNet version resnet: mobilenet_m1, r34, r50, r100, r101,
    Keras application: mobilenet, mobilenetv2, resnet50, resnet50v2, resnet101, resnet101v2, resnet152, resnet152v2
    EfficientNet: efficientnetb[0-7], efficientnetl2,
    Custom 1: ghostnet, mobilefacenet, mobilenetv3_small, mobilenetv3_large, se_mobilefacenet
    Custom 2: botnet50, botnet101, botnet152, resnest50, resnest101, se_resnext
    Or other names from keras.applications like DenseNet121 / InceptionV3 / NASNetMobile / VGG19.
    ```
  - **models.add_l2_regularizer_2_model** will add `l2_regularizer` to `dense` / `convolution` layers, or set `apply_to_batch_normal=True` also to `PReLU` / `BatchNormalization` layers. The actual added `l2` value is divided by `2`.
    ```py
    # Will add keras.regularizers.L2(5e-4) to `dense` / `convolution` layers.
    basic_model = models.add_l2_regularizer_2_model(basic_model, 1e-3, apply_to_batch_normal=False)
    ```
  - **train.Train model parameters** including `basic_model` / `model`. Combine them to initialize model from different sources. Sometimes may need `custom_objects` to load model.
    | basic_model                                                     | model           | Used for                                   |
    | --------------------------------------------------------------- | --------------- | ------------------------------------------ |
    | model structure                                                 | None            | Scratch train                              |
    | basic model .h5 file                                            | None            | Continue training from a saved basic model |
    | None for 'embedding' layer or layer index of basic model output | model .h5 file  | Continue training from last saved model    |
    | None for 'embedding' layer or layer index of basic model output | model structure | Continue training from a modified model    |
  - **train.Train output_weight_decay** controls `L2 regularizer` value added to `output_layer`.
    - `0` for None.
    - `(0, 1)` for specific value, actual added value will also divided by `2`.
    - `>= 1` will be value multiplied by `L2 regularizer` value in `basic_model` if added.
  - **Scheduler** is a list of dicts, each contains a training plan
    - **epoch** indicates how many epochs will be trained. **Required**.
    - **loss** indicates the loss function. If not provided, will try to use the previous one if `model.built` is `True`.
    - **optimizer** is the optimizer used in this plan, `None` indicates using the last one.
    - **bottleneckOnly** True / False, `True` will set `basic_model.trainable = False`, train the output layer only.
    - **centerloss** float value, if set a non zero value, attach a `CenterLoss` to `logits_loss`, and the value means `loss_weight`.
    - **triplet** float value, if set a non zero value, attach a `BatchHardTripletLoss` to `logits_loss`, and the value means `loss_weight`.
    - **alpha** float value, default to `0.35`. Alpha value for `BatchHardTripletLoss` if attached.
    - **lossTopK** indicates the `top K` value for [Sub Center ArcFace](#sub-center-arcface) method.
    - **distill** indicates the `loss_weight` for `distiller_loss` using [Knowledge distillation](#knowledge-distillation), default `7`.
    - **type** `softmax` / `arcface` / `triplet` / `center`, but mostly this could be guessed from `loss`.
    ```py
    # Scheduler examples
    sch = [
        {"loss": losses.scale_softmax, "optimizer": "adam", "epoch": 2},
        {"loss": keras.losses.CategoricalCrossentropy(label_smoothing=0.1), "centerloss": 0.01, "epoch": 2},
        {"loss": losses.ArcfaceLoss(scale=32.0, label_smoothing=0.1), "optimizer": keras.optimizers.SGD(0.1, momentum=0.9), "epoch": 2},
        {"loss": losses.BatchAllTripletLoss(0.3), "epoch": 2},
        {"loss": losses.BatchHardTripletLoss(0.25), "epoch": 2},
        {"loss": losses.CenterLoss(num_classes=85742, emb_shape=256), "epoch": 2},
        {"loss": losses.CurricularFaceLoss(), "epoch": 2},
    ]
    ```
    Some more complicated combinations are also supported.
    ```py
    # `softmax` + `centerloss`, `"centerloss": 0.1` means loss_weight
    sch = [{"loss": keras.losses.CategoricalCrossentropy(label_smoothing=0.1), "centerloss": 0.1, "epoch": 2}]
    # `softmax` / `arcface` + `triplet`, `"triplet": 64` means loss_weight
    sch = [{"loss": keras.losses.ArcfaceLoss(scale=64), "triplet": 64, "alpha": 0.3, "epoch": 2}]
    # `triplet` + `centerloss`
    sch = [{"loss": losses.BatchHardTripletLoss(0.25), "centerloss": 0.01, "epoch": 2}]
    sch = [{"loss": losses.CenterLoss(num_classes=85742, emb_shape=256), "triplet": 10, "alpha": 0.25, "epoch": 2}]
    # `softmax` / `arcface` + `triplet` + `centerloss`
    sch = [{"loss": losses.ArcfaceLoss(), "centerloss": 1, "triplet": 32, "alpha": 0.2, "epoch": 2}]
    ```
  - **Saving strategy**
    - **Model** will save the latest one on every epoch end to local path `./checkpoints`, name is specified by `train.Train` `save_path`.
    - **basic_model** will be saved monitoring on the last `eval_paths` evaluating `bin` item, and save the best only.
    ```py
    ''' Continue training from last saved file '''
    from tensorflow import keras
    import losses, train
    data_path = '/datasets/faces_emore_112x112_folders'
    eval_paths = ['/datasets/faces_emore/lfw.bin', '/datasets/faces_emore/cfp_fp.bin', '/datasets/faces_emore/agedb_30.bin']
    tt = train.Train(data_path, 'keras_mobilenet_emore_II.h5', eval_paths, model='./checkpoints/keras_mobilenet_emore.h5',
                    compile=True, lr_base=0.001, batch_size=480, random_status=3)

    sch = [
      # {"loss": losses.ArcfaceLoss(scale=32), "epoch": 1, "optimizer": optimizer},
      {"loss": losses.ArcfaceLoss(scale=64), "epoch": 35},
      # {"loss": losses.ArcfaceLoss(), "epoch": 20, "triplet": 64, "alpha": 0.35},
    ]
    tt.train(sch, 15) # 15 is the initial_epoch
    ```
    If reload a `centerloss` trained model, please keep `save_path` same as previous, as `centerloss` needs to reload saved `xxx_centers.npy` by `save_path` name.
  - **Gently stop** is a callback to stop training gently. Input an `n` and `<Enter>` anytime during training, will set training stop on that epoch ends.
  - **My history**
    - This is a callback collecting training `loss`, `accuracy` and `evaluating accuracy`.
    - On every epoch end, backup to the path `save_path` defined in `train.Train` with suffix `_hist.json`.
    - Reload when initializing, if the backup `<save_path>_hist.json` file exists.
  - **Evaluation**
    ```py
    import evals
    basic_model = keras.models.load_model('checkpoints/keras_mobilefacenet_256_basic_agedb_30_epoch_39_0.942500.h5', compile=False)
    ee = evals.eval_callback(basic_model, '/datasets/faces_emore/lfw.bin')
    ee.on_epoch_end(0)
    # >>>> lfw evaluation max accuracy: 0.993167, thresh: 0.316535, previous max accuracy: 0.000000, PCA accuray = 0.993167 ± 0.003905
    # >>>> Improved = 0.993167
    ```
    Default evaluating strategy is `on_epoch_end`. Setting an `eval_freq` greater than `1` in `train.Train` will also **add** an `on_batch_end` evaluation.
    ```py
    # Change evaluating strategy to `on_epoch_end`, as long as `on_batch_end` for every `1000` batch.
    tt = train.Train(data_path, 'keras_mobilefacenet_256.h5', eval_paths, basic_model=basic_model, eval_freq=1000)
    ```
## Mixed precision float16
  - [Tensorflow Guide - Mixed precision](https://www.tensorflow.org/guide/mixed_precision)
  - Enable `Mixed precision` at the beginning of all functional code by
    ```py
    keras.mixed_precision.set_global_policy("mixed_float16")
    ```
  - In most training case, it will have a `~2x` speedup and less GPU memory consumption.
## Learning rate
  - `train.Train` parameters `lr_base` / `lr_decay` / `lr_decay_steps` set different decay strategies and their parameters.
  - `tt.lr_scheduler` can also be used to set learning rate scheduler directly.
  - **lr_decay_steps** controls different decay types.
    - Default is `Exponential decay` with `lr_base=0.001, lr_decay=0.05`.
    - For `CosineLrScheduler`, `steps_per_epoch` is set after dataset been inited.
    - For `CosineLrScheduler`, default value of `keep_as_min=1`, means will train `1 epoch` using `lr_min` before each restart.

    | lr_decay_steps | decay type                                       | mean of lr_decay_steps    | mean of lr_decay |
    | -------------- | ------------------------------------------------ | ------------------------- | ---------------- |
    | <= 1           | Exponential decay                                |                           | decay_rate       |
    | > 1, < 500     | Cosine decay, will multiply with steps_per_epoch | first_restart_step, epoch | m_mul            |
    | >= 500         | Cosine decay, specific batch number              | first_restart_step, batch | m_mul            |
    | list           | Constant decay                                   | lr_decay_steps            | decay_rate       |

    ```py
    # lr_decay_steps == 0, Exponential
    tt = train.Train(..., lr_base=0.001, lr_decay=0.05, ...)
    # 1 < lr_decay_steps < 500, Cosine decay, first_restart_step = lr_decay_steps * steps_per_epoch
    # restart on epoch [16 * 1 + 1, 16 * 3 + 2, 16 * 7 + 3] == [17, 50, 115]
    tt = train.Train(..., lr_base=0.001, lr_decay=0.5, lr_decay_steps=16, lr_min=1e-7, ...)
    # 1 < lr_decay_steps < 500, lr_min == lr_base * lr_decay, Cosine decay, no restart
    tt = train.Train(..., lr_base=0.001, lr_decay=1e-4, lr_decay_steps=24, lr_min=1e-7, ...)
    # 500 <= lr_decay_steps, Cosine decay, first_restart_step = lr_decay_steps
    # restart on batch [16000 + steps_per_epoch, 48000 + 2 * steps_per_epoch, 112000 + 3 * steps_per_epoch]
    tt = train.Train(..., lr_base=0.001, lr_decay=0.5, lr_decay_steps=16 * 1000, lr_min=1e-7, ...)
    # lr_decay_steps is a list, Constant
    tt = train.Train(..., lr_base=0.1, lr_decay=0.1, lr_decay_steps=[3, 5, 7, 16, 20, 24], ...)
    ```
  - **Example learning rates**
    ```py
    from myCallbacks import exp_scheduler, CosineLrScheduler, constant_scheduler
    epochs = np.arange(60)
    plt.figure(figsize=(14, 6))
    plt.plot(epochs, [exp_scheduler(ii, 0.001, 0.1) for ii in epochs], label="lr=0.001, decay=0.1")
    plt.plot(epochs, [exp_scheduler(ii, 0.001, 0.05) for ii in epochs], label="lr=0.001, decay=0.05")
    plt.plot(epochs, [constant_scheduler(ii, 0.001, [10, 20, 30, 40], 0.1) for ii in epochs], label="Constant, lr=0.001, decay_steps=[10, 20, 30, 40], decay_rate=0.1")

    steps_per_epoch = 100
    batchs = np.arange(60 * steps_per_epoch)
    aa = CosineLrScheduler(0.001, first_restart_step=50, lr_min=1e-6, warmup=0, m_mul=1e-3, steps_per_epoch=steps_per_epoch)
    aa.build()
    plt.plot(batchs / steps_per_epoch, [aa.on_train_batch_begin(ii) for ii in batchs], label="Cosine, first_restart_step=50, min=1e-6, m_mul=1e-3")
    aa_60 = np.float(aa.on_train_batch_begin(60 * steps_per_epoch))
    plt.text(60, aa_60, "[Cosine]\n({}, {:.2e})".format(60, aa_60), va="bottom", ha="right")

    bb = CosineLrScheduler(0.001, first_restart_step=16, lr_min=1e-7, warmup=1, m_mul=0.4, steps_per_epoch=steps_per_epoch)
    bb.build()
    plt.plot(batchs / steps_per_epoch, [bb.on_train_batch_begin(ii) for ii in batchs], label="Cosine restart, first_restart_step=16, min=1e-7, warmup=1, m_mul=0.4")
    plt.scatter(18, 0.0004, c='r')
    plt.text(18, 0.0004, (18, 0.0004))

    cc = CosineLrScheduler(0.001, first_restart_step=13 * 100, lr_min=1e-7, warmup=4, m_mul=0.5, steps_per_epoch=steps_per_epoch)
    cc.build()
    plt.plot(batchs / steps_per_epoch, [cc.on_train_batch_begin(ii) for ii in batchs], label="Cosine restart, on batch, first_restart_step=1300, min=1e-7, warmup=4, m_mul=0.5")
    plt.scatter(18, 0.0005, c='r')
    plt.text(18, 0.0005, (18, 0.0005))

    plt.xlim(0, 60)
    plt.legend()
    # plt.grid()
    plt.tight_layout()
    ```
    ![](checkpoints/learning_rate_decay.svg)
## Other backbones
  - **EfficientNet** `tf-nightly` / `tf 2.3.0` now includes all `EfficientNet` backbone in `tensorflow.keras.applications`, but it has a `Rescaling` and `Normalization` layer on the head.
    ```py
    tf.__version__
    # '2.3.0-dev20200523'
    mm = tf.keras.applications.efficientnet.EfficientNetB4(include_top=False, weights='imagenet', input_shape=(112, 112, 3))
    [ii.name for ii in mm.layers[:6]]
    # ['input_17', 'rescaling_2', 'normalization_2', 'stem_conv_pad', 'stem_conv', 'stem_bn']
    ```
    So I'm using a modified version in `backbones/efficientnet.py`
    ```py
    from backbones import efficientnet
    mm = efficientnet.EfficientNetB0(weights='imagenet', include_top=False, input_shape=(112, 112, 3))
    [ii.name for ii in mm.layers[:3]]
    # ['input_1', 'stem_conv_pad', 'stem_conv']
    ```
    Other's implementation can be found here [Github qubvel/EfficientNet](https://github.com/qubvel/efficientnet).
  - **ResNeSt / RegNet** [Github QiaoranC/tf_ResNeSt_RegNet_model](https://github.com/QiaoranC/tf_ResNeSt_RegNet_model)
    ```py
    from models.model_factory import get_model
    input_shape = [112, 112, 3]
    n_classes = 100
    fc_activation = 'softmax'
    mm = get_model(model_name="ResNest101",input_shape=input_shape,n_classes=n_classes, verbose=False,fc_activation=fc_activation)
    ```
  - [SE nets](https://github.com/titu1994/keras-squeeze-excite-network)
    ```py
    # This should be under tf 2.3, NOT tf nightly
    tf.__version__
    # '2.3.0'

    !pip install -U git+https://github.com/titu1994/keras-squeeze-excite-network

    from keras_squeeze_excite_network import se_resnext
    mm = se_resnext.SEResNextImageNet(weights='imagenet', input_shape=(112, 112, 3), include_top=False)
    ```
    It's TOO slow training a `se_resnext 101`，takes almost 4 times longer than `ResNet101V2`.
## Optimizers
  - **SGDW / AdamW** [tensorflow_addons AdamW](https://www.tensorflow.org/addons/api_docs/python/tfa/optimizers/AdamW).
    ```py
    # !pip install tensorflow-addons
    !pip install tfa-nightly

    import tensorflow_addons as tfa
    optimizer = tfa.optimizers.SGDW(learning_rate=0.1, weight_decay=5e-4, momentum=0.9)
    optimizer = tfa.optimizers.AdamW(learning_rate=0.001, weight_decay=5e-5)
    ```
    `weight_decay` and `learning_rate` should share the same decay strategy. A callback `OptimizerWeightDecay` will set `weight_decay` according to `learning_rate`.
    ```py
    opt = tfa.optimizers.AdamW(weight_decay=5e-5)
    sch = [{"loss": keras.losses.CategoricalCrossentropy(label_smoothing=0.1), "centerloss": True, "epoch": 60, "optimizer": opt}]
    ```
    - The different behavior of `mx.optimizer.SGD weight_decay` / `tfa.optimizers.SGDW weight_decay` / `L2_regulalizer` is explained [here the discussion](https://github.com/leondgarse/Keras_insightface/discussions/19).
    - [PDF DECOUPLED WEIGHT DECAY REGULARIZATION](https://arxiv.org/pdf/1711.05101.pdf)
    - [Train test of SGDW on cifar10](https://colab.research.google.com/drive/1tD2OrnrYtFPC7q_i62b8al1o3qelU-Vi?usp=sharing)
  - **RAdam / Lookahead / Ranger optimizer** [tensorflow_addons RectifiedAdam](https://www.tensorflow.org/addons/api_docs/python/tfa/optimizers/RectifiedAdam).
    ```py
    # Rectified Adam,a.k.a. RAdam, [ON THE VARIANCE OF THE ADAPTIVE LEARNING RATE AND BEYOND](https://arxiv.org/pdf/1908.03265.pdf)
    optimizer = tfa.optimizers.RectifiedAdam()
    # SGD with Lookahead [Lookahead Optimizer: k steps forward, 1 step back](https://arxiv.org/pdf/1907.08610.pdf)
    optmizer = tfa.optimizers.Lookahead(keras.optimizers.SGD(0.1))
    # Ranger [Gradient Centralization: A New Optimization Technique for Deep Neural Networks](https://arxiv.org/pdf/2004.01461.pdf)
    optmizer = tfa.optimizers.Lookahead(tfa.optimizers.RectifiedAdam())
    ```
## Multi GPU train using horovod or distribute strategy
  - **Horovod** usage is still under test. [Tensorflow multi GPU training using distribute strategies vs Horovod](https://github.com/leondgarse/Keras_insightface/discussions/17)
  - Add an overall `tf.distribute.MirroredStrategy().scope()` `with` block. This is just working in my case... The `batch_size` will be multiplied by `count of GPUs`.
    ```py
    with tf.distribute.MirroredStrategy().scope():
        basic_model = ...
        tt = train.Train(..., batch_size=1024, ...) # With 2 GPUs, `batch_size` will be 2048
        sch = [...]
        tt.train(sch, 0)
    ```
  - Using build-in loss functions like `keras.losses.CategoricalCrossentropy` should specify the `reduction` parameter.
    ```py
    sch = [{"loss": keras.losses.CategoricalCrossentropy(label_smoothing=0.1, reduction=tf.keras.losses.Reduction.NONE), "epoch": 25}]
    ```
***

# TFLite model inference time test on ARM64
  - Test using [TFLite Model Benchmark Tool](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/tools/benchmark)
  - **Platform**
    - CPU: `Qualcomm Technologies, Inc SDM630`
    - System: `Android`
    - Inference: `TFLite`
  - **mobilenet_v2** comparing `orignal` / `dynamic` / `float16` / `uint8` conversion of `TFLite` model. Using header `GDC + emb_shape=512 + pointwise_conv=False`.
    | mobilenet_v2 | Size (MB) | threads=1 (ms) | threads=4 (ms) |
    | ------------ | --------- | -------------- | -------------- |
    | orignal      | 11.576    | 52.224         | 18.102         |
    | orignal xnn  | 11.576    | 29.116         | 8.744          |
    | dynamic      | 3.36376   | 38.497         | 20.008         |
    | dynamic xnn  | 3.36376   | 37.433         | 19.234         |
    | float16      | 5.8267    | 53.986         | 19.191         |
    | float16 xnn  | 5.8267    | 29.862         | 8.661          |
    | uint8        | 3.59032   | 27.247         | 10.783         |

  - **mobilenet_v2** comparing different headers using `float16 conversion + xnn + threads=4`
    | emb_shape | output_layer | pointwise_conv | PReLU | Size (MB) | Time (ms) |
    | ---------:|:------------ |:-------------- | ----- | ---------:| ---------:|
    |       256 | GDC          | False          | False |   5.17011 |     8.214 |
    |       512 | GDC          | False          | False |   5.82598 |     8.436 |
    |       256 | GDC          | True           | False |   6.06384 |     9.129 |
    |       512 | GDC          | True           | False |   6.32542 |     9.357 |
    |       256 | E            | True           | False |   9.98053 |    10.669 |
    |       256 | E            | False          | False |   14.9618 |    11.502 |
    |       512 | E            | True           | False |    14.174 |    11.958 |
    |       512 | E            | False          | False |   25.4481 |    15.063 |
    |       512 | GDC          | False          | True  |   5.85275 |    10.481 |

  - **Backbones comparing** using `float16 conversion + xnn + threads=4`, header `GDC + emb_shape=512 + pointwise_conv=False`
    | Model              | Size (MB) | Time (ms) |
    | ------------------ | --------- | --------- |
    | mobilenet_v3_small | 2.80058   | 4.211     |
    | mobilenet_v3_large | 6.95015   | 10.025    |
    | ghostnet strides=2 | 8.06546   | 11.125    |
    | mobilenet          | 7.4905    | 11.836    |
    | se_mobilefacenet   | 1.88518   | 18.713    |
    | mobilefacenet      | 1.84267   | 20.443    |
    | EB0                | 9.40449   | 22.054    |
    | EB1                | 14.4268   | 31.881    |
    | ghostnet strides=1 | 8.16576   | 46.142    |
    | mobilenet_m1       | 7.02651   | 52.648    |
***

# Sub Center ArcFace
  - [Original MXNet Subcenter ArcFace](https://github.com/deepinsight/insightface/tree/master/recognition/SubCenter-ArcFace)
  - [PDF Sub-center ArcFace: Boosting Face Recognition by Large-scale Noisy Web Faces](https://ibug.doc.ic.ac.uk/media/uploads/documents/eccv_1445.pdf)
  - **This is still under test, Multi GPU is NOT tested**
  - As far as I can see
    - `Sub Center ArcFace` works like cleaning the dataset.
    - In `lossTopK=3` case, it will train `3 sub classes` in each label, and each `sub class` is a `center`.
    - Then choose a `domain center`, and remove those are too far away from this `center`.
    - So it's better train a `large model` to clean the `dataset`, and then train other models on the `cleaned dataset`.
  - **Train Original MXNet version**
    ```sh
    cd ~/workspace/insightface/recognition/SubCenter-ArcFace
    cp sample_config.py config.py
    sed -i 's/config.ckpt_embedding = True/config.ckpt_embedding = False/' config.py
    CUDA_VISIBLE_DEVICES='1' python train_parall.py --network r50 --per-batch-size 512
    # Iter[20] Batch [8540], accuracy 0.80078125, loss 1.311261, lfw 0.99817, cfp_fp 0.97557, agedb_30 0.98167

    CUDA_VISIBLE_DEVICES='1' python drop.py --data /datasets/faces_emore --model models/r50-arcface-emore/model,1 --threshold 75 --k 3 --output /datasets/faces_emore_topk3_1
    # header0 label [5822654. 5908396.] (5822653, 4)
    # total: 5800493

    sed -i 's/config.ckpt_embedding = False/config.ckpt_embedding = True/' config.py
    sed -i 's/config.loss_K = 3/config.loss_K = 1/' config.py
    sed -i 's#/datasets/faces_emore#/datasets/faces_emore_topk3_1#' config.py
    ls -1 /datasets/faces_emore/*.bin | xargs -I '{}' ln -s {} /datasets/faces_emore_topk3_1/
    CUDA_VISIBLE_DEVICES='1' python train_parall.py --network r50 --per-batch-size 512
    # 5800493
    # Iter[20] Batch [5400], accuracy 0.8222656, loss 1.469272, lfw 0.99833, cfp_fp 0.97986, agedb_30 0.98050
    ```
  - **Keras version train mobilenet on CASIA test**
    ```py
    import tensorflow_addons as tfa
    import train, losses, models

    data_basic_path = '/datasets/faces_casia'
    data_path = data_basic_path + '_112x112_folders'
    eval_paths = [os.path.join(data_basic_path, ii) for ii in ['lfw.bin', 'cfp_fp.bin', 'agedb_30.bin']]

    """ First, Train with `lossTopK = 3` """
    basic_model = models.buildin_models("mobilenet", dropout=0, emb_shape=256, output_layer='E')
    tt = train.Train(data_path, save_path='TT_mobilenet_topk_bs256.h5', eval_paths=eval_paths,
        basic_model=basic_model, model=None, lr_base=0.1, lr_decay=0.1, lr_decay_steps=[20, 30],
        batch_size=256, random_status=0, output_wd_multiply=1)

    optimizer = tfa.optimizers.SGDW(learning_rate=0.1, weight_decay=5e-4, momentum=0.9)
    sch = [
        {"loss": losses.ArcfaceLoss(scale=16), "epoch": 5, "optimizer": optimizer, "lossTopK": 3},
        {"loss": losses.ArcfaceLoss(scale=32), "epoch": 5, "lossTopK": 3},
        {"loss": losses.ArcfaceLoss(scale=64), "epoch": 40, "lossTopK": 3},
    ]
    tt.train(sch, 0)

    """ Then drop non-dominant subcenters and high-confident noisy data, which is `>75 degrees` """
    import data_drop_top_k
    # data_drop_top_k.data_drop_top_k('./checkpoints/TT_mobilenet_topk_bs256.h5', '/datasets/faces_casia_112x112_folders/', limit=20)
    new_data_path = data_drop_top_k.data_drop_top_k(tt.model, tt.data_path)

    """ Train with the new dataset again, this time `lossTopK = 1` """
    tt.reset_dataset(new_data_path)
    optimizer = tfa.optimizers.SGDW(learning_rate=0.1, weight_decay=5e-4, momentum=0.9)
    sch = [
        {"loss": losses.ArcfaceLoss(scale=16), "epoch": 5, "optimizer": optimizer},
        {"loss": losses.ArcfaceLoss(scale=32), "epoch": 5},
        {"loss": losses.ArcfaceLoss(scale=64), "epoch": 40},
    ]
    tt.train(sch, 0)
    ```
  - `data_drop_top_k.py` can also be used as a script. `-M` and `-D` are required.
    ```sh
    $ CUDA_VISIBLE_DEVICES='-1' ./data_drop_top_k.py -h
    # usage: data_drop_top_k.py [-h] -M MODEL_FILE -D DATA_PATH [-d DEST_FILE]
    #                           [-t DEG_THRESH] [-L LIMIT]
    #
    # optional arguments:
    #   -h, --help            show this help message and exit
    #   -M MODEL_FILE, --model_file MODEL_FILE
    #                         Saved model file path, NOT basic_model (default: None)
    #   -D DATA_PATH, --data_path DATA_PATH
    #                         Original dataset path (default: None)
    #   -d DEST_FILE, --dest_file DEST_FILE
    #                         Dest file path to save the processed dataset npz
    #                         (default: None)
    #   -t DEG_THRESH, --deg_thresh DEG_THRESH
    #                         Thresh value in degree, [0, 180] (default: 75)
    #   -L LIMIT, --limit LIMIT
    #                         Test parameter, limit converting only the first [NUM]
    #                         ones (default: 0)
    ```
    ```sh
    $ CUDA_VISIBLE_DEVICES='-1' ./data_drop_top_k.py -M checkpoints/TT_mobilenet_topk_bs256.h5 -D /datasets/faces_casia_112x112_folders/ -L 20
    ```
  - **[[Discussions] SubCenter_training_Mobilenet_on_CASIA](https://github.com/leondgarse/Keras_insightface/discussions/20)**

    | Scenario                                    | Max lfw    | Max cfp_fp | Max agedb_30 |
    | ------------------------------------------- | ---------- | ---------- | ------------ |
    | Baseline, topk 1                            | 0.9822     | 0.8694     | 0.8695       |
    | TopK 3                                      | 0.9838     | **0.9044** | 0.8743       |
    | TopK 3->1                                   | 0.9838     | 0.8960     | 0.8768       |
    | TopK 3->1, bottleneckOnly, initial_epoch=0  | **0.9878** | 0.8920     | **0.8857**   |
    | TopK 3->1, bottleneckOnly, initial_epoch=40 | 0.9835     | **0.9030** | 0.8763       |
***

# Knowledge distillation
  - [PDF Improving Face Recognition from Hard Samples via Distribution Distillation Loss](https://arxiv.org/pdf/2002.03662.pdf)
  - [PDF VarGFaceNet: An Efficient Variable Group Convolutional Neural Network for Lightweight Face Recognition](https://arxiv.org/pdf/1910.04985.pdf)
  - `data_distiller.py` works to extract `embedding` data from images and save locally. `MODEL_FILE` can be `Keras h5` / `pytorch jit pth` / `MXNet model`.
    - **--save_npz** Default saving format is `.tfrecord`, which needs less memory while training.
    - **-D xxx.npz** Convert `xxx.npz` to `xxx.tfrecord`.
    - **--use_fp16** Save embedding data in `float16` format, which needs half less disk space than default `float32`.
    ```sh
    $ CUDA_VISIBLE_DEVICES='-1' ./data_distiller.py -h
    # usage: data_distiller.py [-h] -D DATA_PATH [-M MODEL_FILE] [-d DEST_FILE]
    #                          [-b BATCH_SIZE] [-L LIMIT] [--use_fp16] [--save_npz]
    #
    # optional arguments:
    #   -h, --help            show this help message and exit
    #   -D DATA_PATH, --data_path DATA_PATH
    #                         Data path, or npz file converting to tfrecord
    #                         (default: None)
    #   -M MODEL_FILE, --model_file MODEL_FILE
    #                         Model file, keras h5 / pytorch pth / mxnet (default:
    #                         None)
    #   -d DEST_FILE, --dest_file DEST_FILE
    #                         Dest file path to save the processed dataset (default:
    #                         None)
    #   -b BATCH_SIZE, --batch_size BATCH_SIZE
    #                         Batch size (default: 256)
    #   -L LIMIT, --limit LIMIT
    #                         Test parameter, limit converting only the first [NUM]
    #                         (default: -1)
    #   --use_fp16            Save using float16 (default: False)
    #   --save_npz            Save as npz file, default is tfrecord (default: False)
    ```
    ```sh
    $ CUDA_VISIBLE_DEVICES='0' ./data_distiller.py -M subcenter-arcface-logs/r100-arcface-msfdrop75/model,0 -D /datasets/faces_casia_112x112_folders/ -b 32 --use_fp16
    # >>>> Output: faces_casia_112x112_folders_shuffle_label_embs_normed_512.npz
    ```
  - Then this dataset can be used to train a new model.
    - Just specify `data_path` as the new dataset path. If key `embeddings` is in, then it will be a `distiller train`.
    - A new loss `distiller_loss_cosine` will be added to match this `embeddings` data, default `loss_weights = [1, 7]`. Parameter `distill` in `scheduler` set this loss weight.
    - Distill loss can be used along or combined with `softmax` / `arcface` / `centerloss` / `triplet`.
    - The `emb_shape` can be differ from `teacher`, in this case, a dense layer `distill_emb_map_layer` will be added between `basic_model` embedding layer output and `teacher` embedding data.
    ```py
    import train, losses, models
    import tensorflow_addons as tfa

    data_basic_path = '/datasets/faces_casia'
    data_path = 'faces_casia_112x112_folders_shuffle_label_embs_512_fp16.tfrecord'
    eval_paths = [os.parh.join(data_basic_path, ii) for ii in ['lfw.bin', 'cfp_fp.bin', 'agedb_30.bin']]

    basic_model = models.buildin_models("mobilenet", dropout=0.4, emb_shape=512, output_layer='E')
    tt = train.Train(data_path, save_path='TT_mobilenet_distill_bs400.h5', eval_paths=eval_paths,
        basic_model=basic_model, model=None, lr_base=0.1, lr_decay=0.1, lr_decay_steps=[20, 30],
        batch_size=400, random_status=0)

    optimizer = tfa.optimizers.SGDW(learning_rate=0.1, weight_decay=5e-4, momentum=0.9)
    sch = [
        {"loss": losses.ArcfaceLoss(scale=16), "epoch": 5, "optimizer": optimizer, "distill": 128},
        {"loss": losses.ArcfaceLoss(scale=32), "epoch": 5, "distill": 128},
        {"loss": losses.ArcfaceLoss(scale=64), "epoch": 40, "distill": 128},
    ]
    tt.train(sch, 0)
    ```
  - **Knowledge distillation result of training Mobilenet on CASIA**

    | Teacher | emb_shape | Dropout | Optimizer | Distill | Max lfw    | Max cfp_fp | Max agedb_30 |
    | ------- | --------- | ------- | --------- | ------- | ---------- | ---------- | ------------ |
    | None    | 512       | 0       | SGDW      | 0       | 0.9838     | 0.8730     | 0.8697       |
    | None    | 512       | 0.4     | SGDW      | 0       | 0.9837     | 0.8491     | 0.8745       |
    | r100    | 512       | 0       | SGDW      | 7       | 0.9900     | 0.9111     | 0.9068       |
    | r100    | 512       | 0.4     | SGDW      | 7       | 0.9905     | 0.9170     | 0.9112       |
    | r100    | 512       | 0.4     | SGDW      | 128     | **0.9955** | **0.9376** | **0.9465**   |
    | r100    | 512       | 0.4     | AdamW     | 128     | 0.9920     | 0.9346     | 0.9387       |
    | r100    | 512       | 0.4     | AdamW     | 128     | 0.9920     | 0.9346     | 0.9387       |
    | r100    | 256       | 0       | SGDW      | 128     | 0.9937     | 0.9337     | 0.9427       |
    | r100    | 256       | 0.4     | SGDW      | 128     | 0.9942     | 0.9369     | 0.9448       |

  - [Knowledge distillation using Mobilenet on MS1M dataset](https://github.com/leondgarse/Keras_insightface/discussions/30)

    | Teacher | emb_shape | Dropout | Optimizer | Distill | Max lfw | Max cfp_fp | Max agedb_30 |
    | ------- | --------- | ------- | --------- | ------- | ------- | ---------- | ------------ |
    | r100    | 512       | 0.4     | SGDW      | 128     | 0.997   | 0.964      | 0.972833     |
***

# Evaluating on IJB datasets
  - [IJB_evals.py](IJB_evals.py) evaluates model accuracy using [insightface/evaluation/IJB/](https://github.com/deepinsight/insightface/tree/master/evaluation/IJB) datasets.
  - In case placing `IJB` dataset `/media/SD/IJB_release`, basic usage will be:
    ```sh
    # Test mxnet model, default scenario N0D1F1
    CUDA_VISIBLE_DEVICES='1' python IJB_evals.py -m '/media/SD/IJB_release/pretrained_models/MS1MV2-ResNet100-Arcface/model,0' -d /media/SD/IJB_release -L

    # Test keras h5 model, default scenario N0D1F1
    CUDA_VISIBLE_DEVICES='1' python IJB_evals.py -m 'checkpoints/basic_model.h5' -d /media/SD/IJB_release -L

    # `-B` to run all 8 tests N{0,1}D{0,1}F{0,1}
    CUDA_VISIBLE_DEVICES='1' python IJB_evals.py -m 'checkpoints/basic_model.h5' -d /media/SD/IJB_release -B -L

    # `-N` to run 1N test
    CUDA_VISIBLE_DEVICES='1' python IJB_evals.py -m 'checkpoints/basic_model.h5' -d /media/SD/IJB_release -N -L

    # `-E` to save embeddings data
    CUDA_VISIBLE_DEVICES='1' python IJB_evals.py -m 'checkpoints/basic_model.h5' -d /media/SD/IJB_release -E
    # Then can be restored for other tests, add `-E` to save again
    python IJB_evals.py -R IJB_result/MS1MV2-ResNet100-Arcface_IJBB.npz -d /media/SD/IJB_release -B

    # Plot result only, this needs the `label` data, which can be saved using `-L` parameter.
    # Or should provide the label txt file.
    python IJB_evals.py --plot_only /media/SD/IJB_release/IJBB/result/*100*.npy /media/SD/IJB_release/IJBB/meta/ijbb_template_pair_label.txt
    ```
  - See `-h` for detail usage.
    ```sh
    python IJB_evals.py -h
    ```
***

# Related Projects
  - [TensorFlow Addons Losses: TripletSemiHardLoss](https://www.tensorflow.org/addons/tutorials/losses_triplet)
  - [TensorFlow Addons Layers: WeightNormalization](https://www.tensorflow.org/addons/tutorials/layers_weightnormalization)
  - [Github deepinsight/insightface](https://github.com/deepinsight/insightface)
  - [Github cavalleria/cavaface.pytorch](https://github.com/cavalleria/cavaface.pytorch)
  - [Github titu1994/keras-squeeze-excite-network](https://github.com/titu1994/keras-squeeze-excite-network)
  - [Github qubvel/EfficientNet](https://github.com/qubvel/efficientnet)
  - [Github QiaoranC/tf_ResNeSt_RegNet_model](https://github.com/QiaoranC/tf_ResNeSt_RegNet_model)
  - [Partial FC: Training 10 Million Identities on a Single Machine](https://arxiv.org/pdf/2010.05222.pdf)
***
