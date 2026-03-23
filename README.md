# Gz Bazel Dev

This repo sets up a [Bazel](https://bazel.build/) development workspace for [Gazebo](gazebosim.org) packages.
This repo is primarily intended for developers who are familiar with the Bazel build system, and would like to contribute to the Gazebo project (see [contribution guidelines](https://gazebosim.org/docs/latest/contributing/)).

Each Gazebo package repo supports development with Bazel natively (e.g. [Gz Sim](https://github.com/gazebosim/gz-sim/blob/main/BUILD.bazel)).
However, building each package standalone in a separate Bazel workspace can be cumbersome, especially for cross-package development.

Instead of building each Gazebo package directly, you can use this pre-configured "client" repo to build all Gazebo packages in a single bazel workspace.
Each Gazebo package is treated as an [external repo](https://bazel.build/external/overview) for builds within the shared workspace.
This repo uses `local_path_override` directives in [MODULE.bazel](MODULE.bazel) to build each Gazebo package from local source (instead of building them from a bazel registry).
Since the bazel output base is shared between Gazebo packages, builds are typically more cache-efficient and faster.

If you want to use CMake (and colcon) to develop Gazebo packages instead, please see the source installation instructions at [gazebosim.org](https://gazebosim.org/docs/latest/install).

## Supported platforms

Note that Bazel development for Gazebo packages is currently supported only on Ubuntu and MacOS.

## Installation

### Install `vcstool`

Follow the platform-specific instructions for source install at [gazebosim.org](https://gazebosim.org/docs/latest/install) to install `vcstool`.
Note that you only need to install `vcstool`, *not* `colcon`.

The following instructions are for installing `vcstool` in a python virtualenv, at `$HOME/vcs_installation`, on Ubuntu:

```sh
sudo apt update && sudo apt install -y python3-pip python3-venv lsb-release curl git
python3 -m venv $HOME/vcs_installation
. $HOME/vcs_installation/bin/activate
pip3 install vcstool
```

### Install Bazel

We recommend installing `bazelisk`, which wraps `bazel` and removes the need to download version-specific bazel binaries.

Follow the [platform-specific instructions](https://github.com/bazelbuild/bazelisk?tab=readme-ov-file#installation) on their Github repo.

The following instructions are for Linux with AMD64 (i.e. x86-64) architecture, and installs `bazelisk` at `/usr/local/bin/bazel` (recommended):

```sh
sudo curl -L https://github.com/bazelbuild/bazelisk/releases/download/v1.28.1/bazelisk-linux-amd64 -o /usr/local/bin/bazel
sudo chmod +x /usr/local/bin/bazel
```

### Set up bazel workspace

Create a directory for the bazel workspace and fetch the Gazebo package sources for the Rotary release.
If you want to build a [different release](https://gazebosim.org/docs/latest/releases/), adjust the `collection-*.yaml` file path below and re-run `vcs import`.
Note that Gazebo supports Bazel (bzlmod) in Ionic and later releases.

```sh
mkdir -p $HOME/gz_bazel_workspace && cd $HOME/gz_bazel_workspace
curl -O https://raw.githubusercontent.com/gazebo-tooling/gazebodistro/master/collection-rotary.yaml
vcs import < collection-rotary.yaml
deactivate
```

Clone the `gz-bazel-dev` repo alongside the other Gazebo package sources.

```sh
git clone https://github.com/gazebo-tooling/gz-bazel-dev.git
cd gz-bazel-dev
```

### Verify installation

Verify by building `gz-sim`.
Note the use of `@` here to refer to the "external" gz-sim repo. This in turn points to `gz-sim` installed above from source.

```sh
bazel build @gz-sim
```

Non-canonical targets can be referenced by prepending `@<module name>//`. For example:

```sh
bazel build @gz-sim//:gz-sim-physics-system-static
```

Sub-packages can be referenced by their path under the external module `@<module name>//<sub-package path>`. For example:

```sh
bazel build @gz-common//graphics
```

Unit-tests can be run with `bazel test`. For example:

```sh
bazel test @gz-rendering//:all
```

### Clean up `vcstool`

You can delete the virtual env at `$HOME/vcs_installation` if you want, since it is not needed for development with bazel.

```sh
rm -r $HOME/vcs_installation
```

## Limitations

### Failing tests
Some test targets may fail **falsely** when built the from the bazel workspace due to runtime repo differences (`_main` vs `external`).
You can still run these failing test targets directly from the local package source directory, but this will build and run the test target in a separate bazel workspace for that package.
For example

```sh
cd ../gz-common
bazel test graphics:Image_TEST
```

### Missing Bazel support

The following Gazebo packages and features are currently not fully supported or implemented in Bazel:

* `gz-gui`: The `gz-gui` package is not currently supported in Bazel. Consequently, the Gazebo Sim GUI cannot be built or deployed using Bazel. It must be deployed from a binary install or a source install using CMake.
* DEM Heightmaps: Support for DEM heightmaps is not yet available in the Bazel build due to third-party dependency limitations (e.g., gdal not being in BCR).
* Zenoh Backend: The Zenoh backend for Gz Transport is not yet supported.
* `gz-tools`: These packages are not yet supported in Bazel.
* `gz-rendering` only supports the `ogre2` render engine, and only with OpenGL+EGL backend in Bazel.
* Not all python bindings are available yet in Bazel.
