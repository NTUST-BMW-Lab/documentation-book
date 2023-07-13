# 1. Introduction

**O-RAN Management Services (MnS)** offer capabilities to manage and orchestrate O-RAN Network Function. 

![image alt](https://iili.io/HsY9j6J.png)

**O-RAN Network Functions (NFs)** include:
- Near-Real-Time Radio Intelligent Controller (Near-RT RIC)
- O-RAN Central Unit - Control Plane (O-CU-CP)
- O-RAN Central Unit - User Plane (O-CU-UP)
- O-RAN Distributed Unit (O-DU)
- O-RAN Radio Unit (O-RU)

**Service Management and Orchestration Framework (SMO)** is responsible for the management and orchestration of the Network Functions (NFs) under its span of control. The SMO shall provide:
- Integration fabric to enables interoperation and communication between the SMO and the Management Functions (MnFs)
- Data services to provide efficient data collection and storage capabilitis.

O-RAN defines 3 control loops:
- Loop 1: In the O-DU for resource scheduling (<10ms)
- Loop 2: In the Near-RT RIC and O-CU for resource optimization (10ms to 1s)
- Loop 3: In the SMO for ML training, trending, and orchestration (>1s)

The **O1 interface** is used for **Operation and Maintanance (OAM)** functions between the O-RAN SMO and O-RAN NFs (except O-RU).

# References
- [O-RAN.WG3.O1-Interface-for-Near-RT-RIC-R003-v01.00](https://orandownloadsweb.azurewebsites.net/specifications)
- [O-RAN.WG5.O-DU-O1.0-R003-v07.00](https://orandownloadsweb.azurewebsites.net/specifications)
- [O-RAN.WG5.O-CU-O1.0-R003-v05.00](https://orandownloadsweb.azurewebsites.net/specifications)
- [O-RAN.WG10.O1-Interface.0-R003-v10.00](https://orandownloadsweb.azurewebsites.net/specifications)
- [O-RAN.WG10.OAM-Architecture-R003-v09.00](https://orandownloadsweb.azurewebsites.net/specifications)
- [3GPP TS 28.537 version 17.2.0 Release 17](https://www.etsi.org/deliver/etsi_ts/128500_128599/128537/17.02.00_60/ts_128537v170200p.pdf)
- [3GPP TS 28.532 version 16.4.0 Release 16](https://www.etsi.org/deliver/etsi_ts/128500_128599/128532/16.04.00_60/ts_128532v160400p.pdf)
- [3GPP TS 28.545 version 16.1.0 Release 16](https://www.etsi.org/deliver/etsi_ts/128500_128599/128545/16.01.00_60/ts_128545v160100p.pdf)
