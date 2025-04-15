---
title: "Kubernetes Pod Disruption Budget and the Helm hasKey Function"
date: 2022-01-21T14:19:01Z
draft: false
---

# Pod Disruption Budget

When working with Kubernetes, one crucial component of configuration is known as a PDB (Pod Disruption Budget).  A PDB will ensure your workload remains running when you work through a **Voluntary Disruption**.

What on earth is a **Voluntary Disruption**?  A **Voluntary Disruption** is when you trigger an action _that causes the_ disruption.  For example, if you wish to upgrade a Minor AKS version or any action that recycles a Node Pool. Click here ➡ [Disruptions](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) to read up on what Disruptions are.  
 
This is what a PDB manifest looks like this.  This example tells Kubernetes to make sure there's always a minimum of 2 Pods running during a disruption:

```yml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-awesome-microservice-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-awesome-microservice-api
```

We use Helm Charts so part the declaration that wraps the `minAvailable` property looks like this:

```yaml
{{- if hasKey .Values "pdb"  }} 
{{- if hasKey .Values.pdb "minAvailable" }} 
minAvailable: {{ .Values.pdb.minAvailable }}   
{{- end }}   
{{- else }}
minAvailable: {{ max (sub .Values.replicaCount 1) 1 }} 
{{- end }}
```

What on earth is going on here then?!

There are a few rules that I need to accommodate for within our workload PDBs.  These are:
- Apply a specific value that may be contained within a values .yaml file
- Provide a default value if one is not supplied
- Ensure there's at least 1 pod running throughout the disruption so a workload doesn't go offline during this period. 😱

# How to use the value provided by the developer

I've designed our CICD pipeline so we get base configurations (one .NET Framework IIS workloads, one for .NET Core Web workloads, one for ... etc.) from one git repo, and get all application(s) configuration properties from the application git repo itself (📂 /.k8s/).  It is here from within the application's repo we set the properties for a service(s) within a `values-<env>.yaml` file.   If there's more than one application found in an application's git repo, the name is reflected in the name of values .yaml file to provide uniqueness - eg `values-<consumer>-<env>.yaml`.

It is in this values .yaml file we set - if at all - a value to the `pdb.minAvailable` nested property.
 
Here, in this control flow, we are checking that both pdb and minAvailable properties exist.  If they do, we apply the `minAvailable` value.  We use the hasKey function to good effect to check for the existence of a property in another property:

```yml
{{- if hasKey .Values "pdb"  }} 
{{- if hasKey .Values.pdb "minAvailable" }} 
minAvailable: {{ .Values.pdb.minAvailable }}   
{{- end }}   
{{- else }}
```

# How to provide a default value

If the application configuration does not contain a `minAvailable` property, we take the value found in the `replicaCount` value and use this.  However, we do not insist on the same value, but instead 1 less - `(sub .Values.replicaCount 1)`.

```yml
...
{{- else }}
minAvailable: {{ max (sub .Values.replicaCount 1) 1 }} 
{{- end }} 
```

# How to ensure there's at least one pod running

We must ensure at least one instance of a workload is running and if we simply reduced the `replicaCount` by one and left it at that, we could end up with a budget of zero.  I don't want this to happen.  What I do here is use the `max` function to good effect to safeguard against this ever being a zero - and if `replicaCount: 1`, then the expression would read `max (0) 1`, meaning 1 would be the value used. 

```yml
...
{{- else }}
minAvailable: {{ max (sub .Values.replicaCount 1) 1 }} 
{{- end }} 
```

# References

- [Helm Functions](https://github.com/helm/helm-www/blob/main/content/en/docs/chart_template_guide/function_list.md#floor)
- [Flow control](https://helm.sh/docs/chart_template_guide/control_structures/)
- [Disruptions](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)
- [Best practices - Voluntary Disruption](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-scheduler#voluntary-disruptions)
