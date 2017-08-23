---
published: true
date: '2017-08-23 12:29 -0700'
title: Multilayer Planning with the WAN Automation Engine
author: Josh Peters
excerpt: >-
  The WAE network model includes the multivendor devices that participate in the
  IGP (OSPF or IS-IS) and can be extended to include the DWDM topology. Using
  the WAE Design application or WAE APIs, you can use the model to simulate
  “what if” scenarios and optimize paths for L3 and L1 domains. The purpose of
  this tutorial is to demonstrate the multilayer capabilities of WAE.
tags:
  - cisco
  - WAE
  - Design
  - Capacity Planning
  - Multilayer
---
{% include toc %}

# Overview 

The WAE network model includes the multivendor devices that participate in the IGP (OSPF or IS-IS) and can be extended to include the DWDM topology. Using the WAE Design application or WAE APIs, you can use the model to simulate “what if” scenarios and optimize paths for L3 and L1 domains. The purpose of this tutorial is to demonstrate the multilayer capabilities of WAE. 

>
Note: 
>
* The network model used in this tutorial is from the WAE Design samples directory: **us_wan_L1.txt**
* If you don’t have the WAE Design application, see the tutorial [Using dCloud to Access the WAN Automation Engine Demos](https://xrdocs.github.io/automation/tutorials/2017-08-03-using-dcloud-to-access-the-wan-automation-engine-demos). 
* If using dCloud, use the **US East** datacenter and launch the demo for "WAE Live". The WAE Design application will be on the workstation desktop.
{: .notice--info}

# Discovery of the DWDM Topology

The goal in multilayer discovery is to augment the WAE model of the IGP with the DWDM topology. For Cisco optical, the layer 1 topology discovery and mapping between the L1 circuit path and the L3 adjacency is automatic; WAE can interface through CTC to the NCS 2K running 10.6.2 or later to discover the DWDM topology and L3-L1 mapping if LMP is configured on both the NCS 2K and IOS-XR. In IOS-XR, an example of the LMP configuration is shown below:

    RP/0/RP0/CPU0:Napoli-5#show run lmp
    Mon Aug 21 19:31:21.657 CET
    lmp
    gmpls optical-uni
      ...
      controller dwdm0/6/0/20/0
       neighbor Battipaglia
       neighbor link-id ipv4 unicast 20.20.20.60
       neighbor interface-id unnumbered 2130707466
       link-id ipv4 unicast 20.20.20.50
      !
      ...
      neighbor Battipaglia
       ipcc routed
       router-id ipv4 unicast 10.58.244.11
      !
      router-id ipv4 unicast 10.58.244.50
    !
    !

    RP/0/RP0/CPU0:Napoli-5#show run int hundredGigE 0/6/0/20/0
    Mon Aug 21 19:31:38.179 CET
    interface HundredGigE0/6/0/20/0
      description Core Link to Salerno - bundle member
      bundle id 2 mode active
      cdp
      transceiver permit pid all
    !
 
    RP/0/RP0/CPU0:Napoli-5#show run controller dwdm 0/6/0/20/0
    Mon Aug 21 19:31:59.844 CET
    controller dwdm0/6/0/20/0
      g709 enable
      g709 fec high-gain-sd-fec
      wavelength 50GHz-Grid frequency 19315
      admin-state in-service
    !


For third party optical vendors, typically WAE will use the vendor specific NMS for the topology details and populate a vendor agnostic YANG model. Third-party interfaces for L1 vendors require a WAE function pack. Please contact your Cisco sales representative for details.

# Multilayer Planning in WAE Design

WAE Design has visualization, simulation and optimization tools and capabilities for multilayer planning scenarios. This section will cover a few of these scenarios by example. You can follow the examples using the us_wan_L1.txt plan file located in the WAE Design samples directory.

## Example: Visualize the Multilayer Topology 

1. Open us_wan_L1.txt and make sure you have the network plot set to the Simulated Traffic view.
2. Currently you are looking at the layer 3 topology. View the layer 1 topology by selecting the L3/L1 toggle button in the upper left corner of the Design application.
3. From the layouts menu, select the layout “Joint” to see a layout that was setup to see the L3 and L1 views in the same plot.
4. See the settings for this layout by right clicking an empty area of the plot and selecting Plot Options. Then toggle to the Layer 1 tab to see available options.   
5. In this plot, you can also open any site to see the contained L1 and L3 nodes.

![WAE multilayer view](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/wae-ml-visual.gif){: .align-center} 

Another useful tip is the ability to select background objects in the network plot. To enable this setting, go to View -> Preferences and check the box that says “Enable Selection of Background Objects”.
![WAE visualization selection](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/wae-ml-pref.png){: .align-center} 

## Example: Simulate the Impact of a Fiber Cut
The primary purpose of WAE including the layer 1 information is to understand the impact to the layer 3 topology.

1. Using us_wan_L1.txt ensure you have the network plot set to the **Simulated Traffic** view.
2. Highlight a layer 3 circuit, toggle to the L1 view and observe the associated layer 1 circuit 
3. Click an empty area of the plot remove the highlights for the selections
4. Right-click a layer 1 link and select **Fail**
5. Observe the impact of the failure in the L1 and L3 views
6. Right-click and empty area of the plot and select **Recover** and select the L1 link.

![WAE multilayer simulation](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/wae-ml-sim.gif){: .align-center} 

## Example: Modeling DWDM Circuit Paths and Protection Schemes
In the previous example, failing a L1 link simulated the impact to the layer 3 topology. In WAE, layer 3 circuits are mapped to layer 1 circuits, and layer 1 circuits can have circuit paths. In this example, you will use circuit paths to simulate optical restoration and 1+1 protection scenarios.

In WAE, L1 links have properties for distance, delay, loss, metric, feasibility metric, reserved L1 circuit paths, maximum L1 circuit paths and waypoint information. This information helps determine the path of the L1 circuit (which can have a feasibility limit). The colors on the L1 links represent the utilization, which is the number of active and operational circuit paths routed through the L1 link divided by the maximum allowable.
{: .notice--info}

### Simulate Optical Restoration
1. Using us_wan_L1.txt ensure you have the network plot set to the **Simulated Traffic** view.
2. In the L1 view, right-click on the L1 link between **SLC** and **KCY** and select **Filter to L1 Circuits Through L1 Links.**
3. In the L1 Circuits table, identify and select the L1 circuit between **CHI** and **SEA**
4. Right-click an empty area of the plot and select **New -> Layer 1 -> L1 Circuit Paths**
5. For the 1 selected L1 circuit, set the path option to **2** Uncheck the **Standby** box.
6. In the L1 Circuits table, identify and select the L1 circuit between **CHI** and **SEA** then right-click on the L1 link between **SLC** and **KCY** and select **Fail**
7. Observe the dashed line for this circuit. This failure simulation shows path option 1 becoming non-operational and path option 2 becoming operational. 
8. Right-click and empty area of the plot and select **Recover** and select the L1 link.
![WAE Optical Restoration Simulation](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/wae-ml-opt-rest.gif){: .align-center} 

### Simulate 1+1 Protection
1. Using us_wan_L1.txt ensure you have the network plot set to the **Simulated Traffic** view.
2. In the L1 view, right-click on the L1 link between **SLC** and **KCY** and select **Filter to L1 Circuits Through L1 Links.**
3. In the L1 Circuits table, identify and select the L1 circuit between **CHI** and **SEA**
4. Right-click the circuit and select **Filter to L1 Circuit Paths**.
5. Open the Properties menu for the L1 circuit path with path option 2. (This was added in the previous example)
6. Set the checkbox for **Standby** and ensure the checkbox for **Active** is also checked.
7. To modify the path, select **Insert After** and set site **SLC** to **Exclude**. 
8. Select OK to exit the properties menu
9. In the L1 Circuits table, identify and select the L1 circuit between **CHI** and **SEA** then right-click on the L1 link between **SLC** and **KCY** and select **Fail**
10. Observe the circuit path change for this circuit. This failure simulation shows path option 1 becoming non-operational and path option 2, which was already established becoming operational. 
11. Right-click and empty area of the plot and select **Recover** and select the L1 link.
![WAE 1+1 Protection simulation](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/wae-ml-opt-protect.gif){: .align-center} 

In this example, you changed the L1 circuit path manually to be partially disjoint from the other. In WAE Design version 7, there is an optimization tool to compute disjoint L1 circuit paths.
{: .notice--info}

## Example: Multilayer Optimizations for Segment Routing Policies 
If the layer 3 and layer 1 topologies are included in the model, you can take advantage of the WAE optimization tools for Segment Routing Policies.

### Optimize a SR Policy for Latency and Avoidance
This example will use the WAE SR-TE optimization tool. 

1. Right-click an empty area of the plot and select **New -> LSPs -> LSP**
2. In the LSP Properties menu, set the type to SR, set the source to er1.sea and set the destination to be er1.atl. Click OK.
3. Observe the path of the SR policy.
4. Next optimize the SR policy by selecting **Tools -> SR-TE Optimization**
5. In the SR-TE Optimization menu, set Minimize Path Metric to Delay and ensure Avoid Nodes is set to None. Click OK.
6. Observe the optimized path of the SR policy in the new plan file.
7. If you want the lowest latency SR policy that avoids nodes in site KCY, first highlight the site KCY, right-click and select Filter to Contained Nodes. 
8. In the Nodes table, highlight the selected nodes.
9. Select **Tools -> SR-TE Optimization**. In the SR-TE Optimization menu, set Minimize Path Metric to Delay and set Avoid Nodes to "Selected in Table". Click OK.
10. Observe the optimized path of the SR policy in the new plan file.
![WAE Avoidance and Delay Optimization](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/sr-opt-ml.gif){: .align-center}

In this example, the avoidance criteria is referring to L3 nodes. In WAE Design version 7, you can also select avoidance criteria for L1 Nodes and L1 Links.
{: .notice--info}

### Optimize SR Policies for Multilayer Disjointness
In this example, you will use WAE to compute paths for SR policies that are disjoint for the L3 and L1 topologies.

1. Close any open plan files and do not save changes. Open the plan file **us_wan_L1.txt**. It should be in the original state in which you started this tutorial.
2. Right-click an empty area of the plot and select **New -> LSPs -> LSP**
3. In the LSP Properties menu, set the following parameters:
    - Type: SR 
    - Source: er1.sjc
    - Destination: er1.kcy. 
    - In the Advanced tab set Disjoint Group to any string such as 'group1'. 
    - Click OK.
4. Right-click on the LSP in the LSPs table and select **Duplicate**
5. Select the LSPs in the LSPs table, right-click an empty area of the plot and select **New -> LSPs -> LSP Paths**. Leave the defaults and click OK. 
6. Select **Tools -> LSP Disjoint Path Optimization**. 
7. In the LSP Disjoint Path Optimization menu, set the following options: 
    - Disjoint Routing Selection: Create Disjoint Paths between LSPs in Disjoint Groups. 
    - Priorities: set L1 Links to 1 
    - Click OK. 
8. Observe in the new plan file that the SR policies are disjoint at L3 and L1.
![WAE Disjoint Path Optimization](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/sr-disjoint-ml.gif){: .align-center}


## Example: Capacity Planning Optimization
WAE has a tool called Capacity Planning Optimization that attempts to get below a simulated utilization threshold by upgrading the capacity in some way. This section will show some of the options of the tool and the results.

Before you start this section, do the following tasks.
1. Close any open plan files and do not save changes. Open the plan file **us_wan_L1.txt**. It should be in the original state in which you started this tutorial.
2. Right-click an empty area of the plot and select **New -> Demands -> Demand**
3. In the Demand Properties menu set the following options:
    - Name: Can be any string such as 'new_customer'
    - Source: er1.sea
    - Destination: er1.atl
    - Traffic: 700 (Mbps)

### Capacity Planning Optimization Create Parallel Circuits
![WAE Capacity Optimization - Parallel Circuits](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeML-capOpt-parallel.png){: .align-right}
1. Select **Tools -> Capacity Planning Optimization**
2. In the Capacity Planning Optimization menu, select the following options:
    - Maximum Interface Utilization: 80
    - Capacity Increment: 1000
    - Upgrade Existing Circuits: Create Parallel Circuits
    - Create New Adjacencies: Restrict New Adjacencies between Nodes **None**
    - Select OK
    
The result should look similar to the picture on the right.

### Capacity Planning Optimization New Adjacencies (with Optical Bypass)
![WAE Capacity Optimization - Parallel Circuits](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeML-capOpt-bypass.png){: .align-right}
Ensure the plan file us_wan_L1.txt is selected.
This example will examine a case where you can only add new adjacencies between core nodes. First, select only the core nodes in the Nodes table.
1. Select **Tools -> Capacity Planning Optimization**
2. In the Capacity Planning Optimization menu, select the following options:
    - Maximum Interface Utilization: 80
    - Capacity Increment: 1000
    - Upgrade Existing Circuits: (Ignore)
    - Create New Adjacencies: Restrict New Adjacencies between Nodes **Selected in Table (22/33)**
    - In the Layer 1 tab, select the checkbox Create L1 Circuits
    - Select OK
    
The result should look similar to the picture on the right.

### Capacity Planning Optimization LAG Augmentation
![WAE Capacity Optimization - LAG Augmentation](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeML-capOpt-lag.png){: .align-right}
Ensure the plan file us_wan_L1.txt is selected
This time you will see what happens if you can only augment existing LAG members.
1. Select **Tools -> Capacity Planning Optimization**
2. In the Capacity Planning Optimization menu, select the following options:
    - Maximum Interface Utilization: 80
    - Capacity Increment: 1000
    - Check the box Use Capacity of Existing LAG Members
    - Upgrade Existing Circuits: Create Port Circuits (LAGs)
    - Create New Adjacencies: Restrict New Adjacencies between Nodes **None**
    - In the Layer 1 tab, select the checkbox Create L1 Circuits
    - Select OK
    
The result should look similar to the picture on the right. Notice the upgraded links are larger. This is because the capacity of the L3 adjacency is larger as new members have been added to the LAG.

In this example, if you examine the Ports table you will see the ports have different capacities which doesn't make a lot of sense. In a real network WAE can discover the ports that are not being used. Selecting the checkbox Use Capacity of Existing LAG Members will use the ports that are not associated with port circuits.
{: .notice--info}

#### LAG Augmentation and L1 Circuit Activation
In WAE 7 EFT, the ability to use the WAE Design application to deploy changes to the network for the LAG augmentation use case has been demonstrated. The testbed involved the NCS 2000 running 10.6.2 or later and the ASR 9000 running IOS-XR. The L3 LAG augmentation was performed using the Cisco Network Services Orchestrator and the L1 circuit activation uses CTC.  

At this time, we are not planning to use WAE Design to deploy new L3 circuits to the network. WAE does expose APIs which can be leveraged to compute optimized paths and deployed to the network using other management and orchestration tools.
