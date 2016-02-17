---
layout:     post
date:       2016-01-29 19:00:00
author:     "Jakub Kvita"
comments:   true
title:      "Efficient serialization of Torch models"
subtitle:   "How to save smaller Torch models by removing unwanted fields."
---

Basic Torch provides [methods](https://github.com/torch/torch7/blob/master/doc/serialization.md) for writing model to a file and reloading it together with serialization/deserialization methods.

~~~lua
torch.save(filename, object [, format, referenced])
torch.load(filename [, format, referenced])

torch.serialize(object)
torch.deserialize(str)
~~~

These methods are as simple as it gets and can process any Torch object, which is excellent.
However [nn](https://github.com/torch/nn) models contains several fields, which are used during training, but are not necessary while saving. This causes the files to be unnecessary large. What can we do about that?

Deleting unwanted data in [nn](https://github.com/torch/nn) modules
-----------------------------------------
Modules from nn package are relatively simple and can be processed manually. Usually they contain three types of fields:

* `weight` and `bias` -- fields we need.
* `gradWeight` and `gradBias` -- fields, which can be deleted after training.
* `output` and `gradInput`  -- fields, which are usually not required to keep.

Unwanted fields of the module can be reduced by creating Tensor of same type with no dimensions:

~~~lua
data = torch.Tensor():typeAs(data)
~~~

If your models are small, you can do it by hand for every module, but this is usually not an issue for models that small.

Larger models can be processed by function, for example similar to the recursive one [here](https://github.com/torch/nn/issues/112), or by the [net-toolkit](https://github.com/Atcold/net-toolkit) package.

Modules from [rnn](https://github.com/Element-Research/rnn) package
------------------------
Reducing recurrent modules from [rnn](https://github.com/Element-Research/rnn) can be quite tricky, as they contain several different fields and, most importantly, shared clones. Use [`Serial`](https://github.com/Element-Research/dpnn#serial) decorator from [dpnn](https://github.com/Element-Research/dpnn) package to deal with it easily.

First, model has to be decorated and type of serialization (`light`, `medium`, `heavy`) need to be specified. These options align nicely with three types of fields from before. `heavy`, which is default option in Torch, saves everything, `medium` is recommended to use during training and `light` can be used for models in production, as it keeps only weights and biases.

~~~lua
dmodule = nn.Serial(module, [tensortype])
dmodule:lightSerial() __OR__ dmodule:mediumSerial() __OR__ dmodule:heavySerial()
~~~

Regular Torch `save` and `load` methods can be used to process the decorated model. No tweaks are required in this part.
