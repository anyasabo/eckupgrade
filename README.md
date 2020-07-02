Install a > 3.2.0 version of Helm. 

Create a fresh k8s cluster.

Use the `beta` branch.

```
helm install test .
```

Switch to the 112 branch.

```
helm upgrade test .
```

Cleanup:

```
helm delete --purge test
```


This was created with minimal templating, essentially only adding the

```
meta.helm.sh/release-name: {{ .Release.Name }}
meta.helm.sh/release-namespace: {{ .Release.Namespace }}
```

to the default labels, making sure all resources had these labels, and using the release namespace.
