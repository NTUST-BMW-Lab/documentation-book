# 3. O2dms Services
The table below summarizes the corresponding O2dms services
| Service Name | Description |
| -------- | -------- | 
| O2dms_DeploymentLifecycle Services | Service for managing the lifecycle of virtualized deployment | 

\
The table below lists the versions of the corresponding APIs

| API | API Version | 
| -------- | -------- | 
| O2dms_DeploymentLifecycle Service API | 2.10.0 |

\
Protocol Stack
| HTTP |
| :--------: |
| **TLS** |
| **TCP** |
| **IP** |
| **Data Link Layer** |
| **Physical Layer** |

\
The URI (Uniform Resource Identifier) structure that used in HTTP request from the API *service consumer* towards the API *service producer*:
```
{apiRoot}/<apiName>/<apiMajorVersion>/<apiSpecificResourceUriPart>
```
with the following component:
- The ```{apiRoot}``` inidicates the scheme ("https"), the host name, and optional port.
- The ```apiName``` indicated the API name of the service interface.
- The ```apiMajorVersion``` indicates the current major version of the API.
- The ```apiSpecificResourceUriPart``` indicates a resource URI of the API.

## 3.1. O-Cloud Deployment Life Cycle Management
The O-Cloud will provide Deployment Life Cycle Management of O-RAN Cloudified Network Functions with the following capabilities:
- **Deploy:** to deploys O-RAN Cloudified Network Functions on O-Cloud with necessary O-Cloud resources.
- **Terminate:** to terminates O-RAN Cloudified Network Functions on O-Cloud with releasing associated O-Cloud resources. 
- **Scale:** to scales functional behavior and resources of O-RAN Cloudified Network Functions to support services.
- **Heal:** to recover/mitigate the O-RAN Cloudified Network Functions's abnormal behaviour in the network.
- **Health Check:** *For Further Study*
- **Diagnostic:** *For Further Study*

![image alt](https://iili.io/Hira0tp.png)

### 3.1.1. O2dms_DeploymentLifecycle Services
**Instantiate Network Function Deployment Procedure**
```sequence
Note right of SMO: Sub-procedure: Creation of "Individual NF Deployment" resource
Note right of SMO: Precondition: None
SMO->O Cloud: 1. <<O2dms>> POST (CreateVnfRequest)
Note left of O Cloud: 2. Check availability
O Cloud->SMO: 3. <<O2dms>> 201 Created (VnfInstance)
O Cloud-->SMO: 4. <<O2dms>> Send VnfIdentifierCreationNotification
Note right of SMO: Postcondition: NF Deployment is created and in NOT_INSTANTIATED state
Note right of SMO: Sub-procedure: Instantiation of NF Deployment
Note right of SMO: Precondition: NF Deployment is created and in NOT_INSTANTIATED state
SMO->O Cloud: 5. <<O2dms>> POST (InstantiateVnfRequest)
Note left of O Cloud: 6. Create resource
O Cloud->SMO: 7. <<O2dms>> 202 Accepted ()
O Cloud-->SMO: 8. <<O2dms>> Send VnfLcmOperationOccurenceNotification(STARTING)
Note left of O Cloud: 9. Check resources
O Cloud-->SMO: 10. <<O2dms>> Send VnfLcmOperationOccurenceNotification(PROCESSING)
Note left of O Cloud: 11. NF Deployment instantiation
SMO->O Cloud: 12. <<O2dms>> GET
O Cloud->SMO: 13. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=PROCESSING)
Note left of O Cloud: 14. Instantiation completed
O Cloud-->SMO: 15. <<O2dms>> Send VnfLcmOperationOccuranceNotification(COMPLETED)
SMO->O Cloud: 16. <<O2dms>> GET
O Cloud->SMO: 17. <<O2dms>> 200 OK
Note right of SMO: Postcondition: NF Deployment is instantiated and in INSTANTIATED state
```

**Terminate Network Function Deployment Procedure**
```sequence
Note right of SMO: Sub-procedire: Termination of NF Deployment
Note right of SMO: Precondition: NF Deployment is in INSTANTIATED state
SMO->O Cloud: 1. <<O2dms>> POST (TerminateVnfRequest)
Note left of O Cloud: 2. Create resource
O Cloud->SMO: 3. <<O2dms>> 202 Accepted ()
O Cloud-->SMO: 4. <<O2dms>> Send VnfLcmOperationOccuranceNotification(STARTING)
Note left of O Cloud: 5. Check conditions
O Cloud-->SMO: 6. <<O2dms>> Send VnfLcmOperationOccuranceNotification(PROCESSING)
Note left of O Cloud: 7. NF Deployment termination
SMO->O Cloud: 8. <<O2dms>> GET
O Cloud->SMO: 9. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=PROCESSING)
Note left of O Cloud: 10. Termination completed
O Cloud-->SMO: 11. <<O2dms>> Send VnfLcmOperationOccuranceNotification(COMPLETED)
SMO->O Cloud: 12. <<O2dms>> GET
O Cloud->SMO: 13. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=COMPLETED)
Note right of SMO: Postcondition: NF Deployment is terminated (NOT_INSTANTIATED state)
Note right of SMO: Sub-procedure: Deletion of "Individual NF Deployment" resource
Note right of SMO: Precondition: NF Deployment in NOT_INSTANTIATED state
SMO->O Cloud: 14. <<O2dms>> DELETE
Note left of O Cloud: 15. Delete resource
O Cloud->SMO: 16. <<O2dms>> 204 No Content
O Cloud-->SMO: 17. <<O2dms>> Send VnfIdentifierDeletionNotification
Note right of SMO: Postcondition: NF Deployment is removed
```

**Query Network Function Deployment Information Procedure**
```sequence
Note right of SMO: Precondition: The NF Deployment is created None
Note left of O Cloud: Query Information about multiple NF Deployment
SMO->O Cloud: 1. <<O2dms>> GET
O Cloud->SMO: 2. <<O2dms>> 200 OK (VnfInstance)
Note left of O Cloud: Read Information about individual NF Deployment
SMO->O Cloud: 3. <<O2dms>> GET
O Cloud->SMO: 4. <<O2dms>> 200 OK (VnfInstance)
Note right of SMO: Postcondition: SMO has the queried information about NF Deployment
```

**On-demand Healing Procedure**
```sequence
Note right of SMO: Precondition: NF Deployment is in INSTANTIATED state
SMO->O Cloud: 1. <<O2dms>> POST
Note left of O Cloud: 2. Create resource
O Cloud->SMO: 3. <<O2dms>> 202 Accepted ()
O Cloud-->SMO: 4. <<O2dms>> Send VnfLcmOperationOccurenceNotification(STARTING)
Note left of O Cloud: 5. Check resources
O Cloud-->SMO: 6. <<O2dms>> Send VnfLcmOperationOccurenceNotification(PROCESSING)
Note left of O Cloud: 7. NF Deployment healing
SMO->O Cloud: 8. <<O2dms>> GET
O Cloud->SMO: 9. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=PROCESSING)
Note left of O Cloud: 10. Healing completed
O Cloud-->SMO: 11. <<O2dms>> Send VnfLcmOperationOccurenceNotification(COMPLETED)
SMO->O Cloud: 12. <<O2dms>> GET
O Cloud->SMO: 13. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=COMPLETED)
Note right of SMO: Postcondition: NF Deployment is healed and in INSTANTIATED state
```

**Auto-healing Procedure**
```sequence
Note right of SMO: Precondition: NF Deployment is in INSTANTIATED state
Note left of O Cloud: 1. Detects healing condition
Note left of O Cloud: 2. Create resource
O Cloud-->SMO: 3. <<O2dms>> Send VnfLcmOperationOccurenceNotification(STARTING)
Note left of O Cloud: 4. Check resources
O Cloud-->SMO: 5. <<O2dms>> Send VnfLcmOperationOccurenceNotification(PROCESSING)
Note left of O Cloud: 6. NF Deployment auto-healing
SMO->O Cloud: 7. <<O2dms>> GET
O Cloud->SMO: 8. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=PROCESSING)
Note left of O Cloud: 9. Auto-healing completed
O Cloud-->SMO: 10. <<O2dms>> 200 OK VnfLcmOperationOccurenceNotification(COMPLETED)
SMO->O Cloud: 11. <<O2dms>> GET
O Cloud->SMO: 12. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=COMPLETED)
Note right of SMO: Postcondition: NF Deployment is automatically healed and in INSTANTIATED state
```

**Scaling NF Deployment based on Management Request Procedure**
```sequence
Note right of SMO: Precondition: NF Deployment is in INSTANTIATED state
SMO->O Cloud: 1. <<O2dms>> POST (ScaleVnfRequest)
O Cloud->SMO: 2. <<O2dms>> POST (ScaleVnfToLevelRequest)
Note left of O Cloud: 3. Create resource
O Cloud->SMO: 4. <<O2dms>> 202 Accepted()
O Cloud-->SMO: 5. <<O2dms>> Send VnfLcmOperationOccurenceNotification(STARTING)
Note left of O Cloud: 6. Check resources
O Cloud-->SMO: 7. <<O2dms>> Send VnfLcmOperationOccurenceNotification(PROCESSING)
Note left of O Cloud: 8. NF Deployment scaling
SMO->O Cloud: 9. <<O2dms>> GET
O Cloud->SMO: 10. <<O2dms>> 200 OK (VnfLcmOpOcc:operationnState=PROCESSING)
Note left of O Cloud: 11. Scaling completed
O Cloud-->SMO: 12. <<O2dms>> Send VnfLcmOperationOccurenceNotification(COMPLETED)
SMO->O Cloud: 13. <<O2dms>> GET
O Cloud->SMO: 14. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=COMPLETED)
Note right of SMO: Postcondition: NF Deployment is scaled and in INSTANTIATED state
```

**Auto-scaling Procedure**
```sequence
Note right of SMO: Precondition: NF Deployment is in INSTANTIATED state
Note left of O Cloud: 1. O Cloud DMS detects a scaling condition
Note left of O Cloud: 2. Create resource
O Cloud-->SMO: 3. <<O2dms>> Send VnfLcmOperationOccurenceNotification(STARTING)
Note left of O Cloud: 4. Check resources
O Cloud-->SMO: 5. <<O2dms>> Send VnfLcmOperationOccurenceNotification(PROCESSING)
Note left of O Cloud: 6. NF Deployment auto-scaling
SMO->O Cloud: 7. <<O2dms>> GET
O Cloud->SMO: 8. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=PROCESSING)
Note left of O Cloud: 9. Auto-scaling completed
O Cloud-->SMO: 10. <<O2dms>> Send VnfLcmOperationOccurenceNotification(COMPLETED)
SMO->O Cloud: 11. <<O2dms>> GET
O Cloud->SMO: 12. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=COMPLETED)
Note right of SMO: Postcondition: NF Deployment is automatically scaled and in INSTANTIATED state
```

**Change External Connectivity of an NF Deployment Procedure**
```sequence
Note right of SMO: Precondition: NF Deployment is in INSTANTIATED state
SMO->O Cloud: 1. <<O2dms>> POST (ChangeExtVnfConnectivityRequest)
Note left of O Cloud: 2. Create resource
O Cloud->SMO: 3. <<O2dms>> 202 Accepted ()
O Cloud-->SMO: 4. <<O2dms>> Send VnfLcmOperationOccurenceNotification(STARTING)
Note left of O Cloud: 5. Check resources
O Cloud-->SMO: 6. <<O2dms>> Send VnfLcmOperationOccurenceNotification(PROCESSING)
Note left of O Cloud: 7. Changing of NF Deployment external connectivity
SMO->O Cloud: 8. <<O2dms>> GET
O Cloud->SMO: 9. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=PROCESSING)
Note left of O Cloud: 10. Changing external connectivity completed
O Cloud-->SMO: 11. <<O2dms>> Send VnfLcmOperationOccurenceNotification(COMPLETED)
SMO->O Cloud: 12. <<O2dms>> GET
O Cloud->SMO: 13. <<O2dms>> 200 OK (VnfLcmOpOcc:operationState=COMPLETED)
Note left of O Cloud: Postcondition: NF Deployment external connectivity has been changed
```

### 3.1.2. O2dms_DeploymentLifecycle Service API
The O2dms_DeploymentLifecycle enables an *API consumer* (SMO) to invoke lifecycle management operations of NF Deployment towards the O-Cloud DMS.

The figure below shows the resource URI structure defined for the O2dms_DeploymentLifecycle Service API.

![image alt](https://iili.io/Hs3VbIV.png)

#### **a. VNF instances** 
- URI: ```/vnf_instances```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Create a new "Individual VNF instance" resource which represents an individual NF Deployment | 
| GET | Query multiple NF Deployment | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* | 

#### **b. Individual VNF instances** 
- URI: ```/vnf_instances/{vnfInstanceId}```

| HTTP Method | Description | 
| -------- | -------- |
| POST | *not supported* | 
| GET | Read information about an individual NF Deployment | 
| PUT | *not supported* | 
| PATCH | Modify information of an individual NF Deployment | 
| DELETE | Delete an "Individual VNF instance" resource representing an individual NF Deployment | 

#### **c. Instantiate VNF task** 
- URI: ```/vnf_instances/{vnfInstanceId}/instantiate```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Instantiate an NF Deployment | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* | 

#### **d. Terminate VNF instances** 
- URI: ```/vnf_instances/{vnfInstanceId}/terminate```

| HTTP Method | Description | 
| -------- | -------- |
| POST |  Terminate an NF Deployment instance | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **e. Heal VNF task** 
- URI: ```/vnf_instances/{vnfInstanceId}/heal```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Heal an NF Deployment instance | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **f. Scale VNF task** 
- URI: ```/vnf_instances/{vnfInstanceId}/scale```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Scale an NF Deployment instance | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **g. Scale VNF to level task** 
- URI: ```/vnf_instances/{vnfInstanceId}/scale_to_level```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Scale an NF Deployment instance to a target level | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **h. Change external VNF connectivity task** 
- URI: ```/vnf_instances/{vnfInstanceId}/change_ext_conn```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Change the external connectivity of an NF Deployment instance | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **i. VNF LCM operation occurrences** 
- URI: ```/vnf_lcm_op_occs```

| HTTP Method | Description | 
| -------- | -------- |
| POST | *not supported* | 
| GET | Query information about multiple lifecycle management operation occurrences of NF Deployments | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **j. Individual VNF LCM operation occurrence** 
- URI: ```/vnf_lcm_op_occs/{vnfLcmOpOccId}```

| HTTP Method | Description | 
| -------- | -------- |
| POST | *not supported* | 
| GET | Query information about an "Individual VNF LCM operation occurrence" corresponding to the lifecycle of an NF Deployment | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **k. Retry operation task** 
- URI: ```/vnf_lcm_op_occs/(vnfLcmOpOccId}/retry```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Retry a lifecycle management operation occurrence of an NF Deployment | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **l. Rollback operation task** 
- URI: ```/vnf_lcm_op_occs/(vnfLcmOpOccId}/rollback```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Rollback a lifecycle management operation occurrence of an NF Deployment | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **m. Fail operation task** 
- URI: ```/vnf_lcm_op_occs/(vnfLcmOpOccId}/fail```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Fail a lifecycle management operation occurrence of an NF Deployment | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **l. Cancel operation task** 
- URI: ```/vnf_lcm_op_occs/(vnfLcmOpOccId}/cancel```

| HTTP Method | Description | 
| -------- | -------- |
| POST | Cancel a lifecycle management operation occurrence of an NF Deployment | 
| GET | *not supported* | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **n. Subscriptions** 
- URI: ```/subscriptions```

| HTTP Method | Description | 
| -------- | -------- |
| POST |  Subscribe to NF Deployment lifecycle notifications | 
| GET | Query information about multiple subscriptions to NF Deployment lifecycle notifications | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | *not supported* |

#### **o. Individual subscription** 
- URI: ```/subscriptions/{subscriptionId}```

| HTTP Method | Description | 
| -------- | -------- |
| POST | *not supported* | 
| GET | Read an "Individual subscription" resource which represents information about a subscription to NF Deployment lifecycle notifications | 
| PUT | *not supported* | 
| PATCH | *not supported* | 
| DELETE | Terminate a subscription to NF Deployment lifecycle notifications |
