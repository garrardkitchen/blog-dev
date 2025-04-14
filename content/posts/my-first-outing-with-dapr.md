---
title: "My First Outing With Dapr"
date: 2020-11-06T09:54:02Z
tags: [dapr, kubernetes, redis, secret store csi driver, aks, nestjs, keda]
---

TL;DR: Not as forgiving as I'd have liked ...

{{< hint info >}}

I was a speaker at a meet-up in Manchester in late 2020. I spoke about Dapr, Keda and the NestJS Framework.  My talk topic was on "Writing less code - let your architecture and abstractions help with your *-cases".  The `*` in the title is a wildcard for use/edge/corner.  

My code examples can be found here (includes both docker compose & Kubernetes manifests) - https://github.com/garrardkitchen/meetup-nov20
{{< /hint >}}

## Challenge #1 

This took a little longer than I'd have liked!

I was using the internal DNS to resolve the port of my redis service.  My Redis single instance was deployed via a deployment manifest, along with a LoadBalancer Service - purely to give me remote access.


I'd first create a secret, by typing:

```
$ kubectl create secret generic db-passwords --from-literal=redis-password='<password>'
```

This is the deployment manifest:
```yml
apiVersion: v1
kind: Service
metadata:
  name: redis-svc
  namespace: meetup-dapr-demo
  labels:
    run: redis-svc
spec:
  type: LoadBalancer
  ports:
    - port: 6379
      targetPort: 6379
      protocol: TCP
  selector:
    run: redis
---
apiVersion: apps/v1 #  for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: redis
  namespace: meetup-dapr-demo
spec:
  selector:
    matchLabels:
      run: redis
  replicas: 1
  template:
    metadata:
      labels:
        run: redis
    spec:
      containers:
        - name: cache
          image: redis
          args: ["redis-server", "--requirepass", $(PASSWORD) ]
          env:
            - name: PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-passwords
                  key: redis-password
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
          ports:
            - containerPort: 6379

```

This deployed correctly.

I then deployed my Dapr state store component:

```yml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: mystore
  namespace: meetup-dapr-demo
spec:
  type: state.redis
  metadata:
    - name: redisHost
      value: redis.meetup-dapr-demo.svc.cluster.local:6379
    - name: redisPassword
      value: "********"
```

However, I could not for the life of me give my application access to the state store!

```
$ dapr logs -a http-api -k -n meetup-dapr-demo
...
time="2020-11-06T09:47:02.218770653Z" level=error msg="process component mystore error, redis store: error connecting to redis at redis.meetup-dapr-demo.svc.cluster.local:6379: dial tcp: lookup redis.meetup-dapr-demo.svc.cluster.local on 10.0.0.10:53: no such host" app_id=http-api instance=http-api-6bc44f8957-q2lvn scope=dapr.runtime type=log ver=0.11.3
```

Having trying every permutation known to _non-gender-specific-person-entity_ I remembered I was kaing it available behind a service.  So, I'd been using `redis.meetup-dapr-demo.svc.cluster.local:6379` when I should have used ` redis-svc.meetup-dapr-demo.svc.cluster.local:6379`.

Once I'd corrected my mistake, it connected without error.

```yml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: mystore
  namespace: meetup-dapr-demo
spec:
  type: state.redis
  metadata:
    - name: redisHost
      value: redis-svc.meetup-dapr-demo.svc.cluster.local:6379
    - name: redisPassword
      value: "********"
```

## Challenge #2

secrets!

You're application is going to report something similar to this - NOAUTH Authentication required - if you're Dapr is deployed to a different namespace to that of your application:

```
time="2020-11-06T11:19:06.985273661Z" level=error msg="process component mystore error, redis store: error connecting to redis at redis-svc.meetup-dapr-demo.svc.cluster.local:6379: NOAUTH Authentication required." app_id=http-api instance=http-api-7d49cf59d5-9blwf scope=dapr.runtime type=log ver=0.11.3
```

To circumvent this, you must create a role and binding this to the default `ServiceAccount`.  This role `secret-reader` allows a `get` of the `secrets` resource within the `meetup-depr-demo` namespace.  An example manifest is here:

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: meetup-dapr-demo
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dapr-secret-reader
  namespace: meetup-dapr-demo
subjects:
  - kind: ServiceAccount
    name: default
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

Once deployed, you'll see something similar to this in your dapr logs:
```
time="2020-11-06T11:23:20.529658232Z" level=info msg="component loaded. name: mystore, type: state.redis" app_id=http-api instance=http-api-7d49cf59d5-kszdz scope=dapr.runtime type=log ver=0.11.3
```

---

{{< hint info >}}

This post was created some time ago.  Now, we're using the Secrets Store CSI Driver to map Azure KeyVault secrets to containers running in our AKS clusters.

Ref: https://docs.microsoft.com/en-us/azure/aks/csi-secrets-store-driver
{{< /hint >}}

