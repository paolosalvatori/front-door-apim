---
services: azure-resource-manager, api-management, frontdoor, azure-monitor, virtual-network
author: paolosalvatori
---

# Introduction #

This sample demonstrates how to use [Azure Front Door](https://docs.microsoft.com/azure/frontdoor/front-door-overview) as global load balancer in front of [Azure API Management](https://docs.microsoft.com/en-us/azure/api-management/api-management-key-concepts).

## Architecture ##

The following picture shows the architecture and network topology of the sample.

![Architecture](https://raw.githubusercontent.com/paolosalvatori/front-door-apim/master/images/architecture.png)

## Azure API Management ##

API Management helps organizations publish APIs to external, partner, and internal developers to unlock the potential of their data and services. Azure API Management provides the following capabilities:

- Routing Rules
- Local or external caching of response messages to improve performance
- Protect backend API with rate and throttling limits
- Protect a Web API backend using OAuth 2.0 with Azure Active Directory
- Secure access to the backend service of an API using certificates
- Monitoring by sending request and response messages to an Event Hub
- Monitoring requests via Application Insights 
- Change request/response messages via policies

For more information, see [Azure API Management](https://docs.microsoft.com/en-us/azure/api-management/api-management-key-concepts).

## Azure Front Door ##

Azure Front Door is a global HTTP\HTTPS load balancer that works at layer 7 provides. Front Door terminates HTTPS requests at the edge of Microsoft’s network and actively probes to detect application or infrastructure health or latency changes. Front Door then always routes traffic to the fastest available backend. Refer to Front Door's routing architecture details and traffic routing methods to learn more about the service. When using Front Door, once a packet enters the Azure global WAN, the request is sent over extremely low latency connection between any two points. This speed cannot be matched on the public Internet where there would be many hops and much higher latency. Deployed to the edge of Microsoft’s global network, Azure Front Door exploits the same points of presence (POPs) of the Azure CDN and provides web and mobile applications, APIs, and/or cloud services with always-on reliability, high performance, easy scalability and simplified connectivity. Like Traffic Manager, Front Door is resilient to failures, including the failure of an entire Azure region. Azure Front Door provides the following functionalities:

- Application acceleration with anycast and using Microsoft’s massive private global network to directly connect to your Azure deployed backends means your app runs with lower latency and higher throughput to your end users.
- HTTP load balancing enables to host and operate applications resiliently across multiple regions, fail-over instantly and offer users an “always-on” web site / mobile app availability experience.
- SSL offload at a massive scale enables you to maintain security and scale to a rapidly growing or expanding user base, all while reducing latency.
- Path based routing powers global microservice applications with independent routing all under a single global domain.
- A single pane of glass to monitor and gain insight into user’s traffic and distributed backend service’s health.
- Health Probes: in order to determine the health of each backend, each Front Door environment periodically sends a synthetic HTTP/HTTPS request to each of your configured backends. Front Door then uses responses from these probes to determine the "best" backends to which it should route real client requests.
- WAF at the edge provides application security against DDoS attacks or malicious users at the edge providing protection at scale without sacrificing on performance.
- Caching: Azure Front Door delivers large files without a cap on file size. Azure Front Door is able to cache and deliver large files in chunks of 8 MB. In addition, Azure Front Door can dynamically compress content on the edge, resulting in a smaller and faster response to your clients.
- URL Rewrite allows to copy any part of the incoming path that matches to a wildcard path to the forwarded path.
- IPv6, custom SSL certificates, rate limiting, geo-filtering, etc.

For more information, see [Azure Front Door](https://docs.microsoft.com/azure/frontdoor/front-door-overview)


