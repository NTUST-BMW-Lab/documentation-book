# <center><i class="fa fa-edit"></i> PHY STUB SOURCE CODE TRACING </center>
###### tags: `Internship` `Daily Report` `PHY STUB`
:::success
**Learning Objective** :heavy_check_mark: 
* Briefly Describe Involved function on FAPI interface --> for each message
* Show the function execution flow on handling each message --> both for MAC (sender) and PHY (receiver).

Message that I trace are:
1. RX_DAT.indication
2. UL_TTI.request
3. SLOT.indication
4. TX_DATA.request
5. DL_TTI.request
6. UCI.indication

:::
:::info
**Resources** :package: 
1. [5G FAPI: PHY API Specification](https://www.smallcellforum.org/work-items/fapi/#:~:text=The%20latest%20FAPI%20specifications%20are,(P5)%20interface%20%5BSCF222%5D)
2. [O-DU l2 source code](https://gerrit.o-ran-sc.org/r/admin/repos/o-du/l2,general)
:::
----
Outline:
* [PHY STUB SOURCE CODE TRACING](/oKjTAGKiQCGu6wa4zqbV-w)
* [Source Code Mapping](#Source-Code-Mapping)
    * [RX_DATA.indicaton](#RX_DATAindication)
    * [UL_TTI.request](#UL_TTIrequest)
    * [SLOT.indication](#SLOTindication)
    * [TX_DATA.request](#TX_DATArequest)
    * [DL_TTI.request](#DL_TTIrequest)
    * [UCI.indication](#UCIindication)

----
The PHY stub is a crucial component within the Open Distributed Unit (O-DU) system, responsible for simulating or emulating the behavior of the physical layer (PHY) in a wireless communication system. Understanding the implementation details of the PHY stub is essential for comprehending the inner workings of the O-DU system. By tracing the source code associated with the PHY stub, developers can gain insights into its initialization, event handling, and overall functionality.

This documentation aims to provide an overview of how to effectively trace the source code for the PHY stub in O-DU. It serves as a guide for navigating the relevant source code files, understanding the initialization process, examining event handling mechanisms, and exploring any optional event logging or call flow tracing features.

![](https://hackmd.io/_uploads/SJ7gKDGYn.png)


:::info
**Note**: It is assumed that the reader has a basic understanding of the O-DU system architecture and is familiar with the C programming language.

You also have to check the data type for this code because we will use this struct (data type) on many occasion. I already traced this data types on ==[this note](https://hackmd.io/@fauzanmuhammad/phy_stub_datatypes)==.
:::

:::success
Click [here](https://hackmd.io/@fauzanmuhammad/phy_stub_code_tracing_2) for the next part.
:::

# Source Code Mapping
:::warning
O-DU l2 git repository:
> https://gerrit.o-ran-sc.org/r/admin/repos/o-du/l2,general
:::

## RX_DATA.indication
### 1. `l1BuildAndSendRxDataInd` (Main/Starting Function)
> File Directory : .\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**General Information**
This function send [RX_DATA.indication](https://hackmd.io/9tBOS229SCyOm_7Js9DXWQ?view#RX_DATAindication) message from l1 (PHY layer) to l2 (MAC layer). Because of this, this function is the ==main/starting function==. 
:::

:::info
**Declaration**
```c
uint16_t l1BuildAndSendRxDataInd(uint16_t slot, 
                                 uint16_t sfn,
                                 fapi_ul_pusch_pdu_t puschPdu)
```
**Functionality**
Build and send RX_DATA.indication

**Parameters**
| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| sfn                   | uint16_t | SFN (Single Frequency Network). Value: 0->1023|
| slot                  | uint16_t | Slot. Value: 0->159|
|puschPdu|fapi_ul_pusch_pdu_t|puschPdu is a `fapi_ul_pusch_pdu_t` type data. `fapi_ul_pusch_pdu_t` is a struct. You can find the details of this struct on ==[5G FAPI: PHY API Spec Ver. 222.10.06](https://www.smallcellforum.org/work-items/fapi/#:~:text=The%20latest%20FAPI%20specifications%20are,(P5)%20interface%20%5BSCF222%5D)== Table 3-105 on PUSCH PDU section.|

**Return Value**
* ROK    :arrow_right: success
* RFAILED    :arrow_right: failure

**Details**
:::spoiler
:::warning
**The Code starts with this initialization**:
```c=410
   uint8_t idx = 0, ueId = 0;
   fapi_rx_data_indication_t *rxDataInd =NULLP;
   fapi_pdu_ind_info_t       *pduInfo =NULLP;
   uint8_t  *pdu = NULLP;
   uint16_t byteIdx = 0;
   uint32_t msgLen = 0;
   MsgType type = 0;
```
410. Initializing the `idx` and User Equipment Id `ueID` to 0.
411. Making a pointer `*rxDataInd` of `fapi_rx_data_indication_t` struct. This pointer points to NULLP (points to nothing) [5G FAPI: PHY API 222.10.06 ver. Table 3-133]
412. Making a pointer `*pduInfo` of `fapi_pdu_ind_info_t` struct. This pointer points to NULLP (points to nothing) [5G FAPI: PHY API 222.10.06 ver. Table 3-133].
413. Initializing the `*pdu` pointer for pointing to NULLP.
414. Initializing the `byteIdx` to 0.
415. Initializing the length of the message to 0.
416. Changing the message type to 0, meaning this message type is the `ALARM` type.
***
**Getting the UE Id**
```c=418
   GET_UE_ID(puschPdu.rnti, ueId);
```
***
**Checking if the Current UE Uses Contention-Free Random Access (CF-RA)**
```c=419
   if(phyDb.ueDb.ueCb[ueId-1].isCFRA)
   {
      if(!phyDb.ueDb.ueCb[ueId-1].msgRrcReconfigComp)
      {
         /* In CF-RA in case of handover, RRC Reconfiguration Complete is sent
          * by UE once RAR is received from DU */
         phyDb.ueDb.ueCb[ueId-1].ueId = ueId;
         phyDb.ueDb.ueCb[ueId-1].crnti = puschPdu.rnti;
         phyDb.ueDb.ueCb[ueId-1].msgRrcReconfigComp = true;
         type = MSG_TYPE_RRC_RECONFIG_COMPLETE;
      }
      else
         return ROK; 
   }
```
Notes: In a contention-based random access procedure, multiple devices contend for the same resources, which can lead to collisions and decreased efficiency. On the other hand, **CF-RA** is a method that assigns dedicated resources to individual devices in advance, eliminating the contention.

421. **If the current UE uses CF-RA**, the code further checks whether the RRC (Radio Resource Control) Reconfiguration Complete message has been sent. **If not**, it sets the necessary flags and assigns the message type as MSG_TYPE_RRC_RECONFIG_COMPLETE. 
422. Otherwise, it returns **ROK**.

If we trace the code again, we can find the element of the `ueCb`struct that we will oftenly used in advance: 
```c=1
typedef struct ueCb
{
   uint8_t  ueId;
   uint16_t crnti;
   bool     rachIndSent;
   bool     isCFRA;
   bool     msg3Sent;
   bool     msg5ShortBsrSent;
   bool     msg5Sent;
   bool     dlDedMsg;
   bool     msgNasAuthenticationComp;
   bool     msgNasSecurityModeComp;
   bool     msgRrcSecurityModeComp;
   bool     msgRrcReconfigComp;
   bool     msgRegistrationComp;
   uint8_t  rlcSnForSrb1;           /* Sequence number of PDU at RLC for AM mode */
   uint8_t  pdcpSn;                 /* Sequence number of PDU at PDCP */
}UeCb;
```

***
**If the Current UE does not use CF-RA**
```c=419
   else
   {
      if(!phyDb.ueDb.ueCb[ueId-1].msg3Sent)
      {
         phyDb.ueDb.ueCb[ueId-1].ueId = ueId;
         phyDb.ueDb.ueCb[ueId-1].crnti = puschPdu.rnti;
         phyDb.ueDb.ueCb[ueId-1].msg3Sent = true;
         type = MSG_TYPE_MSG3;
         sleep(2);
      }
      else if(!phyDb.ueDb.ueCb[ueId-1].msg5ShortBsrSent)
      {
         phyDb.ueDb.ueCb[ueId-1].msg5ShortBsrSent = true;
         type = MSG_TYPE_SHORT_BSR;
      }
      else if(!phyDb.ueDb.ueCb[ueId-1].msg5Sent)
      {
         phyDb.ueDb.ueCb[ueId-1].msg5Sent = true;
         type = MSG_TYPE_MSG5;
      }
      else if(!phyDb.ueDb.ueCb[ueId-1].msgNasAuthenticationComp)
      {
        phyDb.ueDb.ueCb[ueId-1].msgNasAuthenticationComp = true;
        type = MSG_TYPE_NAS_AUTHENTICATION_COMPLETE;
      }
      else if(!phyDb.ueDb.ueCb[ueId-1].msgNasSecurityModeComp)
      {
         phyDb.ueDb.ueCb[ueId-1].msgNasSecurityModeComp = true;
         type = MSG_TYPE_NAS_SECURITY_MODE_COMPLETE;
      }
      else if(!phyDb.ueDb.ueCb[ueId-1].msgRrcSecurityModeComp)
      {
         phyDb.ueDb.ueCb[ueId-1].msgRrcSecurityModeComp = true;
         type = MSG_TYPE_RRC_SECURITY_MODE_COMPLETE;
      }
      else if(!phyDb.ueDb.ueCb[ueId-1].msgRegistrationComp)
      {
         phyDb.ueDb.ueCb[ueId-1].msgRegistrationComp = true;
         type = MSG_TYPE_REGISTRATION_COMPLETE; 
      }
      else if(!phyDb.ueDb.ueCb[ueId-1].msgRrcReconfigComp)
      {
         phyDb.ueDb.ueCb[ueId-1].msgRrcReconfigComp = true;
         type = MSG_TYPE_RRC_RECONFIG_COMPLETE;
      }
      else
         return ROK;
   }
```
* The code checks different conditions to determine the appropriate message type to be sent. These conditions check flags related to various stages of the communication protocol, such as MSG3, short BSR, MSG5, NAS authentication complete, NAS security mode complete, RRC security mode complete, and registration complete.
* If a flag is not set, it sets the flag and assigns the corresponding message type. If all flags are set, it returns ROK.
****

**Allocating Memory for `rxDataInd`**
```c=482
   MAC_ALLOC(rxDataInd, sizeof(fapi_rx_data_indication_t));
```
The following function `MAC_ALLOC` is explained below:
```c=1
#define MAC_ALLOC(_datPtr, _size)                            \
{                                                            \
   uint8_t _ret;                                             \
   _ret = SGetSBuf(MAC_MEM_REGION, MAC_POOL,                 \
	 (Data **)&_datPtr, _size);                               \
   if(_ret == ROK)                                           \
   {                                                         \
      memset(_datPtr, 0, _size);                             \
      MAC_MEM_LOG("MAC,ALLOC", __FILE__, __LINE__, __FUNCTION__, _size, _datPtr);\
   }                                                         \
   else                                                      \
   {                                                         \
      _datPtr = NULLP;                                       \
   }                                                         \
}
```
The `MAC_ALLOC` function will allocate the memory for `rxDataInd` message based on the size of `fapi_rx_data_indication_t`. And then, the code checks if the `rxDataInd` is still NULL or not.

```c=483
   if(!rxDataInd)
   {
      DU_LOG("\nERROR  -->  PHY_STUB: Memory allocation failed for Rx Data Indication");
      return RFAILED;
   }
   memset(rxDataInd, 0, sizeof(fapi_rx_data_indication_t));
```

If the `rxDataInd` still NULL, it will return `RFAILED`.

488. Then, we set all the value of `rxDataInd` to 0. The purpose is to effectively initializing it to make clean state before populating it with actual data.
***

**Filling the `rxDataInd`'s elements and `pduInfo`**
```c=490
   msgLen = sizeof(fapi_rx_data_indication_t) - sizeof(fapi_msg_t);
   rxDataInd->sfn = sfn;
   rxDataInd->slot = slot;
   rxDataInd->numPdus = 1;

   pduInfo = &rxDataInd->pdus[idx];
   pduInfo->handle = puschPdu.handle;
   pduInfo->rnti = puschPdu.rnti;
   pduInfo->harqId = puschPdu.puschData.harqProcessId;
   pduInfo->pdu_length = puschPdu.puschData.tbSize ;
   pduInfo->ul_cqi = 0;
   pduInfo->timingAdvance = 0;
   pduInfo->rssi = 0;
```
491. Filling the `rx_DataInd`'s sfn element with `sfn` that we input on `l1BuildAndSendRxDataInd` function
492. Filling the `rx_DataInd`'s slot element with `slot` that we input on `l1BuildAndSendRxDataInd` function
493. Setting the `rxDataInd`'s numPdus to 1 

***
**Forming different type of PDU based on the Message Type**

```c=516
   switch(type)
   {
      case MSG_TYPE_MSG3: 
         {
            DU_LOG("\nDEBUG  -->  PHY_STUB: Forming MSG3 PDU ");
            /* For Initial RRC setup Request,
               MAC subheader format is R/R/LCId (1byte)
               LCId is CCCH(0)
               From 38.321 section 6.1.1
               */
            pdu[byteIdx++] = 0;
            /* Hardcoding MAC PDU */
            pdu[byteIdx++] = 16;
            pdu[byteIdx++] = 00;
            pdu[byteIdx++] = 00;
            pdu[byteIdx++] = 00;
            pdu[byteIdx++] = 00;
            pdu[byteIdx++] = 103;

            break;
         }
      case MSG_TYPE_SHORT_BSR:
         {
            DU_LOG("\nDEBUG  -->  PHY_STUB: Forming SHORT BSR PDU ");
            uint8_t lcgId = 0;
            uint8_t bufferSizeIdx = 6;

            /* For Short BSR
               MAC subheader format is R/R/LcId (1Byte)
               LCId is 61
               From 38.321 section 6.1.1
               */
            pdu[byteIdx++] = 61;    // LCID
            pdu[byteIdx++] = (lcgId << 5) | bufferSizeIdx;

            break;
         }

      case MSG_TYPE_MSG5:
      {
         /* For RRC setup complete
          *
          * MAC subheader format is R/F/LCId/L (2/3 bytes)
          * LCId is 1 for SRB1
          * L is length of PDU i.e 6bytes here 
          * From 38.321 section 6.1.1
          *
          * RLC subheader for AM PDU is D/C/P/SI/SN (2 bytes for 12-bit SN)
          * From 38.322, section 6.2.2.4
          */
         DU_LOG("\nDEBUG  -->  PHY_STUB: Forming MSG5 PDU");
         uint8_t  msg5PduLen = 33; /* Length of MSG5 */
         msg5PduLen += 2; /* RLC subheader */
         uint8_t msg5[] = {1, msg5PduLen, 128, phyDb.ueDb.ueCb[ueId-1].rlcSnForSrb1++, 0, phyDb.ueDb.ueCb[ueId-1].pdcpSn++, 16, 0, \
            5, 223, 128, 16, 94, 64, 3, 64, 68, 252, 97, 0, 0, 0, 0, 4, 0, 0, 4, 68, 11, 128, 184, 56, 0, 0, 0, 0, 0};

         msg5PduLen += 2;  /* 2 bytes of MAC header */
         memcpy(pdu, &msg5, msg5PduLen);
         byteIdx += msg5PduLen; /* 4 bytes of header : MAC+RLC */
         break;
      }

      case MSG_TYPE_NAS_AUTHENTICATION_COMPLETE:
      {
        /* For Authentication response where RRC Container is dummy
          *
          * MAC subheader format is R/F/LCId/L (2/3 bytes)
          * LCId is 1 for SRB1
          * L is length of PDU i.e 6bytes here 
          * From 38.321 section 6.1.1
          *
          * RLC subheader for AM PDU is D/C/P/SI/SN (2 bytes for 12-bit SN)
          * From 38.322, section 6.2.2.4
          */
         DU_LOG("\nDEBUG  -->  PHY_STUB: Forming AUTHENTICATION RESPONSE PDU");
         uint8_t  pduLen = 37; /* Length of PDU */
         pduLen += 2; /* RLC subheader */
         uint8_t msg[] = {1, pduLen, 128, phyDb.ueDb.ueCb[ueId-1].rlcSnForSrb1++, 0, phyDb.ueDb.ueCb[ueId-1].pdcpSn++, 0x3a, \
                          0x0e, 0x3f, 0x00, 0xca, 0x95, 0xe9, 0x19, 0x41, 0x3f, 0x00, 0x2b, 0x96, 0x88, 0x06, 0xd7, 0x16, 0xc6, \
                          0x8b, 0xea, 0xae, 0x45, 0xd1, 0x01, 0xfd, 0x34, 0xd4, 0xfd, 0xd5, 0x71, 0x00, 0x00, 0x00, 0x00, 0x00};

         pduLen += 2;  /* 2 bytes of MAC header */
         memcpy(pdu, &msg, pduLen);
         byteIdx += pduLen; /* 4 bytes of header : MAC+RLC */
         break;
      }
      
      case MSG_TYPE_NAS_SECURITY_MODE_COMPLETE:
      {
        /* For NAS security mode complete where RRC Container is dummy
          *
          * MAC subheader format is R/F/LCId/L (2/3 bytes)
          * LCId is 1 for SRB1
          * L is length of PDU i.e 6bytes here 
          * From 38.321 section 6.1.1
          *
          * RLC subheader for AM PDU is D/C/P/SI/SN (2 bytes for 12-bit SN)
          * From 38.322, section 6.2.2.4
          */
         DU_LOG("\nDEBUG  -->  PHY_STUB: Forming NAS SECURITY MODE COMPLETE PDU");
         uint8_t  pduLen = 93; /* Length of PDU */
         pduLen += 2; /* RLC subheader */
         uint8_t msg[] = {1, pduLen, 128, phyDb.ueDb.ueCb[ueId-1].rlcSnForSrb1++, 0, phyDb.ueDb.ueCb[ueId-1].pdcpSn++, 0x3a, 0x2a, 0x3f, 
                          0x02, 0x75, 0xa0, 0xa0, 0xc0, 0x80, 0x3f, 0x00, 0x2f, 0x3b, 0x80, 0x04, 0x9a, 0xa2, 0x81, 0x09, 0x80, 0xc0, 
                          0x28, 0x04, 0xf8, 0xb8, 0x80, 0x1d, 0xbf, 0x00, 0x20, 0x8c, 0x80, 0x05, 0xf9, 0x00, 0x78, 0x88, 0x7a, 0x88, 
                          0xd9, 0x00, 0x00, 0x00, 0x03, 0x08, 0x00, 0x81, 0x97, 0x02, 0x78, 0x38, 0x78, 0x38, 0x17, 0x82, 0x82, 0x00, 
                          0x80, 0x00, 0x00, 0xa9, 0x00, 0x78, 0x88, 0x00, 0x00, 0x00, 0x8b, 0x83, 0xf8, 0x38, 0x60, 0x20, 0x0c, 0xc0, 
                          0x50, 0x0c, 0x00, 0x80, 0x3a, 0x00, 0x00, 0x48, 0x29, 0x80, 0x80, 0x80, 0x00, 0x00, 0x00, 0x00};

         pduLen += 2;  /* 2 bytes of MAC header */
         memcpy(pdu, &msg, pduLen);
         byteIdx += pduLen; /* 4 bytes of header : MAC+RLC */
         break;
      }

      case MSG_TYPE_RRC_SECURITY_MODE_COMPLETE:
      {
         /* For security mode complete where RRC Container is dummy
          *
          * MAC subheader format is R/F/LCId/L (2/3 bytes)
          * LCId is 1 for SRB1
          * L is length of PDU i.e 6bytes here 
          * From 38.321 section 6.1.1
          *
          * RLC subheader for AM PDU is D/C/P/SI/SN (2 bytes for 12-bit SN)
          * From 38.322, section 6.2.2.4
          */
         DU_LOG("\nDEBUG  -->  PHY_STUB: Forming RRC SECURITY MODE COMPLETE PDU");
         uint8_t  pduLen = 12; /* Length of PDU */
         pduLen += 2; /* RLC subheader */
         uint8_t msg[] = {1, pduLen, 128, phyDb.ueDb.ueCb[ueId-1].rlcSnForSrb1++, 0, phyDb.ueDb.ueCb[ueId-1].pdcpSn++, 0x2a, 0x40, \
            0, 0, 0, 0, 0, 0, 0, 0};

         pduLen += 2;  /* 2 bytes of MAC header */
         memcpy(pdu, &msg, pduLen);
         byteIdx += pduLen; /* 4 bytes of header : MAC+RLC */
         break;
      }

      case MSG_TYPE_REGISTRATION_COMPLETE:
      {
         /* For rrc reconfig complete where RRC Container is dummy
          *
          * MAC subheader format is R/F/LCId/L (2/3 bytes)
          * LCId is 1 for SRB1
          * L is length of PDU i.e 6bytes here
          * From 38.321 section 6.1.1
          * 
          * RLC subheader for AM PDU is D/C/P/SI/SN (2 bytes for 12-bit SN)
          * From 38.322, section 6.2.2.4
          */
         DU_LOG("\nDEBUG  -->  PHY_STUB: Forming RRC REGISTRATION COMPLETE PDU");
         uint8_t  pduLen = 12; /* Length of PDU */
         pduLen += 2; /* RLC subheader */
         uint8_t msg[] = {1, pduLen, 128, phyDb.ueDb.ueCb[ueId-1].rlcSnForSrb1++, 0, phyDb.ueDb.ueCb[ueId-1].pdcpSn++, 0x3a, 0x81, \
            0xbf, 0, 0x21, 0x80, 0, 0, 0, 0};

         pduLen += 2;  /* 2 bytes of MAC header */
         memcpy(pdu, &msg, pduLen);
         byteIdx += pduLen; /* 4 bytes of header : MAC+RLC */
         break;
      }

      case MSG_TYPE_RRC_RECONFIG_COMPLETE:
      {
         /* For rrc reconfig complete where RRC Container is dummy
          *
          * MAC subheader format is R/F/LCId/L (2/3 bytes)
          * LCId is 1 for SRB1
          * L is length of PDU i.e 6bytes here
          * From 38.321 section 6.1.1
          *
          * RLC subheader for AM PDU is D/C/P/SI/SN (2 bytes for 12-bit SN)
          * From 38.322, section 6.2.2.4
          */
         DU_LOG("\nDEBUG  -->  PHY_STUB: Forming RRC RECONFIGURATION COMPLETE PDU");
         uint8_t  pduLen = 13; /* PDU length */
         pduLen += 2; /* RLC sub header */
         uint8_t msg[] = {1, pduLen, 128, phyDb.ueDb.ueCb[ueId-1].rlcSnForSrb1++, 0, phyDb.ueDb.ueCb[ueId-1].pdcpSn++, 8, 64, 0, 0,\
            0, 0, 0, 0, 0, 0, 0};

         pduLen += 2;  /* 2bytes of MAC header */
         memcpy(pdu, &msg, pduLen);
         byteIdx += pduLen; /* 4 bytes of header : MAC+RLC*/
         break;

      }

      default:
      break;
   } /* End of switch(type) */
```
The following case is explained below:
1. **`MSG_TYPE_MSG3`**: This case is for forming an MSG3 PDU, which is used for Initial RRC (Radio Resource Control) setup request.
2. **`MSG_TYPE_SHORT_BSR`**: This case is for forming a Short BSR (Buffer Status Report) PDU.
3. **`MSG_TYPE_MSG5`**: This case is for forming an MSG5 PDU, which is used for RRC setup complete.
4. **`MSG_TYPE_NAS_AUTHENTICATION_COMPLETE`**: This case is for forming an Authentication Response PDU.
5. **`MSG_TYPE_NAS_SECURITY_MODE_COMPLETE`**: This case is for forming a NAS Security Mode Complete PDU.
6. **`MSG_TYPE_RRC_SECURITY_MODE_COMPLETE`**: This case is for forming an RRC Security Mode Complete PDU.
7. **`MSG_TYPE_REGISTRATION_COMPLETE`**: This case is for forming an RRC Registration Complete PDU.
8. **`MSG_TYPE_RRC_RECONFIG_COMPLETE`**: This case is for forming an RRC Reconfiguration Complete PDU.

***
**Filling MAC SDU for Padding Bytes**
```c=708
   if(byteIdx < pduInfo->pdu_length)
   {
      /* For Padding
         MAC subheader format is R/R/LCId (1byte)
         LCId is 63 for padding
         From 38.321 section 6.1.1
         */
      pdu[byteIdx++] = 63;

      for(; byteIdx < pduInfo->pdu_length; byteIdx++)
         pdu[byteIdx] = 0;
   }
```
***
**Sending RX Data Indication to MAC**
```c=726
   DU_LOG("\nINFO   -->  PHY STUB: Sending Rx data Indication to MAC");
   procPhyMessages(rxDataInd->header.msg_id, sizeof(fapi_rx_data_indication_t), (void *)rxDataInd);

   if(pduInfo->pdu_length)
      MAC_FREE(pduInfo->pduData, pduInfo->pdu_length);
   MAC_FREE(rxDataInd, sizeof(fapi_rx_data_indication_t));

#ifdef START_DL_UL_DATA 
      if(phyDb.ueDb.ueCb[ueId-1].msgRrcReconfigComp == true)
      {
         startUlData();
      }
#endif
   return ROK;
}
```
There are several procedure and function that will be used on sending the RX_DATA.indication message. There are [`procyPhyMessages`](#procPhyMessages-function) and [`startULData`](). I will cover this on the next section.

:::

### 2. `procPhyMessages` (function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_handle_phy.c

:::success
**Specific Information**
This function will process the message from PHY. This function is called when we already fill the parameters for `rxDataInd` struct on [`l1BuildAndSendRxDataInd`](#l1BuildAndSendRxDataInd-Main-Function) function. 
:::

:::info

**Declaration**
```c
void procPhyMessages(uint16_t msgType, 
                     uint32_t msgSize, 
                     void *msg)
```

**Functionality**
Processes message from PHY

**Parameters**

| Parameter | Type | Description |
| --------- | ---- | ----------- |
|msgType|uint16_t|This parameter defines the message type based on the header that we already filled on [`l1BuildAndSendRxDataInd`](#l1BuildAndSendRxDataInd-Main-Function)|
|msgSize|uint32_t|The size of the message (`fapi_rx_data_indication_t` struct) |
|*msg |  | Pointer to the received message from the PHY layer|

**Return Value**
* ROK    :arrow_right: success
* RFAILED    :arrow_right: failure

**Details**
:::spoiler
:::warning
**If INTEL_FAPI available, Extract the Header**
```c=556
#ifdef INTEL_FAPI
   /* extract the header */
   fapi_msg_t *header = NULLP;
   header = (fapi_msg_t *)msg;
``` 
The `fapi_msg_t` struct looks like this:
```c=1
    typedef struct {
        uint16_t msg_id;
        uint16_t pad;
        uint32_t length;        // Length of the message body in bytes  5G FAPI Table 3-3
    } fapi_msg_t;
```
***
**Calling a `callFlowFromPhytoLwrMac` function**
```c=1
#ifdef CALL_FLOW_DEBUG_LOG 
   callFlowFromPhyToLwrMac(header->msg_id);
```
This function prints `src`, `dest`, `msg infor` about all the `msgs` that received.
***

**Handling Different Message Types Based on `header->msg_id`**
```c=567
#ifdef INTEL_TIMER_MODE
      case FAPI_VENDOR_EXT_UL_IQ_SAMPLES:
         {
            DU_LOG("\nDEBUG  -->  LWR_MAC: Received FAPI_VENDOR_EXT_UL_IQ_SAMPLES");
            //send config req
            uint16_t cellId = 1;
            sendToLowerMac(CONFIG_REQUEST, 0, (void *)&cellId);
            break;
         } 
#endif
      case FAPI_PARAM_RESPONSE:
	 {
            sendToLowerMac(PARAM_RESPONSE, msgSize, msg);
	    break;
	 }
      case FAPI_CONFIG_RESPONSE:
	 {
	    sendToLowerMac(CONFIG_RESPONSE, msgSize, msg);
	    break;
	 }
      case FAPI_SLOT_INDICATION:
	 {
	    if(lwrMacCb.phyState == PHY_STATE_CONFIGURED)
	    {
	       DU_LOG("\nINFO  -->  LWR_MAC: PHY has moved to running state");
	       lwrMacCb.phyState = PHY_STATE_RUNNING;
	       lwrMacCb.cellCb[0].state = PHY_STATE_RUNNING;
	    }

	    fapi_slot_ind_t *slotInd = NULLP;
	    slotInd  = (fapi_slot_ind_t *)msg;
	    procSlotInd(slotInd);
	    break;
	 }
      case FAPI_ERROR_INDICATION:
	 {
	    break;
	 }
      case FAPI_RX_DATA_INDICATION:
	 {
	    fapi_rx_data_indication_t *rxDataInd;
	    rxDataInd = (fapi_rx_data_indication_t *)msg;
	    procRxDataInd(rxDataInd);
	    break;
	 }  
      case FAPI_CRC_INDICATION:
	 {
	    fapi_crc_ind_t  *crcInd;
	    crcInd = (fapi_crc_ind_t *)msg;
	    procCrcInd(crcInd);
	    break;
	 }  
      case FAPI_UCI_INDICATION:
	 {
	    fapi_uci_indication_t *phyUciInd = NULLP;
	    phyUciInd = (fapi_uci_indication_t*)msg;
	    procUciInd(phyUciInd);
	    break;
	 }
      case FAPI_SRS_INDICATION:
	 {
	    break;
	 }  
      case FAPI_RACH_INDICATION:
	 {
	    fapi_rach_indication_t  *rachInd;
	    rachInd = (fapi_rach_indication_t *)msg;
	    procRachInd(rachInd);
	    break;
	 }
      case FAPI_STOP_INDICATION:
	 {
	    DU_LOG("\nINFO  -->  LWR_MAC: Handling Stop Indication");
	    procStopInd();
	    break;
	 }  
   }
```
The explanation for the case is:
1. **`FAPI_VENDOR_EXT_UL_IQ_SAMPLES`**: Logs a debug message and sends a configuration request to the lower MAC layer.
2. **`FAPI_PARAM_RESPONSE`** and **`FAPI_CONFIG_RESPONSE`**: Forwards the message to the lower MAC layer using the `sendToLowerMac` function, passing the corresponding message type and size.
3. **`FAPI_SLOT_INDICATION`**: Checks if the physical layer state is configured, updates the state to running, and processes the slot indication by calling `procSlotInd` with the slotInd parameter.
4. **`FAPI_ERROR_INDICATION`** and **`FAPI_SRS_INDICATION`**: Empty cases
5. **`FAPI_RX_DATA_INDICATION`**: Typecasts the msg to fapi_rx_data_indication_t* and processes the received data indication by calling `procRxDataInd` with the rxDataInd parameter.
6. **`FAPI_CRC_INDICATION`**: Typecasts the msg to fapi_crc_ind_t* and calls `procCrcInd` to process the CRC indication. 
7. **`FAPI_UCI_INDICATION`**: Typecasts the msg to fapi_uci_indication_t* and processes the UCI indication by calling `procUciInd` with the phyUciInd parameter.
8. **`FAPI_RACH_INDICATION`**: Typecasts the msg to fapi_rach_indication_t* and processes the RACH indication by calling `procRachInd` with the rachInd parameter.
9. **`FAPI_STOP_INDICATION`**: Logs an informational message and calls `procStopInd` to handle the stop indication.

Because we want to handle the **`RX_DATA.indication`** message, the next procedure will be handled by [`procRxDataInd`](#procRxDataInd-function) procedure.
:::

### 3. `startULData` (procedure)
> File Directory : .\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**Specific Information**
This function will pump the data from PHY stub to DU after [`procPhyMessages`](#procPhyMessages-function) have done.
:::

:::info

**Declaration**
```c
void startUlData()
```

**Functionality**
Start the uplink data

**Parameters**
None

**Return Value**
None

**Details**
:::spoiler
:::warning

```c=56
{
   uint8_t ueIdx=0, drbIdx=0;

   /* Start Pumping data from PHY stub to DU */
   for(ueIdx=0; ueIdx < phyDb.ueDb.numActvUe; ueIdx++)
   {
      for(drbIdx = 0; drbIdx < NUM_DRB_TO_PUMP_DATA; drbIdx++) //Number of DRB times the loop will run
      {
         DU_LOG("\nDEBUG  --> PHY STUB: Sending UL User Data[DrbId:%d] for UEIdx %d\n",drbIdx,ueIdx);
         l1SendUlUserData(drbIdx,ueIdx);
      }
   } 
}
```
1. Two variables, ueIdx and drbIdx, are declared as uint8_t (unsigned 8-bit integers). These variables will be used as loop counters.
1. The code enters a nested loop structure. The outer loop iterates over active UEs (user equipment) in the phyDb.ueDb database. The phyDb.ueDb.numActvUe variable represents the number of active UEs.
1. The inner loop iterates over DRBs (data radio bearers) and runs NUM_DRB_TO_PUMP_DATA times. NUM_DRB_TO_PUMP_DATA is a constant that determines the number of times the loop will run.
1. Within the inner loop, a debug message is printed using the DU_LOG macro, indicating the DRB ID and UE index for which UL user data is being sent.
1. The function l1SendUlUserData() is called with the drbIdx and ueIdx as arguments. This function is responsible for sending UL user data for the specified DRB and UE.
1. The loops continue to iterate until all active UEs and DRBs have been processed.
:::

### 4. `procRxDataInd` (function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_handle_phy.c

:::success
**Specific Information**
This function will handle Rx Data indication from PHYC. This function is called by [`procPhyMessage`](#procPhyMessages-function) function.
:::

:::info

**Declaration**
```c
uint8_t procRxDataInd(fapi_rx_data_indication_t  *fapiRxDataInd)
```

**Functionality**
Handles Rx Data indication from PHY and sends to MAC

**Parameters**

| Field | Type | Description |
| -------- | -------- | -------- |
| *fapiRxDataInd     | Pointer     | This is a pointer that points to `fapi_rx_data_indication_t` struct     |


**Return Value**
* ROK:arrow_right: Success
* RFAILUE:arrow_right: Failure

**Details**
:::spoiler
:::warning
**Initialization**
```c=312
   Pst           pst;
   uint8_t       pduIdx =0;
   RxDataInd     *rxDataInd = NULLP;
   RxDataIndPdu  *pdu = NULLP;   
```
Several variables are declared:

312. pst of type `Pst`
313. pduIdx of type `uint8_t`
314. rxDataInd of type `RxDataInd`. This is a pointer that points to nothing (NULLP). 
315. pdu of type `RxDataIndPdu`.
***
**Allocating Memory for `rxDataInd`**
```c=317
   MAC_ALLOC_SHRABL_BUF(rxDataInd, sizeof(RxDataInd));
```
Allocate shared memory to be used for LWLC during inter-layer communication. If the memory allocation fails, an error message is logged, and the function returns RFAILED to indicate failure.
```c=318
   if(!rxDataInd)
   {
      DU_LOG("\nERROR  -->  LWR_MAC : Memory Allocation failed in procRxDataInd");
      return RFAILED;
   }
```
***

**Checking the PDUs**
```c=323
   if(!fapiRxDataInd->numPdus)
   {
      DU_LOG("\nDEBUG  -->  LWR_MAC : No PDU in RX_Data.indication at [%d, %d]", fapiRxDataInd->sfn, fapiRxDataInd->slot);
      return ROK;
   }
```
If there are no PDUs (Protocol Data Units) in the `fapiRxDataInd` structure, a debug message is logged, and the function returns **ROK** to indicate success.
*** 
**Filling the `rxDataIndication`'s paremeters**
```c=329
   rxDataInd->cellId = lwrMacCb.cellCb[0].cellId;
   rxDataInd->timingInfo.sfn = fapiRxDataInd->sfn; 
   rxDataInd->timingInfo.slot = fapiRxDataInd->slot;
   rxDataInd->numPdus = fapiRxDataInd->numPdus;
```
***
**Looping for each PDUs to fill each of PDUs**
```c=334
   for(pduIdx = 0; pduIdx < rxDataInd->numPdus; pduIdx++)
   {
      pdu = &rxDataInd->pdus[pduIdx];
      pdu->handle = fapiRxDataInd->pdus[pduIdx].handle;
      pdu->rnti = fapiRxDataInd->pdus[pduIdx].rnti;
      pdu->harqId = fapiRxDataInd->pdus[pduIdx].harqId;
      pdu->pduLength = fapiRxDataInd->pdus[pduIdx].pdu_length;
      pdu->ul_cqi = fapiRxDataInd->pdus[pduIdx].ul_cqi;
      pdu->timingAdvance = fapiRxDataInd->pdus[pduIdx].timingAdvance;
      pdu->rssi = fapiRxDataInd->pdus[pduIdx].rssi;

      MAC_ALLOC_SHRABL_BUF(pdu->pduData, pdu->pduLength);
      memcpy(pdu->pduData, fapiRxDataInd->pdus[pduIdx].pduData, pdu->pduLength);
#ifdef INTEL_WLS_MEM      
      /* Free WLS memory allocated for Rx PDU */
      WLS_MEM_FREE(fapiRxDataInd->pdus[pduIdx].pduData, LWR_MAC_WLS_BUF_SIZE);
#endif
   }
```
* For each PDUs, they will be filled with the `fapiRXDataInd`'s elements value. 
* Memory is allocated for the PDU data using the MAC_ALLOC_SHRABL_BUF macro, and the PDU data is copied from fapiRxDataInd to pdu->pduData.
* If the code is compiled with the INTEL_WLS_MEM flag defined, additional steps are taken to free memory allocated for the Rx PDU in the WLS (Wireless Link Simulation) environment.

***
**Sending the Data to MAC Layer**
```c=353
   /* Fill post and sent to MAC */
   FILL_PST_LWR_MAC_TO_MAC(pst, EVENT_RX_DATA_IND_TO_MAC);
   return (*sendRxDataIndOpts[pst.selector])(&pst, rxDataInd);
}
```
* After processing all PDUs, the pst variable is filled with the appropriate values to send the RX data indication event to the MAC layer.
* Finally, the function calls the appropriate function pointer `sendRxDataIndOpts[pst.selector]` to **send the RX data indication to the MAC layer**, passing the pst and rxDataInd structures as arguments.
:::

### 5. `fapiMacRxDataInd` (function)
> File Directory : .\odu_l2\l2\src\5gnrmac\mac_msg_hdl.c

:::success
**Specific Information**
This function will process RX Data Ind at MAC Layer
:::

:::info

**Declaration**
```c
uint8_t fapiMacRxDataInd(Pst *pst, 
                         RxDataInd *rxDataInd)
```

**Functionality**
Process Rx Data Ind at MAC

**Parameters**

| Field     | Type   | Description           |
| --------- | ------ | --------------------- |
|*pst|pointer|Post structure|
| RxDataInd | Struct | Struct of Rx Data Ind |


**Return Value**
* ROK:arrow_right: Success
* RFAILUE:arrow_right: Failure

**Details**
:::spoiler
:::warning
**Initialization and Debug Log**
```c=215
   uint8_t ueId = 0;
   uint16_t pduIdx, cellIdx = 0;
   DU_LOG("\nDEBUG  -->  MAC : Received Rx Data indication");
```
***
**For the number of PDUs, Getting the Cell Id and Unpacking the RxData**
```c=221
   for(pduIdx = 0; pduIdx < rxDataInd->numPdus; pduIdx++)
   {

      GET_CELL_IDX(rxDataInd->cellId, cellIdx);
      GET_UE_ID(rxDataInd->pdus[pduIdx].rnti, ueId);
      
      if(macCb.macCell[cellIdx] && macCb.macCell[cellIdx]->ueCb[ueId -1].transmissionAction == STOP_TRANSMISSION)
      {
         DU_LOG("\nINFO   -->  MAC : UL data transmission not allowed for UE %d", macCb.macCell[cellIdx]->ueCb[ueId -1].ueId);
      }
      else
      {
         unpackRxData(rxDataInd->cellId, rxDataInd->timingInfo, &rxDataInd->pdus[pduIdx]);
      }
      MAC_FREE_SHRABL_BUF(pst->region, pst->pool, rxDataInd->pdus[pduIdx].pduData,\
         rxDataInd->pdus[pduIdx].pduLength);
   }
   MAC_FREE_SHRABL_BUF(pst->region, pst->pool, rxDataInd, sizeof(RxDataInd));
   return ROK;
```
:::


### Message Sequence (MSC)
:::success
![](https://hackmd.io/_uploads/SkEhk37Fh.png)
:::warning
There are several function that I haven't traced (**`fapiMacRxDataInd()`** and **`unpackRxData()`**). I will complete this on future.

:::

## UL_TTI.request
### 1. `l1HdlUlTtiReq` (Ending Procedure)
> File Directory :.\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**Specific Information**
As we know, Uplink transmission direction is from the UE (PHY layer) to the gNB/base station (MAC Layer). So, when it wants to **REQUEST**, the direction is changed. ==Meaning that gNB wants to send a request to UE to send an uplink data==.

So, **this function is the ending point** of this procedure and we want to trace the code **upwards**.
:::

:::info
**Declaration**
```c
S16 l1HdlUlTtiReq(uint16_t msgLen, 
                  void *msg)puschPdu)
```
**Functionality**
Handles Ul Tti request received from MAC

**Parameters**
| Field  | Type     | Description         |
| ------ | -------- |:------------------- |
| msgLen | uint16_t | Message Length      |
| *msg   |  | UL_TTI.request message pointer|


**Return Value**
None

**Details**
:::spoiler
:::warning
**Initialization**
```c=1212
   p_fapi_api_queue_elem_t ulTtiElem = (p_fapi_api_queue_elem_t)msg;
   fapi_ul_tti_req_t *ulTtiReq = (fapi_ul_tti_req_t *)(ulTtiElem +1);
   uint8_t numPdus = ulTtiReq->nPdus;
```
The `fapi_ul_tti_req_t` struct can be seen below:
```c=1
    typedef struct {
        fapi_msg_t header;
        uint16_t sfn;
        uint16_t slot;
        uint8_t nPdus;
        uint8_t rachPresent;
        uint8_t nUlsch;
        uint8_t nUlcch;
        uint8_t nGroup;
        uint8_t pad[3];
        fapi_ul_tti_req_pdu_t pdus[FAPI_MAX_NUMBER_UL_PDUS_PER_TTI];    // 5G FAPI Table 3-44
        fapi_ue_info_t ueGrpInfo[FAPI_MAX_NUMBER_OF_GROUPS_PER_TTI];
    } fapi_ul_tti_req_t;
```
***

**Debugging to Check If There is a PDUs in `UL_TTI.request`**
```c=1216
#ifdef ODU_SLOT_IND_DEBUG_LOG
   if(numPdus == 0)
   {
      DU_LOG("\nINFO   -->  PHY STUB: No PDU received in UL TTI Req");
   }
   else
   {
      DU_LOG("\nINFO   -->  PHY STUB: Received UL TTI Request");
   }
#endif 
```
Here's the explanation:
* If there is no PDUs, the system will tell that there is no PDUs received in `uL_TTI.request` message.
* If there are PDUs in the syste, the PHY stub will tell that the PDUs have been received.
***

**Checking the PDUs Type**
```c=1227
   while(numPdus)
   {
      if(ulTtiReq->pdus[numPdus-1].pduType == 0)
      {
         DU_LOG("\nINFO   -->  PHY STUB: PRACH PDU");

         /* Send RACH Ind to L2 for first UE */
         if(phyDb.ueDb.ueCb[UE_IDX_0].rachIndSent == false)
         {
            phyDb.ueDb.ueCb[UE_IDX_0].isCFRA = false;
            phyDb.ueDb.ueCb[UE_IDX_0].rachIndSent = true;
            l1BuildAndSendRachInd(ulTtiReq->slot, ulTtiReq->sfn, CB_RA_PREAMBLE_IDX);
            phyDb.ueDb.numActvUe++;
         }
          
      }
      if(ulTtiReq->pdus[numPdus-1].pduType == 1)
      {
         DU_LOG("\nINFO   -->  PHY STUB: PUSCH PDU");
         if (ROK == l1BuildAndSendCrcInd(ulTtiReq->slot, ulTtiReq->sfn,ulTtiReq->pdus[numPdus-1].pdu.pusch_pdu))
         {
            l1BuildAndSendRxDataInd(ulTtiReq->slot, ulTtiReq->sfn, ulTtiReq->pdus[numPdus-1].pdu.pusch_pdu); 
         }
      }
       
      if(ulTtiReq->pdus[numPdus-1].pduType == 2)
      {
         DU_LOG("\nINFO   -->  PHY STUB: PUCCH PDU");

         fapi_ul_tti_req_t ulTtiSlotInd;
         memset(&ulTtiSlotInd, 0, sizeof(fapi_ul_tti_req_t));
         ulTtiSlotInd.slot = ulTtiReq->slot;
         ulTtiSlotInd.sfn  = ulTtiReq->sfn;
         ADD_DELTA_TO_TIME(ulTtiSlotInd, ulTtiSlotInd, SLOT_DELAY, MAX_SLOTS);
         l1BuildAndSendUciInd(ulTtiSlotInd.slot, ulTtiSlotInd.sfn, ulTtiReq->pdus[numPdus-1].pdu.pucch_pdu);
      }
      numPdus--;
   }

```
* If the **PDU type is 0** (**indicating a PRACH PDU**), the code checks if a RACH Indication has been sent for a specific UE (UE_IDX_0) and sends it if not. It increments the number of active UEs (numActvUe) in the phyDb database.
* If the **PDU type is 1** (**indicating a PUSCH PDU**), the code builds and sends a CRC Indication (l1BuildAndSendCrcInd) and a Rx Data Indication ([`l1BuildAndSendRxDataInd`](#l1BuildAndSendRxDataInd-MainStarting-Function)).
* If the **PDU type is 2** (**indicating a PUCCH PDU**), the code creates a new UL TTI request (ulTtiSlotInd) with updated slot information. It then builds and sends a UCI Indication (l1BuildAndSendUciInd).
:::

### 2. `l1ProcessFapiRequest` (procedure)
> File Directory :.\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**Specific Information**
If we trace [`l1HdlUlTtiReq`](#l1HdlUlTtiReq-Ending-Procedure) backwards, that procedure is being called by `l1ProcessFapiRequest` procedure.

So, basically this procedure is to receive message from MAC and calls handler (and the handler is [`l1HdlUlTtiReq`](#l1HdlUlTtiReq-Ending-Procedure) that we already traced before). 
:::

:::info
**Declaration**
```c
void l1ProcessFapiRequest(uint8_t msgType, 
                          uint32_t msgLen, 
                          void *msg)
```
**Functionality**
Receives message from MAC and calls handler

**Parameters**
| Parameter | Type | Description |
| --------- | ---- | ----------- |
|msgType|uint8_t|This parameter defines the message type based on the header.|
|msgLen|uint32_t|The length of the message |
|*msg |  | Message pointer|

**Return Value**
None

**Details**
:::spoiler
:::warning
The entire code of this procedure can be seen below:
```c=1784
{
   switch(msgType)
   {
#ifdef INTEL_FAPI
      case FAPI_PARAM_REQUEST:
         l1HdlParamReq(msgLen, msg);
         break;
      case FAPI_CONFIG_REQUEST:
         l1HdlConfigReq(msgLen, msg);
         break;
      case FAPI_START_REQUEST:
         l1HdlStartReq(msgLen, msg);
         break;
      case FAPI_DL_TTI_REQUEST:
         l1HdlDlTtiReq(msgLen, msg);
         break;
      case FAPI_TX_DATA_REQUEST:
         l1HdlTxDataReq(msgLen, msg);
         break;
      case FAPI_UL_TTI_REQUEST:
         l1HdlUlTtiReq(msgLen, msg);
         break;
      case FAPI_STOP_REQUEST:
         l1HdlStopReq(msgLen, msg);
         break;
      case FAPI_UL_DCI_REQUEST:
         l1HdlUlDciReq(msgLen, msg);
         break;
      default:
         DU_LOG("\nERROR  -->  PHY_STUB: Invalid message type[%x] received at PHY", msgType);
         break;
#endif
   }
}
```
We can see that the [`l1HdlUlTtiReq`](#l1HdlUlTtiReq-Ending-Procedure) is called in this function.

Basically this procedure is to choose what is the next procedure to handle  the message based on the msgType that we input on this procedure. Based on the Ul_TTI.request message, this procedure will call `l1HdlUlTtiReq` procedure.
:::

### 3. `LwrMacSendToL1` (function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_phy.c

:::success
**Specific Information**
If we trace [`l1ProcessFapiRequest`](#l1ProcessFapiRequest-procedure) backwards, that procedure is being called by `LwrMacSendToL1` procedure.
:::

:::info
**Declaration**
```c
uint8_t LwrMacSendToL1(void *msg)
```
**Functionality**
Send FAPI messages to Intel PHY/Phy stub

**Parameters**
| Parameter | Type | Description |
| --------- | ---- | ----------- |
|*msg |  | Message pointer|

**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler
:::warning
**Initialization and Debug Log**
```c=271
{
   uint8_t ret = ROK;
#ifdef INTEL_FAPI
   uint16_t msgLen =0;
   p_fapi_api_queue_elem_t currMsg = NULLP;

#ifdef CALL_FLOW_DEBUG_LOG   
   char message[100];
```
* Initializes a local variable ret with the value ROK.
* Declares a local variable msgLen of type uint16_t and initializes it to 0.
* It declares a pointer currMsg of type `p_fapi_api_queue_elem_t` and initializes it to NULLP (null pointer).
*** 

**Debug Log**
```c=280
   currMsg = (p_fapi_api_queue_elem_t)msg;
   while(currMsg)
   {
      switch(currMsg->msg_type)
      {
         case FAPI_PARAM_REQUEST:
            strcpy(message, "FAPI_PARAM_REQUEST");
            break;
         case FAPI_CONFIG_REQUEST:
            strcpy(message, "FAPI_CONFIG_REQUEST");
            break;
         case FAPI_START_REQUEST:
            strcpy(message, "FAPI_START_REQUEST");
            break;
         case FAPI_DL_TTI_REQUEST:
            strcpy(message, "FAPI_DL_TTI_REQUEST");
            break;
         case FAPI_TX_DATA_REQUEST:
            strcpy(message, "FAPI_TX_DATA_REQUEST");
            break;
         case FAPI_UL_TTI_REQUEST:
            strcpy(message, "FAPI_UL_TTI_REQUEST");
            break;
         case FAPI_STOP_REQUEST:
            strcpy(message, "FAPI_STOP_REQUEST");
            break;
         case FAPI_UL_DCI_REQUEST:
            strcpy(message, "FAPI_UL_DCI_REQUEST");
            break;
         default:
            strcpy(message, "INVALID_MSG");
            break;
      }
      DU_LOG("\nCall Flow: ENTLWRMAC -> PHY : %s\n",message);
      currMsg = currMsg->p_next;
   }
#endif
```
Provide a mechanism for logging the call flow between the MAC layer and PHY layer, specifically for messages of different types.
***

**Sending the Message to be Processed by the Next Procedure**
```c=372
   p_fapi_api_queue_elem_t nextMsg = NULLP;

   /* FAPI header and vendor specific msgs are freed here. Only 
    * the main FAPI messages are sent to phy stub */
   currMsg = (p_fapi_api_queue_elem_t)msg;
   while(currMsg)
   {
      nextMsg = currMsg->p_next;
      msgLen = currMsg->msg_len + sizeof(fapi_api_queue_elem_t);
      if((currMsg->msg_type != FAPI_VENDOR_MSG_HEADER_IND) && \
	    (currMsg->msg_type != FAPI_VENDOR_MESSAGE))
      {
	 l1ProcessFapiRequest(currMsg->msg_type, msgLen, currMsg);
      }
      else
      {
	 LWR_MAC_FREE(currMsg, msgLen);   
      }
      currMsg = nextMsg;
   }
#endif
#endif
   return ret;
```
:::

### 4. `fillDlTtiReq` (Starting Function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_fsm.c

:::success
**Specific Information**
This function is the **starting point** for sending the `UL_TTI.request` message.

DL_TTI.request is always in a pair with the UL_TTI.request message when they want to form.
:::

:::info
**Declaration**
```c
uint16_t fillDlTtiReq(SlotTimingInfo currTimingInfo)
```
**Functionality**
Sends FAPI DL and UL TTI req to PHY

**Parameters**
| Parameter | Type | Description |
| --------- | ---- | ----------- |
|*currTimingInfo | struct | This is the timing info of UL/DL TTI request|

**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler
:::warning

:::

### Message Sequence (MSC)
:::success
![](https://hackmd.io/_uploads/rkp_tRQYn.png)

:::

## SLOT.indication
### 1. `GenerateTicks` (Starting Fucntion)
> File Directory : .\odu_l2\l2\src\phy_stub\phy_stub_thread_hdl.c

:::success
**Specific Information**
This function is the **starting point** for sending the `SLOT.indication` message.

This function will generate ticks to produces/generates slot indications.
:::

:::info
**Declaration**
```c
void GenerateTicks()
```
**Functionality**
Sends FAPI DL and UL TTI req to PHY

**Parameters**
None

**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler
:::warning
**Initialization**
```c=51
{
#ifdef NR_TDD
   float     milisec = 0.5;        /* 0.5ms */
#else
   float     milisec = 1;          /* 1ms */
#endif
   struct timespec req = {0};
   uint8_t ratio = 2;

   slotIndicationStarted = true;
   req.tv_sec = 0;
```
***
**Send Slot Indication to MAC by calling [`l1BuildAndSendSlotIndication`](#2-l1BuildAndSendSlotIndication-procedure) procedure**

```c=67
   req.tv_nsec = milisec * 1000000L * ratio;

   DU_LOG("\nPHY_STUB : GenerateTicks : Starting to generate slot indications");

   while(slotIndicationStarted)
   {
      clock_nanosleep(CLOCK_REALTIME, 0, &req, NULL); 
      /* Send Slot indication indication to lower mac */
      if(l1BuildAndSendSlotIndication() != ROK)
      {
         DU_LOG("\nERROR  --> PHY_STUB : GenerateTicks(): Failed to build and send Slot Indication");
         return;
      }
   }
```
:::

### 2. `l1BuildAndSendSlotIndication` (procedure)
> File Directory : .\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**Specific Information**
This function builds and send the Slot Indication message to MAC.
:::

:::info
**Declaration**
```c
uint16_t l1BuildAndSendSlotIndication()
```
**Functionality**
Send the Slot indication Message to MAC

**Parameters**
None

**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler
:::warning
**Allocating Memory for Slot Indication Message**

```c=827
#ifdef INTEL_FAPI
   fapi_slot_ind_t *slotIndMsg;

   MAC_ALLOC_SHRABL_BUF(slotIndMsg, sizeof(fapi_slot_ind_t));
   if(!slotIndMsg)
   {
      DU_LOG("\nERROR  -->  PHY_STUB: Memory allocation failed for slot Indication Message");
      return RFAILED;
   }
   else
   {
      memset(slotIndMsg, 0, sizeof(fapi_slot_ind_t));
      slotIndMsg->sfn = sfnValue;
      slotIndMsg->slot = slotValue;
```
***

```c=846
      /* increment for the next TTI */
      slotValue++;
      if(sfnValue >= MAX_SFN_VALUE && slotValue > MAX_SLOT_VALUE)
      {
         sfnValue = 0;
         slotValue = 0;
      }
      else if(slotValue > MAX_SLOT_VALUE)
      {
         sfnValue++;
         slotValue = 0;
      }
      fillMsgHeader(&slotIndMsg->header, FAPI_SLOT_INDICATION, \
            sizeof(fapi_slot_ind_t) - sizeof(fapi_msg_t));

      memset(&pst, 0, sizeof(Pst));
      FILL_PST_PHY_STUB_TO_LWR_MAC(pst, EVT_PHY_STUB_SLOT_IND);
      ODU_GET_MSG_BUF(pst.region, pst.pool, &mBuf);
      if(!mBuf)
      {
         DU_LOG("\nERROR  --> PHY_STUB: Failed to allocate memory for slot indication buffer");
         MAC_FREE_SHRABL_BUF(pst.region, pst.pool, slotIndMsg, sizeof(fapi_slot_ind_t));
         return RFAILED;
      }
      CMCHKPK(oduPackPointer, (PTR)slotIndMsg, mBuf);
      ODU_POST_TASK(&pst, mBuf);

```
* The sfnValue and slotValue variables are **incremented** to prepare for the next TTI (Transmission Time Interval).
    * If the maximum SFN (Subframe Number) value (MAX_SFN_VALUE) is reached and the slot value exceeds the maximum slot value (MAX_SLOT_VALUE), both sfnValue and slotValue are reset to 0.
    * If only the slot value exceeds the maximum slot value, sfnValue is incremented by 1, and slotValue is reset to 0.
* The message header of the slotIndMsg is filled with the appropriate values using the `fillMsgHeader` function.
* The `Pst` structure (pst) is initialized, and the appropriate fields are filled to indicate that the message is being sent from the PHY Stub to the LWR MAC layer. The event is set as **`EVT_PHY_STUB_SLOT_IND`**.
* The function returns ROK to indicate a successful execution.
:::

### 3. `lwrMacActvTsk` (Task)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_ex_ms.c

:::success
**Specific Information**
This is a task that specfied to run the slot indication event (`EVT_PHY_STUB_SLOT_IND`) on the previous function, [`l1BuildAndSendSlotIndication`](#2-l1BuildAndSendSlotIndication-procedure).
:::

:::info
**Declaration**
```c
uint8_t lwrMacActvTsk(Pst *pst, 
                      Buffer *mBuf)
```
**Functionality**
Task Activation callback function. 

**Parameters**


| Field | Type   | Description                                              |
| ----- | ------ | -------------------------------------------------------- |
|*pst|Struct|Post structure of the primitive.|
| *mBuf | Struct | Struct Buffer. Packed primitive parameters in the buffer |


**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler


:::warning
**Initialization**
```c=161
   uint8_t ret = ROK;
```
* The function begins by declaring a variable ret and initializing it to ROK (indicating a successful execution).
***

**Switch Statement**
```c=167
#ifndef INTEL_WLS_MEM
      case ENTPHYSTUB:
         {
            switch(pst->event)
            {
               case EVT_PHY_STUB_SLOT_IND:
                  {
                     fapi_slot_ind_t *slotIndMsg;

                     CMCHKUNPK(oduUnpackPointer, (PTR *)&slotIndMsg, mBuf);
                     ODU_PUT_MSG_BUF(mBuf);

                     procPhyMessages(slotIndMsg->header.msg_id, sizeof(fapi_slot_ind_t), (void*)slotIndMsg);
                     MAC_FREE_SHRABL_BUF(pst->region, pst->pool, slotIndMsg, sizeof(fapi_slot_ind_t));
                     break;
                  }

               case EVT_PHY_STUB_STOP_IND:
                 {
                    fapi_stop_ind_t *stopIndMsg;
                    CMCHKUNPK(oduUnpackPointer, (PTR *)&stopIndMsg, mBuf);
                    ODU_PUT_MSG_BUF(mBuf);

                    procPhyMessages(stopIndMsg->header.msg_id, sizeof(fapi_stop_ind_t), (void*)stopIndMsg);
                    MAC_FREE_SHRABL_BUF(pst->region, pst->pool, stopIndMsg, sizeof(fapi_stop_ind_t));
                    break;
                 }
               default:
                  {
                     DU_LOG("\nERROR  -->  LWR_MAC: Invalid event %d received from PHY STUB", pst->event);
                  }
            }
            break;
         }
#endif
```
* **Case EVT_PHY_STUB_SLOT_IND:**
    * When the event is EVT_PHY_STUB_SLOT_IND, it means a Slot Indication message is received from the PHY Stub layer.
    * The message buffer (mBuf) is unpacked to extract the slotIndMsg (Slot Indication message) using the CMCHKUNPK macro.
    * The unpacked slotIndMsg is then freed using ODU_PUT_MSG_BUF since it is no longer needed.
    * The procPhyMessages function is called to process the received Slot Indication message. The message ID and size are passed as parameters to the function.
    * Finally, the slotIndMsg is freed using MAC_FREE_SHRABL_BUF to release the memory allocated for it.

* **Case EVT_PHY_STUB_STOP_IND:**
    * When the event is EVT_PHY_STUB_STOP_IND, it means a Stop Indication message is received from the PHY Stub layer.
    * The message buffer (mBuf) is unpacked to extract the stopIndMsg (Stop Indication message) using the CMCHKUNPK macro.
    * The unpacked stopIndMsg is then freed using ODU_PUT_MSG_BUF since it is no longer needed.
    * The procPhyMessages function is called to process the received Stop Indication message. The message ID and size are passed as parameters to the function.
    * Finally, the stopIndMsg is freed using MAC_FREE_SHRABL_BUF to release the memory allocated for it.
:::

### Message Sequence (MSC)
:::success
![](https://hackmd.io/_uploads/SyR3WLEF3.png)

:::

## TX_DATA.request
### 1. `l1HdlTxDataReq` (Ending Procedure)
> File Directory :.\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**Specific Information**
The TX_Data.request is sent from MAC to PHY layer. And this procedure is the ending procedure of handling the TX Data Request message from MAC.

So, **this function is the ending point** of this procedure and we want to trace the code **upwards**.
:::

:::info
**Declaration**
```c
S16 l1HdlTxDataReq(uint16_t msgLen, 
                   void *msg)
```
**Functionality**
Handles tx_data request received from MAC

**Parameters**
| Field  | Type     | Description         |
| ------ | -------- |:------------------- |
| msgLen | uint16_t | Message Length      |
| *msg   |  | TX_DATA.request message pointer|


**Return Value**
None

**Details**
:::spoiler
:::warning
```c=993
{
#ifdef INTEL_FAPI
   p_fapi_api_queue_elem_t txDataElem = (p_fapi_api_queue_elem_t)msg;
   fapi_tx_data_req_t *txDataReq = (fapi_tx_data_req_t *)(txDataElem +1);

   DU_LOG("\nINFO   -->  PHY STUB: TX DATA Request at sfn=%d slot=%d",txDataReq->sfn,txDataReq->slot);
/*
   if(dlDedMsg)
   {
      DU_LOG("\nINFO   -->  PHY_STUB: TxDataPdu for DED MSG sent");
      dlDedMsg = false;
   }
*/
   MAC_FREE(msg, msgLen);
#endif
   return ROK;
}
```
Here's the explanation:
* The function receives a `msg` parameter, which is a void pointer to the message data.
* The `msg` is cast to a `p_fapi_api_queue_elem_t` type, which represents a FAPI API queue element.
* The `txDataElem` is assigned the value of the casted msg pointer.
* The `txDataReq` is set to point to the actual `fapi_tx_data_req_t` structure within the txDataElem. This structure contains the information about the transmission data request.
* A log statement is printed to indicate that a TX DATA Request has been received, specifying the **SFN** (Subframe Number) and **slot values** extracted from txDataReq.
* The msg and msgLen are freed using `MAC_FREE` to release the allocated memory for the message.
* The function returns **ROK**, indicating successful execution.
:::

### 2. `l1ProcessFapiRequest` (procedure)
> File Directory :.\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**Specific Information**
If we trace [`l1HdlTxDataReq`](#1-l1HdlTxDataReq-Ending-Procedure) backwards, that procedure is being called by `l1ProcessFapiRequest` procedure.

So, basically this procedure is to receive message from MAC and calls handler (and the handler is [`l1HdlTxDataReq`](#1-l1HdlTxDataReq-Ending-Procedure) that we already traced before). 
:::

:::info
**Declaration**
```c
void l1ProcessFapiRequest(uint8_t msgType, 
                          uint32_t msgLen, 
                          void *msg)
```
**Functionality**
Receives message from MAC and calls handler

**Parameters**
| Parameter | Type | Description |
| --------- | ---- | ----------- |
|msgType|uint8_t|This parameter defines the message type based on the header.|
|msgLen|uint32_t|The length of the message |
|*msg |  | Message pointer|

**Return Value**
None

**Details**
:::spoiler
:::warning
The entire code of this procedure can be seen below:
```c=1784
{
   switch(msgType)
   {
#ifdef INTEL_FAPI
      case FAPI_PARAM_REQUEST:
         l1HdlParamReq(msgLen, msg);
         break;
      case FAPI_CONFIG_REQUEST:
         l1HdlConfigReq(msgLen, msg);
         break;
      case FAPI_START_REQUEST:
         l1HdlStartReq(msgLen, msg);
         break;
      case FAPI_DL_TTI_REQUEST:
         l1HdlDlTtiReq(msgLen, msg);
         break;
      case FAPI_TX_DATA_REQUEST:
         l1HdlTxDataReq(msgLen, msg);
         break;
      case FAPI_UL_TTI_REQUEST:
         l1HdlUlTtiReq(msgLen, msg);
         break;
      case FAPI_STOP_REQUEST:
         l1HdlStopReq(msgLen, msg);
         break;
      case FAPI_UL_DCI_REQUEST:
         l1HdlUlDciReq(msgLen, msg);
         break;
      default:
         DU_LOG("\nERROR  -->  PHY_STUB: Invalid message type[%x] received at PHY", msgType);
         break;
#endif
   }
}
```
We can see that the [`l1HdlTxDataReq`](#1-l1HdlTxDataReq-Ending-Procedure) is called in this function.

Basically this procedure is to choose what is the next procedure to handle  the message based on the msgType that we input on this procedure. Based on the TX_DATA.request message, this procedure will call `l1HdlTxDataReq` procedure.
:::

### 3. `LwrMacSendToL1` (function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_phy.c

:::success
**Specific Information**
If we trace [`l1ProcessFapiRequest`](#l1ProcessFapiRequest-procedure) backwards, that procedure is being called by `LwrMacSendToL1` procedure.
:::

:::info
**Declaration**
```c
uint8_t LwrMacSendToL1(void *msg)
```
**Functionality**
Send FAPI messages to Intel PHY/Phy stub

**Parameters**
| Parameter | Type | Description |
| --------- | ---- | ----------- |
|*msg |  | Message pointer|

**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler
:::warning
**Initialization and Debug Log**
```c=271
{
   uint8_t ret = ROK;
#ifdef INTEL_FAPI
   uint16_t msgLen =0;
   p_fapi_api_queue_elem_t currMsg = NULLP;

#ifdef CALL_FLOW_DEBUG_LOG   
   char message[100];
```
* Initializes a local variable ret with the value ROK.
* Declares a local variable msgLen of type uint16_t and initializes it to 0.
* It declares a pointer currMsg of type `p_fapi_api_queue_elem_t` and initializes it to NULLP (null pointer).
*** 

**Debug Log**
```c=280
   currMsg = (p_fapi_api_queue_elem_t)msg;
   while(currMsg)
   {
      switch(currMsg->msg_type)
      {
         case FAPI_PARAM_REQUEST:
            strcpy(message, "FAPI_PARAM_REQUEST");
            break;
         case FAPI_CONFIG_REQUEST:
            strcpy(message, "FAPI_CONFIG_REQUEST");
            break;
         case FAPI_START_REQUEST:
            strcpy(message, "FAPI_START_REQUEST");
            break;
         case FAPI_DL_TTI_REQUEST:
            strcpy(message, "FAPI_DL_TTI_REQUEST");
            break;
         case FAPI_TX_DATA_REQUEST:
            strcpy(message, "FAPI_TX_DATA_REQUEST");
            break;
         case FAPI_UL_TTI_REQUEST:
            strcpy(message, "FAPI_UL_TTI_REQUEST");
            break;
         case FAPI_STOP_REQUEST:
            strcpy(message, "FAPI_STOP_REQUEST");
            break;
         case FAPI_UL_DCI_REQUEST:
            strcpy(message, "FAPI_UL_DCI_REQUEST");
            break;
         default:
            strcpy(message, "INVALID_MSG");
            break;
      }
      DU_LOG("\nCall Flow: ENTLWRMAC -> PHY : %s\n",message);
      currMsg = currMsg->p_next;
   }
#endif
```
Provide a mechanism for logging the call flow between the MAC layer and PHY layer, specifically for messages of different types.
***

**Sending the Message to be Processed by the Next Procedure**
```c=372
   p_fapi_api_queue_elem_t nextMsg = NULLP;

   /* FAPI header and vendor specific msgs are freed here. Only 
    * the main FAPI messages are sent to phy stub */
   currMsg = (p_fapi_api_queue_elem_t)msg;
   while(currMsg)
   {
      nextMsg = currMsg->p_next;
      msgLen = currMsg->msg_len + sizeof(fapi_api_queue_elem_t);
      if((currMsg->msg_type != FAPI_VENDOR_MSG_HEADER_IND) && \
	    (currMsg->msg_type != FAPI_VENDOR_MESSAGE))
      {
	 l1ProcessFapiRequest(currMsg->msg_type, msgLen, currMsg);
      }
      else
      {
	 LWR_MAC_FREE(currMsg, msgLen);   
      }
      currMsg = nextMsg;
   }
#endif
#endif
   return ret;
```
:::

### 4. `sendTxDataReq` (function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_fsm.c

:::success
**Specific Information**
This function is called by `fillDlTtiReq` function (starting function) to fill the data that needed to send the TX_DATA.request
:::

:::info
**Declaration**
```c
uint16_t sendTxDataReq(SlotTimingInfo currTimingInfo,
                       MacDlSlot *dlSlot,
                       p_fapi_api_queue_elem_t prevElem,
                       fapi_vendor_tx_data_req_t *vendorTxDataReq)
```
**Functionality**
Sends FAPI TX data req to PHY

**Parameters**
| Parameter       | Type   | Description                                  |
| --------------- | ------ | -------------------------------------------- |
| currTimingInfo | struct | This is the timing info of UL/DL TTI request |
|*dlSlot|pointer|Pointer to MacDlSlot struct|
|prevElem|struct|Previous element of `p_fapi_api_queue_elem_t`(linked list header taht present at the top of all messages)|
|*vendorTxDataReq|pointer|pointer to `fapi_vendor_tx_data_req_t` struct|

**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler
:::warning
**Initialization and Getting the Cell Id**
```c=4083
   uint8_t  nPdu = 0;
   uint8_t  ueIdx=0;
   uint16_t cellIdx=0;
   uint16_t pduIndex = 0;
   fapi_tx_data_req_t       *txDataReq =NULLP;
   p_fapi_api_queue_elem_t  txDataElem = 0;

   GET_CELL_IDX(currTimingInfo.cellId, cellIdx);
```
***

**Fillinf the TX_DATA request Message**
```c=4094
   if(nPdu > 0)
   {
      LWR_MAC_ALLOC(txDataElem, (sizeof(fapi_api_queue_elem_t) + sizeof(fapi_tx_data_req_t)));
      if(txDataElem == NULLP)
      {
         DU_LOG("\nERROR  -->  LWR_MAC: Failed to allocate memory for TX data Request");
         return RFAILED;
      }

      FILL_FAPI_LIST_ELEM(txDataElem, NULLP, FAPI_TX_DATA_REQUEST, 1, \
            sizeof(fapi_tx_data_req_t));
      txDataReq = (fapi_tx_data_req_t *)(txDataElem +1);
      memset(txDataReq, 0, sizeof(fapi_tx_data_req_t));
      fillMsgHeader(&txDataReq->header, FAPI_TX_DATA_REQUEST, sizeof(fapi_tx_data_req_t));

      vendorTxDataReq->sym = 0;

      txDataReq->sfn  = currTimingInfo.sfn;
      txDataReq->slot = currTimingInfo.slot;
      if(dlSlot->dlInfo.brdcstAlloc.sib1TransmissionMode)
      {
         fillSib1TxDataReq(txDataReq->pdu_desc, pduIndex, &macCb.macCell[cellIdx]->macCellCfg, \
               &dlSlot->dlInfo.brdcstAlloc.sib1Alloc.sib1PdcchCfg->dci.pdschCfg);
         pduIndex++;
         MAC_FREE(dlSlot->dlInfo.brdcstAlloc.sib1Alloc.sib1PdcchCfg,sizeof(PdcchCfg));
         txDataReq->num_pdus++;
      }
      if(dlSlot->pageAllocInfo != NULLP)
      {
         fillPageTxDataReq(txDataReq->pdu_desc, pduIndex, dlSlot->pageAllocInfo);
         pduIndex++;
         txDataReq->num_pdus++;
         MAC_FREE(dlSlot->pageAllocInfo->pageDlSch.dlPagePdu, sizeof(dlSlot->pageAllocInfo->pageDlSch.dlPagePduLen));
         MAC_FREE(dlSlot->pageAllocInfo,sizeof(DlPageAlloc));
      }
```

4097. If the element is nothing (NULL), the Lwr MAC will failed and the funciton will return RFAILED.

***

**Filling the TX_DATA Request Message**
```c=4130
      for(ueIdx=0; ueIdx<MAX_NUM_UE; ueIdx++)
      {
         if(dlSlot->dlInfo.rarAlloc[ueIdx] != NULLP)
         {
            if((dlSlot->dlInfo.rarAlloc[ueIdx]->rarPdschCfg))
            {
               fillRarTxDataReq(txDataReq->pdu_desc, pduIndex, &dlSlot->dlInfo.rarAlloc[ueIdx]->rarInfo,\
                     dlSlot->dlInfo.rarAlloc[ueIdx]->rarPdschCfg);
               pduIndex++;
               txDataReq->num_pdus++;
               MAC_FREE(dlSlot->dlInfo.rarAlloc[ueIdx]->rarPdschCfg, sizeof(PdschCfg));
            }
            MAC_FREE(dlSlot->dlInfo.rarAlloc[ueIdx],sizeof(RarAlloc));
         }

         if(dlSlot->dlInfo.dlMsgAlloc[ueIdx] != NULLP)
         {
            if(dlSlot->dlInfo.dlMsgAlloc[ueIdx]->dlMsgPdschCfg) 
            {
               fillDlMsgTxDataReq(txDataReq->pdu_desc, pduIndex, \
                     dlSlot->dlInfo.dlMsgAlloc[ueIdx], \
                     dlSlot->dlInfo.dlMsgAlloc[ueIdx]->dlMsgPdschCfg);
               pduIndex++;
               txDataReq->num_pdus++;
               MAC_FREE(dlSlot->dlInfo.dlMsgAlloc[ueIdx]->dlMsgPdschCfg,sizeof(PdschCfg));
            }
            MAC_FREE(dlSlot->dlInfo.dlMsgAlloc[ueIdx]->dlMsgPdu, \
                  dlSlot->dlInfo.dlMsgAlloc[ueIdx]->dlMsgPduLen);
            dlSlot->dlInfo.dlMsgAlloc[ueIdx]->dlMsgPdu = NULLP;
            MAC_FREE(dlSlot->dlInfo.dlMsgAlloc[ueIdx], sizeof(DlMsgSchInfo));
         }
      }

      /* Fill message header */
      DU_LOG("\nDEBUG  -->  LWR_MAC: Sending TX DATA Request");
      prevElem->p_next = txDataElem;
   }
#endif
   return ROK;
}
``` 
:::

### 5. `fillDlTtiReq` (Starting Function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_fsm.c

:::success
**Specific Information**
This function is the **starting point** for sending the `TX_DATA.request` message.
:::

:::info
**Declaration**
```c
uint16_t fillDlTtiReq(SlotTimingInfo currTimingInfo)
```
**Functionality**
Sends FAPI DL and UL TTI req to PHY

**Parameters**
| Parameter | Type | Description |
| --------- | ---- | ----------- |
|*currTimingInfo | struct | This is the timing info of UL/DL TTI request|

**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler
:::warning
**Sending the TX Data req Message**
```c=3997
			   /* Intel L1 expects UL_TTI.request following DL_TTI.request */
			   fillUlTtiReq(currTimingInfo, dlTtiElem, &(vendorMsg->p7_req_vendor.ul_tti_req));
			   msgHeader->num_msg++;

			   /* Intel L1 expects UL_DCI.request following DL_TTI.request */
			   fillUlDciReq(dlTtiReqTimingInfo, dlTtiElem->p_next, &(vendorMsg->p7_req_vendor.ul_dci_req));
			   msgHeader->num_msg++;

			   /* send Tx-DATA req message */
			   sendTxDataReq(dlTtiReqTimingInfo, currDlSlot, dlTtiElem->p_next->p_next, &(vendorMsg->p7_req_vendor.tx_data_req));
			   if(dlTtiElem->p_next->p_next->p_next)
			   {
				   msgHeader->num_msg++;
				   prevElem = dlTtiElem->p_next->p_next->p_next;
			   }
			   else
				   prevElem = dlTtiElem->p_next->p_next;
		   }
		   else
		   {
               #ifdef ODU_SLOT_IND_DEBUG_LOG	    
			   DU_LOG("\nDEBUG  -->  LWR_MAC: Sending DL TTI Request");
#endif	    
			   /* Intel L1 expects UL_TTI.request following DL_TTI.request */
			   fillUlTtiReq(currTimingInfo, dlTtiElem, &(vendorMsg->p7_req_vendor.ul_tti_req));
			   msgHeader->num_msg++;

			   /* Intel L1 expects UL_DCI.request following DL_TTI.request */
			   fillUlDciReq(dlTtiReqTimingInfo, dlTtiElem->p_next, &(vendorMsg->p7_req_vendor.ul_dci_req));
			   msgHeader->num_msg++;

			   prevElem = dlTtiElem->p_next->p_next;
		   }
```
As we can see, this function will call [`sendTxDataReq`](#4-sendTxDataReq-function) function to fill the PDU for TX_DATA.request message.
***
**Sending the Message to Lwr L1**

```c=4039
		   prevElem->p_next = vendorMsgQElem;
		   LwrMacSendToL1(headerElem);
		   memset(currDlSlot, 0, sizeof(MacDlSlot));
		   return ROK;
```
:::

### Message Sequence (MSC)
:::success
![](https://hackmd.io/_uploads/rJSseHHY3.png)

:::



## DL_TTI.request
### 1. `l1HdlDlTtiReq` (Ending Procedure)
> File Directory :.\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**Specific Information**
As we know, Uplink transmission direction is from the UE (PHY layer) to the gNB/base station (MAC Layer). So, when it wants to **REQUEST**, the direction is changed. ==Meaning that gNB wants to send a request to UE to send an uplink data==.

So, **this function is the ending point** of this procedure and we want to trace the code **upwards**.
:::

:::info
**Declaration**
```c
S16 l1HdlDlTtiReq(uint16_t msgLen, 
                  void *msg)
```
**Functionality**
Handles Dl Tti request received from MAC

**Parameters**
| Field  | Type     | Description         |
| ------ | -------- |:------------------- |
| msgLen | uint16_t | Message Length      |
| *msg   |  | UL_TTI.request message pointer|


**Return Value**
None

**Details**
:::spoiler
:::warning
**Initialization**
```c=936
   p_fapi_api_queue_elem_t dlTtiElem = (p_fapi_api_queue_elem_t)msg;
   fapi_dl_tti_req_t *dlTtiReq = (fapi_dl_tti_req_t *)(dlTtiElem +1);
   uint8_t numPdus = dlTtiReq->nPdus;
```
The `fapi_ul_tti_req_t` struct can be seen below:
```c=1
    typedef struct {
        fapi_msg_t header;
        uint16_t sfn;
        uint16_t slot;
        uint8_t nPdus;
        uint8_t rachPresent;
        uint8_t nUlsch;
        uint8_t nUlcch;
        uint8_t nGroup;
        uint8_t pad[3];
        fapi_ul_tti_req_pdu_t pdus[FAPI_MAX_NUMBER_UL_PDUS_PER_TTI];    // 5G FAPI Table 3-44
        fapi_ue_info_t ueGrpInfo[FAPI_MAX_NUMBER_OF_GROUPS_PER_TTI];
    } fapi_ul_tti_req_t;
```
***

**Debugging to Check If There is a PDUs in `UL_TTI.request`**
```c=1216
#ifdef ODU_SLOT_IND_DEBUG_LOG
   if(numPdus == 0)
   {
      DU_LOG("\nINFO   -->  PHY STUB: No PDU received in UL TTI Req");
   }
   else
   {
      DU_LOG("\nINFO   -->  PHY STUB: Received UL TTI Request");
   }
#endif 
```
Here's the explanation:
* If there is no PDUs, the system will tell that there is no PDUs received in `uL_TTI.request` message.
* If there are PDUs in the syste, the PHY stub will tell that the PDUs have been received.
***

**Checking the PDUs Type**
```c=1227
   while(numPdus)
   {
      if(ulTtiReq->pdus[numPdus-1].pduType == 0)
      {
         DU_LOG("\nINFO   -->  PHY STUB: PRACH PDU");

         /* Send RACH Ind to L2 for first UE */
         if(phyDb.ueDb.ueCb[UE_IDX_0].rachIndSent == false)
         {
            phyDb.ueDb.ueCb[UE_IDX_0].isCFRA = false;
            phyDb.ueDb.ueCb[UE_IDX_0].rachIndSent = true;
            l1BuildAndSendRachInd(ulTtiReq->slot, ulTtiReq->sfn, CB_RA_PREAMBLE_IDX);
            phyDb.ueDb.numActvUe++;
         }
          
      }
      if(ulTtiReq->pdus[numPdus-1].pduType == 1)
      {
         DU_LOG("\nINFO   -->  PHY STUB: PUSCH PDU");
         if (ROK == l1BuildAndSendCrcInd(ulTtiReq->slot, ulTtiReq->sfn,ulTtiReq->pdus[numPdus-1].pdu.pusch_pdu))
         {
            l1BuildAndSendRxDataInd(ulTtiReq->slot, ulTtiReq->sfn, ulTtiReq->pdus[numPdus-1].pdu.pusch_pdu); 
         }
      }
       
      if(ulTtiReq->pdus[numPdus-1].pduType == 2)
      {
         DU_LOG("\nINFO   -->  PHY STUB: PUCCH PDU");

         fapi_ul_tti_req_t ulTtiSlotInd;
         memset(&ulTtiSlotInd, 0, sizeof(fapi_ul_tti_req_t));
         ulTtiSlotInd.slot = ulTtiReq->slot;
         ulTtiSlotInd.sfn  = ulTtiReq->sfn;
         ADD_DELTA_TO_TIME(ulTtiSlotInd, ulTtiSlotInd, SLOT_DELAY, MAX_SLOTS);
         l1BuildAndSendUciInd(ulTtiSlotInd.slot, ulTtiSlotInd.sfn, ulTtiReq->pdus[numPdus-1].pdu.pucch_pdu);
      }
      numPdus--;
   }

```
* If the **PDU type is 0** (**indicating a PRACH PDU**), the code checks if a RACH Indication has been sent for a specific UE (UE_IDX_0) and sends it if not. It increments the number of active UEs (numActvUe) in the phyDb database.
* If the **PDU type is 1** (**indicating a PUSCH PDU**), the code builds and sends a CRC Indication (l1BuildAndSendCrcInd) and a Rx Data Indication ([`l1BuildAndSendRxDataInd`](#l1BuildAndSendRxDataInd-MainStarting-Function)).
* If the **PDU type is 2** (**indicating a PUCCH PDU**), the code creates a new UL TTI request (ulTtiSlotInd) with updated slot information. It then builds and sends a UCI Indication (l1BuildAndSendUciInd).
:::

### 2. `l1ProcessFapiRequest` (procedure)
> File Directory :.\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**Specific Information**
If we trace [`l1HdlUlTtiReq`](#1-l1HdlDlTtiReq-Ending-Procedure) backwards, that procedure is being called by `l1ProcessFapiRequest` procedure.

So, basically this procedure is to receive message from MAC and calls handler (and the handler is [`l1HdlUlTtiReq`](#1-l1HdlDlTtiReq-Ending-Procedure) that we already traced before). 
:::

:::info
**Declaration**
```c
void l1ProcessFapiRequest(uint8_t msgType, 
                          uint32_t msgLen, 
                          void *msg)
```
**Functionality**
Receives message from MAC and calls handler

**Parameters**
| Parameter | Type | Description |
| --------- | ---- | ----------- |
|msgType|uint8_t|This parameter defines the message type based on the header.|
|msgLen|uint32_t|The length of the message |
|*msg |  | Message pointer|

**Return Value**
None

**Details**
:::spoiler
:::warning
The entire code of this procedure can be seen below:
```c=1784
{
   switch(msgType)
   {
#ifdef INTEL_FAPI
      case FAPI_PARAM_REQUEST:
         l1HdlParamReq(msgLen, msg);
         break;
      case FAPI_CONFIG_REQUEST:
         l1HdlConfigReq(msgLen, msg);
         break;
      case FAPI_START_REQUEST:
         l1HdlStartReq(msgLen, msg);
         break;
      case FAPI_DL_TTI_REQUEST:
         l1HdlDlTtiReq(msgLen, msg);
         break;
      case FAPI_TX_DATA_REQUEST:
         l1HdlTxDataReq(msgLen, msg);
         break;
      case FAPI_UL_TTI_REQUEST:
         l1HdlUlTtiReq(msgLen, msg);
         break;
      case FAPI_STOP_REQUEST:
         l1HdlStopReq(msgLen, msg);
         break;
      case FAPI_UL_DCI_REQUEST:
         l1HdlUlDciReq(msgLen, msg);
         break;
      default:
         DU_LOG("\nERROR  -->  PHY_STUB: Invalid message type[%x] received at PHY", msgType);
         break;
#endif
   }
}
```
We can see that the [`l1HdlUlTtiReq`](#l1HdlUlTtiReq-Ending-Procedure) is called in this function.

Basically this procedure is to choose what is the next procedure to handle  the message based on the msgType that we input on this procedure. Based on the Ul_TTI.request message, this procedure will call `l1HdlUlTtiReq` procedure.
:::

### 3. `LwrMacSendToL1` (function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_phy.c

:::success
**Specific Information**
If we trace [`l1ProcessFapiRequest`](#l1ProcessFapiRequest-procedure) backwards, that procedure is being called by `LwrMacSendToL1` procedure.
:::

:::info
**Declaration**
```c
uint8_t LwrMacSendToL1(void *msg)
```
**Functionality**
Send FAPI messages to Intel PHY/Phy stub

**Parameters**
| Parameter | Type | Description |
| --------- | ---- | ----------- |
|*msg |  | Message pointer|

**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler
:::warning
**Initialization and Debug Log**
```c=271
{
   uint8_t ret = ROK;
#ifdef INTEL_FAPI
   uint16_t msgLen =0;
   p_fapi_api_queue_elem_t currMsg = NULLP;

#ifdef CALL_FLOW_DEBUG_LOG   
   char message[100];
```
* Initializes a local variable ret with the value ROK.
* Declares a local variable msgLen of type uint16_t and initializes it to 0.
* It declares a pointer currMsg of type `p_fapi_api_queue_elem_t` and initializes it to NULLP (null pointer).
*** 

**Debug Log**
```c=280
   currMsg = (p_fapi_api_queue_elem_t)msg;
   while(currMsg)
   {
      switch(currMsg->msg_type)
      {
         case FAPI_PARAM_REQUEST:
            strcpy(message, "FAPI_PARAM_REQUEST");
            break;
         case FAPI_CONFIG_REQUEST:
            strcpy(message, "FAPI_CONFIG_REQUEST");
            break;
         case FAPI_START_REQUEST:
            strcpy(message, "FAPI_START_REQUEST");
            break;
         case FAPI_DL_TTI_REQUEST:
            strcpy(message, "FAPI_DL_TTI_REQUEST");
            break;
         case FAPI_TX_DATA_REQUEST:
            strcpy(message, "FAPI_TX_DATA_REQUEST");
            break;
         case FAPI_UL_TTI_REQUEST:
            strcpy(message, "FAPI_UL_TTI_REQUEST");
            break;
         case FAPI_STOP_REQUEST:
            strcpy(message, "FAPI_STOP_REQUEST");
            break;
         case FAPI_UL_DCI_REQUEST:
            strcpy(message, "FAPI_UL_DCI_REQUEST");
            break;
         default:
            strcpy(message, "INVALID_MSG");
            break;
      }
      DU_LOG("\nCall Flow: ENTLWRMAC -> PHY : %s\n",message);
      currMsg = currMsg->p_next;
   }
#endif
```
Provide a mechanism for logging the call flow between the MAC layer and PHY layer, specifically for messages of different types.
***

**Sending the Message to be Processed by the Next Procedure**
```c=372
   p_fapi_api_queue_elem_t nextMsg = NULLP;

   /* FAPI header and vendor specific msgs are freed here. Only 
    * the main FAPI messages are sent to phy stub */
   currMsg = (p_fapi_api_queue_elem_t)msg;
   while(currMsg)
   {
      nextMsg = currMsg->p_next;
      msgLen = currMsg->msg_len + sizeof(fapi_api_queue_elem_t);
      if((currMsg->msg_type != FAPI_VENDOR_MSG_HEADER_IND) && \
	    (currMsg->msg_type != FAPI_VENDOR_MESSAGE))
      {
	 l1ProcessFapiRequest(currMsg->msg_type, msgLen, currMsg);
      }
      else
      {
	 LWR_MAC_FREE(currMsg, msgLen);   
      }
      currMsg = nextMsg;
   }
#endif
#endif
   return ret;
```
:::

### 4. `fillDlTtiReq` (Starting Function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_fsm.c

:::success
**Specific Information**
This function is the **starting point** for sending the `DL_TTI.request` message.

DL_TTI.request is always in a pair with the UL_TTI.request message when they want to form.
:::

:::info
**Declaration**
```c
uint16_t fillDlTtiReq(SlotTimingInfo currTimingInfo)
```
**Functionality**
Sends FAPI DL and UL TTI req to PHY

**Parameters**
| Parameter | Type | Description |
| --------- | ---- | ----------- |
|*currTimingInfo | struct | This is the timing info of UL/DL TTI request|

**Return Value**
* ROK :arrow_right: Success
* RFAILED :arrow_right: Failure

**Details**
:::spoiler
:::warning

:::

### Message Sequence (MSC)
:::success
![](https://hackmd.io/_uploads/B1aKiIHY3.png)

:::

## UCI.indication
### 1. `l1BuildAndSendUciInd` (Main/Starting Function)
> File Directory : .\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**General Information**
This function will build and send `UCI.indication` message from l1 (PHY layer) to l2 (MAC layer). This function will call `fillUciPduInfo` and `fillPucchF0F1PduInfo` function.

Because of this, this function is the ==main/starting function==. 
:::

:::info
**Declaration**
```c
uint8_t l1BuildAndSendUciInd(uint16_t slot, 
                             uint16_t sfn, 
                             fapi_ul_pucch_pdu_t pucchPdu)
```
**Functionality**
Build and send Uci indication

**Parameters**
| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| sfn                   | uint16_t | SFN (Single Frequency Network). Value: 0->1023|
| slot                  | uint16_t | Slot. Value: 0->159|
|pucchPdu|fapi_ul_pucch_pdu_t|puschPdu is a `fapi_ul_pucch_pdu_t` type data. `fapi_ul_pucch_pdu_t` is a struct. |

**Return Value**
* ROK    :arrow_right: success
* RFAILED    :arrow_right: failure

:::

### 2. `fillUciPduInfo` (Function)
> File Directory : .\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**General Information**
This function is called by [`l1BuildAndSendUciInd`](#1-l1BuildAndSendUciInd-MainStarting-Function) to fill the UCI that needed by PDU information.
:::

:::info
**Declaration**
```c
uint8_t fillUciPduInfo(fapi_uci_pdu_info_t *uciPdu, 
                       fapi_ul_pucch_pdu_t pucchPdu)

```
**Functionality**
Build and send Uci indication

**Parameters**
| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| *uciPdu                   | fapi_uci_pdu_info_t | Pointer to uciPdu|
| pucchPdu                  | fapi_ul_pucch_pdu_t | PDU of PUCCH|

**Return Value**
* ROK    :arrow_right: success
* RFAILED    :arrow_right: failure
:::

### 3. `fillPucchF0F1PduInfo` (Function)
> File Directory : .\odu_l2\l2\src\phy_stub\phy_stub_msg_hdl.c

:::success
**General Information**
This function is called by the previous function to Fills Uci Ind Pdu Info carried on Pucch Format 0/Format 1
:::

:::info
**Declaration**
```c
uint8_t fillPucchF0F1PduInfo(fapi_uci_o_pucch_f0f1_t *pduInfo, 
                             fapi_ul_pucch_pdu_t pucchPdu)
```
**Functionality**
Fills Uci Ind Pdu Info carried on Pucch Format 0/Format 1

**Parameters**
| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| *pduInfo                   | fapi_uci_o_pucch_f0f1_t | PDU Info for checking|
| pucchPdu                  | fapi_ul_pucch_pdu_t | PDU of PUCCH|

**Return Value**
* ROK    :arrow_right: success
* RFAILED    :arrow_right: failure
:::
### 4. `procPhyMessages` (function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_handle_phy.c

:::success
**Specific Information**
This function will process the message from PHY. This function is called when we already fill the parameters for `phyUciInd` struct on [`l1BuildAndSendRxUciInd`](#1-l1BuildAndSendUciInd-MainStarting-Function) function. 
:::

:::info

**Declaration**
```c
void procPhyMessages(uint16_t msgType, 
                     uint32_t msgSize, 
                     void *msg)
```

**Functionality**
Processes message from PHY

**Parameters**

| Parameter | Type | Description |
| --------- | ---- | ----------- |
|msgType|uint16_t|This parameter defines the message type based on the header that we already filled on [`l1BuildAndSendRxUciInd`](#1-l1BuildAndSendUciInd-MainStarting-Function)|
|msgSize|uint32_t|The size of the message (`fapi_uci_indication_t` struct) |
|*msg |  | Pointer to the received message from the PHY layer|

**Return Value**
* ROK    :arrow_right: success
* RFAILED    :arrow_right: failure
:::



### 5. `procUciInd` (Function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_handle_phy.c

:::success
**General Information**
This function will handle the UCI Indication from PHY and sends to MAC
:::

:::info
**Declaration**
```c
uint8_t fillPucchF0F1PduInfo(fapi_uci_o_pucch_f0f1_t *pduInfo, 
                             fapi_ul_pucch_pdu_t pucchPdu)
```
**Functionality**
Handles Uci indication from PHY and sends to MAC

**Parameters**
| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| *fapiUciInd                   | fapi_uci_indication_t | fapi_uci_indication_t message pointer|

**Return Value**
* ROK    :arrow_right: success
* RFAILED    :arrow_right: failure
:::

### 6. `fillPucchF0F1PduInfo` (Function)
> File Directory : .\odu_l2\l2\src\5gnrmac\lwr_mac_handle_phy.c

:::success
**General Information**
Fills Uci Ind Pdu Info carried on Pucch Format 0/Format 1
:::

:::info
**Declaration**
```c
uint8_t fillUciIndPucchF0F1(UciPucchF0F1 *pduInfo, 
                            fapi_uci_o_pucch_f0f1_t *fapiPduInfo)

```
**Functionality**
Fills Uci Ind Pdu Info carried on Pucch Format 0/Format 1

**Parameters**
| Field                 | Type     | Description|
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| *pduInfo                   | UciPucchF0F1 | PDU Info for checking|
| *fapiPduInfo                  | fapi_uci_o_pucch_f0f1_t | PDU of PUCCH|

**Return Value**
* ROK    :arrow_right: success
* RFAILED    :arrow_right: failure
:::

### Message Sequence (MSC)
:::success
![](https://hackmd.io/_uploads/SysELmYt2.png)

:::

# Important Notes
There are several things that I learned from tracing all of those messages:
* `fillDlTTiReq` function is **really important** function. It's because this procedure's purpose is to fill the PDU for `UL_TTI.req`, `TX_DATA.req`, and `DL_TTI.req` mesages. ==Basically, this function will call another function to fill the corresponding PDU of these messages==.

:::success
Click [here](https://hackmd.io/@fauzanmuhammad/phy_stub_code_tracing_2) for the next part.

Click [here](https://hackmd.io/@fauzanmuhammad/phy_stub_datatypes) to check about the data types
:::

