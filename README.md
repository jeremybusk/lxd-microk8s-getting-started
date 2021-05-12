# lxd-microk8s-getting-started
Kubernetes using microk8s in lxd

# Anatomy of Kubernetes

- https://kubernetes.io/docs/concepts/overview/components/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/


# Using Ubuntu and LXD to Create a Quick and Dirty HA k8s Environment

- Create base Ubuntu 20.04 Host - https://ubuntu.com/download/server

Init LXD - https://linuxcontainers.org/lxd/getting-started-cli/
```
apt install zfsutils-linux
lxd init
```

- https://microk8s.io/docs/lxd
- https://microk8s.io/docs/high-availability

```
lxc profile create microk8s
wget https://raw.githubusercontent.com/ubuntu/microk8s/master/tests/lxc/microk8s-zfs.profile -O microk8s.profile && cat microk8s.profile | lxc profile edit microk8s && rm microk8s.profile
lxc launch -p default -p microk8s ubuntu:20.04 testk1
```

# Prep LXC containers apparmor
```
lxc shell testk1
snap install microk8s --classic
cat > /etc/rc.local <<EOF
#!/bin/bash
apparmor_parser --replace /var/lib/snapd/apparmor/profiles/snap.microk8s.*
exit 0
EOF
chmod +x /etc/rc.local
```
run this on testk2 and testk3

# Join microk8s nodes together in HA cluster
```
lxc shell testk1
microk8s add-node  # This needs to be ran after each join as it changes on HA cluster modification
# Copy command it shows you to run on other two nodes
```
Add testk1 and testk2


# Enable Add-ons
https://microk8s.io/docs/addons
```
microk8s enable ingress
microk8s enable dns
micork8s status (You can see 
```

# Run your first deployment
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/


Here is a simple example
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30001
```
