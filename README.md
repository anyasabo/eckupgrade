### ECK Upgrades with Helm

This is a repo designed to test deploying the ECK operator with CRD definitions, and then upgrading the CRDs in the Helm chart. We start with the 1.0.0-beta release and upgrade to the 1.1.2 release. It includes the all in one and the ES quickstart for that version.

### Testing

[Install](https://helm.sh/docs/intro/install/) a >=3.2.0 version of Helm.

Create a fresh k8s cluster.

Use one of the `beta` branches then:

```
helm install test .
```

Switch to one of the `112` branches then:

```
helm upgrade test .
```

### Test Outcomes

#### Going from `beta` to `112`:

This branch uses the ["Method 1"](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/#method-1-let-helm-do-it-for-you) Helm recommended way to install CRDs, using the `crds` directory. 

The CRDs are not upgraded because Helm does not support upgrading CRDs. This also causes the v1 ES resource to fail with:

>Error: UPGRADE FAILED: unable to recognize "": no matches for kind "Elasticsearch" in version "elasticsearch.k8s.elastic.co/v1"

From the [Helm docs](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/)::
>There is no support at this time for upgrading or deleting CRDs using Helm. This was an explicit decision after much community discussion due to the danger for unintentional data loss. Furthermore, there is currently no community consensus around how to handle CRDs and their lifecycle.

#### Going from `beta-template-crds` to `112-template-crds`

I thought maybe if we did not use the special CRD handling Helm has (and just template them like regular resources), it might work, but forgot that there is a necessary order of operations. So you receive:

```
$ helm install test .
Error: unable to build kubernetes objects from release manifest: unable to recognize "": no matches for kind "Elasticsearch" in version "elasticsearch.k8s.elastic.co/v1beta1"
```

because the CRD is not installed before the ES resource. This likely is not a problem for installing just the operator, which led to the next test.

#### Going from `beta-template-crds-no-es` to `112-template-crds-no-es`

For this I removed the Elasticsearch resource from the chart (which is most similar to our all-in-one.yaml). This approach seems to work as expected. The CRDs and operator are installed and upgraded as expected. This seems to be the only viable option. I'm not sure it is something we *should* do though. This is their suggested [method 2](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/#method-2-separate-charts) for charts with CRDs, so I think it would be okay. We can just only install the operator + CRDs in one chart. People wanting to install CRs would need to use a separate chart that is installed after the operator + CRD chart.

#### Going from `beta-es` to `112-es`

For this I tried a chart with just a v1beta1 Elasticsearch resource, and then upgrading the chart to just a v1 Elasticsearch resource. It worked as expected.


### Conclusion

We could support two separate charts if we wanted. One chart to install the CRDs and the operator, another to install stack resources. There is not a viable way to have one chart for both.

It will still be a pain to support cluster-wide, single namespace, and multi-namespace deployments in one chart, but I think that is true no matter what deployment method we choose.


### Cleanup

```
helm uninstall test
kubectl delete crd apmservers.apm.k8s.elastic.co
kubectl delete crd elasticsearches.elasticsearch.k8s.elastic.co
kubectl delete crd kibanas.kibana.k8s.elastic.co
```

For branches using the `crds` directory, the chart installed the CRDs, but Helm intentionally [does not support removing them](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/):

>There is no support at this time for upgrading or deleting CRDs using Helm. This was an explicit decision after much community discussion due to the danger for unintentional data loss. Furthermore, there is currently no community consensus around how to handle CRDs and their lifecycle.


### Notes

This was created by copying the all-in-one yamls and then adding minimal templating, essentially only adding the

```
meta.helm.sh/release-name: {{ .Release.Name }}
meta.helm.sh/release-namespace: {{ .Release.Namespace }}
```

to the default labels, making sure all resources had these labels, and using the release namespace of the hardcoded `elastic-system` namespace. Looking at it further I think those `meta` annotations actually may be added by Helm by default and we do not need to set them manually.


v1beta1 CRDs themselves are deprecated in k8s 1.16 and are being removed in the near future (looks like it recently changed to [1.22](https://github.com/kubernetes/kubernetes/pull/92220/files)). This should work fine though as the patch works with CRD and was designed to work around the Deployments group change.
