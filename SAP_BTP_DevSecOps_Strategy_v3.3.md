# DevSecOps Strategy
SAP Business Technology Platform Applications
Azure DevOps & SAP Cloud ALM Integration
DRAFTDocument Version: 3.3Date: November 2026Author: Rodion RODIONOVOrganization: SAP SE – Cross-Solution Delivery, Integrated ToolchainClassification: SAP Internal
# Document Control
## Revision History
| Version | Date | Author | Changes |
| --- | --- | --- | --- |
| 0.1 | October 2026 | R. Rodionov | Initial draft |
| 1.0 | November 2026 | Harshit Madania | Stakeholder review comments |
| 3.2 | November 2026 | R. Rodionov | Added: BAS→S/4HANA PCE deployment, deployment roles matrix, SIEM integration, SSAM MDK clarification |

## Reviewers & Approvers
| Role | Name | Status | Date |
| --- | --- | --- | --- |
| Solution Architect | [TBD] | Pending |  |
| Security Lead | [TBD] | Pending |  |
| DevOps Lead | [TBD] | Pending |  |
| Project Manager | [TBD] | Pending |  |

## Distribution
| Audience | Access Level |
| --- | --- |
| Project Delivery Team | Full |
| Customer IT Operations | Full |
| SAP Security & Compliance | Full |
| Executive Sponsors | Summary only |

Table of Contents
DevSecOps Strategy1
Document Control2
Revision History2
Reviewers & Approvers2
Distribution3
1. Executive Summary7
2. Scope & Objectives9
2.1 In Scope9
2.2 Out of Scope9
2.3 Objectives10
3. BTP Landscape Architecture11
3.1 Subaccount Topology11
3.2 Core Platform Services11
SAP Cloud Identity Services (CIS)12
SAP Cloud Transport Management Service (CTMS)12
SAP Cloud ALM12
4. Source Code Management – Git in Azure DevOps13
4.1 Repository Strategy13
4.2 Branching Model13
4.3 Code Review & Pull Request Policies14
5. Application Pipeline Strategy15
5.1 Git-Supported Applications15
5.1.1 SSAM – Mobile Development (SAP Mobile Services / MDK)15
5.1.2 SAP Fiori Applications18
5.1.3 SAP Integration Services (SAP Integration Suite)18
5.2 Non-Git Applications21
5.2.1 SAP Build Process Automation (BPA)21
5.2.2 SAP Build Work Zone21
5.2.3 SAP Datasphere / Data Cloud22
5.3 S/4HANA PCE Extensions (from BAS)23
5.3.1 Architecture Overview23
5.3.2 Development Scenarios24
5.3.3 gCTS Integration (Git-enabled CTS)24
5.3.4 Fiori Deployment to S/4HANA BSP Repository25
5.3.5 Prerequisites for BAS-to-S/4HANA Connectivity25
6. CI/CD Pipeline Design28
6.1 Pipeline Architecture28
6.2 Build & Quality Gates28
6.3 Security Scanning in Pipeline29
7. Deployment Strategy30
7.1 SAP Cloud Transport Management Service (CTMS)30
Transport Nodes30
7.2 SAP Cloud ALM Integration30
Cloud ALM Deployment Logging30
7.3 Transport Routes31
7.4 Deployment Roles & Authorization Matrix32
7.4.1 BTP Application Deployments (via CTMS + Cloud ALM)32
7.4.2 S/4HANA PCE Deployments (via CTS / gCTS + Cloud ALM)33
7.4.3 Segregation of Duties Summary34
8. Security Strategy36
8.1 SAP Cloud Identity Services (CIS)36
8.2 Secrets & Credential Management36
8.3 Static Application Security Testing (SAST)37
8.4 Dependency & Vulnerability Scanning (SCA)37
9. Monitoring, Logging & Alerting38
9.1 SAP Cloud ALM – Deployment Logging38
Logging Matrix38
9.2 CI/CD Build Failure Logging38
9.3 SAP Alert Notification Service (ANS) for BTP39
Alert Configuration39
9.4 SIEM Integration40
9.4.1 Log Sources & SIEM Feeds40
9.4.2 SIEM Correlation Rules41
9.4.3 Supported SIEM Platforms42
10. Automation & Orchestration44
10.1 SAP Automation Pilot44
Key Use Cases44
10.2 Deployment Tracking & Self-Healing45
11. Governance & Compliance46
11.1 Change Management Governance46
11.2 Audit Trail46
11.3 Compliance Policies47
12. Roles & Responsibilities48
13. Roadmap & Next Steps49
Immediate Next Steps50
14. Appendix51
A. Glossary51
B. Reference Architecture Diagram52
C. Pipeline YAML Template (Azure DevOps)53
D. CTMS Configuration Reference53
E. SIEM Configuration Reference53

# 1. Executive Summary
This document defines the DevSecOps strategy for managing the full lifecycle of SAP Business Technology Platform (BTP) applications. It addresses source code management, CI/CD pipelines, deployment orchestration, security, monitoring, and automation across two distinct application categories:
Git-supported applications – SSAM (Mobile), Fiori Apps, and Integration Services – that leverage Azure DevOps Git repositories with CI/CD pipelines producing MTAR archives for deployment and Cloud ALM.
Non-Git applications – SAP Build Process Automation (BPA), SAP Build Work Zone, and SAP Data Cloud – that rely on native content transport via CTMS and Cloud ALM without source code version control.
The strategy integrates the following SAP and Microsoft platform services into a cohesive DevSecOps toolchain:
| Service | Role in Toolchain |
| --- | --- |
| Azure DevOps (Git) | Source code management, branching, pull requests |
| Azure DevOps Pipelines / SAP CIS | Build, test, security scan, MTAR packaging |
| SAP Cloud Transport Management (CTMS) | Transport orchestration across DEV → QAS → PROD |
| SAP Cloud ALM | Change & deployment management, monitoring, logging |
| SAP Cloud Identity Services (CIS) | Identity federation, SSO, role-based access |
| SAP Alert Notification Service | Real-time alerting for BTP platform and application events |
| SAP Automation Pilot | Deployment tracking, automated remediation |

Key Principle: Security is embedded at every stage – from code commit (pre-commit hooks, branch policies) through build (SAST, dependency scanning) to deployment (transport approval workflows, audit logging). This is DevSecOps, not Dev-then-Sec-then-Ops.
# 2. Scope & Objectives
## 2.1 In Scope
| Area | Details |
| --- | --- |
| Applications | SSAM (Mobile), Fiori Apps, Integration Services, BPA, Work Zone, Data Cloud |
| Source Control | Azure DevOps Git repositories |
| CI/CD | Azure DevOps Pipelines and/or SAP Continuous Integration & Delivery service |
| Transport & Deployment | SAP CTMS, SAP Cloud ALM Features/Deployment Management |
| Security | SAST, SCA, secrets management, identity governance (CIS) |
| Monitoring & Alerting | Cloud ALM operations, Alert Notification Service, Automation Pilot |
| Environments | DEV, QAS, PROD subaccounts on SAP BTP |

## 2.2 Out of Scope

SAP S/4HANA (ABAP part) on-premise or private cloud change management (ChaRM/CTS+)
Non-SAP application pipelines
Azur DevOps pipelines
Infrastructure-as-Code for BTP subaccount provisioning (unless Automation Pilot is extended)
## 2.3 Objectives
Standardize the development-to-production lifecycle for all SAP BTP applications.
Shift security left – embed security checks into every CI/CD pipeline stage.
Ensure traceability – every deployment to QAS/PROD must be traceable to a Cloud ALM feature/change, a Git commit (where applicable), and an approval workflow.
Automate repetitive deployment and monitoring tasks via Automation Pilot.
Centralize logging – all deployment activities and CI/CD outcomes logged in Cloud ALM.
Alert proactively – leverage Alert Notification Service for real-time incident detection.
# 3. BTP Landscape Architecture
## 3.1 Subaccount Topology
The BTP landscape follows a three-tier subaccount model aligned with the standard SAP transport landscape. An additional platform services subaccount hosts shared DevSecOps tooling.
| Subaccount | Purpose | Key Services |
| --- | --- | --- |
| DEV | Development & unit testing. Developers deploy freely. CTMS source node. | SAP Business Application Studio (BAS), application runtimes, service instances |
| QAS | Quality assurance, integration testing, UAT. Deployments only via CTMS transport. | Application runtimes, service instances (cloned from DEV config) |
| PROD | Production. Deployments only via approved CTMS transport from QAS. | Application runtimes, service instances (production-grade sizing) |
| Platform / DevOps | Shared DevSecOps services. Not application-specific. | CTMS, Cloud ALM (if BTP-based), CI/CD service, Alert Notification, Automation Pilot, CIS tenant |

BTP Global Account  ├─ Subaccount: DEV          (development, BAS, runtimes)  ├─ Subaccount: QAS          (quality, testing)  ├─ Subaccount: PROD         (production)  └─ Subaccount: Platform     (CTMS, CIS, ANS, Automation Pilot)
Cloud ALM Global Account

## 3.2 Core Platform Services
### SAP Cloud Identity Services (CIS)
CIS provides the identity layer for the entire BTP landscape:
Identity Authentication (IAS) – SAML/OIDC-based SSO for all BTP applications and Azure DevOps integration.
Identity Provisioning (IPS) – automated user and group provisioning across subaccounts.
Identity Directory – central user store for BTP, federated with corporate IdP (e.g., Azure AD / Entra ID).
### SAP Cloud Transport Management Service (CTMS)
CTMS orchestrates the movement of application content (MTAR files, BPA projects, Work Zone content, Data Cloud artifacts) across the three-tier landscape. Transport nodes map to subaccounts; transport routes enforce the DEV → QAS → PROD flow.
### SAP Cloud ALM
Cloud ALM serves as the central governance and operations hub:
Features / Change Management – tracks requirements and changes linked to deployments.
Deployment Management – approves and triggers CTMS transports, logs outcomes.
Operations – health monitoring, real user monitoring, integration monitoring.
Design Deion: Cloud ALM is subscribed in the Platform subaccount (or as a SaaS subscription at global-account level) to maintain independence from any single application subaccount. CTMS and CIS are similarly centralized.
# 4. Source Code Management – Git in Azure DevOps
## 4.1 Repository Strategy
All Git-supported applications use Azure DevOps Git as the central source code management platform. Repository naming follows a consistent convention:
| Application | Repository Name Pattern | Example |
| --- | --- | --- |
| SSAM Mobile | sap-mobile-{app-name} | sap-mobile-field-service |
| Fiori Apps | sap-fiori-{app-name} | sap-fiori-purchase-orders |
| Integration Flows | sap-integration-{package-name} | sap-integration-order-replication |

## 4.2 Branching Model
A trunk-based development model with short-lived feature branches is recommended:
| Branch | Purpose | Lifetime | Protection |
| --- | --- | --- | --- |
| main | Production-ready code. Triggers deployment to DEV via CI/CD. | Permanent | Branch policy: PR required, min. 1 reviewer, build must pass, linked work item |
| feature/* | Developer feature branches. Created from main. | Short-lived (days) | None (developer workspace) |
| hotfix/* | Emergency production fixes. Created from main. | Short-lived (hours) | PR required, expedited review |
| release/* | Optional: stabilization branch before QAS promotion. | Short-lived (sprint) | PR required, full review |

feature/ABC-123  →  Pull Request  →  main  →  CI/CD Build  →  MTAR  →  CTMS (DEV)  →  Cloud ALM approval  →  CTMS (QAS)  →  CTMS (PROD)
## 4.3 Code Review & Pull Request Policies
Azure DevOps branch policies enforce the following on the main branch:
Minimum reviewers: At least 1 reviewer must approve (2 recommended for production-critical apps).
Build validation: The CI pipeline must pass (build + unit tests + SAST scan) before merge.
Work item linking: Every PR must be linked to an Azure DevOps work item (mapped to a Cloud ALM feature).
Comment resolution: All review comments must be resolved before merge.
No direct pushes: All changes to main must go through a PR.
Security Gate: PRs that fail the SAST scan or introduce critical/high-severity vulnerabilities in dependency scanning are automatically blocked from merging.
# 5. Application Pipeline Strategy
## 5.1 Git-Supported Applications
These applications follow a unified pipeline pattern: developers work in SAP Business Application Studio (BAS), commit to Azure DevOps Git, and CI/CD pipelines produce MTAR archives that are uploaded to CTMS for transport through the landscape.
### 5.1.1 SSAM – Mobile Development (SAP Mobile Services / MDK)
SSAM mobile applications are built using the SAP Mobile Development Kit (MDK). MDK apps are metadata-driven: the developer creates MDK metadata (rules, actions, pages, services) in BAS, and this metadata is deployed to SAP Mobile Services, which interprets it at runtime on the mobile device. MDK apps do not produce a traditional compiled MTAR via the MBT build tool.
#### Transport Mechanism: SAP Content Agent → cTMS
MDK app content is transported via SAP Cloud Transport Management Service (cTMS) using the SAP Content Agent service. This is the official, SAP-documented integration path (see SAP Tutorial: Transport MDK apps using SAP Content Agent UI).
Key Clarification: MDK apps are transported as content packages via SAP Content Agent, not as MTARs built by MBT. The Content Agent UI discovers MDK app content from SAP Mobile Services, packages it as a transport request, and forwards it to cTMS. This is the same pattern used by other BTP content-based services (e.g., Integration Suite, SAP Build Work Zone).
| Aspect | Details |
| --- | --- |
| Development IDE | SAP Business Application Studio (BAS) with MDK extensions |
| Source Control | Azure DevOps Git (MDK project source files) |
| Build Artifact | MDK metadata content package (exported via SAP Content Agent – not an MTAR from MBT) |
| Transport Tool | SAP Content Agent UI → exports MDK content from Mobile Services → creates transport request in cTMS |
| Transport Engine | SAP Cloud Transport Management Service (cTMS) |
| Governance | SAP Cloud ALM – orchestrates approval and deployment of cTMS transports |
| Deployment Path | Mobile Services (DEV) → Content Agent → cTMS [DEV] → CALM Approval → cTMS [QAS] → CALM Approval → cTMS [PROD] |

#### Prerequisites for MDK → cTMS Integration
| Prerequisite | Details |
| --- | --- |
| SAP Content Agent service | Subscribed in the source (DEV) subaccount. Provides the Content Agent UI for content discovery and export. |
| cTMS destination | A destination (e.g., TransportManagementService) configured in the DEV subaccount, pointing to the cTMS instance using the cTMS service key credentials. |
| SAP Mobile Services | MDK app must be deployed to Mobile Services on the DEV subaccount. The Content Agent discovers content from the Mobile Applications > Native/MDK area. |
| cTMS transport landscape | Transport nodes (DEV, QAS, PROD) and routes configured in cTMS, mapped to the respective BTP subaccounts with Mobile Services. |
| Cloud ALM connection | cTMS connected to Cloud ALM for deployment orchestration and approval workflows. |

#### Pipeline Flow
Option A – MDK-Only Apps (Primary Path):
Developer (BAS/MDK) → Git Push to Azure DevOps → CI/CD Pipeline (optional):  1. Checkout MDK project  2. Validate MDK metadata (schema check, rule validation, lint)  3. Static analysis / code review checks  4. Deploy MDK app to SAP Mobile Services (DEV subaccount)Content Agent Transport:  5. Open SAP Content Agent UI in DEV subaccount  6. Select MDK app content from Mobile Services  7. Export → creates transport request in cTMS (DEV node)Governance:  8. Cloud ALM approval → cTMS imports to QAS (Mobile Services on QAS subaccount)  9. Cloud ALM approval → cTMS imports to PROD (Mobile Services on PROD subaccount)
Option B – MDK App + Server-Side Backend (CAP/OData):
If the mobile solution includes a server-side CAP backend or custom OData service in addition to the MDK metadata, two parallel transport streams are required:
Stream 1 – Backend (MTAR via CI/CD):Developer (BAS) → Git Push → Azure DevOps Pipeline:  1. Checkout backend code (CAP/Node.js)  2. npm install, lint, unit tests, SAST, SCA  3. MBT build → MTAR  4. Upload MTAR to cTMS (DEV node)  5. Cloud ALM approval → cTMS to QAS → cTMS to PRODStream 2 – MDK Metadata (Content Agent):Developer (BAS/MDK) → Deploy to Mobile Services (DEV)  1. Content Agent UI → export MDK content → cTMS (DEV node)  2. Cloud ALM approval → cTMS to QAS → cTMS to PRODNote: Both streams should be coordinated under the same Cloud ALM featureto ensure backend + MDK frontend are promoted together.

CI/CD for MDK Content: The CI/CD pipeline can still validate, lint, and test MDK metadata files in the Git repository. However, the transport to cTMS is handled by Content Agent (not by the pipeline directly uploading an MTAR). The pipeline’s role for MDK-only apps is quality assurance; the Content Agent handles the actual transport packaging.

### 5.1.2 SAP Fiori Applications
| Aspect | Details |
| --- | --- |
| Development IDE | SAP Business Application Studio (BAS) with Fiori tools |
| Source Control | Azure DevOps Git |
| CI/CD | Azure DevOps Pipelines (or SAP CI/CD Service) |
| Build Artifact | MTAR (containing UI5 apps, CAP services, etc.) |
| Deployment Path | MTAR → CTMS → Cloud ALM → QAS → PROD |

Pipeline flow:
Developer (BAS) → Git Push → Azure DevOps Pipeline:  1. Checkout code  2. npm install / dependency resolution  3. ESLint + UI5 linter  4. Unit tests (QUnit / OPA5 / Jest)  5. SAST scan  6. Dependency vulnerability scan  7. MBT build → MTAR  8. Upload MTAR to CTMS (DEV node)  9. Notify Cloud ALM→ CTMS transport to QAS → CTMS transport to PROD
### 5.1.3 SAP Integration Services (SAP Integration Suite)
| Aspect | Details |
| --- | --- |
| Development | SAP Integration Suite Design workspace (Cloud Integration, API Management) |
| Source Control | Azure DevOps Git (optional – via Integration Suite Git integration or manual export/import)https://community.sap.com/t5/technology-blog-posts-by-sap/transport-cap-apps-and-integration-artefacts-directly-from-git-with-sap/ba-p/14497880 |
| CI/CD | Optional – Azure DevOps Pipelines or SAP CI/CD Service. Can also use CTMS-only flow. |
| Build Artifact | MTAR (Integration content package) |
| Deployment Path | MTAR → CTMS → Cloud ALM → QAS → PROD |

Note: CI/CD is optional for Integration Services. When CI/CD is not used, the integration developer exports the content package as an MTAR directly from the Integration Suite and uploads it to CTMS. The CTMS + Cloud ALM approval workflow remains the same regardless.
Pipeline flow (with CI/CD):
Developer (Integration Suite) → Git Push (or API export) → Azure DevOps Pipeline:  1. Checkout / download integration package  2. Validate integration flow configuration  3. Security review (credential externalization check)  4. Package as MTAR  5. Upload to CTMS (DEV node)  6. Notify Cloud ALM→ CTMS transport to QAS → CTMS transport to PROD
Pipeline flow (without CI/CD):
Developer (Integration Suite) → Export MTAR → Upload to CTMS (DEV node)→ Cloud ALM approval → CTMS transport to QAS → CTMS transport to PROD
Note: No, not all Integration Suite artifacts can be packaged as MTAR. The transport mechanism depends on the specific capability. Here's the accurate breakdown:
#### Artifact-Level Transport Support Matrix
| Integration Suite Capability | Transportable Artifacts | Content Agent → cTMS (MTAR) | Alternative |
| --- | --- | --- | --- |
| Cloud Integration | Integration packages (iFlows, message mappings, value mappings, scripts, XSLT, Groovy) | Yes — Confirmed. Content Agent packages as MTAR. | CTS+, MTAR Download, Manual Export/Import |
| API Management | API proxies, API products, API providers, KVMs, certificates | Yes — Confirmed. Content Agent assembles API artifacts in MTAR. | MTAR Download, Manual Export/Import |
| Integration Advisor | MAGs, MIGs, custom messages | Partial — typically transported as part of an integration package. Standalone transport needs verification. | Export/Import via Integration Advisor UI |
| Trading Partner Management | TP profiles, agreements, B2B configs | No explicit Content Agent → cTMS docs. | Manual export/import, or as part of Cloud Integration packages |
| Event Mesh | Queues, topic subscriptions, webhooks | No — tenant-specific config, not transported as MTAR. | Manual configuration, IaC/Terraform |
| Open Connectors | Connector instances, formulas | No — own config model, not MTAR-packaged. | Manual config replication, API export |
| Graph | Business data graph configuration | No — tenant-specific. | Manual configuration per tenant |

Key Takeaway: Only Cloud Integration and API Management have confirmed, fully-supported Content Agent → cTMS (MTAR) transport. Other capabilities require alternative approaches. Manual runbook.. These should still be tracked in Cloud ALM features for traceability.
## 5.2 Non-Git Applications
These applications do not use Git-based source control. Content is developed directly in the application and transported via CTMS with Cloud ALM governance. Security controls shift from code-level scanning to configuration governance, role-based access control, and change approval workflows.
### 5.2.1 SAP Build Process Automation (BPA)
| Aspect | Details |
| --- | --- |
| Development | SAP Build Process Automation Lobby (low-code/no-code) |
| Source Control | Not applicable (content managed within BPA) |
| CI/CD | Not applicable |
| Transport Mechanism | CTMS (BPA integrates natively with CTMS for content export/import) |
| Deployment Path | BPA (DEV) → CTMS → Cloud ALM approval → QAS → PROD |

Developer (BPA Lobby, DEV) → Release project → CTMS picks up content→ Cloud ALM approval → CTMS transport to QAS → CTMS transport to PROD
Security controls:
Role-based access in BPA Lobby (developer vs. admin vs. viewer)
Change approval in Cloud ALM before QAS/PROD deployment
Audit logging of all process automation changes
Review of credentials and destinations used in automations
### 5.2.2 SAP Build Work Zone
| Aspect | Details |
| --- | --- |
| Development | SAP Build Work Zone admin console (site configuration, content federation) |
| Source Control | Not applicable |
| CI/CD | Not applicable |
| Transport Mechanism | CTMS (site content transport) |
| Deployment Path | Work Zone (DEV) → CTMS → Cloud ALM approval → QAS → PROD |

Admin (Work Zone, DEV) → Configure site / content → CTMS export→ Cloud ALM approval → CTMS transport to QAS → CTMS transport to PROD
### 5.2.3 SAP Datasphere / Data Cloud
| Aspect | Details |
| --- | --- |
| Development | SAP Datasphere modeler (spaces, views, data flows, analytic models) |
| Source Control | Not applicable (content managed within Datasphere) |
| CI/CD | Not applicable |
| Transport Mechanism | CTMS (Datasphere integrates with CTMS for content transport) |
| Deployment Path | Datasphere (DEV) → CTMS → Cloud ALM approval → QAS → PROD |

Data Modeler (Datasphere, DEV) → Create/modify spaces, views, flows → CTMS export→ Cloud ALM approval → CTMS transport to QAS → CTMS transport to PROD
## 5.3 S/4HANA PCE Extensions (from BAS)
This section describes how code developed in SAP Business Application Studio (BAS) on BTP is deployed to S/4HANA Private Cloud Edition (PCE) systems. S/4HANA PCE is an on-premise-managed system hosted by SAP, and it requires a specific pipeline and transport approach that differs from pure BTP cloud deployments.
### 5.3.1 Architecture Overview
BAS connects to the S/4HANA PCE system via SAP Cloud Connector, which provides a secure tunnel between BTP and the on-premise/PCE network. Developers use BAS with the ABAP Development Tools (ADT) extension or Fiori tools to develop directly against the S/4HANA DEV system.
Architecture:
### 5.3.2 Development Scenarios
| Scenario | Development in BAS | Deployment Target | Transport Mechanism |
| --- | --- | --- | --- |
| ABAP Cloud (RAP/CAP) | BAS with ADT extension. Developer creates ABAP objects (CDS views, behavior definitions, service bindings) in BAS, which activates them directly on ABAP cloud DEV via the ADT backend connection. | ABAP cloud DEV → QAS → PROD | ABAP Transport Requests managed by cloud TMS. Optionally managed via gCTS (Git-enabled Change and Transport System) with Git integration to Azure DevOps. |
| Fiori Elements / Freestyle UI5 | BAS with Fiori tools. UI5 application developed in BAS, deployed to S/4HANA’s BSP repository (ABAP front-end server) or to BTP HTML5 Repository with S/4HANA OData backend. | S/4HANA PCE BSP repo or BTP HTML5 repo | If BSP target: npm run deploy to S/4HANA DEV → ABAP transport to QAS/PROD.If BTP HTML5 target: MTAR → CTMS → CALM → QAS → PROD. |
| CDS / Analytical Models | CDS views, analytical queries created against S/4HANA DEV. | S/4HANA PCE DEV → QAS → PROD | ABAP Transport Requests (CTS). |

### 5.3.3 gCTS Integration (Git-enabled CTS) optional
For S/4HANA PCE systems, gCTS enables Git-based source code management for ABAP objects. This bridges the gap between Azure DevOps Git and the S/4HANA transport system:
Developer creates/modifies ABAP on S/4HANA DEV.
ABAP objects are committed to a gCTS-managed Git repository on Azure DevOps.
The CI/CD pipeline runs appropriate checks, not integrated with the transport system.

TMS (integrated with Cloud ALM) manages the import to S/4HANA QAS and PROD via CTS.
BAS → S/4HANA PCE Flow (with gCTS):BAS (ADT) → ABAP objects on S4-DEV → gCTS commit to Azure DevOps Git→ CI/CD Pipeline (ATC checks, SAST) → Release transport on S4-DEV→ TMS + Cloud ALM approval → CTS import to S4-QAS→ TMS + Cloud ALM approval → CTS import to S4-PROD
### 5.3.4 Fiori Deployment to S/4HANA BSP Repository (legacy)
For Fiori applications targeting the S/4HANA front-end server (BSP repository):
Developer builds the UI5 application in BAS.
The CI/CD pipeline builds the application, runs tests and security scans.
The pipeline executes npm run deploy (using @sap/ux-ui5-tooling) to deploy the app to the S/4HANA DEV BSP repository via Cloud Connector.
The deployment creates an ABAP transport request on S/4HANA DEV.
Cloud ALM manages the transport approval and CTS import to QAS and PROD.
BAS → S/4HANA BSP Flow:BAS (Fiori tools) → Git Push → Azure DevOps Pipeline:  1. Checkout UI5 project  2. npm install + build  3. ESLint + unit tests + SAST  4. npm run deploy (to S4-DEV BSP via Cloud Connector)  5. ABAP transport request created on S4-DEV→ Cloud ALM approval → CTS import to S4-QAS → CTS import to S4-PROD
### 5.3.5 Prerequisites for BAS-to-S/4HANA Connectivity
| Prerequisite | Details |
| --- | --- |
| SAP Cloud Connector | Installed and configured in the S/4HANA PCE network. Exposes S/4HANA HTTP/HTTPS endpoints to BTP. |
| BTP Destination | Destination configured in BTP DEV subaccount pointing to S/4HANA DEV via Cloud Connector. Authentication: Basic or Principal Propagation (recommended). |
| ADT Backend | S/4HANA DEV system must have ADT services enabled (transaction SICF – /sap/bc/adt path activated). |
| gCTS Configuration | gCTS must be configured on S/4HANA DEV with a Git repository connection to Azure DevOps (SSH key or token-based auth). |
| Transport Landscape | S/4HANA CTS transport routes configured: DEV → QAS → PROD. |
| User Authorization | BAS developer users must have appropriate S/4HANA roles (developer role in DEV, no direct access to QAS/PROD). See Section 7.4. |

### 5.3.6 ABAP Code Quality – ATC Checks on Transport Release
The ABAP Test Cockpit (ATC) is the central quality gate for all ABAP code on S/4HANA PCE. ATC triggered automatically by the S/4HANA system when a developer releases a transport task. If ATC finds blocking findings, the transport release is rejected and the developer must fix the issues before releasing again.
How it works: ATC is configured as a “check-before-release” gate on the S/4HANA DEV system. When a developer clicks “Release” on a transport task or request (via ADT in BAS or transaction SE09/SE10), the system automatically executes the configured ATC check variant against all objects in the transport. If findings with priority 1 (error) are detected, the release is blocked. The developer must resolve all blocking findings and re-attempt the release.
#### ATC Check Categories
| ATC Check Category | What It Covers | Gate Criteria |
| --- | --- | --- |
| ABAP Unit Tests | Developer-written unit tests using the CL_ABAP_UNIT_ASSERT framework. Test classes are embedded within the ABAP objects and are executed by ATC as part of the check run. Unit tests validate custom business logic, calculations, mappings, and service behavior. | All unit tests must pass (0 failures). Code coverage target: ≥ 70% for critical objects (configurable in ATC check variant). |
| Code Inspector (SCI) Checks | Static analysis: naming conventions, performance checks (SELECT in loops, buffering), obsolete statements, syntax checks, authorization checks. | No priority 1 (errors). Priority 2 (warnings) reviewed; threshold configurable. |
| Security Checks | ABAP security vulnerabilities: SQL injection, cross-site scripting, missing authority checks, hard-coded credentials, deprecated crypto functions. | No critical security findings. |
| Cloud Readiness Checks | Checks for ABAP Cloud compatibility (released APIs only, no direct DB access, no classic dynpro). Essential for RAP/ABAP Cloud development on S/4HANA PCE. | No unreleased API usage for ABAP Cloud tier objects. |
| Custom Check Variant | Project-specific check variant (transaction ATC) combining the above categories with custom thresholds and exclusions. | Defined by project/architecture team. Registered in ATC system configuration. |

#### ABAP Unit Tests – Execution via ATC
ABAP Unit Tests are not a separate tool or pipeline step – they are executed within ATC as one of the check categories. When ATC runs on transport release, it automatically discovers and executes all test classes (FOR TESTING) found in the objects being transported. This means:
Test classes live alongside production code in the same ABAP package/class, marked with FOR TESTING and the appropriate risk level (RISK LEVEL HARMLESS / CRITICAL).
ATC executes all unit tests in a single run – no separate test runner is needed.
Test failures block the transport release just like any other priority 1 ATC finding.
Developers are expected to write unit tests for all custom business logic classes, behavior implementations, and utility methods.
#### Transport Release Flow with ATC
Transport Release → ATC → Release or Block:1. Developer completes ABAP development in BAS (ADT) on S4-DEV2. Objects are assigned to a transport request/task3. Developer clicks “Release Task” (ADT or SE09/SE10)  → S/4HANA system automatically triggers ATC check variant:    a) ABAP Unit Tests (all FOR TESTING classes in transported objects)    b) Code Inspector static analysis    c) Security checks    d) Cloud readiness checks (if ABAP Cloud tier)  → ATC Result:    - Priority 1 findings or unit test failures → RELEASE BLOCKED      Developer must fix findings and re-attempt release    - All checks passed → TRANSPORT RELEASED4. Released transport is picked up by CTMS + Cloud ALM5. Cloud ALM approval → CTS import to S4-QAS6. Cloud ALM approval → CTS import to S4-PROD
#### ATC Configuration Prerequisites
| Prerequisite | Details |
| --- | --- |
| ATC enabled for transport release | Configure ATC as “check-before-release” via transaction ATC → “Transport Check” settings on S/4HANA DEV. Set the check variant and blocking priority level. |
| Custom ATC check variant | Create a project-specific check variant in transaction ATC that includes: ABAP Unit Tests, Code Inspector checks, security checks, and cloud readiness checks. Assign it as the default for transport release. |
| ABAP Unit Test classes | Developers must write unit test classes (FOR TESTING) for all custom business logic. Test classes are maintained alongside production code in the same ABAP package. |
| Exemption process | For legitimate ATC findings that cannot be fixed (e.g., false positives, accepted technical debt), define an ATC exemption workflow via transaction ATC → Exemptions. Exemptions require approval by the architecture/security team. |

Important: ATC on transport release is a server-side S/4HANA quality gate – it runs regardless of whether gCTS or CI/CD is used. Even if there is no CI/CD pipeline for the ABAP development stream, ATC still enforces code quality at the transport level. The CI/CD pipeline (if present) provides additional checks (e.g., SAST on the Git side), but ATC is the primary and mandatory quality gate for ABAP.
# 6. CI/CD Pipeline Design
## 6.1 Pipeline Architecture
CI/CD pipelines are implemented in Azure DevOps Pipelines (YAML-based) for Git-supported applications. An alternative or complementary option is the SAP Continuous Integration and Delivery Service on BTP, which provides pre-built pipeline templates for SAP-specific build tools (MBT, MTA).
| Pipeline Option | When to Use | Advantages |
| --- | --- | --- |
| Azure DevOps Pipelines | Primary option when Azure DevOps is the Git platform | Unified Git + CI/CD, rich marketplace extensions, native Azure integration, custom agents |
| SAP CI/CD Service | Alternative when SAP-native tooling is preferred | Pre-built SAP pipeline stages, native CTMS integration, lower configuration effort |

## 6.2 Build & Quality Gates
Every pipeline (regardless of platform) must include these stages:
| Stage | Activities | Gate Criteria |
| --- | --- | --- |
| 1. Source | Git checkout, dependency resolution | Clean checkout, no dependency conflicts |
| 2. Build | MBT build, compile, bundle | Build succeeds without errors |
| 3. Test | Unit tests, integration tests | All tests pass, code coverage ≥ threshold (e.g., 70%) |
| 4. Security Scan | SAST, SCA/dependency scan (see Section 8) | No critical/high findings. Medium findings reviewed. |
| 5. Package | Generate MTAR archive | Valid MTAR produced |
| 6. Deploy to DEV | Upload MTAR to CTMS DEV node | CTMS upload succeeds |
| 7. Notify | Update Cloud ALM feature/deployment status; notify team | Cloud ALM status reflects pipeline outcome |

## 6.3 Security Scanning in Pipeline
Security scanning is a mandatory, non-skippable pipeline stage. See Section 8 for the detailed security strategy. In summary:
SAST – SonarQube (or SonarCloud), Checkmarx, or Fortify integrated as an Azure DevOps Pipeline task.
SCA / Dependency Scanning – npm audit, Mend (formerly WhiteSource), Snyk, or GitHub Advanced Security for Azure DevOps.
Secrets Detection – Pre-commit hooks (e.g., detect-secrets, Azure DevOps Credential Scanner) to prevent accidental credential commits.
# 7. Deployment Strategy
## 7.1 SAP Cloud Transport Management Service (CTMS)
CTMS is the single orchestration layer for all application deployments across the BTP landscape, regardless of whether the application is Git-supported or not.
### Transport Nodes
| Node | Mapped Subaccount | Role |
| --- | --- | --- |
| DEV | DEV subaccount | Source node. Receives MTAR uploads from CI/CD pipelines or direct uploads. |
| QAS | QAS subaccount | Target node (from DEV). Receives transports after Cloud ALM approval. |
| PROD | PROD subaccount | Target node (from QAS). Receives transports after Cloud ALM production approval. |

## 7.2 SAP Cloud ALM Integration
Cloud ALM provides the governance layer on top of CTMS:
Features: Every deployable change is linked to a Cloud ALM feature (story/task). This ensures traceability from business requirement to production deployment.
Deployment Management: Cloud ALM triggers and monitors CTMS transports. Transport requests in CTMS are linked to Cloud ALM features.
Approval Workflow: Deployment to QAS requires approval from the QA lead. Deployment to PROD requires approval from the change advisory board (CAB) or designated approver.
### Cloud ALM Deployment Logging
Cloud ALM logs every deployment event:
| Event | Logged Data | Retention |
| --- | --- | --- |
| Transport created | Transport ID, source node, MTAR details, creator, timestamp | Per Cloud ALM retention policy |
| Transport approved | Approver, timestamp, target node | Per Cloud ALM retention policy |
| Transport deployed | Deployment status (success/failed), target node, timestamp, log details | Per Cloud ALM retention policy |
| CI/CD build status | Build result (success/failed), pipeline URL, commit hash | Per Cloud ALM retention policy |

Logging Rule – CI/CD Build Failures: If a CI/CD pipeline build fails, the failure is logged in Cloud ALM (linked to the feature/deployment). If there is no CI/CD pipeline (non-Git applications), any deployment failures are logged in the application’s native monitoring (e.g., BPA execution logs, Work Zone admin logs) and correlated in Cloud ALM via the CTMS transport status.
## 7.3 Transport Routes
Git-Supported Applications:BAS → Git Commit → CI/CD Pipeline → MTAR → CTMS [DEV] → Cloud ALM Approval → CTMS [QAS] → Cloud ALM Approval → CTMS [PROD]Non-Git Applications:Application (DEV) → Native Export → CTMS [DEV] → Cloud ALM Approval → CTMS [QAS] → Cloud ALM Approval → CTMS [PROD]

S/4HANA PCE Extensions (from BAS):BAS → Git Commit → CI/CD Pipeline → ABAP Transport / gCTS → CTMS [S4-DEV] → CALM Approval → CTS Import [S4-QAS] → CALM Approval → CTS Import [S4-PROD]

## 7.4 Deployment Roles & Authorization Matrix
This section defines who can perform which deployment activities across the landscape. Strict segregation of duties (SoD) is enforced: the person who develops/uploads cannot approve, and the person who approves cannot import.
### 7.4.1 BTP Application Deployments (via CTMS + Cloud ALM)
| Activity | Description | Responsible Role | Tool / Permission |
| --- | --- | --- | --- |
| Upload / Create Transport | Upload MTAR / MDK package / content to CTMS DEV node. Can be manual or triggered by CI/CD pipeline. | Developer (via CI/CD pipeline)or DevOps Engineer (manual upload) | CTMS: ImportOperator role on DEV node.CI/CD service account with CTMS API access. |
| Approve Transport to QAS | Review and approve the transport for deployment to the QAS environment in Cloud ALM. | QA Lead / Test Manager | Cloud ALM: Deployment Manager or Change Approver role.Must NOT be the same person who uploaded. |
| Import to QAS | Execute the transport import into the QAS subaccount. Triggered automatically after Cloud ALM approval, or manually by DevOps. | CTMS (automated)or DevOps Engineer | CTMS: ImportOperator role on QAS node. |
| Approve Transport to PROD | Review and approve the transport for deployment to production in Cloud ALM. Requires successful QAS testing sign-off. | Release Manager / CAB | Cloud ALM: Deployment Manager or Change Approver role.Must NOT be the developer or QAS approver. |
| Import to PROD | Execute the transport import into the PROD subaccount. Triggered after Cloud ALM production approval. | CTMS (automated)or Platform Admin | CTMS: ImportOperator role on PROD node. |

### 7.4.2 S/4HANA PCE Deployments (via CTS / gCTS + Cloud ALM)
| Activity | Description | Responsible Role | Tool / Permission |
| --- | --- | --- | --- |
| Create & Release Transport | Developer creates ABAP objects in BAS/ADT, which generates a transport request on S/4HANA DEV. Developer releases the transport after CI/CD passes. | ABAP Developer | S/4HANA DEV: S_TRANSPRT authorization object with ACTVT 01/02 (create/change).gCTS: Git push permission on Azure DevOps. |
| Approve Transport to QAS | Review and approve the transport for import into S/4HANA QAS. Done in Cloud ALM deployment management. | QA Lead / Test Manager | Cloud ALM: Deployment Manager role.Must NOT be the developer. |
| Import to QAS | Execute the transport import into S/4HANA QAS. Triggered by CTMS/Cloud ALM after approval, or manually by Basis admin. | Basis Administratoror CTMS (automated via CTS) | S/4HANA QAS: S_TRANSPRT ACTVT 43 (import).Transaction STMS. |
| Approve Transport to PROD | Review and approve for production import. Requires QAS sign-off. | Release Manager / CAB | Cloud ALM: Deployment Manager role.Must NOT be the developer. |
| Import to PROD | Execute the transport import into S/4HANA PROD. | Basis Administratoror CTMS (automated via CTS) | S/4HANA PROD: S_TRANSPRT ACTVT 43 (import).Transaction STMS. |

### 7.4.3 Segregation of Duties Summary
Key SoD Rules:
A Developer can upload/create transports but cannot approve or import to QAS or PROD.
A QA Approver can approve for QAS but cannot approve for PROD (different approver required).
A Basis/Platform Admin can import transports but cannot approve them (they execute on behalf of the approver).
No single person should hold upload + approve + import rights simultaneously.
| Role | Upload | Approve QAS | Import QAS | Approve PROD | Import PROD |
| --- | --- | --- | --- | --- | --- |
| Developer | ✓ | ✗ | ✗ | ✗ | ✗ |
| DevOps Engineer | ✓ | ✗ | ✓ | ✗ | ✓ |
| QA Lead | ✗ | ✓ | ✗ | ✗ | ✗ |
| Release Manager / CAB | ✗ | ✗ | ✗ | ✓ | ✗ |
| Basis / Platform Admin | ✗ | ✗ | ✓ | ✗ | ✓ |
| CI/CD Service Account | ✓ | ✗ | ✗ | ✗ | ✗ |

# 8. Security Strategy
Security is integrated at every stage of the software delivery lifecycle (“shift left”). The following subsections detail the security controls across development, build, deployment, and runtime.
## 8.1 SAP Cloud Identity Services (CIS)
CIS is the identity backbone of the BTP landscape, providing:
| Capability | Implementation |
| --- | --- |
| Single Sign-On (SSO) | All BTP applications and Azure DevOps authenticate via CIS (IAS) using SAML 2.0 or OIDC. Corporate IdP (Azure AD / Entra ID) is federated as a proxy identity provider. |
| Multi-Factor Authentication (MFA) | MFA enforced for all administrative access (BTP cockpit, CTMS admin, Cloud ALM admin) and production deployments. |
| Role-Based Access Control (RBAC) | BTP role collections mapped to CIS groups. Principle of least privilege enforced per subaccount. Developers cannot deploy directly to QAS/PROD. |
| User Provisioning | IPS automates user lifecycle (joiner/mover/leaver) across BTP subaccounts. Deprovisioning is triggered by HR events via corporate IdP. |
| Service-to-Service Auth | OAuth 2.0 client credentials flow for CI/CD pipelines accessing BTP APIs (CTMS upload, Cloud ALM status update). Service keys stored in Azure DevOps secure variables. |

## 8.2 Secrets & Credential Management
Azure DevOps: All service keys, API tokens, and credentials are stored in Azure DevOps Variable Groups (linked to Azure Key Vault where possible). Never committed to Git.
BTP: Credentials stored in SAP Credential Store or BTP Destination service. Integration flows must externalize all credentials (no hardcoded values).
Pre-commit hooks: detect-secrets or Azure DevOps Credential Scanner runs on every commit to prevent accidental credential exposure.
## 8.3 Static Application Security Testing (SAST)
SAST is a mandatory pipeline stage for all Git-supported applications:
Tool: SonarQube / SonarCloud (recommended), Checkmarx, or Fortify.
Integration: Azure DevOps Pipeline task; results posted as PR comments and dashboard.
Gate: Critical and high findings block the pipeline. Medium findings generate warnings.
Coverage: JavaScript/TypeScript (Fiori, Mobile), Node.js (CAP), Groovy/XML (Integration flows).
## 8.4 Dependency & Vulnerability Scanning (SCA)
Tool: npm audit (basic), Mend (WhiteSource), Snyk, or GitHub Advanced Security for Azure DevOps.
Integration: Runs as a pipeline stage after dependency installation.
Gate: Known critical CVEs block the pipeline. License compliance violations flagged.
Frequency: On every build (CI trigger) and scheduled weekly scan of all repositories.
# 9. Monitoring, Logging & Alerting
## 9.1 SAP Cloud ALM – Deployment Logging
Cloud ALM serves as the central audit log for all deployment activities across the landscape:
Every CTMS transport (create, approve, deploy, reject, fail) is logged with timestamp, actor, and outcome.
CI/CD pipeline results (build success/failure, test results, scan outcomes) are linked to Cloud ALM features.
Deployment dashboards in Cloud ALM provide a single pane of glass for all application deployments.
### Logging Matrix
| Application Type | CI/CD Build Log | Deployment Log | Failure Logging |
| --- | --- | --- | --- |
| Git-supported (SSAM, Fiori, Integration with CI/CD) | Azure DevOps Pipeline logs + Cloud ALM | Cloud ALM (via CTMS) | CI/CD failure: Cloud ALM + Azure DevOps. Deployment failure: Cloud ALM. |
| Integration (without CI/CD) | N/A | Cloud ALM (via CTMS) | Deployment failure: Cloud ALM. Runtime errors: Integration Suite monitoring. |
| Non-Git (BPA, Work Zone, Data Cloud) | N/A | Cloud ALM (via CTMS) | Deployment failure: Cloud ALM. Application errors: native app monitoring. |

## 9.2 CI/CD Build Failure Logging
When a CI/CD pipeline build fails:
The pipeline posts the failure status to Cloud ALM (feature text updated to “Build Failed”).
The detailed build log remains in Azure DevOps Pipelines for developer diagnosis.
A notification is sent via Azure DevOps notifications and, optionally, via SAP Alert Notification Service.
For applications without CI/CD (non-Git or Integration without pipelines), failures occur at deployment time (CTMS import) and are logged in Cloud ALM deployment management and the application’s native logs.
## 9.3 SAP Alert Notification Service (ANS) for BTP
The SAP Alert Notification Service provides real-time, event-driven alerting for the BTP platform and applications.
### Alert Configuration
| Alert Category | Source | Condition | Action |
| --- | --- | --- | --- |
| Deployment Failure | CTMS / Cloud ALM | Transport import fails on any node | Email + Teams/Slack notification to DevOps team |
| CI/CD Build Failure | Azure DevOps (via webhook to ANS) | Pipeline build/test/scan stage fails | Email notification to development team |
| Application Runtime Error | BTP application logs, Cloud Foundry / Kyma | Error rate exceeds threshold | Email + incident creation in Cloud ALM |
| Security Vulnerability | SCA/SAST tooling | Critical CVE detected in dependency | Urgent email to security lead + DevOps team |
| Platform Health | BTP platform events | Service instance down, quota exceeded, certificate expiry | Email + auto-remediation via Automation Pilot |
| Identity / Access | CIS audit logs | Failed login attempts exceeding threshold, privilege escalation | Email to security team |

Integration Pattern: ANS connects to external channels (email, Microsoft Teams, Slack, ServiceNow, PagerDuty) via webhooks and the SAP BTP Destination service. Cloud ALM can also consume ANS events to create operational incidents.
## 9.4 SIEM Integration
All security-relevant logs from the SAP BTP landscape and S/4HANA PCE systems are forwarded to the organization’s Security Information and Event Management (SIEM) platform for centralized correlation, threat detection, and compliance reporting.
### 9.4.1 Log Sources & SIEM Feeds
| Log Source | Log Type | Feed Mechanism | SIEM Use Case |
| --- | --- | --- | --- |
| SAP Cloud ALM | Deployment events, approval audit trail, feature changes | Cloud ALM API → ANS webhook → SIEM ingestion endpoint (e.g., Splunk HEC, Sentinel Data Connector) | Unauthorized deployment detection, SoD violation monitoring, change audit |
| SAP Alert Notification Service | Platform events, application errors, security alerts | ANS action → Webhook to SIEM | Real-time security event correlation, incident detection |
| SAP Cloud Identity Services (CIS/IAS) | Authentication events, failed logins, MFA events, role changes, user provisioning | IAS Audit Log API → scheduled export to SIEM, or real-time via ANS | Brute-force detection, privilege escalation, access anomaly detection |
| BTP Cloud Foundry / Kyma Audit Log | Service instance creation/deletion, environment changes, security events | SAP Audit Log Viewer API → SIEM ingestion | Infrastructure change tracking, unauthorized service modifications |
| Azure DevOps | Pipeline execution logs, repository access, PR approvals, branch policy overrides | Azure DevOps Service Hooks → SIEM (or Azure Monitor → Sentinel) | CI/CD tampering detection, code access anomalies, policy bypass alerts |
| S/4HANA PCE Security Audit Log | Transaction execution, transport imports, user changes, authorization checks (SM20/RSAU_READ) | SAP Enterprise Threat Detection (ETD) → SIEM, or direct syslog/API export | Unauthorized transport imports, critical transaction monitoring, user behavior analytics |
| SAST / SCA Tools | Security scan findings, vulnerability reports | Tool-native integration (e.g., SonarQube webhook, Snyk API) → SIEM | Vulnerability trend analysis, compliance dashboards |

### 9.4.2 SIEM Correlation Rules
The SIEM platform should be configured with correlation rules specific to the SAP DevSecOps landscape:
| Rule | Detection Logic | Severity |
| --- | --- | --- |
| SoD Violation | Same user uploads and approves a transport within the same change cycle | Critical |
| After-Hours Deployment | Production deployment approved/imported outside business hours without emergency change flag | High |
| Bypassed CI/CD | Transport uploaded to CTMS without a matching CI/CD pipeline run (for Git-supported apps) | High |
| Failed Login Spike | > N failed login attempts on CIS/IAS within M minutes for any single account | High |
| Critical Vulnerability Deployed | SAST/SCA reports critical finding AND transport was still deployed to QAS/PROD | Critical |
| Unauthorized S/4HANA Transport | Transport imported to S/4HANA QAS/PROD without a corresponding Cloud ALM approval | Critical |
| Service Key Rotation Overdue | CI/CD service key or BTP credential older than rotation policy threshold | Medium |

### 9.4.3 Supported SIEM Platforms
| SIEM Platform | Integration Method |
| --- | --- |
| Microsoft Sentinel | Azure DevOps native connector + SAP BTP data via Azure Logic Apps / Function Apps consuming ANS webhooks. SAP S/4HANA via SAP Sentinel Solution (SAL connector). |
| Splunk | ANS → Splunk HTTP Event Collector (HEC). S/4HANA via SAP ETD or direct syslog. Azure DevOps via Splunk Add-on for Azure. |
| SAP Enterprise Threat Detection (ETD) | Native integration with S/4HANA security audit log. Can forward correlated events to external SIEM. |
| IBM QRadar / Other | ANS webhook → syslog or REST API endpoint. Custom parsers for SAP log formats. |

Compliance Requirement: All security-relevant events must be retained in the SIEM for a minimum of 12 months (or as required by the organization’s data retention policy). Cloud ALM, ANS, and CIS audit logs should be exported regularly to prevent data loss due to platform retention limits.

# 10. Automation & Orchestration
## 10.1 SAP Automation Pilot
SAP Automation Pilot is a BTP service that provides automated command execution, scheduling, and orchestration for operational tasks. In the context of this DevSecOps strategy, Automation Pilot serves as the deployment tracking and automated remediation engine.
### Key Use Cases
| Use Case | Description | Trigger |
| --- | --- | --- |
| Deployment Tracking | Track the status of all CTMS transports across the landscape. Aggregate deployment metrics (success rate, mean time to deploy, rollback frequency). | Scheduled (e.g., every 15 minutes) or event-driven (ANS alert) |
| Automated Health Checks | After each deployment to QAS/PROD, run automated smoke tests or health-check commands to validate the deployment. | Post-deployment event from CTMS/Cloud ALM |
| Auto-Remediation | If a deployment fails or a runtime health check detects an issue, Automation Pilot can trigger a rollback or restart the application. | ANS alert (deployment failure or health check failure) |
| Certificate & Credential Rotation | Automated rotation of service keys and certificates used by CI/CD pipelines and BTP services. | Scheduled (before expiry) |
| Environment Provisioning | Automated setup of BTP subaccount configurations, destinations, and service instances for new environments. | On demand |
| Compliance Reporting | Generate periodic reports on deployment compliance (e.g., all PROD deployments have Cloud ALM approval, all Git apps passed SAST). | Scheduled (weekly/monthly) |

## 10.2 Deployment Tracking & Self-Healing
Automation Pilot integrates with CTMS, Cloud ALM, and ANS to create a closed-loop deployment tracking system:
1. CI/CD or manual upload → CTMS transport created2. Cloud ALM approval → CTMS deploys to QAS/PROD3. Automation Pilot runs post-deployment health check4. IF health check passes → Cloud ALM status updated to “Deployed Successfully”5. IF health check fails → ANS alert → Automation Pilot triggers rollback → Cloud ALM status “Rolled Back”6. Deployment metrics aggregated and reported
# 11. Governance & Compliance
## 11.1 Change Management Governance
Every change deployed to QAS or PROD must be linked to a Cloud ALM feature.
Git-supported changes must be traceable to a Git commit and Azure DevOps work item.
Production deployments require documented approval in Cloud ALM (CAB or designated approver).
Emergency changes follow an expedited workflow but still require post-deployment approval and documentation.
## 11.2 Audit Trail
The combined toolchain provides a complete audit trail:
| Layer | Audit Data | Tool | SIEM Feed |
| --- | --- | --- | --- |
| Code change | Who changed what, when, why | Azure DevOps Git | Yes |
| Code review | Reviewer, approval decision | Azure DevOps PRs | Yes |
| Build & scan | Build result, test coverage, scan outcome | Azure DevOps Pipelines + SonarQube/Snyk | Yes |
| Transport | Transport creation, approval, deploy status | CTMS + Cloud ALM | Yes |
| Approval | Who approved QAS/PROD and when | Cloud ALM | Yes |
| Runtime | Application health, errors, performance | Cloud ALM Operations + ANS | Alerts only |
| Identity | Authentication, authorization events | CIS audit logs | Yes |
| S/4HANA transport | CTS import log, transport release | S/4HANA STMS / SM20 | Yes (via ETD) |

## 11.3 Compliance Policies
Segregation of Duties (SoD): Developers cannot approve their own changes. Separate roles for development, QA approval, and production approval (see Section 7.5).
No Direct Production Access: No developer has direct deployment rights to PROD. All changes go through CTMS + Cloud ALM approval.
Regular Access Reviews: Quarterly review of BTP role assignments and S/4HANA authorizations.
Data Protection: No production data in DEV/QAS. Anonymized or synthetic test data used.
SIEM Monitoring: All deployment and security events forwarded to SIEM. Correlation rules for SoD violations, unauthorized access, and policy bypasses (see Section 9.4).

# 12. Roles & Responsibilities
| Role | Responsibilities | Tools |
| --- | --- | --- |
| Developer | Write code in BAS (Fiori, Mobile/MDK, ABAP), commit to Git, create PRs, fix security findings. Develop BPA/Work Zone/Data Cloud content. Upload transports to DEV only. | BAS, Azure DevOps, BPA Lobby, Work Zone admin, Datasphere, S/4HANA (DEV only) |
| DevOps Engineer | Maintain CI/CD pipelines, manage CTMS configuration, configure Automation Pilot, manage ANS subscriptions, execute imports when needed. | Azure DevOps, CTMS, Automation Pilot, ANS |
| Security Engineer | Configure SAST/SCA tools, manage CIS policies, review scan results, define security gates, manage SIEM rules and dashboards. | SonarQube/Snyk, CIS/IAS, SIEM platform |
| QA Lead | Approve QAS deployments in Cloud ALM, validate test results, run acceptance tests. | Cloud ALM |
| Release Manager / CAB | Approve PROD deployments in Cloud ALM, coordinate release schedules. | Cloud ALM |
| Basis / Platform Admin | Manage BTP subaccounts, S/4HANA system administration, CIS configuration, CTMS nodes, execute CTS imports. | BTP Cockpit, CIS/IAS, CTMS, S/4HANA STMS |

# 13. Roadmap & Next Steps
| Phase | Timeline | Activities |
| --- | --- | --- |
| Phase 1: Foundation | Weeks 1–4 | • Set up BTP subaccount topology (DEV, QAS, PROD, Platform)• Configure CIS (IAS federation with corporate IdP, RBAC)• Set up CTMS transport nodes and routes• Subscribe Cloud ALM and configure deployment management• Create Azure DevOps project and initial Git repositories |
| Phase 2: CI/CD & Security | Weeks 5–8 | • Build CI/CD pipeline templates (SAP CID) for Fiori, Mobile, Integration• Integrate SAST tool (SonarQube/SonarCloud) into pipelines• Integrate SCA tool (npm audit + Snyk/Mend) into pipelines• Configure pre-commit hooks for secrets detection• Define and enforce branch policies in Azure DevOps |
| Phase 3: Monitoring & Alerting | Weeks 9–12 | • Configure SAP Alert Notification Service subscriptions and actions• Set up Automation Pilot commands for deployment tracking and health checks• Integrate Cloud ALM operations monitoring• Configure failure notification channels (email, Teams/Slack) |
| Phase 4: Optimization | Weeks 13–16 | • Implement auto-remediation (rollback) via Automation Pilot• Build compliance reporting dashboards• Onboard all remaining applications to the DevSecOps pipeline• Conduct security review and penetration testing• Document operational runbooks |

## Immediate Next Steps
Review and approve this strategy document with all stakeholders.
Confirm tool selection for SAST and SCA scanning.
Provision BTP subaccounts and platform services (Phase 1).
Assign roles and onboard team members to Azure DevOps and Cloud ALM.
Begin CI/CD pipeline development for the first pilot application (e.g., a Fiori app).
# 14. Appendix
## A. Glossary
| Term | Definition |
| --- | --- |
| ADT | ABAP Development Tools – Eclipse/BAS-based IDE for ABAP development |
| ANS | SAP Alert Notification Service – real-time alerting for BTP events |
| ATC | ABAP Test Cockpit – static analysis and checks for ABAP code |
| BAS | SAP Business Application Studio – cloud-based IDE |
| BPA | SAP Build Process Automation – low-code/no-code process automation |
| BSP | Business Server Pages – ABAP repository for UI5/Fiori apps on S/4HANA |
| BTP | SAP Business Technology Platform |
| CAB | Change Advisory Board |
| CIS | SAP Cloud Identity Services (IAS + IPS + Identity Directory) |
| CTS | Change and Transport System – ABAP transport mechanism in S/4HANA |
| CTMS | SAP Cloud Transport Management Service |
| ETD | SAP Enterprise Threat Detection – SIEM for SAP systems |
| gCTS | Git-enabled Change and Transport System – Git integration for ABAP transports |
| MDK | SAP Mobile Development Kit – metadata-driven mobile app framework |
| MTAR | Multi-Target Application Archive – deployable package for BTP |
| PCE | Private Cloud Edition – SAP-managed on-premise S/4HANA |
| SAST | Static Application Security Testing |
| SCA | Software Composition Analysis (dependency scanning) |
| SIEM | Security Information and Event Management |
| SoD | Segregation of Duties |
| SSAM | SAP Mobile Services application (mobile development using MDK) |

## B. Reference Architecture Diagram

## C. Pipeline YAML Template (Azure DevOps)
[Insert reference Azure DevOps YAML pipeline template for SAP Fiori MTAR build, including SAST and SCA stages.]
## D. CTMS Configuration Reference
[Insert CTMS node and route configuration details specific to the customer landscape.]
## E. SIEM Configuration Reference
[Insert SIEM correlation rules, log format specifications, and dashboard templates.]
