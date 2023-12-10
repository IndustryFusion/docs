The following documentation steps illustrates the setup of IFF open source components including PDT inside a factory premise as shown in the figure below. The main goal is to enable a Kuberenets expert to setup the IFF open stack and start with the creation of digital assets, semantic model and validation for industrial machines/processes supporting MQTT/OPC-UA protocols at the machine end.


![image](https://github.com/IndustryFusion/docs/assets/128161316/dfa28417-bd79-465f-9e6e-c25d4029251b)

### 1. OPC-UA Server / MQTT Publisher

a. In order to enable the digitization of the machines with OPC-UA server, make sure that the machine is connected to the factory LAN. Note down the IP address and port of the OPC-UA server. For example, "opc.tcp://192.168.49.198:62548".

Also, to enable the automated dicovery later with Akri Discovery Handler, note down the 'applicationName' value of the OPC-UA server, as shown in the example below.

![image](https://github.com/IndustryFusion/docs/assets/128161316/c7f19949-8e1b-42de-8962-7668e690fccd)

b. In order to enable the digitization of the machines with MQTT Publisher, make sure that the machine is connected to the factory LAN. The machines with MQTT publisher usually comes with an UI for configuring the MQTT central broker IP address and port as shown below. This page will be used later, once the MQTT broker is functional in the central factory server.

![image](https://github.com/IndustryFusion/docs/assets/128161316/7d2eda97-9797-4b08-9285-ca7f4060d443)

### 2. Factory Server
#### a. Hardware Requirements:
* Intel Xeon Processor - Minimum, 8 Cores (16 CPU Threads).
* Memory - Minimum, 16 GB DDR4.
* Storage - Minimum, 512 GB SSD.

#### b. SLE Micro OS - 5.5
Visit this page [OS download](https://www.suse.com/download/sle-micro/) and download the 'SLE-Micro-5.5-DVD-x86_64-GM-Media1.iso' image. Flash the ISO to an USB drive (Min. 16GB). Insert and boot the server from the USB drive. Follow the on-screen steps to complete the OS installation. Skip the 'Product Registration' page if the free version is desired.

For more detailed installation steps follow this [documentation](https://documentation.suse.com/sle-micro/5.5/html/SLE-Micro-all/cha-install.html).

#### c. RKE2 - Kubernetes Distribution
Install the RKE2 latest stable version on the previously installed OS. IFF stack currently uses single node for the factory server, follow the instructions [here](https://docs.rke2.io/install/quickstart#server-node-installation) to install the server node.

Once the installation of RKE2 is done, Kubectl CLI must be able to communicate with the Kubernetes API and execute commands.

#### d. Rancher with Fleet and Elemental Plugins
Follow these [instructions](https://ranchermanager.docs.rancher.com/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster#install-the-rancher-helm-chart) to install the Rancher tool using Helm charts.

**Note:**
1. Use the 'rancher-stable' Helm chart.
2. Use the 'Rancher-generated TLS certificate', so install cert-manager as described.
3. At the last step, set the --version tag to 2.7.3.
4. Also, global.cattle.psp.enabled to false.
5. Set the hostname to your desired name. For e.g, rancher.iff.org

**Important:** 

Once the installation is done, set the DNS for the factory server in the LAN and set the hostname from last step. If not, update /etc/hosts file to have this DNS atleast. The Rancher UI will be live at this DNS in the LAN.

**Troubleshooting:**

If there any issue with resolving DNS name in the factory server. Most probably it is because of NetworkManager.

**Solution:**

Login to the factory server, and perform following steps.

* Edit /etc/sysconfig/network/config and update NETCONFIG_DNS_POLICY=" "
* Edit /etc/NetworkManager/NetworkManager.conf and make sure the below params are defined as shown.
        
         dns=default
         rc-manager=netconfig

* Restart the below services.

        sudo service network restart
        sudo service NetworkManager restart

**Fleet** - Continous Delivery Plugin will be installed by default. 

**Elemental** - OS management plugin must be installed seperatley. Follow these [instructions](https://elemental.docs.rancher.com/quickstart-ui#install-elemental-operator) to install Elemental using Rancher UI. **Note** - Follow the above doc untill you can see the OS Manamagent option in the Rancher Manager menu. Further steps for creating Machine Registration Endpoint and Preparing Seed Image will be described below.

**Machine Registration Endpoint Creation**

Go to OS management option in the Rancher - Click on 'Registration Endpoints' and click 'Create'.

![image](https://github.com/IndustryFusion/docs/assets/128161316/3ad21d4d-a8fa-4eab-9a8b-45a5af26e697)

IFF has tested two types of smartbox/gateway configurations - With Truted Platform Module 2.0 (TPM) and Without TPM 2.0

* With Truted Platform Module 2.0

For TPM 2.0 enabled smartboxes, use the following cloud config in the above shown screen.

```
config:
  cloud-config:
    users:
      - name: root
        passwd: password
  elemental:
    install:
      debug: true
      device: /dev/mmcblk0
      eject-cd: true
      no-format: false
      reboot: true
    reset:
      reboot: true
      reset-oem: true
      reset-persistent: true
machineInventoryLabels:
  machineUUID: ${System Information/UUID}
  manufacturer: ${System Information/Manufacturer}
  productName: ${System Information/Product Name}
  serialNumber: ${System Information/Serial Number}
```

* Without Truted Platform Module 2.0

For TPM 2.0 disabled smartboxes, use the following cloud config in the above shown screen.

```
config:
  cloud-config:
    users:
      - name: root
        passwd: password
  elemental:
    install:
      debug: true
      device: /dev/sda
      eject-cd: true
      no-format: false
      reboot: true
    registration:
      auth: tpm
      emulate-tpm: true
      emulated-tpm-seed: 1
    reset:
      reboot: true
      reset-oem: true
      reset-persistent: true
machineInventoryLabels:
  machineUUID: ${System Information/UUID}
  manufacturer: ${System Information/Manufacturer}
  productName: ${System Information/Product Name}
  serialNumber: ${System Information/Serial Number}
```

**Note:** Update the 'emulated-tpm-seed' to a unique number everytime for each TPM 2.0 disabled machine. Also, according to the smartbox configuration, update the device path and password.

Click 'Create' in the 'Registration Endpoint: Create' page after entering the above cloud configs accordingly. The below page will be visible, select the latest Elemental OS version and click 'Build ISO', then Click 'Download ISO' to download the ISO file.

![image](https://github.com/IndustryFusion/docs/assets/128161316/a0d4dfd4-3c30-440d-b9a9-19f4cb3616b3)

