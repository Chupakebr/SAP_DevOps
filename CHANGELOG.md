# Changelog

All notable changes to this document are tracked here.

## [4.0] – November 2026
### Changed
- Scoped the strategy to three applications only: SSAM (Mobile/MDK), SAP Fiori, and SAP Build Work Zone
- CI/CD is now exclusively the SAP BTP Continuous Integration & Delivery service, limited to its supported pipeline scenarios (Cloud Foundry Environment; SAP Fiori for ABAP Platform)
- Corrected SSAM: no SAP BTP CI/CD scenario exists for MDK — transport is via SAP Content Agent → cTMS with Cloud ALM governance; added architecture diagram and configuration tutorial reference
- Fiori mapped to the two supported CI/CD scenarios for deployment to SAP BTP and on-premise S/4HANA
- Fixed Markdown formatting throughout (lists, code blocks, title block, cross-references)

### Removed
- All ABAP-specific content: S/4HANA PCE Extensions chapter, gCTS, ATC checks, ABAP transport/CTS roles
- Out-of-scope applications: SAP Integration Services, SAP Build Process Automation, SAP Datasphere
- Azure DevOps Pipelines as a CI/CD option (Azure DevOps retained for Git source control only)
- Document Control > Revision History section
- DRAFT and SAP Internal classification markings

## [3.3] – November 2026
### Added
- SIEM integration section (9.4) with log sources, correlation rules, and supported platforms
- S/4HANA PCE Extensions chapter (5.3) covering BAS connectivity, gCTS, Fiori BSP deployment
- ABAP Code Quality section with ATC checks and unit test execution
- Deployment Roles & Authorization Matrix (7.4) for BTP and S/4HANA PCE
- 9.4.3 Supported SIEM platforms table (Microsoft Sentinel, Splunk, IBM QRadar, SAP ETD)

## [3.2] – November 2026
### Added
- BAS → S/4HANA PCE deployment guidance
- Deployment roles matrix
- SIEM integration outline
- SSAM MDK pipeline clarification

## [1.0] – November 2026
### Changed
- Stakeholder review comments incorporated (H. Madania)

## [0.1] – October 2026
### Added
- Initial draft (R. Rodionov)
