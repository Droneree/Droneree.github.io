---
title: AI学习笔记--Tensorflow自定义
date: 2021-06-28 15:00:01
categories:
- 计算机科学
tags: 
- python
- tensorflow
- 人工智能
mathjax: true
---



Tensorflow框架下自定义loss，metrics，training_loop，layer的一些实践经验。<!--more-->

### 自定义损失函数（Loss）

#### 1. Custom loss function with extra arguments

Keras库的损失函数标准的输入参数形式是`loss(y_true, y_pred)`，面向的使用场景是最常见的预测与目标（或标签）的比较。而我希望计算损失函数时用到模型的input作为输入参数，所以需要自定义一个损失函数。

在tensorflow[官方doc](https://www.tensorflow.org/guide/keras/train_and_evaluate#custom_losses)中，给出了两种自定义损失函数的方法。一种是定义一个loss function，另一种是继承`tf.keras.losses.Loss`类。其中第二种允许输入除`(y_true, y_pred)`之外的其他参量。

```python
# custom loss function with extra arguments
class LocLoss(keras.losses.Loss):
    def __init__(self, kg_raw, model_size, nobs, batch_size, name="custom_loss"):
        super().__init__(name=name)
        # these are some extra arguments:
        self.kg_raw = kg_raw 
        self.model_size = model_size
        self.nobs = nobs
        self.batch_size =  batch_size

    def call(self, kg_true, loc_f):
        k = tf.constant([1, 1, self.nobs, 1], tf.int32)
        kg_pred = tf.math.multiply(self.kg_raw, tf.tile(tf.reshape(loc_f, [self.batch_size,self.model_size,1,1]), k))
        mse = tf.math.reduce_mean(tf.square(kg_true - kg_pred))
        return mse
      
# create a model
# specify model structure...
model = keras.Model(inputs=inputs, outputs=outputs)

# compile the model
model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=0.0001),
    loss=LocLoss(inputs,model_size,nobs,batch_size), # model inputs as extra args
    experimental_run_tf_function=False, # neccecarry !
)

# train the model
history = model.fit(train_dataset, epochs=10, validation_data=test_dataset)

```

模型的input是kg_raw，output是一个函数loc_f，target是kg_true，loss是$MSE(kg_{raw}*loc_f, kg_{true})$。注意为了能够让tensorflow自动求导，所有loss中的参量都要以tensorflow tensor的形式计算。

在自定义LocLoss class中，`__init__`方法接受extra arguments（kg_raw, model_size, nobs, batch_size）；`call`方法计算loss，它的输入参数仍然是经典的`(y_true, y_pred)`，只不过这里的y_pred是模型输出的loc_f，计算时需要用到的extra arguments用self.*args调用。

在compile model时将inputs作为输入参数，就实现了含其他参数的损失函数的自定义。这里会碰到一个问题，如果不设置**experimental_run_tf_function=False**，编译会报错（Check [this](https://github.com/tensorflow/tensorflow/issues/32142) for more information）

```
tensorflow.python.eager.core._SymbolicException: Inputs to eager execution function cannot be Keras symbolic tensors, but found [<tf.Tensor '...' shape=(...) dtype=float32>]
```



#### 2. Weighted loss

待填坑。

#### 3. Load pre-trained model and predict

上述含model inputs作为参数自定义loss的训练好的模型，想要保存下来并且之后用来预测。若将整个模型保存，之后load时会报错：

```python
model.save('/save_dir')

trained_model = tf.keras.models.load_model('/save_dir',custom_objects={'loss': LocLoss(inputs)})

```

解决办法是只保存模型的weights，load时也只load weights：

```python
model.save_weights('/save_dir')

model.load_weights('/save_dir')

```

在load之前要预先定义好模型并且compile。

【附】自定义损失函数的一些常见参考问题：

- [Tensorflow2.0中复杂损失函数实现(custom layer + add_loss)](https://zhuanlan.zhihu.com/p/74009996)
- [Tensorflow 2.0 Custom loss function with multiple inputs / mutiple loss](https://stackoverflow.com/questions/58022713/tensorflow-2-0-custom-loss-function-with-multiple-inputs)
- [Tensorflow2.0与Keras搭建个性化神经网络模型](https://www.cnblogs.com/qizhou/p/13530440.html)
- [Loading a keras model with custom loss based on input](https://stackoverflow.com/questions/59729481/loading-a-keras-model-with-custom-loss-based-on-input)
- [Load custom loss with extra input in keras](https://stackoverflow.com/questions/57897080/load-custom-loss-with-extra-input-in-keras)

<br/>

### 自定义评价函数（Metrics）

tensorflow官方[doc](https://www.tensorflow.org/guide/keras/train_and_evaluate#custom_metrics)中也给出了自定义metrics的方法，同自定义loss类似，可以继承`keras.metrics.Metric`类，写好后放进`model.compile()`里编译。

我自定义metrics的动机是想记录模型训练过程中gradient norm的变化，来检查最优化有没有卡住。gradient在tensorflow 2.0之后可以方便的从`GradientTape()`得到，所以需要go lower level，看看在训练时发生了什么。具体需要改写的是`keras.Model`类中的`train_step`方法，该方法决定了`model.fit()`时计算gradient, loss, metrics以及梯度下降的过程。tensorflow官方[doc](https://www.tensorflow.org/guide/keras/customizing_what_happens_in_fit)有详细解释。

```python
# custom metrics 
class GradientNorm(tf.keras.metrics.Metric):
  def __init__(self, name='gradient_norm', **kwargs):
    super().__init__(name=name, **kwargs)
    self.gd_norm = self.add_weight(name='gdnorm', initializer='zeros') 	# variable must be assigned by add_weight method

  def update_state(self, y, gradients, sample_weight=None):
    norm = tf.norm(gradients, ord='euclidean') / tf.cast(tf.size(gradients, out_type=tf.int32), tf.float32)
    self.gd_norm.assign(norm) # variable must be updated by add_weight method

  def result(self):
    return self.gd_norm

  def reset_states(self):
    self.gd_norm.assign(0)
 
# custom what happened during training by overriding train_step()
class CustomModel(keras.Model):
    def train_step(self, data):
        # Unpack the data. Its structure depends on your model and
        # on what you pass to `fit()`.
        x, y = data

        with tf.GradientTape() as tape:
            y_pred = self(x, training=True)  # Forward pass
            # Compute the loss value
            # (the loss function is configured in `compile()`)
            loss = self.compiled_loss(y, y_pred, regularization_losses=self.losses)

        # Compute gradients
        trainable_vars = self.trainable_variables
        gradients = tape.gradient(loss, trainable_vars)

        # Update weights
        self.optimizer.apply_gradients(zip(gradients, trainable_vars))

        # Update metrics
        self.compiled_metrics.reset_states()
        self.compiled_metrics.update_state(y, gradients)

        # Return a dict mapping metric names to current value
        return_metrics = {m.name: m.result() for m in self.metrics}
        return return_metrics

# create a model
# specify model structure...
model = CustomModel(inputs=inputs, outputs=outputs)

# compile the model
model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=0.0001),
    loss=LocLoss(inputs,model_size,nobs,batch_size), # model inputs as extra args
    experimental_run_tf_function=False, # neccecarry !
    metrics=[GradientNorm()], # custom metrics
)

```

这种方法是通过compiled_loss, compiled_metrics计算和更新。如果想要添加多个metrics，而且不同的metrics有不同的输入参量（上面的方法compiled_metrics固定输入参量只能是(y, gradients)），则需要下面的方法，这种方法不在`model.compile()`时指定loss和metrics。

```python
gd_norm = GradientNorm()
# PSNR is another custom metric, which takes (y_true, y_pred) as input
psnr_metric = PSNR() 
loss_tracker = keras.metrics.Mean(name="loss")

class CustomModel(keras.Model):
    def train_step(self, data):
        # Unpack the data. Its structure depends on your model and
        # on what you pass to `fit()`.
        x, y = data

        with tf.GradientTape() as tape:
            y_pred = self(x, training=True)  # Forward pass
            # Compute the loss value
            loss = keras.losses.mean_squared_error(y, y_pred)

        # Compute gradients
        trainable_vars = self.trainable_variables
        gradients = tape.gradient(loss, trainable_vars)

        # Update weights
        self.optimizer.apply_gradients(zip(gradients, trainable_vars))

        # Update metrics (includes the metric that tracks the loss)
        gd_norm.update_state(y, gradients)
        psnr_metric.update_state(y, y_pred)
        loss_tracker.update_state(loss)

        # Return a dict mapping metric names to current value
        return_metrics = {m.name: m.result() for m in self.metrics}
        return return_metrics

    @property
    def metrics(self):
        # We list our `Metric` objects here so that `reset_states()` can be
        # called automatically at the start of each epoch
        # or at the start of `evaluate()`.
        # If you don't implement this property, you have to call
        # `reset_states()` yourself at the time of your choosing.
        return [loss_tracker, gd_norm, psnr_metric]

```

不过这种办法只在tensorflow 2.2以上版本支持，否则会报错（check [this](https://github.com/tensorflow/tensorflow/issues/40041) for more info）

```
ValueError: The model cannot be compiled because it has no loss to optimize.
```

<br/>

### 自定义训练过程（Training Loop)



<br/>

### 自定义层（Layer）

在tensorflow自定义layer有两种选择：继承Layer类或继承Lambda类。这两种都属于`tensorflow.keras.layers`类。

tensorflow官方更推荐继承Layer类而不是Lambda类。原因可参考[官方文档](https://www.tensorflow.org/api_docs/python/tf/keras/layers/Lambda)，和[这篇](https://github.com/stellargraph/stellargraph/issues/709)。通常来说，只有非常简单的操作才推荐用Lambda层，有两个原因，一是因为Lambda层在save model和load model时需要相同的环境配置，这使得模型没那么便携；另一个是用了Lambda层的model在debug的时候很不方便。

下面是一个继承Layer类编写自定义层的例子。

```python
from tensorflow.keras import layers

class Loc_by_1D(layers.Layer):
        def __init__(self, obs_dens, model_size, nobs, batch_size, **kwargs):
            super(Loc_by_1D, self).__init__(**kwargs)
            self.obs_dens = obs_dens
            self.model_size = model_size
            self.nobs = nobs
            self.batch_size = batch_size

        def rotate(self, matrix, shifts):
            """"here are some codes"""
            
        def call(self, inputs):
            kg_f = tf.squeeze(inputs[0]) # multiple inputs as a list
            loc_f = tf.squeeze(inputs[1]) # multiple inputs as a list

						"""some operations on inputs"""
            return kg_pred
          
        def compute_output_shape(self, input_shape):
            shape = list(input_shape)
            assert len(shape) == 2
            return tuple([shape[0], self.model_size, self.nobs])

        def get_config(self):
            config = {'obs_dens': self.obs_dens, 'model_size': self.model_size,
                      'nobs': self.nobs, 'batch_size': self.batch_size}
            base_config = super(Loc_by_1D, self).get_config()
            return dict(list(base_config.items()) + list(config.items()))
          
# the last layer of my keras model
outputs = Loc_by_1D(obs_dens=obs_dens, model_size=model_size, nobs=nobs, batch_size=kwargs['batch_size'])([inputs, x5]) # here the inputs is a list [inputs, x5]

```

这里我自定义了一个叫Loc_by_1D的层，其中包含几个主要的函数

- \__init\__用来初始化，指定一些参数；
- call里一般写的是这个layer的核心功能。当输入给call function的inputs不只一个时，最好用list的形式传递参数（如例子中那样），参考[这篇](https://stackoverflow.com/questions/61891181/how-to-use-multiple-inputs-in-tensorflow-2-x-keras-custom-layer)。
- compute_output_shape用来计算该层输出张量的shape，不是必须的，除了输出shape是动态的情况，都会自动计算；
- get_config用来检查model的时候改层有哪些指定参数。

在自定义的layer里，各种操作也只能用tensorflow的函数库，比如`tf.squeeze()`对应numpy里的`np.squeeze()`，一般来说，numpy里一些简单常用的操作都能在tensorflow找到对应，只不过操作的对象不再是`np.array`而是`tensor`，后者是tensorflow里最常用的数组变量。

<br/>

<br/>

<br/>