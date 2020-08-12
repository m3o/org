Metrics
=======

This is potentially a wide-ranging subject, so some scope needs to be defined. Initially the aim is to provide a basic way to expose metrics from Micro services, which will be used internally to monitor the Micro platform.

This takes several different roadmap items into account.


Requirements
------------
* Introduce a metrics interface which we can build implementations for various metrics stores (Prometheus, Influx etc)
* Capture Go runtime metrics (cpu, memory, gc, goroutines etc)
* Run locally with no external dependancies
* Query the current metrics with a cli command
* Expose metrics for pull-based monitoring (such as Prometheus)
* Provide metrics for push-based monitoring (such as Prometheus push-gateway and InfluxDB)
* Generate request-level metrics for service handlers (request rate, error rate, latency at a minimum)
* Allow for custom metrics to be propagated by the user


Proposal
--------
* Simple metrics interface with methods that take tag-groups and key/values
    - Counter()
    - Guage()
    - Timing()
* Automatic background ticker which gathers Go runtime metrics on a schedule, and uses the interface's methods to submit them
* Service-wide middleware wrapper on service handlers to instrument per-request metrics (tagged with hander method name):
    - Request rate
    - Error rate
    - Latency
* Prometheus implementation would be a good first choice because this simply exposes an HTTP server which provices the latest set of metrics.
    - This requires no external infrastructure
    - Minimal configuration is required (really only the listen port)
    - This _can_ be scraped by an actual Prometheus server (we'll need this to monitor the Micro platform initially)
    - This could power a cli command to query the current set of metrics for a given service instance
    - Prometheus's counter/guage/timing vars could also probably be used to provide metrics for a GDPR GetMetrics() call
* Custom metrics can be sent with the interface methods
