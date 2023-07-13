# OAI CU and OSC L2 Integration Testing for Demo

[toc]

---
## Testing Result
### Goals
:::success
- [x] integrate OAI CU and OSC L2 rel. G
- [ ] integrate OAI CU and OSC DU rel. G -- **on-going**
:::
### Screenshot of Wireshark
- Tony's OAI CU with OSC L2 rel. E (the test bed of his thesis)
    ![](https://hackmd.io/_uploads/S1i3S_QSn.png)
- Tony's OAI CU with OSC L2 rel. G (open source)
    ![](https://hackmd.io/_uploads/BJrH1jVH2.png)
- CU Stub and OSC L2 rel. G (provided by Radysis)
    ![](https://hackmd.io/_uploads/ryx8ROumS3.png)
- Tony's OAI CU with OSC L2 rel. G (provided by Radysis)
    ![](https://hackmd.io/_uploads/rycxm5mBn.png)

---
## Tony's OAI CU and OSC L2 rel. G
### Clone the Code
```cpp=
# Tony's thesis, repo. information needs to be provided by Ferlinda
git clone https://github.com/ferlinda/bmwlab_tony_cu_du.git

# OSC l2 rel. G
git clone "https://gerrit.o-ran-sc.org/r/o-du/l2" -b g-release
```

### Compiling OSC L2
```cpp=
cd <directory>/l2/build/odu
 
# clean off the previous code (TDD mode)
make clean_odu MACHINE=BIT64 MODE=TDD

# compile DU binary code in TDD mode
make odu MACHINE=BIT64 MODE=TDD 
```

### Installing the Dependencies for OAI CU
```cpp=
cd <directory>/bmwlab_tony_cu_du/oai_cu

# export the environmental variables (directory of OAI source code)
source oaienv

# install dependencies
cd cmake_targets
./build_oai -I
```
### Compiling OAI CU
```cpp=
# compile all of the code
cd ~/bmwlab_tony_cu_du/oai_cu/cmake_targets
./build_oai --gNB --nrUE -w SIMU

# sync up the format of F1AP
./f1ap_codec_mod.sh

# compile the part of RAN
cd ~/bmwlab_tony_cu_du/oai_cu/cmake_targets
./build_oai --gNB
```

### Modifying Configuration of OAI CU
```cpp=
vim <directory of OAI CU>/bmwlab_tony_cu_du/oai_cu/targets/PROJECTS/GENERIC-NR-5GC/CONF/cu_gnb.conf
```
![](https://i.imgur.com/x1VbYgf.png =500x)
- set PLMN, it needs to be the same as O-DU Configuration
- set local address for O-CU
- set remote address for O-DU

### Configuring Virtual IP Address
```cpp=
# bind the IP addresses with "local"
sudo ifconfig lo:ODU 192.168.130.81 up
sudo ifconfig lo:CU_STUB 192.168.130.82 up
```

### Running the Code
```cpp=
# OAI CU (run CU first anyway)
cd ~/bmwlab_tony_cu_du/oai_cu/cmake_targets/ran_build/build/
sudo RFSIMULATOR=server ./nr-softmodem --rfsim --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/cu_gnb.conf

# OSC l2 rel. E
cd ~/bmwlab_tony_cu_du/osc_odu_high/bin/odu
sudo ./odu

# OSC l2 rel. G
cd ~/G_O-DU/l2/bin/odu
sudo ./odu
```

---
## Debug Log
- useful command for debug
```cpp=
# find the name of file
sudo find / -name <file name>

# capture the pcap packets
sudo tcpdump -i any -w test.pcap
```
### OAI CU -- Error on CMake
:::danger
- can not find the files while installing OAI CU

![](https://hackmd.io/_uploads/rkiktQmSn.png)
:::
- **solution $\downarrow$**
:::success
- these files can not be generated in Centos OS, so we need to generate the files in ubuntu 18.04.
```cpp=
# in ubuntu 18.04
cd ~/bmwlab_tony_cu_du/oai_cu/cmake_targets
./build_oai --gNB --nrUE -w SIMU
```
```cpp=
cd <the directory of copy for FLPT from ubuntu>
cp -r . /home/oran/bmwlab_tony_cu_du/oai_cu/cmake_targets/ran_build/build/CMakeFiles/FLPT_V2

cd <the directory of copy for FSPT from ubuntu>
cp -r . /home/oran/bmwlab_tony_cu_du/oai_cu/cmake_targets/ran_build/build/CMakeFiles/FSPT_V2
```
:::
:::danger
- environmental variable `PROTOBUF_C_FIELD_FLAG_ONEOF` is not declared here

![](https://hackmd.io/_uploads/S1THsHXSh.png)
:::
- **solution $\downarrow$**
:::success
- edit this file, add the environmental variables for protobuf
```cpp=
sudo vim  /usr/include/protobuf-c/protobuf-c.h
```
- append the declaration for protobuf, LSB is for `PROTOBUF_C_FIELD_FLAG_PACKED`, second bit is for `PROTOBUF_C_FIELD_FLAG_DEPRECATED`, third bit is for `PROTOBUF_C_FIELD_FLAG_ONEOF`

![](https://hackmd.io/_uploads/H1kn0rmHh.png)
```cpp=
PROTOBUF_C_FIELD_FLAG_ONEOF             = (1 << 2),
```
:::

### OSC DU -- Can not find `NR_UL_FREQ` and `NR_DL_FREQ`
:::danger
- can not find variables `NR_UL_FREQ` and `NR_DL_FREQ`

![](https://hackmd.io/_uploads/SyvyaDXB3.png)
:::
- **solution $\downarrow$**
:::success
- in file `/home/oran/G_O-DU/l2/src/du_app/du_cfg.c`
```cpp=166
#ifndef INTEL_WLS_MEM
  duCfgParam.macCellCfg.dlCarrCfg.freq = convertArfcnToFreqKhz(NR_DL_ARFCN);
#else
  duCfgParam.macCellCfg.dlCarrCfg.freq = NR_DL_FREQ;
#endif
```
```cpp=192
#ifndef INTEL_WLS_MEM
  duCfgParam.macCellCfg.dlCarrCfg.freq = convertArfcnToFreqKhz(NR_UL_ARFCN);
#else
  duCfgParam.macCellCfg.ulCarrCfg.freq =  NR_UL_FREQ;
#endif
```
- in file `/home/oran/G_O-DU/l2/src/cm/common_def.h`
```cpp=355
uint8_t countSetBits(uint32_t num);
uint32_t convertArfcnToFreqKhz(uint32_t arfcn);
```
- in file `/home/oran/G_O-DU/l2/src/cm/common_def.c`
```cpp=23
uint32_t arfcnFreqTable[3][5] = {
  {   3000,   5,         0,        0,   599999 }, /*index 0*/
  {  24250,  15,      3000,   600000,  2016666 }, /*index 1*/
  { 100000,  60,  24250.08,  2016667,  3279165 }, /*index 2*/
};
```
- provide functions for translation between ARFCN and Freq KHz
```cpp=445
uint32_t convertArfcnToFreqKhz(uint32_t arfcn)
{
   uint8_t indexTable = 0;
   uint32_t freq = 0;

   for(indexTable = 0; indexTable < 4; indexTable++)
   {
      if(arfcn <= arfcnFreqTable[indexTable][4])
      {
         freq = arfcnFreqTable[indexTable][2] + (arfcnFreqTable[indexTable][1] * (arfcn - arfcnFreqTable[indexTable][3]));
         return (freq*1000);
      }
   }
   DU_LOG("ERROR  -->  DUAPP: ARFCN vaid range is between 0 and 3279165");
   return (freq*1000);
}

uint32_t convertFreqToArfcn(uint32_t freq)
{
   uint8_t indexTable = 0;
   uint32_t arfcn = 0;

   for(indexTable = 0; indexTable < 4; indexTable++)
   {
      if(freq < arfcnFreqTable[indexTable][0])
      {
         arfcn = arfcnFreqTable[indexTable][3] + ((freq - arfcnFreqTable[indexTable][2]) / (arfcnFreqTable[indexTable][1]));
         return (arfcn);
      }
   }
   DU_LOG("ERROR  -->  DUAPP: FREQ vaid range is between 0 and 100000 MHz");
   return (arfcn);
}
```
:::
### OSC DU -- F1 Setup Request Failed
:::danger
- F1 Setup Request failed

![](https://hackmd.io/_uploads/S1P7uOmSh.png)
:::
- **solution $\downarrow$**
:::success
- in file `\l2\src\du_app\du_f1ap_msg_hdl.c`
- append the count of information elements to 5
```cpp=1452
elementCnt = (duCfgParam.duName != NULL) ? 5 : 4; // make sure element count is enough
```
- append new information elements for F1AP to OAI CU
```cpp=1501
#if 1 // make sure this is "1"
      /* DU name IE is of type printableString_t which wireshark is unable to decode.
       * However this string is decoded successfully on online decoders.
       * Since this is an optional IE and the value received in it are not
       * used as of now, eliminating this IE for now to avoid wireshark error.
       */
      /*DU Name*/
      if(duCfgParam.duName != NULL)
      {
         ieIdx++;
         f1SetupReq->protocolIEs.list.array[ieIdx]->id = ProtocolIE_ID_id_gNB_DU_Name;
         f1SetupReq->protocolIEs.list.array[ieIdx]->criticality = Criticality_ignore;
         f1SetupReq->protocolIEs.list.array[ieIdx]->value.present = F1SetupRequestIEs__value_PR_GNB_DU_Name;
         f1SetupReq->protocolIEs.list.array[ieIdx]->value.choice.GNB_DU_Name.size = strlen((char *)duCfgParam.duName);
         DU_ALLOC(f1SetupReq->protocolIEs.list.array[ieIdx]->value.choice.GNB_DU_Name.buf, \
               f1SetupReq->protocolIEs.list.array[ieIdx]->value.choice.GNB_DU_Name.size);
         if(f1SetupReq->protocolIEs.list.array[ieIdx]->value.choice.GNB_DU_Name.buf == NULLP)
         {
            break;
         }
         strcpy((char*)f1SetupReq->protocolIEs.list.array[ieIdx]->value.choice.GNB_DU_Name.buf,
               (char*)&duCfgParam.duName);
      }
#endif
```
:::