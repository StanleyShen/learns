Consul
===============
Consul is a tool for service discovery and configuration. Consul is distributed, highly available and extremely scalable.

Counsul provides serveral key features:

- **Service Discovery** - Consul makes it simple for services to register themselves and to discover other services via a DNS or HTTP interface. External services such as SaaS providers can be registered as well.
- **Health Checking** - Health Checking enables Consul to quickly alert operators about any issues in a cluster. The intergration with service discovery prevents routing traffic to unhealthy hosts and enables service level circuit breakers.
- **Key/Value Storage** - A flexible key/value store enables storing dynamic configuration, feature flagging, coordination,  leader election and more. The simple HTTP API make it easy to use anywhere.
-- **Multi-Datacenter** - A Consul is built to be datacenter aware, and can support any number of regions without complex configuration.

Quick Start
---------
[https://www.consul.io/intro/getting-started/install.html](https://www.consul.io/intro/getting-started/install.html)

Documentation
-------
[https://www.consul.io/docs](https://www.consul.io/docs)

Source Code
------------
[https://github.com/hashicorp/consul](https://github.com/hashicorp/consul)


Loggregator - logging in the Clouds
=============================
Loggregator is the user application logging subsystem of Cloud Foundry.
[https://github.com/cloudfoundry/loggregator](https://github.com/cloudfoundry/loggregator)

Loggregator allows users to:
- Tail their application logs.
- Dump a recent set of application logs (where recent is a configurable number of log packets).
- Continually drain their application logs to 3rd party log archive and analysis services.
- (Operators and administrators only) Access the firehose, which includes the combined stream of logs from all apps, plus metrics data from CF components.


Usage
------
First, make sure you are using golang based CF CLI.
cf logs APP_NAME [ --recent ]

Constraints
------------------
- Loggregator collects STDOUT & STDERR from applications. This may require configuration on the developer's side.
	- Warning: the DEA logging agent expects an application to have open connections on both STDOUT and STDERR. Closing either of these (for example, by redirecting output to /dev/null) will be read by the logging agent as a misbehaving application, and it will disconnect from all sockets for that app.
- A Loggregator outage must not affect the running application.
- Loggregator gathers and stores logs in a best-effort manner. While undesirable, losing the current buffer of application logs is acceptable.
- The 3rd party drain API should mimic Heroku's in order to reduce integration effort for our partners. The Heroku drain API is simply remote syslog over TCP.

Architecture
--------------------------
Loggregator is composed of:

- Sources: Logging agents that run on the Cloud Foundry components.
- Metron: Metron agents are co-located with sources. They collect logs and forward them to:
- Doppler: Responsible for gathering logs from the Metron agents, storing them in temporary buffers, and forwarding logs to 3rd party syslog drains.
- Traffic Controller: Handles client requests for logs. Gathers and collates messages from all Doppler servers, and provides external API and message translation (as needed for legacy APIs).

