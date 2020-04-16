# CX Contact: Multicloud/Azure Design Specification

Overview
========

CX Contact is a logical evolution of two Outbound platforms, Engage Service and
Genesys Outbound for Pure Engage Cloud. CX Contact brings functionality and
features from both these platforms to a single service which provides
world-class multi channel Outbound contact capabilities both on-premise and in
Cloud. CX Contact manages all aspects of Outbound Campaigns life-cycle.

CX Contact is built following "cloud first" approach using all tools, principles
and methodologies which are applied for cloud service (full deployment and
upgrades automation, full regression, blue/green upgrades model). All components
of Cloud Contact are represented as individual microservies, each executed in
Docker containers under N+1 horizontal scaling model principles.

This document on Confluence: [CX Contact Multicloud/Azure Design Specification](https://intranet.genesys.com/pages/viewpage.action?pageId=133617399)

Reference to HLA: [CX Contact Microservice
Group](https://intranet.genesys.com/display/MIC/CX+Contact+Microservice+Group)

Reference to HLD: [CX Contact - High Level
Design](https://intranet.genesys.com/display/RP/CX+Contact+-+High+Level+Design)

Reference to CX Contact source code
repository: <https://git.genesyslab.com/cloudcon/>

Reference to K8s deployment source code
repository: <https://github.com/genesysengage/cxcontact-pipeline>

Azure Architecture Specification updates
----------------------------------------

[Azure Architecture
Specification](https://genesyslab.sharepoint.com/:w:/r/sites/GenesysAzure/_layouts/15/Doc.aspx?sourcedoc=%7B9d41c77e-525b-477a-b902-4481554737e4%7D&action=edit&wdPid=567a7fe7&cid=88ac2caf-608c-41ca-8af0-d46682a209cf) updated
as follows:

1.  Services Table (what services you have)  - Updated.
2.  Cross-region Table (what services are cross region) - Updated. 
3.  NameSpaces Table (what namespaces are you in or have) - Updated.
4.  Data Store Table (what datastores you are using) - Updated.
5.  File Storage Table (file and disk your service is going to use) - Updated.
    

Unit (region) focused architecture diagram
------------------------------------------
![Unit Diagram](https://staszg.github.io/azure/cxcontact-in-azure-one-region.png)

Notes to the above diagram:

1.  Elasticsearch in CXC namespace is required for CX Contact Outbound Analytics
    functionality. This ES is different from the common service for logging and
    metrics ES deployed centrally.

2.  CXC Back-End Ingress is required for OCS from Voice Microservice to reach
    back-end CX Contact components. Please note, that Back-End Ingress is
    absolutely required for on-premise deployments (where OCS is not in same K8s
    cluster as CX Contact). We prefer to keep architecture the same between
    Azure and premise.

Unit group (multi-region) focused architecture diagram
------------------------------------------------------
![Unit Diagram](https://staszg.github.io/azure/cxcontact-in-azure-multi-region.png)

Connectivity Control
====================

Within Cluster
--------------

CX Contact connects to GWS and (optionally, for EMail and SMS Campaigns support)
to Nexus.

### Network Policy for GWS

No Network Policy is defined.

### Network Policy for Nexus

No Network Policy is defined.

### Back-end Ingress

Link: <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/ingress-backend.yaml>

External to VNet
----------------

### Ingress

Link: <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/ingress.yaml>

### Services

Links:

-   <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/amark-app-service.yaml>

-   <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/amark-ui-service.yaml>

-   <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/campaign-manager-service.yaml>

-   <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/compliance-service.yaml>

-   <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/dial-manager-service.yaml>

-   <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/job-scheduler-service.yaml>

-   <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/list-builder-service.yaml>

-   <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/list-manager-service.yaml>

Vnet to Vnet
------------

CX Contact is not using Vnet to Vnet connectivity.

With managed services
---------------------

CX Contact utilizes the following Azure Managed Services:

-   Postgre SQL - contains Contact and Suppression Lists used by Outbound
    Campaigns executed by On-Tenant OCS.
-   Redis Cache - contains user sessions cached data and dialing history data
    for Outbound compliance rules.
-   Azure Files - stores import and export files for Contact and Suppression
    Lists. Stores Compliance Data files downloaded from Compliance Data
    Provider.

Link: <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/values.yaml>

Table of connections for CX Contact
-----------------------------------

| **\#** | **Client** | **Network Type** | **Client Network** | **Server**                   | **Server Network** | **Protocol** | **Port to accept connection** | **Description**                       | **Artifact Links1** |
|--------|------------|------------------|--------------------|------------------------------|--------------------|--------------|-------------------------------|---------------------------------------|---------------------|
| **1**  | CX Contact | K8s              | K8s Network        | GWS                          | K8s Network        | TCP          | 80                            | HTTP APIs                             |                     |
| **2**  | CX Contact | K8s              | K8s Network        | Nexus                        | K8s Network        | TCP          | 80                            | HTTP APIs                             |                     |
| **3**  | CX Contact | Vnet Service     | K8s Network        | Managed REDIS                | Vnet Network       | TCP          | 6379                          | REDIS API Access for the Service      |                     |
| **4**  | CX Contact | Vnet Service     | K8s Network        | Managed PostgreSQL           | Vnet Network       | TCP          | 5432                          | Postgre access for the Service        |                     |
| **5**  | CX Contact | K8s              | K8s Network        | Elastic Search for Analytics | K8s Network        | TCP          | 9200                          | Elastic Search for Outbound Analytics |                     |
| **6**  | Prometheus | Vnet Service     | K8s Network        | CX Contact                   | K8s Network        | TCP          | 81                            | Scrape for metrics                    |                     |
| **7**  | OCS        | K8s              | K8s Network        | CX Contact                   | K8s Network        | TCP          | 80                            | HTTP APIs                             |                     |
| **8**  | CX Contact | K8s              | K8s Network        | Remote SFTP Server           | External           | TCP          | 22                            | SFTP                                  |                     |
| **9**  | CX Contact | K8s              | K8s Network        | Compliance Data Provider     | External           | TCP          | 443                           | HTTP APIs                             |                     |

1 - Network policy links will be provided later, once we develop them.

Secrets
=======

The following CX Contact artifacts are stored as Kubernetes secrets:

-   CX Contact private and public keys for PGP
-   GWS Client Credentials (client name, client secret)
-   Person username and password for Config Server access
-   Credentials for access to password-protected Redis (Access Key)

Link: <https://github.com/genesysengage/cxcontact-pipeline/blob/master/cxcontact/templates/secret.yaml>

Other Service's Resources
=========================

N/A

Upgrade Strategy
================

Rolling upgrade strategy is presently used to upgrade CX Contact nodes (based on
K8s Deployments default strategy).

HA Strategy
===========

N/A

Containerization HLA Exceptions
===============================

Docker Exceptions
-----------------

1.  Exceptions granted for requirement 1c.

    1.  CX Contact UI container includes Nginx deployed on top of the Base
        Docker Image (Nginx is used to serve static content). Nginx is installed
        with **yum** with hash sums checked. Nginx headers will be stripped by
        Akamai and by Azure K8s ingress in Azure.

    2.  SFTP Client will be installed on top of the Base Docker Image (approval
        granted via  [RDA-185](https://jira.genesys.com/browse/RDA-185))

    3.  Sshpass module will be installed on top of the Base Docker Image
        (approval granted via 
        [RDA-186](https://jira.genesys.com/browse/RDA-186))

2.  Exception granted for requirement 1d. One of CX Contact container images 
    (List Builder) exceeds 1 GB in size. This is due to compliance data files
    baked into the container. Exception granted based on the fact that this is
    one image out of 8, and on average CX Contact container size is less then 1
    GB.

Kubernetes Exceptions
---------------------

None

CD Pipeline HLA Exceptions
==========================

None at this time.
