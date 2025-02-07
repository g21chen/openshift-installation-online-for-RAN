# Table of Contents
- [Descriptions](#Descriptions)
- [Preconditions](#Preconditions)
  - [Hardware](#Hardware)
  - [Others](#Others)
- [Installation steps](#Installation-steps)
  - [1 BIOS configuration](#1-BIOS-configuration)
  - [2 Generate redhat discovery ISO file](#2-Generate-redhat-discovery-ISO-file)
    - [2.1 preparing for the following configuration information](#21-preparing-for-the-following-configuration-information)
    - [2.2 precheck the condition for openshift container platform installtion](#22-precheck-the-condition-for-openshift-container-platform-installtion)
    - [2.3 Generate the ISO image with paramters configuration](#23-Generate-the-ISO-image-with-paramters-configuration)
  - [3. mount the iso to target server via BMC GUI](#3-mount-the-iso-to-target-server-via-BMC-GUI)
    - [3.1 login the BMC network](#31-login-the-BMC-network)
    - [3.2 configure server boot from CD/DVD Drive](#32-configure-server-boot-from-CDDVD-Drive)
    - [3.3 mount iso file](#33-mount-iso-file)
    - [3.4 reset server](#34-reset-server)
  - [4. openshift container platform installation](#4-openshift-container-platform-installation)
    - [4.1 host is detected from redhat console GUI](#41-host-is-detected-from-redhat-console-GUI)
    - [4.2 install OCP](#42-install-OCP)
  - [5. openshift container platform postconfiguration](#5-openshift-container-platform-postconfiguration)
  - [6. Reference](#6-reference)

## **Descriptions**
The purpose of this repsosity is to specify the openshift installation online and also the operators configuration.  Note: all vendor specific information and configuration will be hidden due to license/privacy rule. 
The redhat document provide the full installation guide as well. this repository focus on only necessary steps for the beginners. 


## **Preconditions**
  ### **Hardware** 
      1. target HPE DL110 or Dell server or Supermicro or Nvidia server (X86 or ARM CPU model)
      2. Jump server (optional)

  ### **Others**
      1. Internet access via proxy
      2. redhat account
      
## **Installation steps**
  ### **1 BIOS configuration**
    a. By default, the virtuallation and SR-IOV functionality are not enabled in BIOS. if those are required for the applications on top of Openshift Container Platform, those need to be enabled in BIOS configuration.
    b. For the Nvidia or supermicro server with DPU, diable the DPU in case the NIC card is used as common NIC card e.g SR-IOV functionality. see figure 1
    
Figure 1: disable DPU in supermicro/nvidia server with DPU NIC card e.g bluefield NIC 
![DPU-disabled](https://github.com/user-attachments/assets/c3169f85-9515-4012-8e8e-0736d51ff291)


  ### **2 Generate redhat discovery ISO file**
  #### **2.1 preparing for the following configuration information**
      the following information are required as input to install openshift container platform later.
      a. Network configuration 
         - MAC address of target server for infra network
         - Infrastructure vlan if vlan is needed in IP planning
            I: subnetwork
            II: VLAN 
            III: Gateway
            IV: IP address of infra network
       b. Proxy to access internet
          - httpproxy
          - httpsproxy
          - noproxy:  e.g .supermicro-test.vran.xxx.yyy-zzz.net,10.128.0.0/14,172.30.0.0/16,xxx.yyy.zzz.ttt/29   (xxx.yyy.zzz.ttt/29 is the subnetwork of infra network)

       c. DNS server
           -  DNS server iP
           -  add DNS entry to DNS server. the example DNS entries for single node server as below (10.104.xxx.yyy is the infrastructure IP):
              api.supermicro-test.vran.xxx.yyy-zzz.net	      	10.104.xxx.yyy
              api-int.supermicro-test.vran.xxx.yyy-zzz.net	  	10.104.xxx.yyy
              *.apps.supermicro-test.vran.xxx.yyy-zzz.net	    	10.104.xxx.yyy
              master0.supermicro-test.vran.xxx.yyy-zzz.net	  	10.104.xxx.yyy

       d. NTP server (optional)
           if not porvide NTP server, Openshift Container Platform use its own NTP server


       e. ssh key
          Generate the SSH key in one server which install the ssh-keygen tool with following command
          ssh-keygen -t rsa -b 4096 -f ./id_rsa
          after the command execution, One pair of keys are generated, private key: id_rsa and public key: id_rsa.pub
          later the public key need to be provided as input to generate iso file

  #### **2.2 precheck the condition for openshift container platform installtion**
       a. Network
          - ping BMC network reachable.  later the generated iso image need to be mannually mounted to target server via access the BMC network
          - green light on NIC port used for infra network. or check the link status is "UP" from BMC Webgui  or check the link status on switch port connected to target server for infrastruture network
          - ping gateway of infrastructre network reachable
          note: at this time it is not reachable if ping infrastruture IP address

        b. Proxy
           - add that Proxy in other server and able to access the internet.

        c. DNS server
          dig dns entry. e.g following example
          [xxxxx ~]$ dig api.supermicro.xxx.yyy.zzz.net

              ; <<>> DiG 9.16.23-RH <<>> api.supermicro.xxx.yyy.zzz.net
              ;; global options: +cmd
              ;; Got answer:
              ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48866
              ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
              
              ;; OPT PSEUDOSECTION:
              ; EDNS: version: 0, flags:; udp: 1232
              ; COOKIE: bcf5324c8189920c01000000677707b7885865226f134e25 (good)
              ;; QUESTION SECTION:
              ;api.supermicro.xxx.yyy.zzz.net. IN A![Create cluster](https://github.com/user-attachments/assets/21374bd0-60a2-4d95-a509-19c3bd5ad251)

              
              ;; ANSWER SECTION:
              api.supermicro.xxx.yyy.zzz.net. 3600 IN A 10.104.xxx.yyy  -> 10.104.xxx.yyy is the infra IP
              
              ;; Query time: 1 msec
              ;; SERVER: aa.bb.cc.dd#53(aa.bb.cc.dd)  --> aa.bb.cc.dd is the DNS server IP
              ;; WHEN: Thu Jan 02 15:40:07 CST 2025
              ;; MSG SIZE  rcvd: 122

       d. NTP server
          ping IP is reachable

#### **2.3 Generate the ISO image with paramters configuration**
   
   a. login the https://console.redhat.com/openshift/cluster-list.
   
   b. click "Create cluster" button to trigger cluster creation. see figure 2.
       ![Create cluster](https://github.com/user-attachments/assets/85bcfd7f-c78d-4d95-9072-b2d88e2f6c74)
       ![Datacenter](https://github.com/user-attachments/assets/65cb8978-9237-4371-a9b7-f75407c0604d)

   c. configure "cluster detail".

   ![ClusterDetail](https://github.com/user-attachments/assets/22784744-73e1-4c56-8e20-dc2c4c975a98)

   d. configure "Static network"
   
   ![static-network](https://github.com/user-attachments/assets/1f6b6a82-fd17-422b-82b1-b95ed7445f91)

   e. select the default operator
    checkout the lvms operator
   ![default operator](https://github.com/user-attachments/assets/a87c577c-0158-4a98-bbcd-d52f5f390de2)

   f. add host
   
   ![Addhost](https://github.com/user-attachments/assets/a2ca2b81-dd56-43fc-b1fe-468c8606c38a)

   g. configure host
      configure the public key and proxy information, and then generate the iso
   ![configure-host](https://github.com/user-attachments/assets/4a722fb5-593e-4287-b67f-af6401f8e31d)


   f. download the iso and save it to local

   ![download image](https://github.com/user-attachments/assets/cb38e0b6-ef33-4c8e-8a47-1aa92c725850)



### **3. mount the iso to target server via BMC GUI**
#### **3.1 login the BMC network**

![ILO BMC](https://github.com/user-attachments/assets/db3b4aea-de43-45e5-9b1e-248b37d29b53)

#### **3.2 configure server boot from CD/DVD Drive**

![BOOT-FROM-CD](https://github.com/user-attachments/assets/4cc29f5d-c3bc-4efe-9e11-e4cfae913255)


#### **3.3 mount iso file**

![mount iso](https://github.com/user-attachments/assets/b61635a1-afcd-4ac9-97be-7a2e062e1556)

#### **3.4 reset server**
![reset](https://github.com/user-attachments/assets/cda771fa-e070-486c-9de4-6611af1adf75)

after reset, the iso is loaded in server and one simiple redhat OS is available, and it will trigger the connection to redhat assisted installer.




### **4. openshift container platform installation**
#### **4.1 host is detected from redhat console GUI**
if no abnormal occurred, it automatcially detect the host from redhat console GUI

#### **4.2 install OCP**
Click "install" button in redhat console GUI to start the OCP installation if 4.1 is passed.


After OCP installation is completed, download the kubeconfig file and also credentials for the console access.


### **5. openshift container platform postconfiguration**
a. performance profile.
   operformance profile is used to adjust the CPU, memory, hugepage configuration. note: configuration are specfic to the CPU models and server itself. e.g X86 and ARM are different. two different types examples are attached.
   execute the artifacts in /src/

b. drivers
   the kernel drivers depends on the requirement from container based applications. 

c. crio capability
   one patch to fix redhat issue

d. tunning profile
   sysctl adjustment

e. runtime class

f. lvms

g. volume snapshot

h. ocp registry

i.sriov operators

j. ptp sync operator

k. nfd opreator
![image](https://github.com/user-attachments/assets/091e26d7-017f-4598-8a73-eb8f6bd04f37)


l. Nvidia-gpu operators
![image](https://github.com/user-attachments/assets/331a9797-f871-4104-a12e-21f49243db8b)


## **6 Reference**
1. https://docs.openshift.com/container-platform/4.16/installing/overview/index.html
2. Performance profile. https://docs.openshift.com/container-platform/4.16/scalability_and_performance/low_latency_tuning/cnf-tuning-low-latency-nodes-with-perf-profile.html
3. 
      

   
