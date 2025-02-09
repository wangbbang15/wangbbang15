(spliceai_env) rare1@DESKTOP-Q8E0MN3:~/tool/SpliceAI$ spliceai --help
Traceback (most recent call last):
  File "/home/rare1/.local/bin/spliceai", line 5, in <module>
    from spliceai.__main__ import main
  File "/home/rare1/.local/lib/python3.8/site-packages/spliceai/__main__.py", line 5, in <module>
    from spliceai.utils import Annotator, get_delta_scores
  File "/home/rare1/.local/lib/python3.8/site-packages/spliceai/utils.py", line 5, in <module>
    from keras.models import load_model
  File "/home/rare1/.local/lib/python3.8/site-packages/keras/__init__.py", line 3, in <module>
    from keras import __internal__
  File "/home/rare1/.local/lib/python3.8/site-packages/keras/__internal__/__init__.py", line 3, in <module>
    from keras.__internal__ import backend
  File "/home/rare1/.local/lib/python3.8/site-packages/keras/__internal__/backend/__init__.py", line 3, in <module>
    from keras.src.backend import _initialize_variables as initialize_variables
  File "/home/rare1/.local/lib/python3.8/site-packages/keras/src/__init__.py", line 21, in <module>
    from keras.src import models
  File "/home/rare1/.local/lib/python3.8/site-packages/keras/src/models/__init__.py", line 18, in <module>
    from keras.src.engine.functional import Functional
  File "/home/rare1/.local/lib/python3.8/site-packages/keras/src/engine/functional.py", line 23, in <module>
    import tensorflow.compat.v2 as tf
  File "/home/rare1/.local/lib/python3.8/site-packages/tensorflow/__init__.py", line 41, in <module>
    from tensorflow.python.tools import module_util as _module_util
  File "/home/rare1/.local/lib/python3.8/site-packages/tensorflow/python/__init__.py", line 53, in <module>
    from tensorflow.core.framework.graph_pb2 import *
  File "/home/rare1/.local/lib/python3.8/site-packages/tensorflow/core/framework/graph_pb2.py", line 16, in <module>
    from tensorflow.core.framework import function_pb2 as tensorflow_dot_core_dot_framework_dot_function__pb2
  File "/home/rare1/.local/lib/python3.8/site-packages/tensorflow/core/framework/function_pb2.py", line 16, in <module>
    from tensorflow.core.framework import attr_value_pb2 as tensorflow_dot_core_dot_framework_dot_attr__value__pb2
  File "/home/rare1/.local/lib/python3.8/site-packages/tensorflow/core/framework/attr_value_pb2.py", line 16, in <module>
    from tensorflow.core.framework import tensor_pb2 as tensorflow_dot_core_dot_framework_dot_tensor__pb2
  File "/home/rare1/.local/lib/python3.8/site-packages/tensorflow/core/framework/tensor_pb2.py", line 16, in <module>
    from tensorflow.core.framework import resource_handle_pb2 as tensorflow_dot_core_dot_framework_dot_resource__handle__pb2
  File "/home/rare1/.local/lib/python3.8/site-packages/tensorflow/core/framework/resource_handle_pb2.py", line 16, in <module>
    from tensorflow.core.framework import tensor_shape_pb2 as tensorflow_dot_core_dot_framework_dot_tensor__shape__pb2
  File "/home/rare1/.local/lib/python3.8/site-packages/tensorflow/core/framework/tensor_shape_pb2.py", line 36, in <module>
    _descriptor.FieldDescriptor(
  File "/home/rare1/.local/lib/python3.8/site-packages/google/protobuf/descriptor.py", line 561, in __new__
    _message.Message._CheckCalledFromGeneratedFile()
TypeError: Descriptors cannot not be created directly.
If this call came from a _pb2.py file, your generated code is out of date and must be regenerated with protoc >= 3.19.0.
If you cannot immediately regenerate your protos, some other possible workarounds are:
 1. Downgrade the protobuf package to 3.20.x or lower.
 2. Set PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python (but this will use pure-Python parsing and will be much slower).

More information: https://developers.google.com/protocol-buffers/docs/news/2022-05-06#python-updates
(spliceai_env) rare1@DESKTOP-Q8E0MN3:~/tool/SpliceAI$
