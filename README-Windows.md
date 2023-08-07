# OpenMM HIP Plugin for Windows

This plugin adds HIP platform that allows to run [OpenMM](https://openmm.org) on CDNA and RDNA
AMD GPUs on [AMD ROCm™ open software platform](https://rocmdocs.amd.com).

## Install OpenMM with Miniconda

Follow the OpenMM [user guide](http://docs.openmm.org/latest/userguide/application/01_getting_started.html#installing-openmm) 
to install OpenMM in Miniconda.

## Install AMD HIP SDK

Download the [HIP SDK for Windows](https://www.amd.com/en/developer/rocm-hub/hip-sdk.html). During the installation, change
the installation directory to be within `C:\AMD` instead of `C:\Program Files\AMD` so the file path does not contain spaces.

The options for Ray Tracing and Radeon Pro drivers can be unchecked.

## Install Visual Studio

[Visual Studio Community Edition](https://visualstudio.microsoft.com/vs/community/) is sufficient with the 
"Desktop development with C++" workload enabled.

## Building OpenMM-HIP

This project uses [CMake](http://www.cmake.org) for its build system.

The plugin requires source code of OpenMM, it can be downloaded as an archive
[here](https://github.com/openmm/openmm/releases) or as a Git repository:

```sh
git clone https://github.com/openmm/openmm.git -b 8.0.0
```

Clone this repository or download the source code as an archive
[here](https://github.com/bdenhollander/openmm-hip/archive/refs/heads/windows-compatibility.zip)

```sh
git clone https://github.com/bdenhollander/openmm-hip.git -b windows-compatibility
```

To build the plugin, follow these steps:

1. Create a directory in which to build the plugin.

2. Run the CMake GUI, specifying your new directory as the build directory and the top
level directory of this project as the source directory.

3. Press "Configure".

4. Set `OPENMM_DIR` to point to the `Library` directory within the Miniconda environment.
 This is needed to locate the OpenMM header files and libraries.

5. Set `OPENMM_SOURCE_DIR` to point to the directory where OpenMM source code is located.

6. Set `HIP_DIR` and `hip_DIR` to `C:\AMD\ROCm\5.5\lib\cmake\hip`.

7. Set `HIPFFT_DIR` to `C:\AMD\ROCm\5.5\lib\cmake\hipfft`.

8. Set `HIPRTC_DIR` to `C:\AMD\ROCm\5.5\lib\cmake\hiprtc`.

9. Confirm that `OPENMM_BUILD_SHARED_LIB` is checked.

10. Press "Configure" again and check "Advanced" if necessary to make additional variables appear, then press "Generate".

11. Open the generated solution in Visual Studio.

12. Switch to Release mode and build `OpenMMHip` and `OpenMMHipCompiler`. Make note of the directories the files were written to.

13. If `pthread.lib` is the only file not found during linking, install it into Miniconda `conda install -c conda-forge pthreads-win32`.

## Adding HIP to OpenMM

1. Create a directory to use for compiled HIP kernels. Add an Environment variable named `OPENMM_CACHE_DIR` and set the value
to be the file path to that directory.

2. Open the directory that contains the Miniconda environment and browse to `Library\lib\plugins`

3. Copy the following files from `C:\AMD\ROCm\5.5\bin` to `Library\lib\plugins`
   - `amd_comgr0505.dll`
   - `hipfft.dll`
   - `hiprtc0505.dll`
   - `rocfft.dll`

4. Copy the following files from Visual Studio build folders to `Library\lib\plugins`
   - `platforms\hip\Release\OpenMMHIP.dll`
   - `platforms\hip\Release\OpenMMHIP.lib`
   - `plugins\hipcompiler\Release\OpenMMHipCompiler.dll`
   - `plugins\hipcompiler\Release\OpenMMHipCompiler.lib`

5. Verify your installation (HIP must be one of available platforms):

```sh
python -m openmm.testInstallation
```

If there is no HIP among available platforms check why the HIP platform fails to load:

```sh
python -c "import openmm as mm; print('---Loaded---', *mm.pluginLoadedLibNames, '---Failed---', *mm.Platform.getPluginLoadFailures(), sep='\n')"
```

## Benchmark

Launch Miniconda and switch to the directory where the Conda environment is located. Then switch to `share\openmm\examples`.

Run benchmarks (see more options `python benchmark.py --help`):

```sh
python benchmark.py --platform=OpenCL --style=table --test=gbsa,rf,pme,apoa1rf,apoa1pme,apoa1ljpme
```

```sh
python benchmark.py --platform=HIP --style=table --test=gbsa,rf,pme,apoa1rf,apoa1pme,apoa1ljpme
```
