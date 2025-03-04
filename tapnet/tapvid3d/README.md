# Tracking Any Point in 3D (TAPVid-3D)

TAPVid-3D is a dataset and benchmark for evaluating the task of long-range
Tracking Any Point in 3D (TAP-3D).

The benchmark features 4,000+ real-world videos, along with their metric 3D
position point trajectories. The dataset is contains three different video
sources, and spans a variety of object types, motion patterns, and indoor and
outdoor environments. This repository folder contains the code to download and
generate these annotations and dataset samples to view.

**Note that in order to use the dataset, you must accept the licenses of the
constituent original video data sources that you use!** In particular, you must
adhere to the terms of service outlined in:

1.  [Aria Digital Twin](https://www.projectaria.com/datasets/adt/license/)
2.  [Waymo Open Dataset](https://waymo.com/open/terms/)
3.  [Panoptic Studio](http://domedb.perception.cs.cmu.edu/)

To measure performance on the TAP-3D task, we formulated metrics that extend the
Jaccard-based metric used in 2D TAP to handle the complexities of ambiguous
depth scales across models, occlusions, and multi-track spatio-temporal
smoothness. Our implementation of these metrics (3D-AJ, APD, OA) can be found in
`tapvid3d_metrics.py`.

For more details, including the performance of multiple baseline 3D point
tracking models on the benchmark, please see the paper:
[TAPVid-3D:A Benchmark for Tracking Any Point in 3D](http://arxiv.org).

### Getting Started: Installing

Install the helpers for evaluating models on TAPVid-3D by first installing the
root Tracking Any Point repository:

`pip install "git+https://github.com/google-deepmind/tapnet.git"`

or locally as:

`pip install -e .`

Then install the lightweight dependencies for the core data and eval helpers:

`pip install -r https://raw.githubusercontent.com/google-deepmind/tapnet/main/tapnet/tapvid3d/requirements_eval.txt`

For generating the dataset, you'll need to also install the extra requirements:

`pip install -r https://raw.githubusercontent.com/google-deepmind/tapnet/main/tapnet/tapvid3d/requirements_generation.txt`

Replace `/cpu` with `/cu118` or `/cu121` in the first line of
`requirements_generation.txt` if you want to use CUDA for running the semantic
segmentation model.


### How to Download and Generate the Dataset

Scripts to download and generate the annotations of each of the datasets can be
found in the `annotation_generation` subdirectory, and can be run as:

```
python3 -m tapnet.tapvid3d.annotation_generation.generate_adt --help
python3 -m tapnet.tapvid3d.annotation_generation.generate_pstudio --help
python3 -m tapnet.tapvid3d.annotation_generation.generate_drivetrack --help
```

Because of license restrictions in distribution of the underlying source videos
for Aria Digital Twin, you will need to accept their licence terms and download
the ADT dataset following the instructions on their
[website](https://www.projectaria.com/datasets/adt/#download-dataset),
before you can run our generation script.

Similar scripts for the Waymo Open Dataset and Panoptic Studio dataset are also
provided.

To run all generation scripts, follow the instructions and run the commands in
`generate_all.sh`. This will generate all the `*.npz` files, and place them into
a new folder, `tapvid3d_dataset`. To generate only the `minival` split,
replace `--split=all` in this script with `--split=minival`.

To test things are working before full generation, you can run `./run_all.sh
--debug`. This runs generates exactly one `*.npz`/video annotation per data
source.

### Data format

Once the benchmark files are fully generated, you will have roughly 4,500
`*.npz` files, each one with exactly one dataset video+annotation. Each `*.npz`
file, contains:

*   `images_jpeg_bytes`: tensor of shape [`# of frames`, `height`, `width`, 3],
    each frame stored as JPEG bytes that must be decoded

*   `intrinsics`: (fx, fy, cx, cy) camera intrinsics of the video

*   `tracks_xyz`: tensor of shape (`# of frames`, `# of point tracks`, 3),
    representing the 3D point trajectories and the last dimension is the `(x, y,
    z)` point position in meters.

*   `visibility`: tensor of shape (`# of frames`, `# of point tracks`),
    representing the visibility of each point along its trajectory

*   `queries_xyt`: tensor of shape (`# of point tracks`, 3), representing the
    query point used in the benchmark as the initial given point to track. The
    last dimension is given in `(x, y, t)`, where `x,y` is the pixel location of
    the query point and `t` is the query frame.

### Getting Started: Evaluating Your Own 3D Tracking Model

To get started using this dataset to benchmark your own 3D tracking model, check
out `evaluate_model.py`. This script reads the ground truth track prediction,
and predicted tracks and computes the TAPVid-3D evaluation metrics for those
predictions. An example of running this is provided in `run_evaluate_model.sh`,
which computes metrics for our precomputed outputs from
[SpatialTracker](https://henry123-boy.github.io/SpaTracker/) on the `minival`
split, for only the `DriveTrack` subset. `evaluate_model.py`
takes as input a folder with subdirectories (corresponding to up to 3 data
sources [among PStudio/DriveTrack/ADT]). Each
subdirectory should contain a list of `*.npz` files
(one per TAPVid-3D video, each containing
`tracks_XYZ` and `visibility` in the format above).
Running this will return all TAP-3D metrics
described in the paper (these are implemented in `tapvid3d_metrics.py`).

### Visualizing Samples in Colab

You can view samples of the dataset, using a public Colab demo:
<a target="_blank" href=""><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="TAPVid-3D Colab Visualization"/></a>.
You can also view samples you've generated with the `annotation_generation`
scripts with the local Jupyter notebook
`load_and_visualize_tapvid3d_samples.ipynb`.

## Citing this Work

If you find this work useful, consider citing the manuscript:

```
@article{doersch2022tap,
  title={TAPVid-3D:A Benchmark for Tracking Any Point in 3D},
  author={Skanda Koppula, Ignacio Rocco, Yi Yang, Joe Heyward, João Carreira, Andrew Zisserman, Gabriel Brostow, Carl Doersch},
  journal={},
  volume={},
  pages={},
  year={2024}
}
```

## License and Disclaimer

Copyright 2024 Google LLC

All software here is licensed under the Apache License, Version 2.0 (Apache
2.0); you may not use this file except in compliance with the Apache 2.0
license. You may obtain a copy of the Apache 2.0 license at:

https://www.apache.org/licenses/LICENSE-2.0

All other materials, excepting the source videos and artifacts used to generate
annotations, are licensed under the Creative Commons Attribution 4.0
International License (CC-BY). You may obtain a copy of the CC-BY license at:
https://creativecommons.org/licenses/by/4.0/legalcode

The Aria Digital Twin, Waymo Open Dataset, and Panoptic Studio datasets all
have individual usage terms and licenses that must be adhered to when using any
software or materials from here.

Unless required by applicable law or agreed to in writing, all software and
materials distributed here under the Apache 2.0 or CC-BY licenses are
distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
either express or implied. See the licenses for the specific language governing
permissions and limitations under those licenses.

This is not an official Google product.
