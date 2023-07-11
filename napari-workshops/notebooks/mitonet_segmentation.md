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

Let's read the metadata from our [remote image](https://openorganelle.janelia.org/datasets/jrc_mus-kidney), an electron microscopy image of mouse kidney from the [OpenOrganelle project](https://openorganelle.janelia.org/).

```{code-cell} ipython3
group = zarr.open(zarr.N5FSStore('s3://janelia-cosem-datasets/jrc_mus-kidney/jrc_mus-kidney.n5', anon=True)) # access the root of the n5 container

# Open all resolution levels of the data.
ddata = [
    da.from_zarr(group[f'em/fibsem-uint8/s{i}'])
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

Open the Empanada widget

```{code-cell} ipython3
from empanada_napari._slice_inference import test_widget

widget = test_widget()

viewer.window.add_dock_widget(widget, name="empanada")
```

```{image} resources/empanada_open.png
:alt: check the normalize image box
:width: 80%
:align: center
```

Enter the parameters:

- Set `Model` to `MitoNet_v1_mini`
- Set `Image Downscaling` to 4 (you can play with this later)
- Uncheck `Use quantized model`
- If you have a NVidia GPU, then you can try to select `Use GPU` but that is not necessary

You should get an output like the following:

```{image} resources/empanada_2D_result.png
:alt: check the normalize image box
:width: 80%
:align: center
```
