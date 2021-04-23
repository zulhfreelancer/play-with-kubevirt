Commands and codes that I use during [KubeVirt](https://kubevirt.io) exploration

## Installation

### Operator

```
$ export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)
$ echo $VERSION
$ wget https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
$ kubectl apply -f kubevirt-operator.yaml
```

### Enable nested virtualization

```
$ kubectl create configmap kubevirt-config -n kubevirt --from-literal debug.useEmulation=true
# OR
$ kubectl apply -f ./1-setup/kubevirt-configmap.yaml
```

### Custom Resource

```
$ wget https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
$ kubectl apply -f kubevirt-cr.yaml
```

## Verify Installation

```
# deployment
$ kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.phase}"

# all components
$ kubectl get all -n kubevirt
```

## Virtctl CLI

```
$ VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
$ ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
$ echo ${ARCH}
$ curl -L -o virtctl https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-${ARCH}
$ chmod +x virtctl
$ sudo install virtctl /usr/local/bin
```

## Use KubeVirt

### Create VM

```
$ wget https://raw.githubusercontent.com/kubevirt/kubevirt.github.io/master/labs/manifests/vm.yaml
$ kubectl apply -f vm.yaml
```

### Manage VM

```
# get running VMs
$ kubectl get vms
```

```
# start VM
$ virtctl start testvm

# Or, use kubectl
# start the virtual machine:
$ kubectl patch virtualmachine myvm --type merge -p \
    '{"spec":{"running":true}}'

# stop the virtual machine:
$ kubectl patch virtualmachine myvm --type merge -p \
    '{"spec":{"running":false}}'
```

```
# check VM instances
$ kubectl get vmis

# enter VM instance
$ virtctl console testvm

# shutdown VM (will delete VMI but keep the VM)
$ virtctl stop testvm

# delete VM (will delete VM first, followed by VMI - might take a while for the VMI to get deleted)
$ kubectl delete vm testvm
```
