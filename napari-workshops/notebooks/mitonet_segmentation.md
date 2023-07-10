---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.7
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# notebook: mitonet segmentation

The dataset we will use is https://openorganelle.janelia.org/datasets/jrc_mus-kidney.

We will need to install this empanada-napari plugin https://empanada.readthedocs.io/en/latest/empanada-napari.html.

## Setup

```{code-cell} ipython3
:tags: [remove-output]

# this cell is required to run these notebooks on Binder. Make sure that you also have a desktop tab open.
import os
if 'BINDER_SERVICE_HOST' in os.environ:
    os.environ['DISPLAY'] = ':1.0'
```

We start by importing `napari`, our `nbscreenshot` utility and instantiating an empty viewer.

```{code-cell} ipython3
import zarr
import dask.array as da

import napari
from napari.utils import nbscreenshot

# Create an empty viewer
viewer = napari.Viewer()
```

Let's read the metadata from our remote zarr image.

```{code-cell} ipython3
group = zarr.open(zarr.N5FSStore('s3://janelia-cosem-datasets/jrc_mus-kidney/jrc_mus-kidney.n5', anon=True)) # access the root of the n5 container
    
zdata = group['em/fibsem-uint8/s2'] # s0 is the the full-resolution data, use s2
    
# create a 2-level resolution image using s2 and s3
ddata = [
    da.from_zarr(group[f'em/fibsem-uint8/s{i}'], chunks=zdata.chunks)
    for i in range(0, 4)
]

[a.shape for a in ddata]
```

Our image is very large and performing a segmentation on the full data would take a long time.

We will crop a 2D slice of our image and show how to perform a segmentation on that region.

```{code-cell} ipython3
cropped_img = ddata[0][3000:3400, 800, 5000:6000]

viewer.add_image(cropped_img)
```

```{code-cell} ipython3
from empanada_napari._slice_inference import test_widget

widget = test_widget()

viewer.window.add_dock_widget(widget, name="empanada")
```
