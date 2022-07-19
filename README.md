<a target="_blank" href="https://github.com/hashicorp-services/consul-in-prod"># hashicorp-services/consul-in-prod = Reference Implementation GitBook</a>

## Audience

This book provides to <strong>customer-facing implementers</strong> instructions and automation to create -- for <strong>production</strong> usage -- secure and highly-available implementations of <strong>HashiCorp Consul</strong> common within Global 2000 enterprises.
Specifically:

   * HashiCorp Presales Solution Engineering (Kyle Rarey, Ram Ramhariram, Nathan Pearce)
   * SE SME (Ancil McBarnett)
   * HashiCorp Field CTOs (Jake Lundberg)
   * HashiCorp PS (Professional Services)
   * HashiCorp Implementation Services: (Austin Workman, )
   * HashiCorp CS SEs (Customer Service Engineers)
   * Consultants within HashiCorp services partners ()
   <br /><br />

This is being collaboratively developed and maintained by the above plus these stakeholders: TODO: Get the org names correct!
   * Domain Architecture (Wilson Mar, Frank Hane)
   * PSE (Iman, John Boero, Jim Sullivan)
   * Education (Tu Nguyen)
   * Operations Experience
   * (Josh Wolfer)
   * (Tony Pulickal)
   * (Matt Peters)
   * Reference Architecture (Chloe)
   * SE (segment leaders in US, EMEA, APJ)
   * Field (Thomas Kula)
   * CSA
   * CSM
   * IS
   * Consul Marketing PMM (Van Phan)
   * Consul Product Management (Usha Kodali, Abhishek Tiwari)
   * Dev Evangelists (Rosemary Wang)
   <br /><br />

QUESTION: Who should be included?

QUESTION: Who grants access to services partners to this org/repo on GitBook?

<hr />

## About Consul

Consul is designed to be <a href="#Clouds">multi-cloud</a>, <a href="#OS-Platforms">multi-platform</a>, for <a href="#RefArch">multiple application types</a>.

Terminology:

Datacenter : This is defined by your design. It should ideally be a set of nodes that have low latency connections between them and are part of a private network.

Agent: Any Consul process is an agent. It can run in one of two modes - server and client.

Client : A stateless Consul process that accepts queries from applications and forwards them to server nodes.

<strong>Consul servers</strong> are the components that do the heavy lifting. They store information about services and key/value information. An odd number of servers is necessary to avoid stalemate issues during elections. A Consul process maintains cluster state, responds to RPC queries from clients, elects leaders using the Raft consensus protocol and participates in WAN gossip between datacenters. 

For a quorum to be healthy it is necessary for every node to know its neighbors. 
The list of neighbors is kept in the <strong>Raft peers list</strong> in file <tt>consul-data-directory/raft/peers.json</tt>.

```
root@server1:~# cat /var/consul/raft/peers.json
["192.168.1.12:8300","192.168.1.11:8300","192.168.1.13:8300"]
```

There should be at least 2 nodes there in order to form a quorum and elect a leader on startup.

The “peer set” is the set of all members participating in log replication. 
For Consul's purposes, all server nodes are in the peer set of the local datacenter.

Also in the data dictionary are serf/local.snapshot & serf/remote.snapshot. 
Serf contains the list of connected servers and clients like alive/not-alive state.

Outage recovery might sometimes involve manual editing of the peers list where the machines are not recoverable.


<a name="Clouds"></a>

## Clouds

This presents procedures and automation for creating Consul within each cloud:
1. <strong>Google (GCP)</strong> - HashiCorp's hands-on Instruqt labs run on GCP. So production scripts may leverage scripts to install assets.

1. <strong>AWS</strong>

1. <strong>Azure</strong>
<br /><br />

NOTE: We aim to structure our implementation scripts to make it easier to customize across different clouds.



<a name="OS-Platforms"></a>

## OS Platforms

Each implementation has an edition/variation for each technical platform:
* Linux (Ubuntu RedHat)
* Windows
* MacOS
<br /><br />


<a name="RefArch"></a>

## Production Reference Architecture Scale, Scope, and Objectives

<a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/example-scenario/infrastructure/iaas-high-availability-disaster-recovery"><img alt="Azure sample" align="right" width="200" src="https://docs.microsoft.com/en-us/azure/architecture/example-scenario/infrastructure/media/ha-decision-tree.png"></a>
NOTE: This document does not cover setting up of a single stand-alone Consul cluster for purpose of demonstration.

To ensure production-level reliability at Enterpise scale, each implementation here is based on <a target="_blank" href="https://learn.hashicorp.com/tutorials/consul/reference-architecture">, which consists of:
<a target="_blank" href="https://learn.hashicorp.com/tutorials/consul/kubernetes-reference-architecture">for Kubernetes</a>.
It consists of:

* <a target="_blank" href="https://res.cloudinary.com/dcajqrroq/image/upload/v1655690643/vault-multi-region-map-1298x728_yjgvcv.png"><img align="right" alt="multi-region" width="200" src="https://res.cloudinary.com/dcajqrroq/image/upload/v1655690643/vault-multi-region-map-1298x728_yjgvcv.png"></a>Two regions <a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/guide/security/access-azure-kubernetes-service-cluster-api-server">peered</a> together, with 5 nodes per datacenter across 3 Availability Zones (each a separate VPC).

   Consul can be installed into several platforms:

   * Inside AWS using Terraform</br />
   https://github.com/hashicorp-services/ansible-role-consul/tree/aworkman_testing

   * On-prem using Ansible Playbooks<br />
   https://github.com/hashicorp-services/ansible-role-consul/tree/aworkman_testing

   * Inside Kubernetes
   <br /><br />

References:
   * https://github.com/hashicorp/engineering-docs/tree/main/consul (private)
   <br /><br />

* <a target="_blank" href="https://res.cloudinary.com/dcajqrroq/image/upload/v1658153413/app-east-west-968x897_nspfgj.png"><img align="right" alt="app layers" width="200" src="https://res.cloudinary.com/dcajqrroq/image/upload/v1658153413/app-east-west-968x897_nspfgj.png"></a> Although Consul works with multiple platform technologies, a Linux-based sample e-commerce application (HashiCups?) running in Kubernetes with a server node for each of these APIs:
   * front-end web server
   * product
   * shipment
   * payment (external)
   <br /><br />

   (Not included are Redis cache, Elastisearch (parse logs), mail/SMS, ratings, observability, analytics, etc.)<a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/ecommerce-website-running-in-secured-ase">*</a> <a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/scalable-ecommerce-web-app">*</a>

   TODO: Create a diagram to add in the <a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/browse/">Azure Reference Architecture diagrams</a> with <a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/example-scenario/infrastructure/multi-tier-app-disaster-recovery">HA/DR</a>

* Alternative HA-capable apps:

   <a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/example-scenario/magento/magento-azure">Magento (e-commerce) in AKS</a>?

   https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/app-service-web-app/multi-region

   https://docs.microsoft.com/en-us/azure/architecture/example-scenario/infrastructure/multi-tier-app-disaster-recovery-experiment

   https://docs.microsoft.com/en-us/azure/architecture/example-scenario/infrastructure/wordpress

<a target="_blank" href="https://res.cloudinary.com/dcajqrroq/image/upload/v1652637715/consul-svc-regis-1584x1552_jnnu9g.png"><img align="right" alt="Registry" width="200" src="https://res.cloudinary.com/dcajqrroq/image/upload/v1652637715/consul-svc-regis-1584x1552_jnnu9g.png"></a>
As each service is instantiated, its configuration enables Consul to detect it and add it to its Service Registry, and securely route network traffic to them, even across disparate platforms (via a Consul API Gateway).

<a name="Implementations"></a>

## Global App Network Mesh Implementations

NOTE: We avoid the use of the industry term "Service Mesh" to avoid competitive comparisons with competitors because that's a subset of what HashiCorp offers.

<a target="_blank" href="https://res.cloudinary.com/dcajqrroq/image/upload/v1652200423/consult-multi-envoy-1734x972_ymgi7l.png"><img align="right" alt="consul with envoy" width="200" src="https://res.cloudinary.com/dcajqrroq/image/upload/v1652200423/consult-multi-envoy-1734x972_ymgi7l.png"></a>
In each app server node, a Consul sidecar enables a <strong>Consul Global Mesh</strong> which directs L4 (network level 4) traffic across multiple clouds and platform technologies:
<ul>
A. Among app nodes within the same app cluster<br />
B. Database (MySQL, PostgreSQL, Oracle) outside Kubernetes<br />
C. AWS EC2 image running in AWS<br />
D. AWS ECS (Elastic Container Service) <a target="_blank" href="https://www.hashicorp.com/events/webinars/how-comcast-runs-consul-service-mesh-on-amazon-ecs">VID</a><br />
E. AWS EKS (Elastic Kubernetes Service) with Service Mesh <br />
F. VMWare registry image of hypervisor communicating with Kubernetes<br />
G. Pivotal Cloud Foundary<br />
H. RedHat OpenShift<br />
I. Serverless (AWS Lambda, Azure & GCP Functions) running within clouds<br />
</ul>


<a name="DevSecOps"></a>

## DevSecOps Workflow

Our objective here that, after initial install and configuration, requires minimal or no manual intervention to keep running.

<a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/devsecops-infrastructure-as-code"><img align="right" alt="Azure CI/CD" width="200" src="https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/media/devsecops-for-iac.png"></a> 
To save time, enable collaboration, ensure repeatability, and avoid mistakes, the <a name="ConsulFeatures">Consul features above</a> are instantiated using <strong>modern DevSecOps principles</strong>:

* Self-serve Shift-left for developer efficiency (TFSec on desktops)
* Version control code with code reviews (in GitHub)
* <a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/devsecops-in-github"><img alt="GitHub on Azure" align="right" width="200" src="https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/media/devsecops-in-github-data-flow.png"></a>
Automated CI/CD (GitHub Actions) for speed and comprehensive security scanning. TODO: Adadpt example from Azure:

* Infrastructure-as-Code HashiCorp <a target="_blank" href="https://www.youtube.com/watch?v=uBfUN6PemIk" title="Provision to Production with Terraform Enterprise">Terraform Enterprise Workspaces</a> invoked by Bash shell scripts. 

* Dockerized (Docker Compose) from a Docker Registry
* Automated policy enforcement to replace long waits for manual approvals

* Helm charts
* <a target="_blank" href="https://aws.amazon.com/solutions/implementations/aws-landing-zone/">AWS Landing Zones</a>  <a target="_blank" href="https://learn.hashicorp.com/tutorials/terraform/aws-control-tower-aft">using Terraform</a> (<a target="_blank" href="https://www.youtube.com/watch?v=PE1bhqS8BI8" title="by Welly Siauw & David Wright Mar 23, 2022">VIDEO</a>)
* Azure/Terraform-provider-azapi <a target="_blank" href="https://www.youtube.com/watch?v=6DLyQNJWpgA" title="Key Considerations for Getting HashiCorp Terraform into Production by Iman and Chris from Microsoft May 24, 2022">VIDEO</a>
<br /><br />


<a name="Implementations"></a>

## Implementions

Among the many variations, here are the priorities for development:

1. GCP - Ubuntu - core Kubernetes

   * https://github.com/hashicorp/field-workshops-consul by Thomas Kula (PreSales Solutions Engineering) has slides for aws, azure, gcp, multi-cloud. Has an instructor guide to Instruqt tracks.

   * https://github.com/hashicorp/learn-instruqt contains source files for interactive scenarios at https://learn.hashicorp.com
   * https://github.com/hashicorp-services/enablement-consul-instruqt
   * https://github.com/hashicorp-services/enablement-vault-instruqt
   <br /><br />

1. AWS

   * https://github.com/hashicorp-services/accelerator-aws-consul/ (internal) by Kyle Rarey (Implementation Services)
   * https://github.com/hashicorp/terraform-aws-consul-starter 
   <br /><br />

1. Azure

   * https://github.com/hashicorp/terraform-azure-consul-ent-starter
   <br /><br />


<hr />

<a name="ConsulFeatures"></a>

## HashiCorp products and features used

<a target="_blank" href="https://res.cloudinary.com/dcajqrroq/image/upload/v1653578783/consul-tf-vault-1162x809_lypmvs.png"><img align="right" alt="consul tf vault" width="200" src="https://res.cloudinary.com/dcajqrroq/image/upload/v1653578783/consul-tf-vault-1162x809_lypmvs.png"></a>
Each <a href="#Implementations">implementation above</a> makes use of the Enterprise edition of Consul and Vault to satisfy Zero-Trust and other requirements:

   1. <strong>Centralized identity-based authentication</strong> for users and service accounts (with SSO and MFA using Okta OIDC IdP and other Authentication Methods)
   1. <strong>Centralized secrets</strong> management (using Vault)
   1. <strong>Encryption</strong> of app data in transit and at rest (using Vault)
   
   1. Segregation of app data into different <strong>data Namespaces</strong> to reduce access to data in case of breach (which we assume will occur)
   1. <strong>Admin partitions</strong> in a centralized service registry in the Consul Key/Value store - (this is the core feature desired by most enterprises)

   1. <strong>Segmentation of network</strong> traffic to restrict network access in case of breach (which we assume will occur)
   1. <strong>Layer 7 Traffic Managment</strong> for Canary testing, A/B tests, blue/green deploys, and soft multi-tenancy (instead of "East/West" Load Balancers among microservices)
   1. Use of Access Control Lists (<strong>ACLs</strong> tokens) to enforce least-privilege access
   
   1. <strong>Read replicas</strong> to ensure performance and reliability as systems scale
   1. <strong>Automated backups</strong> to ensure quick and reliable recovery in case of disaster at app, cloud Availability Zone, and cloud region levels (for MTTR to meet RPO and RTO standards)
   1. <strong>Autopilot Enterprise Redundancy Zones</strong> to improve resiliency and scaling by adding “non-voting” servers which will be promoted to voting status in case of voting server failure.
   1. <strong>Automated upgrades</strong> of versions
   <br /><br />

<hr />

## Additional Services/Utilities

For a production-level systems in enterprises:

* Among https://www.consul.io/docs/download-tools

   * https://github.com/gliderlabs/registrator = automatically registers and deregisters services for any Docker container by inspecting containers as they come online. See https://imaginea.gitbooks.io/consul-devops-handbook/content/registrator_deployment.html
   <br /><br />

* Log collection and analytics, such as 
   * Prometheus
   * ServiceNow
   * Elasticache
   * Datadog
   * New Relic
   * etc.
   <br /><br />

* Dashboard run analytics - on Azure:<br /><a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/example-scenario/logging/unified-logging"><img alt="Dashboard in Azure" width="1681" src="https://res.cloudinary.com/dcajqrroq/image/upload/v1658173196/azure-log-dashboard-1681x942_n1pjer.png"></a>

<hr />

<a name="ConstructionStages"></a>

## Construction Stages

<strong>Manual steps to create</strong> each <a href="#Implementations">implementation</a>, explained like the <a target="_blank" href="https://imaginea.gitbooks.io/consul-devops-handbook/content/deployment_strategy.html">Imagina GitBook</a>) are logically organized into these sequential stages (using automated means where applicable):

   1. <a href="SolutionDesign.md">Design solution settings</a> - decide on values for variables (), abbreviations (such as <a target="_blank" href="https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations">Azure's</a>)

   1. <strong>Laptop setup</strong> - on each builder laptop: XCode, Homebrew, wget, tree, Jinja, VSCode, Git, GPG, Vault, Consul, Terraform, Packer, Docker, Docker Compose, etc.
   1. <strong>GitHub setup</strong> - with SSH and GPG certificates
   1. <strong>Clone GitHub template repos</strong> - each of the Reference Implementation components

   1. <strong>Establish Auth Method</strong> - for SSO and MFA by each user on Okta, etc.
   1. <strong>Define least-privilege Roles</strong> - the actions allowed/disallowed for each persona (in role files)
   1. <strong>Establish cloud accounts</strong> - with special Administrator access used during setup and regular accounts. This being Enterprise, we assume use of <a target="_blank" href="https://www.youtube.com/watch?v=_VsEa5H3Jz0">multiple cloud accounts</a>.
   1. <strong>Establish cloud admin account</strong> for managing backup data (assuming breach of other accounts)
   
   1. <strong>Obtain Enterprise license</strong> - from a HashiCorp employee (Solution Engineer, etc.)
   1. <strong>Customize settings</strong> - names for each datacenter, region, etc.

      Run GitHub Actions or CircleCI CI/CD which invoke Bash shell scripts and Terraform to create folders, etc.:

   1. <strong>Run bootstraping scripts</strong> to run security scans (secret detection, TFSec, etc.), create folders, bootstrap, configure to reboot automatically, etc.
   1. <strong>Establish Vault</strong> - 
   1. <strong>Establish apps and database</strong>
   1. <strong>Establish Consul</strong> with segrated namespaces, segmented networks, read replicas, automated backup, etc. per <a href="#ConsulFeatures">Features listed above</a>.
   
   1. <strong>Define Intentions and ACLs</strong> - using Consul to manage the sample application

   1. <a href="#Proving"><strong>Prove</strong> - that production-grade mechanisms can actually respond effectively to various operational and security stresses</a>
   1. Maintain system

TODO: Create a flowchart such as <a target="_blank" href="https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/cicd-for-azure-vms">this</a>


<a name="Proving"></a>

## Proof of Production Viability

<a target="_blank" href="https://res.cloudinary.com/dcajqrroq/image/upload/v1652579825/consul-promote-in-az-550x342_a87mri.png"><img align="right" alt="promotion" width="200" src="https://res.cloudinary.com/dcajqrroq/image/upload/v1652579825/consul-promote-in-az-550x342_a87mri.png"></a>
We also provide procedures and automation to <strong>prove</strong> that production-grade mechanisms can actually respond effectively to various stresses (planned and unplanned outages):

   1. ACL blocks access to those not authorized
   1. As each new service comes online, Consul "discovers" them
   1. Nodes auto-starts and re-joins its cluster when appropriate
   1. When a service becomes unhealthy, Consul no longer routes traffic to it

   1. When ACL and Intentions are changed, access is immediately restricted or allowed
   1. Data associated with each change is propagated (via Serf Gossip protocol) among nodes
   1. Each change that occurs in Consul results in notification of network management systems (Palo Alto via NIA CTS)

   1. When a Follower node is no longer functional, others continue working
   1. When the Leader node is no longer functional, the Raft protocol establishes a new Leader
   1. Additional cloud resources are added or removed as demand rises or drops significantly

   1. Logs can be filtered for troubleshooting (using Datadog and other observability tools)
   1. Dashboards (using Grafana) display analytics about key trends    
   1. Alerts are issued when trouble is recognized (due to injection of troubling events)
   <br /><br />

   <a target="_blank" href="https://www.youtube.com/watch?v=CPI6W3LK0-g">VIDEO</a>

<a target="_blank" href="https://res.cloudinary.com/dcajqrroq/image/upload/v1656899266/zerotrust-maturity-22-06-1456x1326_x0xvl6.png"><img align="right" alt="Zero Trust Maturity" width="200" src="https://res.cloudinary.com/dcajqrroq/image/upload/v1656899266/zerotrust-maturity-22-06-1456x1326_x0xvl6.png"></a>Along the way, we point out where each aspect of <a target="_blank" href="https://www.cisa.gov/sites/default/files/publications/CISA%20Zero%20Trust%20Maturity%20Model_Draft.pdf">Zero-Trust Maturity Model</a>, Well-Architected, SOC2, and ISO 27000 frameworks are achieved.

   * <a target="_blank" href="https://docs.google.com/document/d/1zpL5o3uIzCIJbganpZ32CcmDUXBqmImwfa9AVF_uBFo">PRD-EDU-020-Enterprise architecture documentation</a>
   <br /><br />

<hr />

<a name="Resources"></a>

## Workshop Resources

Slide decks from https://hashicorp.github.io/workshops/
and https://github.com/hashicorp/field-workshops-consul

1. <a target="_blank" href="https://docs.google.com/presentation/d/e/2PACX-1vRsbv2Y40V2Eb_7137-jjRbbyWUqdXGrTcxTjBlCdDseX0qbbrfpAobACQZcgcNWbHGdwiI-7xZ-FSk/pub?start=false&loop=false&delayms=3000">Intro to Consul Enterprise</a> - A two hour introductory workshop. (Sales/Consul Team or FM Team)

1. <a target="_blank" href="https://docs.google.com/presentation/d/e/2PACX-1vRm4bc_k1d_ihqCB1WplZD7lNsUkCOtXxP4SlOeZmfpBMwc6lzVrY7aENhlagIqJKYoMTR_Q6TBaMVw/pub?start=false&loop=false&delayms=3000">Life of a Developer with Consul Enterprise</a> - A two hour container based progressive application delivery workshop. (Sales/Consul Team)

1. <a target="_blank" href="https://docs.google.com/presentation/d/e/2PACX-1vQV_YViIEH-jFaInANvBtb4EnRGToU-ce1JR82GYqA68faH9ysQr4WPOVWhNpyigpstRDflXHOtLioy/pub?start=false&loop=false&delayms=3000">Network Infrastructure Automation with Consul Enterprise</a> - A two hour network acceleration workshop with Terraform and Consul-Terraform-Sync. (Sales/Consul Team)

1. <a target="_blank" href="https://docs.google.com/presentation/d/e/2PACX-1vSjSPGg74hbque88aAPgaNnPXKiY6S7KvkbLiH30g0rIUMYyumN8OCJaL3wq_bQasG78dLaUp0tk3JK/pub?start=false&loop=false&delayms=3000">Multi-Cloud Service Networking with Consul Enterprise</a> - An operational <strong>half day</strong> multi-product Zero Trust workshop across AWS, Azure, and GCP. (Sales/Consul Team)

<hr />

## Other Reference Implementations

* https://github.com/mspnp/app-service-environments-ILB-deployments
illustrated/described by https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/enterprise-integration/ase-high-availability-deployment
