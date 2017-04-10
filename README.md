# Inputs
Using docker, setup three instances that communicate to each other.

* a docker instance running rsyslog (centralized logging server)
* a docker instance running rsyslog which forwards to the centralized logging server (logging agent)
* a docker instance running nginx serving static content (app server)

Ensure that the app server's access logs are forwarded to the centralized logging server via the logging agent.

The ideal solution here should be re-usable. In production we would want to run one logging agent and many app server instances together on any host. This would provide us with an easy way to ship logs for each container from each docker host to the centralized logging service.

# Requirements
1. Include the ability to easily configure the address of the centralized logging server (it could change so just re-configuring the logging agent should be easy).
2. Be able to easily choose what files we want to ship to the logging service.

# Running the solution
Currently 3 services are merged into one yaml for convenience. It has comments about how it would be split in a real-world scenario.

1. Start: `docker-compose -f rsyslog.yml up -d`
2. Generate web logs: `curl localhost:8080`
3. Observe log server logs: `docker logs centralizedcontainerlogging_log-server_1`
4. Observe locally saved logs: `cat ./logs/*`
5. Stop `docker-compose -f rsyslog.yml down`

# Adressing Requirements
1. Log server is linked via [docker built-in service discovery](https://docs.docker.com/docker-cloud/apps/service-links/), e.g. referred by
name "log-server".
2. Logspout used as a log-agent is only capturing container's stdout/stderr, which
I believe is correct because a container should run no more than 1 process. Unlike
a "Sidecar" approach you cannot list specific files here. However multiple message-level filters are available:
 * [Logspout filters](https://github.com/gliderlabs/logspout#including-specific-containers) (which I used to select \_web\_)
 * [Logspout ignore](https://github.com/gliderlabs/logspout#ignoring-specific-containers)
 * [Rsyslog selectors]( http://www.rsyslog.com/doc/v8-stable/configuration/filters.html#selectors)


 # Extra credit
Centralized logging is available out-of-box in kubernetes: [fluentd + elasticsearch + grafana/kibana](https://deis.com/blog/2016/kubernetes-logging-with-elasticsearch-and-kibana/)
