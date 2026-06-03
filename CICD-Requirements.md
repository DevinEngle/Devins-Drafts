# CI/CD Connector Requirements for Rapid Clients

_Updated June 2, 2026_

## Summary

Rapid's connector platform relies on a CI/CD (Continuous Integration/Continuous Deployment) process to deliver bug fixes, compliance changes, and new functionality to your Rapid connectors.

To receive automated updates at no additional cost, clients must allow the deployment access outlined in this document.

## Table of Contents

- [Billing Policy](#billing-policy)
- [Technical Requirements](#technical-requirements)
- [Installation Requirements](#installation-requirements)
- [Upgrade Requirements](#upgrade-requirements)
- [Offline Station Requirements](#offline-station-requirements)
- [Replication Limitations](#replication-limitations)
- [IT Checklist](#it-checklist)
- [Common Causes of Failed Deployments](#common-causes-of-failed-deployments)
- [Questions](#questions)

## Updates and Notification Policy

The monthly connector subscription fee includes periodic updates as they become available. 

If an automated deployment is blocked due to client security restrictions and Rapid staff are required to perform manual installation, troubleshooting, coordination with client IT, or other deployment-related activities outside the standard CI/CD process, that work will be billed at current T&M rates.

Our goal is to work with your IT team to establish the necessary access once so that future updates can be delivered automatically, securely, and without additional cost.

Rapid will attempt to provide at least 24 hours' notice prior to scheduled connector upgrades, along with an estimated deployment window. From time to time, an important update may need to be deployed on short notice or without notice. 

## Technical Requirements

To support automated upgrades, Rapid's Azure-based deployment platform must be permitted to:

- Connect to the server for the purpose of deploying Rapid connector updates.
- Write, replace, and update files within the Rapid connector installation directories.
- Stop and restart Rapid connector-related services when required by an upgrade.
- Operate without being blocked, quarantined, rolled back, or otherwise interrupted by endpoint protection, anti-malware, application control, or similar security software.
- Complete deployments without requiring manual approval or intervention for each release.

#### Network Requirements

- Communication with Azure DevOps deployment services, including <https://dev.azure.com>, must be permitted through firewalls, web filters, proxy servers, and endpoint security products.

#### PowerShell Requirements

- PowerShell script execution must be enabled for Rapid deployment and upgrade processes.
- Endpoint protection, application control, and anti-malware products must not block approved Rapid deployment scripts.

## Installation Requirements

1. Connectors are installed on the Counterpoint server.
   - Exception: The _Rapid Custom Task Agent_ generally is installed on the Counterpoint server but can be installed on the SQL server in certain use cases when SQL Server is not installed on the Counterpoint server.
        
2. A version of Counterpoint and SQL Server must be installed that meets the minimum requirements for the specific connector.

3. Administrator-level connector platform access and credentials must be provided.

4. Administrator-level server access must be provided for the initial installation.

5. Initial installation of the CI/CD connector will be scheduled with the client based on availability.
   - All users must be logged out of Counterpoint during installations unless otherwise specified by Rapid.
   - Installations will typically be scheduled after the close of business.
   - Exceptions: The _Rapid CP API_ and _Rapid Custom Task Agent_ can typically be installed during business hours.

## Upgrade Requirements

1. CI/CD connector upgrades will be deployed automatically through Rapid's deployment platform.

2. All users must be logged out of Counterpoint during upgrades unless otherwise specified by Rapid.

3. Upgrades that include database schema changes will typically be scheduled after the close of business.

4. Rapid will attempt to provide at least 24 hours' notice prior to scheduled connector upgrades, along with an estimated deployment window. From time to time, an important update may need to be deployed on short notice or without notice.

## Offline Station Requirements

#### For Clients Using Offline Stations

1. Installations and upgrades require the server database to be provisioned and rebuilt on all offline stations.

2. During both the initial installation and all upgrades, all offline stations must be powered on.

3. If offline stations are not powered on, the database rebuild will be queued and will install the next time the computer is powered on.

4. Users must not use Counterpoint while a rebuild is in progress.

## Replication Limitations

#### For Clients Using Replication

- Certain CI/CD connectors are not currently supported in multi-site replication environments.
- If you have a hub server and one or more remote servers, please consult with Rapid before requesting installation of a CI/CD connector.

## IT Checklist

Before installation or upgrade, verify that:

- Administrator access has been provided to Rapid.
- Azure DevOps communication is permitted.
- PowerShell execution is enabled.
- Endpoint protection, anti-malware, and application control software will not block Rapid deployment activity.
- All required Counterpoint users will be logged out during the deployment window.
- Offline stations (if applicable) will be powered on and available.

## Common Causes of Failed Deployments

The following are the most common reasons automated deployments fail:

- Azure DevOps traffic blocked by firewall, proxy, or web filtering rules.
- Endpoint protection or anti-malware software quarantining deployment files.
- PowerShell execution disabled or restricted.
- Connector services cannot be stopped or restarted.
- Insufficient administrative permissions.
- Users actively logged into Counterpoint during deployment.
- Offline stations unavailable during deployment.
- Application control software preventing Rapid deployment processes from running.

## Questions

Please contact the Rapid Programming Team if your IT staff require additional technical information regarding the deployment process or access requirements.
