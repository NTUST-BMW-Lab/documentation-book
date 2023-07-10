# BubbleRAN
###### tags: bubbleran
<!-- 
## Introduction
 -->
## <center>Installation Setup</center>
> Writter:
    - Ian Joseph Chandra (2023/04/01 - 2023/04/27)

1. Set a **static IP address** for the **server's MAC address** in the router 

     **Notes**
    
    The BubbleRAN installation process **requires internet** to ++download its image++. Suggested to set a **static IP address** for the server's MAC address on the router.
    **Example**:
    1. Router Setting (ex: 192.168.8.45):
        ![](https://i.imgur.com/PQoKDTf.png)
        
    2. Server's IP after installation:
        ![](https://i.imgur.com/XupdrpT.png)

2. Plug the BubbleRAN flashdisk to the server.
![](https://i.imgur.com/nQAsEsc.jpg)

1. Boot the server using the Ubuntu from the BubbleRAN flashdisk.
![](https://i.imgur.com/xOpF8mi.png)

1. After the Ubuntu is installed, the cloud-init will continue the BUbbleRAN.
   
    **Notes**

    Check the cloud-init.log status in a terminal from ```$HOME``` directory:
    ```bash
    tail -f /var/log/cloud-init.log
    ```
    **IMPORTANT!**
    
    <font color="red">**Do not do anything during the cloud-init installation process!**</font> 
    The installation is finished when the log terminal shows this output:
    ```log=
    Cloud-init v. 22.3.4-3-ga512eef2-1~bddeb running 'init-local' at Fri, 14 Apr 2023 08:45:32 +0000. Up 7.44 seconds.
    Cloud-init v. 22.3.4-3-ga512eef2-1~bddeb running 'init' at Fri, 14 Apr 2023 08:47:34 +0000. Up 128.62 seconds.
    ...
    ```
    ```log=2435
    time="2023-04-14T17:24:00+08:00" level=info msg="Install/upgrade performed on: mosaic5g-models"
    Cloud-init v. 22.3.4-3-ga512eef2-1~bddeb running 'modules:final' at Fri, 14 Apr 2023 08:51:17 +0000. Up 351.82 seconds.
    2023-04-14 09:24:27,562 - util.py[WARNING]: Failed to post phone home data to http://[::]:80/phone-home/f8bfdaed-8dc3-85f4-f11c-704d7bc23466/ in 10 tries
    Cloud-init v. 22.3.4-3-ga512eef2-1~bddeb finished at Fri, 14 Apr 2023 09:24:27 +0000. Datasource DataSourceGaiaNet [seed=ds_config_seedfrom,https://gaia.bubbleran.com/][dsmode=net].  Up 2342.09 seconds
    ```

2. Setup BubbleRAN Kubernetes credentials:
    1. Create a directory on ```~/snap/br-t9s/current/.kube/```:
        ```bash
        mkdir ~/snap/br-t9s/current/.kube/
        ```
    2. Create a config file inside ```~/snap/br-t9s/current/.kube/```:
        ```bash
        nano ./config
        ```
    3. Copy the configuration below to **config file**:
        ```json=
        apiVersion: v1
        clusters:
            -   cluster:
                    certificate-authority-data: DATA+OMITTED
                    server: https://192.168.8.45:6443
                name: kubernetes
        contexts:
            -   context:
                    cluster: kubernetes
                    user: kubernetes-admin
                name: kubernetes-admin@kubernetes
        current-context: kubernetes-admin@kubernetes
        kind: Config
        preferences: { }
        users:
            -   name: kubernetes-admin
                user:
                    client-certificate-data: REDACTED
                    client-key-data: REDACTED
    

        
    **IMPORTANT!**
        
        Setup the server IP address on line 5
        Ex: 192.168.8.45   

3. Setup the [Command Control (CTL)](https://bubbleran.com/docs/tutorials/getting-started/software#command-control-ctl-setup):
    1. Check the **snap** package installer status after update:
        ```bash
        sudo snap version
        ```
        ![](https://i.imgur.com/LIyU9X6.png)
    2. Install BubbleRAN (br-t9s) in Ubuntu using Snap:
        ```bash
        sudo snap install br-t9s
        ```
        ![](https://i.imgur.com/eBqqUnQ.png)
    3. Check BubbleRAN update using Snap:
        ```bash
        sudo snap refresh br-t9s
        ```
        ![](https://i.imgur.com/wQIIKXc.png)
    4. Setup alias for BubbleRAN:
        ```bash
        sudo snap alias br-t9s.cli cli
        ```
        ![](https://i.imgur.com/7cb0fxj.png)
    5. Verify BubbleRAN cli version:
        ```bash
        cli --version
        ```
        ![](https://i.imgur.com/z1FZylo.png)
        
        **Info**:
        the **cli** above is the alias of BubbleRAN cli (br-t9s.cli).
        
        
## <center>BubbleRAN [Example Lab](https://bubbleran.com/docs/tutorials/studio/example-lab/)</center>
After finishing the BubbleRAN installation, there are several testing module provided by Eurocome for further understanding about its functionality.

### I. Simple 5G [StandAlone (SA) Deployment](https://bubbleran.com/docs/tutorials/studio/example-lab/#simple-sa-deployment)
1. Create a sample.yaml file on ```$HOME``` directory:
    ```bash
    nano sample.yaml
    ```
    
2. Copy the content below to **sample.yaml** file:
    
    ```json
    apiVersion: core.trirematics.io/v1
    kind: Network
    metadata:
        name: oai-sim
        namespace: trirematics
        annotations:
            trirematics.io/license: Apache-2.0
            trirematics.io/provider: Trirematics
    spec:
        slices:
            -   plmn: "00101"
                dnn: "operator"
                network-mode: "IPv4"
                service-type: eMBB
                differentiator: "0x000000"
                ipv4-range: "12.1.1.0/24"
                ipv6-range: "2001:1:2::/64"
        access:
            -   name: oai-ran
                vendor: oai
                stack: 5g-sa
                model: oai-ran-model
                deployment-mode: monolithic-gnb
                annotations:
                    extras.trirematics.io/t-tracer: "true"
                identity:
                    an-id: 30
                    tracking-area: 1
                radio:
                    device: rf-sim
                cells:
                    -   band: n78
                        arfcn: 641280
                        bandwidth: 40MHz
                        subcarrier-spacing: 30kHz
                core-networks:
                    - oai-cn.oai-sim
        core:
            -   name: oai-cn
                vendor: oai
                stack: 5g-sa
                model: oai-cn-model
                deployment-mode: minimal
                annotations:
                    extras.trirematics.io/mtu: "1500"
                identity:
                    region: 128
                    cn-group: 4
                    cn-id: 5
        dns:
            ipv4:
                default: 8.8.8.8
                secondary: 8.8.4.4
    ---
    apiVersion: core.trirematics.io/v1
    kind: Terminal
    metadata:
        name: ue1
        namespace: trirematics
    spec:
        name: rf-sim
        vendor: oai
        stack: 5g-sa
        model: terminal-model
        deployment-mode: rf-simulator
        preferred-access: oai-ran.oai-sim
        annotations:
            extras.trirematics.io/sst: eMBB
            extras.trirematics.io/sd: "0x000000"
            extras.trirematics.io/nrb: "106"
            extras.trirematics.io/scs: "30kHz"
        target-cores:
            - oai-cn.oai-sim
        identity:
            imsi: "001010000000001"
            pin: "1234"
            opc: "0xc42449363bbad02b66d16bc975d77cc1"
            key: "0xfec86ba6eb707ed08905757b1bb44b8f"
            sqn: "0xff9bb4000001"
            dnn: "operator"
            network-mode: "IPv4"
        radio:
            4g-bands: [ ]
            5g-bands: [ 78 ]
            nsa-bands: [ ]
        readiness-check:
            method: ping
            target: google-ip
            interface-name: oaitun_ue1
    ---
    apiVersion: core.trirematics.io/v1
    kind: Terminal
    metadata:
        name: ue2
        namespace: trirematics
    spec:
        name: rf-sim
        vendor: oai
        stack: 5g-sa
        model: terminal-model
        deployment-mode: rf-simulator
        preferred-access: oai-ran.oai-sim
        annotations:
            extras.trirematics.io/sst: eMBB
            extras.trirematics.io/sd: "0x000000"
            extras.trirematics.io/nrb: "106"
            extras.trirematics.io/scs: "30kHz"
        target-cores:
            - oai-cn.oai-sim
        identity:
            imsi: "001010000000002"
            pin: "1234"
            opc: "0xc42449363bbad02b66d16bc975d77cc1"
            key: "0xfec86ba6eb707ed08905757b1bb44b8f"
            sqn: "0xff9bb4000001"
            dnn: "operator"
            network-mode: "IPv4"
        radio:
            4g-bands: [ ]
            5g-bands: [ 78 ]
            nsa-bands: [ ]
        readiness-check:
            method: ping
            target: google-ip
            interface-name: oaitun_ue1
    ```

3. Install the network using BubbleRAN cli:
    ```bash
    cli install network sample.yaml
    ```

4. Check the deployment status:
    ```bash
    cli observe
    ```
    
    The deployment is finished after the ```WORKLOAD READY``` state is ```true```.
    ![](https://i.imgur.com/RsO30g1.jpg)
    
<!--     :::info
    **Explanation**:
    In this activity, several 5G core network elements are installed:
    - Access and Mobility Management Function (AMF)
    - Session management Function (SMF)
    - User Plane Function (UPF)
    :::
    
    :::info
    #### Access And Mobility Management Function (AMF)
    > source: [iplook.com](https://www.iplook.com/products/5gc-amf)
    
    AMF **terminates** the control plane of different access networks onto the 5G Core Network (5GC) and control which UEs can access the 5GC to exchange traffic with DNs. It also **manages the handover of UEs** when they roam from one gNB to another for session continuity as described in Figure 1.
    ![](https://i.imgur.com/GDOdntw.png)
    Figure 1. AMF 5G Core Architecture
    :::

    :::info
    #### Session Management Function (SMF)
    SMF traces Protocol Data Unit (PDU) and Quality of Service (QoS) flows in the 5GC for UEs. ==SMF ensures the states and status are in sync between Control and User Planes network functions==. It also receives Policy and Charging Control (PCC) rules from Policy Charging Function (PCF) and converts PCC rules into SDF templates, QoS profiles, and rules for UPF, gNB, and UE, respectively, for QoS flows establishment, modification, release, etc.
    :::
 -->
5. [Install Wireshark](https://itslinuxfoss.com/how-to-install-configure-wireshark-on-ubuntu-20-04/)
    - Update the Ubuntu Advance Packaging Tool (APT).
        ```bash
        sudo apt update
        ```
        ![](https://hackmd.io/_uploads/H1UBKC_42.png)
    - Install the Wireshark from Ubuntu APT.
        ```bash
        sudo apt install wireshark
        ```
        ![](https://hackmd.io/_uploads/rk3y5AuEn.png)
        
        The wireshark is installed when we can get the version information.
        ```bash
        wireshark --version
        ```
        ![](https://hackmd.io/_uploads/BJT5qCO4h.png)
    - Setup the access for non-superuser.
        ```bash
        sudo dpkg-reconfigure wireshark-common
        ```
        ![](https://hackmd.io/_uploads/r1bVtJYE3.png)
        
        ```bash
        sudo usermod -a -G wireshark $USER
        ```
    - Modify the 'dumpcap' file permission.
        ```bash
        sudo chgrp wireshark /usr/bin/dumpcap
        sudo chmod 750 /usr/bin/dumpcap
        sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/dumpcap
        ```
        ![](https://hackmd.io/_uploads/B12ywJFEn.png)
        ```bash
        sudo getcap /usr/bin/dumpcap
        ```
        ![](https://hackmd.io/_uploads/SkiuYyFNh.png)

6. Setup Wireshark Filter
    ```bash
    udp_ports="2123 or 2152 or 9999"
    sctp_ports="38412 or 38472 or 36421"
    filter="(sctp port $sctp_ports) or (udp port $udp_ports)"
    ```
    ![](https://hackmd.io/_uploads/BJuV3JY4n.png)

7. Start the Wireshark to listen the PCAP file from OAI-GNB element.
    ```bash
    cli extract pcap {element} -- "$filter" | wireshark -k -i -
    ```
    **Info:**
    Change the **{element}** with the oai-gnb... (press **TAB** button to get the exact name of the element & series).
    
    ![](https://hackmd.io/_uploads/S1qO2yK43.png)

    ![](https://hackmd.io/_uploads/SyhI21YV2.png)

8. Start the OAI-GnB **on another terminal**.
    
    To start the OAI-GnB, use ```cli cic {element} run -- t-dumper```
    ![](https://hackmd.io/_uploads/HyNF4llYn.png)
    To verify the E2E connection, check the AMF and gNB logs using ```extract logs {element}```

9. Generate UE traffic
    The next step is to generate traffic between the UE and the gNB and observe the results. Pick one of the UEs at random for this part of the lab. There are two classes of tests that can be performed:

    - ```rtt``` : Round Trip Time (RTT) measurements using ping.
    - ```throughput``` : Throughput measurements using iperf3.

    In all the following commands of this section, ***{terminal} is a placeholder for the name of the network terminal (a generic term for the UEs)***. To run the RTT test, use the following command:
    ``` cli test rtt {terminal} -- -c 100 -s 64```
    
    ![](https://hackmd.io/_uploads/Sk3WfvGY3.png)
    
    ***Issue***
    ![](https://hackmd.io/_uploads/S1lWkMFFn.png)

    ```bash 
    if you are facing this issues, the problems might be:
    - Wrong YAML file configuration
    - Kubernetes failed to configure the pods
    
    The solutions for this problems might be:
    - check if you have the Terminal object installed via cli observe or equivalently kubectl get term -n trirematics. 
    - Check the list of the pods via kubectl get pods -n trirematics to check whether the UE pod actually exists. 
    - For some reason, the CLI is unable to get the right name and produces "<no-value>". Also while working with the CLI. 
    - having the --verbose flag might help sometimes to debug. 
    ```
    
    

10. Extract the gNB configuration
    You can extract the configuration of the gNB as well as some visuals on your deployment. To extract the configuration, use the following command:
    ```bash
    cli extract config {element} /tmp
    ```
    ![](https://hackmd.io/_uploads/SkFjDPztn.png)

    To extract the visuals, use the following command:
    ```bash
    cli extract graph
    ```
    ![](https://hackmd.io/_uploads/ryc2vDGth.png)

11. Uninstall the network
    To uninstall the network after simulation, use the following command:
    ```bash
    cli remove network sample.yaml
    ```
    ![](https://hackmd.io/_uploads/Syx0vPMFh.png)








