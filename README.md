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
oc project $project_name
```

### 


##### Check logs on pod
```
# check log full
oc logs  -p console-7bbf788749-j6h9x -n openshift-authentication --kubeconfig /home/cloud/ocp4upi/auth/kubeconfig
```
