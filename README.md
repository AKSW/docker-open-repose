# repose-compose
Example setup for simple repose proxy. Limiting request rate to an nginx server based on user IP addresses.

# Main Repose Configuration
Repose main configuration file is [system-model.cfg.xml](./etc/repose/system-model.cfg.xml). The important configuration options are:
* Repose is listening on port 80:
```
<node id="repose_node1" hostname="localhost" http-port="80"/>
```
* There are two filters active:
```
<filter name="ip-user"/>
<filter name="rate-limiting"/>
```
***rate-limiting*** filter requires ***ip-user*** filter to inject X-PP-User HTTP header based on user IP address.
* The default proxy destination is hostname nginx:
```
<endpoint id="service" protocol="http" hostname="nginx" root-path="/" port="80" default="true"/>
```
The hostname nginx should match the service name in docker-compose.yml definition and be in the same docker network as repose service.

# Configuring user IP ranges
The IP ranges configuration is located in [ip-user.cfg.xml](./etc/repose/ip-user.cfg.xml). The default configuration defines one user group ***match-all*** as follows:
```
<group name="match-all">
    <cidr-ip>0.0.0.0/0</cidr-ip>
    <cidr-ip>0::0/0</cidr-ip>
</group>
```

# Rate Limiting for IP ranges
The rate limiting configuration is located in [rate-limiting.cfg.xml](./etc/repose/rate-limiting.cfg.xml). It uses the groups of users you defined in the IP ranges configuration previously (e.g. ***match-all*** group). In the rate limiting you can define the global limit for all the users:
```
<!-- The global limit for all the users! -->
<global-limit-group>
    <limit id="global" uri="*" uri-regex=".*" value="1000" unit="MINUTE"/>
</global-limit-group>
```
This will limit the whole amount of request to 1000 per minute! If this amount of requests is exceeded, no more requests will be possible!

To limit requests for a particular group of users (i.e. ***match-all*** group) you need to define a limit-group:
```
<limit-group id="limited" groups="match-all" default="true">
    <limit id="all" uri="*" uri-regex="/.*" http-methods="POST PUT GET DELETE" unit="MINUTE" value="10"/>
</limit-group>
```
This will limit requests for any given IP address within previously specified IP range (any) to 10 per one minute.

# Additional Information
See [examples](./etc/repose/examples) folder for available filter configurations. Also, refer to official [Repose docs](https://repose.atlassian.net/wiki/).

# Author/Maintainer
* [Ivan Ermilov](http://aksw.org/IvanErmilov)
