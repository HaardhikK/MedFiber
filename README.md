# VoluFiber: Synthetic 3D Fibre Segmentation Workbench

VoluFiber is a synthetic biomedical imaging project for generating, inspecting, segmenting and evaluating dense 3D fibre volumes. The main goal was to create controllable labelled data for fibre segmentation research, especially for cases where real biomedical volumes are difficult to obtain, expensive to annotate, or not available with voxel-level ground truth.
Since the ground truth is known exactly, it can be used to test segmentation ideas, debug failure modes, and evaluate how well algorithms handle dense 3D fibre structures before moving toward real biomedical data.
This repository contains selected visual outputs videos from the project.

## Demo Videos

### Napari 3D Slicing and Label Inspection

This video demonstrates interactive slicing through the generated 3D volume in Napari. It shows how the grayscale volume, labels, and fibre connections can be inspected layer by layer.

<video src="https://github.com/user-attachments/assets/239c56ef-301c-4f63-ae20-e4847147abab" controls width="100%"></video>

### Blender Fibre Generation

This video demonstrates the Blender-side generation process, where fibre chains are simulated, compressed, and converted into intertwined 3D structures.


<video src="https://github.com/user-attachments/assets/e0eb0f55-8358-434d-ac58-fa0cd6e76ea7" controls width="100%"></video>

## Target

The target was to build a complete synthetic-data workflow for 3D fibre segmentation:

- generate realistic dense fibre structures,
- convert them into volumetric microscopy-like data,
- create exact ground-truth labels for every fibre,
- inspect the volume interactively slice by slice,
- run segmentation algorithms,
- compare predicted segments against the known labels.

The motivation is that fibre segmentation models and algorithms need large amounts of labelled 3D data, but real biomedical annotation is slow and expensive. Synthetic data makes it possible to create difficult controlled examples where the true fibre identity, contact regions, boundaries, and segmentation failures are all known.



## Pipeline Overview

The full pipeline follows this flow:

```text
Blender physics simulation
        ↓
fibre centre-line export
        ↓
3D voxel label generation
        ↓
synthetic grayscale volume rendering
        ↓
research-style crop and affinity target generation
        ↓
segmentation algorithm testing
        ↓
evaluation and visual inspection
```

## Blender Fibre Generation

The first stage creates the fibre geometry in Blender.

Each fibre starts as a chain of connected capsule-like segments. A physics simulation then compresses and perturbs the chains so that they become tangled, dense, and realistic while still remaining non-intersecting. The final simulated centre-lines are exported and later converted into data arrays.

In the Blender output, the visible rod-like structures are the simulation scaffolding, while the smoother tube-like curves represent the final fibre shapes used for visual inspection.

The Blender generation video shows how the fibres are created, compressed, and arranged into an intertwined 3D structure.

## Volumetric Data

After Blender generation, the fibre centre-lines are converted into 3D voxel volumes.

The main volumetric files are stored as `.npy` arrays. This format stores the real numerical 3D data directly, unlike PNG or JPG images, which only store flat pictures.

The important volumetric outputs are:

- `volume.npy`: synthetic grayscale microscopy-like 3D volume.
- `labels.npy`: voxel-level ground truth where each fibre has its own integer label.
- `foreground.npy`: binary mask showing all fibre voxels.
- `boundary_contact.npy`: regions where different fibres touch or come very close.
- `boundary_all.npy`: all fibre/background and fibre/fibre boundaries.
- `affinities_gt.npy`: ground-truth neighbour relationships between voxels.
- `distance_to_background.npy`: distance-based structural cue used for visualization and segmentation.

In simple terms, `volume.npy` is what a segmentation algorithm sees, while `labels.npy` is the truth used to check whether the segmentation was correct.

## Ground-Truth Labels

Every generated fibre is assigned a unique label ID. Background voxels are labelled as `0`, and fibre voxels receive fibre-specific labels such as `1`, `2`, `3`, and so on.

This makes it possible to inspect:

- which voxels belong to each fibre,
- whether two fibres are touching,
- where boundaries occur,
- where a segmentation algorithm incorrectly merges two fibres,
- where a segmentation algorithm incorrectly splits one fibre into multiple pieces.

Because the ground truth is generated automatically from the synthetic geometry, the system knows the correct answer for every voxel.

## Research-Style Crops and Affinity Targets

The pipeline also creates smaller training-style crops from the full 3D volume. These are inspired by dense voxel embedding and affinity-graph segmentation papers in connectomics.

Instead of only storing the complete volume, the system selects dense local regions that contain useful fibre contacts and label diversity. These cropped regions are useful for testing segmentation methods on difficult local areas without always processing the entire volume.

The pipeline also generates affinity targets. An affinity target tells whether neighbouring voxels belong to the same fibre or different fibres. This is important because many 3D segmentation systems do not directly predict object IDs; instead, they predict whether nearby voxels should be connected.

## Segmentation Algorithms

Two segmentation approaches were implemented and tested on the generated data.

The first approach is based on affinity graphs. It estimates whether neighbouring voxels should belong to the same fibre using image intensity, local gradients, boundary cues, and distance information. The resulting graph is then partitioned into predicted fibre segments.

The second approach is based on dense voxel-style embeddings. Instead of only looking at raw intensity, it builds voxel features from intensity, gradients, position, boundary response, and distance maps. Voxels with similar features are grouped together, with additional logic to reduce incorrect merges around dense contact regions.

Both approaches are no-training baselines, meaning they are designed to test the dataset and segmentation logic without requiring a neural network training stage.

## Evaluation

The predicted segmentations are compared against the generated ground-truth labels.

The evaluation checks whether the algorithm correctly recovers the fibre instances and whether it makes common segmentation mistakes such as:

- merging two nearby fibres into one segment,
- splitting one fibre into multiple pieces,
- missing thin fibre regions,
- confusing contact regions,
- over-segmenting dense areas.

The project uses metrics such as Dice score, ARI, IoU, object F1, variation of information, and split/merge estimates to quantify segmentation quality.

## Napari Volume Inspection

Napari is used to interactively inspect the generated 3D data.

The Napari demo video shows the 3D volumetric data being sliced through interactively. This makes it possible to move across depth, inspect individual slices, toggle labels, view fibre connections, remove or isolate label layers, and check where fibres touch or separate.

This is useful because 3D segmentation errors are often hard to understand from a single static image. Slice-based inspection makes it easier to debug whether an algorithm failed because of a real contact, a weak boundary, a missing fibre region, or a merge/split error.



## Current Status

The project currently supports:

- physics-based synthetic fibre generation,
- voxel-level ground-truth label creation,
- grayscale volumetric rendering,
- contact and boundary map generation,
- research-style crop generation,
- affinity target generation,
- two segmentation baselines,
- quantitative evaluation,
- 2D and 3D visual inspection.

The main limitation is that the no-training segmentation baselines are not yet neural-network quality. The hardest failure cases are dense contact regions where fibres are very close together, causing occasional split or merge errors.
