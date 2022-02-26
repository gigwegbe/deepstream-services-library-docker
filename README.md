[![GitHub release (latest by date including pre-releases)](https://img.shields.io/github/v/release/prominenceai/deepstream-services-library-docker?include_prereleases)](https://github.com/canammex-tech/deepstream-services-library/releases)
[![GitHub license](https://img.shields.io/github/license/Naereen/StrapDown.js.svg)](https://github.com/prominenceai/deepstream-services-library-docker/blob/master/LICENSE)
[![Ask Me Anything !](https://img.shields.io/badge/Ask%20me-anything-1abc9c.svg)](https://discord.com/channels/750454524849684540/750457019260993636)

# deepstream-services-library-docker
This repo contains a Dockerfile and utility scripts for the [Deepstream Services Library](https://github.com/prominenceai/deepstream-services-library) (DSL). 

Important notes:
* Jetson only - dGPU files are still to be developed.
* Base image - [`nvcr.io/nvidia/deepstream-l4t:6.0-triton`](https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_docker_containers.html#id2)
* Remaining issues require two additional build steps -- in interactive mode -- to build the `libdsl.so` once the container is running.
* The `deepstream-services-library` repo is cloned into `/opt/prominenceai/` collocated with `/opt/nvidia/`
* **CAUTION: this repo is in the early stages of development -- please report issues!**

### Files
* `docker_setup.sh` - one time installation of Docker and its system dependencies.
* `docker_build.sh` - builds the docker image.
* `docker_run.sh` - builds and runs the container in interactive mode - removes the container on exit.
* `Dockerfile` - Docker file used by the `docker_build.sh` script.

*... and many thanks to [@gigwegbe](https://github.com/gigwegbe) for creating the above files!*

## Contents
* [Install system dependencies](#install-system-dependencies)
* [Build the Docker Image](#build-the-docker-image)
* [Build and run the Docker container](#build-and-run-the-docker-container)
* [Build the libdsl.so](#build-the-libdslso)
* [Generate caffemodel engine files](#generate-caffemodel-engine-files-optional)
* [Commit your file changes](#commit-your-file-changes)
* [Troubleshooting](#troubleshooting)

---

### Install system dependencies
First, clone the repo and make all scripts executable.
```
chmod +x *.sh
```
Then, run the one-time setup script (unless you already have Docker setup) - **Caution: this script reboots your Jetson device without warning!** Ensure all work is saved before you execute.
```
./docker_setup.sh
```

### Build the Docker Image
Execute the Docker build script to build the `dsl.alpha:latest` image.
```bash
./docker_build.sh 
```

### Build and run the Docker container
Execute the Docker run script to build and run the container in interactive mode.
```
./docker_run.sh
```

### Build the `libdsl.so`
Once in interactive mode, copy and execute the following commands.
```
# cd /opt/prominenceai/deepstream-services-library
# make -j 4
# make lib
```
**Note:** the library will be copied to `/usr/local/lib` once built.    

### Generate caffemodel engine files (optional)
Enable DSL logging if you wish to monitor the process (optional).
```
# export GST_DEBUG=1,DSL:4
```
execute the python script in the `/opt/prominenceai/deepstream-services-library` root folder.
```
# python3 make_caffemodel_engine_files.py
```
**Note:** this script can take several minutes to run.

The following files are generated (Jetson Nano versions by default)
```
/opt/nvidia/deepstream/deepstream/samples/models/Primary_Detector_Nano/resnet10.caffemodel_b8_gpu0_fp16.engine
/opt/nvidia/deepstream/deepstream/samples/models/Secondary_CarColor/resnet18.caffemodel_b8_gpu0_fp16.engine
/opt/nvidia/deepstream/deepstream/samples/models/Secondary_CarMake/resnet18.caffemodel_b8_gpu0_fp16.engine
/opt/nvidia/deepstream/deepstream/samples/models/Secondary_VehicleTypesresnet18.caffemodel_b8_gpu0_fp16.engine
```
Update the Primary detector path specification in the script to generate files for other devices. 

### Commit your file changes.
**Caution** the `docker_run.sh` script includes the `-rm` flag in the run command to remove the container on exit. All changes you've made in the running container will be lost.

Use the following Docker commands to list the running containers and to commit your changes to a new image.
```
$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED      STATUS      PORTS     NAMES
1a0b1ebbc321   214a38f109f0   "/bin/bash"   2 days ago   Up 2 days             serene_cartwright

$ sudo docker commit 1a0b1ebbc321  <repo-name>:<tag>

```
Then update your `docker_run.sh` script with your new image name.

---

### Troubleshooting
#### Docker errors after updating device software - including the latest version of Docker.
```
docker: Error response from daemon: failed to create shim: OCI runtime create failed: container_linux.go:380: 
starting container process caused: error adding seccomp filter rule for syscall clone3: permission denied: unknown.
```
NVIDIA requires a specific release of Docker - see https://github.com/dusty-nv/jetson-containers/issues/108

Solution, reinstall the correct version with the following commands.
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update
sudo apt-get install nvidia-docker2=2.8.0-1
```
