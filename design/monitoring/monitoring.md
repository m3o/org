# Monitoring

## Introduction

This document outlines a plan for monitoring in Micro, both with regards to the user experience in Micro and as an operator of the platform. This document outlines various phases, each phase has a a goal, proposed solution and alternative solution. 

### Where we are now

* Developers can view the logs for their application using the `micro logs` CLI command. Locally this watches a log file and in the platform / kubernetes profiles this follows the logs for any pods which match the labels for this service, however there is no log retention beyond what comes by default with Kubernetes.

* There is no functionality for developers to trace requests or monitor core service metrics such as CPU and Memory usage.

* There is a Promethius plugin which can be used to export custom metrics, however services don't expose metrics by default and the package is specific for Promethius meaning that whilst the package can be used safely locally (since Promethius uses a pull model), these is no way to query these metrics.

* In the platform, we have Promethius and Grafana deployed. Some m3o services import the Promethius plugin, exposing metrics and others don't. The core micro services do not expose any custom metrics however information such as CPU / Memory can be extracted from the pods.

## Phase 1 - Visibility into requests entering the network

### Goal

Currently the API logs all requests in the CLF, however there is no retention / analysis performed on these logs. The Proxy does not log any inbound requests. 

The goal of this phase is to enable easy monitoring of all requests which have entered the platform over the last two weeks and be able to block any bad-actors manually using their IPs. 

### Requirements

* We need a way to generate logs in the CLF for all incoming traffic, specifically for any requests entering the plaform via the gRPC proxy.

* We need a way to block requests using an IP.

* We need log retention and aggregation.

### Proposal

The first two requirements can be solved using an nginx ingress controller as part of our platform deployment. nginx has support for gRPC and logs all request in the CLF by default, likewise it has built-in support for rate-limiting and allowing / denying access based on IP.

With regards to log retention / aggregation, I propose we use [Grafana Loki](https://grafana.com/oss/loki/). Loki is Promethius but for Logs, uses an almost identical query language and integrates seamlessly into Grafana, allowing us to visualise monitoring and logs in a single dashboard. Loki integrates directly into Kubernetes using [promtail](https://grafana.com/docs/loki/latest/clients/promtail/installation/) which can be installed as a DaemonSet on each Kubernetes node and collects the logs of all containers within a cluster. Metadata such as pod lables is automatically scraped and indexed so we can do queries using service name, version etc.

[This is an example](https://files.slack.com/files-pri/T05675Y01-F01DGU56X5G/image.png) of how Loki logs can be visualized on Grafana alongside our metrics, and [this video](https://www.youtube.com/watch?v=0n2UNzk2OaI&ab_channel=Grafana) does a good job demonstrating the benefits of using Loki in combination with Promethius.

Loki uses BoltDB behind the scenes for indexing which means all of the persistance can be offloaded to a filestore. When using the helm chart, this defaults to a Persistant Volume, however I would suggest swapping this out to use the S3 backend, as we have done for the blob store.

### Alternative Solution

For rate limiting / access control, the simplest solution would be using Cloudflare, however this option has been disregarded due to cost (doing wildcard DNS proxying requires Cloudflare Enterprise which would cost thousands of pounds per month). 

The alternative to using nginx is building rate limiting into the Micro API and Proxy. For the http API there are lots of libraries which offer rate limiting, for gRPC we could leverage the [go-grpc-middleware ratelimit](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/ratelimit/ratelimit.go) package. This is not ideal in my opinion as our implementation of rate-limiting will not be anywhere near as efficient as that of nginx.

As an alternative to using Loki, we could use a managed logging solution such as [Datadog](https://datadoghq.com) which  can be easily integrated (like Loki, it offers a [helm chart](https://docs.datadoghq.com/agent/kubernetes/log/?tab=helm) which uses DaemonSets to collect the logs).

The drawbacks of using Datadog are:
* Whilst Datadog does have a community supported integration into Grafana, it's nowhere near as seamless as Loki which was built from the ground-up with this usecase in mind.
* It will be more expensive than hosting something ourselves
* The API to query logs isn't as extensive as Loki, which restricts us in the future if we want to update `micro logs` to query the log aggregator rather than Kubernetes directly.

The benefits of using Datadog are:
* Reduced operational overhead
* It integrates into Datadog APM if we wanted to use this for tracing (although the API for that is somewhat limited).

## Phase 2 - Analytics

### Goal

Track analytical events within the platform, such as new user signups and visualize this data on an internal dashboard. 

### Requirements

Whilst we already have a method for persisting events to a database (the events service writes all events to the default store with a timeout of 24 hours), we may need to investigate a better suited database... [Note: Alex to expand on this]

We need a method to visualize the data collected

### Proposal

To visualize the data, I would suggest [Metabase](https://github.com/metabase/metabase), a simple application designed for business analytics. Whilst Metabase has a managed offering, I would suggest we run it in Kubernetes alongside Promethius. 

Metabase can connect to most databases and since CockroachDB is compatible with PostgreSQL, it should be able to generate dashboards from the events persisted in Cockroach.

### Alternative Solution

Alternatively, we could use Grafana to visualize this data. Like Metabase, it can import PostgreSQL data sources and visualize it on a dashboard. Whilst this would be simpler operationally (we're already running Grafana), I believe it would be sensible to seperate the business analytics from the operational ones. 

[Note: Alex - any other reasons to use Metabase over Grafana?]

## Phase 3.1 - Tracing (Internal Usage)

### Goal

We don't yet have any mechanism to trace a request. We need a way that as platform operators, we can trace any request on the platform. Trade IDs should be assigned to each request at ingress and propagated downstream. There is an issue that the internal services, e.g. store don't take a context in their interface so the tracing won't conver these requests unless we amend the interface / public functions to accept this argument.

### Proposed Solution

I'd suggest assigning TraceIDs to each request as they enter the network at the API / Proxy layer.

[Grafana Tempo](https://github.com/grafana/tempo) is an obvious solution since it is designed to integrate with Loki and the Grafana dashboard. It uses Loki for trace discovery and then writes the traces to a filestore (such as S3) for later lookup. The main drawback to Tempo is how new it, Grafana only released it this month.

### Alternative Solutions

If we choose not to assign TraceIDs at the API / Proxy layer, they can be generated by nginx at ingress. 

If we chose to use Datadog for tracing in phase one, the simplest solution would be using Datadog's APM. The benefit is that it comes out-of-the box once we install the logs and setup the APM. The downside is that it's a manged solutions which looks like it will cost > ~$50 per month and that the traces are only available for querying for 15 minutes after they're generated (this goes up to 15 days for high latancy traces). There also doesn't appear to be any API for querying traces, hence it might not be possible to expose these traces to our customers.

## Phase 3.2 - Tracing (Providing access via Micro CLI for M3O Users)

### Goal

Give users access to traces for their services via a CLI command.

### Proposed Solution

Add a `micro trace [tradeID]` command to the Micro CLI. There would be a lot of complexity involved in offering this service out-the-box with micro, so I suggest we create the monitoring service to M3O. The proto for this service would reside in micro/micro/protos, and the CLI command would be baked in (it's nicer than the dynamically generated `micro monitoring trace --traceid=foo`). 

If the service isn't running in an environment (such as localhost), we should return an error such as "Monitoring is not available in this environment" rather than the route not found error.

The CLI command would output the logs for the trace, grouped by service. Of course the CLI isn't the medium for visualising traces, however I think it's important to give users access to this as soon as possible. Once we have a web dashboard, we can add tracing there also.

The monitoring service would call our internal tracing provider to retrieve the trace. The only uncertainty here is how to ensure a user can only query a trace for their own namespace. By default Tempo indexes traces using just trace ID, however we'd need to configure this to also include the namespace. 

## Phase 4 - Monitoring (Providing access via Micro CLI for M3O Users)

### Goal

Give users access to basic information relating to how their services are performing via a CLI command. The user should be able to see live information such as CPU and Memory usage per service.

Stretch goal: add a CLI command to get historical usage per service, e.g. `micro stat history [serviceName]`.

### Proposed Solution

Much like the tracing solution, we'll add a custom CLI command: `micro stat [serviceName: optional]`. If no service name is provided, it will output the stats for all services in the namespace. If a service name is provided it will only output the stats for this service.

This CLI command will call the Monitoring.Stat RPC which will load the information from Promethius via the API using a PromQL query. Given that Promethius indexes pod labels, it should be very easy to retrieve this information.

## Phase 5 - Monitoring for M3O users via Web Dashboard

TBD futher down the line.

## Phase 6 - Alerting for M3O users

TBD futher down the line.

## Notes

You can see a graphical representation of this plan [here](https://drive.google.com/file/d/16Vx2xDjDTpCo0eILpFl1WkL2ZeweW0SF/view?usp=sharing).