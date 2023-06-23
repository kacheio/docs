# Kache on Kubernetes

The following describes how to run kache on a local Kubernetes cluster.

!!! warning 
    Please note that this is not intended for use in a production environment. We will provide more sophisticated configurations for operations in the future. 

### Start a Kubernetes cluster

To start a cluster, use minikube or Docker Desktop.

``` sh
minikube start
```

### Deploy ConfigMap

Create a ConfigMap that contains the kache configuration:

``` sh
kubectl create configmap kache-config --from-file=cloud/kubernetes/configmap.yml 
```

Apply the ConfigMap:

``` sh
kubectl apply -f cloud/kubernetes/configmap.yml
```

??? abstract "configmap.yml"
    === "Kubernetes"
    ``` yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
    name: kache-config
    data:
    config.yml: |-
        listeners:
        web1:
            addr: :80
        web2:
            addr: :1337
        upstreams:
        - name: service1
            addr: "http://localhost:8000"
            path: "/service/1"
        - name: service2
            addr: "http://example.com"
            path: "/"
        api:
        port: 1338
        debug: true
        logging:
        level: debug
        provider:
        backend: redis
        redis:
            endpoint: "redis-master:6379"
            username:
            password:
            db:
    ```

### Deploy Redis

``` sh
kubectl apply -f cloud/kubernetes/redis-master.yml
```

??? abstract "redis.yml"
    === "Kubernetes"
    ``` yaml
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: redis-master
    labels:
        app: redis
    spec:
    selector:
        matchLabels:
        app: redis
        role: master
        tier: backend
    replicas: 1
    template:
        metadata:
        labels:
            app: redis
            role: master
            tier: backend
        spec:
        containers:
            - name: master
            image: redis
            resources:
                requests:
                cpu: 100m
                memory: 100Mi
            ports:
                - containerPort: 6379
    ---
    apiVersion: v1
    kind: Service
    metadata:
    name: redis-master
    labels:
        app: redis
        role: master
        tier: backend
    spec:
    ports:
        - port: 6379
        targetPort: 6379
    selector:
        app: redis
        role: master
        tier: backend
    ```

### Deploy Kache

``` sh
kubectl apply -f cloud/kubernetes/kache.yml
```

??? abstract "kache.yml"
    === "Kubernetes"
    ``` yaml
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: kache
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: kache
    template:
        metadata:
        labels:
            app: kache
        spec:
        containers:
            - name: kache
            image: kacheio/kache:main
            imagePullPolicy: Always
            args:
                - "-config.file=/etc/kache/config.yml"
            env:
                - name: NAMESPACE
                valueFrom:
                    fieldRef:
                    fieldPath: metadata.namespace
            volumeMounts:
                - name: config
                mountPath: /etc/kache
            ports:
                - containerPort: 8080
                name: http
                - containerPort: 1337
                name: web
                - containerPort: 1338
                name: api
            resources:
                requests:
                cpu: 100m
                memory: 100Mi
        volumes:
            - name: config
            configMap:
                name: kache-config

    ---
    apiVersion: v1
    kind: Service
    metadata:
    name: kache-service
    labels:
        app: kache
    spec:
    type: LoadBalancer
    ports:
        - name: "http"
        port: 80
        targetPort: http
        - name: "web"
        port: 1337
        targetPort: web
        - name: "api"
        port: 1338
        targetPort: api
    selector:
        app: kache
    ```

### Accessing the service

Check that the Pods are up and running:

``` console
$ kubectl get pods 

NAME                           READY   STATUS    RESTARTS   AGE
kache-54cd8ffd96-xzdqg         1/1     Running   0          14h
redis-master-d4f785667-mpmvg   1/1     Running   0          14h
```

The Kache service is exposed as a LoadBalancer via the service with mapped ports and is accessible on localhost.

``` console
$ kubectl get svc

NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                                      AGE
kache-service   LoadBalancer   10.110.92.73   localhost     80:30135/TCP,1337:32284/TCP,1338:30691/TCP   44h
kubernetes      ClusterIP      10.96.0.1      <none>        443/TCP                                      44h
redis-master    ClusterIP      10.97.188.34   <none>        6379/TCP                                     44h
```

Use the above endpoints to access the service:

``` sh
curl http://127.0.0.1:1337/
```

Access the API:

``` sh
curl http://127.0.0.1:1338/api/
```

## Troubleshooting

If there are problems loading the configuration, verify that the Pod has the latest configuration available:

``` sh
kubectl exec $POD_NAME -- cat /etc/kache/config.yml 
```
