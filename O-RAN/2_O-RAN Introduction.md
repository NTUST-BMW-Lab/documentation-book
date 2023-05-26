###### tags: `Guide book`
# <center>What is RAN?</center>
![](https://i.imgur.com/5zXRm5t.png)

A [radio access network (RAN)](https://www.redhat.com/en/topics/5g-networks/what-is-radio-access-network) is the part of a mobile network that connects end-user devices, like smartphones, to the cloud. This is achieved by sending information via radio waves from end-user devices to a RAN’s transceivers, and finally from the transceivers to the core network which connects to the global internet.

For telecommunications network operators, RANs are crucial connection points that represent significant overall network expenses, perform intensive and complex processing, and now face rapidly increasing demand as more edge and 5G use cases emerge for telco customers.

# RAN History
<iframe width="560" height="315" src="https://www.youtube.com/embed/-fVHO_WCGF8?start=58" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Traditional RAN
![](https://i.imgur.com/jNtWjtg.png)

In the past, Telco provider needs to setup all the equipment including the base station server near the antennas. This condition makes the installation of a base station is expensive and not efficient (since every server require extra space & cooling system for the installation).

---
## Virtual/Cloud RAN (V-RAN/C-RAN)
![](https://i.imgur.com/E2UkDWI.png)

Cloud RAN (C-RAN) or Virtual RAN (vRAN) then being invented to simplify the equipments that required for a base station installation. The purpose of C-RAN is to **centering the logic function of networking operation into a cloud**. This cloud service for RAN is also called **Core Network**. The innovation of fiber optic also give support to move the logic server into a Core network server. This approach is more efficient, since we can reduce the cost for the space and cooling system for base station.

:::info
**Notes**: C-RAN is not O-RAN (Every C-RAN is maintained by a single operator)
:::

## Open RAN (O-RAN)
![](https://i.imgur.com/NQexCa9.png)

Since there are so many Telco providers & hardware vendors, it makes the integration between devices & framework of C-RAN more complicated. Then some communities & companies initiate to make an open-source of C-RAN systems that could be use for every provider and vendors.

[Open Radio Access Network (O-RAN)](https://www.o-ran.org/) is an open-source software solution to enable an open and intelligent 5G radio access network (RAN). It uses an open user interface to connect different telecommunications providers' hardware, software, and even subsystems to achieve flexible network integration and deployment with high scalability. The purpose of O-RAN is to **encourage manufacturers to invest in base station-related research and development, enabling the sharing of technical knowledge and resources**. The other benefit of O-RAN is that it reduces the dependence on traditional RAN equipment from a specific vendor (such as Nokia and Ericsson), which impacts cost savings in building 5G wireless communication networks. The detail of O-RAN introduction is explained in Video by [FujitsuFNC](https://www.youtube.com/c/FujitsuFNC).

<iframe width="560" height="315" src="https://www.youtube.com/embed/NNO-LvVriSY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The goals of O-RAN:
- Interface openness
- Network Intelligence
- Hardware white-box
- Software open-source

# <center>How to Realize O-RAN?</center>
There are several companies that has opened their modules to be implemented in the O-RAN, eventhough only several modules are compatible with the specs from the OSC

# <center>What is OSC?</center>
The [O-RAN Software Community (OSC)](https://o-ran-sc.org/#charter) is a collaboration between the O-RAN Alliance and Linux Foundation with the mission to support software creation for the RAN. The OSC plans to leverage other LF network projects while addressing performance, scale, and 3GPP alignment challenges.

The OSC focuses on aligning with the O-RAN Alliance's open architecture and specifications to achieve a solution that can be utilized for industry deployment. As a new open-source community under the Linux Foundation, the OSC is sponsored by the O-RAN Alliance. It will enable the development of open-source software enabling modular, open, intelligent, efficient, and agile disaggregated RAN.

:::info
**Note**:
In this project, we follows the specs from the OSC & OAI for specific modules.
:::
