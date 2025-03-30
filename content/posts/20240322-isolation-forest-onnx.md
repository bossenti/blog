+++
title = "Converting a Sklearn Isolation Forrest to ONNX Model"
date = "2024-03-22"

toc = true

[taxonomies]
tags=[
    "onnx",
    "python",
    "scikit-learn",
]
categories = ["Tech"]
+++

Today I was faced with a really tricky error when trying to convert my Isolation Forrest model to an ONNX model. I hope
this article can save you some time if you are faced with the same situation.

# The Scenario

I had a really simple example of fitting an Isolation Forrest and then converting it to an ONNX model as you can see
below.

```python
from skl2onnx import to_onnx
from sklearn.ensemble import IsolationForest

model = IsolationForest()
model.fit(X)

model_onnx = to_onnx(
      model,
      initial_types=[('input', FloatTensorType([None, X.shape[1]]))]
)
```

This led me to the following error:

{% alert() %}
RuntimeError: The model is using version 4 of domain 'ai.onnx.ml' not supported
yet by this library. You need to specify target_opset={'ai.onnx.ml': 3}
{% end %}


# The solution approach

First I checked that I had the latest release of `skl2onnx`, which I did (`1.16.0`). Then I just followed the
recommendation
in the log message:

```python
model_onnx = to_onnx(
   model, 
   initial_types=[('input', FloatTensorType([None, temp_scaled_np.shape[1]]))], 
   target_opset={'ai.onnx.ml': 3}
)
```

Unfortunately, this resulted in the following error:

{% alert() %}
RuntimeError: op_version must be specified.
{% end %}


I tried to fix this by extending `target_opset` to `{'ai.onnx.ml': 3, 'ai.onnx': 15}` (according to [ONNX versioning
overview](https://github.com/onnx/onnx/blob/main/docs/Versioning.md#released-versions)). However, this still resulted in the same error.

After some research I found out that setting the `opset` to `'': 15` is the key to success here. So the following
statement allowed me to create the ONNX model.

```python
model_onnx = to_onnx(
   model,
   initial_types=[('input', FloatTensorType([None, temp_scaled_np.shape[1]]))], 
   target_opset={'ai.onnx.ml': 3, 'ai.onnx': 15, '':15}
)
```