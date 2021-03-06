## Module
Root class of all network modules, you'd always subclass this for a new module

```python
class Module(name=None, work_mode='inference')
```
* **name**: module name, optional. If you don't specify the module name, it will be auto-named if this module is a sub-module of another module.
* **work_mode**: working mode, optional. Only used for the unified calling interface, check "Tutorial I" for detailed explanation.

```python
.params                  = []  
.self_updating_variables = [] 
.sub_modules             = OrderedDict()
.name                    = name
.work_mode               = work_mode
```
* **params**: contains all the parameters which should be updated by optimizer (submodule excluded)
* **self_updating_variables**: contains all the parameters which are updated by user specified expression (submoduluded)
* **sub_modules**: contains all the sub-modules

```python
.register_param(x, shape=None, name=None)
.register_self_updating_variable(x, shape=None, name=None)
```
Register and possibly initialize a parameter tensor. Parameters to be updated by optimizer should be registered with `register_param()` meanwhile parameters self-updated should be registerd with `register_self_updating_variable()`

* **x**: Theano shared variable, expression, numpy array or callable. Initial value, expression or initializer for this parameter.
* **shape**: tuple of int, optional. A tuple of integers representing the desired shape of the parameter tensor.
* **name**: str, optional. It's recommended to let the Dandelion framework name the variable automatically.

```python
.collect_params(include=None, exclude=None, include_self=True)
```
Collect parameters to be updated by optimizer.

* **include**: sub-module keys, means which sub-module to include
* **exclude**: sub-module keys, means which sub-module to exclude
* **include_self**: whether include `self.params`
* **return**: list of parameters, in the same order of sub-modules

```python
.collect_self_updates(include=None, exclude=None, include_self=True)
```
Collect all `update` from self_updating_variables.

* **include**: sub-module keys, means which sub-module to include
* **exclude**: sub-module keys, means which sub-module to exclude
* **include_self**: whether include `self.self_updating_variables`
* **return**: update dict, in the same order of sub-modules

```python
.get_weights()
```
Collect all module weights (including submodules)

* **return**: list of tuples with format [variable.value, variable.name]

```python
.set_weights(module_weights, check_name='ignore')
```
Set module weights by default order (same order with `.get_weights()`)

* **module_weights**: same with the return of `.get_weights()`
* **check_name**: `ignore`|`warn`|`raise`. What to do if a weight's name does not match its corresponding variable's name.

```python
.set_weights_by_name(module_weights, unmatched='raise')
```
Set module weights by matching name.

* **module_weights**: same with the return of `.get_weights()`
* **unmatched**:  `ignore`|`warn`|`raise`. What to do if there remain weights or module variables unmatched.

_______________________________________________________________________
## Dropout
Sets values to zero with probability `p`

```python
class Dropout(seed=None, name=None)
```
* **seed**: the random seed (integer) for initialization, optional

```python
.forward(input, p=0.5, shared_axes=(), rescale=True)
```
* **p**: ﬂoat or scalar tensor. The probability of setting a value to zero
* **shared_axes**: tuple of int. Axes to share the dropout mask over. By default, each value can be dropped individually. `shared_axes`=(0,) uses the same mask across the batch. `shared_axes`=(2, 3) uses the same mask across the spatial dimensions of 2D feature maps.
* **rescale**: bool. If True (the default), scale the input by 1 / (1 - `p`) when dropout is enabled, to keep the expected output mean the same.

```python
.predict( input, *args, **kwargs)
```

dummy interface, does nothing but returns the input unchanged.

Note: Theano uses `self_update` mechanism to implement pseudo randomness, so to use `Dropout` class, the followings are recommened:

* (1) define different instance for each droput layer
* (2) compiling function with `no_default_updates=False`

_______________________________________________________________________
## GRU
Gated Recurrent Unit RNN.

```python
class GRU(input_dims, hidden_dim, initializer=init.Normal(0.1), grad_clipping=0, 
          hidden_activation=tanh, learn_ini=False, truncate_gradient=-1, name=None)
```
* **input_dims**: integer or list of integers. If scalar, input dimension = `input_dims`; if list of integers, input dimension = sum(`input_dims`), and GRU’s parameter `W_in` will be initialized unevenly by integers specified in input_dims
* **hidden_dim**: dimension of hidden units, also the output dimension
* **grad_clipping**: float. Hard clip the gradients at each time step. Only the gradient values above this threshold are clipped to the threshold. This is done during backprop. Some works report that using grad_normalization is better than grad_clipping
* **hidden_activation**: nonlinearity applied to hidden variable, i.e., h = hidden_activation(cell). It's recommended to use `tanh` as default.
* **learn_ini**: whether learn initial state
* **truncate_gradient**: if not -1, BPTT will be used, gradient back-propagation will be performed at most `truncate_gradient` steps

```python
.forward(seq_input, h_ini=None, seq_mask=None, backward=False, only_return_final=False, return_final_state=False)
```
* **seq_input**: tensor with shape (T, B, D) in which D is the input dimension
* **h_ini**: initialization of hidden cell, (B, hidden_dim)
* **seq_mask**: mask for `seq_input`
* **backward**: bool. Whether scan in backward direction
* **only_return_final**: bool. If `True`, only return the ﬁnal sequential output (e.g. for tasks where a single target value for the entire sequence is desired). In this case, Theano makes an optimization which saves memory.
* **return_final_state**: If `True`, the final state of `hidden` and `cell` will be returned, both (B, hidden_dim)

```python
.predict = .forward
```

_______________________________________________________________________
## LSTM
Long Short-Term Memory RNN

```python
class LSTM( input_dims, hidden_dim, peephole=True, initializer=init.Normal(0.1), grad_clipping=0, 
            hidden_activation=tanh, learn_ini=False, truncate_gradient=-1, name=None)
```
* **input_dims**: integer or list of integers. If scalar, input dimension = `input_dims`; if list of integers, input dimension = sum(`input_dims`), and LSTM’s parameter `W_in` will be initialized unevenly by integers specified in input_dims
* **hidden_dim**: dimension of hidden units, also the output dimension
* **peephole**: bool. Whether add peephole connection.
* **grad_clipping**: float. Hard clip the gradients at each time step. Only the gradient values above this threshold are clipped to the threshold. This is done during backprop. Some works report that using grad_normalization is better than grad_clipping
* **hidden_activation**: nonlinearity applied to hidden variable, i.e., h = hidden_activation(cell). It's recommended to use `tanh` as default.
* **learn_ini**: whether learn initial state
* **truncate_gradient**: if not -1, BPTT will be used, gradient back-propagation will be performed at most `truncate_gradient` steps

```python
.forward(seq_input, h_ini=None, c_ini=None, seq_mask=None, backward=False, only_return_final=False, return_final_state=False)
```
* **seq_input**: tensor with shape (T, B, D) in which D is the input dimension
* **h_ini**: initialization of hidden state, (B, hidden_dim)
* **c_ini**: initialization of cell state, (B, hidden_dim)
* **seq_mask**: mask for seq_input
* **backward**: bool. Whether scan in backward direction
* **only_return_final**: bool. If `True`, only return the ﬁnal sequential output (e.g. for tasks where a single target value for the entire sequence is desired). In this case, Theano makes an optimization which saves memory.
* **return_final_state**: If `True`, the final state of `hidden` and `cell` will be returned, both (B, hidden_dim)
```python
.predict = .forward
```

_______________________________________________________________________
## GRUCell
Gated Recurrent Unit RNN Cell

```python
class GRUCell(input_dims, hidden_dim, initializer=init.Normal(0.1), grad_clipping=0, 
              hidden_activation=tanh, name=None)
```
* **input_dims**: integer or list of integers. If scalar, input dimension = `input_dims`; if list of integers, input dimension = sum(`input_dims`), and GRUCell’s parameter `W_in` will be initialized unevenly by integers specified in input_dims
* **hidden_dim**: dimension of hidden units, also the output dimension
* **grad_clipping**: float. Hard clip the gradients at each time step. Only the gradient values above this threshold are clipped to the threshold. This is done during backprop. Some works report that using grad_normalization is better than grad_clipping
* **hidden_activation**: nonlinearity applied to hidden variable, i.e., h = hidden_activation(cell). It's recommended to use `tanh` as default
```python
.forward(input, h_pre, mask=None)
```
* **input**: tensor with shape (B, D) in which D is the input dimension
* **h_pre**: initialization of hidden cell, (B, hidden_dim)
* **mask**: mask for `input`
```python
.predict = .forward
```

_______________________________________________________________________
## LSTMCell
Long Short-Term Memory RNN Cell
```python
class LSTMCell(input_dims, hidden_dim, peephole=True, initializer=init.Normal(0.1), grad_clipping=0, 
               hidden_activation=tanh, name=None)
```
* **input_dims**: integer or list of integers. If scalar, input dimension = `input_dims`; if list of integers, input dimension = sum(`input_dims`), and LSTM’s parameter `W_in` will be initialized unevenly by integers specified in input_dims
* **hidden_dim**: dimension of hidden units, also the output dimension
* **peephole**: bool. Whether add peephole connection.
* **grad_clipping**: float. Hard clip the gradients at each time step. Only the gradient values above this threshold are clipped to the threshold. This is done during backprop. Some works report that using grad_normalization is better than grad_clipping
* **hidden_activation**: nonlinearity applied to hidden variable, i.e., h = hidden_activation(cell). It's recommended to use `tanh` as default.

```python
.forward(input, h_pre, c_pre, mask=None)
```
* **input**: tensor with shape (B, D) in which D is the input dimension
* **h_pre**: initialization of hidden state, (B, hidden_dim)
* **c_pre**: initialization of cell state, (B, hidden_dim)
* **mask**: mask for `input`
```python
.predict = .forward
```

_______________________________________________________________________
## Conv2D
Convolution 2D
```python
class Conv2D(in_channels, out_channels, kernel_size=(3,3), stride=(1,1), pad='valid', 
             dilation=(1,1), num_groups=1, W=init.GlorotUniform(), b=init.Constant(0.), 
             flip_filters=True, convOP=tensor.nnet.conv2d, input_shape=(None,None), untie_bias=False, name=None)
```
* **input_channels**: int. Input shape of Conv2D module is (B, input_channels, H_in, W_in)
* **out_channels**: int. Output shape of Conv2D module is (B output_channels, H_out, W_out)
* **kernel_size**: int scalar or tuple of int. Convolution kernel size
* **stride**: Factor by which to subsample the output
* **pad**: `same`/`valid`/`full` or 2-element tuple of int. Control image border padding.
* **dilation**: factor by which to subsample (stride) the input.
* **num_groups**: Divides the image, kernel and output tensors into num_groups separate groups. Each which carry out convolutions separately
* **W**: initialization of filter bank, shape = (out_channels, in_channels, kernel_size[0], kernel_size[1])
* **b**: initialization of convolution bias, shape = (out_channels,) if untie_bias is False; otherwise shape = (out_channels, H_out, W_out)
* **flip_filters**: If `True`, will flip the filter rows and columns before sliding them over the input. This operation is normally referred to as a convolution, and this is the default. If `False`, the filters are not flipped and the operation is referred to as a cross-correlation.
* **input_shape**: optional, (H_in, W_in)
* **untie_bias**: If `False`, the module will have a bias parameter for each channel, which is shared across all positions in this channel. As a result, the b attribute will be a vector (1D). If `True`, the module will have separate bias parameters for each position in each channel. As a result, the b attribute will be a 3D tensor.

_______________________________________________________________________
## ConvTransposed2D
Transposed convolution 2D. Also known as fractionally-strided convolution or deconvolution (although it is not an actual deconvolution operation)
```python
class ConvTransposed2D(in_channels, out_channels, kernel_size=(3,3), stride=(1,1), pad='valid', 
                       dilation=(1,1), num_groups=1, W=init.GlorotUniform(), b=init.Constant(0.), 
                       flip_filters=True, input_shape=(None,None), untie_bias=False, name=None)
```
* **return**: output shape = `(B, C, H, W)`, in which `H = ((H_in - 1) * stride_H) + kernel_H - 2 * pad_H`, and the same with `W`.

All the parameters have the same meanings with `Conv2D` module. In fact, the transposed convolution is equal to upsampling the input then doing conventional convolution. However, for efficiency purpose, here the transposed convolution is implemented via Theano’s `AbstractConv2d_gradInputs` as what is done in Lasagne.

_______________________________________________________________________
## Dense
Fully connected network, also known as affine transform. Apply affine transform `Wx+b` to the last dimension of input.  
The input of `Dense` can have any dimensions, and note that we do not apply any activation to its output by default
```python
class Dense(input_dims, output_dim, W=init.GlorotUniform(), b=init.Constant(0.), name=None)
```
* **input_dims**: integer or list of integers. If scalar, input dimension = input_dims; if list of integers, input dimension = sum(input_dims), and Dense’s parameter `W` will be initialized unevenly by integers specified in input_dims
* **output_dim**: output dimension
* **W**, **b**: parameter initialization

_______________________________________________________________________
## Embedding
Word/character embedding module.

```python
class Embedding(num_embeddings, embedding_dim, W=init.Normal(), name=None)
```
* **num_embeddings**: the Number of different embeddings
* **embedding_dim**: output embedding vector dimension

_______________________________________________________________________
## BatchNorm
Batch normalization module.

```python
class BatchNorm(input_shape=None, axes='auto', eps=1e-4, alpha=0.1, beta=init.Constant(0), gamma=init.Constant(1), 
                mean=init.Constant(0), inv_std=init.Constant(1), mode='high_mem', name=None)
```
* **input_shape**: Tuple or list of ints or tensor variables. Input shape of `BatchNorm` module, including batch dimension. 
* **axes**: `auto` or tuple of int. The axis or axes to normalize over. If `auto` (the default), normalize over all axes except for the second: this will normalize over the minibatch dimension for dense layers, and additionally over all spatial dimensions for convolutional layers.
* **eps**: Small constant 𝜖 added to the variance before taking the square root and dividing by it, to avoid numerical problems
* **alpha**: Coefficient for the exponential moving average of batch-wise means and standard deviations computed during training; the closer to one, the more it will depend on the last batches seen
* **mode**: `low_mem` or `high_mem`. Specify which batch normalization implementation that will be used. As no intermediate representations are stored for the back-propagation, `low_mem` implementation lower the memory usage, however, it is 5-10% slower than `high_mem` implementation. Note that 5-10% computation time difference compare the batch normalization operation only, time difference between implementation is likely to be less important on the full model fprop/bprop.

```python
.forward(input, use_input_mean=True)
```
* **use_input_mean**: default, use mean & std of input batch for normalization; if `False`, `self.mean` and `self.std` will be used for normalization. The reason that input mean is used during training is because at the early training stage, `BatchNorm`'s `self.mean` is far from the expected mean value and can be detrimental for network convergence. It's recommended to use input mean for early stage training; after that, you can switch to `BatchNorm`'s `self.mean` for training & inference consistency.

_______________________________________________________________________
## Center
Estimate class centers by moving averaging.

```python
class Center(feature_dim, center_num, alpha=0.1, center=init.GlorotUniform(), name=None)
```
* **feature_dim**: feature dimension 
* **center_num**: class center number
* **alpha**: moving averaging coefficient, the closer to one, the more it will depend on the last batches seen
* **center**: initialization of class centers, should be in shape of `(center_num, feature_dim)`

```python
.forward(features, labels)
```
* **features**: batch features, from which the class centers will be estimated
* **labels**: `features`'s corresponding class labels
* **return**: centers estimated
```python
.predict()
```
* **return**: centers stored

_______________________________________________________________________
## ChainCRF
Linear chain CRF layer for sequence labeling.

```python
class ChainCRF(state_num, transitions=init.GlorotUniform(), p_scale=1.0, l1_regularization=0.001, 
               state_pad=True, transition_matrix_normalization=True,  name=None)
```
* **state_num**: number of hidden states. If `state_pad` is `True`, then the actual state number inside CRF will be `state_num + 2`.
* **transitions**: initialization of transition matrix, in shape of `(state_num+2, state_num+2)` if `state_pad` is `True`, else `(state_num, state_num)`
* **p_scale**: probability scale factor. The input of this module will be multiplied by this factor.
* **l1_regularization**: L1 regularization coefficient for `transition` matrix
* **state_pad**: whether do state padding. CRF requires two additional dummy states, i.e., `<bos>` and `<eos>` (beginning and endding of sequence). The `ChainCRF` module can pad the state automatically with these two dummy states, or you can incorporate these two states in module input. In the latter case, set `state_pad` to `False`.
* **transition_matrix_normalization**: whether do row-wise normalization of transition matrix. You may expect that each row of the `transition` matrix should sum to 1.0, and to do this, set this flag to `True`.

```python
.forward(x, y)
```
Compute CRF loss

* **x**: output from previous RNN layer, in shape of (B, T, N)
* **y**: tag ground truth, in shape of (B, T), int32
* **return**: loss in shape of (B,) if `l1_regularization` disabled, else in shape of (1,)
```python
.predict(x)
```
CRF Viterbi decoding

* **x**: output from previous RNN layer, in shape of (B, T, N)
* **return**: decoded sequence

_______________________________________________________________________
## Sequential
Sequential container for a list of modules, just for convenience.

```python
class Sequential(module_list, activation=linear, name=None)
```
* **module_list**: list of network sub-modules, these modules **MUST NOT** be sub-modules of any other parent module.
* **activation**: activation applied to output of each sub-module.

```python
.forward(x)
```
Forward pass through the network module sequence.

```python
.predict(x)
```
Inference pass through the network module sequence.

Example:
```python
        conv1 = Conv2D(in_channels=1, out_channels=3, stride=(2,2))
        bn1   = BatchNorm(input_shape=(None, 3, None, None))
        conv2 = Conv2D(in_channels=3, out_channels=5)
        conv3 = Conv2D(in_channels=5, out_channels=8)
        model = Sequential([conv1, bn1, conv2, conv3], activation=relu, name='Seq')

        x = tensor.ftensor4('x')
        y = model.forward(x)
        print('compiling fn...')
        fn = theano.function([x], y, no_default_updates=False)
        print('run fn...')
        input = np.random.rand(4, 1, 32, 33).astype(np.float32)
        output = fn(input)
        print(output)
```