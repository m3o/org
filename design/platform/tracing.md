Tracing
=======

The time has come for distributed tracing. Fortunately we don't need to reinvent the wheel in 2020 because there are some great tools and useful standards at our disposal.


Header injection
----------------

In the Micro platform a sensible idea would be to leave the tracing decision up to the client. In this way we can ensure that by default nothing is traced unless a decision had been specifically made. The Micro API would be a good starting point for tracing decisions. The client would be responsible to establishing the root span with appropriate metadata (eg originating API call or action), and all child spans would be aggregated beneath that (as the context is passed down).


Sampling
--------

It is rare that one would want to trace 100% of the requests in a system, so this needs to be set with a sensible default. However, when tracing distributed communications it is important that every service involved in the lifecycle of a request participates for requests that have been selected. This is passed from client to server in the tracing context headers. The proposed behaviour is that a server will only trace a request if it carries a header stating that it should (all other requests would not be traced). In this way we get to sample which requests are traced, and ensure that we get a full end-to-end trace when they are.


Server side
-----------

Server-side tracing is a good place to start because it gives us good coverage and allows us to lean on the familiar pattern of using handler wrappers. Much as with the metrics wrapper, a tracing wrapper can automatically submit spans with useful metadata that doesn't require the user to even think about it:

- Service name
- Service version
- Method name
- Duration
- Error
- Namespace?


Client side
-----------

Getting the clients to also submit spans can give us a very good picture of what is going on in the platform. More useful for debugging large scale performance issues because we don't just see which RPC handlers are being hit, but which client services are hitting them. We'd end up submitting a similar set of tags:

- Client name (the name of the service making the calls)
- Client version
- Service name (the name of the service being called)
- Method name
- Duration
- Error
- Namespace?


APM
---

RPC is not where tracing ends however. It would be fascinating to have insight to other aspects of application / platform performance, and as we are in control of the framework we're in a good position to be able to do that. Given that we already have access to the tracing context we should be able to trace such things as:

- DB queries (in the store service)
- Pub/sub timings in the broker

By providing our users access to the OpenTracing (OpenTelemetry) headers they could then pass them on to other 3rd party libraries which support those too.


Toolkit
-------

There are some good projects around which will be helpful to us.

- (Jaeger)[https://www.jaegertracing.io/] from Uber is an obvious choice. Mature, well known, and easy to get going on a small scale before going large.
- (Tempo)[https://grafana.com/oss/tempo/] from Grafana also seems like it would be worth checking out given the quality of Grafana itself, the fact that we've looked at Loki for logging, and the potential ease of integration between these tools.
