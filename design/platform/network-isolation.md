Network Isolation
=================

This design relates to isolating our users resources from other users, which is an important precaution when operaing a multi-tenant platform.


Requirements
------------
* Allow a users services to talk to each other (within their namespace)
* Allow a users services to talk to the Micro platform services (currently in the default namespace)
* Allow traffic from the Micro platform namespaces
* Deny traffic from other namespaces
* Potentially allow users to arrange "sharing" resources between specific namespaces (in the future, not initially)


Proposal
--------
The proposal is to use Kubernetes [NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/) resources to define these rules.
* NetworkPolicy is already a part of Kubernetes, but _does_ require a CNI that supports this
* ScaleWay offers several choices of CNI when creating a Kubernetes cluster. The default (or at least what we have now) is [Cilium](https://cilium.io/)
* Cilium _does_ support NetworkPolicy, so that is the first place to start
* Provision defaults at the time that we create a users namespace. It should prove trivial to define a generic NetworkPolicy that provides the basic functionality we require
* At a later date we can choose to look at other CNIs if we desire, without affecting what we've already created
* We can still provision NetworkPolicy resources even when the CNI doesn't support it - they will just have no effect. This allows us to retain maximum compatibility with any Kubernetes deployment, leaving the CNI implementation up to whoever is building each cluster


Examples
--------

### Deny traffic from any other namespace
* This example denies traffic from anywhere other than pods in the user's namespace
* Return packets from other namespaces _is_ allowed (so pods can make requests and receive responses from places like `default` for example)

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: prawn-cruft-prawn
  name: deny-from-other-namespaces
spec:
  podSelector:
    matchLabels:
  ingress:
  - from: # Allows pods in this namespace to communicate with each other
    - podSelector: {}
```

### Allow traffic only from specific namespaces
* This example denies traffic from anywhere other than pods in the user's namespace or namespaces bearing specific labels
* Return packets from other namespaces _is_ allowed (so pods can make requests and receive responses from places like `default` for example)
* The `default` namespace would be able to communicate freely with pods in this namespace (assuming that the `default` namespace has a label `owner: micro` in place)

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: prawn-cruft-prawn
  name: allow-only-from-micro
spec:
  podSelector:
    matchLabels:
  ingress:
  - from: # Allow pods in this namespace to talk to each other
    - podSelector: {}
  - from: # Allow pods in the "default" namespace to talk to pods in this namespace:
    - namespaceSelector:
        matchLabels:
          owner: micro
```


Links
-----
There is a lot to read about NetworkPolicy, but here are some relevant examples to what we are trying to achieve:
* https://cilium.io/
* https://kubernetes.io/docs/concepts/services-networking/network-policies/
* https://github.com/ahmetb/kubernetes-network-policy-recipes/blob/master/04-deny-traffic-from-other-namespaces.md
* https://github.com/ahmetb/kubernetes-network-policy-recipes/blob/master/06-allow-traffic-from-a-namespace.md
