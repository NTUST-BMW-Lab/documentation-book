# O-RAN System Architecture
###### tags: `Guide book`

![](https://i.imgur.com/QjYwvzw.png)
Figure O-RAN System Architecture

**Components**:
<!-- ![](https://i.imgur.com/Uc1OUHm.png)
Figure 4.2 O-RAN -->
1. **Open Radio Unit (O-RU)**
    A node that ==maintain the low pysical (**Low PHY**) layer and the radio frequency (**RF**)== of the base station. In summary, O-RU handles the Physical layer (L1)
2. **Open Distributed Unit (O-DU)**
    A node that ==maintain multiple O-RUs== that can functions as **baseband processing** unit to handles high PHY layer, MAC and RLC layer with network function virtualization (NFV) or Containers. 
3. **Open Central Unit (O-CU)**
    The centralized unit software that ==runs the Radio Resource Control (RRC) and Packet Data Convergence Protocol (PDCP) layers==.
4. **Ran Intelligent Controller (RIC)**
    A RAN Intelligent Controller (RIC) is a software-defined component of the Open Radio Access Network (Open RAN) architecture that’s responsible for controlling and optimizing RAN functions. RIC includes **Non-Real Time (Non-RT)** and **Near-Real Time (Near-RT) RIC**:
    - **Non-RT RIC** ==collects the RAN information and predicts network behavior through machine learning==. 
    - [**Near-RT RIC**](https://hackmd.io/@ianjoseph/H1AYsv07j) automatically ==adjusts the Radio Resource Management (RRM) to the RAN and prevents Quality of Experience (QoE)== from being reduced by the predicted results of Non-RT RIC
