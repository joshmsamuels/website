---
layout: blog
title: "Deprecated APIs Removed In 1.16: Here’s What You Need To Know"
date: 2019-07-18
slug: api-deprecations-in-1-16
---

**Author**: Vallery Lancey (Lyft)

As the Kubernetes API evolves, APIs are periodically reorganized or upgraded.
When APIs evolve, the old API is deprecated and eventually removed.

The 1.16 release will deprecate APIs for four services:

* NetworkPolicy
* PodSecurityPolicy
* DaemonSet, Deployment, StatefulSet, and ReplicaSet
* Ingress

None of these resources will be removed from Kubernetes or deprecated in any way.
However, to continue using these resources, you must use a current version of
the Kubernetes API.

# Migration Details

* NetworkPolicy: will no longer be served from **extensions/v1beta1** in **v1.16**.
  * Migrate to the networking.k8s.io/v1 API, available since v1.8. Existing persisted
    data can be retrieved/updated via the networking.k8s.io/v1 API.
* PodSecurityPolicy: will no longer be served from **extensions/v1beta1** in **v1.16**.
  * Migrate to the policy/v1beta1 API, available since v1.10. Existing persisted
    data can be retrieved/updated via the policy/v1beta1 API.
* DaemonSet, Deployment, StatefulSet, and ReplicaSet:  will no longer be served
from **extensions/v1beta1**, **apps/v1beta1**, or **apps/v1beta2** in **v1.16**.
  * Migrate to the apps/v1 API, available since v1.9. Existing persisted data
    can be retrieved/updated via the apps/v1 API.
* Ingress: will no longer be served from **extensions/v1beta1** in **v1.18**.
  * Migrate to the networking.k8s.io/v1beta1 API, serving Ingress since v1.14.
    Existing persisted data can be retrieved/updated via the networking.k8s.io/v1beta1 API.

# What To Do

Kubernetes 1.16 is due to be released in September 2019, so be sure to audit
your configuration and integrations now!

* Change YAML files to reference the newer APIs
* Update custom integrations and controllers to call the newer APIs
* Update third party tools (ingress controllers, continuous delivery systems)
to call the newer APIs

Migrating to the new Ingress API will only require changing the API path - the
API fields remain the same. However, migrating other resources (EG Deployments)
will require some updates based on changed fields. You can use the
`kubectl convert` command to automatically convert an existing object:
`kubectl convert -f <file> --output-version <group>/<version>`.

For example, to convert
an older Deployment to apps/v1, you can run:
`kubectl convert -f ./my-deployment.yaml --output-version apps/v1`
Note that this may use non-ideal default values. To learn more about a specific
resource, check the Kubernetes [api reference](https://kubernetes.io/docs/reference/#api-reference).

You can test your clusters by starting an apiserver with the above resources
disabled, to simulate the upcoming removal. Add the following flag to the
apiserver startup arguments:

`--runtime-config=apps/v1beta1=false,apps/v1beta2=false,extensions/v1beta1/daemonsets=false,extensions/v1beta1/deployments=false,extensions/v1beta1/replicasets=false,extensions/v1beta1/networkpolicies=false,extensions/v1beta1/podsecuritypolicies=false`

# Want To Know More?

Deprecations are announced in the Kubernetes release notes. You can see these
announcements in
[1.14](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#deprecations)
and [1.15](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.15.md#deprecations-and-removals).

You can read more [in our deprecation policy document](https://kubernetes.io/docs/reference/using-api/deprecation-policy/#deprecating-parts-of-the-api)
about the deprecation policies for Kubernetes APIs, and other Kubernetes components.
Deprecation policies vary by component (for example, the primary APIs vs.
admin CLIs) and by maturity (alpha, beta, or GA).

These details were also [previously announced](https://groups.google.com/forum/#!topic/kubernetes-dev/je0rjyfTVyc)
on the kubernetes-dev mailing list, along with the releases of Kubernetes 1.14
and 1.15. From Jordan Liggitt:

```
In case you missed it in the 1.15.0 release notes, the timelines for deprecated resources in the extensions/v1beta1, apps/v1beta1, and apps/v1beta2 API groups to no longer be served by default have been updated:

* NetworkPolicy resources will no longer be served from extensions/v1beta1 by default in v1.16. Migrate to the networking.k8s.io/v1 API, available since v1.8. Existing persisted data can be retrieved/updated via the networking.k8s.io/v1 API.
* PodSecurityPolicy resources will no longer be served from extensions/v1beta1 by default in v1.16. Migrate to the policy/v1beta1 API, available since v1.10. Existing persisted data can be retrieved/updated via the policy/v1beta1 API.
* DaemonSet, Deployment, StatefulSet, and ReplicaSet resources will no longer be served from extensions/v1beta1, apps/v1beta1, or apps/v1beta2 by default in v1.16. Migrate to the apps/v1 API, available since v1.9. Existing persisted data can be retrieved/updated via the apps/v1 API.

To start a v1.15.0 API server with these resources disabled to flush out dependencies on these deprecated APIs, and ensure your application/manifests will work properly against the v1.16 release, use the following --runtime-config argument:

--runtime-config=apps/v1beta1=false,apps/v1beta2=false,extensions/v1beta1/daemonsets=false,extensions/v1beta1/deployments=false,extensions/v1beta1/replicasets=false,extensions/v1beta1/networkpolicies=false,extensions/v1beta1/podsecuritypolicies=false
```
