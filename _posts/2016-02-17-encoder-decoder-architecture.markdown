---
layout:     post
date:       2016-02-17 22:00:00
author:     "Jakub Kvita"
comments:   true
title:      "Encoder-Decoder Architecture"
subtitle:   "How to initialize the hidden state of the LSTM and GRU from the rnn package"
---

Neural networks with encoder-decoder architecture became very popular during 2015, mainly because of the [Sequence to Sequence Learning with Neural Networks](http://arxiv.org/abs/1409.3215) paper. The authors used multilayered LSTM to map input sequence to a vector with fixed dimensionality. This vector was then unrolled to the output sequence by second LSTM network. Architecture was applied to machine translation task.

To refresh the knowledge about LSTMs and its variants go and read a very good article [Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/). To sum it up, LSTM has two hidden states, which can be initialized - the cell state `C[t-1]` and the output `h[t-1]`. GRU has just one (`h[t-1]`), but it works in the similar fashion.

Due to the [many requests](https://github.com/Element-Research/rnn/issues/16), is is possible to update the hidden states directly in the layers of the [Element-Research/rnn](https://github.com/Element-Research/rnn) package. The `nn.LSTM` uses variable `userPrevCell` to initialize cell state and `userPrevOutput` to insert previous output. `nn.GRU` has only the `userPrevOutput` variable.

If one of the mentioned variables is defined, after the forward+backward cycle, respective gradient variable (`userGradPrevCell` or `userGradPrevOutput`) will contain computed gradient.

### Example

Fully working example is available [here](https://github.com/Element-Research/rnn/blob/master/examples/encoder-decoder-coupling.lua). Example uses two short functions:

~~~ lua
--[[ Forward coupling: Copy encoder cell and output to decoder LSTM ]]--
function forwardConnect(encLSTM, decLSTM)
  decLSTM.userPrevOutput = nn.rnn.recursiveCopy(decLSTM.userPrevOutput, encLSTM.outputs[opt.inputSeqLen])
  decLSTM.userPrevCell = nn.rnn.recursiveCopy(decLSTM.userPrevCell, encLSTM.cells[opt.inputSeqLen])
end

--[[ Backward coupling: Copy decoder gradients to encoder LSTM ]]--
function backwardConnect(encLSTM, decLSTM)
  encLSTM.userNextGradCell = nn.rnn.recursiveCopy(encLSTM.userNextGradCell, decLSTM.userGradPrevCell)
  encLSTM.gradPrevOutput = nn.rnn.recursiveCopy(encLSTM.gradPrevOutput, decLSTM.userGradPrevOutput)
end
~~~

These two functions are all which is necessary to connect two networks. Forward and backward calls are very simple:

~~~ lua
-- Forward pass
local encOut = enc:forward(encInSeq)
forwardConnect(encLSTM, decLSTM)
local decOut = dec:forward(decInSeq)
local Edec = criterion:forward(decOut, decOutSeq)

-- Backward pass
local gEdec = criterion:backward(decOut, decOutSeq)
dec:backward(decInSeq, gEdec)
backwardConnect(encLSTM, decLSTM)
local zeroTensor = torch.Tensor(2):zero()
enc:backward(encInSeq, zeroTensor)
~~~

`zeroTensor` variable is used, because actual output of the encoder network doesn't matter. Therefore gradient is zero.

### Example 2

Encoder-Decoder architecture can be also used with heterogenous networks like CNN together with RNN. The following snippet is used in training of the image captioning network. Connection functions are not necessary, as the values are copied directly from and to the output and the output gradient of the convolutional net.

~~~ lua
local lstm = rnn:get(1)

cnn:forward(image)
lstm.userPrevOutput = nn.rnn.recursiveCopy(lstm.userPrevOutput, cnn.output)

local prediction = rnn:forward(inputs[i])
error = criterion:forward(prediction, targets[i])/#(targets[i])
local gradOutputs = criterion:backward(prediction, targets[i])
rnn:backward(inputs[i], gradOutputs)

cnn:backward(image,lstm.gradPrevOutput)
~~~

If you are more interested and create your own recurrent units, you can see how they have been implemented [here](https://github.com/Element-Research/rnn/blob/master/LSTM.lua#L143).
