# QSIPrep Docker Build Instructions

This document provides step-by-step instructions for building the QSIPrep Docker image.

## Prerequisites

Before you begin, ensure you have the following installed:

- **Docker**: Version 20.10+ with BuildKit support
  - Check: `docker --version`
  - Enable BuildKit: It should be enabled by default in Docker 20.10+, but you can explicitly enable it with:
    ```bash
    export DOCKER_BUILDKIT=1
    ```
- **Git**: For cloning the repository (if needed)
- **Sufficient disk space**: At least 50-100 GB free space for the build and resulting image
- **Internet connection**: Required to pull component images from Docker Hub

## Quick Start

The fastest way to build the image:

```bash
cd /path/to/qsiprep_build
source setup_build.sh
do_build build_fsl
```

This will create a Docker image tagged as `pennlinc/qsiprep_build:latest` with all tools included.

## Build Options

### Option 1: Full Build (With FSL) - Recommended

Build the complete QSIPrep image with all neuroimaging tools including FSL:

```bash
cd /path/to/qsiprep_build
source setup_build.sh
do_build build_fsl
```

**Resulting image**: `pennlinc/qsiprep_build:latest`

**Includes**: FSL, FreeSurfer, ANTs, MRTrix3, 3Tissue, DSI Studio, AFNI, TORTOISE, SynB0, and Micromamba

### Option 2: Lightweight Build (Without FSL) - Faster

Build a lighter version without FSL for faster builds and smaller image size:

```bash
cd /path/to/qsiprep_build
source setup_build.sh
do_build no_fsl
```

**Resulting image**: `pennlinc/qsiprep_build:latest-nofsl`

## Manual Build (Advanced)

If you prefer to build without the setup script, you can run Docker build directly:

```bash
cd /path/to/qsiprep_build

DOCKER_BUILDKIT=1 \
docker build \
  -t pennlinc/qsiprep_build:latest \
  --build-arg TAG_FSL=26.1.0 \
  --build-arg TAG_FREESURFER=26.1.0 \
  --build-arg TAG_ANTS=26.1.2 \
  --build-arg TAG_MRTRIX3=26.1.0 \
  --build-arg TAG_3TISSUE=26.1.0 \
  --build-arg TAG_DSISTUDIO=26.1.2 \
  --build-arg TAG_DSISTUDIOCHEN=26.1.2 \
  --build-arg TAG_MICROMAMBA=26.1.7 \
  --build-arg TAG_AFNI=AFNI_25.2.09 \
  --build-arg TAG_TORTOISE=26.1.3 \
  --build-arg TAG_TORTOISECUDA=26.1.3 \
  --build-arg TAG_SYNB0=26.1.2 \
  --build-arg FSL_BUILD=build_fsl \
  .
```

### Customizing Build Arguments

You can modify the component versions by changing the build arguments:

| Argument | Default | Purpose |
|----------|---------|---------|
| `TAG_FSL` | 26.1.0 | FSL version |
| `TAG_FREESURFER` | 26.1.0 | FreeSurfer version |
| `TAG_ANTS` | 26.1.2 | ANTs version |
| `TAG_MRTRIX3` | 26.1.0 | MRTrix3 version |
| `TAG_3TISSUE` | 26.1.0 | 3Tissue version |
| `TAG_DSISTUDIO` | 26.1.2 | DSI Studio version |
| `TAG_DSISTUDIOCHEN` | 26.1.2 | DSI Studio Chen version |
| `TAG_MICROMAMBA` | 26.1.7 | Micromamba version |
| `TAG_AFNI` | AFNI_25.2.09 | AFNI version |
| `TAG_TORTOISE` | 26.1.3 | TORTOISE version |
| `TAG_TORTOISECUDA` | 26.1.3 | TORTOISE CUDA version |
| `TAG_SYNB0` | 26.1.2 | SynB0 version |
| `FSL_BUILD` | build_fsl | Set to `build_fsl` or `no_fsl` |

## Verifying the Build

After the build completes successfully, verify the image was created:

```bash
# List Docker images
docker images | grep qsiprep_build

# You should see output like:
# pennlinc/qsiprep_build   latest    abc123...  2 hours ago   45GB
```

## Build Architecture

The Dockerfile uses multi-stage builds to combine multiple component images:

1. **FSL Image** (`pennlinc/qsiprep-fsl:26.1.0`)
2. **AFNI Image** (`afni/afni_make_build:AFNI_25.2.09`)
3. **FreeSurfer Image** (`pennlinc/qsiprep-freesurfer:26.1.0`)
4. **ANTs Image** (`pennlinc/qsiprep-ants:26.1.2`)
5. **MRTrix3 Image** (`pennlinc/qsiprep-mrtrix3:26.1.0`)
6. **3Tissue Image** (`pennlinc/qsiprep-3tissue:26.1.0`)
7. **DSI Studio Image** (`pennlinc/qsiprep-dsistudio:26.1.2`)
8. **DSI Studio Chen Image** (`pennlinc/qsiprep-dsistudio-chen:26.1.2`)
9. **Micromamba Image** (`pennlinc/qsiprep-micromamba:26.1.7`)
10. **TORTOISE Image** (`pennlinc/qsiprep-drbuddi:26.1.3`)
11. **TORTOISE CUDA Image** (`pennlinc/qsiprep-drbuddicuda:26.1.3`)
12. **SynB0 Image** (`pennlinc/qsiprep-synb0:26.1.2`)
13. **Atlas Pack Image** (`pennlinc/atlaspack:0.1.0`)

All components are merged into a single final image based on `nvidia/cuda:12.2.2-runtime-ubuntu22.04`.

## What's Included in the Image

- **FSL**: Tools for diffusion MRI processing (eddy, topup, etc.)
- **FreeSurfer**: Anatomical image segmentation and registration
- **ANTs**: Advanced normalization tools for image registration
- **MRTrix3**: Advanced diffusion MRI processing tools
- **3Tissue**: 3-tissue compartment modeling
- **DSI Studio**: Diffusion imaging analysis and visualization
- **AFNI**: Additional neuroimaging analysis tools
- **TORTOISE**: Diffusion MRI registration and motion correction
- **SynB0**: Synthetic B0 field maps for distortion correction
- **Micromamba**: Lightweight conda environment manager
- **Atlas Pack**: Reference atlases for brain imaging
- **NVIDIA CUDA**: GPU support for accelerated processing