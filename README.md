# deepstream-services-library-docker
This repo contains a Dockerfile and utility scripts for the [Deepstream Services Library](https://github.com/prominenceai/deepstream-services-library) (DSL). 

Import notes:
* Jetson only - dGPU files are still to be developed.
* Base image: [`docker pull nvcr.io/nvidia/deepstream-l4t:6.0-samples`](https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_docker_containers.html#id2)
* Remaining issues require two additional build steps -- in interactive mode -- to build the `libdsl.so` once the container is running.
* The `deepstream-services-library` repo is cloned into `/opt/prominenceai/` colocated with `/opt/nvidia/`
* **CAUTION: this repo is in the early stages of development -- please report issues!**

### Files
* `docker_setup.sh` - one time installation of Docker and its system dependencies.
* `docker_build.sh` - builds the docker image.
* `docker_run.sh` - builds and runs the container in interactive mode - removes the container on exit.
* `Dockerfile` - Docker file used by the `docker_build.sh` script.

*... and many thanks to [@gigwegbe](https://github.com/gigwegbe) for creating the above files!*

### Install system dependencies
First, clone the repo and make all scripts executable.
```
chmod +x *.sh
```
Then, run the one-time setup script (unless you already have Docker setup) - **Caution: this script reboots your Jetson device without warning!** Insure all work is saved before you execute.
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
Once in interactive mode, copy and execute the follow comands.
```
# cd /opt/prominenceai/deepstream-services-library
# make -j 4
# make lib
```
**Note:** the library will be copied to `/usr/local/lib` once built.    

---

### Troubleshooting
#### Docker errors after updating device softwere - including latest version of Docker.
```
docker: Error response from daemon: failed to create shim: OCI runtime create failed: container_linux.go:380: 
starting container process caused: error adding seccomp filter rule for syscall clone3: permission denied: unknown.
```
Requires a patch release from NVIDIA - see https://github.com/dusty-nv/jetson-containers/issues/108

Solution
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update
sudo apt-get install nvidia-docker2=2.8.0-1
```

