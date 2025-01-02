# openshift-installation-online-for-RAN
Install the openshift online


# **Descriptions**
The purpose of this repsosity is to specify the openshift installation online and also the operators configuration.  Note: all vendor specific information and configuration will be hidden due to license/privacy rule. 
The redhat document provide the full installation guide as well. this repository focus on only necessary steps for the beginners. 


# **Preconditions**
  # Hardware 
      target HPE DL110 or Dell server or Supermicro or Nvidia server (X86 or ARM CPU model)
      Jump server (optional)

  # Others
      Internet access via proxy
      redhat account
      
# **Installation steps**
  # **1. BIOS configuration**
    a. By default, the virtuallation and SR-IOV functionality are not enabled in BIOS. if those are required for the applications on top of Openshift Container Platform, those need to be enabled in BIOS configuration.
    b. For the Nvidia or supermicro server with DPU, diable the DPU in case the NIC card is used as common NIC card e.g SR-IOV functionality. see figure 1
    
Figure 1: disable DPU for supermicro/nvidia server with DPU NIC card e.g bluefield NIC 
![DPU-disabled](https://github.com/user-attachments/assets/c3169f85-9515-4012-8e8e-0736d51ff291)


  # **2. Generate redhat discovery ISO file**
  # **2.1 preparing for the following configuration information**.
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


    # **2.2 precheck the condition for openshift container platform installtion**
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
              ;api.supermicro.xxx.yyy.zzz.net. IN A
              
              ;; ANSWER SECTION:
              api.supermicro.xxx.yyy.zzz.net. 3600 IN A 10.104.xxx.yyy  -> 10.104.xxx.yyy is the infra IP
              
              ;; Query time: 1 msec
              ;; SERVER: aa.bb.cc.dd#53(aa.bb.cc.dd)  --> aa.bb.cc.dd is the DNS server IP
              ;; WHEN: Thu Jan 02 15:40:07 CST 2025
              ;; MSG SIZE  rcvd: 122

       d. NTP server
          ping IP is reachable

