---
title: "K8s Selectors and Labels"
date: 2022-01-15T13:30:42Z
tags: [kubernetes, k8s, deployment, pod, replicaset, selectors, equality-based, set-based, kubectl]
---

Right, what's the deal with all the labels and metadata in a Deployment manifest?!!!!

Take this [example](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      foo: baa
  template:
    metadata:
      labels:
        app: nginx
        foo: baa
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Here, we see `metadata` twice, and also there's mention of `matchLabels` in `selector`???  What does it all mean???

Ok, let me explain 👀 ...

# The first metadata reference

A deployment manifest _kind_ is a manifest that describes the desired state of your application(s).  I say applications here as a POD can contain more than one container (application).  The desired part of this is found in a ReplicaSet _kind_ manifest.  For example, you'd use a ReplicaSet if you require to have 2 replicas (instances) of your POD running.  A Deployment manifest is a short-hand way of stipulating this, ergo, saves you having to create 2 separate manifests.  Makes sense?  Good.

{{< hint info >}}
Behind the scenes, it is the **Deployment Controller** that monitors your deployment's desired state and if it differs, it will return to it's desired state.        
{{</ hint >}}

So why is there two mentions of `metadata`?  Ok, The first reference identifies this Deployment object itself:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: nginx-deployment
  labels:
    app: nginx
```

So for example, if you want to delete this object, you'd issue either of these **kubectl** commands:

```yaml
kubectl delete deployments nginx-deployment -n default
OR
kubectl delete deployments -l app=nginx-deployment -n default
```

The latter delete example above uses an **equality-based** label selector condition.  There's also a **set-based label** selector condition.  You can read about these here ➡ [labels and selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

# The second metadata reference

The second metadata reference is inside the template that is being used to describe the POD to be created AND is separate from the Deployment itself.

```yml
...
template:
  metadata:
    labels:
      app: nginx
      foo: baa
  spec:
    containers:
    - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

To demonstrate this, let's run the following command:

```powershell
❯ kubectl.exe get pods -l app=nginx -n default --show-labels
NAME                                READY   STATUS         RESTARTS   AGE   LABELS
nginx-deployment-6494589cc9-242v8   1/1     Running        0          3s    app=nginx,foo=baa,pod-template-hash=6494589cc9   
nginx-deployment-6494589cc9-2fdf5   1/1     Running        0          3s    app=nginx,foo=baa,pod-template-hash=6494589cc9   
nginx-deployment-6494589cc9-n8vxb   0/1     ErrImagePull   0          3s    app=nginx,foo=baa,pod-template-hash=6494589cc9      
```

If you look at the LABELS column you can see labels that are not found in the Deployment metadata - eg `foo=baa`.

You can also use set-based selector; here's an example that produces the same outcome:

```powershell
❯ kubectl.exe get pods -l "app in (nginx)" -n default --show-labels
NAME                                READY   STATUS             RESTARTS   AGE     LABELS
nginx-deployment-6494589cc9-6sn7v   1/1     Running            0          6m10s   app=nginx,foo=baa,pod-template-hash=6494589cc9
nginx-deployment-6494589cc9-htxpx   0/1     ImagePullBackOff   0          6m10s   app=nginx,foo=baa,pod-template-hash=6494589cc9
nginx-deployment-6494589cc9-qh8nc   0/1     ErrImagePull       0          6m10s   app=nginx,foo=baa,pod-template-hash=6494589cc9
```

# Binding deployment to pod

So, how do we couple the Deployment with the Pod?  Well, this is where the `selector` comes into play. The `selector` instructs Kubernetes to match on the `app` label for those that have a value of `nginx` and that the `foo` label that has the value of `baa`.  

```yml
spec:
  selector:
    matchLabels:
      app: nginx
      foo: baa
```

I hope this has made sense and has cleared up any confusion you may have had.