# hashicorp-services/consul-in-prod

This book provides instructions to create -- for <strong>production</strong> usage -- 
the most common use of <strong>HashiCorp Consul</strong> within a Global 2000 enterprise.

QUESTION: Do services partners get specific access to this?

Consul is designed to be multi-cloud, multi-platform, for <a href="#RefArch">multiple application types</a>.

<a name="RefArch"></a>

## Reference Architecture Scale

To ensure production-level reliability at Enterpise scale, each datacenter implementation here is based on HashiCorp's Reference Architecture, which consists of:

   * 5 nodes per datacenter
   * a datacenter in 3 availability zones
   * network peering between two regions
   * a real-world sample e-commerce application with microservices front-end, back-end business logic APIs: users, product, inventory, shipment, payment, mail/SMS, ratings, observability, analytics, etc.) on multiple platforms (Linux, Windows, MacOS).
   <br /><br />


## Clouds

This presents procedures and automation for creating Consul within AWS, Azure, and Google clouds.


<a name="Implementations"></a>

## Global Network Mesh Implementations

Consul is designed to automatically detect resources, then securely route network traffic to them, even across disparate platforms (via a Consul API Gateway). In addition to a single base Consul datacenter implementation, several Consul datacenters can communicate as a single Consul Global Mesh across multiple clouds containing these technologies:

   A. Database (MySQL, PostgreSQL, Oracle) outside Kubernetes<br />
   B. AWS EC2 image running in AWS<br />
   C. AWS ECS (Elastic Container Service)<br />
   D. AWS EKS (Elastic Kubernetes Service) with Service Mesh <br />
   E. VMWare hypervisor communicating with Kubernetes<br />
   F. Pivotal Cloud Foundary<br />
   G. RedHat OpenShift<br />
   H. Serverless (AWS Lambda, Azure & GCP Functions) running within clouds<br />
</ul>

<a name="Requirements"></a>

## Requirements

Each <a href="#Implementations">implementation above</a> makes use of Enterprise-grade Consul and Vault to satisfy Zero-Trust and other requirements:

   1. <strong>Centralized identity-based authentication</strong> for users and service accounts (with SSO and MFA using Okta OIDC IdP and other Authentication Methods)
   1. <strong>Centralized secrets</strong> management (using Vault)
   1. <strong>Centralized service registry</strong> (Key/Value store) 
   
   1. <strong>Encryption</strong> of app data in transit and at rest (using Vault)
   1. Segregation of app data into different <strong>data Namespaces</strong> to reduce access to data in case of breach (which we assume will occur)

   1. <strong>Segmentation of network</strong> traffic to restrict network access in case of breach (which we assume will occur)
   1. <strong>Layer 7 Traffic Managment</strong> for Canary testing, A/B tests, blue/green deploys, and soft multi-tenancy (instead of "East/West" Load Balancers among microservices)
   1. Use of Access Control Lists (<strong>ACLs</strong> tokens) to enforce least-privilege access
   
   1. <strong>Read replicas</strong> to ensure performance and reliability as systems scale
   1. <strong>Automated backups</strong> to ensure quick and reliable recovery in case of disaster at app, cloud Availability Zone, and cloud region levels (for MTTR to meet RPO and RTO standards)
   1. <strong>Autopilot Enterprise Redundancy Zones</strong> to improve resiliency and scaling by adding “non-voting” servers which will be promoted to voting status in case of voting server failure.
   1. <strong>Automated upgrades</strong> of versions
   <br /><br />

<a name="DevSecOps"></a>

## DevSecOps

To save time and eliminate mistakes, the <a name="Requirements">requirements above</a> are built into HashiCorp Terraform and other automation assets used to create Consul with Vault, while adopting <strong>modern DevSecOps principles</strong>:

   * Infrastructure-as-Code for repeatability with version control (in GitHub)
   * Automated CI/CD (GitHub Actions) for speed and comprehensive security scanning
   * Self-serve Shift-left for developer efficiency
   * Automated policy enforcement to replace long waits for manual approvals
   <br /><br />

<a name="Proving"></a>

## Proof of Production Viability

We also provide procedures and automation to <strong>prove</strong> that production-grade mechanisms can actually respond effectively to various stresses:

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

Along the way, we point out where each aspect of <a target="_blank" href="https://www.cisa.gov/sites/default/files/publications/CISA%20Zero%20Trust%20Maturity%20Model_Draft.pdf">Zero-Trust Maturity Model</a>, Well-Architected, SOC2, and ISO 27000 frameworks are achieved.

<hr />

<a name="ConstructionStages"></a>

## Construction Stages

<strong>Construction</strong> consists of the following logical steps (all using automated means where applicable):

   1. <strong>Laptop setup</strong> - on each builder laptop: Git, Vault, Consul, Terraform, GPG, and Linux utilities
   1. <strong>GitHub account setup</strong> - with SSH and GPG certificates
   1. <strong>Clone GitHub repos</strong> - each of the Reference Architecture components

   1. <strong>Establish Auth Method</strong> - for SSO and MFA by each user on Okta, etc.
   1. <strong>Define least-privileges</strong> - the actions allowed/disallowed for each persona (in role files)
   1. <strong>Establish cloud accounts</strong> - with special Administrator access used during setup and regular accounts
   1. <strong>Establish cloud account</strong> for managing backup data (assuming breach of other accounts)

   1. <strong>Customize settings</strong> - names for each datacenter, region, etc.
   1. <strong>Run GitHub Actions</strong> - using scripts for CI/CD, with security scans (secret detection, TFSec, etc.)
   1. <strong>Establish Vault</strong> - using CI/CD invoking Terraform
   1. <strong>Establish apps and database</strong> - using CI/CD invoking Terraform
   1. <strong>Establish Consul</strong> - using CI/CD (with segrated namespaces, segmented networks, read replicas, automated backup, etc.)
   
   1. <strong>Define Intentions and ACLs</strong> - using Consul to manage the sample application

   1. <a href="#Proving"><strong>Prove</strong> - that production-grade mechanisms can actually respond effectively to various stresses</a>

<hr />

<a name="Resources"></a>

## Workshop Resources

Slide decks from https://hashicorp.github.io/workshops/

1. <a target="_blank" href="https://docs.google.com/presentation/d/e/2PACX-1vRsbv2Y40V2Eb_7137-jjRbbyWUqdXGrTcxTjBlCdDseX0qbbrfpAobACQZcgcNWbHGdwiI-7xZ-FSk/pub?start=false&loop=false&delayms=3000">Intro to Consul Enterprise</a> - A two hour introductory workshop. (Sales/Consul Team or FM Team)

1. <a target="_blank" href="https://docs.google.com/presentation/d/e/2PACX-1vRm4bc_k1d_ihqCB1WplZD7lNsUkCOtXxP4SlOeZmfpBMwc6lzVrY7aENhlagIqJKYoMTR_Q6TBaMVw/pub?start=false&loop=false&delayms=3000">Life of a Developer with Consul Enterprise</a> - A two hour container based progressive application delivery workshop. (Sales/Consul Team)

1. <a target="_blank" href="https://docs.google.com/presentation/d/e/2PACX-1vQV_YViIEH-jFaInANvBtb4EnRGToU-ce1JR82GYqA68faH9ysQr4WPOVWhNpyigpstRDflXHOtLioy/pub?start=false&loop=false&delayms=3000">Network Infrastructure Automation with Consul Enterprise</a> - A two hour network acceleration workshop with Terraform and Consul-Terraform-Sync. (Sales/Consul Team)

1. <a target="_blank" href="https://docs.google.com/presentation/d/e/2PACX-1vSjSPGg74hbque88aAPgaNnPXKiY6S7KvkbLiH30g0rIUMYyumN8OCJaL3wq_bQasG78dLaUp0tk3JK/pub?start=false&loop=false&delayms=3000">Multi-Cloud Service Networking with Consul Enterprise</a> - An operational <strong>half day</strong> multi-product Zero Trust workshop across AWS, Azure, and GCP. (Sales/Consul Team)

