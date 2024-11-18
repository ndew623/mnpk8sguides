# Some Kubernetes Terminology

## Resources

Resources can refer to things like storage space, RAM, or cpu, but in Kubernetes terminology it can also mean something completely different.

This is from the Kubernetes documentation at https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/

"A **resource** is an endpoint in the Kubernetes API that stores a collection of API **objects** of a certain **kind**; for example, the built-in pods resource contains a collection of Pod objects."

### Objects and Kinds

Just about everything in Kubernetes is an **object** of some **kind**. If you are reading these in order, you just saw an example of a `Deployment` with a `Service`. A Deployment is one **kind** of **object**. A Service is another **kind** of **object**.

You define **objects** with YAML manifests. You set the **kind** in the manifest. If you look back at the Deployment and Service manifests from [How to make a DC/OS-like service in K8s](./dcos-service-in-k8s.md), you'll see that **kind** is an option set at the top of both manifests:
```
apiVersion: apps/v1
kind: Deployment # <-- setting the kind
metadata:
  name: my-new-deployment
spec:
...
```

```
apiVersion: v1
kind: Service # <-- setting the kind
metadata:
  name: myapp-service
spec:
...
```

Remember that Deployments create Pods. A `Pod` is also a **kind** of **object**. You could make a manifest for a `Pod` directly, and that manifest would have `kind: Pod` in it.

You wouldn't usually make a `Pod` directly since pods can't restart or scale themselves. Things like Deployments manage that stuff and make the Pods for you.
But the point is that because a `Pod` is a **kind** of **object**, you could make a Pod manifest, `kubectl apply` it, and create a Pod **object** in the cluster.

Examples of other common **kinds** are Jobs, Namespaces, and Secrets.

### Custom Resources / Custom Resource Definitions (CRDs)

I'm not going to write much here about CRDs because they are a more advanced topic, and this is focused on Kubenetes fundementals. However, it's worth being aware of their existence.

Pods, deployments, and jobs are built-in Kubernetes resources. Some Kuberentes add-ons create their own "custom resources" with their own **kinds**. For example, Istio is software to help manage networking in a cluster. Istio creates a new virtualservice resource, so in clusters using Istio you can make a YAML manifest with `kind: VirtualService`. VirtualService has its own whole set of options that can be set in its manifest, all of which were created by the Istio developers.