# Edit containers config file to add secure and insecure registry

```sh
vi /etc/containers/registries.conf
```
# Pull image

```sh
podman pull [image_name]:[tag]
```

# Crete a path with selinux context for container

```sh
mkdir -pv [path]
sudo semange -a t container_file_t '[path](/.*)?'
sudo restorecon -R [path]
podman unshare chown -Rv 27:27  [path] 
```

# Init a container with volume

```sh
podman run -d --name=[container_name] -v [host_paht]:[container_path]/:Z --pod [pod_name] [image:tag]
```

# See logs

```sh
podman logs [container_name] > [file_name].log
```

# Crete a Pod with env_vars

```sh
podman run -d --name=[container_name] -e [ENV_VAR_KEY]=[env_var_value] -p [host_port]:[container_pod] [image_name]:[tag]
```
# Copy files to container

```sh
podman cp [container_name]:[container_path_file] [local_path]
```

# Login external repo

```sh
podman login -u [user] [external_registry:port] -p '[password]'
```

# Search images in remote registry

```sh
podman search [external_registry:port]
```

# Inspect images in remote registry

```sh
scopeo inspect docker://[external_registry:port]/[image_name]
```
# Use image with diferent tag

```sh
podman run -d --name=[container_name] -p [host_port:container_port] [registry:port]/[image_name]:[tag]
```

# Connect to a container to wxecute commands

```sh
podman exec -it [container_name] [bash or /bin/bash]
```

# Commit container changes

```sh
podman commit [container_name] [registry:port]/[image_name]:[tag]
```

# Push images

```sh
podman push [registry:port]/[image_name]:[tag] [registry:port]/[image_name]:[tag]
```

# Save Image on tar file

```sh
podman save -o [file_name].tar [registry:port]/[image_name]:[tag]
```

# Complete docker file

(https://github.com/psehgaft/Red-Hat-Certified-Specialist-in-Containers-and-Kubernetes?organization=psehgaft&organization=psehgaft)[Dockerfile]

```Dockerfile
#Please use image registry.access.redhat.com/ubi8/ubi:8.4
FROM
#Set yourself as the Maintainer of this image.
MANTAINER
#Provide a brief description of the image.
LABEL description=
#Install nginx and unzip
RUN
#Use ENV to set the variable PORT to 90
ENV 
#Use the variable $PORT to expose the set port
EXPOSE $PORT
#Copy nginx_conf.zip to /tmp/
COPY
#Extract /tmp/nginx_conf.zip to /etc/nginx and overwrite any existing files (hint use -o and -d options for unzip)
RUN
#Extract llama_cart.tar to /usr/share/nginx/html/
ADD
#Set the workdir to /tar_file/ and copy llama_cart.tar to this directory without uncompressing it.
WORKDIR
COPY
#Set the container to start "nginx" with options "-g" and "daemon off" as overwritable options(hint: use ENTRYPOINT and CMD).
ENTRYPOINT
CMD
```

```Dockerfile
# Please use image registry.access.redhat.com/ubi8/ubi:8.4
FROM registry.access.redhat.com/ubi8/ubi:8.4

# Set yourself as the Maintainer of this image.
MANTAINER psehgaft

# Provide a brief description of the image.
LABEL description="Web server from Dockerfile"

# Install nginx and unzip
RUN yum install *y nginx unzip

# Use ENV to set the variable PORT to 90
ENV PORT=90

# Use the variable $PORT to expose the set port
EXPOSE $PORT

# Copy nginx_conf.zip to /tmp/
COPY nginx_conf.zip /tmp/

# Extract /tmp/nginx_conf.zip to /etc/nginx and overwrite any existing files (hint use -o and -d options for unzip)
RUN unzip -o /tmp/nginx_conf.zip -d /etc/nginx

# Extract llama_cart.tar to /usr/share/nginx/html/
ADD llama_cart.tar /usr/share/nginx/html/

# Set the workdir to /tar_file/ and copy llama_cart.tar to this directory without uncompressing it.
WORKDIR /tar_file/
COPY llama_cart.tar .

# Set the container to start "nginx" with options "-g" and "daemon off" as overwritable options(hint: use ENTRYPOINT and CMD).
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off"]
```

# Build container from tar file

On Dockerfile path:

```sh
podman build -t [image_name]:[tag] [working_dir]
```

# OCP deploy APP from Git

```sh
oc new-app --
```

This file is used to test the Git repo setup on the infrastructure machine.
