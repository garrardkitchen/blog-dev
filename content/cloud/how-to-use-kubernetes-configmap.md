---
aliases: [/blog/how-to-use-kubernetes-configmap/]
title: "How to Use Kubernetes ConfigMap"
date: 2020-11-02T15:31:12Z
draft: false
tags: [cloud, kubernetes, configmap, cm, kubectl, secrets, "best practice"]
---


In this article, you'll learn how to create ConfigMaps, consume them from Pods, and keep confidential values in an appropriate secret system. That distinction matters because cloud failures usually emerge at the seams between configuration, identity, networking, and operations.

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
$ kubectl get cm config-demo-1 -o yaml
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
$ kubectl get cm config-demo-2 -o yaml
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
      image: registry.k8s.io/busybox:1.36.1
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

Use a Secret only for confidential values, and combine it with RBAC, encryption at rest, and a deliberate rotation path. The following command demonstrates object creation; avoid literal secrets on a shared command line in real environments.

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


 {{< callout type="error" >}}
This is just a Base64 encoded string of `mypassword`.  This is not secure enough.  We need another way of to protect our sensitive information/passwords.

{{< /callout >}}

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

## 2026 technical review

## Technical review: configuration is visible data

ConfigMaps are for non-confidential configuration. Values are visible through the Kubernetes API to authorised subjects and may appear in Pod specifications, environment inspection, logs, or support bundles. They are not encrypted merely because the cluster is managed.

Kubernetes Secrets use base64 in their API representation, which is encoding rather than confidentiality. Protect them with least-privilege RBAC, encryption at rest, restricted etcd and control-plane access, audit policy, and careful Pod permissions. A subject that can create a Pod in a namespace can often arrange for that Pod to consume namespace Secrets, so review workload-creation privileges as part of the secret boundary.

For cloud workloads, prefer workload identity and an external secret manager where feasible, then mount or retrieve only the values needed. Avoid secret values on command lines because shell history and process inspection may capture them. Do not expose MongoDB directly with a public LoadBalancer for a production example; use a private service, authentication, network policy, backup, and a maintained image pinned by digest or controlled tag.

ConfigMap and Secret values consumed as environment variables do not update inside a running process. Mounted volumes can update eventually, but the application must reread them. Choose an explicit reload or rollout strategy.

## References
- [Kubernetes documentation](https://kubernetes.io/docs/home/)
- [Kubernetes API reference](https://kubernetes.io/docs/reference/kubernetes-api/)

## Closing thought

Kubernetes makes configuration easy to distribute; reliability depends on knowing which values may be visible, which must be protected, and how running Pods learn that either has changed.
