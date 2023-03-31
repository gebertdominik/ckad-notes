k explain rs - explains ReplicaSet - for example it returns a correct API version we should use
k scale rs new-replica-set --replicas 5
k create deployment httpd-frontend --replicas 3 --image httpd:2.4-alpine
k -n finance run redis --image redis
k run redis --image redis:alpine -l tier=db
k expose pod redis --port 6379 --name redis-service
k run custom-nginx --image nginx --port 8080   # Start a nginx pod and let the container expose port 8080
k run httpd --image httpd:alpine --expose --port 80 #Create pod and a ClusterIP service to expose pod using targetPort 80
k create cm webapp-config-map --from-literal APP_COLOR=darkblue --from-literal APP_OTHER=disregard

```
    containers:
    - env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:
            name: webapp-config-map           # The ConfigMap this value comes from.
            key: APP_COLOR # The key to fetch.        configMapRef:
```

```
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    envFrom:
    - secretRef:
        name: db-secret
```

kubectl exec ubuntu-sleeper -- whoami # Check the user that is running the container

```
spec:
  securityContext:
    runAsUser: 1001
    capabilities:
      add:
      - NET_BIND_SERVICE
      drop:
      - all
 ```

 ```
  containers:
    resources:
      limits:
        memory: 20Mi
      requests:
        memory: 5Mi
```

k create sa dashboard-sa

```
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

```
spec:
  tolerations:
    - key: spray
      value: mortein
      effect: NoSchedule
      operator: Equal
```