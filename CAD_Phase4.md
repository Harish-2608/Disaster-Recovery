# **Disaster Recovery with IBM Virtual Servers**

**Building the disaster recovery plan by configuring replication and testing recovery procedures**

In this case we will demonstrate how to implement PowerHA Geographic Mirroring in IBM I

**Note that this solution requires that the IBM I VSI and client application(s) use Independent Auxiliary Storage Pools (IASPs).** If the IBM i VSI and application(s) use only \*SYSBAS, this DR option will not work.

# **Solution Components and Requirements**

#

#

This solution uses the following components

1. Open an IBM Cloud account
2. Create two Power PowerVS location Services and a private subnet in each PowerVS location.
3. Provision IBM i VSIs in each PowerVS location
  1. A "production" IBM i cloud instance with an Independent ASP (IASP) that has been IASP-enabled (i.e. All changes/modifications allowing the IASP to function in a working environment should be completed before

Geographic Mirroring is set up for a DR solution.)

  1. A "DR" IBM i cloud instance with non-configured disks to be used for Geographic Mirroring. It is highly recommended that the number, type and capacity of disks match that of the production IASP.
1. Order Direct Link Connect Classic to connect each PowerVS location to IBM Cloud
2. Order two Vyatta Gateways one in each datacenter to allow for PowerVS location-to-location communication
3. Request a Generic Routing Encapsulation (GRE) tunnel to be provisioned at each PowerVS location.
4. Configure three GRE tunnels in the Vyatka Gateways. Two to connect Vyatta Gateway to the PowerVS location GRE tunnels created in Step 6 above and one across Vyatta Gateways to connect Vyatta-to-Vyatta. This will allow end-to-end PowerVS location-to-location communication for the VSIs in the PowerVS locations and to the IBM Cloud VSIs and other services such as Cloud Object Storage (COS) (if used).
5. Configure a Reverse-proxy Centos VSI to allow access to Private Cloud Object Storage endpoint from PowerVS location

Requirements

Open an IBM Cloud account

Login to [https://cloud.ibm.c](https://cloud.ibm.com/)[o](https://cloud.ibm.com/)[m](https://cloud.ibm.com/)[an](https://cloud.ibm.com/)d follow the procedure to open an Internal to external account.

For internal accounts, you can use your IBM intranet ID and password. For external accounts you will need to provide a billing source such as a credit card.

**Create PowerVS location Service and Subnet(s)**

All Power VSIs are provisioned in what is called a PowerVS location. This is a separate datacenter adjacent to IBM Cloud datacenters. In order to setup your PowerVS location, you will setup a PowerVS location service in the IBM Cloud UI. The PowerVS location service is a service within IBM Cloud which allows you to provision IBM i VSIs. There is a limit of one PowerVS location service per datacenter in IBM Cloud. In our scenario we have created two PowerVS locations, one is Toronto and one in London datacenters.

# Provision AIX and IBM VSIs in each PowerVS location

In each PowerVS location service you can create IBM i VSIs. The details are provided in the next section.

# Order Direct Link Connect Classic to connect PowerVS location to IBM Cloud

You will need to order Direct Link (DL) Connect Classic to allow your Power VSIs to communication with Linux/Window VSIs in IBM Cloud and also with all other IBM Cloud services such as VMWare VMs, and Cloud Object Storage (COS). Ordering a DL may take 1-2 weeks to complete.

There is no charge for this service as of June 2020.

# Order two Vyatta Gateways, one in each datacenter

In iam order to setup communication between the two PowerVS location datacenters, you will need to use Generic Routing Encapsulation (GRE) tunnels. GRE tunnels are provisioned on Vyatta Gateways so you will need to order one Vyatta Gateway in each PowerVS location.

The example here involves ordering one Vyatta in LON06 and the other in TOR01 datacenters where the PowerVS locations exist.

# Request a Generic Routing Encapsulation (GRE) tunnel to be provisioned at each PowerVS location

You will need to open a support ticket with Power Systems and request that a GRE tunnel be provisioned in each PowerVS location. They will provision their end of the GRE tunnel and send you the information so you can continue and provision your end on the Vyatta Gateways. You will need to provide the subnets information in each PowerVS location in the ticket.

# Configure three GRE tunnels in the Vyatta Gateways

We used the following link to configure the GRE.

https://cloud.ibm.com/docs/power-iaas?topic=power-iaas- configuringpower

After the support team finishes configuring the GRE tunnel, you will need to configure your end of the GRE tunnel on the two Vyatta Gateways.

You will need three GRE tunnels

1. _GRE tunnel on Vyatta to terminate the PowerVS location GRE in_ _LON06 (or "datacenter 1")_
2. _GRE tunnel on Vyatta to terminate the PowerVS location GRE in TOR01_

_(or "datacenter 2")_

1. _GRE tunnel across the two Vyatta gateways. One on each side._

# Configure a Reverse-proxy Centos VSI

In this section we will discuss the procedure to configure a reverse proxy to allow access to private COS endpoint.

All access to COS from Power VSI is via this reverse proxy.

You will access it via https://\<reverse\_proxy\_ip\>.

You will need to provision a Centos or Red Hat VSI in IBM cloud to configure at reverse proxy. This VSI must have public access. After configuration, the public access can be disabled.

# Create PowerVS location Services and Subnet(s)

You will need an IBM Cloud account to start this process.

Go to the main IBM Cloud UI page and click on "Catalog" on upper right side of the UI.

Your PowerVS location service will now appear under the Services tab.

You will repeat this process to create a second PowerVS location service. In this example, there are two PowerVS location services, one in London and one in Toronto.

Next you will need to click on the PowerVS location Service you created and provision a subnet to be used by your Power VSI servers

Choose "Subnets" from the menu on the left.

Provide the following information:

1. Name for your subnet
2. CIDR range. This can be any private IP subnet ranges. For example,

192.168.5.0/24. You may choose /21 to /30 based on how many IPs you will require. You may use your own private CIDR if you wish.

1. The rest of the fields will be automatically populated based on the CIDR you provided.

Press "Create Subnet"

There should be a VLAN ID associated with the subnet.

At this point, you will need to open a Support Ticket with Power System to request that the subnet be configured to allow local communication between any Power VSI you create in this PowerVS location service. Provide your PowerVS location service location, and your subnet in the ticket.

Without this step, the Power VSI you create will not be able to ping between each other even if they are on same subnet in the same PowerVS location.

# Provision AIX or IBM i VSIs in each PowerVS location

# Provisioning AIX or IBM i Virtual Server Instances (VSIs) in multiple PowerVS locations for disaster recovery purposes involves setting up redundant infrastructure to ensure high availability and data resiliency. Here's how you can provision VSIs in different PowerVS locations for disaster recovery:

#


# 1.Identify Disaster Recovery Requirements:
Determine your disaster recovery requirements, including Recovery Time Objective (RTO) and Recovery Point Objective (RPO). You should have a clear plan for what services and data need to be replicated to ensure business continuity.

# 2.Select Primary and Secondary Locations:
Choose one PowerVS location as your primary data center where your AIX or IBM i workloads will run. Select one or more secondary PowerVS locations that will serve as disaster recovery sites.

# 3.Set Up Replication:
Implement data replication mechanisms to keep your data synchronized between the primary and secondary locations. For AIX, you can use technologies like PowerHA. For IBM i, you can use built-in options like Remote Journaling, or third-party solutions.

# 4.Provision VSIs in Primary Location:
a. In the primary PowerVS location, provision your AIX or IBM i VSIs as per your production requirements.
b. Ensure that these VSIs are configured to use the necessary data replication mechanisms to maintain data consistency.

# 5.Provision VSIs in Secondary Locations:

a. In each of the secondary PowerVS locations, provision identical AIX or IBM i VSIs with the same configuration as in the primary location. This ensures that you have standby systems ready for failover.
b. Make sure that these secondary VSIs are set to remain in a standby or offline state until needed for disaster recovery.

# 6.Implement Monitoring and Automation:
Set up monitoring tools and automation scripts to detect failures in the primary site. If a failure is detected, initiate the failover process.

# 7.Test the Failover Process:
Regularly test your disaster recovery plan to ensure that the failover process works as expected. This testing helps identify and address any issues before a real disaster occurs.

# 8.Document the Procedures:
Ensure that you have detailed documentation of your disaster recovery procedures. This documentation should include step-by-step instructions for initiating failover and recovery processes.

# 9.Regularly Update and Maintain:
Regularly update your disaster recovery plan and ensure that your configurations in both the primary and secondary locations remain synchronized and up-to-date.
Remember that setting up a robust disaster recovery solution involves careful planning and testing. You may also want to consider third-party disaster recovery solutions that are specifically designed for AIX and IBM i environments to simplify the process.

The procedure is similar for both AIX and IBM i VSI provisioning. Here is a procedure to create an AIX 7.2 VSI. The cost shown are monthly costs, but you are being charged hourly.

Go to the IBM Cloud Catalog and press the "IBM Cloud" on top left side of the UI.

Choose "Services" from the list shown.

Click on the service for datacenter in which you have created a PowerVS location power service. In this case we will choose Toronot01 PowerVS location service.

Since we have already provisioned several VSIs, we see the list shown above. If you are creating VSIs for the first time, your list will be empty.

Press "Create Instance" on upper right-hand side.

This is where you provision AIX or IBM i VSIs.

Choose a name for your VSI, i.e., AIX-72-Tor01 and select how many

VSIs you need to configure. The names of the VSI will be appended with

a "-1", "-2" etc. if you select more than one VSI. Note that the IBM i will give the LPAR a system name consisting of the first 8 characters, and it is best to use only alphanumeric characters for IBM i naming.

You may leave VM pruning and SSH key as-is since the VSIs will have no passwords when you create them for the first time. You will need to create a password via the OS command. This is AIX-specific and those two options are not related to IBM i instances. The O/S and DST passwords for user qsecofr will default to QSECOFR and will be changed later (discussed in the section on configuration).

Scroll down to choose other options.

Here you will choose the following options:

- Operating System – AIX or IBM i or any other image you may have imported via the "Boot Image" menu on the left.
- Image type: AIX 7.1 or 7.2, IBM i 7.3 or 7.4, for example.
- Software license: Be sure to include "IBM i PowerHA" and any others as necessary.
- Disk types: Type 1 or 3. Type 3 is a less expensive option which we selected.
- Machine type: S922 or E980. S922 is the cheaper of the two which we selected.
- Processor: Dedicated or Shared or Shared Capped. We chose "shared" as its less expensive.
- Choose the number of cores and RAM you will need. The minimum core is "0.25". IBM i partitions have typically required at least 4 GB of memory, just to boot, so this should be your minimum value as well.
- You can also attach additional volume to the VSI is you wish. We did not do that here and only used the root volume which is included. As discussed later, it is HIGHLY recommended to add the disks one at a time, after creating the initial Load Source (LS) volume. It makes it much easier to keep track of which disk unit

_ID matches the volume name in the IBM Cloud UI._ •_This solution_

_requires that IBM PowerHA SystemMirror for i (Enterprise Edition) is installed on both the production and DR nodes in the solution. This product is 5770HAS \*BASE and Option 1_

- _No matter what O/S version/release, it is always recommended to have the latest PTFs, which can be found here:_
- _[https://www.ibm.com/support/pages/i](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[b](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[m](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[-](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[i](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[suppo](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[rtrecommend](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[e](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[dfix](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[e](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)[s](https://www.ibm.com/support/pages/ibm-i-support-recommended-fixes)_
- _Since PowerHA for IBM i is owned by HelpSystems, the following link is useful as well, to ensure the latest HA group PTFs are installed:_
- _[https://helpsystemswiki.atlassian.net/wiki/spaces/IWT/pages/1](https://helpsystemswiki.atlassian.net/wiki/spaces/IWT/pages/162627587/PowerHA+PTF+Groups)[6](https://helpsystemswiki.atlassian.net/wiki/spaces/IWT/pages/162627587/PowerHA+PTF+Groups)_

_[2627587/PowerHA+PTF+Grou](https://helpsystemswiki.atlassian.net/wiki/spaces/IWT/pages/162627587/PowerHA+PTF+Groups)[p](https://helpsystemswiki.atlassian.net/wiki/spaces/IWT/pages/162627587/PowerHA+PTF+Groups)[s](https://helpsystemswiki.atlassian.net/wiki/spaces/IWT/pages/162627587/PowerHA+PTF+Groups)_

Next you will scroll down to choose your subnet on which these VSIs will be provisioned. It is assumed you have already created one or more subnets prior to this step.

Click on the "Attached Existing" under networks.

Choose the subnet you wish to attach, and the press "Attach"

Now check the box "I agree to the …." And press "create Instance" in lower right-hand side.

Your VSI is now being provisioned.

# Order Direct Link Connect Classic to connect PowerVS location to IBM Cloud

You will need to order Direct Link (DL) Connect Classic to allow your

Power VSIs in the PowerVS location to communication with Linux/Window VSIs in IBM Cloud and also with all other IBM Cloud services such as Cloud Object Storage and VMware services. This process may take 1-2 weeks to complete.

There are several steps involved in completing DL ordering:

- Order Direct link connect classic service on IBM Cloud UI – see steps below
- Next a support ticket will be created, and Support will send you a word document with questionnaires to be completed concerning various DL settings.
- Complete the questionnaires and upload it to support in the ticket.
- Support will then request that you create a new support ticket with the Power System so they can complete their side of the DL provisioning. Attach information about the DL in the original ticket to this ticket.
- The DL will be provisioned, and you will be notified when complete.
- You can now test connection to any Linux/Windows VSI you may have in IBM Cloud and other IBM Cloud services_._

To start the DL order process, go to IBM Cloud UI and log in.

Choose "Catalog" from upper right-hand side, and search for "direct".

Select "Direct Link Connect on Classic".

Press "Create". There are no options to select.

Now choose "Order Direct Link Connect" from top right-hand side.

- _Choose a "name" for the DL._
- _Choose a location for the DL. This should be the same location as where you created your PowerVS location Service._
- _Choose "link speed" under network provider menu._
- _Choose "Local Routing (free)"_
- Global routing will require additional charges and will allow for easier PowerVS location-to-Powers location communication. You will also need to order a Vyatta Gateway Router to complete your Global routing option via use of a GRE tunnel. Support can help you with this further.
- In our case, we decided to use Local Routing and then order a Vyatta Gateway in each PowerVS location and provision a GRE tunnel endtoend.

Check the box to accept the offer and press "Create"

A support case will be opened with the information required.

After this is complete, you will then be contacted by support and requested to complete and answer some questions in an attached document and send it back as attachment to the same ticket.

After this step is complete, support will request that you open a new IBM support ticket and address it to the Power System. Include the information in the original DL ticket. This new ticket will be sent to the PowerVS location support to configure their side of the DL connection.

This should be the last step before DL communication works. You can test your connection by pinging IBM Cloud Linux/Windows VSI from your Power VSIs and in reverse.

**Request a Generic Routing Encapsulation (GRE) tunnel to be provisioned at each Power VS location**

You will need to open a support ticket to Power Systems and request that a GRE tunnel be provisioned in each Power VS location. You will need to provide information on the subnets you created in the Power VS location. They will provision their end of the GRE tunnel and send you the information you will need so you can continue and provision your end of the GRE tunnel on the Vyatta Gateways.

Verify your Vyatta Gateway access.

The Vyatta Gateway address can be find in the IBM Cloud UI under Devices.

Login to IBM Cloud UI and press "IBM Cloud" on top left-hand side. Click on "Devices"

Choose the Vyatta system you like to configure:

- _[vyat](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[t](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[a](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[-](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[labservic](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[e](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[s](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[-](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[lon.ibm.clo](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[u](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[d](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)_
- _[vyat](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[t](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[a](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[-](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[labservic](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[e](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[s](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[-](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[tor.ibm.clo](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[u](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[d](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)_

LON06:

Click on the London Vyatta: [vyat](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[t](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[a](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[-](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[labservic](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[e](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[s](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[-](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[lon.ibm.clo](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[u](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)[d](https://cloud.ibm.com/gen1/infrastructure/bare-metal/1374067/details)

Under the "Network Details" you will see your Vyatta Gateway IP address:

Your credentials are under the "password" menu on the left-hand side. Click on the icon next to the password to see it unencrypted.

Open a browser and login to the Vyatta Gateway using:

userID: Vyatta

Password: as show in the GUI [https://10.72.74.2](https://10.72.74.203/)[0](https://10.72.74.203/)[3](https://10.72.74.203/)[ss](https://10.72.74.203/)h vyatta@10.72.74.203

Note: Prior to login to a 10.x.x.x private IPs in IBM Cloud you will need to start your MotionPro Plus VPN access. This will give you access to IBM Cloud private IPs.

Login with the userID and password.

Now that you have verified you access to the Vyatta Gateways, you will need to now access it via ssh to continue your GRE tunnel provisioning.

# Setup PowerVS location GRE tunnels in the Vyatta Gateways

The following references may help in configuring GRE tunnels: [https://cloud.ibm.com/docs/virtu](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[a](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[l](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[-](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[rout](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[e](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[r](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[appliance?topic=soluti](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[ontutoria](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[l](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[sconfiguri](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[n](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[g](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[-](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[IPS](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[E](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[C](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[-](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[V](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[P](https://cloud.ibm.com/docs/virtual-router-appliance?topic=solution-tutorials-configuring-IPSEC-VPN)[N](https://docs.huihoo.com/vyatta/6.5/Vyatta-Tunnels_6.5R1_v01.pdf)[https://docs.huihoo.com/vyatta/6.5/Vyat](https://docs.huihoo.com/vyatta/6.5/Vyatta-Tunnels_6.5R1_v01.pdf)[t](https://docs.huihoo.com/vyatta/6.5/Vyatta-Tunnels_6.5R1_v01.pdf)[aTunnels\_6.5R1\_v01.p](https://docs.huihoo.com/vyatta/6.5/Vyatta-Tunnels_6.5R1_v01.pdf)[d](https://docs.huihoo.com/vyatta/6.5/Vyatta-Tunnels_6.5R1_v01.pdf)[f](https://docs.huihoo.com/vyatta/6.5/Vyatta-Tunnels_6.5R1_v01.pdf)https://cloud.ibm.com/docs/power-iaas?topic=power-iaas-configuringpower

Open a command window on your Mac/Window.

Note: Prior to login to a 10.x.x.x private IPs in IBM Cloud you will need to start your MotionPro Plus VPN access.

**Setup GRE PowerVS location Tunnel in LON06:**

userID: Vyatta

Password: as show in the GUI ssh vyatta@10.72.74.203 ssh to

LON06 Vyatta Gateway.

Run the following commands:

We have chosen to call our tunnel "tun0" on the Vyatta Gateway.

- _configure_

- _set interfaces tunnel tun0 address 172.20.2.1/30_
- _set interfaces tunnel tun0 local__-Ip 10.72.74.203_
- _set interfaces tunnel tun0 remote-Ip 10.254.0.26_
- _set interfaces tunnel tun0 encapsulation_
- _set interfaces tunnel tun0 mute 1_
- _commit__exit_

![](RackMultipart20231026-1-kvubvi_html_e3db6191434057bb.jpg)

You can verify that your GRE tunnel is setup by running the following commands:

|  | _configure_ |
| --- | --- |
|  | _show interfaces tunnel_ _Or to get more info:_ |
|  | _Show interface tunnel tun0_ |
|  | _exit_ |

**Setup GRE PowerVS location Tunnel in TOR01:**

userID: Vyatta

Password: as show in the GUI ssh vyatta@10.114.118.34 ssh to Tor01 Vyatta Gateway.

![](RackMultipart20231026-1-kvubvi_html_b627de94b4b819da.jpg)

To show the status:

|  | _configure_ |
| --- | --- |
|  | _show interfaces tunnel_ _Or to get more info:_ |
|  | _Show interface tunnel tun0_ |
|  | _exit_ |

![](RackMultipart20231026-1-kvubvi_html_ccdf106307af9305.jpg)

![](RackMultipart20231026-1-kvubvi_html_ccdf106307af9305.jpg)

Setup GRE tunnel between Two Vyatta Gateways

In this section you will setup a new tunnel in each of the two Vyatta gateways to allow for cross Vyatta connection via a GRE tunnel.

In this case we chose the tunnel address and tunnel source and destination IPs. The tunnel address can be any IP subnet you choose. We named our tunnel "tun1" in both Vyatta Gateways. We have selected a similar IP as the ones used in the PowerVS location GRE tunnels. We choose a CIDR of /30 since we only need two IP address, one in Tor01 and one in Lon06.

- _In Lon06 Vyatta the GRE Vyatta-to-Vyatta tunnel address is 172.20.4.1/30_
- _In Tor01 Vyatta the GRE Vyatta-to-Vyatta tunnel address is 172.20.4.2/30_
- _Your tunnel destination IP is the IP address of the Vyatta gateway in each location_
- _Your tunnel source IP is the IP address of the Vyatta gateway in each location_
- _We call the tunnels tun1 in both locations_

TOR01 GRE Configuration:

![](RackMultipart20231026-1-kvubvi_html_47674993fc2c21b3.jpg)

LON06 GRE Configuration:

# configure

- _set interfaces tunnel tun1 address 172.20.4.2/30_

- _set interfaces tunnel tun1 remote-ip 10.114.118.34_
- _set interfaces tunnel tun1 local-ip 10.72.74.203_

|set interfaces tunnel tun1 encapsulation gre__interfaces tunnel tun1 mtu 1300__commit_ |  _set_ |
| --- | --- |
exit

![](RackMultipart20231026-1-kvubvi_html_f986b6455d12d243.jpg)

The final step needed is to setup static routes in each Vyatta to point the subnets for our PowerVS location to the right tunnels.

Setup GRE tunnel between Two Vyatta Gateways

- Find the subnets you created in each PowerVS location in TOR01 and LON06 by accessing the services in the IBM Cloud UI for each PowerVS location.
- The static routes in LON06 will need to point to the subnets in TOR01 and vice versa.
- We will configure both GREs to the PowerVS location and between Vyattas.
- Run the following commands in each Vyatta Gateway after login in via ssh using the Vyatta userID:
- At this point you should have end-to-end connectivity and be able to ping between your Power VSIs in each PowerVS location and also from the Power VSI to IBM Cloud services such as Linux/Windows VSI.
