# [正则项](https://keras-cn.readthedocs.io/en/latest/other/regularizers/)

正则项在优化过程中层的参数或层的激活值添加惩罚项，这些惩罚项将与损失函数一起作为网络的最终优化目标

惩罚项基于层进行惩罚，目前惩罚项的接口与层有关，但`Dense`, `Conv1D`, `Conv2D`, `Conv3D`具有共同的接口。

### 使用例

    from keras import regularizers
    model.add(Dense(64, input_dim=64，
                    kernel_regularizer=regularizers.l2(0.01),
                    activity_regularizer=regularizers.l1(0.01)))

### 开发新的正则项

任何以权重矩阵作为输入并返回单个数值的函数均可以作为正则项，例如：

    from keras import backend as K

    def l1_reg(weight_matrix):
        return 0.01 * K.sum(K.abs(weight_matrix))

    model.add(Dense(64, input_dim=64,
                    kernel_regularizer=l1_reg)

# [LSTM层](https://keras-cn.readthedocs.io/en/latest/layers/recurrent_layer/)

    keras.layers.recurrent.LSTM(units, activation='tanh', recurrent_activation='hard_sigmoid', use_bias=True, kernel_initializer='glorot_uniform', recurrent_initializer='orthogonal', bias_initializer='zeros', unit_forget_bias=True, kernel_regularizer=None, recurrent_regularizer=None, bias_regularizer=None, activity_regularizer=None, kernel_constraint=None, recurrent_constraint=None, bias_constraint=None, dropout=0.0, recurrent_dropout=0.0)

### 参数

- `kernel_regularizer`: 施加在权重上的正则项
- `bias_regularizer`：施加在偏置向量上的正则项
- `recurrent_regularizer`：施加在循环核上的正则项
- `activity_regularizer`：施加在输出j上的正则项

# [How to Use Weight Regularization with LSTM Networks for Time Series Forcasting](http://machinelearningmastery.com/use-weight-regularization-lstm-networks-time-series-forecasting/)