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
The proposal is to use Kubernetes `NetworkPolicy` resources to define these rules.
* NetworkPolicy is already a part of Kubernetes, but _does_ require a CNI that supports this
* ScaleWay offers several choices of CNI when creating a Kubernetes cluster. The default (or at least what we have now) is ["Cilium"](https://cilium.io/)
* Cilium _does_ support NetworkPolicy, so that is the first place to start
* Provision defaults at the time that we create a users namespace. It should prove trivial to define a generic NetworkPolicy that provides the basic functionality we require
* At a later date we can choose to look at other CNIs if we desire, without affecting what we've already created
* We can still provision NetworkPolicy resources even when the CNI doesn't support it - they will just have no effect. This allows us to retain maximum compatibility with any Kubernetes deployment, leaving the CNI implementation up to whoever is building each cluster


Links
-----
There is a lot to read about NetworkPolicy, but here are some relevant examples to what we are trying to achieve:
* https://github.com/ahmetb/kubernetes-network-policy-recipes/blob/master/04-deny-traffic-from-other-namespaces.md
* https://github.com/ahmetb/kubernetes-network-policy-recipes/blob/master/06-allow-traffic-from-a-namespace.md
