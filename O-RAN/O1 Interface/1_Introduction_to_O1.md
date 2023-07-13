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
