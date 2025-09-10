
# 📘 Podman & OpenShift Quiz

Each question has 3 options. **Only one is correct.**

---

### 1. What are the main differences between Podman and Docker?
- A) Podman requires a central daemon, Docker does not.
- B) Podman is daemonless and rootless, Docker uses a daemon. ✅
- C) Podman cannot build images, Docker can.

---

### 2. How does Kubernetes (and OpenShift) help orchestrate containers compared to running them directly with Podman?
- A) By scheduling, scaling, and networking containers automatically. ✅
- B) By running containers faster than Podman.
- C) By replacing container registries with its own.

---

### 3. Which file do you edit to configure secure and insecure registries for Podman?
- A) `/etc/docker/daemon.json`
- B) `/etc/containers/registries.conf` ✅
- C) `/etc/podman/config.yaml`

---

### 4. What command would you use to pull the UBI 9 image from Red Hat’s registry?
- A) `docker run ubi9`
- B) `oc import-image ubi9`
- C) `podman pull registry.access.redhat.com/ubi9/ubi:9.3` ✅

---

### 5. How do you assign the correct SELinux context to a host directory before mounting it into a Podman container?
- A) `chown -R 0:0 /data`
- B) `semanage fcontext -a -t container_file_t '/data(/.*)?'` ✅
- C) `podman selinux enable /data`

---

### 6. What is the purpose of the `:Z` option when mounting volumes in Podman?
- A) It sets read-only mode for the volume.
- B) It relabels the volume content for exclusive container access. ✅
- C) It compresses the volume.

---

### 7. Which command shows the logs of a Podman container?
- A) `podman logs <container_name>` ✅
- B) `podman show <container_name>`
- C) `podman output <container_name>`

---

### 8. How can you run a Podman container with environment variables and a published port?
- A) `podman run -e VAR=value -p 8080:80 image` ✅
- B) `podman start -env VAR=value -expose 8080:80 image`
- C) `podman env run image:tag`

---

### 9. What is the difference between `podman exec` and `podman logs`?
- A) `exec` runs commands inside a container, `logs` shows container output. ✅
- B) `exec` stops the container, `logs` restarts it.
- C) `exec` builds the image, `logs` exports the image.

---

### 10. How do you build an image from a Dockerfile with Podman?
- A) `podman build -t myimage:1.0 .` ✅
- B) `podman create image Dockerfile`
- C) `podman make image -f Dockerfile`

---

### 11. In the provided Dockerfile example, which command extracts a `.zip` file into `/etc/nginx`?
- A) `COPY nginx_conf.zip /etc/nginx/`
- B) `RUN unzip -o /tmp/nginx_conf.zip -d /etc/nginx` ✅
- C) `ADD nginx_conf.zip /etc/nginx/`

---

### 12. What OpenShift editions exist, and what is the difference between OKE and OCP?
- A) OKE = OpenShift Kubernetes Engine (core), OCP = OpenShift Container Platform (full enterprise). ✅
- B) OKE = free version, OCP = paid version.
- C) OKE = OpenShift for developers, OCP = OpenShift for storage.

---

### 13. In the OpenShift web console, what is the Topology view used for?
- A) To visualize and manage deployed applications graphically. ✅
- B) To configure SELinux settings.
- C) To create Dockerfiles.

---

### 14. Which command do you use to create a new project (namespace) in OpenShift with the CLI?
- A) `oc project new <name>`
- B) `oc new-project <name>` ✅
- C) `oc create ns <name>`

---

### 15. How would you deploy an application from a GitHub repository in OpenShift using the CLI?
- A) `oc new-app https://github.com/sclorg/nodejs-ex.git` ✅
- B) `oc run --git=https://github.com/sclorg/nodejs-ex.git`
- C) `oc create git-app nodejs-ex`

---

### 16. Which command exposes an OpenShift Service as a Route?
- A) `oc expose svc/<service_name>` ✅
- B) `oc route create <service_name>`
- C) `oc new-route <service_name>`

---

### 17. How can you port-forward traffic from a local machine to a Service in OpenShift?
- A) `oc route-forward svc/myapp 8080:80`
- B) `oc port-forward svc/myapp 8080:80` ✅
- C) `oc expose port svc/myapp 8080:80`

---

### 18. What command shows the top CPU/memory usage across all OpenShift pods?
- A) `oc adm top pods -A` ✅
- B) `oc get usage pods`
- C) `oc top all pods`

---

### 19. Which `oc` command would you use to see events sorted by timestamp in a project?
- A) `oc get events --sort-by=.lastTimestamp` ✅
- B) `oc describe events`
- C) `oc logs events`

---

### 20. What are three steps you would take to troubleshoot a pod in CrashLoopBackOff state in OpenShift?
- A) Check logs, describe pod events, verify image/command configuration. ✅
- B) Delete the pod, restart the cluster, reinstall OpenShift.
- C) Change SELinux mode to permissive, remove SCCs, disable quotas.
