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

# Introduction to animation with Napari

+++

Creating a short animation is a great way to introduce your data during a presentation and provide insight into the image analysis. It is especially valuable while working with 3D data that can otherwise be presented only in a form of 2D screenshots.

+++

To grasp the process of creating an animation, it is essential to understand the concept of keyframes (https://en.wikipedia.org/wiki/Key_frame ). Keyframes can be likened to anchor points that steer an animation. Thus, by defining the keyframes, you determine what will be portrayed. The transitions between keyframes are smooth and are automatically filled in for you. Keyframes in a napari animation can capture such events as change of a frame, zoom, camera angle (3D projections) or even brightness of an image. The speed of the animation is controlled by the number of in-between frames (‘Steps’) inserted between the key frames.

+++

Napari provides a simple way to create animations in both manual (interactive) and programmatic (scripting) way through the [Animation Plugin](https://github.com/napari/napari-animation).

In this tutorial we will create a simple animation of [FIB-SEM data](https://openorganelle.janelia.org/datasets/jrc_mus-kidney).

+++

## Setup

```{code-cell} ipython3
:tags: [remove-output]

# this cell is required to run these notebooks on Binder. Make sure that you also have a desktop tab open.
import os
if 'BINDER_SERVICE_HOST' in os.environ:
    os.environ['DISPLAY'] = ':1.0'
```

```{code-cell} ipython3
# Instal animation plugin
# This step has to be performed only once in a given environment.
# You can remove this cell or use '#' to comment the code out after a successful installation.

!pip install napari-animation

# No problem if you execute it again though, Python will just tell you 'Requirement already satisfied'.
```

```{code-cell} ipython3
import os
import zarr
import dask.array as da

import napari

from napari_animation import Animation
from napari_animation.easing import Easing
```

## Reading in data

+++

Get data from OpenOrganelle.

```{code-cell} ipython3
group = zarr.open(zarr.N5FSStore('s3://janelia-cosem-datasets/jrc_mus-kidney/jrc_mus-kidney.n5', anon=True)) # access the root of the n5 container

# Get access to all resolution levels of the data.
ddata = [
    da.from_zarr(group[f'em/fibsem-uint8/s{i}'])
    for i in range(0, 4)
]
```

```{code-cell} ipython3
ddata[3].shape
```

Get an example cube of the data and load it into memory.

```{code-cell} ipython3
cropped_img = ddata[3][50:150:2,400:500,1000:1100].compute()
cropped_img.shape
```

## Visualize in napari

```{code-cell} ipython3
viewer = napari.Viewer()
viewer.add_image(cropped_img)
```

## Use animation plugin interactively

+++

- From the upper menu choose Plugins - wizard (napari-animation)
- With the bottom slides choose first slice \#0 
- Click 'Capture' to set it as a first keyframe
- Choose number of Steps to 30
- Move slider to slice \#40
- Capture the keyframe
- Move slider to slice \#49
- Choose number of Steps to 60
- Capture the keyframe
- Save your first animation

+++

## Use script to record an animation

```{code-cell} ipython3
# define pathway to save your animation
save_path = 'my_animation_from_script.mp4'
```

```{note}
If you don\'t specify a full pathway but only a name of the file it will be saved in your current working directory (cwd).
If you don\'t know where it is use os.getcwd()
```

```{code-cell} ipython3
animation = Animation(viewer)

viewer.reset_view()
viewer.dims.current_step = (0, 0, 0)
animation.capture_keyframe()

viewer.dims.current_step = (40, 0, 0)
animation.capture_keyframe(steps=40)

viewer.dims.current_step = (49, 0, 0)
animation.capture_keyframe(steps=70)

animation.animate(save_path, canvas_only=True, fps=24)
```

```{code-cell} ipython3

```
