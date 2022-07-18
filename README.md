# consul-in-prod
Procedures and assets to create Enterprise-grade Consul &amp;amp; Vault in production

This book provides instructions to create -- for <strong>production</strong> usage -- 
the most common use of <strong>HashiCorp Consul</strong> within a Global 2000 enterprise.

Consul is designed to be multi-cloud, multi-platform, for multiple applications.

So this presents procedures and automation for creating Consul within AWS, Azure, and Google clouds.

<a name="RefArch"></a>

To ensure production-level reliability at Enterpise scale, each implementation is based on HashiCorp's Reference Architecture, which consists of:

   * 5 nodes per datacenter
   * a datacenter in 3 availability zones
   * network peering between two regions
   * a real-world sample application with microservices (front-end, back-end business logic, inventory, shipment, payment, mail/SMS, etc.)
   <br /><br />

<a name="Implementations"></a>

Consul is designed to automatically detect resources, then securely route network traffic to them.
So this book provides procedures and automation that brings up apps and databases running in each of these platforms:

   A. Serverless (AWS Lambda, Azure & GCP Functions) running within clouds<br />
   B. VMWare<br />
   C. Pivotal Cloud Foundary<br />
   D. RedHat OpenShift<br />
   E. Google Kubernetes with Service Mesh<br />
   <br /><br />

<a name="Requirements"></a>

Each <a href="#Implementations">implementation above</a> makes use of Enterprise-grade Consul and Vault to satisfy Zero-Trust and other requirements:

   1. <strong>Centralized identity-based authentication</strong> for users and service accounts (with SSO and MFA using Okta)
   1. <strong>Centralized secrets</strong> management (using Vault)
   1. <strong>Encryption</strong> of data in transit and at rest (using Vault)
   1. Segregation of data into different <strong>data Namespaces</strong> to reduce access to data in case of breach (which we assume will occur)
   1. <strong>Segmentation of network</strong> traffic to restrict network access in case of breach (which we assume will occur)
   1. Use of Access Control Lists (<strong>ACLs</strong>) to enforce least-privilege access
   1. <strong>Read replicas</strong> to ensure performance and reliability as systems scale
   1. <strong>Automated backups</strong> to ensure quick and reliable recovery in case of disaster at app, cloud Availability Zone, and cloud region levels (for MTTR to meet RPO and RTO standards)
   1. <strong>Autopilot Enterprise Redundancy Zones</strong> to improve resiliency and scaling by adding “non-voting” servers which will be promoted to voting status in case of voting server failure.
   1. <strong>Automated upgrades</strong> of versions
   <br /><br />

<a name="DevSecOps"></a>

To save time and eliminate mistakes, the <a name="Requirements">requirements above</a> are built into HashiCorp Terraform and other automation assets used to create Consul with Vault, while adopting <strong>modern DevSecOps principles</strong>:

   * Infrastructure-as-Code for repeatability with version control (in GitHub)
   * Automated CI/CD (GitHub Actions) for speed and comprehensive security scanning
   * Self-serve Shift-left for developer efficiency
   * Automated policy enforcement to replace long waits for manual approvals
   <br /><br />

<a name="Proving"></a>

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

<strong>Construction</strong> consists of the following logical steps (all using automated means where applicable):

   1. <strong>Laptop setup</strong> - on each builder laptop: Git, Vault, Consul, Terraform, GPG, and Linux utilities
   1. <strong>GitHub account setup</strong> - with SSH and GPG certificates
   1. <strong>Clone GitHub repos</strong> - each of the Reference Architecture components

   1. <strong>Establish Okta</strong> - for SSO and MFA by each user
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
