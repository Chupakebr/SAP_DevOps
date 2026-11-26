# DevSecOps Strategy
SAP Business Technology Platform Applications
Azure DevOps & SAP Cloud ALM Integration
DRAFT

Document Version: .
Date: 2026
Author: Rodion RODIONOV
Organization: SAP SE – Cross-Solution Delivery, Integrated Toolchain
Classification: SAP Internal
# Document Control
## Revision History
| Version | Date | Author | Changes |
| --- | --- | --- | --- |
| 0.1 | October 2026 | R. Rodionov | Initial draft |
| 1.0 |  |  |  |
|  |  |  |  |

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

# 1. Executive Summary
This document defines the DevSecOps strategy for managing the full lifecycle of SAP Business Technology Platform (BTP) applications. It addresses source code management, CI/CD pipelines, deployment orchestration, security, monitoring, and automation across two distinct application categories:
Git-supported applications – SSAM (Mobile), Fiori Apps, and Integration Services – that leverage Azure DevOps Git repositories with CI/CD pipelines producing MTAR archives for deployment and Cloud ALM.
Non-Git applications – SAP Build Process Automation (BPA), SAP Build Work Zone, and SAP Data Cloud – that rely on native content transport via CTMS and Cloud ALM without source code version control.
The strategy integrates the following SAP and Microsoft platform services into a cohesive DevSecOps toolchain:
| Service | Role in Toolchain |
| --- | --- |
| Azure DevOps (Git) | Source code management, branching, pull requests |
| Azure DevOps Pipelines / SAP | Build, test, security scan, MTAR packaging |
| SAP Cloud Transport Management (CTMS) | Transport orchestration across DEV → QAS → PROD |
| SAP Cloud ALM | Change & deployment management, monitoring, logging |
| SAP Cloud Identity Services () | Identity federation, SSO, role-based access |
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
| Security | SAST, SCA, secrets management, identity governance () |
| Monitoring & Alerting | Cloud ALM operations, Alert Notification Service, Automation Pilot |
| Environments | DEV, QAS, PROD subaccounts on SAP BTP |

## 2.2 Out of Scope

Non-SAP application pipelines
Infrastructure-as-Code for BTP subaccount provisioning (unless Automation Pilot is extended)
## 2.3 Objectives
Standardize the development-to-production lifecycle for all SAP BTP applications.
Shift security left – embed security checks into every CI/CD pipeline stage.
Ensure traceability – every deployment to QAS/PROD must be traceable to a Cloud ALM feature/change, a Git commit (where applicable), and an approval workflow.
Automate repetitive deployment and monitoring tasks via Automation Pilot.
Centralize logging – all deployment activities and CI/CD outcomes logged in Cloud ALM.
Alert proactively – leverage Alert Notification Service for real-time inent detection.
# 3. BTP Landscape Architecture
## 3.1 Subaccount Topology
The BTP landscape follows a three-tier subaccount model aligned with the standard SAP transport landscape. An additional platform services subaccount hosts shared DevSecOps tooling.
| Subaccount | Purpose | Key Services |
| --- | --- | --- |
| DEV | Development & unit testing. Developers deploy freely. CTMS source node. | SAP Business Application Studio (BAS), application runtimes, service instances |
| QAS | Quality assurance, integration testing, UAT. Deployments only via CTMS transport. | Application runtimes, service instances (cloned from DEV config) |
| PROD | Production. Deployments only via approved CTMS transport from QAS. | Application runtimes, service instances (production-grade sizing) |
| Platform / DevOps | Shared DevSecOps services. Not application-specific. | CTMS, Cloud ALM (if BTP-based), CI/CD service, Alert Notification, Automation Pilot,  tenant |

BTP Global Account
  ├─ Subaccount: DEV          (development, BAS, runtimes)
  ├─ Subaccount: QAS          (quality, testing)
  ├─ Subaccount: PROD         (production)
  └─ Subaccount: Platform     (CTMS, , ANS, Automation Pilot)

## 3.2 Core Platform Services
### SAP Cloud Identity Services (CI)
CI provides the identity layer for the entire BTP landscape:
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
Design Deion: Cloud ALM is subscribed in the Platform subaccount (or as a SaaS subscription at global-account level) to maintain independence from any single application subaccount. CTMS and  are similarly centralized.
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

|  |  |
| --- | --- |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |

|  |  |
| --- | --- |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |

|  |  |
| --- | --- |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |

### 5.1.2 SAP Fiori Applications
| Aspect | Details |
| --- | --- |
| Development IDE | SAP Business Application Studio (BAS) with Fiori tools |
| Source Control | Azure DevOps Git |
| CI/CD | Azure DevOps Pipelines (or SAP CI/CD Service) |
| Build Artifact | MTAR (containing UI5 apps, CAP services, etc.) |
| Deployment Path | MTAR → CTMS → Cloud ALM → QAS → PROD |

Pipeline flow:
Developer (BAS) → Git Push → Azure DevOps Pipeline:
  1. Checkout code
  2. npm install / dependency resolution
  3. ESLint + UI5 linter
  4. Unit tests (QUnit / OPA5 / Jest)
  5. SAST scan
  6. Dependency vulnerability scan
  7. MBT build → MTAR
  8. Upload MTAR to CTMS (DEV node)
  9. Notify Cloud ALM
→ CTMS transport to QAS → CTMS transport to PROD
### 5.1.3 SAP Integration Services (SAP Integration Suite)
| Aspect | Details |
| --- | --- |
| Development | SAP Integration Suite Design workspace (Cloud Integration, API Management) |
| Source Control | Azure DevOps Git (optional – via Integration Suite Git integration or manual export/import) |
| CI/CD | Optional – Azure DevOps Pipelines or SAP CI/CD Service. Can also use CTMS-only flow. |
| Build Artifact | MTAR (Integration content package) |
| Deployment Path | MTAR → CTMS → Cloud ALM → QAS → PROD |

Note: CI/CD is optional for Integration Services. When CI/CD is not used, the integration developer exports the content package as an MTAR directly from the Integration Suite and uploads it to CTMS. The CTMS + Cloud ALM approval workflow remains the same regardless.
Pipeline flow (with CI/CD):
Developer (Integration Suite) → Git Push (or API export) → Azure DevOps Pipeline:
  1. Checkout / download integration package
  2. Validate integration flow configuration
  3. Security review (credential externalization check)
  4. Package as MTAR
  5. Upload to CTMS (DEV node)
  6. Notify Cloud ALM
→ CTMS transport to QAS → CTMS transport to PROD
Pipeline flow (without CI/CD):
Developer (Integration Suite) → Export MTAR → Upload to CTMS (DEV node)
→ Cloud ALM approval → CTMS transport to QAS → CTMS transport to PROD

|  |  |  |  |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

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

Developer (BPA Lobby, DEV) → Release project → CTMS picks up content
→ Cloud ALM approval → CTMS transport to QAS → CTMS transport to PROD
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

Admin (Work Zone, DEV) → Configure site / content → CTMS export
→ Cloud ALM approval → CTMS transport to QAS → CTMS transport to PROD
### 5.2.3 SAP Datasphere / Data Cloud
| Aspect | Details |
| --- | --- |
| Development | SAP Datasphere modeler (spaces, views, data flows, analytic models) |
| Source Control | Not applicable (content managed within Datasphere) |
| CI/CD | Not applicable |
| Transport Mechanism | CTMS (Datasphere integrates with CTMS for content transport) |
| Deployment Path | Datasphere (DEV) → CTMS → Cloud ALM approval → QAS → PROD |

Data Modeler (Datasphere, DEV) → Create/modify spaces, views, flows → CTMS export
→ Cloud ALM approval → CTMS transport to QAS → CTMS transport to PROD

|  |  |  |  |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

|  |  |
| --- | --- |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |

|  |  |  |
| --- | --- | --- |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |

|  |  |
| --- | --- |
|  |  |
|  |  |
|  |  |
|  |  |

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
Secrets Detection – Pre-commit hooks (e.g., detect-secrets, Azure DevOps Credential Scanner) to prevent acental credential commits.
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
Git-Supported Applications:
BAS → Git Commit → CI/CD Pipeline → MTAR → CTMS [DEV] → Cloud ALM Approval → CTMS [QAS] → Cloud ALM Approval → CTMS [PROD]

Non-Git Applications:
Application (DEV) → Native Export → CTMS [DEV] → Cloud ALM Approval → CTMS [QAS] → Cloud ALM Approval → CTMS [PROD]

|  |  |  |  |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

|  |  |
| --- | --- |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |

|  |  |  |  |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

|  |  |  |  |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

|  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |

# 8. Security Strategy
Security is integrated at every stage of the software delivery lifecycle (“shift left”). The following subsections detail the security controls across development, build, deployment, and runtime.
## 8.1 SAP Cloud Identity Services ()
is the identity backbone of the BTP landscape, providing:
| Capability | Implementation |
| --- | --- |
| Single Sign-On (SSO) | All BTP applications and Azure DevOps authenticate via  (IAS) using SAML 2.0 or OIDC. Corporate IdP (Azure AD / Entra ID) is federated as a proxy identity provider. |
| Multi-Factor Authentication (MFA) | MFA enforced for all administrative access (BTP cockpit, CTMS admin, Cloud ALM admin) and production deployments. |
| Role-Based Access Control (RBAC) | BTP role collections mapped to  groups. Principle of least privilege enforced per subaccount. Developers cannot deploy directly to QAS/PROD. |
| User Provisioning | IPS automates user lifecycle (joiner/mover/leaver) across BTP subaccounts. Deprovisioning is triggered by HR events via corporate IdP. |
| Service-to-Service Auth | OAuth 2.0 client credentials flow for CI/CD pipelines accessing BTP APIs (CTMS upload, Cloud ALM status update). Service keys stored in Azure DevOps secure variables. |

## 8.2 Secrets & Credential Management
Azure DevOps: All service keys, API tokens, and credentials are stored in Azure DevOps Variable Groups (linked to Azure Key Vault where possible). Never committed to Git.
BTP: Credentials stored in SAP Credential Store or BTP Destination service. Integration flows must externalize all credentials (no hardcoded values).
Pre-commit hooks: detect-secrets or Azure DevOps Credential Scanner runs on every commit to prevent acental credential exposure.
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
| Application Runtime Error | BTP application logs, Cloud Foundry / Kyma | Error rate exceeds threshold | Email + inent creation in Cloud ALM |
| Security Vulnerability | SCA/SAST tooling | Critical CVE detected in dependency | Urgent email to security lead + DevOps team |
| Platform Health | BTP platform events | Service instance down, quota exceeded, certificate expiry | Email + auto-remediation via Automation Pilot |
| Identity / Access | audit logs | Failed login attempts exceeding threshold, privilege escalation | Email to security team |

Integration Pattern: ANS connects to external channels (email, Microsoft Teams, Slack, ServiceNow, PagerDuty) via webhooks and the SAP BTP Destination service. Cloud ALM can also consume ANS events to create operational inents.

|  |  |  |  |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

|  |  |  |
| --- | --- | --- |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |

|  |  |
| --- | --- |
|  |  |
|  |  |
|  |  |
|  |  |

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
1. CI/CD or manual upload → CTMS transport created
2. Cloud ALM approval → CTMS deploys to QAS/PROD
3. Automation Pilot runs post-deployment health check
4. IF health check passes → Cloud ALM status updated to “Deployed Successfully”
5. IF health check fails → ANS alert → Automation Pilot triggers rollback → Cloud ALM status “Rolled Back”
6. Deployment metrics aggregated and reported
# 11. Governance & Compliance
## 11.1 Change Management Governance
Every change deployed to QAS or PROD must be linked to a Cloud ALM feature.
Git-supported changes must be traceable to a Git commit and Azure DevOps work item.
Production deployments require documented approval in Cloud ALM (CAB or designated approver).
Emergency changes follow an expedited workflow but still require post-deployment approval and documentation.
## 11.2 Audit Trail
The combined toolchain provides a complete audit trail:
|  |  |  |  |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

|  |  |  |
| --- | --- | --- |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |

## 11.3 Compliance Policies

# 12. Roles & Responsibilities
|  |  |  |
| --- | --- | --- |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |

# 13. Roadmap & Next Steps
| Phase | Timeline | Activities |
| --- | --- | --- |
| Phase 1: Foundation | Weeks 1–4 | • Set up BTP subaccount topology (DEV, QAS, PROD, Platform) • Configure  (IAS federation with corporate IdP, RBAC) • Set up CTMS transport nodes and routes • Subscribe Cloud ALM and configure deployment management • Create Azure DevOps project and initial Git repositories |
| Phase 2: CI/CD & Security | Weeks 5–8 | • Build CI/CD pipeline templates () for Fiori, Mobile, Integration • Integrate SAST tool (SonarQube/SonarCloud) into pipelines • Integrate SCA tool (npm audit + Snyk/Mend) into pipelines • Configure pre-commit hooks for secrets detection • Define and enforce branch policies in Azure DevOps |
| Phase 3: Monitoring & Alerting | Weeks 9–12 | • Configure SAP Alert Notification Service subscriptions and actions • Set up Automation Pilot commands for deployment tracking and health checks • Integrate Cloud ALM operations monitoring • Configure failure notification channels (email, Teams/Slack) |
| Phase 4: Optimization | Weeks 13–16 | • Implement auto-remediation (rollback) via Automation Pilot • Build compliance reporting dashboards • Onboard all remaining applications to the DevSecOps pipeline • Conduct security review and penetration testing • Document operational runbooks |

## Immediate Next Steps
Review and approve this strategy document with all stakeholders.
Confirm tool selection for SAST and SCA scanning.
Provision BTP subaccounts and platform services (Phase 1).
Assign roles and onboard team members to Azure DevOps and Cloud ALM.
Begin CI/CD pipeline development for the first pilot application (e.g., a Fiori app).
# 14. Appendix
## A. Glossary
|  |  |
| --- | --- |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |

## B. Reference Architecture Diagram

## C. Pipeline YAML Template (Azure DevOps)
[Insert reference Azure DevOps YAML pipeline template for SAP Fiori MTAR build, including SAST and SCA stages.]
## D. CTMS Configuration Reference
[Insert CTMS node and route configuration details specific to the customer landscape.]
