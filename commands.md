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

# Build & Design

#### Create and scale deployment
```
k create deployment try1 --image=nginx
k scale deployment try1 --replicas=2
```

#### Save deployment as yaml
`k get deployment try1 -o yaml > nginx.yaml`

#### Recreate deployment from yaml
`k create -f nginx.yaml`

#### ReadinessProbe

```
    - image: nginx
        imagePullPolicy: Always
        name: nginx
        readinessProbe:
          periodSeconds: 5
          exec:
            command:
              - cat
              - /tmp/healthy

```

#### Liveness + Readiness probe

```
- name: goproxy
        image: k8s.gcr.io/goproxy:0.1
        ports:
        - containerPort: 8080
        readinessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
```

#### Bash into single container pod
`k exec -it podName-123ad2  -- /bin/bash`

#### Bash into multiple container pod (select nginex container)
`k exec -it podName-123ad2 -c nginx  -- /bin/bash`


#### Create job from yaml
```job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  template:
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["3"]
      restartPolicy: Never

```
`k create -f job.yaml`

#### Get job info
`k get job sleepy -o yaml`

#### Create job which runs 5 times
```job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  completions: 5
  template:
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["3"]
      restartPolicy: Never

```
`k create -f job.yaml`


#### Create job which runs 10 times, and executes 2 pods in parallel
```job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  completions: 5
  parallelism: 2
  template:
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["3"]
      restartPolicy: Never

```
`k create -f job.yaml`

#### Create jobs with a deadline of 15 seconds
```job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  activeDeadlineSeconds: 15
  template:
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["3"]
      restartPolicy: Never

```
`k create -f job.yaml`

#### Create a CronJob which creates a Job every 2 minutes, and times out after 10s
```cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: sleepy
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      activeDeadlineSeconds: 10
      template:
        spec:
          containers:
          - name: resting
            image: busybox
            command: ["/bin/sleep"]
            args: ["3"]
          restartPolicy: Never
```
`k create -f cronjob.yaml`

#### Get pods with label app=app1
`k get pods -l app=app1`

#### Create deployment with CPU/Memory requests/limits
`k create deployment limited --image=busybox --dry-run=client -o yaml`

This should output a deployment without limits/requests:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: limited
  name: limited
spec:
  replicas: 1
  selector:
    matchLabels:
      app: limited
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: limited
    spec:
      containers:
      - image: busybox
        name: busybox
        resources: {}
status: {}
```

Edit and add limits/requests:

```
...
    spec:
      containers:
      - image: busybox
        name: busybox
        resources:
          limits:
            cpu: "1"
            memory: "1Gi"
          requests:
            cpu: "0.5"
            memory: "500Mi"
...
```

#### Execute command to init container in Pod

```init-tester.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-tester
  labels:
    app: inittest
spec:
  containers:
  - name: webservice
    image: nginx
  initContainers:
  - name: failed
    image: busybox
    command: [/bin/true]
```
`k create -f init-tester.yaml`

#### Show Custom Resource Definitions (CRD)
`k get crd`

# Deployment Configuration

#### Create ConfigMap from a literalValue, from a file and from a directory of files

Preparation:
```
7803  echo c > primary/cyan
7809  echo m > primary/magenta
7810  echo y > primary/yellow
7811  echo k > primary/black
7812  echo "known as key" >> primary/black
7813  echo blue > favorite
```
```
kubectl create configmap colors \
  --from-literal=text=black \
  --from-file=./favorite \
  --from-file=./primary/
```
`kubectl get configmap colors -o yaml`

#### Use previously configured ConfigMap value in a pod, and check if value is available in the container
```
...
spec:
  containers:
  - image: nginx
    env:
    - name: ilike
      valueFrom:
        configMapKeyRef:
          name: colors
          key: favorite
    imagePullPolicy: Always
...
```

#### Set up environment variables in a pod, based on ConfigMap
```
...
spec:
  containers:
  - image: nginx
    env:
    - name: ilike
      valueFrom:
        configMapKeyRef:
          name: colors
          key: favorite
    envFrom:      
    - confingMapRef:
        name: colors
    imagePullPolicy: Always
...
```

`kubectl exec -c nginx -it try1-1234sd -- /bin/bash -c'echo $ilike'`

#### Create ConfigMap from YAML file

```car-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fast-car
  namespace: default
data:
  car.make: Ford
  car.model: Mustang
  car.trim: Shelby
```

`k create -f car-map.yaml`

#### Add ConfigMap settings to Pod as a volume

```
...
spec:
  containers:
  - image: nginx
    volumeMounts:
    - mountPath: /etc/cars
      name: car-vol
    env:
    - name: ilike
...
  securityContext:{}
  volumes:
  - name: car-vol
    configMap::
      defaultMode: 420
      name: fast-car
...
```

#### Create PersistentVolume from YAML file
```PVol.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvvol-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /opt/sfw
    server: cp
    readOnly: false
```

`k create -f PVol.yaml`
`k get pv`

#### Create PersistentVolumeClaim
```pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-one
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
```
`k create -f pvc.yaml`

After creating PVC, the status of PV should change to `Bound`

#### Use the volume in a Pod

```
...
spec:
  containers:
  - image: nginx
    volumeMounts:
    - mountPath: /opt
      name: nfs-vol
...
  securityContext:{}
  volumes:
  - name: nfs-vol
    persistentVolumeClaim:
      claimName: pvc-one
...

```

#### View the update history of the deployment
`k rollout history deployment deploymentName`

#### Compare two deployment revisions
```
k rollout history deployment deploymentName --revision=1 > one.out
k rollout history deployment deploymentName --revision=2 > two.out
diff one.out two.out
```

#### Verify what would be undone while undoing the rollout(show the template prior to using it)
`kubectl rollout undo --dry-run=client deployment/deploymentName`

#### Undo rollout to specific revision
`kubectl rollout undo deployment deploymentName --to-revision=1`

