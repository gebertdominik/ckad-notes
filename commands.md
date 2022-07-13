# Architecture

#### Display Taints
 `kubectl describe nodes | grep -i taint`

#### Remove Taint
`kubectl taint nodes --all node-role.kubernetes.io/master`

#### List all API objects and their APIGROUP
`kubectl api-resources`

#### Create pod using yaml file

```basic.yaml
apiVersion: v1
kind: Pod
metadata:
  name: basicpod
  labels:
    type: webserver
spec:
  containers:
  - name: webcont
    image: nginx
    ports:
    - containerPort: 80
 ```

 `kubectl create -f basic.yaml`

#### Get pod info(with internal IP)
`kubectl get pod -o wide`

#### Delete pod
`kubectl delete pod basicpod`

#### Create service
```basicservice.yaml
apiVersion: v1
kind: Service
metadata:
  name: basicservice
spec:
  selector:
    type: webserver
  ports:
  - protocol: TCP
    port: 80
```

`kubectl create -f basicservice.yaml`

#### Create multicontainer pod
```basic.yaml
apiVersion: v1
kind: Pod
metadata:
  name: basicpod
  labels:
    type: webserver
spec:
  containers:
  - name: webcont
    image: nginx
    ports:
    - containerPort: 80
  - name: fdlogger
    image: fluent/fluentd
 ```

 `kubectl create -f basic.yaml`

#### Create deployment (without yaml)
 `kubectl create deployment firstpod --image=nginx`

#### Get pods from specified namespace
`kubectl get pod -n mynamespace`

#### Get all pods from all namespaces
`kubectl get pod --all-namespaces`

#### Get several resources at once
`kubectl get deploy,rs,po,svc,ep`
