# Longhorn test setup

## Setup

Goal: deploy a local 3x microk8s cluster and install Longhorn into it. 

Then deploy microk8s via charm-microk8s, and install longhorn via helm

```
juju add-model longhorn localhost

juju deploy ./bundles/longhorn-bundle.yaml


juju scp k8s/*.yaml 0:/tmp


# run on some microk8s machine

juju ssh 0 -- sudo microk8s enable hostpath-storage dns helm

juju ssh 0 -- sudo microk8s helm repo add longhorn https://charts.longhorn.io

juju ssh 0 -- sudo microk8s helm repo update

juju ssh 0 -- sudo microk8s.kubectl create namespace longhorn-system

juju ssh 0 -- sudo microk8s helm3 install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --values /tmp/longhorn-values.yaml


# wait until settled, 10m or so

juju ssh 0 -- sudo microk8s.kubectl get all -A
```

## Verification

```
juju ssh 0 -- sudo microk8s.kubectl get all -A

juju ssh 0 -- sudo microk8s.kubectl get storageclass # should have longhorn class

# create pvc and pod
juju ssh 0 -- sudo microk8s.kubectl apply -f /tmp/my-pvc.yaml
juju ssh 0 -- sudo microk8s.kubectl apply -f /tmp/test-pod.yaml

# check
juju ssh 0 -- sudo microk8s.kubectl get pvc
juju ssh 0 -- sudo microk8s.kubectl get pv
juju ssh 0 -- sudo microk8s.kubectl exec -it test-pod -- /bin/sh
grep longhorn /proc/mounts 
/dev/longhorn/pvc-6295c49b-6643-45a0-ac89-98f33bbc3f00 /usr/share/nginx/html ext4 rw,relatime 0 0

```
