### ECK Upgrades with Helm

This is a repo designed to test deploying the ECK operator with CRD definitions, and then upgrading the CRDs in the Helm chart. We start with the 1.0.0-beta release and upgrade to the 1.1.2 release. It includes the all in one and the ES quickstart.

### Testing

[Install](https://helm.sh/docs/intro/install/) a >=3.2.0 version of Helm.

Create a fresh k8s cluster.

Use the `beta` branch.

```
helm install test .
```

Switch to the 112 branch.

```
helm upgrade test .
```

### Outcome

The CRDs are not upgraded because Helm does not support upgrading CRDs. This also causes the v1 ES resource to fail with:

>Error: UPGRADE FAILED: unable to recognize "": no matches for kind "Elasticsearch" in version "elasticsearch.k8s.elastic.co/v1"

From the Helm docs:
>There is no support at this time for upgrading or deleting CRDs using Helm. This was an explicit decision after much community discussion due to the danger for unintentional data loss. Furthermore, there is currently no community consensus around how to handle CRDs and their lifecycle.

I think this may be worked around by moving the CRDs from the `crds` directory and just treating them like a regular template, but would need to do more research on if other projects do that and if there would be other unforeseen issues.

### Cleanup

```
helm uninstall test
kubectl delete crd apmservers.apm.k8s.elastic.co
kubectl delete crd elasticsearches.elasticsearch.k8s.elastic.co
kubectl delete crd kibanas.kibana.k8s.elastic.co
```

Even though the chart installed the CRDs, Helm intentionally [does not support removing them](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/):

>There is no support at this time for upgrading or deleting CRDs using Helm. This was an explicit decision after much community discussion due to the danger for unintentional data loss. Furthermore, there is currently no community consensus around how to handle CRDs and their lifecycle.


### Notes

This was created by copying the all-in-one yamls and then adding minimal templating, essentially only adding the

```
meta.helm.sh/release-name: {{ .Release.Name }}
meta.helm.sh/release-namespace: {{ .Release.Namespace }}
```

to the default labels, making sure all resources had these labels, and using the release namespace.
