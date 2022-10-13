# Some note operation at Openshift 4.11

### 1. Login to openshift
#### 1.1 Login with token

Kiểm tra trạng thái của cluster
```
oc get clusterversion
```

```bash
oc login --token=sha256~DGoQBL1PxcLir6drRa_S0zfRnxNn0X9cuS42VnFxh1Y --server=https://api.ocp4.example.com:6443
```
#### 1.2 Login to project
```bash
# list project
oc get project
```
```
# login to project
oc project $project_name
```
```
oc get csr
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
```
```
oc edit deployment/ibm-cpd-scheduler-pjc
```

```
oc delete pod ibm-cpd-scheduler-webhook-5b5c77db79-646d2
```


```
oc get node
```

Get machine config to see update at nodes
```
oc get mcp
```
#### 1.4 Kiểm tra operator



```bash
# list danh sách operator
oc get clusteroperator
```
similar

```bash
oc get co
```

```
oc get clusteroperator -o yaml $operator_name
```

### 1.4 Kiểm tra event
```
oc get events

# or 
watch 'oc get events'
```



### 2. Thao tác với pod

Get pod in a namesapce

```
oc get pods -n openshift-console
```

Get pod from a pod in a namespace

```
oc logs  -p console-569dc66bf9-kmmrm -n authentication openshift-authentication
```

Execute a command in a pod
```
oc exec zen-metastoredb-0 ls -alh /var/log
```
Login to pod
```
oc rsh zen-metastoredb-0
```




```
# check log full
oc logs  -p console-7bbf788749-j6h9x -n openshift-authentication --kubeconfig /home/cloud/ocp4upi/auth/kubeconfig
```
oc get clusteroperator

### 2. Thao tác với oc describe
```
 oc describe $source $name
```
ví dụ:
```
oc describe co console
```

### Magic
```
oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Removed"}}'
```

Get resource within a project 
```
oc api-resources --verbs=list --namespaced -o name | xargs -t -n 1 oc get --show-kind --ignore-not-found -n ibm-common-services
```

```
oc patch pv datadir-zen-metastoredb-0 --type=merge -p '{"metadata":{"finalizers":null}}'
```
Mount folder
```
oc rsync /tmp/cockroach-v21.2.9.linux-amd64/lib/ zen-metastoredb-0:/tmp
``` 
