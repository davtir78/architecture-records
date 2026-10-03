---
template: reference-architecture
owner: "David Tirabassi, Enterprise/Solution Architect"
status: endorsed
version: "1.1"
date: 2026-09-22
applies-to: "Any organisation designing or tailoring an integration platform. The site's integration patterns and decision framework are built on this architecture."
---

# Integration Reference Architecture

## Overview

This reference architecture defines a comprehensive Integration Platform designed to establish an infrastructure that facilitates the seamless connection between data and service producers and the corresponding consumers. It outlines the essential capabilities required to manage APIs, route messages, transform data, and orchestrate services, wrapped in robust governance, security, and operational pipelines.

The model is organised around the path an integration takes: **consumers** on one side, **producers** on the other, and the **integration layer** that connects them. Five cross-cutting layers (information, governance, security, management and monitoring, and development and testing tools) apply across all three.

### Scope

* **In scope:** the capabilities of an integration platform (API management, message routing and transformation, batch and change-data movement, file transfer, event streaming and workflow) and the governance, security, monitoring and delivery tooling around them.
* **Out of scope:** processing of event streams, which is a data-platform concern (see the [Data Platform Reference Architecture](https://www.itarchitecturepatterns.net/reference-architectures/data-platform-reference-architecture)), and the choice of products, which each organisation makes when it tailors this architecture to itself.

### Principles

| Principle | What it means here |
| :--- | :--- |
| Capabilities, not products | Each component is described by what it does. Products appear only in the examples column, so the model stays valid when a product changes. |
| Decouple producers from consumers | The integration layer sits between them, so neither needs to know the other's technology, location or timing. |
| Reuse before build | Governance components (the service catalogue, the API developer portal) exist so that an existing service is found before a new integration is built. |
| Secure by boundary | The trust boundary an integration crosses (internal, external or cloud) decides the controls it carries, and is asked before the technology is chosen. |

## Component Model Layers

A layer groups components that serve the same purpose. Each component is a capability, and the examples column shows products for the main cloud platforms and the open-source options.

### 1. Consumers

Represents the various systems, platforms, and clients that consume and exchange data or services via the Integration Platform.

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **Internal IT Systems** | Internal operational systems and services residing within the primary enterprise corporate network. | Internal App Integration (AWS); Private Virtual Network (Azure); Private VPC Access (GCP); Core APIs (Open Source) |
| **External IT Systems** | Systems belonging to the organization but situated outside the local high-security network perimeter. | AWS Client VPN (AWS); Azure Bastion / VPN (Azure); Google Cloud VPN (GCP); OpenVPN (Open Source) |
| **External Partners** | Third-party organizations, vendors, and business partners exchanging business documents or services safely. | AWS Transfer Family (AWS); Azure B2B Connectivity (Azure); Analytics Hub (GCP); AS2 / SFTP Protocols (Open Source) |
| **Web Sites** | Internal or external web applications that query transactional endpoints or API gateways. | CloudFront CDN (AWS); Azure Front Door (Azure); GCP Cloud CDN (GCP); Next.js Web Apps (Open Source) |
| **Mobile Devices** | Native iOS or Android applications consuming secure organizational REST or GraphQL services. | AWS AppSync (AWS); Azure Mobile Apps (Azure); Firebase Mobile SDK (GCP); React Native Integration (Open Source) |
| **Field Devices** | Remote sensors, automated machines, or telemetry endpoints sending data streams from the field. | AWS IoT Core (AWS); Azure IoT Hub (Azure); GCP IoT Core (Partner) (GCP); MQTT Broker (Open Source) |

### 2. Integration Layer

The core functional engine of the platform, composed of specialized components that transform, route, orchestrate, and mediate data flows between disparate systems.

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **API Management** | Create, document, secure, throttle, and govern APIs. Exposes digital assets cleanly to internal and external developers. | Amazon API Gateway (AWS); Azure API Management (Azure); Google Apigee (GCP); Kong, 3scale (Red Hat), WSO2 (Open Source) |
| **Middleware Services (ESB)** | Enterprise Service Bus providing message brokering, protocol translation, routing, data transformation, service orchestration, and adapter connectivity. | AWS EventBridge / Step Functions (AWS); Azure Service Bus / Logic Apps (Azure); GCP Application Integration (GCP); Red Hat Integration, MuleSoft, WSO2, Camel (Open Source) |
| **ETL / Batch Processing** | Batch data movement extracting data from sources, transforming it into compatible formats, and loading it into targets. | AWS Glue (AWS); Azure Data Factory (Azure); GCP Cloud Data Fusion (GCP); Talend Data Integration, Airbyte (Open Source) |
| **Change Data Capture (CDC)** | Captures and tracks changes made at a data source in real-time or near real-time, ensuring synchronization. | AWS Database Migration Service (DMS) (AWS); Azure SQL CDC / ADF (Azure); GCP Datastream (GCP); Debezium, Fivetran (Open Source) |
| **Master Data Management (MDM)** | Centralized reference repository ensuring critical enterprise data elements remain consistent, synchronized, and high quality. | Amazon Neptune / partner MDM (AWS); CluedIn MDM / Profisee (Azure); GCP Semarchy / partner MDM (GCP); Talend MDM, Pimcore (Open Source) |
| **Secure File Transfer (SFT)** | Robust managed file transfer (MFT) securing files in transit and at rest with comprehensive auditing. | AWS Transfer Family (AWS); Azure Storage (SFTP) (Azure); GCP SFTP Gateway (GCP); rclone, Apache NiFi, SFTP (Open Source) |
| **Event Streaming** | Ingests, durably stores, and delivers high-velocity event streams to independent consumers, decoupling producers from consumers and absorbing load spikes. (Processing those streams is a data-platform concern — see the Kappa pattern.) | Amazon MSK (Kafka) (AWS); Azure Event Hubs (Azure); Google Pub/Sub (GCP); Apache Kafka, Redpanda (Open Source) |
| **Workflow & Orchestration** | Automation of business processes and rules. Coordinates task sequences, escalations, human-in-the-loop steps, and decision tables. | AWS Step Functions (AWS); Azure Logic Apps (Azure); GCP Workflows (GCP); Camunda BPM, jBPM, temporal.io (Open Source) |
| **Artificial Intelligence (AI)** | Empowers the integration layer with intelligent decision-making, running transaction flows through machine learning models for predictive routing. | Amazon SageMaker (AWS); Azure Machine Learning (Azure); Google Vertex AI (GCP); TensorFlow, PyTorch (Open Source) |
| **Native / Connector Integration** | Pre-built, vendor-supplied connectors embedded within enterprise SaaS and COTS platforms, used to exchange data without building or hosting custom integration code. Trades control and portability for speed of delivery. | Amazon AppFlow (AWS); Power Platform Connectors (Azure); Application Integration connectors (GCP); Airbyte, n8n (Open Source) |
| **Manual / Ad-hoc Transfer** | Governed human-operated data movement for infrequent, low-volume, or exception-path transfers where automation cannot be justified. Executed from a hardened, ephemeral desktop with full audit capture — the controlled fallback, not an absence of pattern. | Amazon WorkSpaces + Transfer Family (AWS); Azure Virtual Desktop (Azure); Cloud Workstations (GCP); Apache Guacamole jump host, audited SFTP (Open Source) |

### 3. Producers

Represents the various systems, platforms, and databases that safely receive, produce, and exchange transactional data.

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **Internal IT Systems** | Internal operational systems and services residing within the primary enterprise corporate network. | Internal App Integration (AWS); Private Virtual Network (Azure); Private VPC Access (GCP); Core APIs (Open Source) |
| **External IT Systems** | Systems belonging to the organization but situated outside the local high-security network perimeter. | AWS Client VPN (AWS); Azure Bastion / VPN (Azure); Google Cloud VPN (GCP); OpenVPN (Open Source) |
| **External Partners** | Third-party organizations, vendors, and business partners exchanging business documents or services safely. | AWS Transfer Family (AWS); Azure B2B Connectivity (Azure); Analytics Hub (GCP); AS2 / SFTP Protocols (Open Source) |
| **Web Sites** | Internal or external web applications that query transactional endpoints or API gateways. | CloudFront CDN (AWS); Azure Front Door (Azure); GCP Cloud CDN (GCP); Next.js Web Apps (Open Source) |
| **Mobile Devices** | Native iOS or Android applications consuming secure organizational REST or GraphQL services. | AWS AppSync (AWS); Azure Mobile Apps (Azure); Firebase Mobile SDK (GCP); React Native Integration (Open Source) |
| **Field Devices** | Remote sensors, automated machines, or telemetry endpoints sending data streams from the field. | AWS IoT Core (AWS); Azure IoT Hub (Azure); GCP IoT Core (Partner) (GCP); MQTT Broker (Open Source) |

## Cross-Cutting Layers

Capabilities that apply across the consumers, integration and producers layers above.

### Information Layer

Provides a structured framework for data models and centralized vocabulary to ensure consistent semantic structures across services.

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **Data Definition & Modelling** | Defines reusable schemas and objects utilized in service definitions and database schemas. | AWS Glue Schema Registry (AWS); Azure Schema Registry (Azure); GCP Schema Registry (GCP); Apicurio Registry, Avro / JSON Schema (Open Source) |
| **Common Vocabulary** | Centralized repository for common business objects and fields, allowing domain teams to selectively define integration contract content. | AWS Glue Data Catalog (AWS); Azure Purview Data Catalog (Azure); GCP Dataplex Catalog (GCP); Backstage.io Catalog, Confluent Schema Registry (Open Source) |

### Governance Layer

Monitors and directs integration asset lifecycles, guaranteeing cataloging, versioning, reuse, and compliance across developer teams.

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **API Developer Portal** | A central web-based hub where internal or external developers can discover, explore, test, and access APIs. Includes documentation, SDKs, and analytics. | Amazon API Gateway Developer Portal (AWS); Azure API Management Developer Portal (Azure); Apigee Integrated Developer Portal (GCP); Backstage, Kong Developer Portal (Open Source) |
| **Service Catalogue** | Centralized registry cataloging all deployed APIs, microservices, structures, dependencies, versions, and deprecations. | AWS Glue Catalog (AWS); Azure Purview (Azure); GCP Analytics Hub (GCP); Backstage.io, Apicurio Registry (Open Source) |
| **Service Registry** | A runtime registry that manages active services, their endpoints, states, metadata, and dynamic routing configurations. | AWS Cloud Map (AWS); Azure Resource Graph (Azure); GCP Service Directory (GCP); Eureka, Consul (Open Source) |

### Security Layer

Active enforcement controls distributed across the platform to safeguard infrastructure, verify identity, manage keys, and shield transactions.

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **Authentication & Authorisation** | Verifies user/client identity (OAuth2, OIDC) and executes granular, role-based access control (RBAC) to APIs and admin functions. | Amazon Cognito / IAM (AWS); Azure Entra ID (Active Directory) (Azure); Google Identity Platform (GCP); Keycloak, Authelia (Open Source) |
| **Data Security** | Enforces data encryption at rest and in transit, tokenization, data masking, and sensitive data protection policies. | AWS KMS / Macie (AWS); Azure Information Protection / SQL Encryption (Azure); GCP Cloud KMS / DLP API (GCP); Apache Ranger, HashiCorp Vault (Open Source) |
| **Secrets Management** | Securely stores, encrypts, and rotates sensitive credentials, connection strings, certificates, and API tokens. | AWS Secrets Manager / SSM (AWS); Azure Key Vault (Azure); GCP Secret Manager (GCP); HashiCorp Vault (Open Source) |
| **Transport Security** | Establishes secure, encrypted point-to-point data transmission channels using mutual TLS (mTLS) and SSL. | AWS Certificate Manager (ACM) (AWS); Azure App Gateway TLS (Azure); GCP Load Balancing TLS (GCP); NGINX mTLS, Linkerd Service Mesh (Open Source) |
| **Policy Enforcement** | Active edge protection enforcing rate limiting, IP whitelists, request sanitization, and SQL injection shielding. | AWS WAF / Shield (AWS); Azure WAF (Azure); Google Cloud Armor (GCP); Kong Plugins, Open Policy Agent (OPA) (Open Source) |

### Management and Monitoring

Monitors availability, logs traces, triggers automated alerting, handles software exceptions, and administers batch jobs.

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **Service Management** | Lifecycle management controls executing service deployments, scaling, version rollbacks, and active containers management. | AWS Systems Manager / ECS (AWS); Azure Automation / Container Apps (Azure); GCP Cloud Run / GKE (GCP); Kubernetes, Ansible (Open Source) |
| **Metrics Monitoring** | Aggregates and tracks real-time platform performance metrics (latency, CPU, traffic) to trigger alerts for out-of-threshold metrics. | Amazon CloudWatch (AWS); Azure Monitor (Azure); GCP Cloud Operations (GCP); Prometheus, Grafana (Open Source) |
| **Logging & Auditing** | Collects, structures, and stores log files from all systems for compliance audit trails, threat detection, and search. | AWS CloudTrail / CloudWatch Logs (AWS); Azure Activity Log / Log Analytics (Azure); GCP Cloud Logging (GCP); ELK Stack (Elasticsearch), Grafana Loki (Open Source) |
| **Error Handling** | Standardizes platform exceptions, redirects failures to Dead Letter Queues (DLQ), and alerts operations for manual resolution. | AWS SQS DLQ (AWS); Azure Service Bus DLQ (Azure); GCP Pub/Sub Dead Lettering (GCP); Apache Camel Error Handler (Open Source) |
| **Job Scheduling** | Governs and schedules background scripts, automated tasks, and batch ETL jobs to run on cron intervals. | AWS EventBridge Scheduler / Batch (AWS); Azure Automation / Event Grid (Azure); GCP Cloud Scheduler (GCP); Apache Airflow, temporal.io (Open Source) |

### Development and Testing Tools

Capabilities that empower integration developers to model, write, test, package, and dynamically deploy integration services.

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **Integrated Development Environment** | Local or hosted environments loaded with extensions to construct integration flows, mappings, and schemas. | AWS Cloud9 (AWS); VS Code Online (Azure); GCP Cloud Workstations (GCP); VS Code, IntelliJ IDEA, Eclipse (Open Source) |
| **Testing Tools** | Automates API contract testing, unit mockings, and volumetric load testing to guarantee quality prior to deployment. | AWS Device Farm (AWS); Azure DevTest Labs (Azure); GCP Firebase Test Lab (GCP); Postman, SoapUI, Apache JMeter, Pact (Open Source) |
| **CI / CD** | Automates packaging, static analysis scanning, testing, and multi-environment deployment pipelines. | AWS CodePipeline (AWS); Azure DevOps Pipelines (Azure); GCP Cloud Build (GCP); GitLab CI, GitHub Actions, Jenkins (Open Source) |
| **Configuration Management & Automation** | Ensures declarative Infrastructure-as-code (IaC) and system configurations are tracked in source control and deployed predictably. | AWS CloudFormation / CDK (AWS); Azure ARM Templates / Bicep (Azure); GCP Deployment Manager (GCP); Terraform, Ansible, Git (Open Source) |

## From Architecture to Patterns

The architecture names the capabilities; the pattern library shows how each is built. Every component in the integration layer that has patterns has one for each trust boundary, and the [Integration Patterns and Decision Framework](https://www.itarchitecturepatterns.net/articles/integration-patterns-and-decision-framework) chooses between them.

| COMPONENT | PATTERNS | WHAT THEY SHOW |
| :--- | :--- | :--- |
| **API Management** | [Integration API Management (Internal)](https://www.itarchitecturepatterns.net/patterns/int-api-internal) · [Integration API Management (External)](https://www.itarchitecturepatterns.net/patterns/int-api-external) · [Integration API Management (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-api-cloud) | Internal, external and cloud variants. Create, document, secure, throttle, and govern APIs. |
| **Middleware Services (ESB)** | [Integration Middleware Services (Internal)](https://www.itarchitecturepatterns.net/patterns/int-middleware-internal) · [Integration Middleware Services (External)](https://www.itarchitecturepatterns.net/patterns/int-middleware-external) · [Integration Middleware Services (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-middleware-cloud) | Internal, external and cloud variants. Enterprise Service Bus providing message brokering, protocol translation, routing, data transformation, service orchestration, and adapter connectivity. |
| **Event Streaming** | [Integration Event Streaming (Internal)](https://www.itarchitecturepatterns.net/patterns/int-esp-internal) · [Integration Event Streaming (External)](https://www.itarchitecturepatterns.net/patterns/int-esp-external) · [Integration Event Streaming (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-esp-cloud) | Internal, external and cloud variants. Ingests, durably stores, and delivers high-velocity event streams to independent consumers, decoupling producers from consumers and absorbing load spikes. |
| **ETL / Batch Processing** | [Integration ETL / Batch Processing (Internal)](https://www.itarchitecturepatterns.net/patterns/int-etl-internal) · [Integration ETL / Batch Processing (External)](https://www.itarchitecturepatterns.net/patterns/int-etl-external) · [Integration ETL / Batch Processing (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-etl-cloud) | Internal, external and cloud variants. Batch data movement extracting data from sources, transforming it into compatible formats, and loading it into targets. |
| **Secure File Transfer (SFT)** | [Integration Secure File Transfer (Internal)](https://www.itarchitecturepatterns.net/patterns/int-filetransfer-internal) · [Integration Secure File Transfer (External)](https://www.itarchitecturepatterns.net/patterns/int-filetransfer-external) · [Integration Secure File Transfer (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-filetransfer-cloud) | Internal, external and cloud variants. Robust managed file transfer (MFT) securing files in transit and at rest with comprehensive auditing. |
| **Change Data Capture (CDC)** | [Integration Change Data Capture (Internal)](https://www.itarchitecturepatterns.net/patterns/int-cdc-internal) · [Integration Change Data Capture (External)](https://www.itarchitecturepatterns.net/patterns/int-cdc-external) · [Integration Change Data Capture (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-cdc-cloud) | Internal, external and cloud variants. Captures and tracks changes made at a data source in real-time or near real-time, ensuring synchronization. |
| **Master Data Management (MDM)** | [Integration Master Data Management (Internal)](https://www.itarchitecturepatterns.net/patterns/int-mdm-internal) · [Integration Master Data Management (External)](https://www.itarchitecturepatterns.net/patterns/int-mdm-external) · [Integration Master Data Management (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-mdm-cloud) | Internal, external and cloud variants. Centralized reference repository ensuring critical enterprise data elements remain consistent, synchronized, and high quality. |
| **Workflow & Orchestration** | [Integration Workflow & Orchestration (Internal)](https://www.itarchitecturepatterns.net/patterns/int-workflow-internal) · [Integration Workflow & Orchestration (External)](https://www.itarchitecturepatterns.net/patterns/int-workflow-external) · [Integration Workflow & Orchestration (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-workflow-cloud) | Internal, external and cloud variants. Automation of business processes and rules. |
| **Artificial Intelligence (AI)** | [Integration AI / ML Services (Internal)](https://www.itarchitecturepatterns.net/patterns/int-aiml-internal) · [Integration AI / ML Services (External)](https://www.itarchitecturepatterns.net/patterns/int-aiml-external) · [Integration AI / ML Services (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-aiml-cloud) | Internal, external and cloud variants. Empowers the integration layer with intelligent decision-making, running transaction flows through machine learning models for predictive routing. |
| **Native / Connector Integration** | [Integration Native Connectors (Internal)](https://www.itarchitecturepatterns.net/patterns/int-native-internal) · [Integration Native Connectors (External)](https://www.itarchitecturepatterns.net/patterns/int-native-external) · [Integration Native Connectors (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-native-cloud) | Internal, external and cloud variants. Pre-built, vendor-supplied connectors embedded within enterprise SaaS and COTS platforms, used to exchange data without building or hosting custom integration code. |
| **Manual / Ad-hoc Transfer** | [Integration Manual / Ad-hoc Transfer (Internal)](https://www.itarchitecturepatterns.net/patterns/int-manual-internal) · [Integration Manual / Ad-hoc Transfer (External)](https://www.itarchitecturepatterns.net/patterns/int-manual-external) · [Integration Manual / Ad-hoc Transfer (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-manual-cloud) | Internal, external and cloud variants. Governed human-operated data movement for infrequent, low-volume, or exception-path transfers where automation cannot be justified. |

## Key Decisions

| Decision | Status | Record |
| :--- | :--- | :--- |
| Choose a pattern from the trust boundary and the platform component, not from a product | accepted | [Integration Patterns and Decision Framework](https://www.itarchitecturepatterns.net/articles/integration-patterns-and-decision-framework) |
| Name every pattern from this architecture's own component names | accepted | [The pattern library, named from this architecture's components](https://www.itarchitecturepatterns.net/patterns) |
| Treat stream processing as a data-platform concern; the integration layer carries and delivers events | accepted | [Data Platform Reference Architecture](https://www.itarchitecturepatterns.net/reference-architectures/data-platform-reference-architecture) |

## Roadmap and Lifecycle

Not applicable here: this reference model is vendor-neutral, so it doesn't classify products. An organisation adopting it records, for each component, the product it has chosen and whether that product is strategic, emerging, tactical, contained or being decommissioned.

## Change History

| Version | Date | Author | Change |
| :--- | :--- | :--- | :--- |
| 1.0 | 2026-05-31 | David Tirabassi | Published on this site |
| 1.1 | 2026-09-22 | David Tirabassi | Component tables regenerated from a single data source, so component names match the pattern library's titles |
