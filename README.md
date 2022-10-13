# Some note operation at Openshift 4.11

### 1. Login to openshift
#### 1.1 Login with token
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
