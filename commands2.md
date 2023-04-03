k explain rs - explains ReplicaSet - for example it returns a correct API version we should use
k scale rs new-replica-set --replicas 5
k create deployment httpd-frontend --replicas 3 --image httpd:2.4-alpine
k -n finance run redis --image redis
k run redis --image redis:alpine -l tier=db
k expose pod redis --port 6379 --name redis-service
k run custom-nginx --image nginx --port 8080   # Start a nginx pod and let the container expose port 8080
k run httpd --image httpd:alpine --expose --port 80 #Create pod and a ClusterIP service to expose pod using targetPort 80
k create cm webapp-config-map --from-literal APP_COLOR=darkblue --from-literal APP_OTHER=disregard

 ```yaml
    containers:
    - env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:
            name: webapp-config-map           # The ConfigMap this value comes from.
            key: APP_COLOR # The key to fetch.        configMapRef:
```

 ```yaml
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    envFrom:
    - secretRef:
        name: db-secret
```

kubectl exec ubuntu-sleeper -- whoami # Check the user that is running the container

 ```yaml
spec:
  securityContext:
    runAsUser: 1001
    capabilities:
      add:
      - NET_BIND_SERVICE
      drop:
      - all
 ```

 ```yaml
  containers:
    resources:
      limits:
        memory: 20Mi
      requests:
        memory: 5Mi
```

k create sa dashboard-sa

 ```yaml
spec:
  selector:
    matchLabels:
      name: web-dashboard
  template:
    metadata:
        name: web-dashboard
    spec:
      serviceAccount: dashboard-sa
      serviceAccountName: dashboard-sa
```

k set serviceaccount deployment web-dashboard dashboard-sa # bad autocompletion

kubectl taint nodes node01  spray=mortein:NoSchedule # add taint
k taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule- # remove taint

 ```yaml
spec:
  tolerations:
    - key: spray
      value: mortein
      effect: NoSchedule
      operator: Equal
```

```yaml

spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
```

kubectl -n elastic-stack logs kibana

```yaml
spec:
  containers:
  - name: simple-webapp
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
    livenessProbe:
      httpGet:
        path: /live
        port: 8080
      periodSeconds: 1
      initialDelaySeconds: 80
```

k label node node01 color=blue # Adds label to a node

`kubectl top node ## CPU/memory stats for nodes

```yaml
spec:
  initContainers:
  - command:
    - sh
    - -c
    - sleep 600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: warm-up-1
```

k get pods -l env=prod,bu=finance,tier=frontend # Get pods with labels
k set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2
k create job throw-dice-job --image kodekloud/throw-dice
`kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *"`

PVC and PV can't be created using imperative command - go to doc, and grab definition

kubectl config --kubeconfig=/root/my-kube-config use-context research # Switch context to a new one defined in a separate file


helm search hub wordpress
helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo joomla
helm install bravo bitnami/drupal
helm uninstall bravo
helm pull --untar bitnami/apache

k api-resources

kubectl proxy 8001&
root@controlplane:~# curl localhost:8001/apis/authorization.k8s.io

Add the --runtime-config=rbac.authorization.k8s.io/v1alpha1 option to the kube-apiserver.yaml # to enable v1alpha1 version
k convert -f ingress-old.yaml > ingress-new.yaml # convert tool is not installed by default - follow doc to install

k get role
kubectl describe rolebinding kube-proxy -n kube-system
k get pods --as dev-user
kubectl create role developer --verb=create --verb=list --verb=delete --resource=pods # k create role -h
kubectl create rolebinding  dev-user-binding --role=developer --user=dev-user # k create rolebinding -h

kubectl create clusterrolebinding storage-admin --clusterrole=storage-admin --user=michelle
`k create clusterrole storage-admin --verb='*' --resource=sc --resource=persistentvolumes`

kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins' # enabled admission controller including default ones