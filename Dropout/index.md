# [Dropout: A simple way to prevent neural networks from overfitting](http://jmlr.org/papers/v15/srivastava14a.html)

Deep neural nets with a large number of parameters are very powerful machine learning systems. However, overfitting is a serious problem in such networks. Dropout is a technique for addressing this problem. **The key idea** is to randomly drop units from the neural network during training. This prevents units from co-adapting too much.

# [LSTM层](https://keras-cn.readthedocs.io/en/latest/layers/recurrent_layer/)

	keras.layers.recurrent.LSTM(units, activation='tanh', recurrent_activation='hard_sigmoid', use_bias=True, kernel_initializer='glorot_uniform', recurrent_initializer='orthogonal', bias_initializer='zeros', unit_forget_bias=True, kernel_regularizer=None, recurrent_regularizer=None, bias_regularizer=None, activity_regularizer=None, kernel_constraint=None, recurrent_constraint=None, bias_constraint=None, dropout=0.0, recurrent_dropout=0.0)

- `dropout`: 0~1之间的浮点数，控制输入线性变换的神经元断开比例
- `recurrent_dropout`: 0~1之间的浮点数，控制循环状态的线性变换的神经元断开比例

# [Dropout层](https://keras-cn.readthedocs.io/en/latest/layers/core_layer/#dropout)

	keras.layers.core.Dropout(rate, noise_shape=None, seed=None)

为输入数据施加Dropout。Dropout将在训练过程中每次更新参数时随机断开一定百分比的输入神经元，Dropout层用于防止过拟合。

### 参数

- `rate`: 0~1的浮点数，控制需要断开的神经元的比例
- `noise_shape`：整数张量，为将要应用在输入上的二值Dropout mask的shape，例如你的输入为`(batch_size, timesteps, features)`，并且你希望在各个时间步上的Dropout mask都相同，则可传入`noise_shape=(batch_size, 1, features)`。
- `seed`：整数，使用的随机数种子

# [Dropout Regularization in Keras](http://machinelearningmastery.com/dropout-regularization-deep-learning-models-keras/)

Dropout is easily implemented by randomly selecting nodes to be dropped-out with a given probability each weight update cycle. This is how Dropout is implemented in Keras. Dropout is only used during the training of a model and is not used when evaluating the skill of the model.

### Tips for using dropout

- Generally, use a small dropout value of 20%-50% of neurons with 20% providing a good starting point. A probability too low has minimal effect and a value too high results in under-learning by the network.
- Using a larger network. You are likely to get better performance when dropout is used on a larger network, giving the model more of an opportunity to learn independent representations.
- Use dropout on incoming (visible) as well as hidden units. Application of dropout at each layer of the network has shown good results.
- Use a large learning rate with decay and a large momentum. Increase your learning rate by a factor of 10 to 100 and use a high momentum value of 0.9 or 0.99.
- Constrain the size of network weights. A large learning rate can result in very large network weights. Imposing a constraint on the size of network weights such as max-norm regularization with a size of 4 or 5 has been shown to improve results.