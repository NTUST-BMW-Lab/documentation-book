# Introduction to O2

![image alt](https://iili.io/HiXsroG.png)

O2 interface is an open interface within O-RAN architecture providing secured communication between the SMO and O-Cloud. It enables the management of O-Cloud infrastructure and the life cycle management of O-RAN cloudified network functions that run on O-Cloud. The O2 interface provides services that are categorized into two logical groups: **Infrastructure Management Services (IMS)** and **Deployment Management Services (DMS)**.

![image alt](https://iili.io/HiXt5OP.png)

Service Based Architecture (SBA) introduces the roles of *service producer* and *service consumer* together with standardized *service-based interfaces*. In this case, SMO functional blocks act as a *service consumer* and O-Cloud functional blocks act as a *service producer*. The O2 interface broken down into two *service-based interfaces*, i.e. O2ims and O2dms.

- **Federated O-Cloud Orchestration and Management (FOCOM).** The FOCOM is the primary consumer of services provided by the IMS. It is responsible for accounting and asset management of the cloud resources.
- **Network Function Orchestrator (NFO).** The NFO is the primary consumer of the DMS. It is responsible for orchestrating the assembly of the network functions in the O-Cloud.
- **OAM Functions.** The OAM functions are responsible for FCAPS management of O-RAN managed entities.
- **Infrastructure Management Services (IMS).** The IMS generally provides services for consumption by the FOCOM. It is responsible for management of the O-Cloud resources.
- **Deployment Management Services (DMS).** The DMS generally provides services for consumption by the NFO. It is responsible for management of network function (NF) deployments into the O-Cloud. (Each O-Cloud can consists of multiple DMS)
