---
services: azure-resource-manager, api-management, frontdoor, azure-monitor, virtual-network
author: paolosalvatori
---

# JMeter Distributed Test Harness #

This sample demonstrates how to use [Azure Front Door](https://docs.microsoft.com/azure/frontdoor/front-door-overview) as global load balancer in front of [Azure API Management](https://docs.microsoft.com/en-us/azure/api-management/api-management-key-concepts).

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

## Architecture ##

The following picture shows the architecture and network topology of the sample.

![Architecture](https://raw.githubusercontent.com/paolosalvatori/jmeter-distributed-test-harness/master/images/TestHarness.png)

JMeter master and slave nodes expose a Public IP using the [Java Remote Method Invocation](https://en.wikipedia.org/wiki/Java_remote_method_invocation) communication protocol over the public internet. In order to lock down security, [Network Security Groups](https://docs.microsoft.com/en-us/azure/virtual-network/security-overview) are used to allow inbound traffic on the TCP ports used by JMeter on master and slave nodes only from the virtual machines that are part of the topology. 

 The following picture shows the Network Security Group of the master node. 

![Master NSG](https://raw.githubusercontent.com/paolosalvatori/jmeter-distributed-test-harness/master/images/MasterNSG.png)

At point 1 you can note that the access via RDP is allowed only from a given public IP. You can restrict the RDP access to master and slave nodes by specifing a public IP as value of the **allowedAddress** parameter in the **azuredeploy.parameters.json** file.
At point 2 and 3 you can see that the access to ports **1099** and **4000-4002** used by JMeter on the master node is restricted to the public IPs of the slave nodes.

 The following picture shows the Network Security Group of the slave node. 

![Slave NSG](https://raw.githubusercontent.com/paolosalvatori/jmeter-distributed-test-harness/master/images/SlaveNSG.png)

At point 1 you can note that the access via RDP is allowed only from a given public IP. You can restrict the RDP access to master and slave nodes by specifing a public IP as value of the **allowedAddress** parameter in the **azuredeploy.parameters.json** file.
At point 2 and 3 you can see that the access to ports **1099** and **4000-4002** used by JMeter on the master node is restricted to the public IPs of the master node.
 
You can connect to master and slave nodes via RDP on port 3389. In addition, you can connect to the JMeter master virtual machine via [Azure Bastion](https://docs.microsoft.com/en-us/azure/bastion/bastion-overview) which provides secure and seamless RDP/SSH connectivity to your virtual machines directly in the Azure portal over SSL. You can customize the ARM template to disable the access to virtual machines via RDP by eliminating the corresponding rule in the Network Security Groups or you can eliminate Azure Bastion if you don't want to use this access type. 
 
A [Custom Script Extension for Windows](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows) downloads and executes a PowerShell script that performs the following tasks:

- Automatically installs Apache JMeter on both the master and slave nodes via [Chocolatey](https://chocolatey.org/packages/jmeter) .  - Customizes the JMeter properties file to disable RMI over SSL and set 4000 TCP port for client/server communications. 
- Downloads the [JMeter Backend Listener for Application Insights](https://github.com/adrianmo/jmeter-backend-azure) that can be used to send test results to Azure Application Insights.
- Creates inbound rules in the Windows Firewall to allow traffic on ports 1099 and 4000-4002.
- Creates a Windows Task on slave nodes to launch JMeter Server at the startup.
- Automatically starts Jmeter Server on slave nodes.

In addition, all the virtual machines in the topology are configured to collect diagnostics logs, Windows Logs, and performance counters to a Log Analytics workspace. The workspace makes use of the following solutions to keep track of the health of the virtual machines:

- [Agent Health](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/solution-agenthealth)
- [Service Map](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/service-map)
- [Infrastructure Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/vminsights-enable-overview)

# Deployment #
You can use the **azuredeploy.json** ARM template and parameters file included in this repository to deploy the JMeter test harness. Make sure to edit the **azuredeploy.parameters.json** file to customize the installation. In particular, you can customize the list of virtual machines by editing the **virtualMachines** parameter in the parameters file. You can also use the **deploy.sh** Bash script to deploy the ARM template.

# Testing #
You can connect to the JMeter master node with the credentials specified in the ARM template to run tests using the JMeter UI or command-line tool. For more information on how to run tests on remote nodes, see: 

- [Apache JMeter Distributed Testing Step-by-step](https://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html)
- [Jmeter Remote Testing](https://jmeter.apache.org/usermanual/remote-test.html)

You can also use the **run.ps1** PowerShell script to run tests on the master node or remote nodes. The script allows to specify the thread number, warmup time, and duration of the test. In order to use this data as parameters, the JMeter test file (.jmx) needs to use define corresponding parameters. As a sample, see the **bing-test.jmx** JMeter test in this repository.

![Run SCript](https://raw.githubusercontent.com/paolosalvatori/jmeter-distributed-test-harness/master/images/RunScript.png)

This script allows to save JMeter logs, results and dashboard on the local file system.

![JMeter Dashboard](https://raw.githubusercontent.com/paolosalvatori/jmeter-distributed-test-harness/master/images/Dashboard.png)

You can use **Windows PowerShell** or **Windows PowerShell ISE** to run commands. For example, the following command:

```powershell
.\run.ps1 -JMeterTest .\bing-test.jmx -Duration 60 -WarmupTime 30 -NumThreads 30 -Remote "191.233.25.31, 40.74.104.255, 52.155.176.185"
```

generates the following JMeter command:

```batch
C:\ProgramData\chocolatey\lib\jmeter\tools\apache-jmeter-5.2.1\bin\jmeter -n -t ".\bing-test.jmx" -l "C:\tests\test_runs\bing-test_1912332531_4074104255_52155176185\test_20200325_052525\results\resultfile.jtl" -e -o "C:\tests\test_runs\bing-test_1912332531_4074104255_52155176185\test_20200325_052525\output" -j "C:\tests\test_runs\bing-test_1912332531_4074104255_52155176185\test_20200325_052525\logs\jmeter.jtl" -Jmode=Stand
ard -Gnum_threads=30 -Gramp_time=30 -Gduration=60 -Djava.rmi.server.hostname=51.124.79.211 -R "191.233.25.31, 40.74.104.255, 52.155.176.185"
```

# Possible Developments #
This solution uses Public IPs to let master and slave nodes to communicate with each other. An alternative solution could be deploying master and slave nodes in different virtual networks located in different regions and use [global virtual network peering](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview) to connect these virtual networks. Using this approach, the master node could communicate with slave nodes via private IP addresses.

This topology has the following advantages over a topology that uses private IP addresses:

- Reduced complexity 
- Lower total cost of ownership (TCO)
- Extensibility to other cloud platforms.

As an example of extensibility, you can provision slave nodes across multiple regions, across multiple Azure subscriptions and even on other cloud platforms like AWS or GCP. I personally tested this possibility by provisioning additional slave nodes on AWS.

Last but not least, the ARM template can be easily changed to replace **virtual machines** with **virtual machine scale sets** (VMSS).
