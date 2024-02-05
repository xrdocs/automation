---
published: true
date: '2023-12-12 15:35 +0100'
title: Deploying multi vendor L2VPN services using Crosswork Network Controller
author: Bradley Riapolov
position: top
tags:
  - Crosswork
  - Automation
  - Multi Vendor
excerpt: >-
  This article will cover how Cisco Crosswork Network Controller can be used to
  deploy L2VPN services across a multi vendor network.
---
{% include toc icon="table" title="Deploying multi vendor L2VPN services using Crosswork Network Controller" %}

# Authors
Bradley Riapolov and Tahsin Chowdhury
# Introduction
Service Provider networks inherently span vast and diverse landscapes. These networks often involve intricate blends of hardware and software solutions, often sourced from various vendors. While these ecosystems offer advantages, the task of aligning one vendor's tools to effectively manage changes on another vendor's hardware continues to present a formidable challenge.


Cisco's Crosswork Controller serves as one of these network orchestration tools. It's essential to understand the range of supported use cases to fully appreciate its [capabilities and limitations](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-network-controller/5-0/Solution-Workflow-Guide/bk-crosswork-network-controller-5-0-solution-workflow-guide/m-solution-overview.html#Cisco_Generic_Topic.dita_e1b66704-bf56-4676-8c0b-f956b30bc05f).

In the realm of multi-vendor support, exercising caution is essential—an imperative that applies to any tool claiming such capabilities. Anticipating a tool to accommodate the myriad variations and deployment configurations, all subjected to rigorous testing through release processes, is not a practical expectation. Nonetheless, this shouldn't deter exploration; rather, it's an invitation to embark on uncharted paths and discover if orchestrating and visualizing a multi-vendor setup is attainable.

# A Multi-Vendor Objective

In this particular scenario, the task was to establish and visualize a L2VPN point-to-point configuration between Cisco's IOS XR and Juniper's Junos using the Crosswork Network Controller GUI. Our objective is to lead you through the steps we took in this process, pushing the boundaries of what is presently recognized as officially supported (we hope to inspire others to explore the realm of possibilities with just a little added effort). For our setup, we used a network with Cisco IOS XRv9000 virtual routers and Juniper vMX virtual routers along with CNC, CDG, SR-PCE, NSO (with CNC core function packs).

# A Step-by-Step Solution

We built a working configuration between the two devices:

Cisco IOS XRv9000:
```
interface GigabitEthernet0/0/0/1.101 l2transport
 description T-SDN Interface
 encapsulation dot1q 512
!
l2vpn
 ignore-mtu-mismatch
 pw-class TEST
  encapsulation mpls
   control-word
   transport-mode vlan
  !
 !
 xconnect group TEST
  p2p TEST
   interface GigabitEthernet0/0/0/1.101
   neighbor ipv4 198.19.1.99 pw-id 101
    pw-class TEST
   !
  !
 !
!
```
Juniper vMX:
```
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 101 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 101 vlan-id 512
set interfaces ge-0/0/1 unit 101 family ccc
set protocols l2circuit neighbor 198.19.1.4 interface ge-0/0/1.101 virtual-circuit-id 101
set protocols l2circuit neighbor 198.19.1.4 interface ge-0/0/1.101 ignore-encapsulation-mismatch
set protocols l2circuit neighbor 198.19.1.4 interface ge-0/0/1.101 ignore-mtu-mismatch
set protocols ldp interface ge-0/0/0.0
set protocols ldp interface lo0.0
set protocols mpls label-switched-path lsp_to_pe2 to 198.19.1.4
set protocols mpls interface ge-0/0/0
```
**Note:** We configured our virtual environment to ignore the MTU setting so the l2circuit (Juniper)/ xconnect(Cisco) could come up (notice the L2circuit is **UP** on both ends). This is specific to this virtual environment. It's strongly advised to use similar MTU in production networks.
{: .notice}


Juniper ouput:
<div class="highlighter-rouge">
<pre class="highlight">
<code>
lab@PE1> show l2circuit connections 
Layer-2 Circuit Connections:

Legend for connection status (St)   
EI -- encapsulation invalid      NP -- interface h/w not present   
MM -- mtu mismatch               Dn -- down                       
EM -- encapsulation mismatch     VC-Dn -- Virtual circuit Down    
CM -- control-word mismatch      Up -- operational                
VM -- vlan id mismatch		 CF -- Call admission control failure
OL -- no outgoing label          IB -- TDM incompatible bitrate 
NC -- intf encaps not CCC/TCC    TM -- TDM misconfiguration 
BK -- Backup Connection          ST -- Standby Connection
CB -- rcvd cell-bundle size bad  SP -- Static Pseudowire
LD -- local site signaled down   RS -- remote site standby
RD -- remote site signaled down  HS -- Hot-standby Connection
XX -- unknown

Legend for interface status  
Up -- operational            
Dn -- down                   
Neighbor: 198.19.1.99 
    Interface                 Type  St     Time last up          # Up trans
    ge-0/0/0.101(vc 101)      rmt   <mark>Up</mark>     Jul 13 11:41:51 2023           1
      Remote PE: 198.19.1.99, Negotiated control-word: Yes (Null)
      Incoming label: 299840, Outgoing label: 299840
      Negotiated PW status TLV: No
      Local interface: ge-0/0/0.101, Status: <b>Up</b>, Encapsulation: VLAN
      Flow Label Transmit: No, Flow Label Receive: No
</code>
</pre>
</div>
Cisco output:
<div class="highlighter-rouge">
<pre class="highlight">
<code>
Node-4# show l2vpn xconnect

Legend: ST = State, UP = Up, DN = Down, AD = Admin Down, UR = Unresolved,
        SB = Standby, SR = Standby Ready, (PP) = Partially Programmed,
        LU = Local Up, RU = Remote Up, CO = Connected, (SI) = Seamless Inactive

XConnect                   Segment 1                       Segment 2                
Group      Name       ST   Description            ST       Description            ST    
------------------------   -----------------------------   -----------------------------
TEST       TEST       <mark>UP</mark>   Gi0/0/0/1.512          <mark>UP</mark>       198.19.1.99     101    <mark>UP</mark>    
</code>
</pre>
</div>

With the multi-vendor configuration working, our next goal was to provide a functional setup (along with the visualization) through Cisco Network Controller only. This involves using NSO (Network Services Orchestrator) to deliver these network changes. Let us adapt the NSO files accordingly. We “git-cloned” the repository (tsdn-juniper) that Cisco Automation engineers had posted on [github](https://github.com/maddn/tsdn-juniper.git) into our NSO VM.  

We extended the l2vpn service package in that repository to make our example  configuration work. We copied the "flat-l2vpn-juniper" folder into the package folder in our NSO environment. We followed the **Installation** section of that repository to install the "flat-l2vpn-juniper" package.  

Using the Juniper config syntax stated in the above example, we used NSO CLI to configure the commands and use **commit dry-run outformat xml** to get the configuration template and replace the required parameters with input variable (seen in the XML section below, but did not apply the commit).

```
cisco@nso-cnc5-brown-nss-j:~$ ncs_cli -u admin -C
admin connected from 10.16.27.1 using ssh on nso-cnc5-brown-nss-j
admin@ncs#
admin@ncs# config
Entering configuration mode terminal
admin@ncs(config)# devices device vmx99 config configuration interfaces interface ge-0/0/1 vlan-tagging
admin@ncs(config-interface-ge-0/0/1)# encapsulation vlan-ccc
admin@ncs(config-interface-ge-0/0/1)# unit 101 encapsulation vlan-ccc
admin@ncs(config-unit-101)# vlan-id 512
admin@ncs(config-unit-101)# family ccc
admin@ncs(config-unit-101)# top
admin@ncs(config)# devices device vmx99 config configuration protocols l2circuit neighbor 198.19.1.4 interface ge-0/0/1.101 virtual-circuit-id 101 
admin@ncs(config-interface-ge-0/0/1.101)# ignore-encapsulation-mismatch 
admin@ncs(config-interface-ge-0/0/1.101)# ignore-mtu-mismatch 
admin@ncs(config-interface-ge-0/0/1.101)# top
admin@ncs(config)# devices device vmx99 config configuration protocols mpls label-switched-path lsp_to_pe2 to 198.19.1.4                          
admin@ncs(config-label-switched-path-lsp_to_pe2)# 
admin@ncs(config-label-switched-path-lsp_to_pe2)# 
admin@ncs(config-label-switched-path-lsp_to_pe2)# 
admin@ncs(config-label-switched-path-lsp_to_pe2)# top
admin@ncs(config)# 
admin@ncs(config)# commit dry-run outformat xml
result-xml {
    local-node {
        data <devices xmlns="http://tail-f.com/ns/ncs">
               <device>
                 <name>vmx99</name>
                 <config>
                   <configuration xmlns="http://xml.juniper.net/xnm/1.1/xnm">
                     <interfaces>
                       <interface>
                         <name>ge-0/0/1</name>
                         <unit>
                           <name>101</name>
                           <encapsulation>vlan-ccc</encapsulation>
                           <vlan-id>512</vlan-id>
                           <family>
                             <ccc/>
                           </family>
                         </unit>
                       </interface>
                     </interfaces>
                     <protocols>
                       <mpls>
                         <label-switched-path>
                           <name>lsp_to_pe2</name>
                           <to>198.19.1.4</to>
                         </label-switched-path>
                       </mpls>
                       <l2circuit>
                         <neighbor>
                           <name>198.19.1.4</name>
                           <interface>
                             <name>ge-0/0/1.101</name>
                             <virtual-circuit-id>101</virtual-circuit-id>
                             <ignore-encapsulation-mismatch/>
                             <ignore-mtu-mismatch/>
                           </interface>
                         </neighbor>
                       </l2circuit>
                     </protocols>
                   </configuration>
                 </config>
               </device>
             </devices>
    }
}
admin@ncs(config)#
admin@ncs(config)# end
Uncommitted changes found, commit them? [yes/no/CANCEL] no
admin@ncs#
```

We saved the config template inside **~/ncs-run/packages/flat-l2vpn-juniper/templates** folder as **cisco-flat-L2vpn-fp-junos-p2p-l2circuit-template.xml**.
This file has been added with this repository.

```
cisco@nso-cnc5-brown-nss-j:~$ cd ncs-run/packages/flat-l2vpn-juniper/
cisco@nso-cnc5-brown-nss-j:~/ncs-run/packages/flat-l2vpn-juniper$ 
cisco@nso-cnc5-brown-nss-j:~/ncs-run/packages/flat-l2vpn-juniper$ 
cisco@nso-cnc5-brown-nss-j:~/ncs-run/packages/flat-l2vpn-juniper$ cd templates/
cisco@nso-cnc5-brown-nss-j:~/ncs-run/packages/flat-l2vpn-juniper/templates$ 
cisco@nso-cnc5-brown-nss-j:~/ncs-run/packages/flat-l2vpn-juniper/templates$ 
cisco@nso-cnc5-brown-nss-j:~/ncs-run/packages/flat-l2vpn-juniper/templates$ ls
cisco-flat-L2vpn-fp-junos-p2p-l2circuit-template.xml  cisco-flat-L2vpn-fp-junos-template.xml
```

We went to the folder **~/ncs-run/packages/flat-l2vpn-juniper/python/flat_l2vpn_juniper** and edited the **Junos.py** to add the p2p section inside the **def conf_l2vpn(self, site, local):** function.

```
class Junos:
    def conf_l2vpn(self, site, local):
        self.log.info(f"Configuring Flat L2VPN/SR on Junos for service {site.pe}")

        if self.service.service_type == "evpn-vpws":
            l2vpn_vars = ncs.template.Variables()
            l2vpn_vars.add("LOCAL_NODE", "true" if local is True else "false")
            l2vpn_template = ncs.template.Template(site)
            l2vpn_template.apply("cisco-flat-L2vpn-fp-junos-template", l2vpn_vars)
        
        """ Add this section""" 
        elif self.service.service_type == "p2p":
            l2vpn_vars = ncs.template.Variables()
            l2vpn_vars.add("LOCAL_NODE", "true" if local is True else "false")
            l2vpn_template = ncs.template.Template(site)
            l2vpn_template.apply("cisco-flat-L2vpn-fp-junos-p2p-l2circuit-template", l2vpn_vars)
       """ End of section """
```
This file has been added in our repository.

To make this configuration work, we had to modify the template for NSO Core Function Package for L2vpn (**cisco-flat-L2vpn-fp-internal**).

-  Went to the folder **~/ncs-run/packages/cisco-flat-L2vpn-fp-internal/templates**.
-  Edited both **cisco-flat-L2vpn-fp-cli-evpn-vpws-template.xml** and **cisco-flat-L2vpn-fp-cli-p2p-template.xml** with the content below:
     
```
<l2vpn xmlns="http://tail-f.com/ned/cisco-ios-xr">
      <ignore-mtu-mismatch/>
      <pw-class>
        ......< skipping some sections>.....
        <encapsulation>
          <mpls>
             ......< skipping some sections>.....
            <transport-mode>
               <vlan/>
            </transport-mode>
           </mpls>
        </encapsulation>
      </pw-class>
      ......< skipping some sections>.....
    </l2vpn>
```

Reloaded the packages in NSO (**packages reload**).

```
cisco@nso-cnc5-brown-nss-j:~/ncs-run$ ncs_cli -u admin -C
admin connected from 10.16.27.1 using ssh on nso-cnc5-brown-nss-j
admin@ncs# 
admin@ncs# packages reload

…..............< skipping some output >..................

reload-result {
    package cisco-flat-L2vpn-fp-internal
    result true
}

…..............< skipping some output >..................

reload-result {
    package cisco-iosxr-cli-7.46
    result true
}
…..............< skipping some output >..................

reload-result {
    package flat-l2vpn-juniper
    result true
}
reload-result {
    package flat-l2vpn-multi-vendors
    result true
}
…..............< skipping some output >..................
```

Now we were ready to deploy and visualize this in Cisco Crosswork Network Controller.  (We documented these steps in the [Github Repo](https://github.com/tchowdhu/CNC-Conf-Cisco-Junos-L2VPN-P2P/blob/main/README.md))

- Select L2VPN under Services Section.
         
<img width="468" alt="image" src="https://github.com/xrdocs/automation/assets/13122698/903e8860-ceb2-4363-91c2-3a8a85b48522">

- Either use GUI hitting "+" sign or import a configuration file in JSON format.

<img width="468" alt="image" src="https://github.com/xrdocs/automation/assets/13122698/1b6a7025-0a8e-4266-81b5-d664741cac2c">

- We will import a JSON configuration file for the service. The JSON file has been added in this repository.

<img width="468" alt="image" src="https://github.com/xrdocs/automation/assets/13122698/e40b5c53-bc96-4084-b4f3-eb41ee99e320">

- Check the imported parameters and hit "commit changes" (optional, also can do a dry-run before commit).

<img width="468" alt="image" src="https://github.com/xrdocs/automation/assets/13122698/1e9b1308-24a9-45eb-9645-ed306017de60">

- And finaly - visualize the service in CNC (remember, we never touched the command line on the routers, everything was done via Crosswork Network Controller):
  
<img width="468" alt="image" src="https://github.com/xrdocs/automation/assets/13122698/63d41e54-2636-4b1e-8b39-a93653e09ac0">


Let us now examine the CLI on the routers to verify the output.

Cisco IOS XRv9000 (show l2vpn xconnect):
```
Legend: ST = State, UP = Up, DN = Down, AD = Admin Down, UR = Unresolved,
        SB = Standby, SR = Standby Ready, (PP) = Partially Programmed,
        LU = Local Up, RU = Remote Up, CO = Connected, (SI) = Seamless Inactive

XConnect                   Segment 1                       Segment 2                
Group      Name       ST   Description            ST       Description            ST    
------------------------   -----------------------------   -----------------------------
TEST       TEST       UP   Gi0/0/0/1.512          UP       198.19.1.99     101    UP    
----------------------------------------------------------------------------------------
```

Juniper vMX (show l2circuit connections):
```
Layer-2 Circuit Connections:

Legend for connection status (St)   
EI -- encapsulation invalid      NP -- interface h/w not present   
MM -- mtu mismatch               Dn -- down                       
EM -- encapsulation mismatch     VC-Dn -- Virtual circuit Down    
CM -- control-word mismatch      Up -- operational                
VM -- vlan id mismatch		 CF -- Call admission control failure
OL -- no outgoing label          IB -- TDM incompatible bitrate 
NC -- intf encaps not CCC/TCC    TM -- TDM misconfiguration 
BK -- Backup Connection          ST -- Standby Connection
CB -- rcvd cell-bundle size bad  SP -- Static Pseudowire
LD -- local site signaled down   RS -- remote site standby
RD -- remote site signaled down  HS -- Hot-standby Connection
XX -- unknown

Legend for interface status  
Up -- operational            
Dn -- down                   
Neighbor: 198.19.1.4 
    Interface                 Type  St     Time last up          # Up trans
    ge-0/0/1.512(vc 101)      rmt   Up     Aug 23 18:06:20 2023           1
      Remote PE: 198.19.1.4, Negotiated control-word: Yes (Null)
      Incoming label: 80002, Outgoing label: 80007
      Negotiated PW status TLV: No
      Local interface: ge-0/0/1.512, Status: Up, Encapsulation: VLAN
      Flow Label Transmit: No, Flow Label Receive: No
```

# Conclusion

Establishing operational multi-vendor setups might appear daunting, yet there's always a pathway when you're willing to explore how these products operate at their core.
