
# OCP WorkGroup DO180
## Audience
Developers, sysadmins, and DevOps engineers starting with **containers** and **Red Hat OpenShift**.

## Objectives
- Understand containers, Podman, and Kubernetes basics.
- Run and manage containers locally with Podman.
- Transition workloads into OpenShift using CLI and Web Console.
- Deploy applications from images, Dockerfiles, and Git repos.
- Troubleshoot containers and pods effectively.

---

## Agenda 


**Containers and Kubernetes** — concepts, images, registries |
**Podman Essentials** — registry config, pull, run, volumes, env, logs |
**Build with Dockerfiles** — building images, commits, pushes |
**Red Hat OpenShift Components & Editions** |
**Navigate the OpenShift Web Console** |
**Kubernetes & OpenShift CLI** — oc client walkthrough |
**Deploy Applications** — from images, Git repos, templates |
**Networking** — Services, Routes, exposing apps |
**Monitoring & Troubleshooting** — oc adm, events, pod logs |

---

## 1. Containers and Kubernetes 
- **Containers**: lightweight, isolated runtime environments.
- **Podman vs Docker**: Podman is daemonless, rootless, Red Hat supported.
- **Kubernetes/OpenShift**: orchestrates containers, scaling, networking, RBAC.

---

## 2. Podman Essentials 

### 2.1 Registry Setup
```sh
vi /etc/containers/registries.conf
```
```ini
[registries.search]
registries = ['registry.do180.lab:5000', 'registry.access.redhat.com', 'registry.redhat.io', 'docker.io', 'quay.io']

[registries.insecure]
registries = ['registry.do180.lab:5000']
```

### 2.2 Pull and Run Containers
```sh
podman pull registry.access.redhat.com/ubi9/ubi:9.3
podman run -it --rm registry.access.redhat.com/ubi9/ubi:9.3 bash
```

### 2.3 Volumes & SELinux
```sh
mkdir -pv /data/app
sudo semanage fcontext -a -t container_file_t '/data/app(/.*)?'
sudo restorecon -Rv /data/app
podman run -d --name web -v /data/app:/var/www:Z httpd
```

### 2.4 Logs & Exec
```sh
podman logs web
podman exec -it web bash
```

### 2.5 Pods & Env Vars
```sh
podman run -d --name db -e MYSQL_ROOT_PASSWORD=redhat -p 3306:3306 mysql:8
```

---

## 3. Building Images with Dockerfiles 

### Example: UBI + Nginx
```Dockerfile
FROM registry.access.redhat.com/ubi8/ubi:8.4
LABEL description="Web server from Dockerfile"
RUN yum -y install nginx unzip && yum clean all
ENV PORT=90
EXPOSE $PORT
COPY nginx_conf.zip /tmp/
RUN unzip -o /tmp/nginx_conf.zip -d /etc/nginx
ADD llama_cart.tar /usr/share/nginx/html/
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off"]
```

### Build & Run
```sh
podman build -t webserver:1.0 .
podman run -d --name=web -p 8080:90 webserver:1.0
```

---

## 4. Red Hat OpenShift Components & Editions 
- **OpenShift Kubernetes Engine (OKE)**: core features.
- **OpenShift Container Platform (OCP)**: full enterprise edition.
- **OpenShift Online / Dedicated**: managed services.
- Components: Master (API, etcd, controllers), Workers (pods), Operators, Networking, Storage.

---

## 5. Navigate the OpenShift Web Console 
- Login with developer/admin view.
- **Topology view**: drag-and-drop apps.
- Monitoring dashboards.
- Project/namespace switching.

**Lab**: Deploy an app from container image via console.

---

## 6. Kubernetes & OpenShift CLI 

### oc basics
```sh
oc login --token=sha256~XXX --server=https://api.cluster:6443
oc new-project training
oc new-app --name hello quay.io/redhattraining/hello-world-nginx
oc get pods,svc,route
```

### Inspect
```sh
oc describe pod <pod>
oc logs <pod>
oc exec -it <pod> -- bash
```

---

## 7. Deploy Applications

### From Image
```sh
oc new-app registry.access.redhat.com/ubi9/httpd-24
```

### From Git
```sh
oc new-app https://github.com/sclorg/nodejs-ex.git
```

### From Template
```sh
oc get templates -n openshift
oc new-app --template=mysql-persistent -p MYSQL_USER=user -p MYSQL_PASSWORD=pass
```

---

## 8. Networking 

### Service + Route
```sh
oc expose svc/hello-world-nginx
oc get route hello-world-nginx -o wide
```

### Port-forward
```sh
oc port-forward svc/hello-world-nginx 8080:80
```

---

## 9. Monitoring & Troubleshooting (10 min)

```sh
oc get events --sort-by=.lastTimestamp | tail -20
oc adm top nodes
oc adm top pods -A
oc describe pod <pod>
oc logs -f <pod>
```

---

## Hands-On Labs Summary
1. Configure Podman registries and pull an image.
2. Run a Podman container with volume + SELinux context.
3. Build and run a custom Nginx image from Dockerfile.
4. Login to OpenShift, create a project, deploy a container image.
5. Expose the app via Route and test externally.
6. Deploy app from Git repo in OpenShift.
7. Monitor pods and troubleshoot errors.

---

## References (APA)

- Red Hat. (2024). *Podman and Buildah documentation*. Red Hat Customer Portal. https://access.redhat.com/documentation
- Red Hat. (2024). *OpenShift Container Platform 4.x documentation*. https://docs.openshift.com/
- Kubernetes Authors. (2024). *Kubernetes concepts and API overview*. https://kubernetes.io/docs/home/
- Walsh, D. (2022). *Podman: The next generation of Linux container tools*. O’Reilly Media.
- Hedges, S. (2023). *Learning OpenShift*. Packt Publishing.
