---
title: "How to Use Kubernetes Configmap"
date: 2020-11-02T15:31:12Z
draft: false
tags: [kuberetes, configmap, cm, kubectl, secrets, best practice]
---

There's a ton of material out there on how to use a ConfigMap.  In this post I will provide a recap on the basics then I drill into how to protect your secrets!  

There are a few ways to create a configMap.  Here, I cover just two of these ways;`--from-env-file` and --from-literal.  I won't cover options like from volume.

## How to create a ConfigMap from a literal

To create a configMap from literals and from the command line, you would type this:

```ps
$ kubectl create configmap config-demo-lit --from-literal=user.name=garrardkitchen --from-literal=user.type=admin
```

To confirm the values, you would type this:
```
kubectl get cm config-demo-lit -o yaml
apiVersion: v1
data:
  user.name: garrardkitchen
  user.type: admin
kind: ConfigMap
metadata:
  creationTimestamp: "2020-11-02T16:06:30Z"
  name: config-demo-lit
  namespace: dapr-demo
  resourceVersion: ****
  selfLink: /api/v1/namespaces/dapr-demo/configmaps/config-demo-lit
  uid: ****
```

---

## How to create a ConfigMap from an .env file

### From the command line

To create a configMap from the command line, you would type this:

```ps
$ kubectl create configmap demo-config --from-env-file=config/.env.prod
```

To confirm the values, you would type this:
```
$ kubectl cm config-demo-1 -o yaml
apiVersion: v1
data:
  foo: baa
  name: garrard
kind: ConfigMap
metadata:
  creationTimestamp: "2020-11-02T15:44:52Z"
  name: config-demo-1
  namespace: dapr-demo
  resourceVersion: ****
  selfLink: /api/v1/namespaces/dapr-demo/configmaps/config-demo-1
  uid: ****
```
👆 `cm` is shorthand for `configmap`

### From a Kubernetes Manifest file

To create a configMap from a manifest, you would create a `yml|yaml` file using the `kind: ConfigMap` like this:

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-demo-2
  namespace: dapr-demo
data:
  foo: baa
  name: garrard  
```  

To confirm the values, you would type this:
```
$ kubectl cm config-demo-2 -o yaml
apiVersion: v1
data:
  foo: baa
  name: garrard
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"foo":"baa","name":"garrard"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"config-demo-2","namespace":"dapr-demo"}}
  creationTimestamp: "2020-11-02T15:49:48Z"
  name: config-demo-2
  namespace: dapr-demo
  resourceVersion: *****
  selfLink: /api/v1/namespaces/dapr-demo/configmaps/config-demo-2
  uid: ****
```

## How to use this in a pod

Here, I'm setting up environment variables from different ConfigMaps. `config-demo-2` is set up from manifest file and `config-demo-lit` is set up from literals.

This is an example pod manifest called `pod-demo.yml`
```yml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-cache
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "env"]
      env:
        - name: NAME
          valueFrom:
            configMapKeyRef:
              name: config-demo-2
              key: name
        - name: ROLE
          valueFrom:
            configMapKeyRef:
              name: config-demo-lit
              key: user.type
  restartPolicy: Never
```

All that this 👆 does, is output to STDOUT, a list of environment variables.

To apply this manifest, type:

```ps
kubectl.exe apply -f .\pod-demo.yml
```

To confirm the 2 environment variables have been set, type:

```
$ kubectl logs test-pod
...
HOSTNAME=test-pod
NAME=garrard
ROLE=admin
...
```

## How to use Secrets

TBC

## How to stop people from finding out your secrets.

At the end of the day, the secrets are only Base64 encoded.  Anyone with the appropriate level of permissions will be able to see your secrets.  One way to stop users from seeing your secrets is by only allow particular groups of people access.

To create a secret, type:

```
$ kubectl create secret generic db-passwords --from-literal=mongodb-password='mypassword'
```

To see what secrets we have, type:
```
$ kubectl get secrets
NAME                           TYPE                                  DATA   AGE
dapr-operator-token-mgdqs      kubernetes.io/service-account-token   3      2d13h
dapr-sidecar-injector-cert     Opaque                                2      2d13h
dapr-trust-bundle              Opaque                                3      2d13h
dashboard-reader-token-j9rcg   kubernetes.io/service-account-token   3      2d13h
db-passwords                   Opaque                                1      5s
default-token-xl2rz            kubernetes.io/service-account-token   3      2d14h
sh.helm.release.v1.dapr.v1     helm.sh/release.v1                    1      2d13h
```

To see the actual password, type:

```
$ kubectl get secrets db-passwords -o yaml
apiVersion: v1
data:
  mongodb-password: bXlwYXNzd29yZA==
kind: Secret
metadata:
  creationTimestamp: "2020-11-03T10:00:14Z"
  name: db-passwords
  namespace: dapr-demo
  resourceVersion: ****
  selfLink: /api/v1/namespaces/dapr-demo/secrets/db-passwords
  uid: ****
```


{{< hint danger >}}

This is just a Base64 encoded string of `mypassword`.  This is not secure enough.  We need another way of to protect our sensitive information/passwords.

{{< /hint >}}

So, what do we do?

Here's a link to how Kubernetes deals with secrets



To use this secret with a deployment, save this to `aks-deploy-mongodb-demo.yml`:

_Please note, this is not a production configuration_

```yml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-svc
  namespace: dapr-demo
  labels:
    run: mongodb-svc
spec:
  type: LoadBalancer
  ports:
  - port: 27017
    targetPort: 27017    
    protocol: TCP  
  selector:
    run: mongodb
---
apiVersion: apps/v1 #  for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: mongodb
  namespace: dapr-demo
spec:
  selector:
    matchLabels:
      run: mongodb
  replicas: 1
  template:
    metadata:
      labels:
        run: mongodb        
    spec:
      containers:
      - name: mongodb        
        image: mongo
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_USERNAME
          valueFrom:
            configMapKeyRef:
              name: config-demo-lit
              key: user.name              
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            configMapKeyRef:
              name: config-demo-lit
              key: user.name        
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-passwords
              key: mongodb-password                      
        - name: MONGO_DBNAME
          value: "orders"

```

To deploy the above 👆, type this:

```ps
$ kubectl apply -f .\aks-deploy-mongodb-demo.yml
```