# <center><i class="fa fa-edit"></i> FAPI </center>
## Introduction
![](https://hackmd.io/_uploads/HyHh5e2v3.png)
![](https://hackmd.io/_uploads/Sk1Ong3w2.png)
![](https://hackmd.io/_uploads/SJF-dH5K2.png)

- O-DU Low: O-DU High communicates with O-DU Low on the FAPI interface. FAPI messages shown below are supported, as per FAPI interface files shared by Intel:
  - **P5 messages - PHY mode control interface**
    - PARAM.request/PARAM.response
    - CONFIG.request/CONFIG.response
    - START.request
    - STOP.request
    - STOP.indication
  - **P7 messages - Main data path interface**
    - DL_TTI.request
    - UL_TTI.request
    - SLOT.indication
    - UL_DCI.request
    - TX_Data.request
    - RX_Data.indication
    - CRC.indication
    - UCI.indication
    - RACH.indication
  - **P19: RF frontend control**
      - For beamforming using active antenna arrays
      - Not present in 4G FAPI

![](https://hackmd.io/_uploads/rk1vuqiF2.png)
![](https://hackmd.io/_uploads/BynOu9jF2.png)

- **FAPI Message Nomenclature**:
    - **MAC (L2/L3)** send `.request` messages to PHY
    - **PHY** replies with `.response` messages
    - **PHY** can send `.indication` messages asynchronously to MAC
    - Examples:
        - **MAC -> PHY**: `CONFIG.request`
        - **PHY -> MAC**: `CONFIG.response`
        - **PHY -> MAC**: `SLOT.indication`

## API Message Order
![](https://hackmd.io/_uploads/BkmwJU5tn.png)
![](https://hackmd.io/_uploads/HyEHGLqK3.png)

![](https://hackmd.io/_uploads/Syq3Tr9t3.png)


# <center><i class="fa fa-edit"></i> SU-MIMO & MU-MIMO FAPI Parameters</center>
###### tags: `Internship` `Daily Report` `PHY Interface` `FAPI`
:::success
**Learning Objective** :heavy_check_mark: 
* Learn and document the difference of both MIMO parameters
:::
:::info
**Resources** :package: 
1. [5G FAPI: PHY API Specification](https://www.smallcellforum.org/work-items/fapi/#:~:text=The%20latest%20FAPI%20specifications%20are,(P5)%20interface%20%5BSCF222%5D) 
2. [O-DU Gerrit Repo](https://gerrit.o-ran-sc.org/r/admin/repos/o-du/l2,general)
:::


DL and UL SU-MIMO and MU-MIMO (including Massive-MIMO) is done based on **[SRS](https://hackmd.io/@ra-jordhie/Phylayer-Background#63-Sounding)**/**[CSI](https://hackmd.io/@ra-jordhie/Phylayer-Background#62-Channel-State-Information-CSI)**.
## 1. Spatial Multiplexing
FAPI 2.2.6 Spatial Multiplexing
:::danger
**5G FAPI: PHY API Specification**
- **Issue date: 16 May 2022**
- **Version: 222.5.0**
:::

### 1.1 Precoding and Beamforming Based on Semistatic Tables
FAPI 2.2.6.1 Precoding and Beamforming Based on Semistatic Tables
![](https://hackmd.io/_uploads/rJ0TqWZK3.png)
- For semi-static spatial multiplexing, the precoding and beamforming is defined in 
tables loaded into the PHY at configuration time. In slot messages a precoding index 
and/or beamforming index is included in each PDU.
- **[Precoding](https://hackmd.io/@ra-jordhie/Phylayer-Background#6-Precoding)** and **Digital Beamforming** can be viewed as a ==cascade of two matrix multiplications PM and DB== as depicted in Figure 2-45. For precoding operation an index to a pre-stored precoding matrix PM(idx) is specified in the message. Digital beamforming is represented by a matrix DB. For efficient messaging columns of DB are picked from a pre-stored digital beamforming table (DBT) as illustrated in Figure 2-46. It suffices to specify the index of the beam to be applied for each input port in the message. Baseband ports shown in Figure 2-45 are logical entities that may be further processed by a digital frontend (DFE) unit. Within the context of 5GNR a baseband port typically corresponds to component carrier that is handled by a specific RF path (i.e. TXRU). 
![](https://hackmd.io/_uploads/HJoE3-Ztn.png)

### 1.2 Precoding based on PHY-determined Weights

2.2.6.2 Precoding based on PHY-determined Weights
- Alternatively, downlink precoders/uplink combiners can be determined in PHY by collecting SRS samples which can be used to form uplink codebook-based combiners.  SRS samples can also be used to form downlink precoders and non-codebook uplink combiners.

![](https://hackmd.io/_uploads/Sytj2Z-F3.png)
![](https://hackmd.io/_uploads/H1wEkzWFn.png)
![](https://hackmd.io/_uploads/ByiryfWt3.png)
![](https://hackmd.io/_uploads/Sk0Lkz-F3.png)

- Digital precoding (for downlink) is depicted in Figure 2-47, and digital combining (for uplink) is depicted in Figure 2-48. Downlink precoder determination and uplink combiner determination are based on a signalling of Ues and layers (see Table 2–3)for joint spatial multiplexing over the same set of RBs and symbols.
- The set of all the RBs and symbols in a slot over which the same set of UE channels and layers is scheduled is denoted as a “MU-MIMO Group”, as illustrated in Figure 2-49.
- Baseband ports shown in Figure 2-47 and Figure 2-48 are logical entities that may be additionally processed by a digital frontend (DFE) unit. Within the context of 5GNR a baseband port typically corresponds to component carrier that is handled by a specific RF path (i.e. TXRU). 

### 1.3 Downlink Spatial Stream Limits
- The downlink spatial stream limits refer to the maximum number of downlink spatial streams that can be supported by a PHY. The maximum number of DL streams is subject to L1 capability, which includes parameters such as maxNumberDlSpatialStreams, maxNumCarriersBWLayersProductDL, and maxNumCarriersBWAntennasProductDL. Downlink spatial streams may correspond to downlink data flows, as indexed by "c_eAxC_ID". The MAC assigns DL spatial stream indices between 0 and [(max# downlink streams)-1], so that each resource element (RE) and symbol output from a precoder is unambiguously associated with a unique DL stream index. The DL stream indices are associated with different PDUs:
    -  PDCCH PDU: one stream index per DCI.
    -  PDSCH PDU: one stream index per layer.
    -  CSI-RS PDU: one stream index per cdm port in a cdm group.
    -  SSB PDU: one stream index.

### 1.4 Uplink Spatial Stream Limits
- The uplink spatial stream limits refer to the maximum number of uplink spatial streams that can be supported by a PHY. The maximum number of UL streams is subject to L1 capability, which includes parameters such as maxNumberUlSpatialStreams, maxNumCarriersBWLayersProductUL, and maxNumCarriersBWAntennasProductUL. Uplink spatial streams may correspond to uplink data flows, as indexed by "c_eAxC_ID" in section 4.5.4.1. The MAC assigns UL spatial stream indices between 0 and [(max # uplink streams)-1], so that each resource element (RE) and symbol output from a combiner is unambiguously associated with at least one UL channel PDU. The UL stream indices are associated with different PDUs:
    -  PUCCH PDU: (at least) one stream index.
    -  PUSCH PDU: (at least) one stream index per layer.
    -  PRACH PDU: as many stream indices as logical channels (at least one).
    -  SRS PDU: as many stream indices as logical channels, except if SRS PDUs are associated with RAW ports. In the latter case, MAC does not assign UL spatial streams to the SRS PDU

## 2. PARAM Configuration Messages
3.3.1 PARAM
:::danger
**5G FAPI: PHY API Specification**
- **Issue date: 16 May 2022**
- **Version: 222.5.0**
:::

### 2.1 PARAM request, response, and TLV format
- MIMO is configured in PARAM TLV Configurations.
- TLV: Type-Length-Value
- The **`PARAM.request`** message is given in Table 3–6. From this table it can be seen that `PARAM.request` contains a list of optional TLVs with L2/L3 signalling.

| Field | Type | Description |
| -------- | -------- | -------- |
| TLVs     | Variable     | See Table 3-10     |

- The **`PARAM.response`** message is given in Table 3–7. From this table it can be seen that **`PARAM.response`** contains a list of TLVs providing information about the PHY. When the PHY is in the IDLE state, this information relates to the PHY’s overall capability. When the PHY is in the CONFIGURED state this information relates to the current configuration. 

| Field | Type | Description |
| -------- | -------- | -------- |
| Error Code     | uint8_t    | See Table 3-8     |
| Number of TLVs     | uint8_t     | Number of TLVs contained in the message 
body.     |
| TLVs     | Variable     | See Table 3-11     |

- `PARAM` and `CONFIG` TLVs are used in the PARAM and CONFIG message exchanges, respectively. For both the PARAM and CONFIG TLVs the TLV format is given in Table 3–9. Each TLV consists of:
    - Tag parameter of 2 bytes
    - Length parameter of 2 bytes
    - Value parameter
- The length of the Value parameter ensures the complete TLV is a multiple of 4-bytes (32-bits).


| Type     | Description | 
| -------- | -------- | 
| uint16_t     | Tag     | 
| uint16_t     | Length (in bytes)     | 
| variable    | Value     | 

### 2.2 MIMO-related TLVs

#### 2.2.1 PDCCH Parameters
PDDCH notes is **[here](https://hackmd.io/@ra-jordhie/Phylayer-Background#1-Physical-Downlink-Control-Channel-PDCCH-DL)**.

This table contains the configuration parameters of PDCCH (Table 3-15 PDCCH parameters).

| Tag    | Field                      | Type    | Description                                                  |
| ------ | -------------------------- | ------- | ------------------------------------------------------------ |
| 0x000B | cce Mapping Type           | uint8_t | Bitmap indicating support format. For each bit: 0 = not supported 1 = supported Bits: 0 = Interleaved 1 = Non-interleaved |
| 0x000C | coreset Outside First 3    | Ofdm    | Syms Of Slot |
|        |                            | uint8_t | 0 = not supported 1 = supported |
| 0x000D | precoder Granularity Coreset | uint8_t  | 0 = not supported 1 = supported|
| **0x000E** | **pdcch MuMimo**               | **uint8_t** | **0 = not supported 1 = supported**|
| 0x000F | pdcch Precoder Cycling     | uint8_t | 0 = not supported 1 = supported|
| 0x0010 | max Pdcchs Per Slot        | uint8_t | Value 1->255|

#### 2.2.2 PDSCH Parameters
PDSCH notes is **[here](https://hackmd.io/@ra-jordhie/Phylayer-Background#2-Physical-Data-Shared-Channel-PDSCH-DL)**.

This table contains the configuration parameters of PDSCH (Table 3-17 PDSCH parameters).

| Tag    | Field                           | Type    | Description                               |
| ------ | ------------------------------- | ------- | ----------------------------------------- |
| **0x001B** | **max Number Mimo Layers Pdsch**    | **uint8_t** | **Value: 1->8**                               |
| 0x001C | supported Max Modulation Order Dl | uint8_t | Value: 0 = QPSK 1 = 16 QAM 2 = 64 QAM 3 = 256 QAM                               |
| **0x001D** | **max MuMimo Users Dl**              | **uint8_t** | **Value: 1->255 Note: 1 indicates no DL MU-MIMO support. This TLV refers to multiple users with layers uniquely identified by DMRS and is fully independent of spatial multiplexing TLVs in Table 3-32.**   |

#### 2.2.3 PUSCH Parameters
PUSCH notes is **[here](https://hackmd.io/@ra-jordhie/Phylayer-Background#5-Physical-Uplink-Shared-Channel-PUSCH)**.

This table contains the configuration parameters of PUSCH (Table 3-19 PUSCH parameters).
Certainly! Here's the revised markdown table with the modifications:

| Tag    | Field                               | Type    | Description                                                                                                     |
| ------ | ----------------------------------- | ------- | --------------------------------------------------------------------------------------------------------------- |
| **0x002D** | **max Number Mimo Layers Non Cb Pusch** | **uint8_t** | **Value: 0 -> 4 L1 should ensure that maxNumberMimoLayers NonCbPusch + maxNumberMimoLayers CbPusch > 0**                                                                                              |
| **0x0049** | **max Number Mimo Layers Cb Pusch**    | **uint8_t** | **Value: 0 -> 4 L1 should ensure that maxNumberMimoLayers NonCbPusch + maxNumberMimoLayers CbPusch > 0**                                                                                              |
| 0x002E | supported Max Modulation Order Ul   | uint8_t | Value: 0 = QPSK 1 = 16 QAM 2 = 64 QAM 3 = 256 QAM|
| **0x002F** | **max MuMimo Users Ul**                  | uint8_t | **Value: 1 -> 255 Note: 1 indicates no UL MU-MIMO support.**|

#### 2.2.4 Spatial Multiplexing and MIMO Capabilities

The full TLV table are located in Table 3-32 Spatial Multiplexing and MIMO Capabilities.

## 3. DL_TTI.request
:::danger
This section is based on:
**5G FAPI: PHY API Specification**
- **Issue date: December 2022**
- **Version: 222.10.06**
:::
### 3.1 Message Body
:::success
FAPI Sec. 2.4.2 DL_TTI.request
:::
From Table 3-71 `DL_TTI.request` message body:

* **`DL_TTI.request`** **message body**

| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| sfn                   | uint16_t | SFN (Single Frequency Network). Value: 0->1023|
| slot                  | uint16_t | Slot. Value: 0->159|
| numPDUs               | uint16_t | Number of PDUs that are included in this message. All PDUs in the message are numbered in order. Value: 0->65535|
| numDL-Types-v3        | uint8_t  | Max number of DL PDU types or DCIs supported by `DL_TTI.request`|
| numPDUs-OfEachType-v3 | uint16_t | Number of PDUs of each typr that are indcluded in this message. The format is: [0]: number of PDDCH PDUs, [1]: number of PDSCH PDUs, [2]: number of CSI-RS PDUs, [3]: number of SSB PDUs, [4]: number of DIDCIs across all PDCCH PDUs in this message, [5]: number of PRS PDUs |
|topLevelRM-Patterns-v4|struct|See the next table|
| numGroups             | uint16_t | Number of PDU Groups included in this message|

* For number of PDUs

| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| pdu-Type| uint16_t | [0]: PDDCH PDU, [1]: PDSCH PDU, [2]: CSI-RS PDU, [3]: SSB PDU, [5]: PRS PDU |
| pdu-Size| uint16_t | Size of the PDU control information (in bytes). This length value includes the 4 bytes required for the PDU type and PDU size parameters. Value: 0->65535|
| dl-PDU-Configuration| struct | |

* In this message, if numGroup>0, then for each group:
:::warning
**For FAPIv4, groups represent MU MIMO groups, as described in section 2.2.6.2; muMimoGroups are listed in increasing order of their start symbol. muMimogroups starting at the same symbol, are listed channel by channel in increasing order of their first PRB index.**
:::

| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|**numPDUs-v4**| **uint8_t** | **Number of PDUs in this group. Value: 1->32 For SU-MIMO, one group includes one PDU only. For MU-MIMO, one group includes up to 32 PDUs.For each PDU type, each PRB in a symbol can be referenced by at most one MuMIMO group.**|
| prg-Size-v4| uint16_t | This is for reducing the layer mapping table|
| prb-BitmapStart-v6| uint16_u | Offset form reference CRB of prb-Bitmap. Value: 0->274|
| **prb-SCS-v6**| **uint8_t**  | **subcarrier spacing g [3GPP TS 38.211, sec 4.2] common to all PDUs in this muMimoGroup**|
| prb-Bitmap-v4 | uint8_t | bitmap of PRBs assumed common to all UEs in the group, for joint spatial precoding |
|**numPRGs-v5**|**uint16_t**|**Number of PRgs with at least one PRB allocated in prbBitmap. Shall be set to 0 if and only if the PDUs in the MU MIMO group are used with semi-static precoding.**|
|symbol-Bitmap-v4| uint16_t | Bitmap of symbols for the joint spatial precoding group.|

* For each of the numPDUs within the group

| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|pdu-Index-v4| uint16_t | This calue is an index for number of PDU identified by numPDU in this message|
|dci-Index-v6| uint16_t | This is the dciIndex prameter associated with a DCI in the PDCCH PDU pduIdx|

### 3.2 PDCCH
:::success
FAPI Sec. 3.4.2.1 PDCCH PDU
:::
- A PDCCH PDU includes 1 or more DCI PDUs 

#### 3.2.1 DL DCI PDU
- MIMO support is mentioned in DL DCI PDU for PHYs supporting FAPIv4 MU-MIMO groups (**Table 3-76 DL DCI PDU**):

![](https://hackmd.io/_uploads/ryiELHZF3.png)
![](https://hackmd.io/_uploads/SygdLrWYn.png)
Source: [fapi_interface.h](https://gerrit.o-ran-sc.org/r/admin/repos/o-du/l2,general)
Table 3-97 is [here](https://hackmd.io/@ra-jordhie/MIMO-FAPI-Param#471-Tx-Precoding-and-Beamforming-PDU).

#### 3.2.2 PDCCH PDU Parameters FAPIv4
- **Table 3-78 PDCCH PDU Parameters FAPIv4**

![](https://hackmd.io/_uploads/By_UwBWKh.png)

### 3.3 PDSCH
:::success
FAPI Sec. 3.3.2.2 PDSCH PDU
:::
#### 3.3.1 PDSCH PDU
- MIMO support is mentioned in PDSCH PDU for PHYs supporting FAPIv4 MU-MIMO groups (**Table 3-79 PDSCH PDU**):

![](https://hackmd.io/_uploads/HytnKSWK3.png)
![](https://hackmd.io/_uploads/HkUJqBbK2.png)
Source: [fapi_interface.h](https://gerrit.o-ran-sc.org/r/admin/repos/o-du/l2,general)
Table 3-97 is [here](https://hackmd.io/@ra-jordhie/MIMO-FAPI-Param#471-Tx-Precoding-and-Beamforming-PDU).

#### 3.3.2 PDSCH PDU Parameters FAPIv4
- **Table 3-84 PDCCH PDU Parameters FAPIv4**

![](https://hackmd.io/_uploads/BkxYcH-Kh.png)
![](https://hackmd.io/_uploads/HJShqrZF2.png)

### 3.4 CSI-RS
:::success
FAPI Sec. 3.4.2.3 CSI-RS PDU
:::
#### 3.4.1 CSI-RS PDU
- MIMO support is mentioned in CSI-RS PDU for PHYs supporting FAPIv4 MU-MIMO groups (**Table 3-86 CSI-RS PDU**):

![](https://hackmd.io/_uploads/SyKOoBWKh.png)
![](https://hackmd.io/_uploads/Byol2SWKn.png)
Source: [fapi_interface.h](https://gerrit.o-ran-sc.org/r/admin/repos/o-du/l2,general)
Table 3-97 is [here](https://hackmd.io/@ra-jordhie/MIMO-FAPI-Param#471-Tx-Precoding-and-Beamforming-PDU).

#### 3.4.2 CSI-RS PDU Parameters FAPIv4
- **Table 3-88 CSI-RS PDU Parameters FAPIv4**

![](https://hackmd.io/_uploads/S1pP2HbYh.png)
![](https://hackmd.io/_uploads/Hysi3S-tn.png)

### 3.5 SSB/PBCH
:::success
FAPI Sec. 3.4.2.4 SSB PDU
:::
#### 3.5.1 SSB/PBCH PDU
- MIMO support is mentioned in CSI-RS PDU for PHYs supporting FAPIv4 MU-MIMO groups (**Table 3-89 SSB/PBCH PDU**):

![](https://hackmd.io/_uploads/Bk44k8ZYn.png)


#### 3.5.2 SSB PDU Parameters FAPIv4
- **Table 3-93 SSB PDU Parameters FAPIv4**

![](https://hackmd.io/_uploads/r1xu6BWF3.png)

### 3.6 PRS
:::success
FAPI Sec. 3.4.2.4a PRS PDU
:::
#### 3.5.1 PRS PDU
- MIMO support is mentioned in PRS PDU (native, no need for FAPIv4) (**Table 3-94 PRS PDU**):

![](https://hackmd.io/_uploads/Sy9bgIZYh.png)

### 3.7 Tx Precoding and Beamforming
:::success
FAPI Sec. 3.4.2.5 Tx Precoding and Beamforming PDU
FAPI Sec. 3.4.2.6 Rel-16 mTRP Tx Precoding and Beamforming PDU
FAPI Sec. 3.4.2.7 Tx Precoding based on Channel Reciprocity PDU
FAPI Sec. 3.4.2.8 Tx Precoding Based on Dynamic Weight Signaling by L2/L3
:::
- The precoding and beamforming PDU is included in the PDCCH, PDSCH, CSI-RS and SSB PDUs as previously mentioned.

#### 3.7.1 Tx Precoding and Beamforming PDU
- Semistatic beamforming weights (i.e. signaled via beam ids) are applied to a group of consecutive PRB’s. These **beamforming groups (BFG)** are grouped with non overlapping consecutive BFGSize PRBs starting from CRB0. The BFGSize is a multiple of the PRGSize.

![](https://hackmd.io/_uploads/ryfWm8bK2.png)
![](https://hackmd.io/_uploads/BkNg7IZKh.png)


#### 3.7.2 Rel-16 mTRP Tx Precoding and Beamforming PDU
- The precoding and beamforming PDU is included in the PDCCH, PDSCH, CSI-RS and SSB PDUs, when PHY hosts both ports of a Rel-16 mTRP configuration, per 3GPP 38.300 sec. 6.12.
- L1 only expects this table if it supports Rel-16 mTRP.

![](https://hackmd.io/_uploads/Hk4of8WYh.png)
![](https://hackmd.io/_uploads/H1dKfI-Yh.png)

#### 3.7.3 Tx Precoding based on Channel Reciprocity PDU
- The Tx Precoding PDU based on Channel Reciprocity sampling; in this release, this PDU may be included, when precoding is performed as described in FAPI sec. 2.2.6.2 UL-SCH.

![](https://hackmd.io/_uploads/B1DuMI-Kh.png)

#### 3.7.4 Tx Precoding Based on Dynamic Weight Signaling by L2/L3
- The Tx Precoding PDU based on dynamic weight signaling by L2/L3; in this release, this PDU may be included, when precoding is performed as described in section FAPI sec. 2.2.6.2 UL-SCH.

![](https://hackmd.io/_uploads/BkeSnX8ZKn.png)



## 4. UL_TTI.request
:::danger
This section is based on:
**5G FAPI: PHY API Specification**
- **Issue date: December 2022**
- **Version: 222.10.06**
:::

### 4.1 Message Body

:::success
FAPI Sec. 3.4.3 UL_TTI.request
:::

### 4.2 PRACH
![](https://hackmd.io/_uploads/H1Y6DU-Y3.png)

![](https://hackmd.io/_uploads/rJqsv8Wth.png)
![](https://hackmd.io/_uploads/HyS-_IbF3.png)

![](https://hackmd.io/_uploads/HyPgu8-Fh.png)

### 4.3 PUSCH
![](https://hackmd.io/_uploads/r1Q4dLbKn.png)

### 4.4 PUCCH

### 4.5 SRS

### 4.6 RX Beamforming

## 5. SRS Indication
:::success
FAPI Sec. 3.4.10 SRS Indication
:::

# <center><i class="fa fa-edit"></i> HARQ FAPI Parameters</center>
###### tags: `Internship` `Daily Report` `PHY Interface` `FAPI`
:::success
**Learning Objective** :heavy_check_mark: 
* Learn and document HARQ parameters on FAPI
:::
:::info
**Resources** :package: 
1. [5G FAPI: PHY API Specification](https://www.smallcellforum.org/work-items/fapi/#:~:text=The%20latest%20FAPI%20specifications%20are,(P5)%20interface%20%5BSCF222%5D) 
2. [O-DU Gerrit Repo](https://gerrit.o-ran-sc.org/r/admin/repos/o-du/l2,general)
:::

## 1. Transport Channel
### 1.1 DL-SCH 
:::danger
This section is based on:
**5G FAPI: PHY API Specification**
- **Issue date: December 2022**
- **Version: 222.10.06**
:::
:::success
FAPI Sec. 2.2.5.3 DL-SCH
:::
The [DL-SCH transport channel](https://hackmd.io/@ra-jordhie/Phylayer-Background#52-Transport-Channel) is used to send data from the gNB to a single UE. HARQ is always applied on the DL-SCH transport channel at least for unicast PDSCH. Therefore, when scheduling a downlink PDSCH transmission which will require HARQ-ACK feedback from UE, the L2/L3 software has to schedule uplink transmission on PUCCH or PUSCH for the UE to feed back an ACK/NACK response. To transmit a DL-SCH PDU, the L2/L3 software must provide the following information:
- In `DL_TTI.request` a PDSCH PDU and PDCCH PDU are included. The PDCCH PDU contains control regarding the DL frame transmission
- In `TX_DATA.request` a MAC PDU containing the data is included
- At the expected slot UCI HARQ information is included in a later UL_TTI.request, where the timing of this message is variable. The HARQ can be sent on the PUSCH or PUCCH, therefore, the information of the HARQ response on the uplink is sent in either:
    - PUSCH PDU – is used if the UE is scheduled to transmit data and the ACK/NACK response
    - PUCCH PDU – is used if the UE is just scheduled to transmit the ACK/NACK response
- The PHY will return the ACK/NACK response information in the `UCI.indication` message

![](https://hackmd.io/_uploads/r1T3V9st2.png)

With DCI Format 1-1, the DL SCH channel can send two data transport blocks to a UE which requires a single PDCCH (DCI) PDU, a single PDSCH PDU and two MAC PDUs:
- In the first transmission slot in the DL_TTI.request a PDSCH and PDCCH 
(DCI) PDU is included. The PDCCH PDU contains control regarding the DL 
frame transmission. A PDSCH PDU includes two codewords, one for each 
transport block specified in the DCI PDU.
- In TX_DATA.request two MAC PDUs containing the data are included.

![](https://hackmd.io/_uploads/rkv_XiiFn.png)


Multi-slot transmission is also an option for the DL-SCH, where the same MAC PDU is transmitted for N slots. 
![](https://hackmd.io/_uploads/BJX97ssY3.png)

### 1.2 UL-SCH 
:::danger
This section is based on:
**5G FAPI: PHY API Specification**
- **Issue date: December 2022**
- **Version: 222.10.06**
:::
:::success
FAPI Sec. 2.2.6.2 UL-SCH
:::

The UL-SCH transport channel is used to send data from the UE to the gNB. HARQ is always applied on the UL-SCH transport channel and the ACK/NACK value is indicated when the next transmission for the HARQ process is scheduled via DCI. To transmit a UL-SCH PDU, the L2/L3 software must provide the following information:
- Within the UL_DCI.request for slot N a DCI PDU is included. The DCI PDU contains control information regarding the UL frame transmission being scheduled and indicates new data or retransmission for a specified HARQ process.
- In UL_TTI.request for slot N+K1 an ULSCH PDU is included, where the value of K1 is variable. 
- The PHY will return CRC information for the received data in the CRC.indication message
- The PHY will return the received uplink data in the RX_DATA.indication message.
- If UCI information was expected in the uplink slot, the PHY will return the UCI.indication message.

![](https://hackmd.io/_uploads/BkOuQ0jKh.png)

Mulyi-slot transmission is the same as DL-SCH which UL-SCH with configured grant, where the same MAC PDU is transmitted for N slots:
- In UL_TTI.request for slot N+M a PUSCH PDU is included, the UE will have previously been configured to be aware that multi-slot transmission is used.
- The PHY will return CRC information for the received data in a the CRC.indication message
- The PHY will return the received uplink data in the RX_DATA.indication message.

![](https://hackmd.io/_uploads/H1XqgyhY2.png)

## 2. Transport Channel