.. srsLTE COTS UE Application Note

.. _cots_ue_appnote:

COTS UE Application Note
====================================


Introduction
************
srsLTE is usually run end-to-end using PCs and RF-frontends for the UE, eNB and EPC. But the standard srsUE can be replaced with a COTS handest, allowing 
users to connect to the network and perform multiple tasks. There are two options for network set-up when connecting a COTS UE. The network can be left as is, 
and the UE can communicate locally within the network. Or, the EPC can be connected to the internet through the P-GW, allowing the UE to access the internet for 
web-browsing, email etc. 

Hardware Required
----------------------------
Creating a network and connecting a COTS UE requires the following: 

 - PC with a Linux based OS, with srsLTE installed and built
 - An RF-frontend capable of both Tx & Rx
 - A COTS UE with compatable USIM/ SIM card 
 
The following diagram outlines the set-up: 
 
 .. image:: .imgs/cots_ue.png
 
For this implementation the following equipment was used: 
	
	- Dell XPS 13 running Ubuntu 20.04
	- B200 mini USRP
	- OnePlus 3t with a Sysmocom USIM 
	
Driver & Conf. File Set-Up
******************************
Before instantiating the network and conneting the UE you need to first ensure you have the correct drivers installed and that the configuartion files are edited appropriately. 

Drivers
----------
Firstly, check that you have the appropriate drivers for your SDR installed. If not they must be downloaded from the relevant source. If the drivers are already installed ensure 
they are up to date and are from a stable release. This step can be skipped if you have the correct drivers and know them to be working. 

* RF front-end drivers:

  * UHD:                 https://github.com/EttusResearch/uhd
  * SoapySDR:            https://github.com/pothosware/SoapySDR
  * BladeRF:             https://github.com/Nuand/bladeRF
  * ZeroMQ:              https://github.com/zeromq

When the drivers have been installed/ updated you should connect your hardware and check that everything is working correctly. To do this for a USRP using the UHD drivers run the following command:: 

	uhd_usrp_probe

This should be done anytime you are using a USRP before carrying out any testing or implementation to check a stable connection to the radio. Note, you should be using a USB 3.0 interface
when using an SDR for this use case.  

If you have had to install or update your drivers and everything is working as intended, then you will need to rebuild srsLTE to ensure it picks up on the new/ updated drivers. 

To make a clean build execute the following commands in your terminal:: 
	
		cd ./srsLTE/build
		rm CMakeCache.txt
		make clean
		cmake ..
		make
		
Your hardware and drivers should now be working correctly and be ready to use for connecting a COTS UE to srsLTE. 

Conf. Files
----------------
The base configuration files for srsLTE can be installed by running the following command in the build folder:: 

	sudo srslte_install_configs.sh <user/service>
	
You have the option to install the conifugrations files to the user directory or as a service. For this example the configuration files have been installed as a service by
running the following command ``sudo srslte_install_configs.sh service``. The config files can then be found in the following folder: ``~./etc/srslte``

You will need to edit the following files before you can run a COTS UE over the network: 

 - epc.conf
 - enb.conf
 - user_db.csv 
 
The eNB & EPC config files will need to be edited such that the MMC & MNC values are the same across both files. The user DB file needs to be updated so that 
it contains the credentials associated with the USIM card being used in the UE. 
 
**EPC:**

The following screenshot shows where to change the MMC & MNC values in the EPC config file: 
	
	.. image:: .imgs/epc_conf.png
		:align: center
		:height: 180px
	
Line 27 and 28 must be changed, for Sysmocom USIMS these values are 901 & 70. These values will be dependent on the USIM being used. 
	
**eNB:**

The above changes must be mirrored in the eNB config. file. The following screenshot shows this: 

	.. image:: .imgs/enb_conf.png
		:align: center
		:height: 150px
		
Here, the MMC and MNC values at lines 21 & 22 are changed to the values used in the EPC. 

For both of the config files the rest of the values can be left at the defulat values. They may be changed as needed, but further customization 
is not necessary to enable the successful connection of a COTS UE. 

**User DB:**

The following table describes the fields contained in the ``user_db.csv`` file, found in the same folder as the .conf files. As standard, this file 
will come with two dummy UEs entered in to the CSV, these help to provide an example of how the file should be filled in. 

.. list-table:: DB Fields
	:widths: 10 10 10 10 10 10 10 10 10 10
	:header-rows: 1

	* 	- Name
		- Auth
		- IMSI
		- Key
		- OP Type
		- OP
		- AMF
		- SQN
		- QCI
		- IP Alloc
	*	- Any human readable value
		- Authentication algorithm (xor/ mil)
		- UE's IMSI value
		- UE's key, hex value
		- Operatoer's code type (OP/ OPc)
		- OP/ OPc code, hex value
		- Authentication management field, hex value must be above 8000
		- UE's Sequence number for freshness of the authentication
		- QoS Class Identifier for the UE's default bearer
		- IP allocation stratagy for the SPGW

The AMF, SQN, QCI and IP Alloc fields can be populated with the following values: 
	
	- 9000, 000000000000, 9, dynamic

This will result in a user_db.csv file that should look something like the following: 

	.. image:: .imgs/user_db.png
		:align: center

Line 22 shows the entry for the USIM being used in the COTS UE. The values assigned to the AMF, SQN, QCI & IP Alloc are default values above, 
as outlined :ref:`here <config_csv>` in the EPC documentation. Ensure there is no white space between the values in each entry, as this will cause 
the file to be read wrong. 

The configuration files and DB should now be set up appropriatly to allow a COTS UE to connect to the eNB and Core. 

Connecting a COTS UE to srsLTE
****************************************
The final step in connecting a COTS UE to srsLTE is to first spin up the network and then connect to that network from the UE. The following sections 
will outline how this is achieved. 

Running srsEPC & srsENB
---------------------------------------
First navigate to the srsLTE folder. Then initialise the EPC by running::
	
	sudo srsepc
	
The following output should be displayed on the console: 

	.. image:: .imgs/epc_setup.png
		:align: center
		:height: 180px
		
The eNB can then be brought online in a seperate console by running::

	sudo srsenb 
	
The console should display the following: 

	.. image:: .imgs/enb_setup.png
		:align: center
		:height: 220px
		
The EPC console should now print an update if the eNB has sucessfuly connected to the core: 
		
	.. image:: .imgs/enb_connect.png
		:align: center
		:height: 180px
		
The network is now ready for the COTS UE to connect. 
		
Connecting the UE
---------------------------

Connecting the UE to the network is a quick and easy process if the above steps have been completed successfully. 

Firstly, make sure the USIM is inserted properly in the UE. Then follow the following steps: 

	- Open the Settings menu and navigate to the Sim & Network options

	.. image:: .imgs/ue_settings.jpg
		:align: center
		:height: 250px

	- Open this menu and proceed to the sub-menu asociated with the USIM being used. It should look something like the following: 

	.. image:: .imgs/sim_settings.jpg
		:align: center
		:height: 250px

	- Under the Network Operators find the network which you have just instantiated using srsLTE

	.. image:: .imgs/networks.jpg
		:align: center
		:height: 250px

	- Select the network that is a combination of your MMC & MNC values. For this example it is the network labelled 90170 4G. The UE should then automatically connect to the network. 
	
The UE should now be conected to the network. To check for a successfull connection use the logs prointed to the console. 

Confirming Connection
--------------------------------

Once the UE has been connected to the network, logs will be output to the consoles running the eNB and EPC. These can be used to confirm a succesful connection of the UE. 

**EPC Logs:**

The following ouptut is shown for the EPC after a successful attach. First a confirmation message in the form of *UL NAS: Received Attach Complete* will be displayed, secondly
the EPS bearers will be given out and the ID confirmed on the output, and lastly the *Sending EMM Impformation Message* output will be shown. If all of these are displayed in the 
logs, then an attach is succesful. These messages are seen in the last five lines of the console output in the following figure. 

	.. image:: .imgs/epc_attach.png
		:align: center
		:height: 400px

**eNB Logs:**

The eNB logs also display messages to confirm an attach. A *RACH* message should be seen followed by a *USER 0xX connected* message. Where "*0xX*" is a hex ID respresenting the UE. 

NOTE, you may see some other RACHs and *Disconnecting rtni=0xX* messages. This may be from other devices trying to connect to the network, if you have seen a clear connection between the UE and network 
these can be ignored. 

The following figure shows an output from the eNB that indicates a successful attach. 

	.. image:: .imgs/enb_attach.png
		:align: center
		:height: 240px

The UE is now connected to the network. This can be used to test modifications to the network or other use-cases in which a user would like to test 
network connectivity with a COTS UE. The UE should now automatically connect to this network each time. You should keep the UE in aeroplane mode until you want to connect it to the network. 

The UE will not be able to communicate outside of the network. To do this please follow the steps in the following section.

Connecting a COTS UE to the Internet
*********************************************
The following steps aim to show how to successfully enable an internet connection on a COTS UE, using srsEPC and srsENB. This will follow on from the steps outlined above, with one prerequisit before spinning up 
the network. The other steps required will be as before. 

Adding an APN to the UE
-------------------------------------
Firstly an APN is needed to allow the UE to access the internet. This is created from the UE and then a change is made to the EPC config file to reflect this. 

From the UE navigate to the Network settings for the SIM being used. From here an APN can be added, usually under *"Access point names"*. Create a new APN with the name and APN "test123", as shown in the following figure. 

	.. image:: .imgs/apn_ue.jpg
		:align: center
		:height: 250px

The addition of this APN must be reflected in the EPC config file, to do this add the APN to the config. This is shown in the following figure: 

	.. image:: .imgs/apn_epc.png
		:align: center
		:height: 250px
		
The APN has been added at line 30 in the above figure. This must match the APN on the UE to enable a successful connection. 
 
Run Masquerading Script
------------------------------------
To allow connection to the internet via the EPC, the pre-configured masquerading script must be run. This can be found in ``srsLTE/srsepc``. 

Before running the script it is imporant to identify the interface being used to connecct your PC to the internet. As the script requires this to be passed 
in as an arguement. This can be done by running the following command::

	route

You will see an output similar to the following: 

	.. image:: .imgs/route.png
		:align: center

The interface (Iface) associated with the *default* destination is one which must be passed into the masq. script. In the above figure that is the wlp2s0 interface. 

The masq. script can now be run from the follow folder: ``srsLTE/srsEPC``:: 

	sudo ./srsepc_if_masq.sh <interface>

If it has executed successfully you will see the following message: *Masquerading Interface <interface>* .

Instantiate Network
----------------------------
The network can now be instantiated in the same way as before. 

Firstly, run the EPC, followed by the eNB, and lastly connect the UE to the network. Confirmation messages will be displayed if a successful attach has been achieved in the same way as before. 
The user should not have to do anything else to get the UE to connect. 

Connect UE
-----------------
The UE can now be connected to the network. To enable connection to the internet ensure mobile data is turned on. The UE should now be able to browse the internet, send email and stream video - as if connected to a standard 
4G network. 

Troubleshooting
*******************
- Some users may experience trouble connecting to the internet, even after running the masquerading scipt. Ensure that IP forwarding is enabled, and check your network configuration as this may be stopping the UE from connecting successfully. 

- Users may also have trouble connecting to the networ. Firstly check that all information in the config. and DB files is correct. You may also need to adjust the gain parameters in the eNB config. file - without high enough power (<pmax threshold), the UE won't PRACH. 

- Some SIMs may not be compatable in UEs that are "locked" to certain network operators. 

