Monitoring
==========

There are several aspects to "monitoring", but for now the scope is to monitor the Micro platform, services, and infrastructure. At a later date we may choose to offer some monitoring services to our users. This will be informed by the experience of monitoring our own stack first, but for now is not on the agenda.


Background
----------

The Kubernetes ecosystem really assumes Prometheus as the de-facto way to monitor things. It has its pros and cons, but the critical point should be the level of adoption in the space, which is unquestionable.


Requirements
------------

* Capture performance data to assist with scaling decisions, and to establish behavioural baselines
* Generate notifications when certain key thresholds are crossed
* Monitor infrastructure:
    - ETCd
    - NATS
    - K8S
    - CockroachDB
* Monitor Micro platform services:


Challenges
----------

* Keeping the cost down (monitoring systems can quickly become a big-data problem if not used effectively)
* Keeping the notifications relevant, not generating false positives


Proposal
--------

* Decide on what the critical metrics are for alerting (start with things that have already given trouble).
* Host a small Prometheus instance somewhere. It would be nice for this to be outside of K8S (because that is one of the main targets for monitoring), but being inside the K8S cluster makes it trivial to detect pods and scrape their etrics.
* Start with very aggressive retention policies.
* Sign up for a free OpsGenie account (limited to 5 users). This integrates easily, and takes care of on-call rotas / time windows etc.
* Host a Grafana instance to provide dashboarding and alerting capabilities.
* Out-of-the-box dashboards are available which make sense of the metrics we'll gather for Kubernetes, which facilitate scaling decisions as well as highlighting especially resource-hungry deployments.
* Aggregation for service status can be achieved by keeping an eye on the Kubernetes deployments. Assuming that we have reliable liveness and readiness probes we can actually get a very good _generic_ view of the health of a service (healthy / ready pods).
