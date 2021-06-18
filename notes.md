# ITK hacking
## Support itk.Image use in Dask Array `map_blocks`
We need to add pickle support to `itk.Image`
Github issue: https://github.com/InsightSoftwareConsortium/ITK/issues/1091

Matt McCormick suggests: 
The process may be a great example basis for a Dask Blog post 
on how to add required pickle support for data structures from 
native Python packages.

John Kirkham says there's also useful discussion about pickle & serialization here:
Related issue: https://github.com/InsightSoftwareConsortium/ITK/issues/1948

## Process
### Create conda env
`conda create -n itk-dev python=3.8 pip ipython jupyter pytest`

### Activate conda env
`conda activate itk-dev`

### Hack this file
`code ~/anaconda3/envs/itk-dev/lib/python3.8/site-packages/itk/itkImagePython.py`

// Here's what to do


// Need to get this test to pass
```python
import itk
import pickle
import numpy as np

# Create an itk.Image
array = np.random.randint(0, 256, (8, 12)).astype(np.uint8)
image = itk.image_from_array(array)
image.SetSpacing([1.0, 2.0])
image.SetOrigin([11.0, 4.0])
theta = np.radians(30)
cosine = np.cos(theta)
sine = np.sin(theta)
rotation = np.array(((cosine, -sine), (sine, cosine)))
image.SetDirection(rotation)

# Verify serialization works with a np.ndarray
serialized = pickle.dumps(array)
deserialized = pickle.loads(serialized)
assert np.array_equal(array, deserialized)

# How to check for image consistency
image_copy = itk.image_duplicator(image)
compared = itk.comparison_image_filter(image, image_copy)
assert np.sum(compared) == 0

# We need to add support for this
serialized = pickle.dumps(image)
deserialized = pickle.loads(serialized)
compared = itk.comparison_image_filter(image, deserialized)
assert np.sum(compared) == 0
```
