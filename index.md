---
---

This procedure outlines how to enable the storage network on the VME Manager VM. This will enable GLFS access for the VME Manager, allowing snapshots captured by the VME Manager to be stored on the shared GLFS storage.




Before making any changes, please back up the network configuration on the Ubuntu-based Control Nodes and the Ubuntu-based VME Manager VM.




## On the Control Node:
1. Create a backup of 60-mvm-mgmt.yaml

sudo cp /etc/netplan/60-mvm-mgmt.yaml /etc/netplan/60-mvm-mgmt.yaml.org.bkp




2. Create a backup of 01-base.yaml

Bash
sudo cp /etc/netplan/01-base.yaml /etc/netplan/01-base.yaml.org.bkp




3. Save the active Netplan configuration output

Bash
sudo netplan get > /etc/netplan/netplanoutput.org.bkp




##On the VME Manager VM:

1. Create a backup of 50-cloud-init.yaml

Bash
sudo cp /etc/netplan/50-cloud-init.yaml /etc/netplan/50-cloud-init.yaml.org.bkp




2. Save the active Netplan configuration output

Bash
sudo netplan get > /etc/netplan/netplanoutput.org.bkp




Now that you've backed up your existing configuration files, here are the steps to create the new netplan configuration file /etc/netplan/61-mvm-strg.yaml for storage network and update 01-base.yaml on the Control Nodes. No changes required in the 60-mvm-mgmt.yaml.


1. Create /etc/netplan/61-mvm-strg.yaml

Open the new Netplan file for editing using sudo nano or vi editor

Bash
sudo vi /etc/netplan/61-mvm-strg.yaml


2. Paste the following content into the editor.


YAML
network:
  bridges:
    strg:
      addresses:
      - 172.28.2.101/21
      dhcp4: false
      interfaces:
      - VLAN110
      link-local:
      - ipv4
      - ipv6
      mtu: 9000
      openvswitch: {}
      routes:
      - to: 172.28.0.0/21
        via: 172.28.0.1
  version: 2



###Note:### Remember to change the IP details in the configuration to match your specific storage network requirements.

3. Save and exit the editor

4. Update the permissions

Bash
sudo chmod 600 /etc/netplan/61-mvm-strg.yaml







2.Update /etc/netplan/01-base.yaml

Open the new Netplan file for editing using sudo nano or vi editor

sudo vi /etc/netplan/01-base.yaml

       2. Update the VLAN section of the YAML as below:




Replace

  vlans:

    VLAN110:

      addresses:

      - 172.28.2.101/21

      dhcp4: false

      id: 110

      link: bond1

      mtu: 9000

      routes:

      - to: 172.28.0.0/21

        via: 172.28.0.0

With

  vlans:

    VLAN110:

      id: 110

      link: bond1

 

      3. Save and exit the editor




  3. Apply the Netplan Configuration

 After updating these files, you need to apply the changes for them to take effect. It's highly recommended to use netplan try first to avoid locking yourself out of the network.

 

Test the configuration

     sudo netplan try

     If there are no errors, it will prompt you to press Enter to apply, or it will revert after a timeout.

    If there are errors, it will tell you what's wrong, and you can fix the YAML before applying.

      2. Apply the configuration permanently (if netplan try was successful):

     sudo netplan apply

After netplan apply, your network interfaces should be reconfigured according to these new settings.

 

Please do these changes on all the Control Nodes

 Now we will proceed with making the network changes on the VME Manager VM.




Create /etc/netplan/01-storage-network.yaml

 

      1. Open the new Netplan file for editing using nano or vi editor

sudo vi /etc/netplan/01-storage-network.yaml

      2. Paste the following content into the editor.

 network:

  version: 2

  ethernets:

    eth1:

      dhcp4: no

      addresses: [172.28.2.110/21]

 Note: Remember to change the addresses and routes IP details in the configuration to match your specific storage network requirements.

       3. Save and exit the editor

       4. Update the permissions

sudo chmod 600 /etc/netplan/01-storage-network.yaml




       2.  Apply the Netplan Configuration

 

After updating these files, you need to apply the changes for them to take effect. It's highly recommended to use netplan try first to avoid locking yourself out of the network.

 

       1. Test the configuration

           sudo netplan try

    If there are no errors, it will prompt you to press Enter to apply, or it will revert after a timeout.

    If there are errors, it will tell you what's wrong, and you can fix the YAML before applying.

        2. Apply the configuration permanently (if netplan try was successful):

            sudo netplan apply

After netplan apply, your network interfaces should be reconfigured according to these new settings.

       

       3.  Create Route and Bridge for VME Manager

Login to HPE VM Essentials GUI through a browser and create a network route.

      Go to Infrastructure --> Network --> Routers

      Click on Add --> OVS Bridge Domain




 




       2. Fill the required details as shown in the figure and click on “Add Network Router”





        3. Add a storage network interface to the VME Manager VM.

Go to Infrastructure --> Compute --> Virtual Machines

Look for the VME Manger VM by the name vmemanager.houcbglr1.hpecorp.net Or something similar. Click on it, it will show the details of the VME Manager VM.

Now click on the “Actions” button at the top right corner and then click configure.

              




It will open the “Reconfigure Server” page for the VME Manager VM





Add the storage interface as shown in the figure and click on reconfigure.




In case the the VME Manager VM Powers off ater this action. Go to the Control node and power on the VME Manager VM

On the Control Node

List the VMs

virsh list --all


       2. Start the VME Manager VM

 virsh start vmemanager.houcbglr1.hpecorp.net




With these configurations successfully applied, the storage network is now enabled on the VME Manager VM, allowing it to communicate directly with your HPE GreenLake for File Storage (GLFS).

To verify the setup:

On the VME Manager VM, execute sudo netplan get to confirm that the storage network interface is active and shows the expected IP configuration.

Attempt to communicate with your GLFS IP address (e.g., ping 172.28.2.2). A successful ping confirms connectivity to the GLFS storage.

Your VME Manager VM is now prepared to leverage GLFS for its storage requirements, including snapshot storage.

 

1.2. Mount GLFS Share on VME Manager VM
This procedure outlines how to mount a GLFS share on a VME Manager VM. Once the GLFS share is mounted on the VME Manager, it can be used to store snapshots directly on the GLFS share.

 

1.2.1.1. 1. Install nfs-common
To mount the GLFS on the VME Manager, we need to install nfs-common on the VME Manager VM.

Populate the ubuntu.sources file with the correct content. Open the ubuntu.sources file using the vi editor:

sudo vi /etc/apt/sources.list.d/ubuntu.sources

Paste the below content into the file:

Types: deb
URIs: http://archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Run the command below to update the cache:

sudo apt update

Install the nfs-common package:

sudo apt install nfs-common

 

1.2.1.2. Mount GLFS Share on VME Manager
Before we mount the GLFS on the VME Manager VM, stop the morpheus-ui service:

morpheus-ctl stop morpheus-ui

Mount the GLFS on the VME Manager using the commands below:

mkdir -p /mnt/glfs_view; mount -o nfsvers=4 172.28.2.2:/esxshare /mnt/glfs_view

Create a directory to store all the backups:

mkdir /mnt/glfs_view/pcai-backup

chmod 777 /mnt/glfs_view/pcai-backup

Reconfigure the application

morpheus-ctl reconfigure

Start the morpheus-ui service:


morpheus-ctl start morpheus-ui

 

1.2.1.3. Configure VME Manager to Store Snapshots on GLFS share
 

Launch the VME Manager GUI through the browser and log in.

Navigate to:

Infrastructure

Storage

File Shares

Add

Local Storage

 

Fill in the details as shown in the figure below.


After saving the changes, you should be able to see the Share Path under File Shares.


Your setup is now ready to back up snapshots on the GLFS